---
layout: project
title: Domino Stacking Robot
date: 2026-01-04
image: /assets/images/domino-stacking-robot.jpg   # full path or URL
authors: Gregory Aiosa, Daniel Augustin, Michael Jenz, Chenyu Zhu
---

## Key Contributions
* **Autonomous Perception:** Developed an RGB-D pose estimation pipeline for small-scale object detection.
* **Collision-Aware Planning:** Implemented a staged manipulation strategy to navigate tight spatial constraints.
* **Force-Feedback Control:** Integrated joint effort monitoring to achieve robust placement on non-uniform surfaces.
* **ROS 2 Ecosystem:** Orchestrated a multi-node system involving MoveIt, RealSense, and custom CV nodes.

---

## System Architecture
The system operates within the **ROS 2** ecosystem, leveraging a service-oriented architecture to coordinate between perception and motion planning.



### 1. Domino Vision Algorithm
The vision pipeline identifies both the **position and orientation** of dominoes to generate actionable TF frames.
* **Position ID:** Uses color filtering and depth data to compute 3D coordinates.
* **Orientation ID:** Employs OpenCV bounding boxes to extract the yaw (z-axis rotation) and convert it to a quaternion.

> **Challenge:** We initially assumed a perfectly flat table. When physical variations caused compounding errors, we pivoted to a force-sensing approach rather than relying solely on vision.



### 2. Domino Movement Algorithm
To avoid collisions between the bulky Franka Emika gripper and the dense domino patterns, we utilized a three-stage manipulation pipeline:
1.  **Initial Pickup:** Retrieval from the detected starting pose.
2.  **Staging (Reorientation):** The robot stands the domino up vertically. This "top-down" grip is essential for high-density placement.
3.  **Final Placement:** Precise insertion into the goal pattern (Straight, Circle, or Squiggle).

### 3. Force Sensing Placement Algorithm
To compensate for uneven workspace surfaces, we implemented a **force-feedback loop** that monitors **Joint 2 effort** on the FER.
* **The Logic:** The robot moves in 1mm increments; if the effort threshold is exceeded, contact is confirmed.
* **Scene Management:** We dynamically add/remove collision objects in the MoveIt planning scene during this phase to prevent "stuck in object" errors.



---

## Technical Implementation

### Calibration
Accurate extrinsic calibration was achieved using `easy_handeye2`. 
* **Method:** Eye-in-hand calibration.
* **Storage:** Calibration files are published via the `handeye_publisher` node during the main launch sequence.

### Nodes & ROS 2 Structure


<div style="background-color: #f9f9f9; padding: 20px; border-radius: 8px; border-left: 5px solid #2d5af7;">
  <p><strong>🤖 place_dominoes</strong><br>
  High-level state machine & motion planning via MoveIt.</p>
  
  <p><strong>👁️ find_dominoes</strong><br>
  OpenCV-based perception and TF broadcasting.</p>
  
  <p><strong>🏷️ apriltag_node</strong><br>
  Optional secondary localization support.</p>
</div>


---

## Quickstart

### Real-World Demo
# 1. Launch the system
ros2 launch place_dominoes domino.launch.xml open_cv:=true demo:=false pattern:=straight_line.yaml

# 2. Trigger movement
ros2 service call /place_dominoes std_srvs/srv/Empty

# 3. Topple
ros2 service call /topple_dominoes std_srvs/srv/Empty