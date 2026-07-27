# 3D/WebGL 架构踩坑记录

## 问题 1：AnimatePresence 导致 WebGL 上下文丢失

### 症状
- Tab 切换后 3D 场景消失
- 控制台报错：WebGL context lost
- 切换回原 Tab 后场景无法恢复

### 根本原因
`AnimatePresence` 在切换 Tab 时会卸载组件，导致 Three.js Canvas 被销毁，WebGL 上下文丢失

### 解决方案
**Canvas 始终挂载，只切换可见性**：

```tsx
{/* ❌ 错误：AnimatePresence 会卸载 Canvas */}
<AnimatePresence mode="wait">
  {activeTab === 'simulator' && (
    <Canvas>
      <Scene />
    </Canvas>
  )}
</AnimatePresence>

{/* ✅ 正确：Canvas 始终挂载，用 opacity 控制可见性 */}
<motion.div
  animate={{
    opacity: activeTab === 'simulator' ? 1 : 0,
    pointerEvents: activeTab === 'simulator' ? 'auto' : 'none'
  }}
  transition={{ duration: 0.3 }}
>
  <Canvas>
    <Scene />
  </Canvas>
</motion.div>
```

**关键点**：
- Canvas 必须始终挂载在 DOM 中
- 使用 `opacity` + `pointerEvents` 控制可见性和交互
- 避免使用条件渲染（`{condition && <Canvas />}`）

---

## 问题 2：z-index 层级地狱

### 症状
- InfoPanel 覆盖控制面板和仪表盘
- 面板之间相互遮挡
- 修改 10+ 次仍无法解决

### 根本原因
- 使用 `absolute` 定位导致受滚动影响
- 多个面板 z-index 没有明确层级关系
- 3D Canvas 和 UI 面板层级混乱

### 解决方案
**建立明确的层级体系**：

```
z-0: 3D Canvas（底层）
z-10: 标题、状态指示器（装饰层）
z-20: InfoPanel（信息层）
z-30: 控制面板/仪表盘（交互层）
z-50: 顶部导航栏（最高层）
```

**使用 `fixed` 替代 `absolute`**：

```tsx
{/* ❌ 错误：absolute 受滚动影响 */}
<div className="absolute top-20 left-4 z-20">
  <ControlPanel />
</div>

{/* ✅ 正确：fixed 固定在视口 */}
<div className="fixed top-20 left-4 z-30">
  <ControlPanel />
</div>
```

**关键原则**：
- 3D Canvas 始终在最底层（z-0）
- 交互面板（控制板/仪表盘）z-index 高于信息面板
- 使用 `fixed` 定位避免滚动影响
- 顶部导航栏 z-50 最高

---

## 问题 3：粒子系统性能问题

### 症状
- 粒子数量超过 5000 后帧率下降
- 移动端卡顿严重
- 内存占用过高

### 根本原因
- 所有粒子放在同一个 Points 对象中
- 每帧更新所有粒子位置
- 未使用 GPU 加速

### 解决方案
**按物理过程分层**：

```tsx
{/* ✅ 正确：按物理过程分层 */}
<group>
  {/* 背景星空 - 3000 颗，静态 */}
  <points ref={starsRef}>
    <bufferGeometry>
      <bufferAttribute count={3000} array={stars.positions} />
    </bufferGeometry>
  </points>

  {/* 快中子 - 2000 个，高速向外 */}
  <points ref={fastNeutronsRef}>
    <bufferGeometry>
      <bufferAttribute count={2000} array={fastNeutrons.positions} />
    </bufferGeometry>
  </points>

  {/* 热中子 - 1500 个，慢速布朗运动 */}
  <points ref={thermalNeutronsRef}>
    <bufferGeometry>
      <bufferAttribute count={1500} array={thermalNeutrons.positions} />
    </bufferGeometry>
  </points>

  {/* 裂变碎片 - 1000 个，重粒子轨迹 */}
  <points ref={fissionFragmentsRef}>
    <bufferGeometry>
      <bufferAttribute count={1000} array={fissionFragments.positions} />
    </bufferGeometry>
  </points>

  {/* 伽马光子 - 800 个，高速直线 */}
  <points ref={gammaPhotonsRef}>
    <bufferGeometry>
      <bufferAttribute count={800} array={gammaPhotons.positions} />
    </bufferGeometry>
  </points>

  {/* 链式反应 - 3000 个，螺旋扩散 */}
  <points ref={chainReactionRef}>
    <bufferGeometry>
      <bufferAttribute count={3000} array={chainReaction.positions} />
    </bufferGeometry>
  </points>

  {/* 切伦科夫辐射 - 1200 个，蓝色光锥 */}
  <points ref={cherenkovRef}>
    <bufferGeometry>
      <bufferAttribute count={1200} array={cherenkov.positions} />
    </bufferGeometry>
  </points>
</group>
```

