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
- **实战经验**：8 个真实项目案例 + 9 份详细踩坑记录

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
│   ├── nuclear-reactor-design.md     # 核反应堆模拟器案例
│   ├── piecewise-optimization.md     # 分段线性函数优化案例
│   ├── gravity-tools-lightweight.md  # 引力工具集轻量化迁移案例
│   ├── obsidian-knowledge-system.md  # Obsidian 知识库自动化案例
│   ├── hand-ar-spaceship.md          # Hand-AR 飞船展板案例
│   └── protein-analysis.md          # 蛋白质结构可视化案例
└── pitfalls/                         # 踩坑记录
    ├── webgl-mobile-adaptation.md        # WebGL 移动端适配
    ├── mobile-layout.md                  # 移动端布局问题
    ├── performance.md                    # 性能优化
    ├── interaction.md                    # 交互设计
    ├── 3d-webgl-architecture.md          # 3D/WebGL 架构
    ├── math-optimization-reasoning.md    # 数学优化推理
    ├── single-file-html-deployment.md    # 单文件 HTML 轻量化部署
    ├── obsidian-vault-automation.md      # Obsidian Vault 自动化
    └── unity-ar-hand-tracking.md          # Unity AR 手部追踪
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
- 数学优化推理 → `pitfalls/math-optimization-reasoning.md`
- 单文件 HTML 部署 → `pitfalls/single-file-html-deployment.md`
- Obsidian Vault 自动化 → `pitfalls/obsidian-vault-automation.md`
- Unity AR 手部追踪 → `pitfalls/unity-ar-hand-tracking.md`

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

### 案例 4：分段线性函数优化分配

**项目类型**：数学优化推理

**核心亮点**：
- 分段线性函数最小化问题
- 排名约束下的资源分配
- 两阶段枚举 + 构造求解

**详细文档**：[examples/piecewise-optimization.md](./examples/piecewise-optimization.md)

### 案例 5：引力工具集轻量化迁移

**项目类型**：单文件 HTML 轻量化部署

**核心亮点**：
- 体积从 1.8MB 降至 180KB（减少 90%）
- CDN importmap 加载 three.js
- 材质共享 + Vercel 缓存策略

**详细文档**：[examples/gravity-tools-lightweight.md](./examples/gravity-tools-lightweight.md)

### 案例 7：Hand-AR 数字化宇宙飞船展板

**项目类型**：Unity AR 手势交互应用

**核心亮点**：
- Unity + MediaPipe 手部追踪，21 点关键点
- 三手势系统（张手爆炸 / 握拳还原 / 食指蓝图）
- 三程序集分离架构（核心/MediaPipe/Editor 独立）
- MCP 正交坐标系旋转，稳定性提升 5 倍
- 捏合三重滤波（迟滞 + 防抖 + 朝向校验）

**详细文档**：[examples/hand-ar-spaceship.md](./examples/hand-ar-spaceship.md)

### 案例 8：蛋白质结构可视化工具

**项目类型**：生物信息学 Web 可视化

**核心亮点**：
- Molstar 5.5.0 + CDN iframe 隔离，绕过 Next.js 16 Turbopack 循环依赖
- React StrictMode iframe 复用 + 延迟清理，解决 3D 结构画布空白
- IntersectionObserver 懒加载 + 200px 预加载，避免多实例 WebGL 竞争
- 三层降级兜底（3D → PAE 热图 → 文件下载）

**详细文档**：[examples/protein-analysis.md](./examples/protein-analysis.md)

### 案例 6：Obsidian 知识库自动化系统

**项目类型**：Python 自动化工具集 + Obsidian 知识库架构

**核心亮点**：
- 11 个 Python 脚本（6200+ 行），4 阶段 12 子任务
- 712 篇笔记全量整理（新增 3376 条 wikilink）
- 配置外部化 + 三级降级，去个人化后可分享
- 多 Agent 并行编排，保护上下文窗口

**详细文档**：[examples/obsidian-knowledge-system.md](./examples/obsidian-knowledge-system.md)

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
- React StrictMode 下 iframe 重复创建导致 Molstar 渲染失败

**解决方案**：
- Canvas 始终挂载，用 opacity 切换可见性
- 明确 z-index 层级体系（Canvas z-0 < InfoPanel z-20 < ControlPanel z-30 < Navbar z-50）
- 粒子按物理过程分层（快中子、热中子、裂变碎片等）
- 三重设备检测 + 手动切换
- 基于真实物理数据校准参数
- iframe 复用 + 延迟清理 + postMessage 状态同步

**详细文档**：[pitfalls/3d-webgl-architecture.md](./pitfalls/3d-webgl-architecture.md)

### 6. 数学优化推理

**核心问题**：
- 常数项倍乘误判
- 临界点跨越处理不当
- 枚举与构造脱节
- 边际相等简化失效

**解决方案**：
- 多角度验证（枚举 + 构造 + 边际分析）
- 临界点显式判断，避免隐式跨越
- 常数项独立处理，避免倍乘干扰

**详细文档**：[pitfalls/math-optimization-reasoning.md](./pitfalls/math-optimization-reasoning.md)

### 7. 单文件 HTML 轻量化部署

**核心问题**：
- 内联 base64 导致体积爆炸
- CDN importmap 配置复杂
- 材质资源重复加载
- Vercel 缓存策略不当

**解决方案**：
- 优先 CDN importmap 加载 three.js
- 材质共享，避免重复实例化
- 轻量级转移策略（blackhole 1.8MB → reactor 180KB）

**详细文档**：[pitfalls/single-file-html-deployment.md](./pitfalls/single-file-html-deployment.md)

### 9. Unity AR 手部追踪

