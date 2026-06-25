# 沉思工具优化 SKILL

## 📋 概述

本文档记录了「沉思空间」(Contemplation Space) 工具从 0 到 1 的完整优化过程，涵盖 WebGL 特效适配、移动端布局、性能优化、用户体验等多个维度。适用于需要处理以下场景的开发者：

- WebGL 特效在不同屏幕比例下的适配
- 移动端全屏沉浸式工具开发
- 多主题切换的 UI/UX 设计
- 大型 SPA 应用的代码分割与性能优化

---

## 🎯 核心问题与解决方案

### 1. WebGL 特效过度放大问题

**问题描述：**
- 6 个新增主题（ColorBends、Beams、Hyperspeed、Iridescence、Aurora、EvilEye）在手机上显示时，特效被放大好多倍，只能看到局部
- PC 端正常，移动端完全变形

**根本原因：**
```glsl
// ❌ 错误的 UV 归一化方式
varying vec2 vUv;
void main() {
  // vUv 是 0-1 范围，但在 contain 模式下会被拉伸
  vec2 uv = vUv * 2.0 - 1.0;
  // contain 模式修正宽高比（之前 x/y 写反导致竖屏拉伸）
  float aspect = uResolution.x / uResolution.y;
  if (aspect > 1.0) {
    uv.x *= aspect;  // 横屏
  } else {
    uv.y /= aspect;  // 竖屏
  }
}
```

**解决方案：**
```glsl
// ✅ 正确的 UV 归一化方式（基于 gl_FragCoord）
void main() {
  // 标准归一化：基于像素坐标，y 归一化
  vec2 uv = (gl_FragCoord.xy * 2.0 - uResolution.xy) / uResolution.y;
  uv /= uScale;  // 通过 scale 参数控制整体缩放
}
```

**关键要点：**
1. 使用 `gl_FragCoord` 而非 `vUv`，直接基于像素坐标计算
2. `uResolution` 必须使用设备像素（`gl.canvas.width/height`），而非 CSS 像素
3. 移除 `100vmax` 正方形容器，改为 `absolute inset-0 overflow-hidden`
4. 通过 `uScale` 参数控制整体缩放，避免黑边

**参考代码：**
```javascript
// JavaScript 端
const renderer = new Renderer({ alpha: false, antialias: true });
const gl = renderer.gl;

function resize() {
  const { width, height } = container.getBoundingClientRect();
  renderer.setSize(width, height);
  // 使用设备像素
  program.uniforms.uResolution.value = [
    gl.canvas.width, 
    gl.canvas.height, 
    gl.canvas.width / gl.canvas.height
  ];
}
```

---

### 2. 移动端布局冲突

**问题描述：**
- 主题切换栏与语录卡片边框遮挡
- 主题切换栏与右下角无障碍按钮重合
- 解签工具底部按钮被导航栏遮挡

**解决方案：**

#### 2.1 主题切换栏位置调整
```tsx
// ❌ 之前：右侧纵向排列
<div className="absolute top-1/2 -translate-y-1/2 right-3 z-40">

// ✅ 之后：左侧纵向排列，避开右侧无障碍按钮
<div 
  className="absolute top-1/2 -translate-y-1/2 left-3 z-40 flex flex-col gap-1.5 max-h-[60vh] overflow-y-auto py-2 px-1"
  style={{
    scrollbarWidth: 'none',
    msOverflowStyle: 'none',
    marginTop: 'calc(2rem + env(safe-area-inset-top))',
  }}
>
```

#### 2.2 顶部控制栏居中
```tsx
// ❌ 之前：左上角
<div className="absolute top-0 left-0 z-40">

// ✅ 之后：顶部居中
<div 
  className="absolute top-0 left-1/2 -translate-x-1/2 z-40 flex items-center gap-2 p-3"
  style={{ paddingTop: 'calc(0.75rem + env(safe-area-inset-top))' }}
>
```

#### 2.3 底部按钮避开导航栏
```tsx
// 添加 marginBottom 避开 fixed 底部导航
<div
  className="relative flex items-center justify-between px-4 py-3 flex-shrink-0"
  style={{
    marginBottom: 'calc(64px + env(safe-area-inset-bottom))',
  }}
>
```

**关键要点：**
1. 使用 `env(safe-area-inset-*)` 适配刘海屏
2. 使用 `dvh` 而非 `vh`，避免移动端地址栏显隐导致的布局抖动
3. 固定定位元素需要计算高度，避免重叠

---

### 3. 性能优化：代码分割

**问题描述：**
- 主 JS 文件 1018KB，首屏加载慢
- 所有工具组件打包在一起

**解决方案：**

#### 3.1 动态导入
```tsx
// ❌ 之前：静态导入
import Divination from '../components/tools/Divination';
import Contemplation from '../components/tools/Contemplation';

// ✅ 之后：动态导入
const Divination = lazy(() => import('../components/tools/Divination'));
const Contemplation = lazy(() => import('../components/tools/Contemplation'));

// 使用 Suspense 包裹
<Suspense fallback={<LoadingFallback />}>
  <Contemplation onBack={handleBack} />
</Suspense>
```

#### 3.2 Vite 分包配置
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

**优化效果：**
- 主包从 1018KB 降至 519KB
- 沉思工具单独打包 91KB
- 第三方库单独打包，利用浏览器缓存

---

### 4. 用户体验优化

#### 4.1 新手引导
```tsx
const [guideVisible, setGuideVisible] = useState(false);

useEffect(() => {
  const hasSeenGuide = localStorage.getItem('contemplation-guide-seen');
  if (!hasSeenGuide) {
    setGuideVisible(true);
  }
}, []);

const dismissGuide = useCallback(() => {
  setGuideVisible(false);
  localStorage.setItem('contemplation-guide-seen', 'true');
}, []);
```

