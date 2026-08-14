
# Hough Transform for Digital Image Processing

This repository contains the implementation and experimental analysis of several Hough Transform techniques developed as part of the Digital Image Processing course at Shiraz University.

The project focuses on implementing the fundamental algorithms from scratch using numerical and matrix operations rather than relying on built-in computer vision functions.

Three major computer vision problems are investigated:

1. Traffic Light Detection and State Classification using Hough Circles
2. Autonomous Lane Detection using the Polar Hough Line Transform
3. Real Object Localization using the Generalized Hough Transform

---

## Project Overview

The Hough Transform is a classical computer vision technique used to detect geometric structures and objects in images.

In this project, different variations of the Hough Transform are implemented and evaluated for detecting circles, lines, and arbitrary object shapes.

A major objective of the project is to understand the mathematical and computational foundations of these algorithms.

Therefore, the core image processing operations are implemented manually using NumPy and fundamental mathematical operations.

High-level computer vision functions for edge detection and Hough Transform operations are avoided in the main processing pipelines.

---

# Part 1 - Traffic Light Detection and State Classification

The first part of the project focuses on detecting traffic light lamps and identifying their active states using a custom Hough Circle Transform.

The system processes traffic light images and attempts to locate the three circular lamp positions.

Each detected lamp is then classified as:

- RED
- YELLOW
- GREEN
- UNLIT

## Processing Pipeline

The traffic light detection pipeline consists of the following stages:

1. Image loading and resizing
2. Grayscale conversion
3. Region of Interest detection
4. Gaussian smoothing
5. Sobel edge detection
6. Gradient-based Hough Circle voting
7. Hough peak extraction
8. Geometric filtering
9. Lamp color classification
10. Final visualization

---

## Custom Edge Detection

Before performing Hough voting, the input image is converted to grayscale and smoothed using a manually generated Gaussian kernel.

The Sobel operator is then applied to calculate horizontal and vertical image gradients.

The gradient magnitude and orientation are calculated from the horizontal and vertical derivatives.

The gradient information is particularly important because it is later used to optimize the Hough Circle voting process.

---

## Gradient-Based Hough Circle Transform

A circle can be represented using three parameters:

- center coordinate `a`
- center coordinate `b`
- radius `r`

The standard circle equation is:

```text
(x - a)^2 + (y - b)^2 = r^2
```

A conventional brute-force Hough Circle Transform requires searching through many possible center positions, radii, and angles.

This approach is computationally expensive.

To improve efficiency, the implementation uses the gradient direction calculated during edge detection.

For every edge pixel and candidate radius, possible circle centers are estimated along the gradient normal direction.

Instead of voting across the complete angular space, each edge pixel votes only for two possible center locations.

This significantly reduces the computational complexity of the circle detection stage.

---

## Traffic Light Geometry

A standard traffic light contains three approximately circular lamps arranged vertically.

The system uses this geometric structure to improve detection reliability.

Candidate circles are filtered according to:

- vertical alignment
- similar radius
- horizontal center consistency
- relative lamp position

These constraints help eliminate false circle detections caused by background objects or the traffic light housing.

---

## Traffic Light State Classification

After detecting the lamp positions, the RGB values inside each circle are analyzed.

An inner circular mask is used so that the dark outer boundary of the traffic light does not dominate the color statistics.

The algorithm evaluates the relative intensity and dominance of the red, green, and blue channels.

The detected lamp is then classified as:

```text
RED
YELLOW
GREEN
UNLIT
```

A lamp is classified as `UNLIT` when none of the active color conditions exceed the required activity threshold.

---

## Traffic Light Results

The system was evaluated on several traffic light images.

For the green traffic light image, the detected state sequence was:

```text
UNLIT
UNLIT
GREEN
```

For the red traffic light images, the detected state sequence was:

```text
RED
UNLIT
UNLIT
```

The final detected circles and classifications are drawn directly on the original images.

Intermediate results are also generated, including:

- Original image
- Grayscale image
- Edge map
- Hough accumulator heatmap
- Final detection overlay

---

# Part 2 - Autonomous Lane Detection

The second part of the project implements an autonomous lane detection system using the Polar Hough Line Transform.

The objective is to detect the primary left and right lane boundaries from a sequence of road images.

The final detected lanes are combined to visualize the estimated drivable region.

---

## Lane Detection Pipeline

The lane detection system consists of:

1. Image preprocessing
2. Color thresholding
3. Gaussian smoothing
4. Region of Interest masking
5. Custom edge detection
6. Polar Hough Line Transform
7. Hough peak extraction
8. Lane classification
9. Lane averaging
10. Temporal smoothing
11. Lane overlay generation

