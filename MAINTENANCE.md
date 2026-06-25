# SKILL 维护规则

## 维护原则

1. **渐进式披露** - SKILL.md 保持精简，详细内容按需加载
2. **单一职责** - 每个文件只负责一个主题
3. **扁平结构** - 避免深层嵌套，SKILL.md 直接链接所有资源
4. **版本控制** - 每次更新记录版本号和变更内容
5. **上下文友好** - 控制单个文件大小，避免拉爆上下文

## 文件组织结构

```
contemplation-skill/
├── SKILL.md                          # 核心指令（< 250 行）
├── MAINTENANCE.md                    # 维护规则（本文件）
├── README.md                         # 项目说明
├── commands/                         # 斜杠命令
│   ├── README.md                     # 命令索引
│   ├── extract-experience.md         # 经验提取命令
│   └── generate-docs.md              # 文档生成命令
├── templates/                        # 输出模板
│   ├── pitfall.md                    # 踩坑记录模板
│   └── project-example.md            # 项目案例模板
├── examples/                         # 实际案例
│   ├── contemplation-design.md       # 沉思工具案例
│   ├── divination-optimization.md    # 抽签工具优化案例
│   └── nuclear-reactor-design.md     # 核反应堆模拟器案例
└── pitfalls/                         # 踩坑记录
    ├── webgl-mobile-adaptation.md    # WebGL 移动端适配
    ├── mobile-layout.md              # 移动端布局
    ├── performance.md                # 性能优化
    ├── interaction.md                # 交互设计
    └── 3d-webgl-architecture.md      # 3D/WebGL 架构
```

## 更新流程

### 1. 新项目完成后

使用 `/extract-experience` 命令自动提取经验：

```bash
/extract-experience
```

**自动执行的操作**：
1. 识别项目中的踩坑点
2. 使用 `templates/pitfall.md` 格式化踩坑记录
3. 追加到对应的 `pitfalls/` 文件
4. 使用 `templates/project-example.md` 创建项目案例
5. 更新 SKILL.md 索引
6. 更新版本号

### 2. 手动更新流程

#### 2.1 添加新的踩坑记录

**步骤 1**：确定踩坑类型

| 类型 | 文件 | 适用场景 |
|------|------|----------|
| WebGL 移动端适配 | `pitfalls/webgl-mobile-adaptation.md` | Shader、纹理、渲染问题 |
| 移动端布局 | `pitfalls/mobile-layout.md` | 安全区域、视口、触摸区域 |
| 性能优化 | `pitfalls/performance.md` | 加载速度、运行时性能 |
| 交互设计 | `pitfalls/interaction.md` | 触摸反馈、手势、动画 |
| 3D/WebGL 架构 | `pitfalls/3d-webgl-architecture.md` | Canvas 生命周期、粒子系统 |
| 新类型 | 创建新文件 | 不在上述范围内的类型 |

**步骤 2**：使用模板格式化

复制 `templates/pitfall.md` 模板，填写以下内容：
- 问题描述（症状）
- 根本原因
- 解决方案（代码示例）
- 验证清单

**步骤 3**：追加到对应文件

```markdown
---

## 问题 N：[问题名称]

### 症状
[描述]

### 根本原因
[解释]

### 解决方案
[代码示例]

### 验证清单
- [ ] [验证项]
```

**步骤 4**：更新 SKILL.md 索引

在 SKILL.md 的"踩坑记录"部分添加链接：

```markdown
## 踩坑记录

1. [WebGL 移动端适配](./pitfalls/webgl-mobile-adaptation.md) - [简要描述]
2. [移动端布局](./pitfalls/mobile-layout.md) - [简要描述]
...
N. [新踩坑](./pitfalls/[filename].md) - [简要描述]
```

#### 2.2 添加新的项目案例

**步骤 1**：使用模板创建案例

复制 `templates/project-example.md`，填写以下内容：
- 项目背景
- 设计过程（6 个阶段）
- 关键决策
- 核心成果
- 踩过的坑
- 经验总结

**步骤 2**：保存到 `examples/` 目录

文件名格式：`[project-name].md`

**步骤 3**：更新 SKILL.md 索引

在 SKILL.md 的"实际案例"部分添加链接：

```markdown
## 实际案例

1. **沉思工具** - [简要描述]
   - [详细文档](./examples/contemplation-design.md)

2. **抽签工具** - [简要描述]
   - [详细文档](./examples/divination-optimization.md)

3. **[新项目]** - [简要描述]
   - [详细文档](./examples/[filename].md)
```

#### 2.3 更新版本号

