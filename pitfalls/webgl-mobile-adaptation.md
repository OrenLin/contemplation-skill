# WebGL 移动端适配踩坑记录

## 问题 1：Shader 效果在移动端过度放大

### 症状
- PC 端显示正常
- 移动端显示时，效果被放大 5-10 倍，只能看到局部

### 根本原因
使用 `vUv`（0-1 范围）配合 contain 模式修正宽高比时，x/y 写反导致竖屏拉伸：

```glsl
// ❌ 错误的 UV 归一化方式
varying vec2 vUv;
void main() {
  vec2 uv = vUv * 2.0 - 1.0;
  // contain 模式修正宽高比（之前 x/y 写反导致竖屏拉伸）
  float aspect = uResolution.x / uResolution.y;
  if (aspect > 1.0) {
    uv.x *= aspect;  // 横屏
  } else {
    uv.y /= aspect;  // 竖屏 - 错误！
  }
}
```

### 解决方案
使用 `gl_FragCoord` 标准归一化，基于像素坐标：

```glsl
// ✅ 正确的 UV 归一化方式
void main() {
  // 标准归一化：基于像素坐标，y 归一化
  vec2 uv = (gl_FragCoord.xy * 2.0 - uResolution.xy) / uResolution.y;
  uv /= uScale;  // 通过 scale 参数控制整体缩放
}
```

### 为什么这样有效
- 除以 `uResolution.y` 使得垂直范围始终是 -1..1（经过 *2-1 居中后）
- 水平范围自动根据宽高比缩放
- 不需要 contain/cover 分支判断

---

## 问题 2：uResolution 使用 CSS 像素

### 症状
- 即使修复了 UV 归一化，Shader 仍然渲染比例错误

### 根本原因
传递 CSS 像素尺寸而非设备像素尺寸：

```javascript
// ❌ 错误：CSS 像素
program.uniforms.uResolution.value = [container.clientWidth, container.clientHeight, ...];
```

### 解决方案
始终使用 `gl.canvas.width/height`（设备像素）：

```javascript
// ✅ 正确：设备像素
function resize() {
  const { width, height } = container.getBoundingClientRect();
  renderer.setSize(width, height);
  program.uniforms.uResolution.value = [
    gl.canvas.width,   // 设备像素宽度
    gl.canvas.height,  // 设备像素高度
    gl.canvas.width / gl.canvas.height  // 宽高比
  ];
}
```

---

## 问题 3：100vmax 正方形容器导致缩放

### 症状
- 即使 Shader 正确，效果仍然被放大
- 因为容器本身被强制为正方形

### 根本原因
包装器使用 `100vmax` 创建居中正方形，然后 `overflow-hidden` 裁剪。在竖屏手机上裁剪了大部分效果：

```tsx
// ❌ 错误：强制正方形容器
<div style={{ width: '100vmax', height: '100vmax', position: 'absolute', inset: 0 }} />
```

### 解决方案
移除正方形容器，让 Shader 填充实际屏幕宽高比：

```tsx
// ✅ 正确：填充屏幕，Shader 处理宽高比
<div className="absolute inset-0 overflow-hidden">
  <EvilEye />
</div>
```

---

## 问题 4：缺少缩放控制参数

### 症状
- 效果正确显示，但无法调整整体大小

### 解决方案
添加 `uScale` uniform 控制整体缩放：

```glsl
uniform float uScale;

void main() {
  vec2 uv = (gl_FragCoord.xy * 2.0 - uResolution.xy) / uResolution.y;
  uv /= uScale;  // 控制整体缩放
}
```

```javascript
// JavaScript 端
program.uniforms.uScale = { value: 0.8 };  // 默认值
```

### 各主题缩放参考值（竖屏移动端）
- EvilEye：scale 0.5（眼睛占据约 50% 屏幕高度）
- Beams：光束位置 ±0.6（原来是 ±2，超出屏幕）
- ColorBends/Hyperspeed/Iridescence/Aurora：scale 0.6-0.8

---

## 完整适配流程

1. 打开组件的 fragment shader
2. 将 `vUv` 替换为 `gl_FragCoord` 归一化（见问题 1）
3. 确保 `uResolution` uniform 声明为 `vec3`（width, height, aspect）
4. 更新 JS 端传递设备像素（见问题 2）
5. 添加 `uScale` uniform（如果不存在）；将其暴露为 prop
6. 移除任何 `100vmax` 正方形包装器（见问题 3）
7. 调整 `uScale` 参数 — 从 0.5-0.8 开始，调整使效果填充屏幕而不裁剪关键视觉元素

---

## 验证清单

- [ ] Shader 使用 `gl_FragCoord` 归一化，而非 `vUv` + contain
- [ ] `uResolution` 使用 `gl.canvas.width/height`（设备像素）
- [ ] 没有 `100vmax` 正方形包装器围绕 Shader 容器
- [ ] `uScale` 按主题调整，使效果填充屏幕而不裁剪
- [ ] 在任何测试的宽高比下都没有黑边（竖屏、横屏、正方形）

---

## 参考资源

- [The Book of Shaders](https://thebookofshaders.com/) - Shader 学习资源
- [OGL 文档](https://github.com/oframe/ogl) - 轻量级 WebGL 库
- [reactbits.dev](https://www.reactbits.dev) - WebGL 效果组件库
