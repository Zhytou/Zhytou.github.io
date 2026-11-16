---
title: "Games203: 3D Reconstruction and Understanding"
date: 2026-10-08T13:52:39+08:00
draft: true
---

[Overview](https://www.cs.utexas.edu/~huangqx/Games_3D_Recons_Understanding.html)

## 1 Introduction and Overview

### 1.1 Topics to be covered

1. 3D Reconstruction
   - scanning
   - scan registration
   - surface reconstruction
   - structure from motion
   - multi-view stereo
   - map synchronization
2. How to represent 3D data
   - Conversion between different 3D representations(point cloud -> signed distance field -> mesh)
3. How to understand 3D data
   - Design algorithms to extract semantic information from one or collection of shapes

### 1.2 Scanning

1. Active Sensors:
   - RGB-D Cameras (Structured Light, Time-of-Flight): Provide both color and per-pixel depth at interactive rates.
   - LiDAR (Light Detection and Ranging): Emits laser pulses to measure long-range, high-precision point clouds (essential for autonomous driving and large-scale mapping).
2. Passive Sensors:
   - Standard multi-view digital cameras relying on computer vision algorithms to infer depth.

## 2 Registration

goal: align two shapes to find a rigid transformation between them

## 3 Surface Reconstruction

goal: convert captured point cloud to reconstructed surface model

### 3.1 Implicit Representation-Based Reconstruction Methods

[Surface Reconstruction from Unorganized Points](https://sites.stat.washington.edu/wxs/Siggraph-92/siggraph92.pdf)

- do PCA

[A Volumetric Method for Building Complex Models from Range Images](https://graphics.stanford.edu/papers/volrange/volrange.pdf)

### 3.2 Partition of Unity

## 4 Structure from Motion

goal: recover 3D sparse structures and camera motion parameters from a set of unordered 2D photographs.

### bundle adjustment

## 5 Multi-View Stereo

## 6 Map Synchronization

goal: compute maps among a collection of objects
