# [Raytracing] Triangle mesh collision implementation

### 1. Ray-Triangle intersection problem

Ray-Triangle intersection is a problem deciding whether a given ray of form $r(t) = o + dt$ intersects a given triangle. 

Although the math involved is relatively simple, it is a function that is called very frequently within raytracing and ends up taking a huge amount of time. I tried to implement this part in reasonable efficiency, while also retrieving information such as the angle between the ray and the triangle, or the barycentric location within the triangle.

### 2. Data structure: Triangle strip

![image](https://github.com/havocado/notes/assets/47484587/aea22eae-6436-40e3-a343-89bf8110b895)

*(Image: https://en.wikipedia.org/wiki/Triangle_mesh)*

**Triangle mesh** - It is common to allow only triangles in a mesh representation, as (1) triangle guarantees that all vertices are on a plane, whereas 4 or more vertices may not be on the same plane and (2) any polygon meshes can be represented as triangle meshes.

**Indexed vertices** - Assuming single-precision floating points, a vertex coordinate takes 12 bytes, while an unsigned integer usually takes 4 or 8 bytes only. Since a vertex may be included in multiple faces, it is memory-efficient to keep an indexed list of vertex coordinates and assign the indices to the corresponding faces.

**Triangle strip** - A bunch of triangles with indexed vertices. The triangles are not necessarily ordered, and contain no information about the structure, but it is very simple to implement.

In my C++ raytracer, I would have a `Mesh` class that inherits `HittableObject` class:

(I omitted the constructors and other irrelevant functions to keep it simple - full code [here](https://github.com/havocado/raytracer-by-hailey))
```cpp
class HittableObject {
public:
    Point3 position;
    Matrix3x3 rotationMatrix;

    virtual CollisionData rayCollisionPoint(const Ray& r) = 0;
}

// Mesh contains a list of vertex coordinates and faces
class Mesh: public HittableObject {
public:
    std::vector<Point3> vertices;
    std::vector<Face> faces;

    void addVertex(/*params*/);
    void addFace(/*params*/);

    CollisionData rayCollisionPoint(const Ray& r);
}
```

And a `Face` class that has a pointer to its mesh and 3 vertex indices:
```
class Face {
public:
    Mesh* mesh;
    std::vector<int> vertexIndices; // size: 3

    CollisionData rayCollisionPoint(const Ray& r)
}
```

### 2-1. Other sources

- PBRT defines a separate class for normals ([here](https://pbr-book.org/3ed-2018/Geometry_and_Transformations/Normals)), so it is not mixed up with other 3D vectors. However it brings in a lot of rewriting operators and conversion functions, so I let my normals as 3D vectors to keep things simple.

- Some other simpler form of raytracer, such as [Raytracing in One Weekend](https://raytracing.github.io/books/RayTracingInOneWeekend.html) or [Scratchapixel](https://www.scratchapixel.com/lessons/3d-basic-rendering/introduction-to-ray-tracing) uses implicit spheres only instead of meshes, so simplifying the problem of intersections and normals.

### 3. Ray-plane intersection

Ray-triangle intersection works in 2 steps:

1) **Ray-plane intersection**
2) Triangle inside-outside test

Additionally, raytracers usually use methods such as Bounding Volume Hierarchies (BVH) or k-Dimensional Trees (k-D Trees) to entirely avoid the collision when ray entirely avoids a region, or when there are other meshes closer to the mesh origin. I decided to leave most of the optimizations for after all necessary raytracing functions are implemented, so ray-triangle intersection test is done for all existing triangles in the scene.

#### 3-1. The plane equation

Line-plane intersection is taught in highschool math. A plane is characterized by 4 numbers A, B, C and D where

$$Ax+By+Cz+D=0$$

And the normal $n$ of this plane is known to be

$$n = \begin{pmatrix} A, B, C \end{pmatrix}$$

In vector notation, this can be written as

$$r \cdot n+\text{coeff}=0 \ \ \ \tag{1}$$

Where $\text{coeff}$ is a constant that can be obtained by substituting any point on the face into the equation (1).

#### 3-2. Substituting ray to the plane equation

Ray is expressed as a function $r(t)=o+d(t)$.
- $o$ is a 3D vector representing the coordinate of the ray origin
- $d$ is a 3D vector representing the direction of the ray
- $r(t)=(o+dt)$ represents the collision coordinate of the ray, where $t$ is the variable of interest that raytracing algorithms try to find.

Substituting $r=(o+dt)$ to (1), we get

$$(o + dt) \cdot n + \text{coeff} = 0$$

Reordering the equation using that dot products are commutative (in $\mathbb{R}$)/distributive and that scalar multiplications are commutative/distributive/associative over vectors,

$$o \cdot n + d \cdot n t + \text{coeff} = 0$$

All variables except t are known in the equation, so we separate out t, using that the evaluated dot products are scalars:

$$t = - \frac{o \cdot n + \text{coeff}}{d \cdot n}$$

Solves the Ray-plane intersection problem.

Note that positive t implies the plane is in front of the ray origin (valid intersection), where negative t implies the plane is behind the ray origin (invalid intersection). The intersection test returns false if t < 0, and proceeds to triangle inside-outside test otherwise.

#### 3-3. Implementation of ray-plane intersection

It is a common practice to save the normal within the face object. I also decided to precompute $\text{coeff}$ and save it as a member of the same object. This uses an extra floating point variable per face and saves a dot product operation on each collision testing.

On face initialization, I would precompute the normal and $\text{coeff}$:

```cpp
/* Precomputation */
// Read vertex coordinates from the mesh
Vec3 v0 = this->mesh->vertices[this->vertexIndices[0]];
Vec3 v1 = this->mesh->vertices[this->vertexIndices[1]];
Vec3 v2 = this->mesh->vertices[this->vertexIndices[2]];

// Computing normal: cross product of two edges
Vec3 e1 = v1 - v0;
Vec3 e2 = v2 - v1;
this->normal = unit_vector(cross(e1, e2));

// Computing coeff
this->coeff = dot(this->normal, v0);
```

And on each collision testing, I would use the precomputed variables to compute t:
```cpp
float t = (-1.f) * (dot(this->normal, r.origin()) + this->coeff) / (dot(this->normal, r.direction()));
if (t < 0.f) {
    return {false}; // Return CollisionData initialized with parameter 'false'
}
// Do triangle inside-outside test
```


