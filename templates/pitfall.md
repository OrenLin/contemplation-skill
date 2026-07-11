# 踩坑记录模板

## 问题 N：[问题名称]

### 症状

[描述用户看到的现象，要具体、可观察]

- 现象 1
- 现象 2
- 现象 3

### 根本原因

[解释为什么会发生这个问题，要深入技术细节]

1. 原因 1
2. 原因 2
3. 原因 3

### 解决方案

[提供具体的代码示例和修复步骤]

**方案 A：[方案名称]**

```typescript
// 代码示例
```

**方案 B：[方案名称]**

```typescript
// 代码示例
```

**推荐方案**：[说明为什么推荐某个方案]

### 验证清单

- [ ] [验证项 1]
- [ ] [验证项 2]
- [ ] [验证项 3]
- [ ] [验证项 4]

### 相关资源

- [资源 1](链接)
- [资源 2](链接)

---

## 使用示例

### 问题 1：AnimatePresence 导致 WebGL 上下文丢失

### 症状

- Tab 切换后 3D 场景消失
- 控制台报错：WebGL context lost
- 切换回原 Tab 后场景无法恢复

### 根本原因

1. `AnimatePresence` 在切换 Tab 时会卸载组件
2. Three.js Canvas 被销毁，WebGL 上下文丢失
3. 重新挂载时无法恢复上下文

### 解决方案

**方案 A：Canvas 始终挂载，用 opacity 控制可见性**

```tsx
<motion.div
  animate={{
    opacity: activeTab === 'simulator' ? 1 : 0,
    pointerEvents: activeTab === 'simulator' ? 'auto' : 'none'
  }}
  transition={{ duration: 0.3 }}
>
  <Canvas>
    <Scene />
  </Canvas>
</motion.div>
```

**方案 B：使用 CSS display 属性**

```tsx
<div style={{ display: activeTab === 'simulator' ? 'block' : 'none' }}>
  <Canvas>
    <Scene />
  </Canvas>
</div>
```

**推荐方案**：方案 A，因为支持动画过渡，用户体验更好。

### 验证清单

- [ ] Tab 切换后场景保持可见
- [ ] 控制台无 WebGL 错误
- [ ] 切换回原 Tab 后场景正常
- [ ] 动画过渡流畅

### 相关资源

- [React Three Fiber 文档](https://docs.pmnd.rs/react-three-fiber)
- [Framer Motion 文档](https://www.framer.com/motion/)
