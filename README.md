# Event-Based Object Tracking with Dynamic Vision Sensors
![Reference Visualization](reference.gif)

## Project Description

This project implements an object tracking pipeline using event-based data from a Dynamic Vision Sensor (DVS). Unlike conventional frame-based cameras, event cameras asynchronously output brightness-change events with microsecond temporal resolution.

The goal of the project is to:

* Process raw event streams
* Convert asynchronous events into structured representations
* Perform object localization and tracking over time
* Evaluate tracking performance under controlled experimental conditions

The implementation is contained in:

```
ObjectTrackingWithEventCamera.ipynb
```

## Core Concept

Each event consists of:

[
(x, y, t, p)
]

where:

* (x, y) = pixel coordinates
* (t) = timestamp
* (p) = polarity

Events are aggregated into temporal windows and transformed into grid-based representations suitable for tracking algorithms.

## Objective

The system investigates how event representations and temporal aggregation strategies affect tracking accuracy, stability, and latency.

## Ground Truth Generation

Ground truth (GT) wheel angles are **automatically extracted from the event stream** using classical vision methods. No manual labeling is used.

### Pipeline

1. Events are grouped into fixed time bins (e.g., 10 ms).
2. For each bin:
   - Events are accumulated into an image.
   - Log compression is applied.
   - Canny edge detection extracts edges.
   - Hough line transform detects dominant line angles.
3. The angle closest to the previous bin’s angle is selected  
   → enforces **temporal continuity**.
4. Angles are π-periodic, so they are:
   - properly unwrapped
   - shifted so the first valid angle = 0

This produces a dense, smooth angular trajectory:

θ(t)

along with a validity mask.

**Key idea:**  
Temporal continuity + π-periodic unwrapping ensures physically consistent angle estimation instead of noisy per-bin detections.

---

## Model Architecture and Learning Objective

The model predicts **angular increments**, not absolute angles.

Δθ(t)

### Input Representation

For each time window:

- Event frames are constructed from time bins
- Spatially cropped
- Downsampled
- Log-scaled
- Flattened into feature vectors

### Architecture

A spiking MLP with Leaky Integrate-and-Fire (LIF) neurons:

Input → Linear → LIF → Linear → LIF → Linear → Δθ

LIF neurons maintain membrane state across timesteps, providing temporal memory inside each window.

### Training Objective

Two loss components are used:

1. Masked Huber loss on Δθ  
2. Trajectory consistency loss:
   - Predicted Δθ is integrated (cumulative sum)
   - Compared to ground truth angle trajectory

Total loss = local increment accuracy + global trajectory consistency

**Key idea:**  
Supervising both instantaneous motion and integrated trajectory prevents drift and stabilizes learning.
