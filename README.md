# 产品设计 0 到 1 SKILL

> 从需求分析到部署上线的完整产品设计方法论，结合顶级 UI/UX 设计原则和实战踩坑经验

## 📖 简介

这是一套完整的产品设计和实现流程 SKILL，涵盖从 0 到 1 的每个阶段。通过真实项目案例（沉思工具、核反应堆模拟器等）总结出的最佳实践和踩坑记录，帮助开发者快速掌握产品设计要点。

## 🎯 核心特性

- **完整流程**：6 个阶段覆盖产品设计全生命周期
- **工具链协作**：brainstorming → writing-plans → frontend-design → test-driven-development → executing-plans → dogfood
- **斜杠命令**：快速执行常用任务（`/brainstorm`、`/extract-experience`、`/generate-docs`）
- **输出模板**：标准化的踩坑记录和项目案例格式
- **移动优先**：所有设计决策优先考虑移动端体验
- **性能至上**：每个技术选型都考虑性能影响
- **实战经验**：3 个真实项目案例 + 5 份详细踩坑记录

## 📚 文档结构

```
contemplation-skill/
├── SKILL.md                          # 主技能文档（核心方法论，< 250 行）
├── MAINTENANCE.md                    # 维护规则（多项目经验积累）
├── README.md                         # 本文件
├── commands/                         # 斜杠命令
│   ├── README.md                     # 命令索引
│   ├── extract-experience.md         # 经验提取命令
│   └── generate-docs.md              # 文档生成命令
├── templates/                        # 输出模板
│   ├── pitfall.md                    # 踩坑记录模板
│   └── project-example.md            # 项目案例模板
├── examples/                         # 实战案例
│   ├── contemplation-design.md       # 沉思工具设计案例
│   ├── divination-optimization.md    # 抽签工具优化案例
│   └── nuclear-reactor-design.md     # 核反应堆模拟器案例
└── pitfalls/                         # 踩坑记录
    ├── webgl-mobile-adaptation.md    # WebGL 移动端适配
    ├── mobile-layout.md              # 移动端布局问题
    ├── performance.md                # 性能优化
    ├── interaction.md                # 交互设计
    └── 3d-webgl-architecture.md      # 3D/WebGL 架构
```

## 🚀 快速开始

### 1. 安装 SKILL

将 `SKILL.md` 文件添加到你的 TRAE IDE 的 skills 目录中。

### 2. 使用斜杠命令

本 SKILL 提供多个斜杠命令快速执行常用任务：

- `/brainstorm` - 启动需求头脑风暴
- `/plan` - 生成实现计划
- `/build` - 执行构建流程
- `/extract-experience` - 从当前项目提取经验到 SKILL
- `/generate-docs` - 生成产品文档和介绍

详细命令说明见 [commands/README.md](./commands/README.md)

### 3. 使用流程

按照 SKILL.md 中的 6 个阶段顺序执行：

1. **需求分析与设计** - 使用 `brainstorming` 和 `writing-plans`
2. **UI/UX 设计** - 使用 `frontend-design` 和 `theme-factory`
3. **技术实现** - 使用 `test-driven-development` 和 `executing-plans`
4. **移动端适配** - 参考 `pitfalls/mobile-layout.md`
5. **性能优化** - 参考 `pitfalls/performance.md`
6. **测试与发布** - 使用 `dogfood` 进行最终测试

### 4. 查阅踩坑记录

在开发过程中遇到问题时，查阅对应的 pitfalls 文档：

- WebGL 效果问题 → `pitfalls/webgl-mobile-adaptation.md`
- 布局问题 → `pitfalls/mobile-layout.md`
- 性能问题 → `pitfalls/performance.md`
- 交互问题 → `pitfalls/interaction.md`
- 3D 架构问题 → `pitfalls/3d-webgl-architecture.md`

### 5. 提取项目经验

项目完成后，使用 `/extract-experience` 命令自动提取经验：

