# Obsidian 知识库自动化系统

## 项目背景
**项目类型**：Python 自动化工具集 + Obsidian 知识库架构
**目标用户**：有 100+ 笔记的 Obsidian 用户，希望系统化整理知识库
**核心问题**：手动整理 712 篇笔记的标签、链接、分类、可视化，耗时且不可持续
**项目规模**：11 个 Python 脚本（6200+ 行），4 个阶段，12 个子任务

---

### 设计过程

#### 1. 需求分析
**使用的工具**：`brainstorming`、`writing-plans`
**关键步骤**：
1. 全量扫描 712 篇笔记，统计标签/wikilink/frontmatter 现状
2. 识别 7 大领域分布失衡（健康 14 篇 vs 世界观 301 篇）
3. 设计 4 阶段升级路线：基础整理 → 战略升级 → 深度挖掘 → 可视化
**输出**：
- 健康报告（孤立笔记 174 篇，标签冲突 23 处）
- 领域平衡诊断（7 领域理想 vs 实际分布）
- 12 任务执行计划

#### 2. 架构设计
**设计决策**：
- 脚本工作目录放在 Vault 内的 `编程/` 文件夹（便于 iCloud 同步）
- 所有脚本共享 `common.py` 工具模块（路径检测/配置加载/dry-run）
- 配置外部化到 `config.yaml`，去个人化后可分享
- 统一 `--dry-run` 模式（先预览再执行）
**关键特性**：
- 11 个脚本覆盖从打标签到可视化的全链路
- 三级降级策略（环境变量 → 当前目录 → 交互输入）
- scipy 缺失时自动降级为简单分组算法

#### 3. 技术实现
**技术栈**：
- Python 3.8+（标准库为主，scipy 可选）
- PyYAML（配置文件，可选）
- Obsidian Dataview / Templater / Canvas / Galaxy View 插件
**核心代码**：
```python
# 通用 dry-run 机制
class DryRun:
    enabled = False
    
    @classmethod
    def write_text(cls, path, content):
        """dry-run 模式下只打印不写入"""
        if cls.enabled:
            print(f"[DRY-RUN] 将写入: {path} ({len(content)} 字符)")
        else:
            path.write_text(content, encoding="utf-8")
            print(f"[OK] 已写入: {path}")
```

#### 4. 深度挖掘
**挖掘维度**：
- 知识网络中心性（入度/出度/betweenness）
- 隐藏聚类（层次聚类发现 10 个数据驱动主题群）
- 思想演进链（按时间排序串联同主题笔记）
- 辩证笔记（识别对立观点并配对）
- 语义相似度链接（Jaccard ≥ 0.4 的笔记对）

#### 5. 可视化输出
**输出形态**：
- 全局 MOC 仪表盘（KPI 卡片 + 雷达图 + Dataview 块）
- Canvas 知识地图（7 领域节点 + 平衡度颜色 + 5 年愿景节点）
- HTML 深度画像报告（编辑级杂志风格）

---

### 关键决策

| 决策 | 选择 | 原因 |
|------|------|------|
| 文件读写方式 | Python 脚本（非工具直接访问） | iCloud 路径权限限制 |
| 配置管理 | config.yaml + 三级降级 | 去个人化，可分享 |
| 依赖管理 | scipy 可选 + 降级算法 | 降低安装门槛 |
| 任务编排 | 多 Agent 并行 + 阶段依赖 | 保护上下文窗口 |
| 安全机制 | 统一 --dry-run | 先预览再执行，防误操作 |
| frontmatter 解析 | 基于行扫描（非 YAML 库） | 避免 tags 列表/日期解析边界问题 |
| 可视化方案 | Dataview + Mermaid + Canvas | Obsidian 原生渲染，无外部依赖 |
| 分享包 | 去个人化脚本 + agent.md + HTML 指南 | 让任何人能从 0 复现 |

---

