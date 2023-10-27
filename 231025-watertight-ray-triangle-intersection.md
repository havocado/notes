# Using Numerical Analysis for Computer Graphics - Watertight Ray/Triangle Intersection

### 1. Computer Graphics and SFU MACM 316 - Numerical Analysis

SFU Computing Science program requires MACM 316: Numerical Analysis for all students. It is the only Math/MACM course required in upper division, and unfortunately, it is one of the most hated course in the program.

Some of the topics covered in MACM 316 include
- Number systems and errors (floating point representations)
- Systems of linear equations

While reading about Watertight Ray/Traingle Intersection algorithm I realized that the algorithm is a good example of practical use of these topics, so I decided to write them up.

This article assumes background knowledge in floating point representations and counting flops, covered in SFU MACM 316.

#### 1-2. TL;DR

- Perform ray-triangle intersection by affine transforming the triangle coordinates to be relative to the ray, then the target intersection point becomes at (0, 0, z) in which the floating point error gets minimized. When edge positions show that the intersection seems to be close to at least one edge, fallback the whole computation to double precision. The affine transformation gives advantage in flops because the zero term in the intersection point can be removed from the edge test functions.


### 2. Problem description

**Ray-triangle intersection** in raytracing is a problem of deciding if a line intersects a triangle in a 3D space, and finding the intersection point. The input is given as

- A line described by 1 point and 1 vector: Origin and direction
- A triangle described by 3 points, and an optional face normal (face orientation vector)

A straightforward method would be testing if the line intersects a plane that includes the traingle, and then checking if the intersection point is within a triangle. The method is explained [here](https://github.com/havocado/notes/blob/2310/ray-triangle-intersection/231018-triangle-collision-raytracing.md).

However this method involves a lot of floating point operations that uses coordinates. This means if the coordinates are much larger than the objects, floating point errors may add up.

Example scene: The radius of the mint colored sphere has diameter 1m.

| Original scene at (0,0,0)    | Everything shifted 20,000 meters away from origin         |
|----------------------------------------|-----------------------------------|
| <img src="https://github.com/havocado/notes/assets/47484587/49fd7d3d-3b4c-41d9-be12-b7ab766ccbbb" width="400">        | <img src="https://github.com/havocado/notes/assets/47484587/f38cf609-230b-493a-8bac-d323ae4fb73b" width="400"> |

The single precision floating point epsilon is $\approx$ 0.0156 at 20,000. The error accumulates on every floating point operation that involves the coordinates, ending up in visible errors.

![image](https://github.com/havocado/notes/assets/47484587/a63bbc29-3e7c-42d3-bef1-c138a61fa8cc)

There are two issues here:
1) Use of single precision float, instead of double precision float
2) Accumulation of errors

We don't want to use too much double precision floating points, since double precision takes 2x memory and 2x time. It is common that raytracing takes hours to days, so it is better to avoid unnecessary double precisions.

In fact, **Watertight Ray/Triangle intersection** fix these problems in a way that

1) **Uses double precision only when required, single precision otherwise (double precision fallback)**
2) **Takes less flops**
3) **Watertight in terms of errors (achieves minimum error possible in floating points)**

### 3. Watertight Ray/Triangle intersection

