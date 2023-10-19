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

### 3. Ray-plane intersection

Ray-triangle intersection works in 2 steps:

1) Ray-plane intersection
2) Triangle inside-outside test

#### 3-1. Ray-plane intersection

Line-plane intersection is taught in highschool math; 



