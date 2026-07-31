# Igloo Terrain

A virtual reality scene set in an Arctic environment, featuring an igloo, procedurally generated snow-covered terrain, water, clouds, and snowfall.

The terrain is generated using **Perlin noise**, **Fractal Brownian Motion**, and a configurable **falloff function**.

<img width="1920" height="1080" alt="project" src="https://github.com/user-attachments/assets/127634cb-066b-42b2-922f-25bfb9f69387" />

## Features

* Procedurally generated terrain
* Custom two-dimensional Perlin noise implementation
* Fractal Brownian Motion for detailed terrain variation
* Falloff-based terrain boundaries
* Height-based vertex coloring
* Custom water and skybox shaders
* Snow generated using Unity's Particle System
* Virtual reality support
* Manually placed environmental models

## User Documentation

### Requirements

The scene is designed for virtual reality and may require relatively powerful hardware.

The recommended setup is:

* A VR-ready Windows PC
* A PC-compatible VR headset
* A graphics card capable of rendering the terrain, shaders, and particle effects smoothly

PC VR is recommended for the best experience and a VR headset is required for full experience. 

### Running the Application

1. Download `Igloo.zip`.
2. Extract the archive.
3. Connect and configure your VR headset.
4. Run `TheOneHopefully.exe`.

## Developer Documentation

The project was created using the **Unity VR Template** in **Unity 2022.3.62f1**.

### Technologies

* Unity
* C#
* Unity VR Template
* Shader Graph
* Unity Particle System
* Procedural mesh generation

## Technical Overview

The terrain-generation system consists of three main scripts:

| Script                | Purpose                                                                |
| --------------------- | ---------------------------------------------------------------------- |
| `PerlinNoise.cs`      | Generates smooth two-dimensional Perlin noise values.                  |
| `CalculateFallOff.cs` | Calculates terrain falloff values near the terrain boundaries.         |
| `MeshGenerator.cs`    | Generates the terrain mesh, vertices, triangles, colors, and collider. |

---

## `PerlinNoise.cs`

`PerlinNoise.cs` implements a two-dimensional Perlin noise algorithm for procedural terrain and texture generation.

Perlin noise produces smooth and natural-looking pseudorandom values. It is commonly used for generating terrain, clouds, textures, and other organic patterns.

### Static Fields

#### `private static int[] permutationTable`

A permutation table containing 256 values.

The table:

* Serves as the basis of the hashing function
* Determines the gradient arrangement used during noise generation
* Can be replaced with another permutation table to generate a different noise pattern

A single permutation table is sufficient when generating one deterministic terrain layout.

#### `private static int[] p`

A duplicated version of the permutation table containing 512 values.

Duplicating the table:

* Simplifies index wrapping
* Prevents indices from exceeding the table bounds
* Avoids repeatedly applying modulo operations when accessing gradient hashes

### Static Constructor

#### `static PerlinNoise()`

The static constructor:

1. Allocates the `p` array with a length of 512.
2. Copies the 256-element permutation table into both halves of the array.

### Private Methods

#### `private static float Fade(float t)`

Applies a smoothing curve to the interpolation factor.

This function creates smooth transitions between neighboring grid values and reduces visible block-like artifacts.

<img width="534" height="524" alt="image" src="https://github.com/user-attachments/assets/c363e532-1ccf-4b40-bb67-bd4437593675" />

**Parameters**

* `t` — Input value in the range `[0, 1]`

**Returns**

A smoothed value in the range `[0, 1]`.

#### `private static float Lerp(float t, float a, float b)`

Performs linear interpolation between two values.

**Parameters**

* `t` — Interpolation factor in the range `[0, 1]`
* `a` — Starting value
* `b` — Ending value

**Returns**

A value interpolated between `a` and `b`.

#### `private static float Gradient(int hash, float x, float y)`

Calculates the dot product between a gradient vector and a distance vector.

The gradient vector can point in one of eight possible directions.

<img width="253" height="201" alt="image-2" src="https://github.com/user-attachments/assets/f6398d6c-20de-4dcd-aa6e-43eba7008ce9" />

**Parameters**

* `hash` — Hash value used to select the gradient direction
* `x` — X component of the distance from a grid corner
* `y` — Y component of the distance from a grid corner

**Returns**

The dot product of the selected gradient vector and the distance vector.

### Public Methods

#### `public static float Noise(float x, float y)`

Calculates the Perlin noise value at a given point.

**Parameters**

* `x` — X coordinate in noise space
* `y` — Y coordinate in noise space

