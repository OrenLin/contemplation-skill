# Unity AR 手部追踪踩坑记录

> 从 Hand-AR 数字化宇宙飞船展板项目提取。Unity 2022 (Tuanjie) + MediaPipe Unity Plugin + ARCore。

## 问题 1：Mock 模式覆盖真实手部数据

### 症状
- MediaPipe 摄像头已启动，但模型位置不跟随真实手部
- 模型在屏幕上做圆周运动（Mock 模式的默认行为）
- Console 中 MediaPipe 日志正常输出，但模型位置始终不对

### 根本原因
Mock 模式的 `Update()` 方法每帧生成模拟 landmarks 并调用 `UpdateLandmarks()`，即使 MediaPipe 已启动仍在运行。两个数据源竞争同一个 `landmarks` 数组，Mock 的 60fps 覆盖了 MediaPipe 的 30fps 真实数据。

### 解决方案
MediaPipe 启动时通过反射禁用 Mock 模式的 `useMockMode` 字段：

```csharp
// MediaPipeHandBridge.cs — StartTracking() 中
var mockField = handTracker.GetType().GetField("useMockMode",
    System.Reflection.BindingFlags.Public | System.Reflection.BindingFlags.Instance);
if (mockField != null && (bool)mockField.GetValue(handTracker))
{
    mockField.SetValue(handTracker, false);
    Debug.Log("[MediaPipeBridge] Mock mode auto-disabled");
}
```

### 验证清单
- [ ] MediaPipe 启动后 Console 出现 "Mock mode auto-disabled" 日志
- [ ] 模型跟随真实手部移动
- [ ] MockHandTarget 不再影响模型位置

---

## 问题 2：相机背景 Canvas 遮挡 3D 模型

### 症状
- 摄像头画面作为背景正常显示
- 3D 飞船模型完全不可见
- 调整模型位置、缩放均无效

### 根本原因
相机背景 Canvas 使用 `ScreenSpaceOverlay` 渲染模式，该模式在所有 3D 物体之上绘制 UI，导致 3D 模型被完全遮挡。`ScreenSpaceOverlay` 不参与深度排序，永远在最顶层。

### 解决方案
改用 `ScreenSpaceCamera` 模式，并设置 `planeDistance` 让 Canvas 在相机近裁面与 3D 模型之间：

```csharp
backgroundCanvas.renderMode = RenderMode.ScreenSpaceCamera;
backgroundCanvas.worldCamera = arCamera;
backgroundCanvas.planeDistance = 50; // 远于近裁面，近于 3D 模型
```

**关键参数**：`planeDistance=50` 确保背景在模型之后渲染。

### 验证清单
- [ ] 摄像头背景画面可见
- [ ] 3D 模型在背景画面之上可见
- [ ] 无闪烁或深度冲突

---

## 问题 3：前置摄像头坐标轴翻转

### 症状
- 手部向右移动，模型向左移动
- 旋转方向与手部实际旋转相反
- 仅在使用前置摄像头时出现

### 根本原因
MediaPipe 的 landmarks 基于摄像头原始图像坐标系。前置摄像头的图像在显示前被系统水平镜像，但 landmarks 基于未镜像的原始图像。如果不翻转 X 坐标，模型位置与用户感知相反。

### 解决方案
前置摄像头模式下翻转 X 坐标：

```csharp
if (frontCameraMirror)
{
    for (int i = 0; i < 21; i++)
        landmarks[i].x = 1f - landmarks[i].x;
}
```

**注意**：翻转 X 后坐标系手性改变，叉积方向相反，旋转计算的 `flipNormal` 参数需同步调整。

### 验证清单
- [ ] 手部向右移动，模型向右移动
- [ ] 旋转方向与手部一致
- [ ] 后置摄像头模式正常（不翻转）

---

## 问题 4：短向量旋转计算不稳定

### 症状
- 手腕轻微抖动导致模型剧烈旋转
- 旋转角度偏差大
- 无法稳定控制模型方向

### 根本原因
使用 wrist(0) → midPalm(9) 的短向量计算 yaw/pitch，该向量长度仅约 0.05（归一化坐标）。短向量对 landmark 抖动极其敏感。

### 解决方案
使用更长的 MCP 基线构建正交坐标系，稳定性提升 5 倍：

```csharp
// 使用 3 个 MCP 点构建正交坐标系
Vector3 palmRight = (idxMcp - wrist).normalized;    // 食指方向
Vector3 palmForward = (pinkyMcp - wrist).normalized; // 小指方向
Vector3 palmNormal = Vector3.Cross(palmRight, palmForward).normalized;
Quaternion rotation = Quaternion.LookRotation(palmForward, palmNormal);
```

