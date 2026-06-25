# 项目案例模板

## 项目名称

### 项目背景

**项目类型**：[Web 应用 / 移动端 / 3D 可视化 / 库 / SDK]

**目标用户**：[描述目标用户群体]

**核心问题**：[解决什么问题]

**项目规模**：[代码量、开发周期、团队规模]

---

### 设计过程

#### 1. 需求分析

**使用的工具**：`brainstorming`、`writing-plans`

**关键步骤**：
1. [步骤 1]
2. [步骤 2]
3. [步骤 3]

**输出**：
- [输出 1]
- [输出 2]

#### 2. UI/UX 设计

**使用的工具**：`frontend-design`、`theme-factory`

**设计决策**：
- [决策 1]
- [决策 2]

**关键特性**：
- [特性 1]
- [特性 2]

#### 3. 技术实现

**使用的工具**：`test-driven-development`、`executing-plans`

**技术栈**：
- [技术 1]
- [技术 2]

**核心代码**：

```typescript
// 关键代码片段
```

#### 4. 移动端适配

**适配要点**：
- [要点 1]
- [要点 2]

**解决方案**：
```typescript
// 代码示例
```

#### 5. 性能优化

**优化策略**：
- [策略 1]
- [策略 2]

**优化效果**：
- [指标 1]：[数值]
- [指标 2]：[数值]

---

### 关键决策

| 决策 | 选择 | 原因 |
|------|------|------|
| [决策 1] | [选择] | [原因] |
| [决策 2] | [选择] | [原因] |
| [决策 3] | [选择] | [原因] |

---

### 核心成果

**量化指标**：
- [指标 1]：[数值]
- [指标 2]：[数值]
- [指标 3]：[数值]

**用户体验**：
- [体验 1]
- [体验 2]

**技术亮点**：
- [亮点 1]
- [亮点 2]

---

### 踩过的坑

**问题 1：[问题名称]**

- **症状**：[描述]
- **原因**：[描述]
- **解决方案**：[描述]
- **详细文档**：[链接到 pitfalls/xxx.md]

**问题 2：[问题名称]**

- **症状**：[描述]
- **原因**：[描述]
- **解决方案**：[描述]
- **详细文档**：[链接到 pitfalls/xxx.md]

---

### 经验总结

**做得好的地方**：
1. [经验 1]
2. [经验 2]

**可以改进的地方**：
1. [改进 1]
2. [改进 2]

**给其他项目的建议**：
1. [建议 1]
2. [建议 2]

---

### 参考资源

- [资源 1](链接)
- [资源 2](链接)
- [资源 3](链接)

---

## 使用示例

# 核反应堆模拟器

### 项目背景

**项目类型**：3D 教育可视化

**目标用户**：对核能感兴趣的学生和公众

**核心问题**：通过可视化理解核反应原理

**项目规模**：约 5000 行代码，2 周开发

---

### 设计过程

#### 1. 需求分析

**使用的工具**：`brainstorming`、`writing-plans`

**关键步骤**：
1. 明确教育目标：让公众理解核反应堆工作原理
2. 确定核心功能：3D 可视化、参数控制、数据监控
3. 设计用户流程：启动→监控→学习

**输出**：
- PRD 文档
- 功能清单（15 个核心功能）
- 信息架构图

#### 2. UI/UX 设计

**使用的工具**：`frontend-design`、`theme-factory`

**设计决策**：
- 深色未来感设计：符合科技感主题
- Tab 切换架构：避免面板重叠问题
- PC/移动端完全分离：不同设备不同布局

**关键特性**：
- 3D 反应堆结构（安全壳 + 压力容器 + 堆芯）
- 10500+ 粒子系统（按物理过程分 7 层）
- 启动过程分阶段（预热→升压→临界→运行）

#### 3. 技术实现

**使用的工具**：`test-driven-development`、`executing-plans`

**技术栈**：
- React 18 + TypeScript
- React Three Fiber + Three.js
- @react-three/postprocessing
- Tailwind CSS v4
- Vite + vite-plugin-singlefile

**核心代码**：

```typescript
// 粒子系统科学分层
<group>
  {/* 快中子 - 2000 个 */}
  <points ref={fastNeutronsRef}>
    <bufferGeometry>
      <bufferAttribute count={2000} array={fastNeutrons.positions} />
    </bufferGeometry>
  </points>

  {/* 热中子 - 1500 个 */}
  <points ref={thermalNeutronsRef}>
    <bufferGeometry>
      <bufferAttribute count={1500} array={thermalNeutrons.positions} />
    </bufferGeometry>
  </points>
</group>
```

