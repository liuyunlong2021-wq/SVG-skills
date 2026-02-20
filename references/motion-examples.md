# Motion Examples & Best Practices

Provide exact reference points for high-quality SVG/CSS animations.

## 1. Squash & Stretch (挤压与拉伸 / 弹性与重力)

To give objects weight and elasticity, use `transform-origin` with precise `transform` rules containing both translations and scales.

```css
.bouncing-ball {
  /* Critical: Origin must be at the bottom center to stretch upwards / squash into ground */
  /* Safari/Chrome Fix: Always include transform-box for precise SVG scaling */
  transform-box: fill-box;
  transform-origin: 50% 100%;
  animation: bounce 1.2s cubic-bezier(0.28, 0.84, 0.42, 1) infinite;
}

@keyframes bounce {
  0%, 100% {
    /* Airborne: Stretched vertically, thinned horizontally */
    transform: translateY(-150px) scale(0.9, 1.1);
  }
  45%, 55% {
    /* Impact with ground: Normal shape just before/after squishing */
    transform: translateY(0) scale(1, 1);
  }
  50% {
    /* Max Impact: Squished fat and wide on the ground */
    transform: translateY(0) scale(1.3, 0.7);
  }
}
```

**⚠️ Nesting Warning**: Never apply CSS `transform` animations directly to an SVG element that already has a static `transform="translate(X,Y)"` coordinate. Doing so will overwrite the position and send the object flying across the screen.
Instead, wrap the object in a static `<g>` group for positioning, and animate the inner element:

```xml
<!-- GOOD: Outer group sets position, inner group handles animation -->
<g transform="translate(100, 200)">
  <g class="bouncing-ball">...</g>
</g>
```

## 2. Overlapping Action & Delay (跟随动画与时间差)

Characters running or objects blowing in the wind should have trailing parts (hair, tails, clothing) that lag behind the main motion. Do this using `animation-delay` on child elements sharing the same keyframe, or slightly offset keyframe percentages.

```xml
<style>
  .tail-segment {
    transform-box: fill-box;
    transform-origin: top center;
    animation: sway 1s ease-in-out infinite alternate;
  }
  
  @keyframes sway {
    0%   { transform: rotate(-15deg); }
    100% { transform: rotate(15deg); }
  }
</style>

<!-- Base body element drives the root motion -->
<g class="dog-body" transform="translate(400, 400)">
  <!-- Inner segments use animation. Notice translations are separated from rotations where possible -->
  <path class="tail-segment" style="animation-delay: 0s;" d="..." />
  <g transform="translate(0, 20)">
    <path class="tail-segment" style="animation-delay: -0.2s;" d="..." />
  </g>
  <g transform="translate(0, 40)">
    <path class="tail-segment" style="animation-delay: -0.4s;" d="..." />
  </g>
</g>
```

## 3. Parallax Scrolling & Seamless Loops (视差移动背景)

Background elements should move at different speeds (slower for distant clouds, faster for foreground rocks) to create a perfect seamless wide-loop.

```css
.cloud-layer-bg {
  /* Slow, far away */
  animation: slideLeft 30s linear infinite;
}

.ground-layer-fg {
  /* Fast, close */
  animation: slideLeft 6s linear infinite;
}

@keyframes slideLeft {
  /* For seamless SVG loops, clone the grouped elements so they span 2x the width */
  /* If elements take up 1200px (width), shift them rigidly by -1200px */
  from { transform: translateX(0); }
  to   { transform: translateX(-1200px); }
  /* Ensure easing is always 'linear' to avoid jerky restarts */
}
```
