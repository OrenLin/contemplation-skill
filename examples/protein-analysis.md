# 蛋白质结构可视化工具

### 项目背景

**项目类型**：Web 应用（生物信息学可视化）

**目标用户**：科研人员 / 学生 / 对蛋白质结构感兴趣的公众

**核心问题**：在浏览器中可视化 AlphaFold 预测的蛋白质 3D 结构，并集成到基因分析报告中，无需安装 PyMOL / ChimeraX 等桌面工具

**项目规模**：约 600 行核心组件代码，含 iframe 隔离方案与多层兜底机制

---

### 设计过程

#### 1. 需求分析

**关键步骤**：
1. 明确数据源：AlphaFold DB 提供的 mmCIF / BCIF / PDB 结构文件 + PAE 置信度热图
2. 确定核心功能：3D 结构交互（旋转/缩放/平移）、懒加载、渲染失败兜底、原始文件下载
3. 设计集成方式：作为报告页面的子组件，每个基因对应一个 3D 结构卡片

**输出**：
- 组件接口定义（geneName / uniprotAccession / species / forceVisible）
- 多层兜底策略（3D 渲染 → PAE 热图 → 文件下载）

#### 2. UI/UX 设计

**设计决策**：
- 深色画布背景（`bg-gray-900`）突出 3D 结构
- 分阶段加载提示（10s / 25s / 60s）而非单一超时
- 静态兜底模式：渲染失败时自动切换到 PAE 热图 + 下载链接
- 懒加载占位：未进入视口时显示轻量占位，可手动"立即加载"

**关键特性**：
- 拖动旋转 · 滚轮缩放 · 右键平移
- AlphaFold DB 外链（版本号可见）
- 渲染状态实时反馈（渲染中… / 渲染完成）

#### 3. 技术实现

**技术栈**：
- Next.js 16 + React + TypeScript
- Molstar 5.5.0（via CDN，iframe 隔离）
- IntersectionObserver 懒加载
- postMessage 跨 iframe 通信

**核心架构决策**：

| 决策 | 选择 | 原因 |
|------|------|------|
| Molstar 集成方式 | CDN + iframe 隔离 | 绕过 Next.js 16 Turbopack 对 Molstar 子路径包的循环依赖/打包损坏 |
| Molstar 版本 | 5.5.0 锁定 | 高版本 ESM 循环依赖导致 `Cannot read properties of undefined (reading 'Empty')` |
| 懒加载方案 | IntersectionObserver + 200px 预加载 | 报告页多实例并发初始化会导致 WebGL 上下文竞争 |
| StrictMode 兼容 | iframe 复用 + 延迟清理 | 避免 double-invoke effect 导致的 iframe 重复创建销毁 |
| 兜底策略 | 三层降级（3D → PAE 热图 → 下载链接） | 大蛋白质或低性能环境下保证可用性 |

**核心代码**：

```typescript
// iframe 复用逻辑：避免 StrictMode 下重复创建销毁
let iframe = iframeRef.current
const needsRecreate = !iframe || iframe.dataset.src !== src || !container.contains(iframe)
if (needsRecreate) {
  container.innerHTML = ''
  iframe = document.createElement('iframe')
  iframe.src = src
  iframe.dataset.src = src
  iframeRef.current = iframe
  container.appendChild(iframe)
}

// 复用 iframe 时主动查询状态
if (!needsRecreate && iframe.contentWindow) {
  iframe.contentWindow.postMessage({ type: 'molstar-query-status' }, '*')
}
```

```typescript
// IntersectionObserver 用 state + ref callback（避免 StrictMode ref 失效）
const [wrapperEl, setWrapperEl] = useState<HTMLDivElement | null>(null)

useEffect(() => {
  if (!wrapperEl) return
  const io = new IntersectionObserver(
    (entries) => {
      for (const entry of entries) {
        if (entry.isIntersecting) {
          setIsVisible(true)
          io.disconnect()
          break
        }
      }
    },
    { rootMargin: '200px 0px', threshold: 0.01 }
  )
  io.observe(wrapperEl)
  return () => io.disconnect()
}, [wrapperEl])
```

#### 4. 移动端适配

**适配要点**：
- 3D 画布固定 420px 高度，宽度自适应
- 触摸手势由 Molstar 原生支持
- 加载提示与按钮尺寸符合触摸友好标准

#### 5. 性能优化

**优化策略**：
- IntersectionObserver 懒加载（200px 预加载，进入视口才初始化 Molstar）
- iframe 复用避免重复 WebGL 上下文创建
- 静态兜底避免长时间白屏（60s 超时后切 PAE 热图）
- 分阶段提示（10s / 25s）让用户感知加载进度

---

### 关键决策

