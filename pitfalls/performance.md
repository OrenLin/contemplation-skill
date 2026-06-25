# 性能优化踩坑记录

## 问题 1：主包体积过大（1018KB）

### 症状
- 首屏加载时间超过 5 秒
- 用户等待时间长，跳出率高
- 移动端网络下几乎无法使用

### 根本原因
- 所有工具组件静态导入，打包在一起
- 第三方库（React、Zustand、OGL）全部在主包中
- 没有代码分割

### 解决方案

#### 1. 动态导入工具组件
```tsx
// ❌ 错误：静态导入
import Divination from '../components/tools/Divination';
import Contemplation from '../components/tools/Contemplation';
import PhilosophyInsight from '../components/tools/PhilosophyInsight';

// ✅ 正确：动态导入
import { lazy, Suspense } from 'react';

const Divination = lazy(() => import('../components/tools/Divination'));
const Contemplation = lazy(() => import('../components/tools/Contemplation'));
const PhilosophyInsight = lazy(() => import('../components/tools/PhilosophyInsight'));

// 使用 Suspense 包裹
<Suspense fallback={<LoadingFallback />}>
  <Divination onBack={handleBack} />
</Suspense>
```

#### 2. Vite 分包配置
```typescript
// vite.config.ts
export default defineConfig({
  build: {
    sourcemap: 'hidden',
    rollupOptions: {
      output: {
        manualChunks: {
          // 核心框架
          'vendor-react': ['react', 'react-dom'],
          // 状态管理
          'vendor-state': ['zustand'],
          // WebGL 渲染
          'vendor-webgl': ['ogl'],
        },
      },
    },
  },
});
```

### 优化效果
- **主包体积**：1018KB → 519KB（减少 49%）
- **沉思工具**：单独打包 91KB
- **抽签工具**：单独打包 114KB
- **第三方库**：单独打包，利用浏览器缓存

---

## 问题 2：图片加载慢

### 症状
- 页面加载时图片区域空白
- 大图片加载时间过长

### 解决方案

#### 1. 图片懒加载
```tsx
<img
  src="image.jpg"
  loading="lazy"
  alt="描述"
/>
```

#### 2. 使用 WebP 格式
```tsx
<picture>
  <source srcset="image.webp" type="image/webp" />
  <source srcset="image.jpg" type="image/jpeg" />
  <img src="image.jpg" alt="描述" />
</picture>
```

#### 3. 响应式图片
```tsx
<img
  src="image-800.jpg"
  srcSet="image-400.jpg 400w, image-800.jpg 800w, image-1200.jpg 1200w"
  sizes="(max-width: 600px) 400px, (max-width: 1000px) 800px, 1200px"
  alt="描述"
/>
```

---

## 问题 3：长列表渲染卡顿

### 症状
- 滚动长列表时卡顿
- 内存占用过高
- 移动端尤其明显

### 解决方案

#### 1. 虚拟滚动
使用 `react-window` 或 `react-virtualized`：

```tsx
import { FixedSizeList } from 'react-window';

<FixedSizeList
  height={600}
  itemCount={items.length}
  itemSize={80}
  width="100%"
>
  {({ index, style }) => (
    <div style={style}>
      {items[index].name}
    </div>
  )}
</FixedSizeList>
```

#### 2. 分页加载
```tsx
const [page, setPage] = useState(1);
const [items, setItems] = useState([]);

const loadMore = () => {
  const newItems = await fetchItems(page + 1);
  setItems([...items, ...newItems]);
  setPage(page + 1);
};

// 滚动到底部时加载
const handleScroll = (e) => {
  if (e.target.scrollHeight - e.target.scrollTop === e.target.clientHeight) {
    loadMore();
  }
};
```

---

## 问题 4：滚动事件性能问题

### 症状
- 滚动时页面卡顿
- 事件处理函数执行过于频繁

### 解决方案

#### 1. 防抖（Debounce）
```tsx
import { debounce } from 'lodash';

const handleScroll = debounce(() => {
  // 处理滚动逻辑
}, 200);

window.addEventListener('scroll', handleScroll);
```

#### 2. 节流（Throttle）
```tsx
import { throttle } from 'lodash';

const handleScroll = throttle(() => {
  // 处理滚动逻辑
}, 100);

window.addEventListener('scroll', handleScroll);
```

#### 3. 使用 Intersection Observer
```tsx
useEffect(() => {
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          // 元素进入视口
        }
      });
    },
    { threshold: 0.5 }
  );

  const element = document.getElementById('target');
  if (element) {
    observer.observe(element);
  }

  return () => observer.disconnect();
}, []);
```

---

## 问题 5：WebGL 性能问题

### 症状
- WebGL 效果卡顿
- 设备发热严重
- 电池消耗快

### 解决方案

#### 1. 降低渲染分辨率
```javascript
// 使用设备像素比，但不超过 2
const dpr = Math.min(window.devicePixelRatio, 2);
renderer.setSize(width, height);
renderer.setPixelRatio(dpr);
```

#### 2. 减少动画复杂度
- 降低粒子数量
- 简化 Shader 计算
- 减少纹理采样次数

#### 3. 暂停不可见动画
```tsx
const [isVisible, setIsVisible] = useState(false);

useEffect(() => {
  const observer = new IntersectionObserver(
    ([entry]) => setIsVisible(entry.isIntersecting)
  );
  
  observer.observe(containerRef.current);
  return () => observer.disconnect();
}, []);

// 只在可见时运行动画
useEffect(() => {
  if (!isVisible) return;
  
  const animate = () => {
    // 渲染逻辑
    requestAnimationFrame(animate);
  };
  
  const frameId = requestAnimationFrame(animate);
  return () => cancelAnimationFrame(frameId);
}, [isVisible]);
```

---

## 性能测试工具

### 1. Lighthouse
```bash
# 在 Chrome DevTools 中运行
# 或使用命令行
lighthouse https://example.com --output html
```

### 2. Web Vitals
```tsx
import { getCLS, getFID, getLCP } from 'web-vitals';

getCLS(console.log);  // Cumulative Layout Shift
getFID(console.log);  // First Input Delay
getLCP(console.log);  // Largest Contentful Paint
```

### 3. React DevTools Profiler
- 记录组件渲染时间
- 发现不必要的重渲染
- 优化组件结构

---

## 性能优化清单

- [ ] 主包体积 < 500KB
- [ ] 代码分割，按需加载
- [ ] 图片懒加载
- [ ] 使用 WebP 格式
- [ ] 长列表使用虚拟滚动
- [ ] 滚动事件防抖/节流
- [ ] WebGL 降低分辨率（DPR <= 2）
- [ ] 暂停不可见动画
- [ ] 使用 Lighthouse 测试
- [ ] 监控 Core Web Vitals

---

## 参考资源

- [Web Vitals](https://web.dev/vitals/)
- [React Performance](https://react.dev/learn/render-and-commit)
- [Vite Code Splitting](https://vitejs.dev/guide/build.html#code-splitting)