**Returns**

A normalized noise value in the range `[0, 1]`.

**Process**

1. Determine the grid cell containing the input point.
2. Calculate the point's relative position inside the cell.
3. Calculate smoothed interpolation weights for both axes using `Fade`.
4. Retrieve the hashes for the four corners of the cell.
5. Calculate the gradient dot product at each corner.
6. Interpolate the corner values along the X and Y axes.
7. Normalize the result to the range `[0, 1]`.

The final normalization is not required for every Perlin noise application, but it is useful when generating a height map.

---

## `CalculateFallOff.cs`

`CalculateFallOff` is a static class used to calculate falloff values for islands and terrains with defined boundaries.

It creates a smooth transition from the center of the terrain toward its edges.

The falloff can be used to:

* Create an island-like terrain shape
* Reduce terrain height near the boundaries
* Prevent abrupt edges in the generated mesh

### Public Methods

#### `public static float EvaluateFalloff(int x, int z, int size_x, int size_z, float flatness, float steepness)`

Calculates the falloff value for a terrain position.

**Parameters**

* `x` — X coordinate of the terrain position
* `z` — Z coordinate of the terrain position
* `size_x` — Terrain width
* `size_z` — Terrain length
* `flatness` — Size of the flatter area near the terrain center
* `steepness` — Steepness of the transition toward the edges

**Returns**

A value in the range `[0, 1]`.

* `0` represents no falloff
* `1` represents maximum falloff

**Process**

1. Normalize the terrain coordinates to the range `[-1, 1]`.
2. Calculate the position's distance from the center of the terrain.
3. Use the largest distance component to determine the falloff.
4. Apply a sigmoid-like function controlled by `flatness` and `steepness`.

---

## `MeshGenerator.cs`

`MeshGenerator.cs` procedurally generates terrain using Perlin noise and Fractal Brownian Motion.

The script creates a configurable mesh with:

* Adjustable width and length
* A procedural height profile
* Boundary falloff
* Height-based vertex colors
* Collision support

### Dependencies

The script depends on the following Unity components:

* `MeshFilter` — Stores the generated mesh
* `MeshCollider` — Provides collision for the generated terrain

It also uses the following custom classes:

* `PerlinNoise`
* `CalculateFallOff`

### Public Parameters

The following values are example values used by the current implementation. They can be adjusted in the Unity Inspector.

### Terrain Size

| Field        | Example value | Description    |
| ------------ | ------------: | -------------- |
| `int size_x` |         `255` | Terrain width  |
| `int size_z` |         `255` | Terrain length |

### Fractal Brownian Motion

| Field             | Example value | Description                           |
| ----------------- | ------------: | ------------------------------------- |
| `float amplitude` |        `5.0f` | Initial strength of each noise octave |
| `float frequency` |        `0.3f` | Initial noise sampling frequency      |
| `int numOctaves`  |           `4` | Number of combined noise layers       |

### Scaling and Transformation

| Field               | Example value | Description                                          |
| ------------------- | ------------: | ---------------------------------------------------- |
| `float noiseScale`  |        `0.1f` | Scales the input coordinates used for noise sampling |
| `float heightScale` |       `10.0f` | Scales the generated terrain height                  |

### Falloff Parameters

| Field             | Example value | Description                                   |
| ----------------- | ------------: | --------------------------------------------- |
| `float flatness`  |        `1.0f` | Controls the size of the flatter central area |
| `float steepness` |        `2.0f` | Controls the steepness of the falloff curve   |

### Visual Configuration

| Field               | Description                                            |
| ------------------- | ------------------------------------------------------ |
| `Gradient gradient` | Maps normalized terrain height values to vertex colors |

### Private Fields

| Field                       | Description                                                                     |
| --------------------------- | ------------------------------------------------------------------------------- |
| `Mesh mesh`                 | Reference to the generated mesh                                                 |
| `MeshCollider meshCollider` | Reference to the terrain collider                                               |
| `Vector3[] vertices`        | Generated mesh vertices                                                         |
| `Color[] colors`            | Generated vertex colors                                                         |
| `int[] triangles`           | Triangle index buffer                                                           |
| `float minTerrainHeight`    | Lowest generated terrain height                                                 |
| `float maxTerrainHeight`    | Highest generated terrain height                                                |
| `float maxPossible`         | Maximum height that can theoretically be produced by the configured FBM octaves |

### Methods

#### `float FractalBrownianMotion(float x, float y, int numOctaves)`

