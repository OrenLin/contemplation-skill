# 移动端布局踩坑记录

## 问题 1：底部按钮被固定导航栏遮挡

### 症状
- "再抽一签"按钮被底部固定导航栏遮挡
- 用户无法点击按钮

### 根本原因
底部操作区没有考虑固定导航栏的高度和安全区域

### 解决方案
添加 `marginBottom` 避开固定导航栏：

```tsx
<div
  className="relative flex items-center justify-between px-4 py-3 flex-shrink-0"
  style={{
    marginBottom: 'calc(64px + env(safe-area-inset-bottom))',
  }}
>
```

**关键点**：
- `64px` 是导航栏高度
- `env(safe-area-inset-bottom)` 是底部安全区域（iPhone X 及以上）
- 使用 `calc()` 动态计算

---

## 问题 2：主题切换栏与无障碍按钮冲突

### 症状
- 右侧主题切换栏与右下角无障碍按钮（`fixed bottom-24 right-4`）重叠
- 主题切换栏与语录卡片边框遮挡

### 解决方案
**移动位置**：
- 主题切换栏从右侧移到左侧
- 顶部控制栏（返回+语言）从左上角移到顶部居中

```tsx
{/* 主题切换：左侧纵向滚动条 */}
<div
  className="absolute top-1/2 -translate-y-1/2 left-3 z-40 flex flex-col gap-1.5 max-h-[60vh] overflow-y-auto py-2 px-1"
  style={{
    scrollbarWidth: 'none',
    msOverflowStyle: 'none',
    marginTop: 'calc(2rem + env(safe-area-inset-top))',
  }}
>
```

```tsx
{/* 顶部控制栏：顶部居中 */}
<div
  className="absolute top-0 left-1/2 -translate-x-1/2 z-40 flex items-center gap-2 p-3"
  style={{ paddingTop: 'calc(0.75rem + env(safe-area-inset-top))' }}
>
```

---

## 问题 3：CSS 动画重置滚动位置

### 症状
- 向下滚动解签面板时，面板会弹回顶部
- 用户体验极差

### 根本原因
`transform: scaleY()` 动画 + `both` 填充模式会重置内部 `overflow-y-auto` 的滚动位置

### 解决方案
使用纯 `opacity` 动画，移除 `transform`：

```css
/* ❌ 错误：transform 动画 */
.divination-scroll-unfold {
  animation: unfold 0.6s ease-out both;
}

@keyframes unfold {
  from {
    opacity: 0;
    transform: scaleY(0);
  }
  to {
    opacity: 1;
    transform: scaleY(1);
  }
}

/* ✅ 正确：纯 opacity 动画 */
.divination-scroll-unfold {
  animation: fadeIn 0.6s ease-out both;
}

@keyframes fadeIn {
  from {
    opacity: 0;
  }
  to {
    opacity: 1;
  }
}
```

---

## 问题 4：vh 单位在移动端抖动

### 症状
- 移动端地址栏显隐导致 `vh` 值变化
- 布局抖动，用户体验差

### 解决方案
使用 `dvh`（dynamic viewport height）替代 `vh`：

```css
/* ❌ 错误：vh 单位 */
.container {
  height: 100vh;
}

/* ✅ 正确：dvh 单位 */
.container {
  height: 100dvh;
}
```

**浏览器支持**：
- iOS Safari 15.4+
- Android Chrome 108+
- 其他现代浏览器均支持

**降级方案**：
```css
.container {
  height: 100vh; /* 降级 */
  height: 100dvh; /* 现代浏览器 */
}
```

---

## 问题 5：安全区域适配不完整

### 症状
- 刘海屏设备上，顶部内容被刘海遮挡
- 底部内容被 Home Indicator 遮挡

### 解决方案
使用 `env(safe-area-inset-*)` 适配：

```tsx
{/* 顶部安全区域 */}
<div
  className="absolute top-0 left-0 right-0 z-40"
  style={{ paddingTop: 'calc(0.75rem + env(safe-area-inset-top))' }}
>

{/* 底部安全区域 */}
<div
  className="fixed bottom-0 left-0 right-0 z-40"
  style={{ paddingBottom: 'env(safe-area-inset-bottom)' }}
>
```

**需要在 HTML 中添加**：
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
```

---

## 问题 6：触摸区域过小

### 症状
- 按钮太小，用户难以点击
- 误触率高

### 解决方案
确保触摸区域最小 44x44px：

```tsx
{/* ❌ 错误：触摸区域过小 */}
<button className="w-6 h-6">
  <Icon />
</button>

{/* ✅ 正确：触摸区域足够 */}
<button className="w-11 h-11 flex items-center justify-center">
  <Icon />
</button>
```

**或者使用 padding 扩大触摸区域**：
```tsx
<button className="p-3">
  <Icon className="w-6 h-6" />
</button>
```

---

## 问题 7：横向溢出

### 症状
- 页面可以横向滚动
- 用户体验差

### 解决方案
1. **全局禁止横向滚动**：
```css
body {
  overflow-x: hidden;
}
```

2. **检查元素宽度**：
```tsx
{/* ❌ 错误：固定宽度可能超出 */}
<div className="w-[400px]">

