---
title: FocG
---

# Fundamentals of Computer Graphics

## Chapter 10 - Surface Shading

### Diffuse shading

Lambertian + Ambient

`Lambertian`: said of an object where shading does not depend of the viewpoint. It obeys _Lambert's consine law_.

$$c = c_r * (c_a + c_l * \max(0, n \cdot l))$$

* $$c_l$$: light color
* $$c_a$$: ambient color
* $$c_r$$: diffuse reflectance of the surface

### Phong shading

Adding the highlights:

$$c = c_l * \max(0, e \cdot r)^p$$

- $$e$$: view direction
- $$r$$: light reflection direction (accross the normal n)
- $$p$$: Phong exponent

or

$$c = c_l * (h \cdot n)^p$$

* $$h$$: halfway vector (between $$l$$ and $$e$$) = $$\frac{e + l}{\lVert e + l \rVert}$$


## Chapter 11 - Texture Mapping

`Texture coordinate function`: assigne texture coordinate to every point on a surface. (alternative names: `UV mapping`, `surface parameterization`).

Two main approaches:

* Compute the texture coordinantes geometrically from the spatial coordinates of the surface point.
* For meshe surfaces, store values of the textures coordinates at vertices, and interpolate them accross the surface.

### Geomatrically determined coordinates

`cubemap`: Collection of 6 square texture, accessed by projecting onto a cube.
