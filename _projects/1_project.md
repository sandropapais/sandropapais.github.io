---
layout: page
title: EagerMOT - Multi-Object Tracking
description: Advanced multi-object tracking framework combining 3D object detection with robust tracking algorithms
img: assets/img/eagermot_preview.jpg
importance: 1
category: research
github: https://github.com/sandropapais/EagerMOT
related_publications: true
---

## EagerMOT: 3D Multi-Object Tracking Framework

EagerMOT is an advanced multi-object tracking system that combines state-of-the-art 3D object detection with robust tracking algorithms for autonomous driving applications.

### Key Features

- **Real-time Performance**: Optimized for real-time processing of sensor data
- **Multi-Modal Fusion**: Combines LiDAR point clouds and camera images  
- **Robust Tracking**: Maintains object identity across frames even with occlusions
- **KITTI Benchmark**: Evaluated on standard autonomous driving datasets

### Technical Approach

The system employs a two-stage approach:

1. **3D Object Detection**: Leverages deep learning models to detect objects in 3D space
2. **Multi-Object Tracking**: Associates detections across time using advanced data association algorithms

### Results

Our approach achieves competitive performance on the KITTI tracking benchmark, demonstrating effectiveness in real-world autonomous driving scenarios.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/tracking_result1.jpg" title="Tracking results" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/tracking_result2.jpg" title="Multi-object tracking" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Multi-object tracking results showing robust identity maintenance across frames.
</div>