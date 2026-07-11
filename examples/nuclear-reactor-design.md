# 核反应堆模拟器设计案例

## 项目背景

创建一个高质量的 3D 交互式核反应堆网页，用于教育目的。要求：
- 深色未来感设计
- 中央 3D 核反应堆可视化
- 参数控制面板和数据仪表盘
- 真实压水堆（PWR）物理参数
- 响应式设计（PC + 移动端）
- 科学准确的粒子效果

## 设计过程

### 1. 需求探索（brainstorming）

**核心问题**：
- 目标用户：对核能感兴趣的学生和公众
- 核心价值：通过可视化理解核反应原理
- 关键功能：3D 反应堆、参数控制、数据监控、原理学习

**技术选型决策**：
- 3D 渲染：React Three Fiber（R3F）+ Three.js
- 后处理：@react-three/postprocessing（Bloom、ChromaticAberration）
- 动画：Framer Motion
- 样式：Tailwind CSS v4
- 构建：Vite + vite-plugin-singlefile（单文件打包）

### 2. 布局架构演进

**初始方案（失败）**：
```tsx
// ❌ 问题：InfoPanel 覆盖控制面板
<div className="relative">
  <Canvas />
  <ControlPanel />
  <Dashboard />
  <InfoPanel />  // 遮挡了其他面板
</div>
```

**问题**：
- InfoPanel 使用 `absolute` 定位，覆盖在控制面板上方
- 修改 10+ 次 z-index 仍无法解决
- 滚动时面板位置错乱

**最终方案（成功）**：
```tsx
// ✅ 明确层级体系
<div className="relative w-full min-h-screen">
  {/* z-0: 3D Canvas 底层 */}
  <Canvas className="fixed inset-0 z-0" />

  {/* z-20: InfoPanel 信息层 */}
  <div className="fixed bottom-0 left-0 right-0 z-20">
    <InfoPanel />
  </div>

  {/* z-30: 控制面板/仪表盘 交互层 */}
  <div className="fixed top-20 left-4 z-30">
    <ControlPanel />
  </div>
  <div className="fixed top-20 right-4 z-30">
    <Dashboard />
  </div>

  {/* z-50: 顶部导航 最高层 */}
  <nav className="fixed top-0 left-0 right-0 z-50" />
</div>
```

**关键原则**：
- Canvas 始终在底层（z-0）
- 信息面板中层（z-20）
- 交互面板高层（z-30）
- 导航栏最高层（z-50）
- 使用 `fixed` 替代 `absolute` 避免滚动影响

### 3. Tab 切换架构

**问题**：
- 使用 AnimatePresence 切换 Tab 时卸载 Canvas
- WebGL 上下文丢失，切换回来场景消失

**解决方案**：
```tsx
// ✅ Canvas 始终挂载，用 opacity 控制可见性
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

{/* UI 面板用 AnimatePresence */}
<AnimatePresence mode="wait">
  {activeTab === 'simulator' ? <SimulatorUI /> : <LearningUI />}
</AnimatePresence>
```

### 4. 响应式设计

**三重设备检测**：
```typescript
export function useDevice() {
  const [device, setDevice] = useState<'desktop' | 'mobile'>(() => {
    // 1. User Agent
    const isMobileUA = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);

    // 2. 屏幕宽度
    const isSmallScreen = window.innerWidth <= 768;

    // 3. 触摸能力
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
- 左侧：控制面板（可折叠）
- 右侧：仪表盘（可折叠）
- 中央：3D 场景
- 底部：原理学习模块

**移动端布局**：
- 全屏：3D 场景
- 底部：操作按钮（控制台/仪表盘）
- 弹出面板：点击按钮后从底部滑出
- 折叠提示：明显的"▼ 点击折叠面板"按钮

### 5. 科学可视化

**真实物理参数**：
```typescript
const REACTOR_CONSTANTS = {
  // 温度（°C）
  COLD_TEMP: 25,
  NORMAL_TEMP_MAX: 325,      // PWR 堆芯出口温度
  WARNING_TEMP: 350,
  CRITICAL_TEMP: 400,

  // 压力（MPa）
  COLD_PRESSURE: 0.1,
  NORMAL_PRESSURE: 15.5,     // PWR 运行压力
  WARNING_PRESSURE: 17,
  CRITICAL_PRESSURE: 19,

  // 中子通量（n/cm²·s）
  MAX_NEUTRON_FLUX: 1e14,
  CRITICAL_FLUX: 1e13,

  // 效率
  MAX_EFFICIENCY: 35,        // 核电站热效率 33-35%
  MIN_EFFICIENCY: 25,
};
```

**启动过程分阶段**：
1. **预热阶段**：温度从 25°C 升至 280°C
2. **升压阶段**：压力从 0.1 MPa 升至 15.5 MPa
3. **临界过程**：控制棒抽出至 30% 位置，开始链式反应
4. **稳定运行**：功率达到目标值，冷却剂循环

**粒子系统科学分层**（10500+ 粒子）：
- **背景星空**（3000 颗）：球形分布，多色（白/青/绿/紫）
- **快中子**（2000 个）：青绿色，高速向外辐射（模拟 2MeV 快中子）
- **热中子**（1500 个）：黄绿色，慢速布朗运动（慢化后的低能中子）
- **裂变碎片**（1000 个）：橙红色，重粒子轨迹（U-235 裂变产物）
- **伽马光子**（800 个）：紫白色，高速直线（裂变瞬间释放）
- **链式反应**（3000 个）：金红色，螺旋扩散（中子诱发新裂变）
- **切伦科夫辐射**（1200 个）：蓝色，光锥（带电粒子超光速运动）

**视觉效果**：
- 双层等离子体核心（白色内核 + 绿色外层）
- 三层冲击波（橙→红→深红，递进色温）
- 16 条能量射线（从 8 条增至 16 条）
- 三维临界警告光环（多角度多层）
- AdditiveBlending 实现发光叠加

### 6. 折叠交互设计

**PC 端折叠按钮**：
```tsx
{/* 折叠按钮 - 面板顶部 */}
<motion.button
  onClick={() => setPanelVisible(false)}
  className="w-full mb-2 py-1.5 rounded-lg bg-black/80 backdrop-blur-md border border-cyan-400/40 text-cyan-400 text-[11px] font-bold tracking-wider flex items-center justify-center gap-2 font-body hover:bg-cyan-500/10 hover:border-cyan-400/70 transition-all"
  whileHover={{ scale: 1.02 }}
  whileTap={{ scale: 0.98 }}