**使用 AdditiveBlending 实现发光效果**：

```tsx
<pointsMaterial
  size={0.2}
  vertexColors
  transparent
  opacity={0.8}
  sizeAttenuation
  blending={THREE.AdditiveBlending}  // 发光叠加
/>
```

**关键点**：
- 按物理过程分层，每层独立控制
- 静态粒子（背景星空）不需要每帧更新
- 动态粒子根据功率动态调整 opacity
- 使用 `AdditiveBlending` 实现发光叠加效果
- 粒子总数建议：背景 3000+ / 动态粒子 5000+

---

## 问题 4：响应式 3D 场景布局

### 症状
- PC 端和移动端布局混乱
- 移动端面板遮挡 3D 场景
- 设备检测不准确

### 根本原因
- PC 和移动端使用相同的布局
- 设备检测只依赖 User Agent
- 没有手动切换模式

### 解决方案
**三重设备检测**：

```typescript
export function useDevice() {
  const [device, setDevice] = useState<'desktop' | 'mobile'>(() => {
    // 1. User Agent 检测
    const ua = navigator.userAgent;
    const isMobileUA = /iPhone|iPad|iPod|Android/i.test(ua);

    // 2. 屏幕宽度检测
    const isSmallScreen = window.innerWidth <= 768;

    // 3. 触摸能力检测
    const hasTouch = 'ontouchstart' in window || navigator.maxTouchPoints > 0;

    // 综合判断
    const isMobile = isMobileUA && (isSmallScreen || hasTouch);

    // 从 localStorage 读取用户选择
    const saved = localStorage.getItem('device-mode');
    return (saved as 'desktop' | 'mobile') || (isMobile ? 'mobile' : 'desktop');
  });

  const setDeviceMode = (mode: 'desktop' | 'mobile') => {
    setDevice(mode);
    localStorage.setItem('device-mode', mode);
  };

  return { device, isMobile: device === 'mobile', setDeviceMode };
}
```

**PC 端布局**：

```tsx
{/* 左侧控制面板 */}
<div className="fixed top-20 left-4 z-30">
  <ControlPanel />
</div>

{/* 右侧仪表盘 */}
<div className="fixed top-20 right-4 z-30">
  <Dashboard />
</div>

{/* 中央 3D 场景 */}
<Canvas className="fixed inset-0 z-0">
  <Scene />
</Canvas>
```

**移动端布局**：

```tsx
{/* 全屏 3D 场景 */}
<Canvas className="fixed inset-0 z-0">
  <Scene />
</Canvas>

{/* 底部弹出面板 */}
<AnimatePresence>
  {mobilePanel && (
    <motion.div
      initial={{ y: '100%' }}
      animate={{ y: 0 }}
      exit={{ y: '100%' }}
      className="fixed bottom-16 left-3 right-3 z-30 max-h-[60vh] overflow-y-auto"
    >
      {mobilePanel === 'control' && <ControlPanel />}
      {mobilePanel === 'dashboard' && <Dashboard />}
    </motion.div>
  )}
</AnimatePresence>

{/* 底部操作按钮 */}
<div className="fixed bottom-4 left-4 right-4 z-20 flex gap-2">
  <button onClick={() => setMobilePanel('control')}>🎛 控制台</button>
  <button onClick={() => setMobilePanel('dashboard')}>📊 仪表盘</button>
</div>
```

