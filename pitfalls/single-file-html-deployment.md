# 单文件 HTML 轻量化部署踩坑记录

> 本文档着重总结「将一个复杂重量级 HTML 应用，轻量级转移/迁移为可快速加载、易维护的版本」的经验。
> 来源项目：黑洞引力工具（1.8MB 内联 base64）→ 核反应堆工具（180KB CDN importmap）。
> 适用场景：Three.js / WebGL 单文件 HTML 应用、移动端优先、Vercel 部署。

---

## 问题 1：内联 base64 导致文件体积爆炸（1.8MB）

### 症状
- 单文件 HTML 体积 1.8MB，其中 three.js 内联 base64 占 1.27MB
- 移动端首次加载 5-10 秒，弱网下几乎不可用
- 每次访问都要重新解码 base64，无法利用浏览器缓存

### 根本原因
1. 为了"完全离线、无 CDN 依赖"，把 three.js 整个源码 base64 编码后塞进 importmap
2. base64 比原码大 33%，且主线程同步解码阻塞渲染
3. 浏览器无法缓存 data: URL，每次访问全量重新解码

### 解决方案

**方案 A：内联 base64（重量级，仅限离线场景）**
```html
<script type="importmap">
{ "imports": { "three": "data:text/javascript;base64,XXX..." } }
</script>
```
- 优点：完全离线
- 缺点：体积膨胀 33%、主线程阻塞、无法缓存、修改需 Python 解包/重打包

**方案 B：CDN importmap（轻量级，推荐）**
```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.170.0/examples/jsm/"
  }
}
</script>
<script type="module">
import * as THREE from 'three';
import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
</script>
```
- 优点：业务文件仅 180KB，three.js 走 CDN 且浏览器缓存，二次访问秒开
- 缺点：依赖 CDN 可用性（jsDelivr 稳定性高，基本无问题）

**推荐方案**：方案 B。体积从 1.8MB 降到 180KB（减少 90%），二次访问利用浏览器缓存接近秒开。

### 选型决策树
```
需要完全离线运行？
  ├─ 是 → 内联 base64（接受体积代价）
  └─ 否 → CDN importmap（默认选择）
```

### 验证清单
- [ ] 文件体积 < 500KB（业务代码部分）
- [ ] 移动端 4G 网络下首屏 < 3s
- [ ] 二次访问利用浏览器缓存（DevTools Network 看到 three.js from disk cache）
- [ ] CDN 版本锁定（three@0.170.0，避免主版本漂移）

---

## 问题 2：importmap 后误插代码导致 three.js 加载失败

### 症状
- 页面白屏，loading 卡住不动
- 控制台报 JSON 解析错误或 `Cannot use import statement outside a module`

### 根本原因
1. importmap 必须是独立的 `<script type="importmap">` 块
2. 在 importmap 块内或其后紧跟非 module 的 `<script>` 插入业务代码，破坏了 importmap 的 JSON 结构
3. importmap 必须出现在所有 `<script type="module">` 之前（HTML 顺序）

### 解决方案

**错误写法**（破坏 importmap）：
```html
<script type="importmap">
{ "imports": { "three": "..." } }
</script>
<script>
// ❌ 这里不能插业务代码，会破坏 importmap 解析
console.log("业务代码");
</script>
```

**正确写法**（importmap 独立，业务代码放 module）：
```html
<script type="importmap">
{ "imports": { "three": "..." } }
</script>
<script type="module">
import * as THREE from 'three';
// ✅ 业务代码在这里
</script>
```

### 三条铁律
1. **importmap 必须独立 script 块**，块内不放任何业务代码
2. **importmap 后不能有 modulepreload**（`<link rel="modulepreload">` 与 importmap 冲突会导致加载失败）
3. **importmap 必须在使用前**（HTML 顺序上，importmap 在所有 module script 之前）

### 验证清单
- [ ] importmap 是独立的 `<script type="importmap">` 块
- [ ] 没有 `<link rel="modulepreload">`
- [ ] importmap 出现在第一个 `<script type="module">` 之前
- [ ] 浏览器控制台无 importmap 相关报错

---

## 问题 3：JS 对象字面量语法错误导致整页白屏

### 症状
- 页面白屏，无任何 3D 内容
- 控制台报 `Unexpected token` 语法错误
- 整个 `<script type="module">` 块完全不执行

### 根本原因
1. JS 对象字面量误用 `key=value` 而非 `key:value`（如 `neutrons=true` 应为 `neutrons:true`）
2. ES module 中任何一处语法错误会导致整个模块不执行
3. 单文件 HTML 中业务代码量大，一个字符错误全盘崩溃

### 解决方案