1. 识别踩过的坑 → 添加到对应 `pitfalls/` 文件
2. 总结关键决策 → 添加到 `examples/` 目录
3. 更新 SKILL.md 索引 → 保持文档同步

详细维护规则见 [MAINTENANCE.md](./MAINTENANCE.md)

## 📋 核心原则

1. **移动优先**：所有设计决策优先考虑移动端体验
2. **性能至上**：每个技术选型都要考虑性能影响
3. **用户中心**：所有功能围绕用户实际需求设计
4. **渐进增强**：基础功能优先，高级功能逐步添加
5. **可维护性**：代码结构清晰，便于后续迭代

## 🛠️ SKILL 工具链

| 阶段 | 工具 | 用途 |
|------|------|------|
| 需求分析 | `brainstorming` | 头脑风暴，探索产品方向 |
| 计划制定 | `writing-plans` | 制定详细实现计划 |
| UI 设计 | `frontend-design` | 创建高质量前端界面 |
| 主题系统 | `theme-factory` | 多主题管理 |
| 测试驱动 | `test-driven-development` | TDD 开发 |
| 执行计划 | `executing-plans` | 按计划实现功能 |
| 探索测试 | `dogfood` | 发布前最终测试 |

## 📖 案例概览

### 案例 1：沉思工具

**项目类型**：沉浸式语录浏览工具

**核心亮点**：
- 10 个主题，每个主题 10 条双语语录
- WebGL 特效背景（OGL 实现）
- 三种交互方式：点击、箭头、滑动
- 主包体积优化：1018KB → 519KB（减少 49%）

**详细文档**：[examples/contemplation-design.md](./examples/contemplation-design.md)

### 案例 2：抽签工具优化

**项目类型**：移动端抽签工具 UX 优化

**核心亮点**：
- 解决底部按钮被导航栏遮挡问题
- 使用 `dvh` 替代 `vh` 避免布局抖动
- 安全区域适配（刘海屏、灵动岛）

**详细文档**：[examples/divination-optimization.md](./examples/divination-optimization.md)

### 案例 3：核反应堆模拟器

**项目类型**：3D 交互式核反应堆教育可视化

**核心亮点**：
- 3D 反应堆结构（安全壳 + 压力容器 + 堆芯）
- 10500+ 粒子系统（按物理过程分 7 层）
- 基于真实 PWR 参数（325°C、15.5MPa）
- 启动过程分阶段（预热→升压→临界→运行）
- 三重设备检测 + 手动切换 PC/Mobile 模式

**详细文档**：[examples/nuclear-reactor-design.md](./examples/nuclear-reactor-design.md)

## 🔍 踩坑记录索引

### 1. WebGL 移动端适配

**核心问题**：Shader 效果在移动端过度放大

**解决方案**：
- 使用 `gl_FragCoord` 归一化替代 `vUv`
- `uResolution` 使用设备像素而非 CSS 像素
- 添加 `uScale` uniform 控制整体缩放

**详细文档**：[pitfalls/webgl-mobile-adaptation.md](./pitfalls/webgl-mobile-adaptation.md)

### 2. 移动端布局

**核心问题**：
- 底部按钮被导航栏遮挡
- 主题切换器与无障碍按钮冲突
- CSS 动画重置滚动位置
- PC 端折叠按钮不明显
- z-index 层级地狱

**详细文档**：[pitfalls/mobile-layout.md](./pitfalls/mobile-layout.md)

### 3. 性能优化

**核心问题**：
- 主包体积过大（1018KB）
- 图片加载慢
- 长列表渲染卡顿
- 滚动事件性能问题

**解决方案**：
- 动态导入 + Vite 分包
- 图片懒加载 + WebP
- 虚拟滚动（react-window）
- 防抖节流

**详细文档**：[pitfalls/performance.md](./pitfalls/performance.md)

### 4. 交互设计

**核心问题**：
- 触摸反馈缺失
- 手势冲突
- 新手引导设计不当
- 多交互方式冲突