{/* ✅ 正确：使用百分比或 max-width */}
<div className="w-full max-w-[400px]">
```

3. **使用 flex/grid 布局**：
```tsx
<div className="flex flex-wrap gap-2">
  {items.map(item => (
    <div className="flex-1 min-w-[120px]">
      {item}
    </div>
  ))}
</div>
```

---

## 问题 8：PC端面板折叠按钮不明显（核反应堆项目）

### 症状
- 用户不知道控制面板和仪表盘可以折叠
- 折叠后找不到展开按钮

### 根本原因
缺少明确的折叠/展开交互提示

### 解决方案
**折叠按钮设计**：
- 固定在面板顶部，宽度 100%
- 使用醒目的颜色和图标（◀/▶）
- 添加悬停和点击动画

```tsx
{/* PC端折叠按钮 */}
<motion.button
  onClick={() => setPanelVisible(false)}
  className="w-full mb-2 py-1.5 rounded-lg bg-black/80 backdrop-blur-md border border-cyan-400/40 text-cyan-400 text-[11px] font-bold tracking-wider flex items-center justify-center gap-2 font-body hover:bg-cyan-500/10 hover:border-cyan-400/70 transition-all"
  whileHover={{ scale: 1.02 }}
  whileTap={{ scale: 0.98 }}
>
  <span>◀ 点击折叠控制台</span>
</motion.button>

{/* 展开按钮（折叠后显示） */}
<motion.button
  onClick={() => setPanelVisible(true)}
  className="fixed top-20 left-4 z-20 py-3 px-3 rounded-lg bg-black/80 backdrop-blur-md border border-cyan-400/60 text-cyan-400 text-[11px] font-bold tracking-wider flex items-center gap-2 font-body hover:bg-cyan-500/15 hover:border-cyan-400 transition-all shadow-[0_0_15px_rgba(0,212,255,0.2)]"
  whileHover={{ scale: 1.05 }}
  whileTap={{ scale: 0.95 }}
>
  <span className="animate-pulse">▶</span>
  <span>展开控制台</span>
</motion.button>
```

**移动端折叠提示**：
- 弹出面板顶部显示"▼ 点击折叠面板"
- 面板未打开时底部显示"▲ 点击上方按钮操作"
- 使用动画提示（上下浮动）

```tsx
{/* 移动端折叠提示按钮 */}
<motion.button
  onClick={() => setMobilePanel(null)}
  className="w-full mb-2 py-2 rounded-lg bg-black/80 backdrop-blur-md border border-cyan-400/40 text-cyan-400 text-xs font-bold tracking-wider flex items-center justify-center gap-2 font-body"
  whileTap={{ scale: 0.97 }}
>
  <span>▼ 点击折叠面板</span>
</motion.button>

{/* 未打开时的提示 */}
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

**关键点**：
- 折叠按钮必须居中且宽度 100%
- 使用脉冲动画吸引注意力
- 展开按钮添加发光阴影效果

---

## 问题 9：z-index 层级地狱（核反应堆项目）

### 症状
- InfoPanel 覆盖控制面板和仪表盘
- 面板之间相互遮挡
- 修改 10+ 次仍无法解决

### 根本原因
- 使用 `absolute` 定位导致受滚动影响
- 多个面板 z-index 没有明确层级关系
- 3D Canvas 和 UI 面板层级混乱

### 解决方案
**明确层级体系**：
```
z-0: 3D Canvas（底层）
z-10: 标题、状态指示器（装饰层）
z-20: InfoPanel（信息层）
z-30: 控制面板/仪表盘（交互层）
z-50: 顶部导航栏（最高层）
```

**使用 `fixed` 替代 `absolute`**：
```tsx
{/* ❌ 错误：absolute 受滚动影响 */}
<div className="absolute top-20 left-4 z-20">
  <ControlPanel />
</div>

{/* ✅ 正确：fixed 固定在视口 */}
<div className="fixed top-20 left-4 z-30">
  <ControlPanel />
</div>
```

**关键原则**：
- 3D Canvas 始终在最底层（z-0）
- 交互面板（控制板/仪表盘）z-index 高于信息面板
- 使用 `fixed` 定位避免滚动影响
- 顶部导航栏 z-50 最高

---

## 验证清单

- [ ] 底部按钮避开固定导航栏（使用 `marginBottom`）
- [ ] 主题切换器在左侧，顶部控制栏居中
- [ ] 滚动容器不使用 `transform` 动画
- [ ] 使用 `dvh` 替代 `vh`
- [ ] 安全区域适配完整（顶部、底部、左右）
- [ ] 触摸区域最小 44x44px
- [ ] 无横向溢出
- [ ] PC端折叠按钮明显且易用
- [ ] 移动端折叠提示按钮居中显示
- [ ] 折叠/展开动画流畅
- [ ] z-index 层级关系明确（Canvas < InfoPanel < ControlPanel < Navbar）
- [ ] 在 iOS Safari、Android Chrome、微信浏览器中测试

---

## 参考资源

- [MDN: env()](https://developer.mozilla.org/en-US/docs/Web/CSS/env)
- [CSS Tricks: Safe Area Insets](https://css-tricks.com/the-notch-and-css/)
- [Can I Use: dvh](https://caniuse.com/viewport-unit-variants)
- [MDN: position: fixed](https://developer.mozilla.org/en-US/docs/Web/CSS/position#fixed)
