---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

title: Main Notes
---
# Top-level rendering technique / rendering algorithms (TODO not sure of the correct term, ourmachinary.com calls them "high-level pipeline algorithms")
## Forward rendering
 Rasterize each geometric object in the scene, and during each fragment shading iterate each light in the scene. (description ignoring optimizations).

Usually two passes:
* Opaque pass
* Transparent path (sorted front to back in view space)

All lighting is performed in the fragment shader.

## Deferred shading
Geometry pass rasterizes each geometric object into 2D image buffers to store geometric information, required to perform lighting calculation in a later Lighting pass.
Lighting pass draw a geometric representation of the light volume, to find lit pixels from the `G-buffer`.

### G-buffer

The `G-buffer` (Geometric Buffer) usually contains:
* Screen-space depth
* Surface normals
* Diffuse color
* Specular power (and specular color)

### Method
Advantage: light calculation is only computed once per light per fragment.
Drawbacks:
* Cannot handle transparency (only 1 geometric "structure" per fragment). Usually transparency is rendered in a later forward pass.
* Can only simulate a single lighting model (not sure why?)

[Possible implementation (DX)](https://www.3dgep.com/forward-plus/#Deferred_Shading)


## Forward+ (or Tiled forward shading)

A first light culling pass into a uniform grid of tiles in screen space -> partition of lights per-tile.

Second pass is standard forward rendering, but fragments only iterate over lights in their tile.

[Overview](https://www.3dgep.com/forward-plus/#Forward)

## Light pre-pass
{% include g3ded p=15 %}

## Multi-fragments
Using bucket sort (and adaptive bucket depth peeling): {% include g3ded p=1 %}

# Ray Tracing (RT)

* `Ray-casting`:
  Non recursive RT, a ray is traced from the eye to the closest object. This is a backward simulation. Then shading takes place using traditional 3D CG shading models (e.g Phong). Essentially, rays are traced to solve visibility. Advantage over rasterization is ability to deal with non-planar surfaces (any surface for which we can compute a ray intersection).

* `Whitted ray-tracing` (Turner Whitted):
  Recursive RT, naturally handling reflections, refractions, shadows. This algorithm extended RT from a visibility only matter, to a matter of light transport.

* `Path tracing`:
  Monte carlo method which models global illumination. It inregrates over all illuminance arriving to a surface point, and reduce it via BRDF to determine how much goes toward the camera.

# Techniques

## Upsampling

### Geometry aware
* Joint bilateral filter {% include g3ded p=56 %}
* [Neural Graphics](#neural_graphics)

##  Level of detail (LOD)
Brancheless geometry shader: {% include g3ded p=81 %}

## Neural Graphics
The use of deep learning and neural networks for real-time computer graphics.

### DLSS (Deep Learning Super Sampling) (Nvidia tech)
* DLSS Super Resolution: takes in lower res and motion/depth buffers, to upscale an image.
* DLSS Frame generation: insert new frames at full resolution in between "normal frames" to augment framerate.


# Illumination and Shading

* Illumination: ?The luminous flux transport from light sources to scene points, via direct and indirect paths?
* Illumination model: Given the illumination incident at a point on a surface, quantifies the reflected light.
* Shading: Process of assigning a color to a fragment.

([see slides making the distinction illumination model/shading](https://www.cs.brandeis.edu/~cs155/Lecture_16.pdf))

## Light sources

### Geometry
* Directional (representation in deferred: full-screen quads)
* Point (representation in deferred: sphere)
* Spot (representation in deferred: cone)

* Area (not supported by usual rendering technique)

## Material properties

**Photon-material interactions**:
* absorption
* reflection
* transmission.

Materials that do not transmit light are `opaque`.

`Specular reflection` follows the `Law of reflection`: $$\theta_{i(ncident)} = \theta_{r(eflected)}. There is no scattering.

`Diffuse reflection`: Reflections of radation that undergo scattering.

`Transparency`: transmission without appreciable scattering of photons. i.e. It follows the Law of refraction (Snell Descartes law):
  $$sin(\theta_1)/sin(\theta_2) = n_2/n_1$$

`Translucency`: transmission **with** scattering.


## Illumination models

### Classical

* Ambient Illumination: $$ I_a = K_a \times I_a $$
* Diffuse (Lambertian) reflection: $$ I_d = K_d I_{light} cos(\theta) = K_d I_{light} (N \cdot L) $$
* Specular reflection, Phong illumination model: $$ I_s = K_s I_{light} (R \cdot V)^n $$

### (TODO Are those considered illuminations models?)

* `The Fresnel equations` (or `Fresnel coefficients)`: describe the reflection and transmission of light (or electromagnetic radiation in general) when incident on an interface between different optical media.
* `The rendering equation` (Jim Kajia)


## Shading

* Flat shading: Single intensity for each polygon.
* Gouraud shading: Apply an illumination model at each vertex, linearly interpolates intensity values accross the surface.
* Phong shading: Interpolate the normal vectors accross the surface, then apply an illumination model at each fragment.

# Effects

## Texture mapping
Pixel derivatives:
* Hardware computation approaches analyse in {% include g3ded s=4.3 p=56 %}

## Reduced resolution effects
* Bloom
* Depth of field (DOF)
* Motion blur
* Soft particles
* Screen space ambient occlusion (SSAO)


# Animation
[Skinned instancing, dudash 07](https://developer.download.nvidia.com/SDK/10.5/direct3d/Source/SkinnedInstancing/doc/SkinnedInstancingWhitePaper.pdf)

# Visibility

## View frustum culling (VFC)
Intended to cull geometry which lies entirely outside of the view frustum.

* Radar VFC: {% include g3ded p=77 s=5.3 %}

## Occlusion culling
Approaches:
* Potentially Visible Set (PVS)
* Portals (visibility through openings) and Antiportals (occlusion behind objects)
* Hardware occlusion queries (OQs): {% include g3ded p=36 %}
* Dynamic (based on HZB, bounded) visibility: {% include g3ded ch=3 p=35 %}

## Contribution culling
Culling geometry when covering a small area (under a covera threashold): {% include g3ded p=48 %}

## Fragment culling
* Early Z test
  * More efficient if the scene was sorted front to back (limit overdraw)

* Z3 (useful for forward rendering): {% include g3ded ch=6 p=91 %}

## Light culling
* Frustum culling on the light volumes (for lights of known limited range).


# Data structures

## Hierarchies

* Bounding volume hierarchy (BVH) **TODO**
* Octree **TODO**

# Equations / Bases

* Computing normals from heightmap: Sobel filter (mention in {% include g3ded p=191 %})

* Computing view-space Z from depth buffer and near/far planes: {% include g3ded p=26 eq=2.4 %}
* Computing View-space X Y from screen-space X Y and view projection matrix: {% include g3ded p=26 eq=2.5 %}

## Representation

### Sphere
- $$c$$: center point
- $$r$$: radius

### Plane
- $$n$$ is the plane normal (can be computed by 3 points in the plane and a cross product).
- $$d$$ , distance from the origin to the plane with P in the plane.

**constant-normal form** is $$d = n \cdot P$$, which is equivalent to the implicit equation $$ax + by + cz - d = 0$$ (with $$n(a,b,c)$$ and $$P(x, y, z)$$).

### Cone
- $$T$$: tip
- $$h$$: height
- $$d$$: direction vector
- $$r$$: base radius

### Tangent-space (TBN)
Representation via quaternions: {% include g3ded s=7.5 p=108 %}

## Culling equations

* Frustum-sphere: https://www.3dgep.com/forward-plus/#frustum-sphere-culling
* Frustum-cone: https://www.3dgep.com/forward-plus/#frustum-cone-culling

# Conventions

## Row/Column major
`Row` (resp `Column`) major order means the consecutive elements of a row (resp. column) are contiguous in memory.

Usually, `column-major` matrices pre-multiply vectors, whereas `row-major` post-multiply.


# Glossary

* `attenuation`: Fall-off of the intensity of a light.
* `BVH` (Bounding volume hierarchy).
* `DLSS` (Deep Learning Super Sampling).
* `DXR` (DirectX Ray tracing).
* `G-buffer` (Geometric buffer): A GPU buffer used to store geometric information of thescene. Used with deferred rendering. [See](#g-buffer).
* `HZB` (Hierarchical Z-buffer).
* `LOD` (Level of detail).
* `pipeline state`: A specific configuration of the rendering pipeline. Once applied, it does control how objects are rendered.
  * Program (i.e. active shaders)
  * Rasterizer state
    * polygon fill mode
    * culling mode,
    * scissor culling
    *  viewports
  * Blend state
  * Depth/Stencil state
  * Render target
* `RT` (Ray Tracing).
* `stereo calibration`: Find corresponding points in two cameras.
* `TBN` (Tangent Bitangent Normal).
* `technique`: A combination of passes executed in a particular order to implement a rendering algorithm.
* `VFC` (View frustum culling).