**关键点**：
- 三重检测：User Agent + 屏幕宽度 + 触摸能力
- 支持手动切换 PC/Mobile 模式
- 使用 localStorage 持久化用户选择
- PC 端：左右面板 + 中央 3D 场景
- 移动端：底部弹出面板 + 全屏 3D 场景

---

## 问题 5：科学可视化参数不准确

### 症状
- 温度、压力等参数不符合实际
- 启动过程过于简单
- 缺乏科学依据

### 根本原因
- 参数随意设定，没有参考真实数据
- 启动过程没有分阶段
- 颜色变化没有物理意义

### 解决方案
**基于真实物理数据设定参数**：

```typescript
// 真实压水堆（PWR）参数
const REACTOR_CONSTANTS = {
  // 温度参数（°C）
  COLD_TEMP: 25,             // 冷态温度
  NORMAL_TEMP_MAX: 325,      // 正常运行最高温度（堆芯出口）
  WARNING_TEMP: 350,         // 警告温度阈值
  CRITICAL_TEMP: 400,        // 临界温度（燃料损伤风险）

  // 压力参数（MPa）
  COLD_PRESSURE: 0.1,        // 冷态压力
  NORMAL_PRESSURE: 15.5,     // 正常运行压力（15.5 MPa）
  WARNING_PRESSURE: 17,      // 警告压力阈值
  CRITICAL_PRESSURE: 19,     // 临界压力

  // 中子通量参数（n/cm²·s）
  MAX_NEUTRON_FLUX: 1e14,    // 最大中子通量
  CRITICAL_FLUX: 1e13,       // 临界通量

  // 效率参数
  MAX_EFFICIENCY: 35,        // 最大热效率（35%）
  MIN_EFFICIENCY: 25,        // 最低运行效率

  // 控制棒参数
  FULL_INSERTION: 100,       // 完全插入（停堆）
  FULL_EXTRACTION: 0,        // 完全抽出（最大功率）
  CRITICAL_POSITION: 30,     // 临界位置（开始链式反应）
};
```

**启动过程分阶段**：

```typescript
const [startupPhase, setStartupPhase] = useState<
  'idle' | 'preheating' | 'pressurizing' | 'critical' | 'running'
>('idle');

useEffect(() => {
  if (state.isRunning) {
    // 阶段 1：预热（温度 < 280°C）
    if (state.temperature < 280) {
      setStartupPhase('preheating');
    }
    // 阶段 2：升压（压力 < 15 MPa）
    else if (state.pressure < 15) {
      setStartupPhase('pressurizing');
    }
    // 阶段 3：临界（控制棒位置 > 30%）
    else if (state.controlRodPosition > 30) {
      setStartupPhase('critical');
    }
    // 阶段 4：稳定运行
    else {
      setStartupPhase('running');
    }
  }
}, [state.isRunning, state.temperature, state.pressure, state.controlRodPosition]);
```

**颜色随状态变化**：

```tsx
// 温度颜色映射
const getTemperatureColor = (temp: number) => {
  if (temp < 100) return '#00d4ff';      // 蓝色（冷）
  if (temp < 200) return '#00ff88';      // 绿色（温）
  if (temp < 300) return '#ffaa00';      // 橙色（热）
  return '#ff4400';                       // 红色（过热）
};

// 堆芯颜色随温度变化
<mesh>
  <sphereGeometry args={[2, 32, 32]} />
  <meshStandardMaterial
    color={getTemperatureColor(state.temperature)}
    emissive={getTemperatureColor(state.temperature)}
    emissiveIntensity={state.power / 100}
  />
</mesh>
```

**关键点**：
- 基于真实物理数据设定参数范围
- 启动过程分阶段：预热→升压→临界→稳定运行
- 颜色随状态变化：温度低→蓝，温度高→红
- 控制棒逻辑：插入越深功率越低（反比关系）

---

## 问题 6：Three.js 版本不锁定导致 ESM 链路断裂

### 症状
- 使用 `three@latest` 或未指定版本的 CDN 链接
- 某次 Three.js 升级后，importmap 导入的模块 API 变更
- 控制台报错：`Uncaught TypeError: X is not a function` 或 `is not exported`

