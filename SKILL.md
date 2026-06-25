---
name: contemplation-immersive-tool
description: Build immersive contemplation/meditation tools with multi-theme WebGL animated backgrounds, mobile-first layout, and performance optimization. Use this when creating meditation apps, contemplation spaces, relaxation tools with animated shader backgrounds, multi-theme switching interfaces, or when adapting WebGL shader effects (from reactbits.dev or similar) for mobile screens. Covers shader UV normalization fixes, mobile safe-area layout, code-splitting, and onboarding UX.
license: MIT
---

# Contemplation Immersive Tool

Build production-grade immersive tools with animated WebGL backgrounds that work flawlessly across desktop and mobile. This skill encodes the complete workflow — from design brainstorming through shader adaptation, mobile layout, performance tuning, and onboarding UX — distilled from a real production deployment.

## When To Use

- Creating meditation, contemplation, or relaxation tools with animated backgrounds
- Adapting WebGL shader effects (e.g. from [reactbits.dev](https://www.reactbits.dev)) for mobile screens
- Building multi-theme switching interfaces with WebGL backgrounds
- Fixing shader "over-zoom" issues where effects appear magnified on mobile
- Implementing mobile-first immersive layouts with safe-area handling
- Optimizing large React SPA bundle sizes via code-splitting

## Critical Pitfalls (Avoid These)

These are the exact traps hit during development. Each has a verified fix below.

### Pitfall 1: Shader Over-Zoom on Mobile

**Symptom:** Effect looks correct on desktop but appears magnified 5-10x on mobile, showing only a tiny fragment of the intended visual.

**Root Cause:** Using `vUv` (0-1 range) with a "contain" aspect-ratio correction. The contain math had x/y swapped, causing vertical stretch on portrait screens.

```glsl
// ❌ BROKEN: vUv + contain mode
varying vec2 vUv;
void main() {
  vec2 uv = vUv * 2.0 - 1.0;
  float aspect = uResolution.x / uResolution.y;
  if (aspect > 1.0) {
    uv.x *= aspect;   // landscape
  } else {
    uv.y /= aspect;   // portrait — WRONG, stretches vertically
  }
}
```

**Fix:** Use `gl_FragCoord`-based normalization with y as the reference axis. This auto-adapts to any aspect ratio without conditional contain logic.

```glsl
// ✅ CORRECT: gl_FragCoord normalization
void main() {
  vec2 uv = (gl_FragCoord.xy * 2.0 - uResolution.xy) / uResolution.y;
  uv /= uScale;  // control overall zoom via uScale uniform
}
```

**Why it works:** Dividing by `uResolution.y` normalizes so that vertical extent is always -1..1 (after the *2-1 centering). Horizontal extent scales automatically with aspect ratio. No contain/cover branching needed.

### Pitfall 2: uResolution Using CSS Pixels

**Symptom:** Shader renders at wrong scale or aspect even after the UV fix.

**Root Cause:** Passing CSS pixel dimensions instead of device pixel dimensions.

```javascript
// ❌ BROKEN: CSS pixels
program.uniforms.uResolution.value = [container.clientWidth, container.clientHeight, ...];
```

**Fix:** Always use `gl.canvas.width/height` (device pixels) after `renderer.setSize()`.

```javascript
// ✅ CORRECT: device pixels
function resize() {
  const { width, height } = container.getBoundingClientRect();
  renderer.setSize(width, height);
  program.uniforms.uResolution.value = [
    gl.canvas.width,
    gl.canvas.height,
    gl.canvas.width / gl.canvas.height,
  ];
}
```

### Pitfall 3: 100vmax Square Container Causing Zoom

**Symptom:** Even with correct shader, effects still appear zoomed because the container itself is a forced square.

**Root Cause:** Wrapper used `100vmax` to create a centered square, then `overflow-hidden` cropped it. On portrait phones this cropped most of the effect.

```tsx
// ❌ BROKEN: forced square container
<div style={{ width: '100vmax', height: '100vmax', position: 'absolute', inset: 0 }} />
```

**Fix:** Remove the square container. Let the shader fill the actual screen aspect ratio.

```tsx
// ✅ CORRECT: fill screen, shader handles aspect
<div className="absolute inset-0 overflow-hidden">
  <EvilEye />
</div>
```

### Pitfall 4: Bottom Button Obscured By Fixed Nav

**Symptom:** "Draw again" button hidden behind the fixed bottom navigation bar.

**Fix:** Add `marginBottom` equal to nav height plus safe-area inset.

```tsx
<div style={{ marginBottom: 'calc(64px + env(safe-area-inset-bottom))' }}>
```

### Pitfall 5: Theme Switcher Overlapping Accessibility Button

**Symptom:** Right-side theme switcher collides with the fixed accessibility button (`fixed bottom-24 right-4`) and the quote card border.

**Fix:** Move theme switcher to the left side; move top controls (back/language) to top-center.

### Pitfall 6: CSS Animation Resetting Scroll Position

**Symptom:** Scrolling down in a panel causes it to snap back to top.

**Root Cause:** A `transform: scaleY()` animation with `both` fill mode resets the internal `overflow-y-auto` scroll position on each frame.

**Fix:** Use opacity-only animations; never `transform` on scroll containers.

### Pitfall 7: vh Unit Jitter On Mobile

**Symptom:** Layout jitters when mobile browser address bar shows/hides.

**Fix:** Use `dvh` (dynamic viewport height) instead of `vh`.

## Workflow

This skill follows the brainstorming → spec → plan → implement cycle used in production.

### Phase 1: Brainstorming (use brainstorming skill)

Before any code, run the brainstorming skill to:
1. Explore intent — what themes, what moods, what interactions
2. Propose 2-3 approaches with tradeoffs
3. Present design sections, get approval per section
4. Write spec to `docs/superpowers/specs/YYYY-MM-DD-<topic>-design.md`

**Key questions to resolve:**
- How many themes? (production used 10)
- How many quotes per theme? (production used 10)
- What interaction modes? (tap card, arrows, swipe, collapse/expand)
- Onboarding needed? (yes — first-visit guide)

### Phase 2: Spec & Plan (use writing-plans skill)

Convert the approved design into an implementation plan with:
- File structure (background components per theme)
- Data model (themes + quotes with zh/en)
- State management (current theme, current quote, expanded/collapsed)
- Deployment target (Vercel SPA with URL routing)

### Phase 3: Source Shader Components

Pull animated background components from [reactbits.dev](https://www.reactbits.dev) — specifically the backgrounds section. Each background ships as a self-contained React + OGL component with configurable props (colors, speed, scale, etc.).

**Components used in production:**
- Galaxy, Ferrofluid, LetterGlitch, Balatro (original 4 themes)
- ColorBends, Beams, Hyperspeed, Iridescence, Aurora, EvilEye (added 6 themes)

### Phase 4: Shader Adaptation (THE critical step)

For EVERY background component, apply the UV normalization fix. This is non-negotiable for mobile.

1. Open the component's fragment shader
2. Replace `vUv`-based UV with `gl_FragCoord`-based UV (see Pitfall 1)
3. Ensure `uResolution` uniform is declared as `vec3` (width, height, aspect)
4. Update the JS side to pass device pixels (see Pitfall 2)
5. Add a `uScale` uniform if not present; expose it as a prop
6. Remove any `100vmax` square wrapper (see Pitfall 3)
7. Tune `uScale` per theme — start at 0.5-0.8, adjust so the effect fills the screen without cropping key visual elements

**Per-theme scale tuning reference (portrait mobile):**
- EvilEye: scale 0.5 (eye occupies ~50% screen height)
- Beams: beam positions ±0.6 (was ±2, off-screen)
- ColorBends/Hyperspeed/Iridescence/Aurora: scale 0.6-0.8

### Phase 5: Mobile Layout

Apply these layout rules to the immersive container:

```tsx
<div className="fixed inset-0 overflow-hidden bg-black" style={{ height: '100dvh' }}>
  {/* Background layer — pointerEvents toggle based on card state */}
  <div
    className="absolute inset-0"
    style={{ pointerEvents: quoteExpanded ? 'none' : 'auto', touchAction: 'none' }}
  >
    {renderBackground()}
  </div>

  {/* Top controls — CENTERED to avoid left-side theme switcher */}
  <div
    className="absolute top-0 left-1/2 -translate-x-1/2 z-40"
    style={{ paddingTop: 'calc(0.75rem + env(safe-area-inset-top))' }}
  />

  {/* Theme switcher — LEFT side, vertical, scrollable */}
  <div
    className="absolute top-1/2 -translate-y-1/2 left-3 z-40 flex flex-col gap-1.5 max-h-[60vh] overflow-y-auto"
    style={{ marginTop: 'calc(2rem + env(safe-area-inset-top))' }}
  />

  {/* Quote card — centered, smaller on mobile */}
  <div className="absolute inset-0 z-20 flex items-center justify-center">
    <div className="w-[80%] max-w-[300px]" />
  </div>
</div>
```

### Phase 6: Performance — Code Splitting

Large SPAs bundle everything into one JS file. Split it.

**Step 1: Dynamic import tool components**

```tsx
import { lazy, Suspense } from 'react';

const Contemplation = lazy(() => import('../components/tools/Contemplation'));
const Divination = lazy(() => import('../components/tools/Divination'));

function LoadingFallback() {
  return (
    <div className="fixed inset-0 flex items-center justify-center bg-black">
      <div className="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-white" />
    </div>
  );
}

// Usage
<Suspense fallback={<LoadingFallback />}>
  <Contemplation onBack={handleBack} />
</Suspense>
```

**Step 2: Vite manualChunks for vendor libs**

```typescript
// vite.config.ts
build: {
  sourcemap: 'hidden',
  rollupOptions: {
    output: {
      manualChunks: {
        'vendor-react': ['react', 'react-dom'],
        'vendor-state': ['zustand'],
        'vendor-webgl': ['ogl'],
      },
    },
  },
},
```

**Measured impact:** Main bundle 1018KB → 519KB. Contemplation chunk isolated at ~91KB. Vendor chunks cached across navigations.

### Phase 7: URL Routing For Direct Access

To allow direct links like `/contemplation`, sync a URL hash/query param with the active tool state, and add a `vercel.json` rewrite so SPA routes don't 404.

```json
// vercel.json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```

### Phase 8: Onboarding UX

First-visit guide, shown once, dismissible three ways (button, backdrop tap, escape):

```tsx
const [guideVisible, setGuideVisible] = useState(false);

useEffect(() => {
  if (!localStorage.getItem('contemplation-guide-seen')) {
    setGuideVisible(true);
  }
}, []);

const dismissGuide = useCallback(() => {
  setGuideVisible(false);
  localStorage.setItem('contemplation-guide-seen', 'true');
}, []);
```

Guide covers: theme switching, quote switching (tap/arrows/swipe), collapse/expand, interactive effects after collapse.

### Phase 9: Quote Card Interaction

Three interaction modes, all wired to the same `switchQuote` function:

1. **Tap card body** → next quote
2. **Tap left/right arrow buttons** → prev/next (with `e.stopPropagation()` so they don't trigger the card tap)
3. **Touch swipe** (>50px horizontal) → prev/next

Arrow buttons: positioned at card edges, half-translated outside, 40px circles, gradient backgrounds, opacity 40% resting / 80% hover.

## Tech Stack

- **React 18 + TypeScript** — UI framework
- **OGL** — lightweight WebGL library (preferred over Three.js for shader-only effects)
- **Vite** — build tool with manualChunks
- **Tailwind CSS** — atomic styling
- **Zustand** — state management
- **Vercel** — deployment with SPA rewrites

## Source References

- **Animated backgrounds:** [reactbits.dev](https://www.reactbits.dev) — pull components, then apply shader adaptation (Phase 4)
- **Shader theory:** [The Book of Shaders](https://thebookofshaders.com/)
- **OGL docs:** [github.com/oframe/ogl](https://github.com/oframe/ogl)

## File Structure (Production Reference)

```
src/
├── components/
│   ├── tools/
│   │   ├── Contemplation.tsx           # main immersive component
│   │   └── contemplation/
│   │       ├── quotes.ts               # 10 themes × 10 quotes, zh/en
│   │       ├── GalaxyBackground.tsx
│   │       ├── FerrofluidBackground.tsx
│   │       ├── LetterGlitchBackground.tsx
│   │       ├── BalatroBackground.tsx
│   │       ├── ColorBendsBackground.tsx
│   │       ├── BeamsBackground.tsx
│   │       ├── HyperspeedBackground.tsx
│   │       ├── IridescenceBackground.tsx
│   │       ├── AuroraBackground.tsx
│   │       ├── EvilEyeBackground.tsx
│   │       └── EvilEye.jsx              # raw OGL component
│   └── EvilEye.jsx                     # adapted shader component
└── pages/
    └── Tools.tsx                       # lazy-loaded entry
```

## Verification Checklist

Before shipping, verify each item:

- [ ] Shader uses `gl_FragCoord` normalization, not `vUv` + contain
- [ ] `uResolution` uses `gl.canvas.width/height` (device pixels)
- [ ] No `100vmax` square wrappers around shader containers
- [ ] `uScale` tuned per theme so effect fills screen without cropping
- [ ] Root container uses `100dvh`, not `100vh`
- [ ] Fixed elements use `env(safe-area-inset-*)`
- [ ] Bottom buttons have `marginBottom` clearing the fixed nav
- [ ] Theme switcher on left, top controls centered (avoid right-side accessibility button)
- [ ] Scroll containers never have `transform`-based CSS animations
- [ ] Tool components lazy-loaded with `Suspense` fallback
- [ ] `manualChunks` configured for vendor libs
- [ ] `vercel.json` SPA rewrite present
- [ ] First-visit onboarding guide implemented, dismissible, persisted in localStorage
- [ ] Quote card supports tap + arrows + swipe
- [ ] No black bars on any tested aspect ratio (portrait, landscape, square)
