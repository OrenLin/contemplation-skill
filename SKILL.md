---
name: product-design-0to1
description: |
  从 0 到 1 的完整产品设计流程，结合顶级 UI/UX 设计原则和 SKILL 工具链协作。
  当用户需要创建新产品、优化现有产品、或进行产品设计决策时使用此技能。
  支持：需求分析、UI/UX 设计、技术实现、移动端适配、性能优化、测试发布。
version: 2.0.0
tags: [product-design, uiux, mobile-first, performance, skill-chain, 3d-webgl]
---

# 产品设计 0 到 1 SKILL

## 快速开始

1. 确定当前阶段：需求分析 / UI 设计 / 技术实现 / 移动端适配 / 测试发布
2. 按需加载对应资源文件
3. 使用斜杠命令快速执行常用任务

## 核心原则

1. **移动优先** - 所有设计优先考虑移动端
2. **性能至上** - 每个技术选型考虑性能影响
3. **用户中心** - 功能围绕实际需求设计
4. **渐进增强** - 基础功能优先，高级功能逐步添加
5. **可维护性** - 代码结构清晰，便于迭代

## 完整流程

### 阶段 1：需求分析与设计

**使用工具**：`brainstorming`、`writing-plans`

**关键步骤**：
1. 产品定位（解决什么问题？目标用户？）
2. 功能规划（MVP、优先级、依赖关系）
3. 信息架构（页面结构、导航逻辑）
4. 交互设计（用户流程、关键场景）

**输出**：PRD、功能清单、信息架构图

**参考**：[examples/contemplation-design.md](./examples/contemplation-design.md)

---

### 阶段 2：UI/UX 设计

**使用工具**：`frontend-design`、`frontend-skill`、`theme-factory`

**设计原则**：
- 视觉层次（字体、颜色、间距）
- 一致性（颜色、字体、间距、圆角系统）
- 反馈机制（加载状态、错误提示）
- 可访问性（对比度、键盘导航、屏幕阅读器）
- 情感化设计（微交互、动画）

**移动端要点**：
- 触摸友好（≥44x44px）
- 安全区域（`env(safe-area-inset-*)`）
- 视口适配（`dvh` 替代 `vh`）

**参考**：[pitfalls/mobile-layout.md](./pitfalls/mobile-layout.md)

---

### 阶段 3：技术实现

**使用工具**：`test-driven-development`、`executing-plans`

**技术栈选择**：
- 轻量优先（OGL > Three.js，Zustand > Redux）
- 生态成熟（社区活跃、TypeScript 支持）
- 性能导向

**代码分割**：
```typescript
build: {
  rollupOptions: {
    output: {
      manualChunks: {
        'vendor-react': ['react', 'react-dom'],
        'vendor-state': ['zustand'],
        'vendor-webgl': ['ogl'],
      },
    },
  },
}
```

**3D/WebGL 项目**：
- Canvas 生命周期：始终挂载，用 opacity 切换可见性
- z-index 层级：Canvas(z-0) < InfoPanel(z-20) < ControlPanel(z-30) < Navbar(z-50)
- 粒子系统：按物理过程分层，使用 AdditiveBlending
- 响应式布局：PC/移动端完全分离，三重设备检测

**参考**：[pitfalls/3d-webgl-architecture.md](./pitfalls/3d-webgl-architecture.md)

---

### 阶段 4：移动端适配

**使用工具**：`dogfood`

**适配清单**：
- [ ] `100dvh` 替代 `100vh`
- [ ] `env(safe-area-inset-*)` 处理安全区域
- [ ] 触摸区域 ≥44x44px
- [ ] 无横向溢出
- [ ] 折叠按钮明显且易用

**参考**：[pitfalls/mobile-layout.md](./pitfalls/mobile-layout.md)

---

### 阶段 5：性能优化

**目标**：
- 主包 < 500KB
- 首屏 < 3s
- Lighthouse > 90

