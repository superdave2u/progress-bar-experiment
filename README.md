# Progress Bar Engagement Prototype

## Purpose
This prototype compares **two types of progress bar fill rates** to explore potential differences in user engagement:
- **Linear fill rate**: progress increases uniformly over time.
- **Eased fill rate**: progress is faster at the beginning and slows near completion (applies a square root easing function).

The intent is to evaluate whether easing influences perceived progress or satisfaction.

---

## Technologies Used
- [p5.js](https://p5js.org/) for canvas rendering and animation.

---

## Components and Functions

### `class ProgressBar`
Abstract base class. Encapsulates core logic for timing and rendering.

```javascript
constructor(totalTime, barLength = 300, showPercentage = false)
````

* `totalTime`: total fill duration in seconds.
* `barLength`: width of the progress bar.
* `showPercentage`: toggle between showing percent vs. seconds remaining.

```javascript
start()
```

* Records the start timestamp using `millis()`.

```javascript
calculateProgress()
```

* Returns a float from `0.0` to `1.0` based on elapsed time.

```javascript
display(progress)`
```

* Abstract. Must be implemented by subclass.

```javascript
renderBar(progress, eased = false)
```

* Renders background and filled portion.
* If `eased` is true, modifies `progress` using:

  ```javascript
  progress = sqrt(progress);
  ```

```javascript
writeOutput(filledLength)`
```

* Displays percent or remaining time on top of the bar.

---

### `class LinearProgressBar extends ProgressBar`

Displays a **linearly increasing** progress bar.

```javascript
display(progress) {
    this.renderBar(progress);
}
```

---

### `class NonLinearProgressBar extends ProgressBar`

Displays a **non-linearly increasing** progress bar (faster initially, slows near the end).

```javascript
display(progress) {
    this.renderBar(progress, true);
}
```

---

## Control Flow

```javascript
let linearBar, nonLinearBar;
let currentBar;
let experiment = 0;
```

Tracks two bars and controls which is active.

### `setup()`

Initializes the canvas and starts the first experiment.

```javascript
createCanvas(800, 200);
linearBar = new LinearProgressBar(10, 500, true);
nonLinearBar = new NonLinearProgressBar(10, 500, true);
currentBar = linearBar;
currentBar.start();
```

### `draw()`

Renders progress continuously.

```javascript
const progress = currentBar.calculateProgress();
currentBar.display(progress);
```

* On completion of the first bar, switches to the second:

  ```javascript
  if (progress >= 1.0) {
      if (experiment === 0) {
          experiment++;
          currentBar = nonLinearBar;
          currentBar.start();
      } else {
          noLoop(); // Stop animation after both tests
      }
  }
  ```

---

## Usage

1. Open in a browser.
2. Observe two sequential progress bars:

   * First: linear fill.
   * Second: eased fill.

---

## Notes

* Designed for quick visual comparison.
* Label adapts to show either percent or time.