| 决策 | 选择 | 原因 |
|------|------|------|
| 3D 渲染库 | Molstar | 生物信息学领域标准，支持 mmCIF/BCIF/PDB |
| 集成方式 | CDN iframe 隔离 | 绕过打包器对 Molstar 子路径包的循环依赖问题 |
| 懒加载 | IntersectionObserver | 多实例并发会导致 WebGL 上下文竞争 |
| 兜底 | 三层降级 | 大蛋白质/低性能环境下保证可用性 |
| StrictMode 兼容 | iframe 复用 + 延迟清理 | 架构层解决方案，非渲染层修补 |

---

### 核心成果

**量化指标**：
- 支持的蛋白质结构格式：mmCIF / BCIF / PDB
- 懒加载预加载距离：200px
- 超时兜底：60s
- 渲染成功率：开发环境 100%（StrictMode 兼容后）

**用户体验**：
- 3D 结构可交互（旋转/缩放/平移）
- 加载进度有分阶段提示
- 渲染失败有明确兜底（PAE 热图 + 下载链接）
- 懒加载不阻塞首屏

**技术亮点**：
- iframe 隔离方案绕过打包器循环依赖
- iframe 复用 + 延迟清理解决 StrictMode 生命周期冲突
- postMessage 状态同步保证 UI 一致性
- 三层降级兜底保证极端场景可用

---

### 踩过的坑

**问题 1：Molstar ESM 循环依赖导致打包失败**

- **症状**：`TypeError: Cannot read properties of undefined (reading 'Empty')` / `ew.Loci is not a function`
- **原因**：Molstar 5.5+ 的子路径包与 Next.js 16 Turbopack 存在循环依赖，打包器损坏内部模块
- **解决方案**：降级到 5.5.0 + CDN 加载 + iframe 隔离，完全绕过打包器
- **详细文档**：[3d-webgl-architecture.md 问题 11](../pitfalls/3d-webgl-architecture.md)

**问题 2：React StrictMode 下 3D 结构渲染成功但画布空白**

- **症状**：渲染成功但画布空白，能操作视图控件但蛋白质不显示，反复修复多次仍失败
- **原因**：StrictMode double-invoke effect 导致 iframe 创建→销毁→重建，WebGL 上下文丢失，重建初始化不完整
- **解决方案**：iframe 复用 + 延迟清理 + postMessage 状态同步
- **详细文档**：[3d-webgl-architecture.md 问题 11](../pitfalls/3d-webgl-architecture.md)

**问题 3：IntersectionObserver 在 StrictMode 下不触发**

- **症状**：组件卡在"待加载"状态，滚动到视口不触发加载
- **原因**：useRef 在 StrictMode remount 后指向已失效的旧 DOM 节点
- **解决方案**：用 state + ref callback 替代 useRef 跟踪目标元素
- **详细文档**：[3d-webgl-architecture.md 问题 11](../pitfalls/3d-webgl-architecture.md)

---

### 经验总结

**做得好的地方**：
1. 用 iframe 隔离方案彻底绕过打包器与 Molstar 的兼容性问题，而非在打包配置上反复修补
2. 识别到"同一个 BUG 改了好多次改不掉"是架构层问题，从 iframe 生命周期管理入手而非渲染逻辑
3. 三层降级兜底（3D → PAE 热图 → 下载链接）保证极端场景可用性

**可以改进的地方**：
1. 初期低估了 StrictMode 对 iframe/WebGL 生命周期的冲击，导致反复修复
2. 懒加载与 StrictMode 的交互应该更早纳入测试
3. 应在架构设计阶段就考虑"同一组件多实例 + StrictMode"的边界场景

**给其他项目的建议**：
1. 当一个 BUG 反复修复失败时，停止在症状层修补，审视架构层是否有生命周期冲突
2. 第三方 3D 库（Molstar/Three.js）与 React 打包器集成有风险时，优先考虑 iframe 隔离
3. StrictMode 下所有涉及 DOM 引用（useRef）和副作用清理的地方都要验证 remount 兼容性
4. iframe 方案必须配套 postMessage 状态同步，否则父组件 UI 会与 iframe 实际状态脱节

---

### 参考资源

- [Molstar 文档](https://molstar.org/)
- [AlphaFold DB](https://alphafold.ebi.ac.uk/)
- [React StrictMode 文档](https://react.dev/reference/react/StrictMode)
- [IntersectionObserver API](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver)

---

### 脱敏声明

- **审查人**：Agent
- **审查日期**：2026-07-31
- **已检查项**：
  - [x] 识别类（部门/公司/用户群/团队规模）— 无，使用泛化描述
  - [x] 技术类（API/域名/内部服务/包名/文档链接）— 仅保留公开 CDN 与公开 API 路径结构
  - [x] 数据类（业务数据/日志/表名字段名）— 无业务数据
  - [x] 个人信息（邮箱/手机/工号/姓名/社交链接）— 无
- **残留风险**：无
- **占位符使用记录**：无（所有内容均为公开技术信息或泛化描述）