**策略**：
- 动态导入 + Vite 分包
- 图片懒加载 + WebP
- 虚拟滚动（长列表）
- 防抖节流

**参考**：[pitfalls/performance.md](./pitfalls/performance.md)

---

### 阶段 6：测试与发布

**测试清单**：
- [ ] 功能测试（核心功能、边界情况）
- [ ] 兼容性测试（iOS Safari、Android Chrome、微信浏览器）
- [ ] 性能测试（首屏 < 3s、交互 < 100ms）
- [ ] 可访问性测试（键盘导航、屏幕阅读器）

**发布流程**：
```bash
npm run build
git add -A
git commit -m "feat: 完成功能 X"
git push origin main
```

---

## 斜杠命令

使用 `/` 前缀快速执行常用任务：

- `/brainstorm` - 启动需求头脑风暴
- `/plan` - 生成实现计划
- `/build` - 执行构建流程
- `/extract-experience` - 从当前项目提取经验到 SKILL
- `/generate-docs` - 生成产品文档和介绍

详见 [commands/](./commands/) 目录

---

## 工具链

| 阶段 | 工具 | 用途 |
|------|------|------|
| 需求 | `brainstorming` | 探索产品方向 |
| 计划 | `writing-plans` | 制定实现计划 |
| UI | `frontend-design` | 创建前端界面 |
| 主题 | `theme-factory` | 多主题管理 |
| 测试 | `test-driven-development` | TDD 开发 |
| 执行 | `executing-plans` | 按计划实现 |
| 验证 | `dogfood` | 探索性测试 |

---

## 踩坑记录

1. [WebGL 移动端适配](./pitfalls/webgl-mobile-adaptation.md) - Shader 放大、UV 归一化
2. [移动端布局](./pitfalls/mobile-layout.md) - 安全区域、折叠按钮、z-index
3. [性能优化](./pitfalls/performance.md) - 代码分割、主包优化
4. [交互设计](./pitfalls/interaction.md) - 触摸反馈、手势支持
5. [3D/WebGL 架构](./pitfalls/3d-webgl-architecture.md) - Canvas 生命周期、粒子分层

---

## 实际案例

1. **沉思工具** - 沉浸式语录浏览，10 主题，主包优化 49%
   - [详细文档](./examples/contemplation-design.md)

2. **抽签工具** - 移动端 UX 优化，解决按钮遮挡
   - [详细文档](./examples/divination-optimization.md)

3. **核反应堆模拟器** - 3D 教育可视化，10500+ 粒子，科学参数
   - [详细文档](./examples/nuclear-reactor-design.md)

---

## 维护规则

新项目完成后，使用 `/extract-experience` 命令提取经验：

1. 识别踩过的坑 → 添加到对应 `pitfalls/` 文件
2. 总结关键决策 → 添加到 `examples/` 目录
3. 更新 SKILL.md 索引 → 保持文档同步

详见 [MAINTENANCE.md](./MAINTENANCE.md)

---

## 检查清单

### 设计阶段
- [ ] 产品定位明确
- [ ] 核心功能清晰
- [ ] 信息架构合理
- [ ] 交互流程顺畅

### 实现阶段
- [ ] 代码结构清晰
- [ ] 性能考虑充分
- [ ] 测试覆盖完整

### 发布阶段
- [ ] 功能测试通过
- [ ] 兼容性测试通过
- [ ] 性能测试通过
- [ ] 生产环境验证通过

---

## 外部资源

### 设计
- [reactbits.dev](https://www.reactbits.dev) - React 组件库
- [The Book of Shaders](https://thebookofshaders.com/) - Shader 学习
- [OGL 文档](https://github.com/oframe/ogl) - 轻量 WebGL

### 技术
- [React](https://react.dev/) - 官方文档
- [Vite](https://vitejs.dev/) - 构建工具
- [Tailwind CSS](https://tailwindcss.com/) - 原子化 CSS
- [React Three Fiber](https://docs.pmnd.rs/react-three-fiber) - R3F 文档