### 核心成果
**量化指标**：
- 笔记总数：712 篇
- 新增 wikilink：3376 条（含 83 条语义链接）
- 生成枢纽笔记：8 个（OKR 化）
- 生成桥接笔记：30 个
- 生成演进链：6 条
- 生成辩证笔记：5 篇
- 领域平衡诊断：7 领域（最失衡：健康 13%）
- 脚本总数：11 个（6200+ 行）
- 去个人化后可分享：✅

**系统产出**：
- 健康报告 + 中心性报告 + 聚类报告 + 盲点报告
- 可视化仪表盘 MOC（16 区块 + 8 Dataview 块）
- Canvas 知识地图（42 节点 + 40 边）
- 5 年愿景笔记（7 领域 OKR）
- 分享包（agent.md + 11 脚本 + HTML 指南）

**技术亮点**：
- 三级配置降级（环境变量 → 目录检测 → 交互输入）
- scipy 缺失时优雅降级为简单分组
- 基于行的 frontmatter 解析器（避免 YAML 边界问题）
- Wikilink 去重（覆盖 `[[target]]`、`[[target|alias]]`、`[[target#heading]]`）
- Canvas JSON 安全保存（颜色字符串化 + 中文不转义）

---

### 踩过的坑

**问题 1：iCloud 路径无法用工具直接访问**
- **症状**：Read/Edit 工具对 iCloud 路径返回权限错误
- **原因**：iCloud 沙盒权限 + 路径特殊字符
- **解决方案**：统一用 Python 脚本读写
- **详细文档**：[obsidian-vault-automation.md](../pitfalls/obsidian-vault-automation.md)

**问题 2：Frontmatter 解析边界情况**
- **症状**：追加字段写到正文区域、tags 格式不一致
- **原因**：YAML 库对 Obsidian frontmatter 的兼容性问题
- **解决方案**：基于行扫描的自定义解析器
- **详细文档**：[obsidian-vault-automation.md](../pitfalls/obsidian-vault-automation.md)

**问题 3：硬编码个人数据导致不可分享**
- **症状**：换一个 vault 完全无法运行
- **原因**：路径/主题映射/关键词全部硬编码
- **解决方案**：config.yaml + 三级降级检测
- **详细文档**：[obsidian-vault-automation.md](../pitfalls/obsidian-vault-automation.md)

**问题 4：Dataview/Mermaid 语法兼容性**
- **症状**：仪表盘 Dataview 块不渲染、Mermaid 饼图不显示
- **原因**：FROM 语法缺空字符串、pie 语法有空行
- **解决方案**：严格遵循 Obsidian 解析规范
- **详细文档**：[obsidian-vault-automation.md](../pitfalls/obsidian-vault-automation.md)

**问题 5：多 Agent 并行上下文撑爆**
- **症状**：712 篇笔记全量扫描超出单 Agent 上下文窗口
- **原因**：未做任务拆分和并行编排
- **解决方案**：分阶段并行，sub-agent 只返回摘要
- **详细文档**：[obsidian-vault-automation.md](../pitfalls/obsidian-vault-automation.md)

---

### 经验总结
**做得好的地方**：
1. 先 dry-run 预览再执行，避免误操作
2. 配置外部化，让脚本可分享
3. 多 Agent 并行编排，保护上下文窗口
4. 去个人化处理，生成完整的分享包

**可以改进的地方**：
1. 初期硬编码路径，后期返工去个人化
2. frontmatter 解析应一开始就用自定义解析器
3. 应提前设计任务依赖图，避免串行等待

**给其他项目的建议**：
1. 任何涉及文件系统操作的工具，第一天就做配置外部化
2. 面向 Obsidian 的脚本，用 Python 读写而非工具直接访问
3. 大批量数据处理时，用 sub-agent 并行 + 只返回摘要
4. 分享前做一轮去个人化审查

---

### 参考资源
- [Obsidian Dataview 文档](https://blacksmithgu.github.io/obsidian-dataview/)
- [Obsidian Canvas 格式](https://help.obsidian.md/Canvas/Canvas+format)
- [Obsidian Templater](https://silentvoid13.github.io/Templater/)
- [Mermaid 语法](https://mermaid.js.org/)
- [Galaxy View 插件](https://github.com/NexusSteve/obsidian-galaxy)