### 根本原因
1. Three.js 在小版本间也会重命名或移除内部 API
2. examples/jsm 下的后处理模块路径与版本强绑定
3. ESM importmap 写死了模块路径，版本漂移即断裂

### 解决方案
锁定到具体版本号，禁止使用 `latest`：

```html
<!-- ❌ 错误：使用 latest -->
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@latest/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@latest/examples/jsm/"
  }
}
</script>

<!-- ✅ 正确：锁定 r170 -->
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
  }
}
</script>
```

### 验证清单
- [ ] importmap 中所有 Three.js 引用使用同一具体版本号
- [ ] 不使用 `@latest` 或无版本号
- [ ] jsDelivr/unpkg 路径与版本号一致

---

## 问题 7：EffectComposer 遗漏 OutputPass 导致画面发暗

### 症状
- 使用 UnrealBloomPass 后整体画面偏暗、偏灰
- 颜色饱和度丢失，高光区域过曝
- 与预期色调差距明显

### 根本原因
1. UnrealBloomPass 在 HDR 线性空间工作
2. 缺少 OutputPass 时，渲染管线未正确转换回 sRGB 输出空间
3. 颜色管理（ColorManagement）未完成闭环

### 解决方案
后处理链必须包含 OutputPass 作为最后一步：

```js
import { EffectComposer } from 'three/addons/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/addons/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/addons/postprocessing/UnrealBloomPass.js';
import { OutputPass } from 'three/addons/postprocessing/OutputPass.js';

const composer = new EffectComposer(renderer);
composer.addPass(new RenderPass(scene, camera));

const bloomPass = new UnrealBloomPass(
  new THREE.Vector2(window.innerWidth, window.innerHeight),
  0.8,  // strength
  0.4,  // radius
  0.85  // threshold
);
composer.addPass(bloomPass);

// ✅ 必须添加，不可省略
composer.addPass(new OutputPass());
```

### 验证清单
- [ ] 后处理链最后一步为 OutputPass
- [ ] 画面亮度与未开 Bloom 时一致或略亮
- [ ] 颜色无灰蒙感

---

## 问题 8：中文 3D 标注用 TextGeometry 失败

### 症状
- TextGeometry 渲染中文显示为方块或空白
- 需要额外加载几 MB 的中文字体 JSON
- 移动端加载缓慢

### 根本原因
1. TextGeometry 依赖 font JSON，中文字符集巨大，字体文件动辄数 MB
2. Three.js 的 font loader 对 CJK 支持有限
3. 性能与加载时间不可接受

### 解决方案
使用 CSS2DRenderer + HTML 标签替代：

```js
import { CSS2DRenderer, CSS2DObject } from 'three/addons/renderers/CSS2DRenderer.js';

// 创建标注
const labelDiv = document.createElement('div');
labelDiv.className = 'reactor-label';
labelDiv.textContent = '压力容器';
labelDiv.style.cssText = 'color:#ffcc88;font-size:12px;pointer-events:none;';

const label = new CSS2DObject(labelDiv);
label.position.set(0, 5, 0);
scene.add(label);

// 独立的 CSS2DRenderer
const labelRenderer = new CSS2DRenderer();
labelRenderer.setSize(window.innerWidth, window.innerHeight);
labelRenderer.domElement.style.position = 'absolute';
labelRenderer.domElement.style.top = '0';
labelRenderer.domElement.style.pointerEvents = 'none';
document.body.appendChild(labelRenderer.domElement);

// 渲染循环中调用
function animate() {
  composer.render();
  labelRenderer.render(scene, camera);
}
```

### 验证清单
- [ ] 中文标注正常显示，无方块
- [ ] 标注跟随 3D 物体位置
- [ ] 无额外大字体文件加载

---

## 问题 9：InstancedMesh 未用导致重复结构卡顿

### 症状
- 燃料棒、控制棒等数百个重复几何体使用独立 Mesh
- draw call 数飙升，移动端帧率骤降
- 内存占用高

### 根本原因
1. 每个 Mesh 独立提交 draw call
2. 材质与几何体未共享
3. GPU 无法批量渲染

### 解决方案
使用 InstancedMesh 单 draw call 渲染所有重复结构：

