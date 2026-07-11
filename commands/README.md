# 斜杠命令

使用 `/` 前缀快速执行常用任务。

## 可用命令

### 需求分析阶段

- `/brainstorm` - 启动需求头脑风暴
  - 使用 `brainstorming` 技能探索产品方向
  - 明确产品定位、目标用户、核心功能

- `/plan` - 生成实现计划
  - 使用 `writing-plans` 制定详细实现计划
  - 包含功能清单、技术选型、里程碑

### 构建阶段

- `/build` - 执行构建流程
  - 使用 `executing-plans` 按计划实现功能
  - 包含代码实现、测试、优化

### 经验提取

- `/extract-experience` - 从当前项目提取经验到 SKILL
  - 识别踩过的坑 → 添加到对应 `pitfalls/` 文件
  - 总结关键决策 → 添加到 `examples/` 目录
  - 更新 SKILL.md 索引 → 保持文档同步
  - 详见 [extract-experience.md](./extract-experience.md)

### 文档生成

- `/generate-docs` - 生成产品文档和介绍
  - 生成 README.md、产品介绍、使用文档
  - 详见 [generate-docs.md](./generate-docs.md)

## 使用方式

在对话中输入 `/` 加上命令名称即可触发。例如：

```
/brainstorm
```

系统会自动加载对应的命令定义和执行流程。