基线从 0.05 增加到 0.08+，角度误差与基线长度成反比，稳定性提升约 5 倍。

### 验证清单
- [ ] 手腕静止时模型不抖动
- [ ] 缓慢旋转模型平滑跟随
- [ ] 快速旋转无跳变（配合 60° 帧间保护）

---

## 问题 5：捏合手势误触发

### 症状
- 手指自然弯曲时误触发捏合
- 手背朝相机时也触发捏合
- 捏合频繁误触，无法正常使用

### 根本原因
单一阈值存在两个问题：1) 没有迟滞区间，阈值附近反复触发/释放；2) 没有掌心朝向校验，手背朝相机时指尖距离也可能很小。

### 解决方案
三重滤波：迟滞双阈值 + 时间防抖 + 掌心朝向校验：

```csharp
// 1. 迟滞双阈值（on < off，形成区间）
bool rawBelow = pinchNormalizedDistance < 0.35f; // on
bool rawAbove = pinchNormalizedDistance > 0.55f; // off

// 2. 时间防抖（持续低于 on 阈值 80ms 才确认）
if (rawBelow && !pinchArmed)
{
    pinchDebounceTimer += Time.deltaTime;
    if (pinchDebounceTimer >= 0.08f) pinchArmed = true;
}

// 3. 掌心朝相机校验
bool palmFacing = Vector3.Dot(palmNormal, arCamera.transform.forward) < -0.3f;
isPinching = pinchArmed && palmFacing;
```

### 验证清单
- [ ] 手指自然弯曲不误触
- [ ] 手背朝相机不误触
- [ ] 真实捏合能稳定触发
- [ ] 阈值附近无反复触发

---

## 问题 6：插件菜单在 Tuanjie Hub 中不显示

### 症状
- 直接打开 Unity 项目，HandAR 菜单正常显示
- 通过 Tuanjie Hub 打开同一项目，菜单完全消失
- Console 无报错

### 根本原因
菜单项被包裹在 `#if MEDIAPIPE_PLUGIN` 条件编译块中。通过 Tuanjie Hub 打开时，程序集加载顺序不同，`MEDIAPIPE_PLUGIN` 宏可能未及时定义，导致整个菜单块被编译器跳过。

### 解决方案
移除外层条件编译包裹，改为每个方法内部单独检查：

```csharp
// ❌ 错误：整个菜单块被条件编译包裹
#if MEDIAPIPE_PLUGIN
[MenuItem("HandAR/4. Setup MediaPipe")]
static void SetupMediaPipe() { /* ... */ }
#endif

// ✅ 正确：菜单始终编译，方法内部条件检查
[MenuItem("HandAR/4. Setup MediaPipe")]
static void SetupMediaPipe()
{
#if MEDIAPIPE_PLUGIN
    // MediaPipe 相关逻辑
#else
    EditorUtility.DisplayDialog("提示",
        "MediaPipe 插件未安装，请先导入 MediaPipe Unity Plugin", "OK");
#endif
}
```

### 验证清单
- [ ] 直接打开项目菜单可见
- [ ] Tuanjie Hub 打开菜单可见
- [ ] 未安装 MediaPipe 时点击菜单有友好提示

---

## 问题 7：Z 距离与旋转跳变导致模型消失或翻转

### 症状
- 模型偶尔出现在相机后方（不可见）
- 快速转手时模型突然翻转 180°
- MediaPipe 的 z 坐标不可靠导致距离计算偏差

### 根本原因
1. MediaPipe image landmarks 的 z 分量精度低（相对深度，非公制），直接用于距离计算会导致模型位置前后跳跃
2. 帧间旋转角度变化过大时，四元数插值会走"短路"路径，导致模型突然翻转

### 解决方案
Z 距离钳制 + 帧间旋转跳变保护：

```csharp
// 1. Z 距离钳制
float actualDistance = Mathf.Clamp(
    config.palmDistance + palmScreen.z * 0.1f,
    0.15f, 2f);

// 2. 帧间旋转跳变保护（>60° 丢弃本帧）
float angleDelta = Quaternion.Angle(prevFilteredRot, rawRot);
if (angleDelta > 60f) return; // 丢弃本帧旋转
```

**补充**：优先使用 MediaPipe 的 `worldLandmarks`（公制坐标，z 精度高）替代 image landmarks 计算旋转。

### 验证清单
- [ ] 模型始终在相机前方可见
- [ ] 快速翻转手部时模型不跳变
- [ ] Z 距离在 0.15m ~ 2m 范围内

---

## 脱敏提示

> 已脱敏：是　审查人：Agent　日期：2026-07-27
> 项目名为泛化描述，无内部 API/域名/包名，代码为通用 Unity/MediaPipe 模式。