---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: home
---

# Top-level rendering technique / rendering algorithms (not sure of the correct term)

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
{% include 3dged p=15 %}

## Multi-fragments
Using bucket sort (and adaptive bucket depth peeling): {% include 3dged p=1 %}

# Techniques

## Upsampling

### Geometry aware
Joint bilateral filter {% include 3dged p=56 %}

# Illumination and Shading

* Illumination: The luminous flux transport from light sources to scene points, via direct and indirect paths.
* Shading: process of assigning a color to a a pixel.

## Light sources

### Geometry
* Directional (representation in deferred: full-screen quads)
* Point (representation in deferred: sphere)
* Spot (representation in deferred: cone)

* Area (not supported by usual rendering technique)

# Glossary

* `G-buffer` (Geometric buffer): A GPU buffer used to store geometric information of thescene. Used with deferred rendering. [See](#g-buffer).