>
  <span>◀ 点击折叠控制台</span>
</motion.button>

{/* 展开按钮 - 折叠后显示 */}
<motion.button
  onClick={() => setPanelVisible(true)}
  className="fixed top-20 left-4 z-30 py-3 px-3 rounded-lg bg-black/80 backdrop-blur-md border border-cyan-400/60 text-cyan-400 text-[11px] font-bold tracking-wider flex items-center gap-2 font-body hover:bg-cyan-500/15 hover:border-cyan-400 transition-all shadow-[0_0_15px_rgba(0,212,255,0.2)]"
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.95 }}
>
  <span className="animate-pulse">▶</span>
  <span>展开控制台</span>
</motion.button>
```

**移动端折叠提示**：
```tsx
{/* 弹出面板顶部 */}
<motion.button
  onClick={() => setMobilePanel(null)}
  className="w-full mb-2 py-2 rounded-lg bg-black/80 backdrop-blur-md border border-cyan-400/40 text-cyan-400 text-xs font-bold tracking-wider flex items-center justify-center gap-2 font-body"
  whileTap={{ scale: 0.97 }}
>
  <span>▼ 点击折叠面板</span>
</motion.button>

{/* 未打开时的提示 - 底部浮动 */}
{!mobilePanel && (
  <motion.div
    className="absolute bottom-20 left-1/2 -translate-x-1/2 z-10 pointer-events-none"
    animate={{ y: [0, -3, 0] }}
    transition={{ duration: 1.5, repeat: Infinity }}
  >
    <span className="text-[10px] text-cyan-400 tracking-widest font-mono">▲ 点击上方按钮操作</span>
  </motion.div>
)}
```

### 7. 部署适配

**问题**：
- axureshow 要求根目录必须有 `index.html`
- 构建产物在 `dist/` 子目录

**解决方案**：
```bash
# 复制到根目录
cp dist/index.html index.html
cp dist/index.html nuclear-reactor-standalone.html
```

**单文件打包**：
```typescript
// vite.config.ts
import { viteSingleFile } from 'vite-plugin-singlefile'

export default defineConfig({
  plugins: [react(), tailwindcss(), viteSingleFile()],
  build: {
    assetsInlineLimit: 100000000,  // 100MB，所有资源内联
    chunkSizeWarningLimit: 100000,
    rollupOptions: {
      output: {
        inlineDynamicImports: true,
      },
    },
  },
})
```

**输出**：
- `dist/index.html`（标准构建）
- `index.html`（根目录，供 axureshow 部署）
- `nuclear-reactor-standalone.html`（独立文件，可直接打开）

## 关键决策

| 决策 | 选择 | 原因 |
|------|------|------|
| 3D 库 | R3F + Three.js | 生态成熟，文档完善 |
| 后处理 | @react-three/postprocessing | 与 R3F 无缝集成 |
| 布局方案 | Tab 切换 | 避免面板重叠问题 |
| 设备检测 | 三重检测 + 手动切换 | 准确且灵活 |
| 粒子分层 | 按物理过程分层 | 科学准确，性能可控 |
| 打包方案 | vite-plugin-singlefile | 单文件部署，便于分享 |

## 踩过的坑

1. **z-index 层级地狱**：修改 10+ 次仍无法解决，最终建立明确层级体系
2. **Canvas 生命周期**：AnimatePresence 卸载导致 WebGL 上下文丢失
3. **面板重叠**：InfoPanel 覆盖控制面板，改用 Tab 切换架构
4. **响应式布局**：PC 和移动端使用相同布局导致混乱，改为完全分离
5. **科学参数**：初始参数随意设定，后基于真实 PWR 数据校正
6. **粒子性能**：所有粒子放一起导致卡顿，按物理过程分层后性能提升
7. **部署问题**：axureshow 要求根目录有 index.html，需复制构建产物

## 详细踩坑记录

- [3D/WebGL 架构踩坑](../pitfalls/3d-webgl-architecture.md)
- [移动端布局踩坑](../pitfalls/mobile-layout.md)（问题 8-9）

## 项目成果

- **粒子总数**：从 2800 提升至 10500+
- **科学准确性**：基于真实 PWR 参数
- **响应式**：PC 和移动端完全分离，支持手动切换
- **交互体验**：折叠按钮明显，提示清晰
- **部署灵活**：支持多种部署方式（axureshow、BytePlus、本地打开）

## 参考资源

- [压水堆原理](https://zh.wikipedia.org/wiki/%E5%8E%8B%E6%B0%B4%E5%A0%86)
- [核反应堆物理](https://en.wikipedia.org/wiki/Nuclear_reactor_physics)
- [React Three Fiber 文档](https://docs.pmnd.rs/react-three-fiber)
- [Three.js 文档](https://threejs.org/docs/)