**修改后必做语法检查**：
```bash
node -e "
const fs = require('fs');
const html = fs.readFileSync('public/gravity/reactor.html', 'utf8');
const vm = require('vm');
const scripts = [...html.matchAll(/<script(?![^>]*src)[^>]*>([\s\S]*?)<\/script>/g)].map(m => m[1]);
scripts.forEach((src, i) => {
  // 剥离 import/export 后用 vm 检查
  const stripped = src.replace(/^\s*import[^\n]*\n/gm, '').replace(/^\s*export[^\n]*\n/gm, '');
  try { new vm.Script(stripped); console.log('SCRIPT ' + i + ' OK'); }
  catch(e) { console.log('SCRIPT ' + i + ' ERROR: ' + e.message); }
});
"
```

**注意**：
- importmap 块是 JSON 不是 JS，会报 `Unexpected token ':'`，属正常
- ES module 的 `import/export` 需先剥离才能用 vm 检查

### 验证清单
- [ ] 所有 inline script 语法检查通过（除 importmap）
- [ ] 关键代码片段都存在（grep 检查函数名）
- [ ] 括号平衡为 0（花括号/圆括号/方括号）
- [ ] 浏览器控制台无 SyntaxError

---

## 问题 4：同步生成多张大纹理阻塞主线程

### 症状
- 页面加载卡顿数秒，期间 UI 完全无响应
- 移动端尤其严重，可能被浏览器判定为"页面无响应"

### 根本原因
1. 4 张 2048² 程序纹理（Canvas 生成）在主线程同步执行
2. 每张 2048² 纹理生成耗时 500-1000ms，4 张累计 2-4 秒
3. 主线程被占满，无法渲染 loading 动画

### 解决方案

**按画质分级纹理分辨率**：
```js
const PERF = {
  level: isMobile ? 0 : (isTablet ? 1 : 2),  // 0=低, 1=中, 2=高
  // 纹理分辨率按档位分级
  texSize: PERF.level >= 2 ? 2048 : (PERF.level >= 1 ? 1024 : 512),
};
// 移动端 512²，桌面端 2048²
```

**纹理缓存（避免重复生成）**：
```js
function createContainerStack(){
  if(!createContainerStack.rustTex){
    createContainerStack.rustTex = createRustTexture();  // 首次生成，缓存
  }
  // 多个 mesh 共享同一纹理（纹理共享是安全的，不像材质）
  const mat = new THREE.MeshStandardMaterial({map: createContainerStack.rustTex});
}
```

### 验证清单
- [ ] 移动端纹理分辨率 ≤ 1024²
- [ ] 同一纹理只生成一次（用函数属性缓存）
- [ ] loading 动画在纹理生成期间不卡顿
- [ ] 主线程单次同步任务 < 500ms

---

## 问题 5：共享材质导致动画元素全部闪烁

### 症状
- 修改一个 mesh 的 opacity，其他不相关 mesh 也跟着变
- 多个发光元素同步闪烁，无法独立控制

### 根本原因
1. 多个 mesh 共享同一个材质实例（`new THREE.MeshBasicMaterial(...)` 只 new 一次）
2. 修改 `mesh.material.opacity` 实际修改的是共享材质，所有引用该材质的 mesh 都受影响

### 解决方案

**动画化元素必须用独立材质实例**：
```js
const glowMat = new THREE.MeshBasicMaterial({color:0xff2a1a, transparent:true, opacity:0.9});
const eye = new THREE.Mesh(geo, glowMat);

// ❌ 错误：共享同一材质
const tooth = new THREE.Mesh(geo, glowMat);
eye.material.opacity = 0.5;  // tooth 也跟着变！

// ✅ 正确：独立材质实例
const eyeMat = new THREE.MeshBasicMaterial({color:0xff2a1a, transparent:true, opacity:0.9});
const eye = new THREE.Mesh(geo, eyeMat);
```

**非动画元素可共享**（节省内存）：
```js
const staticMat = new THREE.MeshStandardMaterial({...});
const box1 = new THREE.Mesh(geo, staticMat);
const box2 = new THREE.Mesh(geo, staticMat);  // OK，不修改 opacity
```

**规则**：只要这个材质的 opacity / color / emissiveIntensity 会在动画循环中被修改，就必须独立实例化。

### 验证清单
- [ ] 所有动画化元素的材质都是独立实例
- [ ] 修改单个 mesh 的 opacity 不影响其他 mesh
- [ ] 静态元素可共享材质以节省内存

---

## 问题 6：默认视角与 controls.target 未同步

### 症状
- 初始进入页面，相机位置正确但朝向错误
- 调用 `targetCamera('mobile_intro')` 后视角与初始视角不一致

