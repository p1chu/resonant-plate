# Resonant Plate

A browser-based tool for generating Chladni-pattern and gradient particle images —
built to produce background imagery for the Aligned AI site, both as exported
stills and as a live embedded canvas.

Everything lives in one self-contained file: **`resonant-plate.html`**. No build
step, no dependencies, no network calls (fonts aside). Open it directly in a
browser.

## Running it

- **Tuning / authoring:** open `resonant-plate.html` in a browser. The control
  panel is top-right; click **Controls** if it's collapsed.
- **Embedded background:** load the same file with `?embed` appended
  (`resonant-plate.html?embed`). That hides the HUD and control panel, leaving
  only the live canvas — same file, no separate build.
- **Published copy:** https://claude.ai/code/artifact/9ea0cf81-3ea1-453e-86a6-7badf42eb830
  (requires claude.ai login). This is a *publish of* the local file; the local
  file is the source of truth.

## The two Interpret modes

The **Shape** section's `Interpret` toggle switches between two genuinely
different mechanisms — not variations on one.

### Outline

Upload an SVG; particles trace it. The SVG is rasterized twice via the browser's
own renderer (once forced to solid fill, once to stroke-only) into masks that
replace the Chladni equation as the field driving particle motion: `0` on the
path (a node particles converge on), a small residual value inside the fill,
`1` outside (pushes particles back in).

A chamfer **distance transform** over the mask gives every off-shape point its
distance to the nearest on-shape pixel, so particles outside the shape move
*directly* toward it at a speed calibrated to arrive within `Settle time`.
Without this, convergence relied on undirected random walk and had no time bound.

`Coverage` exempts a percentage of particles from the shape entirely — they
wander the ambient field as residual scatter. Outline-mode only concept.

### Density

A parametric vertical gradient. **No image involved** — it's pure math, so it
activates the moment you select it and needs no upload.

- Below the `Stop` line: a solid `fillRect()` in the particle color.
- Above it: a painted alpha gradient (`createLinearGradient`) *plus* scattered
  particles, both derived from the same `densityAtY()` curve.

Why both a painted wash and particles: a point-cloud alone is statistically
noisy near any boundary, so particles alone always showed a visible seam against
the solid region no matter how many you used. The wash guarantees continuity;
the particles supply the sand-like grain, most visible near the top where the
wash is nearly transparent.

`Coverage` does not apply here — every particle is density-driven. (Leaving the
stray mechanism active in this mode was a real bug: stray particles followed the
Chladni field and produced a bright band exactly where the spec wanted nothing.)

## Motion

- **Style: Steering** — particles seek toward nodal lines (adapted from Patt
  Vira's p5.js Chladni sketch, ported to dependency-free canvas).
- **Style: Grain** — each particle random-walks with step size proportional to
  field strength (adapted from PettaBoy's Cymatics-Simulator-Chladni). Grainier,
  more textural.
- **Vibration envelope** — all motion is scaled by a global smooth `0 → 1 → 0`
  curve over `Duration`, mimicking briefly playing a tone through the plate and
  cutting it. When it completes the multiplier locks at exactly `0` and *every*
  particle is motionless — this deliberately overrides Grain's floor jitter and
  stray wandering, both of which are otherwise "always alive".
- Triggered on load, on click anywhere, on `Shuffle now`, and on auto-cycle
  ticks. `Auto-cycle` is **off** by default — patterns settle and stay.

## Canvas sizing

`Aspect` picks a ratio; the stage is always a proportional "contain" fit inside
the viewport — never stretched on either axis.

- Presets (1:1, 4:5, 3:4, 9:16, 5:4, 4:3, 16:9)
- `Match SVG` — matches an uploaded shape's own proportions (auto-selected on upload)
- `Fill viewport` — edge to edge
- `Dimensions` — exact pixel output (e.g. 4000×3000), decoupled from display
  size; previewed scaled down. **Export honors this**, so it's how you get
  high-resolution stills.

SVG masks re-rasterize on aspect change so an uploaded shape never distorts.

## Export

`Export PNG` composites onto a fresh canvas, re-applying the blur (which
normally lives on a separate CSS layer and never touches canvas pixels) scaled
to the actual pixel resolution — so the file matches what's on screen.

It tries the Claude artifact `downloads` capability first (needed because that
sandbox blocks script-triggered downloads), then falls back to a normal download
link when opened as a real file or embedded on a site.

## Notes / open threads

- The whole tool was developed without live browser verification — the Chrome
  automation extension was disconnected throughout. Changes were validated by
  Node syntax checks, HTML tag-balance checks, element-ID cross-referencing, and
  numeric unit tests of the math (distance field, rejection sampling, envelope
  curve, gradient stops). **Visual behavior has been confirmed only by the
  user's screenshots.**
- Density mode is the active area of iteration; the fade between solid and
  scattered regions has gone through several rounds.
- Performance at high `Particles` values (50k–150k) has not been profiled.
- `scripts in /private/tmp/.../scratchpad/` (`extract_script.py`, `check_tags.py`)
  were the verification helpers — they live in a temp dir and may be gone.

## Lineage

Mathematical basis from `hilbertcube/Chladni-Patterns-Generator` (Python, for
reference plots). Particle behavior adapted from Patt Vira's p5.js tutorial
sketch and `PettaBoy/Cymatics-Simulator-Chladni`. All JS here is original,
dependency-free canvas — p5.js was deliberately dropped.
