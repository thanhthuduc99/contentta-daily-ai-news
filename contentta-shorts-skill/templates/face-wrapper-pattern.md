# Face Wrapper Pattern

Face video landscape (1920×1080) crop vào vertical canvas (1080×1920).

## CSS

```css
#face-wrapper {
  position: absolute;
  top: 0; left: 0;
  width: 1920px; height: 1080px;
  transform-origin: 0 0;
  transform: translate(0px, 1136px) scale(0.5625);
  z-index: 0;
}
```

## Math

### BOTTOM mode (face docked lower half)
- `scale(0.5625)` = 1080 / 1920
- Face renders: 1080 × 607.5 px
- `translate(0, 1136)` → face top at y=1136
- Face area: y=1136 → y=1743.5
- Black gap below: y=1743.5 → y=1920 (176.5px)

### FULLSCREEN mode (face fills canvas)
- `scale(1.7778)` = 1920 / 1080
- Face renders: 3413 × 1920 px (cover crop)
- `translate(-1166.5, 0)` → horizontally centered
- `x = -(1920 * 1.7778 - 1080) / 2 = -1166.5`

### HIDDEN mode
- `opacity: 0` — face mất, scene panel phủ full canvas
- Fade: 0.3s ease power2.in

## GSAP master timeline

```js
const BOTTOM = { x: 0, y: 1136, scale: 0.5625, opacity: 1 };

mainTl.set("#face-wrapper", BOTTOM, 0);

// Face hidden during full-canvas scenes
mainTl.to("#face-wrapper", { opacity: 0, duration: 0.30, ease: "power2.in" }, sceneStart - 0.15);
mainTl.to("#face-wrapper", { opacity: 1, duration: 0.30, ease: "power2.out" }, sceneEnd - 0.30);

// Seam line follows face visibility
mainTl.to("#seam", { opacity: 0, duration: 0.30, ease: "power2.in" }, sceneStart - 0.15);
mainTl.to("#seam", { opacity: 1, duration: 0.30, ease: "power2.out" }, sceneEnd - 0.30);

// Ken Burns slow zoom
mainTl.to("#face-video", { scale: 1.04, duration: TOTAL, ease: "none" }, 0);
```
