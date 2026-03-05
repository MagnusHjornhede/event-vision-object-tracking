# Event-Based Object Tracking with Dynamic Vision Sensors

<br>

![Reference Visualization](reference.gif)

This project implements an object tracking pipeline using event-based data from a Dynamic Vision Sensor (DVS).

Unlike conventional frame-based cameras that capture images at fixed frame rates, event cameras asynchronously report brightness changes with microsecond temporal resolution. This fundamentally different sensing paradigm enables low-latency, high-dynamic-range perception.

Implementation: `ObjectTrackingWithEventCamera.ipynb`

<br>

## Project Goal

The objective is to estimate the angular motion of a rotating object directly from raw event streams.

The project investigates how:

- Event representations
- Temporal aggregation strategies
- Spiking neural network dynamics

affect tracking accuracy, stability, and robustness.

If you're new to event-based vision:
- Event camera overview: https://ieeexplore.ieee.org/document/4444573  
- Survey on event-based vision (Gallego et al., 2020): https://arxiv.org/abs/1904.08405  

<br>

## Pipeline

event stream
→ temporal binning
→ event frames
→ feature extraction
→ spiking neural network
→ angular velocity estimation

<br>

## Event Representation

Each event is represented as:

`(x, y, t, p)`

where:

- x, y — pixel coordinates  
- t — timestamp  
- p — polarity (brightness increase or decrease)

Events are grouped into fixed temporal bins and converted into grid-based representations suitable for neural processing.

This transforms asynchronous event streams into structured tensors while preserving temporal dynamics.

<br>

## Ground Truth Generation

Ground truth (GT) wheel angles are extracted automatically from the event stream using classical computer vision. No manual annotation is required.

### Pipeline

1. Events are grouped into fixed time bins (e.g., 10 ms).
2. For each bin:
   - Events are accumulated into an image.
   - Log compression stabilizes intensity.
   - Canny edge detection extracts edges.
   - Hough line transform detects dominant line angles.
3. The angle closest to the previous bin’s angle is selected  
   → enforcing temporal continuity.
4. Because angles are π-periodic:
   - They are unwrapped correctly.
   - The first valid angle is set to zero as reference.

This produces a dense angular trajectory:

θ(t)

along with a validity mask.

The key idea is that temporal continuity and π-periodic unwrapping produce physically consistent motion estimates instead of noisy frame-wise detections.

If unfamiliar with these techniques:
- Canny edge detection: https://en.wikipedia.org/wiki/Canny_edge_detector  
- Hough transform: https://en.wikipedia.org/wiki/Hough_transform  

<br>

## Model Architecture

The model predicts angular increments rather than absolute angles:

Δθ(t) 

### Input Representation

For each temporal window:

- Event frames are constructed from time bins
- Spatially cropped
- Downsampled
- Log-scaled
- Flattened into feature vectors

<br>

### Network Structure

The tracking model is a spiking multilayer perceptron using Leaky Integrate-and-Fire (LIF) neurons:

Input → Linear → LIF → Linear → LIF → Linear → Δθ

LIF neurons maintain membrane state across timesteps, enabling temporal integration within each window.

The implementation uses spiking dynamics inspired by neuromorphic computing frameworks such as:
- snnTorch: https://snntorch.readthedocs.io/

<br>

## Training Objective

Training combines two complementary loss components:

1. Masked Huber loss on predicted angular increments (Δθ)
2. Trajectory consistency loss:
   - Predicted Δθ values are integrated (cumulative sum)
   - The resulting trajectory is compared to ground truth θ(t)

Total loss = local motion accuracy + global trajectory consistency

This prevents drift and stabilizes long-term motion estimation.

<br>

## Why Event-Based Tracking?

This project combines several modern research ideas:

- **event-based sensing**
- **self-generated ground truth via classical vision**
- **spiking neural networks**
- **trajectory-aware training**

Together they form a compact example of **continuous-time tracking using asynchronous sensors** and **neuromorphic-inspired models**.

<br>

## References

If you're new to event-based vision:

Event camera overview  
https://ieeexplore.ieee.org/document/4444573

Event-based vision survey (Gallego et al., 2020)  
https://arxiv.org/abs/1904.08405

Canny edge detector  
https://en.wikipedia.org/wiki/Canny_edge_detector

Hough transform  
https://en.wikipedia.org/wiki/Hough_transform
