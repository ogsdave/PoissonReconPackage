# PoissonReconPackage

This project is a demo to show how to use PoissonRecon and SurfaecTrimmer in your own project, with STL vector of float*/int* as input and output.

## Quick run
Here I add a CallPoisson program. Run `make` to build all projects.

To use PoissonRecon, you can use commands below in terminal:
```
Bin/Linux/CallRecon --in Data/sphere_points_cut.xyz --out Data/test.obj --depth 10 --density --trim 3.5
```
And you will see Data/test.obj as generated file. By changing trim value from 0 to 4, you can see different result. Value 0 will be the results of originally poisson surface reconstruction. While value 3.5 will cut come low density faces.

![Results for density value 0](/Img/result_0.png)
![Results for density value 3.5](/Img/result_3.5.png)

---

## How to use in your own project
The usage of PoissonRecon and SurfaceTrimmer is coded in Src/CallRecon.cpp. 

In this file I include Ply.h, PoissonRecon.h and SurfaceTrimmer.h, then construct input points and normals to two `std::vector<float*>`. Use code below to call Poisson surface reconstruction, and result will be within `mesh`.
```
Execute< 2 , PlyValueVertex< Real > , true  >( pts, norms, mesh );
```

Then you can see how to retrieve the vertices and faces from generated mesh with code below.
```
// retrieve vertices, density value (optional) and faces from mesh
{
  mesh.resetIterator();
  // get vertices
  for (int i = 0; i < mesh.inCorePoints.size(); ++i) {
    float *v = new float[4];
    v[0] = mesh.inCorePoints[i].point.coords[0]; // x
    v[1] = mesh.inCorePoints[i].point.coords[1]; // y
    v[2] = mesh.inCorePoints[i].point.coords[2]; // z
    v[3] = mesh.inCorePoints[i].value; // density value
    vts.push_back(v);
  }
  for (int i = 0; i < mesh.outOfCorePointCount(); ++i) {
    float *v = new float[4];
    PlyValueVertex< Real > vt;
    mesh.nextOutOfCorePoint(vt);
    v[0] = vt.point.coords[0];
    v[1] = vt.point.coords[1];
    v[2] = vt.point.coords[2];
    v[3] = vt.value;
    vts.push_back(v);
  }
  // get faces
  for (int i = 0; i < mesh.polygonCount(); ++i) {
    int *f = new int[3];
    vector<CoredVertexIndex> face;
    mesh.nextPolygon(face);
    for (int j = 0; j < face.size(); ++j) {
      if (face[j].inCore) {
        f[j] = face[j].idx;
      }
      else {
        f[j] = face[j].idx + (int)(mesh.inCorePoints.size());
      }
    }
    faces.push_back(f);
  }
}
```

After retrieveing vertices and faces into `std::vector<float*>` and `std::vector<int*>`, use code below to call surface trimmer.
```
// call SurfaceTrimmer
CallSurfaceTrimmer(arg.trimVal, vts, faces, tVts, tFaces);
```

Note that the each vertex has for value (x, y, z and density value), density value are used for surface trimmer. If you donot need surface trimmer, you may also use only three value to represent a vertex.

In CallRecon.cpp, I have four basic functions `simpleCmdParse`, `argValidate`, `readPoints` and `writeModel`, you can replace them 

---

## What did I exactly do

See this [commit](https://github.com/xiasun/PoissonReconPackage/commit/1497259cb8fa8057d55a031328ce9b931fac468a) to know what I exactly do.
I add a new setOctree method in MultiGridOctreeData to handle input as `vector<float*>`, then create PoissonRecon.h and SurfaceTrimmer.h to migrate their original implementation into headers that could be used by other program.

---

## Sample Data
I have four sample data provided.

![sphere_mesh_original.obj: the model of a sphere](/Img/input_model.png)
![sphere_mesh_cut.obj: the model of a cutted sphere](/Img/input_model_cut.png)
![sphere_points_original.xyz: the pointset of a sphere (normals included)](/Img/input_points.png)
![sphere_points_cut.xyz: the pointset of a cutted sphere (normals included)](/Img/input_points_cut.png)

---

## Conclusion
This project is based on [Screened Poisson Surface Reconstruction Version 5](http://www.cs.jhu.edu/~misha/Code/PoissonRecon/Version5/).
The latest code is released on [this repository](https://github.com/mkazhdan/PoissonRecon) and [this page](http://www.cs.jhu.edu/~misha/Code/PoissonRecon/). Many thanks to Professor [Michael Kazhdan](https://github.com/mkazhdan) and all contributors for introducing all these amazing algorithms and codes.