遵循语义化版本规范：

- **MAJOR** (x.0.0)：不兼容的 API 变更
- **MINOR** (0.x.0)：新增功能（向后兼容）
- **PATCH** (0.0.x)：bug 修复（向后兼容）

**示例**：
- 添加新的踩坑记录：`1.0.0` → `1.1.0`
- 修复文档错误：`1.1.0` → `1.1.1`
- 重构核心流程：`1.1.1` → `2.0.0`

在 SKILL.md 的 YAML frontmatter 中更新：

```yaml
---
name: product-design-0to1
description: |
  [描述]
version: 1.1.0  # 更新这里
tags: [product-design, uiux, mobile-first, performance, skill-chain, 3d-webgl]
---
```

### 3. 提交变更

```bash
git add -A
git commit -m "docs: [变更类型]

- [变更 1]
- [变更 2]
- 更新版本号 x.x.x → x.x.x"
git push origin main
```

**提交信息规范**：

| 类型 | 说明 | 示例 |
|------|------|------|
| `feat` | 新功能 | `feat: 添加斜杠命令 /extract-experience` |
| `fix` | 修复 | `fix: 修复移动端布局问题` |
| `docs` | 文档更新 | `docs: 添加核反应堆项目案例` |
| `refactor` | 重构 | `refactor: 重构 SKILL.md 结构` |
| `chore` | 杂项 | `chore: 更新版本号` |

## 上下文控制策略

### 1. SKILL.md 大小控制

**目标**：< 250 行

**策略**：
- 只保留核心流程和索引
- 详细内容放在子文件
- 使用链接引用

### 2. 子文件大小控制

**目标**：每个文件 < 300 行

**策略**：
- 单个踩坑点 < 100 行
- 单个案例 < 300 行
- 超过限制时拆分为多个文件

### 3. 按需加载

**原则**：Agent 只在需要时读取文件

**示例**：
```markdown
## 踩坑记录

1. [WebGL 移动端适配](./pitfalls/webgl-mobile-adaptation.md)
```

Agent 看到链接后，根据当前任务决定是否读取。

### 4. 避免重复

**检查清单**：
- [ ] 是否已有类似的踩坑记录？
- [ ] 是否可以复用现有的解决方案？
- [ ] 是否可以链接到已有的文档？

## 质量保证

### 1. 代码示例

**要求**：
- 可直接复制运行
- 包含必要的上下文
- 使用 TypeScript（如适用）
- 添加注释说明

### 2. 验证清单

**要求**：
- 每个问题都要有验证步骤
- 验证步骤可执行
- 验证结果可观察

### 3. 链接检查

**要求**：
- 内部链接有效
- 外部链接可访问
- 链接文本清晰

### 4. 语言一致性

**要求**：
- 所有文档使用同一种语言（中文）
- 代码注释使用同一种语言
- 术语统一

## 常见问题

### Q1: 如何决定创建新的 pitfalls 文件还是追加到现有文件？

**判断标准**：
- 问题类型与现有文件主题一致 → 追加
- 问题类型是新的领域 → 创建新文件
- 现有文件超过 300 行 → 考虑拆分

### Q2: 如何避免上下文拉爆？

**策略**：
1. SKILL.md 保持精简（< 250 行）
2. 详细内容按需加载
3. 单个文件 < 300 行
4. 避免重复内容

### Q3: 如何处理跨领域的问题？

**策略**：
- 选择最相关的领域
- 在其他领域添加链接引用
- 必要时创建跨领域文档

### Q4: 如何保持文档的时效性？

**策略**：
1. 每次项目完成后立即提取经验
2. 定期检查链接有效性
3. 更新过时的代码示例
4. 记录版本变更历史

### Q5: 如何评估 SKILL 的质量？

**评估指标**：
- 踩坑记录数量
- 项目案例数量
- 文档完整性
- 代码示例可运行性
- 用户反馈

## 维护检查清单

### 每周检查

- [ ] 检查是否有新的踩坑需要记录
- [ ] 检查链接有效性
- [ ] 检查代码示例可运行性

### 每月检查

- [ ] 评估 SKILL 质量
- [ ] 更新过时的内容
- [ ] 优化文档结构
- [ ] 收集用户反馈

### 每季度检查

- [ ] 重构 SKILL.md（如需要）
- [ ] 更新版本号
- [ ] 清理冗余内容
- [ ] 添加新的模板

## 参考资源

- [SKILL 设计原则](https://example.com/skill-design)
- [渐进式披露最佳实践](https://example.com/progressive-disclosure)
- [上下文窗口管理](https://example.com/context-management)
