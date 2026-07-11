# 交互设计踩坑记录

## 问题 1：触摸反馈缺失

### 症状
- 用户点击按钮没有视觉反馈
- 不确定是否点击成功
- 用户体验差

### 解决方案

#### 1. 使用 :active 伪类
```css
button:active {
  transform: scale(0.95);
  opacity: 0.8;
}
```

#### 2. Tailwind CSS 实现
```tsx
<button className="active:scale-95 active:opacity-80 transition-all">
  点击我
</button>
```

#### 3. 添加触觉反馈（移动端）
```tsx
const handleClick = () => {
  // 触觉反馈（如果支持）
  if ('vibrate' in navigator) {
    navigator.vibrate(50);
  }
  
  // 执行点击逻辑
  doSomething();
};
```

---

## 问题 2：手势冲突

### 症状
- 左右滑动和上下滑动冲突
- 卡片滑动和页面滚动冲突
- 用户体验混乱

### 解决方案

#### 1. 明确手势优先级
```tsx
const handleTouchStart = (e) => {
  touchStartRef.current = {
    x: e.touches[0].clientX,
    y: e.touches[0].clientY,
  };
};

const handleTouchEnd = (e) => {
  if (!touchStartRef.current) return;
  
  const deltaX = e.changedTouches[0].clientX - touchStartRef.current.x;
  const deltaY = e.changedTouches[0].clientY - touchStartRef.current.y;
  
  // 水平滑动 > 50px 且大于垂直滑动
  if (Math.abs(deltaX) > 50 && Math.abs(deltaX) > Math.abs(deltaY)) {
    if (deltaX < 0) {
      switchQuote('next');
    } else {
      switchQuote('prev');
    }
  }
  
  touchStartRef.current = null;
};
```

#### 2. 使用 touchAction 控制
```tsx
{/* 只允许垂直滚动 */}
<div style={{ touchAction: 'pan-y' }}>
  {/* 内容 */}
</div>

{/* 禁止所有默认触摸行为 */}
<div style={{ touchAction: 'none' }}>
  {/* 自定义手势 */}
</div>
```

---

## 问题 3：新手引导设计不当

### 症状
- 引导过于复杂，用户不耐烦
- 引导过于简单，用户不理解
- 引导无法关闭，用户反感

### 解决方案

#### 1. 简洁的引导内容
```tsx
const guideItems = [
  { icon: '🎨', title: '主题切换', desc: '左侧图标点击切换不同主题背景' },
  { icon: '←→', title: '语录切换', desc: '左右滑动或点击箭头切换语录' },
  { icon: '⬇️', title: '折叠展开', desc: '点击下方按钮折叠语录卡片' },
  { icon: '👆', title: '互动特效', desc: '折叠后滑动屏幕，让画面动起来' },
];
```

#### 2. 多种关闭方式
```tsx
{/* 点击背景关闭 */}
<div
  className="fixed inset-0 z-50 bg-black/80"
  onClick={dismissGuide}
>
  {/* 引导内容 */}
  <div onClick={(e) => e.stopPropagation()}>
    {/* 阻止事件冒泡 */}
    
    {/* 关闭按钮 */}
    <button onClick={dismissGuide}>×</button>
    
    {/* 确认按钮 */}
    <button onClick={dismissGuide}>我知道了</button>
  </div>
</div>
```

#### 3. 只显示一次
```tsx
const [guideVisible, setGuideVisible] = useState(false);

useEffect(() => {
  const hasSeenGuide = localStorage.getItem('contemplation-guide-seen');
  if (!hasSeenGuide) {
    setGuideVisible(true);
  }
}, []);

const dismissGuide = () => {
  setGuideVisible(false);
  localStorage.setItem('contemplation-guide-seen', 'true');
};
```

---

## 问题 4：多交互方式冲突

### 症状
- 点击卡片切换下一条
- 左右箭头也切换
- 滑动也切换
- 三种方式冲突

### 解决方案

#### 1. 统一交互逻辑
```tsx
const switchQuote = useCallback((direction: 'next' | 'prev') => {
  if (isTransitioning) return;
  
  audioManager.play('click');
  setQuoteVisible(false);
  
  setTimeout(() => {
    setCurrentQuoteIndex((prev) => {
      if (direction === 'next') {
        return (prev + 1) % currentTheme.quotes.length;
      } else {
        return (prev - 1 + currentTheme.quotes.length) % currentTheme.quotes.length;
      }
    });
    setQuoteVisible(true);
  }, 500);
}, [isTransitioning, currentTheme.quotes.length]);
```