**核心问题**：
- Mock 模式覆盖真实手部数据
- 相机背景 Canvas 遮挡 3D 模型
- 前置摄像头坐标轴翻转
- 短向量旋转计算不稳定
- 捏合手势误触发
- 插件菜单在 Tuanjie Hub 中不显示
- Z 距离与旋转跳变

**解决方案**：
- 反射禁用 Mock 模式
- ScreenSpaceCamera + planeDistance=50
- 前置摄像头 X 翻转（x = 1 - x）
- MCP 正交坐标系替代短向量（5x 稳定）
- 三重滤波（迟滞双阈值 + 防抖 + 掌心朝向）
- 移除外层条件编译，方法内条件检查
- Z 钳制 0.15~2m + 旋转跳变 >60° 丢弃

**详细文档**：[pitfalls/unity-ar-hand-tracking.md](./pitfalls/unity-ar-hand-tracking.md)

### 8. Obsidian Vault 自动化

**核心问题**：
- iCloud 路径无法用工具直接访问
- Frontmatter 解析边界情况
- 硬编码个人数据导致不可分享
- Wikilink 去重、Canvas JSON、Dataview 语法兼容性
- 多 Agent 并行上下文撑爆

**解决方案**：
- 统一用 Python 脚本读写 Vault
- 基于行扫描的自定义 frontmatter 解析器
- config.yaml + 三级降级配置
- 分阶段并行，sub-agent 只返回摘要

**详细文档**：[pitfalls/obsidian-vault-automation.md](./pitfalls/obsidian-vault-automation.md)

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

### Obsidian 自动化参考
- [Obsidian Dataview](https://blacksmithgu.github.io/obsidian-dataview/) - 笔记查询
- [Obsidian Canvas 格式](https://help.obsidian.md/Canvas/Canvas+format) - Canvas JSON
- [Obsidian Templater](https://silentvoid13.github.io/Templater/) - 模板引擎
- [Galaxy View 插件](https://github.com/NexusSteve/obsidian-galaxy) - 星系可视化

## 📝 版本历史

- **v2.7.0** (2026-07-31)
  - 新增蛋白质结构可视化工具项目案例（Molstar + iframe 隔离，StrictMode 复用，三层降级兜底）
  - 3D/WebGL 架构踩坑新增问题 11（React StrictMode 下 iframe 重复创建导致 Molstar 渲染失败）
  - 三处索引同步（SKILL/MAINTENANCE/README）

- **v2.6.0** (2026-07-27)
  - 新增 Hand-AR 数字化宇宙飞船展板项目案例（Unity + MediaPipe，18 脚本，7 踩坑）
  - 新增 Unity AR 手部追踪踩坑记录（7 个问题：Mock 覆盖、Canvas 遮挡、坐标翻转、旋转不稳定、捏合误触、Hub 菜单、Z 距离跳变）
  - tags 新增 unity-ar
  - 三处索引同步（SKILL/MAINTENANCE/README）

- **v2.5.1** (2026-07-27)
  - **SKILL.md 规范化适配 TRAE 导入**（依据 [官方 Skill 文档](https://docs.trae.cn/ide_skills)）
  - `description` 多行块标量 → 单行（符合官方示例，避免自动填充异常）
  - 新增"何时使用"节（提升 AI 自动激活命中率）
  - "斜杠命令"章节改名"约定指令"，明确非 TRAE 内置斜杠命令，需自然语言触发
  - 新增"导入说明"节（zip 上传格式、存放路径、激活方式）
  - version 2.5.0 → 2.5.1

- **v2.5.0** (2026-07-27)
  - **脱敏 gate**：`extract-experience` 新增第 7.5 步强制脱敏审查（识别类/技术类/数据类/个人信息 4 类检查项）
  - **review gate**：取消自动 push，改为生成变更摘要 + 人工确认后才推送（公开仓库不可逆）
  - **三处索引闭环**：第 6 步从"只更新 SKILL.md"改为同步更新 SKILL/MAINTENANCE/README，附同步检查清单
  - **单一信源**：第 4 步删除过时的类型列表，改为引用 MAINTENANCE.md 类型映射表
  - **模板脱敏字段**：project-example 新增"脱敏声明"，pitfall 新增"脱敏提示"
  - tags 新增 `desensitization-gate`

- **v2.4.1** (2026-07-27)
  - 文档同步：MAINTENANCE.md、README.md 与 SKILL.md v2.4.0 对齐
  - 补全文件树、案例概览、踩坑索引、版本历史
  - 类型映射表新增单文件 HTML 部署、Obsidian Vault 自动化

- **v2.4.0** (2026-07-27)
  - 新增 Obsidian 知识库自动化项目案例（11 脚本 6200+ 行，712 笔记全量整理）
  - 新增 Obsidian Vault 自动化踩坑记录（8 个问题）
  - tags 新增 obsidian-vault-automation

- **v2.3.0** (2026-07-27)
  - 新增引力工具集轻量化迁移项目案例（1.8MB → 180KB，减少 90%）
  - 新增单文件 HTML 轻量化部署踩坑记录
  - tags 新增 single-file-html、cdn-importmap

- **v2.2.0** (2026-07-27)
  - 核反应堆模拟器追加末日废土版 v2 迭代案例
  - 3D/WebGL 架构、性能、移动端布局踩坑扩充

- **v2.1.0** (2026-07-11)
  - 新增分段线性函数优化分配项目案例
  - 新增数学优化推理踩坑记录（5 个问题）
  - MAINTENANCE.md 文件树与类型映射表同步

- **v2.0.0** (2026-06)
  - SKILL 结构重构：引入 commands/、templates/、MAINTENANCE.md 多项目经验积累框架
  - 斜杠命令体系（/extract-experience、/generate-docs）
  - 标准化踩坑与案例输出模板

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