---

## Color Thresholding

Road lane markings are typically white or yellow.

Color thresholding is therefore used to suppress irrelevant image regions and preserve likely lane pixels.

White and yellow candidate pixels are extracted using RGB channel conditions.

The resulting mask is combined with the grayscale image before edge detection.

---

## Region of Interest

A trapezoidal Region of Interest is applied to the lower portion of each road image.

This allows the algorithm to focus on the road surface while ignoring irrelevant regions such as:

- sky
- roadside objects
- distant background structures
- vehicle hood

The ROI is generated using vectorized NumPy operations.

---

## Custom Canny-Style Edge Detection

A Canny-style edge detection pipeline is implemented manually.

The main stages include:

### Gaussian Smoothing

A Gaussian kernel is generated mathematically and applied to reduce image noise.

### Sobel Gradient

Horizontal and vertical image derivatives are calculated using Sobel kernels.

### Non-Maximum Suppression

Non-Maximum Suppression is used to thin the detected edges.

The gradient orientation is divided into directional categories and neighboring pixels are compared along the gradient direction.

### Hysteresis Thresholding

Strong and weak edge pixels are identified using two thresholds.

Weak edges connected to strong edges are preserved, while isolated weak responses are removed.

---

# Polar Hough Line Transform

The standard Polar Hough Transform represents a line using:

```text
rho = x cos(theta) + y sin(theta)
```

Each detected edge pixel votes for possible line parameters in Hough space.

The votes are accumulated in a two-dimensional accumulator representing `rho` and `theta`.

Strong peaks in this accumulator correspond to dominant lines in the image.

---

## Lane Classification

Detected Hough lines are converted back into image-space lines.

The slope of each line is used to determine whether it represents the left or right lane boundary.

In general:

```text
Negative slope -> Left lane

Positive slope -> Right lane
```

Nearly horizontal lines are rejected because they are unlikely to represent lane boundaries.

Multiple detected segments belonging to the same side are combined to generate a single representative lane line.

---

## Temporal Smoothing

Because the input consists of sequential road images, lane positions may vary slightly between frames.

Temporal smoothing is used to reduce visual jitter.

The current lane coordinates are blended with lane coordinates from previous frames.

This produces smoother lane movement across the image sequence.

---

## Lane Overlay

After detecting the left and right lane boundaries, the area between them is filled with a semi-transparent polygon.

The final output therefore provides a visual estimate of the drivable road region.

The processed frames are combined into an animated GIF running at approximately 2 frames per second.

---

# Part 3 - Real Object Localization

The third part investigates localization of an arbitrary object using the Generalized Hough Transform.

Unlike circles and lines, arbitrary objects cannot be described using a simple parametric equation.

The Generalized Hough Transform solves this problem by representing the geometric relationship between object boundary points and a reference point.

In this project, the target object is a flower.

---

## Object Localization Pipeline

The processing pipeline includes:

1. Template image preprocessing
2. Test image preprocessing
3. Grayscale conversion
4. Edge detection
5. Gradient orientation calculation
6. Reference point selection
7. R-Table construction
8. Generalized Hough voting
9. Accumulator analysis
10. Object localization
11. Final visualization

---

## Edge and Gradient Detection

Edge detection is performed on both the template image and the test image.

For every edge pixel, the gradient magnitude and gradient direction are calculated.

Gradient orientation provides important information about the local geometry of the object boundary.

---

## Reference Point

A reference center is selected for the template object.

The displacement between every template edge pixel and this reference point is calculated.

These displacement vectors describe the geometric relationship between the object's boundary and its center.

---

# R-Table Construction

The R-Table is the central data structure used by the Generalized Hough Transform.

Template edge pixels are grouped according to their gradient orientation.

For each orientation, displacement vectors pointing from the edge location toward the reference point are stored.

Conceptually, the table maps:

```text
Gradient Orientation
        |
        v
Displacement Vectors to Object Center
```

This allows the geometric structure of an arbitrary object to be represented without requiring a parametric shape equation.

---

# Generalized Hough Voting

During detection, each edge pixel in the test image is examined.

Its gradient orientation is used to retrieve corresponding displacement vectors from the R-Table.

Each displacement predicts a possible object center.

Votes are accumulated in a two-dimensional accumulator representing possible reference-point locations.

The strongest peak in the accumulator indicates the most likely location of the target object.

---

## Object Localization Results

The Generalized Hough Transform produces several intermediate visualizations:

- Original test image
- Grayscale test image
- Edge map
- Hough accumulator heatmap
- Final detected object location

The strongest accumulator peak is selected as the estimated position of the flower.