**详细文档**：[pitfalls/interaction.md](./pitfalls/interaction.md)

### 5. 3D/WebGL 架构

**核心问题**：
- AnimatePresence 导致 WebGL 上下文丢失
- z-index 层级地狱（多面板重叠）
- 粒子系统性能问题
- 响应式 3D 场景布局
- 科学可视化参数不准确

**解决方案**：
- Canvas 始终挂载，用 opacity 切换可见性
- 明确 z-index 层级体系（Canvas z-0 < InfoPanel z-20 < ControlPanel z-30 < Navbar z-50）
- 粒子按物理过程分层（快中子、热中子、裂变碎片等）
- 三重设备检测 + 手动切换
- 基于真实物理数据校准参数

**详细文档**：[pitfalls/3d-webgl-architecture.md](./pitfalls/3d-webgl-architecture.md)

## ✅ 检查清单

### 设计阶段
- [ ] 产品定位明确
- [ ] 核心功能清晰
- [ ] 信息架构合理
- [ ] 交互流程顺畅
- [ ] 设计规范完整

### 实现阶段
- [ ] 代码结构清晰
- [ ] 性能考虑充分
- [ ] 测试覆盖完整
- [ ] 文档齐全

### 发布阶段
- [ ] 功能测试通过
- [ ] 兼容性测试通过
- [ ] 性能测试通过
- [ ] 可访问性测试通过
- [ ] 生产环境验证通过

## 🔗 相关资源

### 设计资源
- [reactbits.dev](https://www.reactbits.dev) - React 组件库，包含 WebGL 效果
- [The Book of Shaders](https://thebookofshaders.com/) - Shader 学习资源
- [OGL 文档](https://github.com/oframe/ogl) - 轻量级 WebGL 库

### 技术资源
- [React 文档](https://react.dev/) - React 官方文档
- [Vite 文档](https://vitejs.dev/) - Vite 构建工具
- [Tailwind CSS](https://tailwindcss.com/) - 原子化 CSS 框架
- [React Three Fiber](https://docs.pmnd.rs/react-three-fiber) - R3F 文档
- [Three.js](https://threejs.org/docs/) - Three.js 文档

### 核反应堆项目参考
- [压水堆原理](https://zh.wikipedia.org/wiki/%E5%8E%8B%E6%B0%B4%E5%A0%86)
- [核反应堆物理](https://en.wikipedia.org/wiki/Nuclear_reactor_physics)

## 📝 版本历史

- **v1.1.0** (2026-06-22)
  - 新增核反应堆模拟器案例
  - 新增 3D/WebGL 架构踩坑记录
  - 更新移动端布局踩坑（折叠按钮、z-index 层级）
  - 添加 3D/WebGL 项目架构阶段

- **v1.0.0** (2026-06-20)
  - 初始版本
  - 沉思工具设计案例
  - 抽签工具优化案例
  - 4 份踩坑记录

## 🤝 贡献

欢迎提交 Issue 和 Pull Request 来完善这个 SKILL。

## 📄 许可证

MIT License

## 💡 使用建议

1. **按需查阅**：不必按顺序阅读所有文档，遇到问题时查阅对应的 pitfalls
2. **结合实践**：每个案例都包含完整的设计过程和决策理由，建议结合实际项目理解
3. **灵活运用**：6 个阶段不必严格遵循，可根据项目特点调整
4. **持续更新**：踩坑记录会持续补充，欢迎贡献你的经验

## 🎓 学习路径

**初学者**：
1. 阅读 SKILL.md 了解整体流程
2. 查看 contemplation-design.md 了解完整案例
3. 遇到具体问题时查阅对应的 pitfalls

**进阶者**：
1. 直接查阅 nuclear-reactor-design.md 了解复杂 3D 项目
2. 阅读 3d-webgl-architecture.md 了解 WebGL 架构要点
3. 参考检查清单确保项目质量

**专家**：
1. 贡献新的踩坑记录
2. 优化现有文档
3. 分享你的项目案例