```js
const ROD_COUNT = 200;
const rodGeo = new THREE.CylinderGeometry(0.1, 0.1, 3, 12);
const rodMat = new THREE.MeshStandardMaterial({ color: 0x3a8a6a });
const instancedRods = new THREE.InstancedMesh(rodGeo, rodMat, ROD_COUNT);

const dummy = new THREE.Object3D();
for (let i = 0; i < ROD_COUNT; i++) {
  // 按环形布局
  const angle = (i / ROD_COUNT) * Math.PI * 2;
  const radius = 2.5;
  dummy.position.set(
    Math.cos(angle) * radius,
    0,
    Math.sin(angle) * radius
  );
  dummy.updateMatrix();
  instancedRods.setMatrixAt(i, dummy.matrix);
}
scene.add(instancedRods);

// ✅ 仅在位置变化时更新
function updateControlRods(position) {
  if (position !== lastPosition) {
    for (let i = 0; i < ROD_COUNT; i++) {
      dummy.position.y = position;
      dummy.updateMatrix();
      instancedRods.setMatrixAt(i, dummy.matrix);
    }
    instancedRods.instanceMatrix.needsUpdate = true;
    lastPosition = position;
  }
}
```

### 验证清单
- [ ] 重复结构使用 InstancedMesh
- [ ] draw call 数显著下降
- [ ] 矩阵仅在变化时更新

---

## 问题 10：变量初始化顺序导致 ReferenceError

### 症状
- 控制台报错：`ReferenceError: Cannot access 'X' before initialization`
- 常见于 `let`/`const` 声明的变量在声明前被引用
- 场景对象、UI 对象、配置常量最易出现

### 根本原因
1. Three.js 单文件项目中变量声明顺序随意
2. 粒子系统、UI 控件在初始化时引用了尚未声明的场景对象
3. ES 模块的 TDZ（暂时性死区）比 var 更严格

### 解决方案
提前声明关键变量，并在使用处加存在性保护：

```js
// ✅ 在文件顶部提前声明
let scene, camera, renderer, composer;
let reactorGroup, ui, neutronSystem;
let PERF = { level: 0, neutronCount: 125 };

function init() {
  scene = new THREE.Scene();
  // ...
  reactorGroup = new THREE.Group();
  scene.add(reactorGroup);
}

function initParticles() {
  // ✅ 加存在性保护
  if (!reactorGroup || !ui) {
    requestAnimationFrame(initParticles);
    return;
  }
  // ...
}
```

### 验证清单
- [ ] 关键对象在文件顶部声明
- [ ] 异步初始化函数加存在性保护
- [ ] 无 TDZ 相关的 ReferenceError

---

## 验证清单

- [ ] Canvas 始终挂载，不随 Tab 切换卸载
- [ ] z-index 层级关系明确（Canvas < InfoPanel < ControlPanel < Navbar）
- [ ] 粒子系统按物理过程分层
- [ ] 使用 AdditiveBlending 实现发光效果
- [ ] 设备检测准确（三重检测 + 手动切换）
- [ ] PC 端和移动端布局分离
- [ ] 科学参数基于真实数据
- [ ] 启动过程分阶段
- [ ] 颜色随状态变化
- [ ] importmap 中 Three.js 版本锁定
- [ ] 后处理链包含 OutputPass
- [ ] 中文标注使用 CSS2DRenderer
- [ ] 重复结构使用 InstancedMesh
- [ ] 变量初始化顺序正确，无 TDZ 错误
- [ ] 在 iOS Safari、Android Chrome、微信浏览器中测试

---

## 参考资源

- [React Three Fiber 文档](https://docs.pmnd.rs/react-three-fiber)
- [Three.js 文档](https://threejs.org/docs/)
- [Three.js r170 release notes](https://github.com/mrdoob/three.js/releases/tag/r170)
- [EffectComposer 后处理](https://threejs.org/docs/#examples/en/postprocessing/EffectComposer)
- [压水堆原理](https://zh.wikipedia.org/wiki/%E5%8E%8B%E6%B0%B4%E5%A0%86)
- [核反应堆物理](https://en.wikipedia.org/wiki/Nuclear_reactor_physics)
