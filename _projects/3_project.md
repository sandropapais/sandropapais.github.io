---
layout: page
title: Radar Vehicle Tracking Simulator
description: MATLAB-based simulator for radar-based vehicle tracking and extended Kalman filter implementation
img: assets/img/radar_tracking.jpg
importance: 3
category: simulation
github: https://github.com/sandropapais/Radar_Vehicle_Tracking_Simulator
related_publications: false
---

## Radar Vehicle Tracking Simulator

A comprehensive MATLAB simulation environment for developing and testing radar-based vehicle tracking algorithms using extended Kalman filters.

### Overview

This simulator provides a realistic environment for testing radar tracking algorithms with various scenarios including:
- Multiple target tracking
- Sensor noise modeling
- Extended Kalman filter implementation
- Performance evaluation metrics

### Key Features

- **Realistic Radar Model**: Includes range and bearing measurements with configurable noise
- **Multi-Target Scenarios**: Support for tracking multiple vehicles simultaneously  
- **Kalman Filtering**: Implementation of Extended Kalman Filter (EKF) for state estimation
- **Visualization Tools**: Real-time plotting of tracks and ground truth
- **Performance Metrics**: RMSE calculation and tracking accuracy assessment

### Technical Implementation

The simulator uses a constant velocity motion model with radar measurements:

```matlab
% State vector: [x, y, vx, vy]'
% Measurement vector: [range, bearing]'

function [x_pred] = predict_state(x, dt)
    F = [1 0 dt 0;
         0 1 0 dt;
         0 0 1 0;
         0 0 0 1];
    x_pred = F * x;
end
```

### Applications

- **Autonomous Vehicles**: Testing radar-based perception systems
- **Traffic Monitoring**: Multi-vehicle tracking applications  
- **Algorithm Development**: Rapid prototyping of tracking algorithms
- **Educational Tool**: Teaching Kalman filtering concepts

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/radar_sim1.jpg" title="Tracking simulation" class="img-fluid rounded z-depth-1" %}
    </div>
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/radar_sim2.jpg" title="Multi-target scenario" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Left: Single vehicle tracking scenario. Right: Multi-target tracking with crossing paths.
</div>