**引导内容：**
1. 主题切换：左侧图标点击切换不同主题背景
2. 语录切换：左右滑动或点击箭头切换语录
3. 折叠展开：点击下方按钮折叠语录卡片
4. 互动特效：折叠后滑动屏幕，让画面动起来

#### 4.2 语录卡片交互
```tsx
// 点击卡片切换下一条
<div
  className="relative rounded-3xl p-5 cursor-pointer"
  onClick={() => switchQuote('next')}
>
  {/* 左箭头 */}
  <button
    onClick={(e) => { e.stopPropagation(); switchQuote('prev'); }}
    className="absolute left-0 top-1/2 -translate-y-1/2 -translate-x-1/2 w-10 h-10"
  >
  
  {/* 右箭头 */}
  <button
    onClick={(e) => { e.stopPropagation(); switchQuote('next'); }}
    className="absolute right-0 top-1/2 -translate-y-1/2 translate-x-1/2 w-10 h-10"
  >
</div>
```

**交互方式：**
1. 点击卡片任意位置：下一条
2. 左右箭头按钮：精确控制
3. 触摸滑动：左右滑动 > 50px 切换

---

## 🛠️ 技术栈与工具

### 核心库
- **React 18** + **TypeScript** - UI 框架
- **OGL** - 轻量级 WebGL 库（替代 Three.js）
- **Vite** - 构建工具
- **Tailwind CSS** - 原子化 CSS

### 状态管理
- **Zustand** - 轻量级状态管理

### 部署
- **Vercel** - 自动部署
- **GitHub** - 代码托管

---

## 📦 文件结构

```
src/
├── components/
│   ├── tools/
│   │   ├── Contemplation.tsx          # 沉思工具主组件
│   │   └── contemplation/
│   │       ├── quotes.ts              # 10 个主题，100 条语录
│   │       ├── GalaxyBackground.tsx   # 宇宙主题
│   │       ├── FerrofluidBackground.tsx  # 电磁主题
│   │       ├── LetterGlitchBackground.tsx  # 数字世界
│   │       ├── BalatroBackground.tsx  # 轮回主题
│   │       ├── ColorBendsBackground.tsx  # 光线主题
│   │       ├── BeamsBackground.tsx    # 规则主题
│   │       ├── IridescenceBackground.tsx  # 灵感主题
│   │       ├── AuroraBackground.tsx   # 自然主题
│   │       ├── EvilEyeBackground.tsx  # 混沌主题
│   │       └── HyperspeedBackground.tsx  # 时间主题
│   ├── EvilEye.jsx                    # 邪眼 WebGL 组件
│   ├── ColorBends.jsx                 # 色彩弯曲
│   ├── Beams.jsx                      # 光束
│   ├── Hyperspeed.jsx                 # 超光速
│   ├── Iridescence.jsx                # 虹彩
│   └── Aurora.jsx                     # 极光
└── pages/
    └── Tools.tsx                      # 工具页面（动态导入）
```

---

## 🎨 Shader 开发最佳实践

### 1. UV 归一化
```glsl
// 推荐：基于 gl_FragCoord
vec2 uv = (gl_FragCoord.xy * 2.0 - uResolution.xy) / uResolution.y;

// 不推荐：基于 vUv + contain 模式
vec2 uv = vUv * 2.0 - 1.0;
```

### 2. 分辨率传递
```javascript
// 使用设备像素
program.uniforms.uResolution.value = [
  gl.canvas.width,   // 设备像素宽度
  gl.canvas.height,  // 设备像素高度
  gl.canvas.width / gl.canvas.height  // 宽高比
];
```

### 3. 鼠标/触摸跟踪
```javascript
function onPointerMove(e) {
  const rect = container.getBoundingClientRect();
  // 转换为 -1 到 1 范围
  mouse.tx = ((e.clientX - rect.left) / rect.width) * 2 - 1;
  mouse.ty = -(((e.clientY - rect.top) / rect.height) * 2 - 1);
}
```

### 4. 缩放控制
```glsl
// 通过 uScale 参数控制整体缩放
vec2 uv = (gl_FragCoord.xy * 2.0 - uResolution.xy) / uResolution.y;
uv /= uScale;  // uScale = 0.5 时，内容缩小一半
```

---

## 📱 移动端适配清单

- [ ] 使用 `dvh` 替代 `vh`
- [ ] 使用 `env(safe-area-inset-*)` 适配刘海屏
- [ ] 触摸事件添加 `passive: false`
- [ ] 固定定位元素计算高度，避免重叠
- [ ] WebGL 使用 `gl_FragCoord` 归一化
- [ ] 代码分割，按需加载
- [ ] 新手引导，降低学习成本

---

## 🚀 部署流程

```bash
# 1. 构建
npm run build

# 2. 提交
git add -A
git commit -m "feat: 优化内容"
git push origin main

# 3. Vercel 自动部署
# 访问 https://www.vectorfuture.xyz/contemplation
```

---

## 📚 参考资源

- [OGL 官方文档](https://github.com/oframe/ogl)
- [The Book of Shaders](https://thebookofshaders.com/)
- [WebGL 基础](https://webglfundamentals.org/)
- [Vite 代码分割](https://vitejs.dev/guide/build.html#chunking-strategy)

---

## 🤝 贡献者

- **优化思路**：用户反馈驱动
- **技术实现**：AI 辅助开发
- **测试验证**：多设备真机测试

---

## 📄 许可证

MIT License

---

**最后更新：** 2026-06-25  
**版本：** v1.0.0
