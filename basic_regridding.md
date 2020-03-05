# Basic regridding

Analytical fluid dynamics problems are typically translated into a partial differential equation that has to be solved for some boundary conditions. This introduces the difficulty of having to deal with the geometrical shape of the boundary.

Computational fluid dynamics methods add one small level of complexity on top of this: we need to define an integration grid, including both the boundaries and the points in space were we want our simulation to be accurate.

Our main question here is just how to refine a given grid in OpenFOAM:

## TL;DR
1. Open `./system/blockMeshDict`
2. Increase the mesh resolution inside `blocks` (see below)
3. Execute `blockMesh`
4. Execute `setFields` to update the initial conditions

```c++
blocks
(
    hex (0 1 2 3 4 5 6 7) (1000 1 1) simpleGrading (1 1 1)
    \\ hex, shape, hexaedron in this case
    \\ (0 1 ...), ordered list of vertices (defined apart)
    \\ (1000 1 1) resolution in x, y and z
);
```

## Basic example: square pipe

Our domain is a pipe of square section 1x1, running from -5 to 5 in the x axis.

The information about the grid is stored in `./system/blockMeshDict`. The grid itself is defined under the values `vertices` and `blocks`.

### Vertices

The field vertices contains the "corners" of our domain. Each of them is identified by its order position in the list, starting at 0.

```c++
vertices
(
    (-5 -1 -1) // Vertex 0
    (5 -1 -1) // ...
    (5 1 -1)
    (-5 1 -1)
    (-5 -1 1)
    (5 -1 1)
    (5 1 1)
    (-5 1 1) // Vertex 7
);
```
### Blocks

The field blocks turns a set of vertices into a volumetric grid. Their order is important, and a bit tricky (check [4.1.3](https://www.openfoam.com/documentation/user-guide/mesh-description.php#x11-300004.1.1))

```c++
blocks
(
    hex (0 1 2 3 4 5 6 7) (1000 1 1) simpleGrading (1 1 1)
    // See TL;DR for a step by step description of this file
);
```

### blockMesh

The command `blockMesh` reads `blockMeshDict` and generates a new mesh. The mesh can be inspected with `paraview`.

### setFields

The command `setFields` initializes the initial conditions to match the mesh and the specifications of `setFieldsDict`.

## Practical example: a domain with two grids

Now we want two different grid densities for the square section pipe we mentioned before.

We achieve this by adding more vertices to the center of the pipe:

```c++
vertices
(
    (-5 -1 -1) // Vertex 0
    (0 -1 -1)
    (0 1 -1)
    (-5 1 -1)
    (-5 -1 1)
    (0 -1 1)
    (0 1 1)
    (-5 1 1)
    (0 -1 -1)
    (5 -1 -1)
    (5 1 -1)
    (0 1 -1)
    (0 -1 1)
    (5 -1 1)
    (5 1 1)
    (0 1 1) // Vertex 15
);

```

With the vertices just defined, we can create two blogs differing in density:

```c++
blocks
(
    hex (0 1 2 3 4 5 6 7) (25 1 1) simpleGrading (1 1 1) // Coarse grid
    hex (8 9 10 11 12 13 14 15) (100 1 1) simpleGrading (1 1 1) // Fine grid
);
```

The result looks like:

![](img/fine-coarse.png)
