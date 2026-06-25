# /extract-experience 命令

## 命令描述

从当前项目提取经验到 SKILL 文档，用于持续积累和优化产品设计知识库。

## 触发条件

用户输入 `/extract-experience` 或明确要求"提取经验"、"总结经验"、"更新 SKILL"。

## 执行流程

### 1. 识别项目上下文

首先分析当前项目的关键信息：
- 项目类型（Web 应用、移动端、3D 可视化等）
- 技术栈（React、Vue、Three.js、OGL 等）
- 核心功能特性
- 遇到的主要挑战

### 2. 提取踩坑记录

扫描项目代码和对话历史，识别以下类型的问题：

#### 2.1 布局问题
- 移动端适配问题（dvh、安全区域、触摸区域）
- 响应式布局问题（断点、媒体查询）
- z-index 层级问题
- 滚动和定位问题

#### 2.2 交互问题
- 触摸反馈缺失
- 手势冲突
- 动画性能问题
- 状态管理问题

#### 2.3 性能问题
- 首屏加载慢
- 包体积过大
- 运行时卡顿
- 内存泄漏

#### 2.4 3D/WebGL 问题
- Shader 兼容性
- 粒子系统性能
- Canvas 生命周期
- 响应式 3D 场景

#### 2.5 部署问题
- 构建配置
- 单文件打包
- CDN 部署
- 多平台适配

### 3. 格式化踩坑记录

使用 [templates/pitfall.md](../templates/pitfall.md) 模板格式化每个踩坑点：

```markdown
## 问题 N：[问题名称]

### 症状
[描述用户看到的现象]

### 根本原因
[解释为什么会发生这个问题]

### 解决方案
[提供具体的代码示例和修复步骤]

### 验证清单
- [ ] [验证项 1]
- [ ] [验证项 2]
```

### 4. 更新 pitfalls 文件

根据问题类型，将踩坑记录追加到对应的文件：

- `pitfalls/mobile-layout.md` - 移动端布局问题
- `pitfalls/interaction.md` - 交互设计问题
- `pitfalls/performance.md` - 性能优化问题
- `pitfalls/webgl-mobile-adaptation.md` - WebGL 移动端适配
- `pitfalls/3d-webgl-architecture.md` - 3D/WebGL 架构问题

如果问题类型不在现有文件中，创建新的 pitfalls 文件。

### 5. 创建项目案例

使用 [templates/project-example.md](../templates/project-example.md) 模板创建项目案例：

```markdown
# [项目名称]

## 项目背景
[描述项目目标和背景]

## 设计过程
[描述使用的设计流程和工具]

## 关键决策
[列出重要的技术决策和原因]

## 核心成果
[量化项目成果]

## 踩过的坑
[链接到详细的踩坑记录]
```

保存到 `examples/[project-name].md`。

### 6. 更新 SKILL.md 索引

在 SKILL.md 中更新以下部分：

1. **踩坑记录索引** - 添加新的 pitfalls 文件链接
2. **实际案例** - 添加新的项目案例链接
3. **版本信息** - 更新版本号（如 v1.0.0 → v1.1.0）

### 7. 生成更新报告

输出结构化的更新报告：

```markdown
## 经验提取报告

### 新增踩坑记录
- [问题名称] → pitfalls/[filename].md

### 新增项目案例
- [项目名称] → examples/[filename].md

### 更新的文件
- SKILL.md（版本 X.X.X → X.X.X）
- pitfalls/[filename].md（新增 N 个问题）
- examples/[filename].md（新建）

### 建议的后续优化
- [优化建议 1]
- [优化建议 2]
```

### 8. 提交到 Git

自动提交更新：

```bash
git add -A
git commit -m "docs: 提取 [项目名称] 项目经验

- 新增 N 个踩坑记录
- 新增项目案例
- 更新 SKILL.md 索引"
```

## 注意事项

1. **避免重复** - 检查是否已有类似的踩坑记录
2. **保持简洁** - 每个踩坑点控制在 200-300 字
3. **代码示例** - 提供可直接复制的代码片段
4. **验证清单** - 每个问题都要有验证步骤
5. **渐进式披露** - SKILL.md 只保留索引，详细内容在子文件

## 示例

### 输入
```
/extract-experience
```

### 输出
```
## 经验提取报告

### 新增踩坑记录
- AnimatePresence 导致 WebGL 上下文丢失 → pitfalls/3d-webgl-architecture.md
- z-index 层级地狱 → pitfalls/3d-webgl-architecture.md
- 粒子系统性能问题 → pitfalls/3d-webgl-architecture.md

### 新增项目案例
- 核反应堆模拟器 → examples/nuclear-reactor-design.md

### 更新的文件
- SKILL.md（版本 v1.0.0 → v1.1.0）
- pitfalls/3d-webgl-architecture.md（新增 5 个问题）
- examples/nuclear-reactor-design.md（新建）

### 建议的后续优化
- 添加 WebGL 性能基准测试
- 补充更多 3D 场景的响应式案例
```
