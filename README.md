# Progress Bar Engagement Experiment

[Demo](https://superdave2u.github.io/progress-bar-experiment/)

## Purpose

This prototype compares **progress bar fill options** to explore potential differences in user engagement. A progress bar is a visual element that implies progress; the experiment tests how the *shape* of the fill changes perceived progress when actual time is constant.

The original experiment compared two fill rates:

- **Linear fill rate**: progress increases uniformly over time.
- **Eased fill rate** (`easeOutSqrt`): progress is faster at the beginning and slows near completion (square root easing).

The intent is to evaluate whether easing influences perceived progress or satisfaction.

## Platform

The original prototype ran two hardcoded bars once and required a page reload to run again. It has since been rebuilt into a small platform for running fill experiments:

- **Fill options registry (`EASINGS`)**: pure functions from raw time progress (0..1) to displayed progress (0..1). Built in:
  - `linear` — uniform (the baseline)
  - `easeOutSqrt` — the original eased bar (`sqrt(p)`)
  - `easeOutCubic` — even more aggressive fast-start curve
  - `easeInOutCubic` — slow-fast-slow
  - `easeInQuad` — slow start, fast finish (the "deadline panic" profile)
- **Per-bar fill rates**: every bar carries its own duration in seconds, so a fast linear bar can race a slow eased one.
- **Multiple bars side by side, one trigger**: add bar rows with different options and rates, then trigger all of them with the same **Run all** button. The canvas resizes to fit the bars.
- **Mobile friendly**: the canvas measures the surrounding card and recomputes bar geometry on every resize (including rotation and URL-bar changes), so bars fit any viewport without horizontal scrolling; touch targets get larger tap areas.
- **Parallel or sequential modes**: parallel renders bars simultaneously (cleanest A/B comparison); sequential runs them one after another (the original behavior).
- **Restart without reloading**: the same button reruns the experiment; completion is detected and the animation loop stops itself.

### Label semantics

Two label styles are available, and they measure different things on purpose:

- **Percent** reflects the *displayed* fill — perceived progress.
- **Seconds remaining** derives from *real elapsed time* — the honest clock.

Watching an eased bar with a seconds label shows the real time draining evenly while the fill races ahead; watching it with a percent label shows the fill slowing near the end. Both are part of the experiment.

## Technologies Used

- [p5.js](https://p5js.org/) for canvas rendering and animation.
- No build step: single `index.html`, deployed to GitHub Pages.

## Architecture

```
EASINGS registry (pure fill options)
        │
        ▼
class ProgressBar  (strategy-based: easing injected, layout per bar)
        │
        ▼
Experiment state   (barConfigs, mode, runningIndex)
        │
        ▼
DOM controls       (per-bar rows, mode + label selects, one Run button)
        │
        ▼
p5 setup()/draw()  (render loop, parallel/sequential scheduling)
```

### `class ProgressBar`

The base class encapsulates timing, layout, and rendering. The fill option is injected as a key into `EASINGS` instead of being subclass-specific, and each bar owns its `x`/`y` layout so multiple bars can render independently.

```javascript
new ProgressBar({ easingKey, duration, showPercentage, color, x, y })
```

- `easingKey`: name of a fill option in `EASINGS`.
- `duration`: total fill duration in seconds (the fill rate).
- `showPercentage`: `true` shows perceived percent; `false` shows real seconds remaining.
- `x`, `y`: bar layout, so bars can stack.

Key methods:

- `start()` — records the start timestamp using `millis()`.
- `calculateProgress()` — raw time progress, 0..1.
- `easedProgress()` — applies the injected fill option to raw progress.
- `display()` — renders background, fill, and label at the bar's own coordinates.

## Control Flow

`setup()` builds the canvas and the initial two-bar configuration. `draw()` renders every bar in parallel mode, or the active bar in sequential mode, and stops the loop (`noLoop()`) when the experiment completes. The Run button rebuilds bars from the DOM configuration, resizes the canvas, and restarts the loop.

## Usage

1. Open [the demo](https://superdave2u.github.io/progress-bar-experiment/).
2. For each bar row, choose a fill option and a duration.
3. Pick a mode (parallel or sequential) and a label style.
4. Press **Run all** — every bar is triggered by that same button.
5. Press **Restart** to run again with different settings.

## Adding a new fill option

One line in the `EASINGS` registry:

```javascript
const EASINGS = Object.freeze({
    // ...existing options...
    easeOutQuart: (p) => 1 - pow(1 - p, 4),
});
```

It appears in every bar's dropdown immediately.

## Experiment ideas

- Linear vs eased at identical durations (the original A/B).
- Fast-short vs slow-long bars: does perceived speed track the fill option or the rate?
- `easeInQuad` (slow start) against `easeOutCubic` (fast start) at the same duration.
- Sequential mode as a "before and after" demonstration for design reviews.