#### 2. 阻止事件冒泡
```tsx
{/* 卡片点击 */}
<div onClick={() => switchQuote('next')}>
  {/* 左箭头 - 阻止冒泡 */}
  <button
    onClick={(e) => {
      e.stopPropagation();
      switchQuote('prev');
    }}
  >
    ←
  </button>
  
  {/* 右箭头 - 阻止冒泡 */}
  <button
    onClick={(e) => {
      e.stopPropagation();
      switchQuote('next');
    }}
  >
    →
  </button>
</div>
```

---

## 问题 5：动画过于复杂

### 症状
- 动画时间过长，用户等待
- 动画效果花哨，分散注意力
- 动画性能差，卡顿

### 解决方案

#### 1. 控制动画时长
```tsx
{/* ❌ 错误：动画时间过长 */}
<div className="transition-all duration-1000">

{/* ✅ 正确：动画时间适中 */}
<div className="transition-all duration-300">
```

**推荐时长**：
- 微交互（按钮悬停）：150-200ms
- 状态切换（展开/折叠）：300-400ms
- 页面过渡：400-600ms

#### 2. 使用 CSS 动画而非 JS
```css
/* ✅ 性能更好 */
@keyframes fadeIn {
  from { opacity: 0; }
  to { opacity: 1; }
}

.fade-in {
  animation: fadeIn 0.3s ease-out;
}
```

#### 3. 避免布局抖动
```css
/* ❌ 错误：改变布局属性 */
@keyframes slideIn {
  from { height: 0; }
  to { height: 100px; }
}

/* ✅ 正确：使用 transform */
@keyframes slideIn {
  from { transform: translateY(-100%); }
  to { transform: translateY(0); }
}
```

---

## 问题 6：状态反馈不明确

### 症状
- 加载状态不明确
- 成功/失败状态不清晰
- 用户不知道发生了什么

### 解决方案

#### 1. 加载状态
```tsx
{isLoading ? (
  <div className="flex items-center justify-center">
    <div className="animate-spin rounded-full h-8 w-8 border-t-2 border-b-2 border-blue-500" />
  </div>
) : (
  <div>内容</div>
)}
```

#### 2. 成功/失败状态
```tsx
const [status, setStatus] = useState<'idle' | 'loading' | 'success' | 'error'>('idle');

{status === 'success' && (
  <div className="text-green-500 flex items-center gap-2">
    <span>✓</span>
    <span>操作成功</span>
  </div>
)}

{status === 'error' && (
  <div className="text-red-500 flex items-center gap-2">
    <span>✗</span>
    <span>操作失败，请重试</span>
  </div>
)}
```

---

## 问题 7：无障碍设计缺失

### 症状
- 无法使用键盘导航
- 屏幕阅读器无法识别
- 颜色对比度不足

### 解决方案

#### 1. 键盘导航
```tsx
<button
  onKeyDown={(e) => {
    if (e.key === 'Enter' || e.key === ' ') {
      e.preventDefault();
      handleClick();
    }
  }}
>
  点击我
</button>
```

#### 2. ARIA 标签
```tsx
<button
  aria-label="关闭对话框"
  aria-expanded={isOpen}
  aria-controls="dialog-content"
>
  ×
</button>

<div
  id="dialog-content"
  role="dialog"
  aria-labelledby="dialog-title"
>
  <h2 id="dialog-title">对话框标题</h2>
  {/* 内容 */}
</div>
```

#### 3. 颜色对比度
```css
/* ❌ 错误：对比度不足 */
.text-gray-300 on bg-white

/* ✅ 正确：对比度符合 WCAG */
.text-gray-700 on bg-white
```

---

## 交互设计清单

- [ ] 所有可点击元素有触摸反馈
- [ ] 手势冲突已解决（使用 touchAction）
- [ ] 新手引导简洁、可关闭、只显示一次
- [ ] 多交互方式统一逻辑、不冲突
- [ ] 动画时长适中（150-600ms）
- [ ] 状态反馈明确（加载、成功、失败）
- [ ] 支持键盘导航
- [ ] ARIA 标签完整
- [ ] 颜色对比度符合 WCAG 标准

---

## 参考资源

- [MDN: Touch Events](https://developer.mozilla.org/en-US/docs/Web/API/Touch_events)
- [WAI-ARIA 实践指南](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/WAI-ARIA_basics)
- [WCAG 2.1 标准](https://www.w3.org/WAI/WCAG21/quickref/)
