# Lidar-dataset
A human pose dataset

# Method for Correcting Body Joint Coordinates under Occlusion

First, we use photos of a person **without a blanket** to obtain reference distance information between body joints.  
For example, the length of the torso can be calculated as:

$$
d = \sqrt{(x_1 - x_2)^2 + (y_1 - y_2)^2 + a \cdot (z_1 - z_2)^2}
$$

Here, the factor **`a`** is introduced because the scale of the **Z-axis** is not equivalent to that of the **X/Y axes**.

---

When a blanket covers the body, we assume that the **Z-coordinate values of the annotated points may be inaccurate**, while the **X and Y coordinates remain reliable**.  
In this case, we can leverage the reference distance information obtained earlier to correct the coordinates:

- Since the **shoulders are almost always visible**, we can use the distances between **shoulders, elbows, and hips** as fixed references.  
- These stable reference distances allow us to **adjust the positions of covered joints** with inaccurate Z-coordinates.  
- Once the shoulder, elbow, and hip positions are corrected, we can further propagate these corrections to refine the coordinates of other joints.  
# Method for Classifying Human Postures

Using the previously obtained **3D joint coordinates**, we classify body postures based on geometric rules.  
Since the **Z-axis scale is not equivalent to the X/Y axes**, the Z component of every vector is multiplied by a constant factor `a` before computing distances or angles.  

---

### Vector Normalization with Z-Scaling
Given two joints with coordinates $\left(x_1,\ y_1,\ z_1\right)$ and $\left(x_2,\ y_2,\ z_2\right)$,  
the adjusted vector is defined as:

$$
\vec{v} = \big(x_2 - x_1,\; y_2 - y_1,\; a \cdot (z_2 - z_1)\big)
$$

All subsequent angle calculations are performed using these adjusted vectors.

---

### 1. Arm Position (Close to Body vs. Open)
- **Reference joints:** Hip, Shoulder, and Elbow  
- **Rule:** Compute the angle formed by these three joints.  
  - If the angle **> 30°**, the arm is considered **open**.  
  - Otherwise, it is considered **close to the body**.

---

### 2. Arm Flexion (Straight vs. Bent)
- **Reference joints:** Shoulder, Elbow, and Wrist  
- **Rule:** Compute the angle at the elbow.  
  - If the angle **< 150°**, the arm is considered **bent**.  
  - Otherwise, it is considered **straight**.

---

### 3. Leg Position (Together vs. Apart)
- **Reference joints:** Left/Right Hips and Knees  
- **Rule:** Form two vectors (hip-to-knee for each leg) and measure their angle.  
  - If the angle **> 30°**, the legs are considered **apart**.  
  - Otherwise, the legs are considered **together**.

---

### 4. Leg Flexion (Straight vs. Bent)
- **Reference joints:** Hip, Knee, and Ankle  
- **Rule:** Compute the angle at the knee.  
  - If the angle **< 150°**, the leg is considered **bent**.  
  - Otherwise, it is considered **straight**.

---

### 5. Body Orientation (Supine vs. Side-lying)
- **Reference joints:** Left and Right Shoulders  
- **Rule:** Compare the X and Y coordinate distance between the two shoulders.  
  - If the distance is **less than half** of the normal supine posture, the body is considered **side-lying**.  

---

### 6. Classification Outcome
By combining the above binary conditions (arm open/close, arm bent/straight, leg apart/together, leg bent/straight, body supine/side-lying),  
we can categorize human postures into **32 distinct classes**.

---

# What Defines a Single Data Sample?

Our LiDAR operates at **30 FPS**.  
To ensure sufficient variation between consecutive actions,  
we select frames at an interval of approximately **5 frames** (≈ 0.17 seconds).

---

# What Data Did We Use for Training?

For training, we used a total of **10,000 images**:  
- **5,000 images without a blanket**  
- **5,000 images with a blanket**

The dataset was **randomly split** into **training** and **testing** sets with an **8:2 ratio**,  
ensuring that both categories (with and without blanket) were proportionally represented in each subset.

---

# Data Augmentation Methods

To improve model generalization and increase dataset diversity,  
we applied the following **data augmentation techniques** during training:

- **Random Flipping**: Randomly flipped images to simulate mirrored postures.  

- **Random Scaling (Label-Guided)**:  
  The scaling operation was performed using the **labeled joint coordinates**.  
  Specifically, we determined the bounding region of the human body from the annotated keypoints,  
  then applied random scaling to this region.  
  This ensures that the augmentation focuses on the human body rather than irrelevant background areas.  

- **Noise Injection**: Added random noise to the input images to enhance robustness against sensor imperfections and environmental disturbances.

---

# How to Obtain the Final Joint Coordinates

The simplest approach is to directly take the location of the maximum value in the heatmap as the joint position.  
However, in our method, we use a **weighted sum** formulation.

---

For the $k$-th joint, its coordinate $J_k$ is calculated as:

$$
J_k = \sum_{z=1}^{D} \sum_{y=1}^{H} \sum_{x=1}^{W} p \cdot \tilde{H}_k(p)
$$

where:
- \(p = (x, y, z)\) represents the coordinate position,  
- \(H, W, D\) are the dimensions of the heatmap,  
- $\tilde{H}_k(p)$ is the **normalized heatmap** value at location \(p\).

---

The normalized heatmap \(\tilde{H}_k(p)\) is obtained by applying softmax:

$$
\tilde{H}_k(p) = \frac{e^{H_k(p)}}{\sum_{z=1}^{D} \sum_{y=1}^{H} \sum_{x=1}^{W} e^{H_k(q)}}
$$

---

Finally, the resulting coordinate \(J_k\) is mapped back to the original input image coordinates by applying the appropriate scaling factor.



