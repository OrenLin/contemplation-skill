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
- [ ] 在 iOS Safari、Android Chrome、微信浏览器中测试

---

## 参考资源

- [React Three Fiber 文档](https://docs.pmnd.rs/react-three-fiber)
- [Three.js 文档](https://threejs.org/docs/)
- [压水堆原理](https://zh.wikipedia.org/wiki/%E5%8E%8B%E6%B0%B4%E5%A0%86)
- [核反应堆物理](https://en.wikipedia.org/wiki/Nuclear_reactor_physics)
