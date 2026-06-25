# 抽签工具优化案例

## 项目背景

抽签工具是一个沉浸式的占卜应用，用户通过摇签、抽签获得人生指引。在移动端测试时发现底部操作按钮被固定导航栏遮挡，影响用户体验。

## 问题发现

### 症状
- "再抽一签"按钮被底部固定导航栏遮挡
- 用户无法点击按钮，必须滚动才能看到
- 移动端地址栏显隐导致布局抖动

### 根本原因
1. 底部操作区没有考虑固定导航栏的高度
2. 使用 `vh` 单位，移动端地址栏显隐导致视口高度变化
3. 没有使用安全区域适配

## 解决方案

### 方案 1：添加底部间距

**实现代码**：
```tsx
<div
  className="relative flex items-center justify-between px-4 py-3 flex-shrink-0"
  style={{
    marginBottom: 'calc(64px + env(safe-area-inset-bottom))',
  }}
>
  <button>查看签文</button>
  <button>再抽一签</button>
</div>
```

**关键点**：
- `64px` 是底部导航栏高度
- `env(safe-area-inset-bottom)` 是底部安全区域（iPhone X 及以上）
- 使用 `calc()` 动态计算总间距

### 方案 2：使用 dvh 替代 vh

**实现代码**：
```tsx
// ❌ 错误：vh 单位
<div style={{ height: '100vh' }}>

// ✅ 正确：dvh 单位
<div style={{ height: '100dvh' }}>
```

**原因**：
- `vh` 在移动端地址栏显隐时会变化
- `dvh`（dynamic viewport height）会自动调整
- 避免布局抖动

### 方案 3：修复滚动回弹问题

**问题**：
- 向下滚动解签面板时，面板会弹回顶部
- 用户体验极差

**根因**：
- `transform: scaleY()` 动画 + `both` 填充模式重置滚动位置

**解决方案**：
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

## 技术细节

### 安全区域适配

需要在 HTML 中添加：
```html
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
```

### 浏览器支持

- `dvh`：iOS Safari 15.4+、Android Chrome 108+
- `env(safe-area-inset-*)`：iOS 11.2+、Android Chrome 69+

### 降级方案

```css
.container {
  height: 100vh; /* 降级 */
  height: 100dvh; /* 现代浏览器 */
}
```

## 验证清单

- [ ] 底部按钮避开固定导航栏
- [ ] 使用 `dvh` 替代 `vh`
- [ ] 安全区域适配完整
- [ ] 滚动容器不使用 `transform` 动画
- [ ] 在 iOS Safari、Android Chrome、微信浏览器中测试

## 经验总结

1. **移动端适配要考虑全面**：导航栏、安全区域、地址栏
2. **避免在滚动容器上使用 transform 动画**：会导致滚动位置重置
3. **使用现代 CSS 单位**：`dvh` 比 `vh` 更适合移动端
4. **提供降级方案**：确保在不支持的浏览器中也能正常工作

## 参考资源

- [MDN: env()](https://developer.mozilla.org/en-US/docs/Web/CSS/env)
- [CSS Tricks: Safe Area Insets](https://css-tricks.com/the-notch-and-css/)
- [Can I Use: dvh](https://caniuse.com/viewport-unit-variants)
- [移动端布局踩坑](../pitfalls/mobile-layout.md)