Combines multiple layers of Perlin noise to generate more detailed terrain.

Each noise layer is referred to as an octave.

**Parameters**

* `x` — X coordinate used for noise sampling
* `y` — Y coordinate used for noise sampling
* `numOctaves` — Number of noise layers to combine

**Returns**

A value representing the generated height at the specified point.

**Process**

1. Initialize the accumulated result, amplitude, and frequency.
2. For each octave:

   * Sample Perlin noise at the current frequency.
   * Multiply the sampled value by the current amplitude.
   * Add the result to the accumulated height.
   * Reduce the amplitude by half.
   * Double the frequency.
3. Return the accumulated result.

Using several octaves combines large terrain shapes with smaller surface details.

#### `void Start()`

Unity lifecycle method called when the object is initialized.

**Process**

1. Create a new `Mesh` object.
2. Assign the mesh to the object's `MeshFilter`.
3. Retrieve the `MeshCollider` component.
4. Call `CreateShape()` to generate the mesh data.
5. Call `UpdateMesh()` to apply the generated data.

#### `void CreateShape()`

Generates the geometry and vertex colors of the terrain.

##### Vertex Generation

1. Allocate space for `(size_x + 1) * (size_z + 1)` vertices.
2. Iterate over every terrain position.
3. Calculate the height using Fractal Brownian Motion.
4. Calculate and apply the falloff value.
5. Store the resulting position in the `vertices` array.
6. Track the minimum and maximum generated heights.

##### Triangle Generation

1. Allocate `size_x * size_z * 6` triangle indices.
2. Treat each terrain grid cell as a quad.
3. Split each quad into two triangles.
4. Store the six required vertex indices in the `triangles` array.

##### Color Generation

1. Allocate one color for every vertex.
2. Normalize each vertex height to the range `[0, 1]` using `Mathf.InverseLerp`.
3. Evaluate the configured gradient using the normalized height.
4. Assign the resulting color to the vertex.
5. Apply blue explicitly to vertices whose height is equal to zero.

#### `void UpdateMesh()`

Applies the generated data to the Unity mesh and updates its collider.

**Process**

1. Clear any existing mesh data.
2. Assign the generated vertices, triangles, and colors.
3. Recalculate normals for correct lighting.
4. Recalculate bounds for visibility culling and rendering.
5. Assign the updated mesh to the `MeshCollider`.

---

## Water

Water is implemented using a Shader Graph named `Water`.

The generated shader is used by a material applied to a Unity Plane mesh.

<img width="1864" height="1412" alt="image-12" src="https://github.com/user-attachments/assets/0a98b1ae-9119-4657-bbe3-ea5462aadc3f" />

---

## Skybox

The skybox is implemented using the `SkyBoxCloud` Shader Graph.

It uses the following Shader Graph subgraphs:

* `SkySubGraph`
* `CloudSubGraph`

<img width="1723" height="1115" alt="image-6" src="https://github.com/user-attachments/assets/e38768d4-acf6-4a04-b72b-81f8eb4fd43a" />

### `SkySubGraph`

Generates the night-sky portion of the skybox, including its star field.

<img width="2185" height="1081" alt="image-4" src="https://github.com/user-attachments/assets/38b2ccdc-d412-4874-bac5-da7c01f54dc1" />

### `CloudSubGraph`

Generates the cloud texture used by the skybox shader.

<img width="2177" height="1016" alt="image-7" src="https://github.com/user-attachments/assets/3dcaddba-3067-4a20-987e-fabe0ae7135b" />

---

## Snow

Snowfall is generated using Unity's Particle System.

The particle system is configured in the Unity Editor to distribute snow particles throughout the scene.

<img width="927" height="85" alt="image-8" src="https://github.com/user-attachments/assets/844a10f5-818e-4a46-af19-7ba3d151a31c" />

<img width="924" height="185" alt="image-9" src="https://github.com/user-attachments/assets/dfdc208f-0abb-47bb-9279-31788f740bcc" />

<img width="925" height="71" alt="image-10" src="https://github.com/user-attachments/assets/d08bc09a-04c5-4c1f-a6dc-2772a427f83d" />

<img width="922" height="52" alt="image-11" src="https://github.com/user-attachments/assets/1b57893d-bfd0-49a1-ad3c-694f810e27f9" />

---

## Trees and Igloo

The trees and igloo were placed manually in the scene.

The models were obtained from:

* [Sketchfab](https://sketchfab.com/)
* [Unity Asset Store](https://assetstore.unity.com/)
