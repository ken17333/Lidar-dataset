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