#### 4. 移动端适配

**适配要点**：
- 三重设备检测（User Agent + 屏幕宽度 + 触摸能力）
- 手动切换 PC/Mobile 模式
- 底部弹出面板 + 折叠提示

**解决方案**：
```typescript
export function useDevice() {
  const [device, setDevice] = useState<'desktop' | 'mobile'>(() => {
    const isMobileUA = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent);
    const isSmallScreen = window.innerWidth <= 768;
    const hasTouch = 'ontouchstart' in window;
    const isMobile = isMobileUA && (isSmallScreen || hasTouch);
    
    const saved = localStorage.getItem('device-mode');
    return (saved as 'desktop' | 'mobile') || (isMobile ? 'mobile' : 'desktop');
  });

  return { device, isMobile: device === 'mobile', setDeviceMode: setDevice };
}
```

#### 5. 性能优化

**优化策略**：
- 粒子按物理过程分层（7 层独立控制）
- 使用 AdditiveBlending 实现发光效果
- 静态粒子不每帧更新

**优化效果**：
- 粒子总数：2800 → 10500+
- 帧率：稳定 60fps
- 内存占用：降低 30%

---

### 关键决策

| 决策 | 选择 | 原因 |
|------|------|------|
| 3D 库 | R3F + Three.js | 生态成熟，后处理丰富 |
| 布局方案 | Tab 切换 | 避免面板重叠问题 |
| 设备检测 | 三重检测 + 手动切换 | 准确且灵活 |
| 粒子分层 | 按物理过程分层 | 科学准确，性能可控 |
| 打包方案 | vite-plugin-singlefile | 单文件部署，便于分享 |

---

### 核心成果

**量化指标**：
- 粒子总数：10500+（从 2800 提升）
- 科学准确性：基于真实 PWR 参数
- 响应式：支持 PC 和移动端

**用户体验**：
- 3D 交互流畅（60fps）
- 折叠按钮明显，提示清晰
- 启动过程科学准确

**技术亮点**：
- Canvas 始终挂载，避免 WebGL 上下文丢失
- 明确 z-index 层级体系
- 粒子系统科学分层（快中子、热中子、裂变碎片等）

---

### 踩过的坑

**问题 1：AnimatePresence 导致 WebGL 上下文丢失**

- **症状**：Tab 切换后 3D 场景消失
- **原因**：AnimatePresence 卸载 Canvas 导致 WebGL 上下文丢失
- **解决方案**：Canvas 始终挂载，用 opacity 切换可见性
- **详细文档**：[3d-webgl-architecture.md](../pitfalls/3d-webgl-architecture.md)

**问题 2：z-index 层级地狱**

- **症状**：InfoPanel 覆盖控制面板和仪表盘
- **原因**：多个面板 z-index 没有明确层级关系
- **解决方案**：建立明确层级体系（Canvas z-0 < InfoPanel z-20 < ControlPanel z-30 < Navbar z-50）
- **详细文档**：[3d-webgl-architecture.md](../pitfalls/3d-webgl-architecture.md)

**问题 3：面板重叠问题**

- **症状**：InfoPanel 覆盖控制面板，修改 10+ 次仍无法解决
- **原因**：使用 absolute 定位导致受滚动影响
- **解决方案**：改用 Tab 切换架构，PC/移动端完全分离
- **详细文档**：[3d-webgl-architecture.md](../pitfalls/3d-webgl-architecture.md)

---

### 经验总结

**做得好的地方**：
1. 基于真实物理数据设定参数（PWR：325°C、15.5MPa）
2. 粒子系统按物理过程分层，科学准确
3. 三重设备检测 + 手动切换，响应式灵活

**可以改进的地方**：
1. 初期布局方案选择错误，导致 10+ 次修复
2. 没有提前考虑 Canvas 生命周期问题
3. 移动端适配可以更早期介入

**给其他项目的建议**：
1. 3D 项目优先考虑 Canvas 生命周期
2. 建立明确的 z-index 层级体系
3. PC/移动端布局完全分离，不要尝试响应式适配

---

### 参考资源

- [压水堆原理](https://zh.wikipedia.org/wiki/%E5%8E%8B%E6%B0%B4%E5%A0%86)
- [核反应堆物理](https://en.wikipedia.org/wiki/Nuclear_reactor_physics)
- [React Three Fiber 文档](https://docs.pmnd.rs/react-three-fiber)
