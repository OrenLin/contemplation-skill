# 引力工具集轻量化迁移案例

## 项目背景

**项目类型**：3D 可视化工具集（单文件 HTML）

**目标用户**：移动端用户为主，弱网环境访问

**核心问题**：黑洞工具采用内联 base64 方案，文件 1.8MB，移动端加载 5-10 秒，几乎不可用

**项目规模**：2 个独立 3D 工具（黑洞 + 核反应堆），共约 8000 行单文件 HTML

**线上地址**：https://www.vectorfuture.xyz/gravity

---

### 设计过程

#### 1. 需求分析

**使用的工具**：`brainstorming`

**关键步骤**：
1. 识别痛点：黑洞工具 1.8MB 加载慢，用户多次反馈"打不开"
2. 评估迁移可行性：黑洞需离线运行保留原方案；新建反应堆工具采用轻量方案
3. 确定迁移目标：新工具文件体积 < 300KB，移动端首屏 < 3s

**输出**：
- 两种加载方案对比文档
- 轻量级迁移路径

#### 2. 技术选型

**关键决策**：放弃内联 base64，改用 CDN importmap

**技术栈**：
- Three.js 0.170.0（CDN importmap，版本锁定）
- 单文件 HTML（CSS + JS 全部内联）
- Vercel 部署（vercel.json rewrite 规则）

**核心代码**（CDN importmap）：
```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
  }
}
</script>
<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
// 业务代码
</script>
```

#### 3. 性能分级

**PERF 对象**（按设备分级）：
```js
const isMobile = /Mobi|Android|iPhone|iPad/i.test(navigator.userAgent);
const PERF = {
  level: isMobile ? 0 : (isTablet ? 1 : 2),  // 0=低, 1=中, 2=高
  pixelRatio: isMobile ? Math.min(dpr, 1.25) : Math.min(dpr, 2),
  shadows: !isMobile,
  antialias: !isMobile,
  maxNeutrons: isMobile ? 60 : 180,
  tubeDetail: isMobile ? 20 : 40,
  texSize: PERF.level >= 2 ? 2048 : (PERF.level >= 1 ? 1024 : 512),
};
```

#### 4. 移动端适配

**适配要点**：
- 底部抽屉式控制台（默认收起，拉手展开）
- 右上角设置按钮（图标+文字胶囊，新用户易识别）
- 触控优化：单指旋转 / 双指缩放 / 禁用平移
- safe-area-inset 适配刘海屏
- 移动端默认低画质（PERF.level=0）

**默认视角与 controls.target 同步**：
```js
if(isMobile){
  camera.position.set(26, 17, 30);            // 与 mobile_intro.pos 一致
  controls.target.set(-4, 8, -6);             // 与 mobile_intro.target 一致
  camera.fov = 58;                            // 移动端略放宽 FOV
  camera.updateProjectionMatrix();
  controls.update();
}
```

#### 5. 性能优化

**优化策略**：
- CDN 加载 three.js（180KB vs 1.8MB）
- 纹理分辨率按 PERF.level 分级（2048/1024/512）
- 纹理缓存（函数属性缓存，避免重复生成）
- InstancedMesh 渲染重复结构（燃料棒 169 根单 draw call）
- 粒子系统预分配对象池 + 隔帧更新
- 动画元素独立材质实例（避免共享材质闪烁）

**优化效果**：
- 文件体积：1.8MB → 180KB（减少 90%）
- 移动端首屏：5-10s → 1-2s
- 二次访问：< 1s（浏览器缓存 three.js）

---

### 关键决策

| 决策 | 选择 | 原因 |
|------|------|------|
| three.js 加载方式 | CDN importmap | 体积减少 90%，浏览器缓存，易维护 |
| 版本锁定 | three@0.170.0 | 避免主版本漂移导致 breaking change |
| 画质分级 | PERF 对象（0/1/2） | 移动端低画质保性能，桌面端高画质保视觉 |
| 纹理生成 | Canvas 程序纹理 | 无需外部图片，单文件交付 |
| 纹理缓存 | 函数属性缓存 | 避免重复生成阻塞主线程 |
| 重复结构 | InstancedMesh | 燃料棒 169 根单 draw call |
| 粒子系统 | 预分配 + 隔帧更新 | 减半更新频率，补偿 dt |
| 动画材质 | 独立实例 | 避免共享材质导致全部闪烁 |
| 部署 | Vercel + vercel.json rewrite | push 到 main 自动部署 |

---

### 核心成果

**量化指标**：
- 文件体积：1.8MB → 180KB（减少 90%）
- 移动端首屏（4G）：5-10s → 1-2s
- 二次访问：< 1s（浏览器缓存）
- 画质分级：3 档（低/中/高）
- 场景元素：燃料棒/控制棒/飞船/大机器人/拾荒机器人/无人机/集装箱/管道/警示灯/雷达

**用户体验**：
- 移动端秒开（CDN + 缓存）
- 按钮可识别（图标+文字胶囊）
- 默认视角合理（路人+无人机+大机器人同时入镜）
- 飞船灯光柔化（decay=2 物理衰减 + 冷色辅光）

