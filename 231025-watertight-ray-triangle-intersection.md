# Using Numerical Analysis for Computer Graphics - Watertight Ray/Triangle Intersection

## 1. Computer Graphics and SFU MACM 316 - Numerical Analysis

SFU Computing Science program requires MACM 316: Numerical Analysis for all students. It is the only Math/MACM course required in upper division, and unfortunately, it is one of the most hated course in the program.

Some of the topics covered in MACM 316 include
- Number systems and errors (floating point representations)
- Systems of linear equations

While reading about Watertight Ray/Traingle Intersection algorithm I realized that the algorithm is a good example of practical use of these topics, so I decided to write them up.

This article assumes background knowledge in floating point representations and counting flops, covered in SFU MACM 316.

### 1-2. TL;DR

- Perform ray-triangle intersection by affine transforming the triangle coordinates to be relative to the ray, then the target intersection point becomes at (0, 0, z) in which the floating point error gets minimized. When edge positions show that the intersection seems to be close to at least one edge, fallback the whole computation to double precision. The affine transformation gives advantage in flops because the zero term in the intersection point can be removed from the edge test functions.


## 2. Problem description

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

There are two causes here.

1) Use of single precision float (instead of double precision float)
2) Accumulation of errors through +, -, x, /

We don't want to use too much double precision floating points, since double precision takes 2x memory and 2x time. It is common that raytracing takes hours to days, so it is better to avoid unnecessary double precisions.

In fact, **Watertight Ray/Triangle intersection** fix these problems in a way that

1) **Uses double precision only when required, single precision otherwise** (double precision fallback)
2) **Takes less flops**
3) **Watertight** (May contain floating point errors, but the error is very low and does not give empty spots between the faces)

## 3. Watertight Ray/Triangle intersection

Watertight Ray/Triangle intersection works in 2 stages.

### 3-1. Affine transformation

Affine transformation: We construct a 4x4 matrix, called an Affine transformation matrix, that maps the world coordinate to a space that the ray origin is at (0, 0, 0) and the ray direction is on the +z direction. 

<details>
  <summary><b>Why matrix multiplication?</b> (SFU MATH 240 - Linear Algebra things)</summary>
  <table><tr><td>
  <p>Rotation and scale operations can be done as a matrix multiplication on 3d vectors, as shown in the image below.
  <br>
  <p></p><img src="https://github.com/havocado/notes/assets/47484587/b6a40ab9-fa2f-4f7a-aa08-cef82d785b38" width="500">

  *(Image: https://en.wikipedia.org/wiki/Transformation_matrix)*

  <p>The image shows possible operations on a 2D square. In our case of 3d, the matrices would be 3x3.

  <p>One advantage we get from this kind of representation is that matrix multiplications are associative, so if we have two or more points to apply the transformation on, we could multiply the transformation matrix first, and then do only one matrix operations per point.
  </td></tr></table>
</details>

<details>
  <summary><b>Why 4x4 matrix, not 3x3?</b></summary>
  <table><tr><td>
  <p>We are using Homogeneous coordinates and Affine transformations.
    
  <p>Since I am not assuming the reader to have any background except basic numerical analysis and linear algebra, I tried to explain as briefly as possible.
    
  <ul>
  <li>Scale and rotation operations are linear and can be represented as a 3x3 matrix, but translation doesn't have a 3x3 matrix representation, because translation operation is not linear.
  <li>However it is possible to make it a 4x4 matrix by extending the coordinate of the points by one. This is called a 'homogeneous coordinate', and we put 1 as the fourth entry of the coordinate.
  <li>Then we compute a 4x4 matrix that performs translation and rotation. This matrix is called an 'affine transformation matrix'.
  <li>After the matrix multiplication, we convert the 4d homogeneous coordinate back to the usual 3d coordinate.
  <li>Note: Affine transformation also can perform scale/shear operations, but this doesn't apply on our case, because we don't scale/shear rays in raytracing.</ul>
  </td></tr></table>
</details>

We apply the transformation to the three vertices of the traingles. This make three matrix-vector multiplications, dimensions 4x4 and 4.

#### 3-1-1. Counting flops

Counting floating points operations **(counting flops)** is a way to roughly measure the speed of methods, taught in SFU MACM 316 - Numerical Analysis.

Counting flops comes from the idea that floating point operations are much slower compared to most other parts of the program such as integer operations. It is not an accurate measure of time, and ignores non-floating point operations or memory reads/writes. However it is a useful approximation of numerical computation costs.

We count the number of +, -, *, /, and additionally, <, <=, and >, >= as one flop each.
- <details>
  <summary><b>Why these 8 operations?</b></summary>
    <table><tr><td>
  <p>MACM 316 - Numerical Analysis specifies that +, -, *, / count as one flop each. 
    
  <p><, <=, and >, >= counts as a flop by <a href="https://arstechnica.com/civis/threads/counting-flops.512780/#:~:text=Since%20comparisons%20are%20typically%20done%20in%20arithmetic%2C">this</a>
  
  <p>I recently learned that some people count a+b*c as one flop, as some hardwares can efficiently compute the a+b*c form of computation. I only need a rough approximation so I will stick with counting each of +, -, *, / as one flop. 
    </td></tr></table>
</details>

Counting the number of flops in the affine transformation step:
<table><tr><td>
1) Each entry of the resulting 4d coordinate. is a dot product of a row and the point. <br>
&nbsp; - Each row and the point have 4 entries, which make the dot product cost 4 multiplication. <br>
&nbsp; - There are 4 multiplied terms, so the dot product cost an extra 3 additions. <br>
Hence the total cost of a dot product is <b>7 flops</b>.<br>
2) There are 4 entries in the resulting coordinate. This makes 7*4=<b>28 flops</b> for each matrix operation. <br>
3) Additionally, We need to convert the homogeneous coordinate back to the 3d cartesian coordinate. This takes 3 divisions in total, so we have <b>31 flops</b> per vertex in total.<br>
4) We have a traingle, so we need 3 matrix operations in total. This makes it <b>93 flops</b> in total.
</td></tr></table>

Leaves us 91 flops in total.

