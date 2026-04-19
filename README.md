# Hilbert Wormhole

A generative SVG animation that descends perpetually through a tiled Hilbert curve. The viewer can customize the appearance of the wormhole.

---

## What It Is

A Hilbert curve is a space-filling curve — a continuous line that folds back on itself so completely that at sufficient iterations it passes through every point in a square without ever crossing itself. It was described by mathematician David Hilbert in 1891. The folding follows a precise recursive rule: each iteration subdivides every square into four quadrants and connects them in a U-shape, rotating and reflecting each quadrant so the path flows continuously. At low orders it looks like a simple cross or key shape. At high orders the density becomes architectural — a single unbroken line that reads as a maze, a circuit board, a city grid.

The wormhole renders this curve as an SVG polyline, then composites dozens of concentric scaled copies to produce a tunnel that scrolls infinitely toward the viewer. The loop is seamless — there is no reset frame.

Three color stops tween across the full depth range — mouth, mid, back. Rotation, brightness, and stroke weight all interpolate across the same gradient. The result reads differently at every speed and depth setting.

---

## Controls

| Control | What It Does |
|---------|-------------|
| **color 1** | Color at the tunnel mouth (overscaled front layers) |
| **color 2** | Color at the midpoint of the full layer range |
| **color 3** | Color at the deepest visible layers |
| **order** | Hilbert curve complexity — order 2 is a simple cross, order 7 is a dense grid |
| **thick** | Stroke weight at normal scale; adjusts proportionally across all layers |
| **layers** | Total layer count including overshoot |
| **depth** | Scale ratio between adjacent layers — higher values produce a tighter, deeper tunnel |
| **dim** | Minimum brightness at the back of the stack |
| **rotation** | Maximum rotation angle across the depth range |
| **speed** | Animation rate in layers/sec |
| **pause** | Freeze the animation |
| **reset** | Return phase to zero |
| **debug** | Split view: raw Hilbert path (left) + wormhole (right) |

---

## How the Loop Works

Each slot has a continuous position that accounts for the animation phase:

```
pos = slot - phase
d   = Math.pow(depth, pos - EXTRA_FRONT)   // scale: 1.0 when pos = EXTRA_FRONT
```

As `phase` increments from 0 → 1 each cycle, every slot drifts toward the viewer. At `phase = 1.0` it wraps back to `0.0` — seamless, because the slot distribution at wrap is identical to the start shifted by one position.

All tweens — color, rotation, brightness — are computed from `t = pos / (total - 1)`, which is anchored to apparent visual depth rather than slot index. Colors stay locked to depth across the wrap with no jump or reassignment.

---

## The Overshoot Zone

The first 12 slots are always oversized — larger than the canvas, bleeding off the edges. These form the tunnel mouth. They fade in via alpha as they grow to enormous scale, then disappear. This is what makes the descent feel like forward motion through space rather than a stack of layers scrolling past.

---

## Tiling

For non-square viewports, the Hilbert curve tiles along the long axis. Portrait panels use a 90° rotation of the base curve; tiles connect seamlessly at their entry and exit points without mirroring.

---

## Notes for Modification

**Color is anchored to visual depth, not slot index.** `t` is computed from `pos`, which moves with `phase`. Computing `t` from the raw slot integer instead causes a visible color jump every time phase wraps.

**The overshoot loop must include all slots.** `total` covers both the overshoot zone and the visible stack. Shortening the loop to the visible stack only removes the tunnel mouth.

**Edge fades use SVG opacity, not RGB.** `edgeFade` drives `g.setAttribute('opacity', ...)`. Multiplying it into color channels fades to black rather than transparent.

**`init()` rebuilds the path string.** Only triggered when order or panel dimensions change. Do not call it inside the render loop — at high orders the path computation is expensive.

**Speed is quadratic.** The slider maps `[0, 100]` → `t² × 2.0` layers/sec. Low values are intentionally very slow.

---

## Technical

- Single HTML file, no dependencies
- Pure SVG — no canvas, no WebGL
- Deployed to GitHub Pages as `index.html`

---

## License

MIT