**技术亮点**：
- CDN importmap 版本锁定
- PERF 对象按设备分级
- 程序纹理生成 + 函数属性缓存
- 动画元素独立材质实例
- 默认视角与 controls.target 同步

---

### 踩过的坑

**问题 1：内联 base64 导致文件体积爆炸（1.8MB）**
- **症状**：移动端加载 5-10 秒，弱网不可用
- **原因**：three.js 整个源码 base64 内联，体积膨胀 33%，无法缓存
- **解决方案**：改用 CDN importmap，体积降到 180KB
- **详细文档**：[single-file-html-deployment.md](../pitfalls/single-file-html-deployment.md)

**问题 2：importmap 后误插代码导致 three.js 加载失败**
- **症状**：页面白屏，loading 卡住
- **原因**：importmap 块内插入业务代码破坏 JSON 结构
- **解决方案**：importmap 独立 script 块，业务代码放 module
- **详细文档**：[single-file-html-deployment.md](../pitfalls/single-file-html-deployment.md)

**问题 3：JS 对象字面量语法错误导致整页白屏**
- **症状**：`neutrons=true` 应为 `neutrons:true`，整页白屏
- **原因**：ES module 一处语法错误导致整个模块不执行
- **解决方案**：修改后必做 vm.Script 语法检查
- **详细文档**：[single-file-html-deployment.md](../pitfalls/single-file-html-deployment.md)

**问题 4：同步生成多张大纹理阻塞主线程**
- **症状**：4 张 2048² 纹理同步生成，卡顿 2-4 秒
- **原因**：主线程被占满，无法渲染 loading
- **解决方案**：纹理分辨率按 PERF.level 分级 + 函数属性缓存
- **详细文档**：[single-file-html-deployment.md](../pitfalls/single-file-html-deployment.md)

**问题 5：共享材质导致动画元素全部闪烁**
- **症状**：修改一个 mesh 的 opacity，其他 mesh 跟着变
- **原因**：多个 mesh 共享同一材质实例
- **解决方案**：动画化元素必须独立材质实例
- **详细文档**：[single-file-html-deployment.md](../pitfalls/single-file-html-deployment.md)

**问题 6：默认视角与 controls.target 未同步**
- **症状**：初始相机位置正确但朝向错误
- **原因**：只设 camera.position 未设 controls.target
- **解决方案**：初始视角与 CAM_PRESETS 同步
- **详细文档**：[single-file-html-deployment.md](../pitfalls/single-file-html-deployment.md)

**问题 7：Vercel CDN 缓存导致"看不到更新"**
- **症状**：已部署，手机端仍是旧版本
- **原因**：CDN 缓存 + 手机浏览器缓存激进
- **解决方案**：等 1-2 分钟 / 强制刷新 / `?fresh=1` 参数 / 版本号检测兜底
- **详细文档**：[single-file-html-deployment.md](../pitfalls/single-file-html-deployment.md)

---

### 经验总结

**做得好的地方**：
1. CDN importmap 选型正确，体积减少 90%
2. PERF 对象按设备分级，平衡性能与视觉
3. 程序纹理 + 函数属性缓存，避免主线程阻塞
4. 修改后必走语法检查 + 括号平衡 + HTTP 验证流程
5. 动画元素独立材质实例，避免共享闪烁

**可以改进的地方**：
1. 初期未做 PERF 分级，导致移动端卡顿
2. 默认视角未同步 controls.target，导致朝向错误
3. 未提前考虑 Vercel CDN 缓存，用户反馈"看不到更新"

**给其他项目的建议**：
1. 单文件 HTML 优先 CDN importmap，除非需完全离线
2. 移动端必须做画质分级（PERF.level）
3. 纹理分辨率按档位分级，避免主线程阻塞
4. 动画元素必须独立材质实例
5. 修改后必走：语法检查 → 关键片段检查 → 括号平衡 → HTTP 验证
6. 部署后用 curl 验证线上代码已更新
7. 加版本号检测兜底 + `?fresh=1` 参数跳过缓存

---

### 轻量级转移迁移路径（核心经验）

将一个重量级 HTML 应用轻量级转移的 6 步：

1. **评估迁移可行性**：是否需要完全离线？
2. **剥离内联依赖**：base64 模块 → CDN importmap，版本锁定
3. **业务代码迁移**：从 base64 解出 → 放入 `<script type="module">`
4. **性能分级**：新增 PERF 对象，纹理/粒子/几何按档位分级
5. **验证**：语法检查 → 关键片段 → 括号平衡 → HTTP 200 → 真机
6. **部署与缓存兜底**：push → curl 验证 → 版本号检测 → `?fresh=1`

**迁移效果**：体积减少 90%，首屏从 5-10s 降到 1-2s，二次访问 < 1s。

---

### 参考资源

- [importmap 规范](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type/importmap)
- [Three.js CDN](https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js)
- [Vercel 部署文档](https://vercel.com/docs)
- [jsDelivr CDN](https://www.jsdelivr.com/)
- [单文件 HTML 轻量化踩坑](../pitfalls/single-file-html-deployment.md)
