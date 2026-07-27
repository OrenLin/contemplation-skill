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

## 问题 6：WebGL 项目缺少画质分级策略

### 症状
- 移动端与桌面端使用相同渲染参数
- 移动端帧率低、发热严重
- 桌面端效果未充分利用高性能硬件

### 根本原因
1. 没有根据设备能力动态调整渲染参数
2. 粒子数、几何分段、后处理开关写死
3. 缺少统一的画质配置对象

### 解决方案
使用全局 PERF 对象管理三档画质：

```js
const isMobile = /iPhone|iPad|iPod|Android/i.test(navigator.userAgent)
  || window.innerWidth <= 768;

const PERF = {
  level: isMobile ? 0 : 1,  // 0=低, 1=中, 2=高
  pixelRatio: isMobile ? Math.min(devicePixelRatio, 1.5) : Math.min(devicePixelRatio, 2),
  neutronCount: isMobile ? 125 : 219,
  floorSegments: isMobile ? 48 : 128,
  enableBloom: !isMobile,
  enableShadows: !isMobile,
  enableAntialias: !isMobile,
};

renderer.setPixelRatio(PERF.pixelRatio);
renderer.shadowMap.enabled = PERF.enableShadows;

if (PERF.enableBloom) {
  composer.addPass(bloomPass);
}
```

### 验证清单
- [ ] 低/中/高三档画质参数定义清晰
- [ ] 移动端默认低画质
- [ ] 用户可手动切换画质

---

## 问题 7：物理模拟未固定步进导致性能波动

### 症状
- 帧率波动时物理模拟不稳定
- 高帧率设备计算过频，低帧率设备模拟跳跃
- 移动端发热严重

### 根本原因
1. 物理更新与渲染帧率绑定
2. 每帧都执行物理计算
3. 无步进累积机制

### 解决方案
固定 30Hz 物理步进，用累积器解耦：

```js
const FIXED_TIMESTEP = 1 / 30;
let physicsAccumulator = 0;

function animate() {
  const delta = clock.getDelta();
  
  // 渲染每帧执行
  composer.render();
  
  // 物理固定 30Hz
  physicsAccumulator += delta;
  while (physicsAccumulator >= FIXED_TIMESTEP) {
    updatePhysics(FIXED_TIMESTEP);
    physicsAccumulator -= FIXED_TIMESTEP;
  }
  
  requestAnimationFrame(animate);
}
```

### 验证清单
- [ ] 物理步进固定为 30Hz 或更低
- [ ] 渲染与物理解耦
- [ ] 帧率波动时模拟稳定

---

## 问题 8：粒子系统未用对象池导致 GC 卡顿

### 症状
- 粒子频繁创建销毁，帧率周期性卡顿
- 内存占用呈锯齿状
- Performance 面板可见频繁 GC

### 根本原因
1. 粒子用 `new` 动态创建
2. 寿命结束后直接释放
3. 没有对象池复用

### 解决方案
预分配粒子池，复用死掉的粒子：

```js
const MAX_PARTICLES = 219;
const positions = new Float32Array(MAX_PARTICLES * 3);
const velocities = new Float32Array(MAX_PARTICLES * 3);
const lives = new Float32Array(MAX_PARTICLES);

// 初始化池
for (let i = 0; i < MAX_PARTICLES; i++) {
  resetParticle(i);
}

function resetParticle(i) {
  positions[i * 3] = sourceX + (Math.random() - 0.5) * 0.5;
  positions[i * 3 + 1] = sourceY;
  positions[i * 3 + 2] = sourceZ + (Math.random() - 0.5) * 0.5;
  velocities[i * 3] = (Math.random() - 0.5) * 2;
  velocities[i * 3 + 1] = Math.random() * 3;
  velocities[i * 3 + 2] = (Math.random() - 0.5) * 2;
  lives[i] = 1.0;
}

function updateParticles(delta) {
  for (let i = 0; i < MAX_PARTICLES; i++) {
    if (lives[i] <= 0) {
      resetParticle(i);  // 复用
      continue;
    }
    positions[i * 3] += velocities[i * 3] * delta;
    positions[i * 3 + 1] += velocities[i * 3 + 1] * delta;
    positions[i * 3 + 2] += velocities[i * 3 + 2] * delta;
    lives[i] -= delta * 0.5;
  }
  geometry.attributes.position.needsUpdate = true;
}
```

### 验证清单
- [ ] 粒子池预分配，运行时无 new
- [ ] 死粒子复用而非销毁
- [ ] Performance 面板无频繁 GC

---

## 问题 9：粒子每帧更新导致移动端卡顿

### 症状
- 桌面端流畅，移动端粒子系统导致掉帧
- 粒子更新占用主线程过多时间

### 根本原因
1. 每帧更新所有粒子位置
2. 移动端 CPU 性能有限
3. 视觉上粒子隔帧更新几乎无差异

### 解决方案
隔帧更新粒子，视觉无损性能翻倍：

```js
let frameCount = 0;

function animate() {
  frameCount++;
  
  // 渲染每帧
  composer.render();
  
  // 粒子隔帧更新
  if (frameCount % 2 === 0) {
    updateParticles(clock.getDelta());
  }
  
  // 物理 30Hz
  // ...
  
  requestAnimationFrame(animate);
}
```

### 验证清单
- [ ] 粒子使用隔帧更新
- [ ] 移动端帧率提升
- [ ] 视觉上无明显差异

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
- [ ] 三档画质分级，移动端默认低
- [ ] 物理模拟固定 30Hz 步进
- [ ] 粒子使用预分配对象池
- [ ] 粒子隔帧更新

---

## 参考资源

- [Web Vitals](https://web.dev/vitals/)
- [React Performance](https://react.dev/learn/render-and-commit)
- [Vite Code Splitting](https://vitejs.dev/guide/build.html#code-splitting)