### 根本原因
1. 初始化时只设置了 `camera.position`，未同步 `controls.target`
2. 相机预设 `CAM_PRESETS` 中的 target 与初始化代码中的 target 不一致

### 解决方案

**初始视角必须与预设同步**：
```js
const CAM_PRESETS = {
  mobile_intro: {pos: new THREE.Vector3(26, 17, 30), target: new THREE.Vector3(-4, 8, -6)},
};

if(isMobile){
  camera.position.set(26, 17, 30);            // 与 mobile_intro.pos 一致
  controls.target.set(-4, 8, -6);             // 与 mobile_intro.target 一致
  camera.fov = 58;                            // 移动端略放宽 FOV
  camera.updateProjectionMatrix();
  controls.update();
}
```

**移动端 FOV 调整**：
```js
if(isMobile){
  camera.fov = 58;  // 略放宽，让更多背景元素入镜
  camera.updateProjectionMatrix();
}
```

### 验证清单
- [ ] 初始 `camera.position` 与 `CAM_PRESETS.xxx.pos` 一致
- [ ] 初始 `controls.target` 与 `CAM_PRESETS.xxx.target` 一致
- [ ] 调用 `controls.update()` 让 target 生效
- [ ] 移动端 FOV 适配（默认 50° → 58°）

---

## 问题 7：Vercel CDN 缓存导致"看不到更新"

### 症状
- 已 push 到 main，Vercel 显示部署成功
- 用户手机端打开仍是旧版本
- 桌面端能看到更新，移动端看不到

### 根本原因
1. Vercel 部署有 CDN 缓存层
2. 手机浏览器缓存比桌面端更激进
3. Service Worker / localStorage 缓存未失效

### 解决方案

**用户侧排查**：
1. 等待 1-2 分钟（CDN 边缘节点同步）
2. 手机端强制刷新 / 清除缓存
3. 加 `?fresh=1` 参数强制跳过 localStorage：`https://www.vectorfuture.xyz/gravity/reactor?fresh=1`

**代码侧兜底**（版本号检测）：
```js
const APP_VERSION = '1.4.0';
const saved = localStorage.getItem('app_version');
if(saved && saved !== APP_VERSION){
  localStorage.clear();
  location.reload();
}
localStorage.setItem('app_version', APP_VERSION);
```

### 验证清单
- [ ] push 后等 1-2 分钟再验证线上
- [ ] 用 `curl` 检查线上 HTML 是否包含新代码（grep 关键函数名）
- [ ] 移动端加 `?fresh=1` 参数测试
- [ ] 版本号检测兜底已部署

---

## 轻量级转移完整迁移路径

将一个重量级 HTML 应用（内联 base64）轻量级转移为 CDN 版本的完整步骤：

### Step 1：评估迁移可行性
```
当前方案是否需要完全离线？
  ├─ 是 → 不迁移，保留内联方案
  └─ 否 → 可迁移到 CDN 方案
```

### Step 2：剥离内联依赖
1. 提取 importmap 中的 base64 模块，记录模块名和版本
2. 改写为 CDN importmap（jsDelivr / unpkg）
3. 版本锁定（`three@0.170.0`，避免主版本漂移）

### Step 3：业务代码迁移
1. 将业务代码从 base64 模块中解出
2. 放入 `<script type="module">` 块
3. 保留原有 `import` 语句

### Step 4：性能分级
1. 新增 `PERF` 对象，按设备分级
2. 纹理分辨率、粒子数量、几何分段数按 `PERF.level` 分级
3. 移动端默认低画质

### Step 5：验证
1. 语法检查（vm.Script）
2. 关键片段检查（grep 函数名）
3. 括号平衡检查
4. HTTP 200 加载验证
5. 移动端真机验证

### Step 6：部署与缓存兜底
1. push 到 main 触发 Vercel 部署
2. `curl` 验证线上代码已更新
3. 版本号检测兜底
4. `?fresh=1` 参数跳过缓存

### 迁移效果对比
| 指标 | 迁移前（内联 base64） | 迁移后（CDN importmap） |
|------|----------------------|------------------------|
| 文件体积 | 1.8MB | 180KB（减少 90%） |
| 首屏加载（4G） | 5-10s | 1-2s |
| 二次访问 | 5-10s（无缓存） | <1s（浏览器缓存） |
| 可维护性 | 需 Python 解包/重打包 | 直接 Edit |
| 修改一个函数 | 重新打包整个文件 | 改一行即可 |

---

## 参考资源

- [importmap 规范](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script/type/importmap)
- [Three.js CDN](https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js)
- [Vercel 部署文档](https://vercel.com/docs)
- [jsDelivr CDN](https://www.jsdelivr.com/)