The detected location is then visualized on the original color image.

---

# Vectorization and Performance

Performance optimization is an important component of this project.

Pixel-level nested Python loops are avoided whenever possible.

NumPy vectorization techniques are used extensively throughout the implementation.

Important optimization techniques include:

- NumPy broadcasting
- Boolean masking
- Array slicing
- `stride_tricks`
- `einsum`
- `np.roll`
- `np.where`
- `np.add.at`

These techniques allow large groups of pixels to be processed simultaneously.

---

## Hough Voting Optimization

Hough voting can become computationally expensive because many edge pixels may vote for many possible parameters.

Different optimization strategies are therefore used depending on the problem.

For circle detection, gradient orientation reduces the number of possible circle centers.

For line detection, voting operations are vectorized across edge coordinates and sampled angles.

For object localization, gradient orientations are used to retrieve only relevant displacement vectors from the R-Table.

---

# Performance Analysis

Execution time is measured for the major processing stages.

The analyzed stages include:

- Preprocessing
- Edge Detection
- Hough Voting
- Post-processing
- Visualization

This makes it possible to identify the computationally expensive components of each pipeline.

The experiments demonstrate that vectorization significantly improves the practicality of Hough-based detection methods.

---

# Project Structure

```text
hough-transform-image-processing/
|
|-- Part1/
|   |-- Task1.ipynb
|   |-- images/
|   |   `-- traffic_lights/
|   `-- outputs_part1_corrected/
|
|-- Part2/
|   |-- Task2.ipynb
|   `-- lane_detection.gif
|
|-- Part3/
|   |-- Task3.ipynb
|   |-- images/
|   |   `-- object_localization/
|   `-- outputs/
|       `-- object_localization/
|
|-- Report.pdf
`-- README.md
```

---

# File Description

| File / Directory | Description |
| --- | --- |
| `Part1/Task1.ipynb` | Traffic light detection and classification using Hough Circles |
| `Part1/images/traffic_lights/` | Input traffic light images |
| `Part1/outputs_part1_corrected/` | Intermediate and final traffic light detection results |
| `Part2/Task2.ipynb` | Autonomous lane detection using the Polar Hough Line Transform |
| `Part2/lane_detection.gif` | Animated lane detection result |
| `Part3/Task3.ipynb` | Object localization using the Generalized Hough Transform |
| `Part3/images/object_localization/` | Template and test images used for object localization |
| `Part3/outputs/object_localization/` | Intermediate and final localization results |
| `Report.pdf` | Complete academic project report |
| `README.md` | Repository documentation |

---

# Technologies and Methods

- Python
- Jupyter Notebook
- NumPy
- Matplotlib
- OpenCV for permitted image I/O and basic operations
- ImageIO
- Digital Image Processing
- Gaussian Filtering
- Sobel Edge Detection
- Non-Maximum Suppression
- Hysteresis Thresholding
- Region of Interest Masking
- Hough Circle Transform
- Polar Hough Line Transform
- Generalized Hough Transform
- R-Table
- Gradient-Based Voting
- Traffic Light Classification
- Lane Detection
- Object Localization
- Vectorized Numerical Computing

---

# Key Concepts Demonstrated

This project demonstrates several fundamental concepts in classical computer vision and digital image processing:

- Image preprocessing
- Custom convolution
- Gradient calculation
- Edge detection from scratch
- Gradient magnitude and orientation
- Hough parameter spaces
- Hough accumulator construction
- Circle detection
- Line detection
- Arbitrary shape localization
- Gradient-guided voting
- Peak extraction
- Geometric filtering
- Color-based classification
- Lane boundary estimation
- Temporal smoothing
- Generalized Hough Transform
- R-Table construction
- NumPy vectorization
- Algorithm performance analysis

---

# Conclusion

This project demonstrates how different variations of the Hough Transform can be applied to practical computer vision problems.

The Hough Circle Transform is used to detect and classify traffic light lamps, the Polar Hough Line Transform is used to identify road lane boundaries, and the Generalized Hough Transform is used to localize an arbitrary object based on its edge geometry.

Implementing the major algorithms from scratch provides a deeper understanding of Hough parameter spaces, accumulator voting, gradient information, edge detection, geometric constraints, and computational optimization.

The experiments also demonstrate the importance of vectorized numerical operations when implementing computationally demanding image processing algorithms.

---

## Course Information

**Course:** Digital Image Processing  
**University:** Shiraz University  
**Instructor:** Prof. Dr. Zohreh AzimiFar  
**Homework:** 3 - Hough Transform  
**Semester:** Spring 2026

---

## Author

Saghar Kheradmand
