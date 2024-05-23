# Real-Time Rendering - 4th edition

## Effects and subset glossary

User controlled clipping planes via FS `discard`

_Deferred shading_ does _visibility_ and _shading_ in distinct passes. It was given rise by the
availability of _multiple render targets_ (MRT) p51

Deferred shading is categorized as:
* a type of _rendering pipeline_, and then as a class of _rendering method_ p51.
* a _technique_ p127

_Parallax_ is the impact the observer's viewpoint have on the observation of an object.
(And apparently, the change in what is observed as the viewpoint changes).

_CIE_: Commission Internationale d'Eclairage.

Smoothstep and quintic equations, p181

## Chapter 1 - Introduction p1

Some operators:
* _Binomial coefficient_ (i.e. "_k parmi n_") $\binom{n}{k}$ = $C^k_n = \frac{n!}{k! (n-k)!}$
* Unary _perp dot product_, noted $^\perp$, work on a vector to give another vector perpendicular to it.

## Chapter 2 - The Graphics Rendering Pipeline p11

_Shading_ is the operation of determining the effect of light on a material.
It involves computing a __shading equation__ at various points on the object.

### Geometry processing p14

Model space -[model matrix]> World space -[view transform]> View (eye, camera) space -[projection matrix]-> clipping space / homogeneous clip space (clip coordinates, which are homogeneous)

In the clipping space, there is the _canonical view volume_ (unit cube) to which
the _drawing primitives_ are clipped by the _clipping stage_.

**Note**: the projection transforms the _view volume_
(bounded by a rectangular box (orthographic) or frustum (perspective) in view space)
into the unit cube of clipping space.

After clipping, _perspective division_ places vertices into _normalized device coordinates_ $[-1, 1]^3$.

_normalized device coordinates_ (NDC) -[screen mapping]-> (x, y [screen coordinates], z)[window coordinates]


Optionals:
* Tesselation shader
* Geometry shader
* Stream output (_transform feedback_ in OpenGL)

### Rasterization p21

_primitive assembly_ (_triangle setup_): Setup differentials, edge equations,...

triangle traversal: find which pixels or samples are inside a triangle, and generate fragments with interpolated properties.\
A _fragment_ is the piece of a triangle partially or fully overlapping a pixel (p49)

##Pixel processing
Fragment shading
Merging (ROP): **raster operations** or **blend operations**. merge fragment shading output color(s) with data already present in the color buffer, handles depth buffer (z-buffer), stencil buffer. The **framebuffer** consists of all enabled buffers.

Ch3 The Graphics Processing Unit

Details of why a GPU is very efficient for graphics rendering, with details of the data-parallel architecture:
**warps** (nvidia) or **wavefronts** (amd) grouping predefined numbers of **threads** of with the same shader program. Resident warps are said to be **in flight**, defining **occupancy**. Branching might lead to **thread divergence**.

Programmable shader stage has 2 types on input: **uniform** and **varying**.
Flow control is divided between **static flow control** based on uniforms (no thread divergence) and **dynamic flow control**, which depends on varyings.

A detailed history of programmable shading and graphics API p37.

Geometry shader can be used to efficiently generate cascaded shadow maps

Stream output can be used to skin a model and then reuse transformed vertices

#Pixel Shader
Multiple Render Targets (MRT) allow pixel shaders to generates distinct values and saved them to distinct buffers (called Render Target). E.g color, depth, object ids, world space distance, normals

In the context of fragment shaders, a **quad** is a group of 2x2 adjacent pixels, processed together to give access to gradients / derivatives (e.g. texture mipmap level selection)
Gradients cannot be accessed in parts affected by dynamic flow control p51. (I do not understand why)

Render Target can only be written at the pixel's location. Dedicated buffer type allow write access to any location. Unordered Access View UAV (DX) / Shader Storabe Buffer Object SSBO (OGL).
Data race are mitigated via dedicated _atomic units_, but they might lead to stalls.

## Merging stage
Output merger (DX) / Per-sample operations (OGL)

## Compute shader

A form of GPU computing, in that it is a shader not locked  into a location of the graphics pipeline.
Compute shaders give explicit access to a thread ID. They are executed by _thread group_, all threads  in the group guaranteed tonrun concurrently, and share a small amount of memory.


## Chapter 6 - Texturing p169

_texture mapping_ is assigning to a point (usually in model space) its _textures coordinates_.\
Note: _texture coordinates_ (usually $[0,1]^2$) are distinct from _texture space location_ (in the case on an image, it could be the pixel position)

![](images/rtr/06-texture_pipeline.png)

_Wang tiles_: small set of square tiles with matching edges.

Texels have integer coordinates (do they mean indices?). In OpenGL, and since DX10, the center of a texel has coordinates (0.5, 0.5).
Origine is lower-left (OpenGL) or upoer-left (DX)

_dependant texture read_ mean fragment shader has to "compute" the texture coordinates instread of using the (u, v) passed from the VS verbatim. There is an older, more specific definition: a texture read whose coordinates are dependent upon another texture read.

_power-of-two_ (POT) textures are $2^m \times 2^n$ texels. Opposed to NPOT.

_detail textures_ can be overlaid on magnified textures (cf [Blending and filtering materials](#blending_and_filtering_materials))

_mipmap chain_: set of images constituting al levels of a mipmapped texture.
The continuous coordinate $d$, or _texture level of detail_, is the parameter  determining where to sample in the levels of a mipmap chain (along the mipmap pyramid axis). OpenGL calls it $\lambda$.

The goal is a pixel-to-texel of at least 1:1 (so sampling (pixels) is at least twice the max signal frequency (1/2texels)).\
$(u, v, d)$ triplet allows for _trilinear interpolation_.\
A _level of detail bias_ (LOD bias) can be added to $d$ before sampling.

A major limitation of mipmaps is overblurring, because the filtering is isotropic (but the pixel backprojection into texture space can be far from a square).

_anisotropic filtering_:
_Summed-Area Table_ (SAT) p186\
_Unconstrained Anisotropic Filtering_ (in modern GPUs) p188\
_Elliptical Weighted Area_ (EWA) is a hight qualitu software approach.

_cube maps_ p190

Texture compression p192
* _S3 texture compression_ (S3TC) chosen as a standard in DX under the name _DXTC_ or _BC_. De facto OpenGL standard.
* _Ericson Texture Compression_ (ETC) p194, used in OpenGL ES (and GL 4). One component variant _Ericson Alpha Compression_.
* Variants for normal compression p195
  * _Adaptative Scalable Texture Compression_ (ASTC) p196.\
    All variants have _data compression assymetry_, decoding can be orders of magnitude faster than encoding.

There are approaches to reduce banding (e.g. histogram renormalization) p196

### Procedural Texturing p198

Noise function sampled at successive power-of-two frequencies (_octaves_),
weighted and summed to obtain a _turbulence_ function.
(Note _wavelet noise_ improves on _Perlin noise_).

### Material mapping p201
The idea is when using Material Parametrization, to fetch parameter values from textures.
Further, the texture value could control the dynamic flow of a shader, selecting different shading equations.\
**Warning**: shading model inputs with a linear relationship to output color (e.g. albedo) can be filtered with standard techniques. This is not the case for non-linear (roughness, normals,...)

### Alpha-mapping p202

The alpha value can be used for alpha-blending (blend stage) or alpha-testing(discard)
Used for _decaling_, _cutouts_ (e.g. _cross-tree_, _billboarding_),
Alpha-test has to take special care wrt mipmapping p204

_Alpha-to-coverage_ translates transparency as a MSAA coverage mask p207\
**Attention**: linear interpolation treat RGBA as premultiplied (which is usually not the case). A workaround is to paint transparent pixels p208

_Appearance modeling_ (Ch 9 p367) assumes different scales of observations,
influencing modelling approaches:
* _Macroscale_ (large scale, usually several pixels), as triangles
* _Mesosclae_ (middle scale, about 1 pixel), as textures
* _Microscale_ (smaller than a pixel), via the BRDF (Chapter 9)
* _Nanoscale_ ($[1, 100]$ wavelengths in size, unusual in real-time), by using wave optics models (Chapter 9)

### Bump-mapping p208
A family of small-scale (meso-feature) detail representation, essentially modifying the shading normal at fragment lcoation.\
The frame of reference of the normal map is usuall surface space, with a TBN matrix to change from object to surface.

Types of bump mapping:
* Blinn's method p211, via _offset vector bump map_ (offset map) or _height map_
* Normal mapping, via a _normal map_ (difficult to filter)

### Parallax mapping p214

_Parallax mapping_ in general aims  to determine what is seen at a pixel using a heightfield. It can give better occlusion/shadow clues.

Approaches:
* _Parallax mapping_ is an euristic (and cheap) approach that offsets texture coordinates before sampling the  albedo.
* _Parallax occlusion mapping_ (POM) or _relief mapping_ use ray marching along the view vector to find a better intersection with the heightfield. This approach can be extended with self-shadowing.
* _Shell mapping_ p220 approach allows to render silouhette edges of objects (otherwise showing the underlying primitive straight edge).

### Textured lights p221

Projective textures (called _light attenuation textures_ in this context)
can be used to modulate the light intensity of lights
(limited to a cone or frustum for normal texture, or all directions for cubemap).\
They are then called _gobo_ or _cookie_ lights.


## Chapter 8 - Light and Color

Our perception of color is a _psychophysical_ phenomenon: a psychological perception of physical stimuli.
* _Radiometry_ is about measuring electromagnetic radiation.
* _Photometry_ deals with light values weighted by the sensitivity of the human eye.
* _Colorimetry_ deals with the relationship between spectral power distribution and the perception of colors.


### Radiometry p267
* _radiant flux_ $\Phi$ in watts (W): flow of radiant energy over time.
* _irradiance_ $E = d\Phi / dA$ in W/m².
* _radiant intensity_ $I = d\Phi / d\omega$ in W/sr, flux density per direction (solid angle)
* _radiance_ $L = d^2\Phi/ (dA d\omega)$ in W/(m² sr), measure electromagnetic radiation in a single ray: the density of radiant flux with respect to both area and solid angle. It is what sensors (eye, camera, ...) measure. The area is measured in a plane perpendicular to the ray ("projected area").

_radiance distribution_ describes all light travelling anywhere in space.
In lighting equation, radiance at point $\mathbf{x}$ along direction $\mathbf{d}$ often appears as $L_o(x, d)$ (going out) or $L_i(x,d)$ (entering). By convention $\mathbf{d}$ always points away from $\mathbf{x}$.

_spectral power distribution_ SPD is a plot showing how energy is distributed across wavelength.

All radiometric quantities have spectral distributions.

### Photometry

Each radometric quantity has an equivalent photometric quantity (the differences are the _CIE photometric curve_ used for conversion, and units of measurement)

* _luminous flux_ in lumen (lm)
* _illuminance_ in lux (lx)
* _luminous intensity_ in candela (cd)
* _luminance_ in nit = cd/m²

### Colorimetry p272

CIE's XYZ coordinates define a color's _chromaticity_ and _luminance_.
_chromaticity diagram_, e.g; CIE 1931 taken by projection on the X+Y+Z=1 plane,
 is the lobe with the curved outline is the visible spectrum, with the straight line at the bottom called the _purple line_. The _white point_ define the _achromatic_ stimulus.
More perceptually uniform diagrams are developped, such as CIE 1976 UCS (part of the CIELUV color space).
A triangle in the chromaticity diagram represent the _gamut_ of a color space, with vertices being the primaries.
Given a $(x, y)$ color point on the chromaticity diagram and drawing a line
with the white point give the _excitation purity_ (\~= _saturation_) as relative distance from the point to the edge,
and the _dominant wavelength (\~= _hue_) as the intersection with the edge.

_spectral reflectance curve_ describes how much of each wavelength a surface reflects.

### Scene to screen p281

A _display standard_, such as sRGB, seems to encompass both a color-space (a RGB gamut defined by primitive colors, and a whitepoint) and display encoding function/curve. Also white luminance level (cd/m²).

#### HDR

_Dynamic range_ is about luminance?

_Perceptual quantizer (PQ)_ and _hybrid log-gamma_ (HLG) are non-linear display encodings defines by Rec. 2100 display standard.

_tone mapping_ or _tone reproduction_ is the process converting scene radiance values to display radiance values .
The applied transform is called the _end-to-end transfer function_ or _scene-to-screen transform_.
It transform the _image state_ from _scene-referred_ to _display-referred_.
Imaging pipeline:
![](images/rtr/08-imaging_pipeline.png)

Tone mapping can be seen as an instance of _image reproduction_: its goal is to create a display-referred images reproducing at-best the perceptual impression of observing the original scene.

Image reproduction is challenging: luminance of scene exceeds display capabilities by orders of magnitudes, and saturation of some colors are far out the display gamut.
It is achieves by leveraging properties of the human visual system:
* _adaptation_: compensation for differences in absolute luminance.

_exposure_ in rendering is a linear scaling on the scene-referred image **before** tone reproduction transform.

_global tone mapping_ (scaling by exposure then applying tone reproduction transform) where the same mapping is applied to all pixels, vs _local tone mapping_ with different mappings based on surronding pixels.

_color grading_ manipulates image colors with the intention to make it look better in some sense. It is a form of _preferred image reproduction_.

## Chapter 9 - Physically Based Shading p293

TODO: clarify the list of physical interactions between light and matter, and their classification.

Light-matter interaction: the oscillating electrical field of light causes the electrical charges in matter to oscillate in turn.
Oscillating charges emit new light waves, redirecting some energy of the incoming light wave in new direction. This reaction is _scaterring_ (usually, same frequency).
An isolated molcelule scatters in all directions, with directional variation in intensity.

Particles smaller than a wavelength _scatter_ light with constructive interference.
Particles beyond the size of a wavelength do not scatter in phase, the scattering increasingly favors the forward direction and the wavelength dependency decrease.

In _homogeneous medium_, scattered waves interfere destructively in all direction except the original direction of propagation. The ratio of phase velocities defines the _index of refraction_ IOR $n$. Some media are _absorptive_, decreasing the wave applitude exponentially with distance. The rate is defined by the _attenuation index_ $\kappa$. Those two optical properties are often combined into a single complex number $n + i\kappa$, the _complex index of refraction_.

A planar surface separating different indices of refraction scatter light a specific way: _transmitted wave_ and _reflected wave_.

Transmitted wave is refracted at an angle $\theta_t$, following _Snell's law_:
$$
\sin(\theta_t) = \frac{n_1}{n_2} \sin(\theta_i)
$$

In _geometrical optics_ light is modeled as rays instead of waves, and surfaces irregularities ar much smaller or much larger than wavelength.

On the other hand, when irregularities are in the range 1-100 wavelengths, _diffractions_ occurs.

_microgeometry_ are surface irregularities too small to be individually renderer (i.e. smaller than a pixel).
For rendering, microgeometry is treated statistically by the _roughness_.

_subsurface scattering_ p305
* _local subsurface scattering_ can be used when the entry-exit distance is smaller than the shading scale.
This local model should separate the _specular term_ for surface reflection from _diffuse term_ for local subsurface scattering.
* _global subsurface scattering_ is needed when the distance is larger than the shading scale (the inter-samples distance).

Camera p307

Sensors meadure **irradiance** over their surface. Adding enclosure, aperture and lens combine effect to make the sensor _directionally specific_, so the system measure **radiance**.

#### BRDF p309

The ultimate goal of physically based rendering is to compute $L_i(\mathbf{c}, -\mathbf{v})$
for the set of view rays $\mathbf{v}$ entering the camera positioned at $\mathbf{c}$.

_participating media_ does affect the radiance of a ray via absorption or scattering.

Without participating media, given $\mathbf{p}$ is the intersection of the view ray with
the closest object surface:
$L_i(\mathbf{c}, -\mathbf{v}) = L_o(\mathbf{p}, \mathbf{v})$

The goal is now to calculate $L_o(\mathbf{p}, \mathbf{v})$.
We limit the discussion to local reflectance phenomena (no transparency, no global subsurface scattering), i.e. light received and redirected outward by the currently shaded point only:
* surface reflection
* local subsurface scattering

Local reflectance is quantified by the _bidirectional reflectance distribution function_ BRDF.
BRDF only depends on incoming light direction $\mathbf{l}$ and outgoing view direction $\mathbf{v}$,
and is noted $f(\mathbf{l}, \mathbf{v})$.
BRDF is often used to mean the _spatially varying BRDF_ SVBRDF (capturing the variation of the BRDF based on spatial location on the surface).

* A general BRDF at a given point has four parameter, elevations $\theta_i, \theta_o$, azymuths $\phi_i, \phi_o$.
* _Isotropic BRDF_ remains the same for a given azimuth difference between incoming and outgoing ray (i.e. not affected by a rotation of the surface around the normal).
It can be reduced to 3 parameters elevations $\theta_i, \theta_o$, azymuth difference $\phi$.

![](images/rtr/09-brdf_parameters.png)

BRDF varies based on wavelength.
For real-time rendering the BRDF returns a spectral distribution as an RGB triples.

_Reflectance equation_, with $\Omega$ the hemisphere above the surface centered on $\mathbf{n}$:
$$
L_o(\mathbf{p}, \mathbf{v}) =
\int_{\mathbf{l} \in \Omega}{f(\mathbf{l}, \mathbf{v}) L_i(\mathbf{p}, \mathbf{l}) (\mathbf{n} \cdot \mathbf{l}) d\mathbf{l}}
$$

BRDF physical properties:
* _Helmoltz reciprocity_ $f(\mathbf{l}, \mathbf{v}) = f(\mathbf{v}, \mathbf{l})$ (often violated by renderers). Can be used to assert a BRDF physical plausibility.
* _conservation of energy_ outgoing energy cannot exceed incoming energy.

_directional-hemispherical reflectance_ can measure to what degree a BRDF is energy conserving. It measures the amount of light,
coming from a single direction $\mathbf{l}$, reflected over all directions:
$$
R(\mathbf{l}) =
\int_{\mathbf{v} \in \Omega}
    {f(\mathbf{l}, \mathbf{v}) (\mathbf{n} \cdot \mathbf{v}) d\mathbf{v}}
$$

_hemispherical-direction reflectance_ measure the amount of light, coming from all directions of the hemisphere, that is reflected toward a given direction $\mathbf{v}`:

$$
R(\mathbf{v}) =
\int_{\mathbf{l} \in \Omega}
    {f(\mathbf{l}, \mathbf{v}) (\mathbf{n} \cdot \mathbf{l}) d\mathbf{l}}
$$

For a reciprocal BRDF, directional-hemispherical reflectance is equal to the hemispherical-directional reflectance. In this situation, _directional albedo_ is a blanket term for both, they can be used interchangeably.

**Note**: For a BRDF to be energy conserving, $R(\mathbf{l})$ is in the range $[0, 1]$ for any $\mathbf{l}$, but the BRDF does not have this restriction and can go above $1$.

A Lambertian BRDF has a constant value (independent of incoming direction $\mathbf{l}$).
So does the directional-hemispherical reflectance, whose constant value is then call _diffuse color_ $\mathbf{c}_{diff}$ or the _albedo_ $\rho$ (or in this chapter, _subsurface albedo_ $\rho_\text{ss})$:
$$
R(\mathbf{l}) = \pi f(\mathbf{l}, \mathbf{v}) = \rho_\text{ss}
$$
$$
f(\mathbf{l}, \mathbf{v}) = \frac{\rho_\text{ss}}{\pi}
$$

#### Illumination p315

$L_i(\mathbf{l})$ term in the reflectance equation is the incoming light from all directions.

_Global illumination_ calculates $L_i(\mathbf{l})$ by simulating how light propagates and reflects throughout the scene, by using the _rendering equation_

**Note**: The reflectance equation is a special case of the rendering equation.

_Local illumination_ algorithms uses the reflectance equation to compute shading locally at each seruface point,
and are given $L_i(\mathbf{l})$ as input which does not need to be computed.

We can define a light color $\mathbf{c}_{light}$ as the reflected radiance from a white Lambertian surface facing toward the light ($\mathbf{n} = \mathbf{l}$)

In the case of directional and punctual lights and local illumination (i.e. direct light contribution), each surface point $\mathbf{p}$ receive (at most) a single ray from each light source, along direction $\mathbf{l}_c$, simplifying the reflectance equation to:
$$
L_o(\mathbf{v}) =
\pi f(\mathbf{l}_c, \mathbf{v}) \mathbf{c}_{light} (\mathbf{n} \cdot \mathbf{l}_c)
$$

Clamping the dot product to zero to discard lights under the surface, the resulting contribution for $n$ lights is:
$$
L_o(\mathbf{v}) =
\pi \sum_{i=1}^{n} f(\mathbf{l}_{c_i}, \mathbf{v}) \mathbf{c}_{light_i} (\mathbf{n} \cdot \mathbf{l}_{c_i})^+
$$
**Note**: this ressembles equation (5.6) p109, with $\pi$ cancelled out by the $/\pi$ often appearing in the BRDFs.

#### On to a specific phenomenas: Fresnel reflectance p316

_Fresnel equations_ are complex and not presented in the chapter.

Light incident on flat surface splits into a reflected part and a refracted part.
The direction of reflection $\mathbf{r}_i$ forms the same angle $\theta_i$ with the surface normal $mathbf{n}$ as $mathbf{l}$:
$$
\mathbf{r}_i = 2 (\mathbf{n} \cdot \mathbf{l}) \mathbf{n} - \mathbf{l}
$$

_Fresnel reflectance_ $F$ is the amount of light reflected (as a fraction of incoming light, i.e. a reflectance).
It depends on $\theta_i, n_1, n_2$.

_External reflection_ when $n_1 < n_2$, _internal reflection_ when $n_1 > n_2$.

Characteristics of $F(\theta_i)$ for a given substance interface:
* At _normal incidence_ ($\theta_i = 0°$), the value is is a property of the substance, noted $F_0$ : the _normal-incidence Fresnel reflectance_.
  It can be seen as the charactistic specular color of this substance.
* As $\theta_i$ increases, $F(\theta_i)$ tend to increase, reaching a value of $1$ for all frequencies (white) at $\theta_i = 90°$. This increase in reflectance at glancing angles is often called the _Fresnel effect_.

An approximation of the complex equation for Fresnel reflectance is given by Schlick:
$$
F(\mathbf{n}, \mathbf{l}) \approx F_0 + (1 - F_0) (1 - (\mathbf{n} \cdot \mathbf{l})^+)^5
$$

Pointers to other approximations are given p320, or the option to use other powers than $5$.

A more general form of the Schlick approximation, giving control over the color at 90° $F_{90}$:
$$
F(\mathbf{n}, \mathbf{l}) \approx F_0 + (F_{90} - F_0) (1 - (\mathbf{n} \cdot \mathbf{l})^+)^{\frac{1}{p}}
$$

* Dielectrics (insulators) have low $F_0$, which makes the Fresnel effect especially visible. Their optical properties rarely varies over the visible spectrum.
* Metals have a high $F_0$ (usually above 0.5), which is colored. They immediately absorb any transmitted light: they do not exhibit subsurface scattering or transparency. So all the visible color comes from $F_0$.
* Semiconductors have $F_0$ in between, but are rarely needed to render. Usually, the range [0.2, 0.45] is avoided for practical realistic purposes.

Parameterizing Fresnel values p324 discusses parameters of PBR models.

From the observation that metals have no diffuse color,
and that dielectrics have a restricted set of possible $F_0$ values:
an often-used parameterization combines specular color $F_0$ and diffuse color $\rho_\text{ss}$,
parameters being an RGB surface color $\mathbf{c}_\text{surf}$ and a scalar _metalness_ $m$.
* If $m = 1$, $F_0$ is set to $\mathbf{c}_\text{surf}$, $\rho_{ss}$ is set to black.
* If $m = 0$, $F_0$ is set to a plausible dielectric value (e.g. 0.04, or another param),
$\rho_\text{ss}$ is set to $\mathbf{c}_\text{surf}$

Metalness has some drawbacks:
* cannot express some types of materials, such a conted dielectrics with tinter $F_0$.
* artifacts can occur on the boundaries between metal and dielectric.

Parameterization trick: since $F_0$ lower than 0.02 are very unusual,
low values can be reserved for a specular occlusion mask,
to suppress specular highlights in cavities.

##### Internal reflection p325

When $n_1 > n_2$, then $\theta_t > \theta_i$. So a _critical angle_ $\theta_c$ exists where no transmission occurs: all incoming light is reflected. This phenomenon is _total internal reflection_ occurring when $\theta_i > \theta_c$.

$F(\theta_i)$ curve for internal reflection is a "compressed" version of the curve for external reflection,  with the same $F_0$ and perfect reflectance reached at $\theta_c$ instead of 90°.

It can occur in dielectrics (not metals which absorb), which have real-valued refractive indices:
$$
\sin \theta_c = \frac{n_2}{n_1} = \frac{1 - \sqrt{F_0}}{1 + \sqrt{F_0}}
$$

#### Microgeometry p327

Models surface irregularities that are smaller than a pixel, but larger than 100 wavelengths.
Most surfaces have an _isotropic_ distribution of the microscale surface normals (i.e. rotationally symmetrical).

Effects of microgeometry on reflectance:
* multiple surface normals
* _Shadowing_: the occlusion of the light source by microscale surface details. -> hidden form $\mathbf{l}$.
* _Masking_: some facets hiding others from the point of view. -> hidden form $\mathbf{v}$
* _Interreflection_: light may undergo multiple boundes before reaching the eye.
** sublte in dielectrics, light is attenuated by the Fresnel reflectance at each bounce.
** source of any visible diffuse reflection in metals (I do not understand why the distribution of normals is not also a source of diffuse reflection?), since metals do no exhibit subsurface scattering. They are more deeply colored than primary reflection, since they result from light interacting multiple times with the surface.

For all surfaces types, visible size of irregularities decresses as angle to the normal $\theta_i$ increases. This combines with the Fresnel effect to make surfaces appear highly reflective at glancing angles (lighting and viewing).


#### Microfacet theory p 331

_Microfacet theory_ is a mathematical analysis of the effects of microgeometry on reflectance,
on which many BRDF models are based.
The theory is based on the modeling of microgeometry as a collection of _microfacets_:
* each is flat, with a single normal $\mathbf{m}$.
* each individually reflectl ight according to the micro-BRDF $f_\mu(\mathbf{l}, \mathbf{v}, \mathbf{m})$. Their combined reflectance add up to the overall surface BRDF.

The model has to define the statistical distribution of microfacet normals:
* $D(\mathbf{m})$ is the _normal distribution function_ NDF (or _distribution of normals_).
  It tells us how many of the microfacets have normals pointing in certain directions.
* $G_1(\mathbf{m}, \mathbf{v})$ is the _masking function_,
the fraction of microfacets with normal $\mathbf{m}$ that are visible along view vector $\mathbf{v}$. (note: does not address shadowing).
* The product $G_1(\mathbf{m}, \mathbf{v})D(\mathbf{m})$ is the _distribution of visible normals_.

The projections of the microsurface and macrosurface onto the plane perpendicaly to any view direction are equal:

$$
\int_{m \in \Theta} D(\mathbf{m})(\mathbf{v} \cdot \mathbf{m}) d\mathbf{m}
= \mathbf{v} \cdot \mathbf{n}
\text{, dot products are NOT clamped to 0 (9.22)}
$$
$$
\int_{m \in \Theta} G_1(\mathbf{m}, \mathbf{v}) D(\mathbf{m})(\mathbf{v} \cdot \mathbf{m})^+ d\mathbf{m}
= \mathbf{v} \cdot \mathbf{n}
\text{, dot products ARE clamped to 0 (9.23)}
$$

Heitz solved the dilemma (for now) as to which $G_1$ to use.
From the masking function proposed in the litterature, only two satisfy equation 9.23:
* Torrance-Sparrow "V-cavity".
* Smith masking function.
  * Closer match to behavior of random microsurfaces that Torrance-Sparrow.
  * _normal-masking independance_: does not dependepns on the direction of $\mathbf{m}$ as long as it is frontfacing.
  * drawbacks:
    * theoretically not consistent with structure of actual surfaces
    * practically, it is quite accurate for random surfaces, but less when there is a strong dependency between normal direction and masking.

Smith $G_1$:
$$
G_1(\mathbf{m}, \mathbf{v}) =
\frac{\chi^+(\mathbf{m} \cdot \mathbf{v})}{1 + \Lambda(\mathbf{v})}
$$
with:
* $\chi^+(x)$ the positive characteristic function: 1 when $x > 0$, 0 when $x \leq 0$.
* $\Lambda$ function differing for each NDF.

From those elements, the overall macrosurface BRDF can be derived:
$$
f(\mathbf{l}, \mathbf{v}) =
\int_{m \in \Omega}
    f_\mu(\mathbf{l}, \mathbf{v}, \mathbf{m})
    G_2(\mathbf{l}, \mathbf{v}, \mathbf{m})
    D(\mathbf{m})
    \frac{(\mathbf{m} \cdot \mathbf{l})^+}{|\mathbf{n} \cdot \mathbf{l}|}
    \frac{(\mathbf{m} \cdot \mathbf{v})^+}{|\mathbf{n} \cdot \mathbf{v}|}
    d\mathbf{m}
$$

$G_2(\mathbf{l}, \mathbf{v}, \mathbf{m})$ is the _joint masking-shadowing function_.
It derives from $G_1$, and accounts for masking as well as shadowing (but not interreflection),
giving the fraction of microfacets with normal $\mathbf{m}$ that are visible from two directions:
* view vector $\mathbf{v}$
* light vector $\mathbf{l}$

Several forms of $G_2$ are given p335. Heitz recommends the _Smith height-correlated masking-shadowing function_:
$$
G_2(\mathbf{l}, \mathbf{v}, \mathbf{m}) =
\frac{\chi^+(\mathbf{m} \cdot \mathbf{v}) \chi^+(\mathbf{m} \cdot \mathbf{l})}
     {1 + \Lambda(\mathbf{v}) + \Lambda(\mathbf{l})}
$$

For rendering, the general microfacet BRDF is used to derive a closed-form solution given a specific choice of micro-BRDF $f_\mu$.

#### BRDF models for surface reflection p336

With few exceptions, specular BRDF terms used in PBR are derived from microfacet theory.

The _half vector_ $\mathbf{h}$ is given by:
$$
\mathbf{h} = \frac{\mathbf{l} + \mathbf{v}}{||\mathbf{l} + \mathbf{v}||}
$$

In the case of specular reflection, each microfacet is a perflectly smooth Fresnel mirror (reflect each incoming ray in a single reflected direction).
So, $f_\mu(\mathbf{l}, \mathbf{v}, \mathbf{m})$ is zero unless $\mathbf{m}$ is aligned with $\mathbf{h}$.

This collapses the integral into an evaluation of the integrated function at $\mathbf{m} = \mathbf{h}$:
$$
f_{\text{spec}}(\mathbf{l}, \mathbf{v}) =
\frac{
    F(\mathbf{h}, \mathbf{l})
    G_2(\mathbf{l}, \mathbf{v}, \mathbf{h})
    D(\mathbf{h})
}
{
    4 |\mathbf{n} \cdot \mathbf{l}| |\mathbf{n} \cdot \mathbf{v}|
}
$$

Note: the book points at optimization to avoid calculating $\mathbf{h}$ and to remove $\chi^+$.

Now formulas for the NDF $D$ and the masking-shadowing function $G_2$ must be found.

##### Normal Distribution Functions p337

The shape of the NDF determines the width and shape of the cone of reflected rays
(i.e. the specular lobe), which determines the size and shape of specular highlights.

###### Isotropic NDF p338

An isotropic NDF is _shape-invariant_ if the effect of its roughness parameter is equivalent to scaling the microsurface.
* An arbitrary isotropic NDF has a $\Lambda$ function depending on two variables, roughness and incidence angle.
* For a shape-invariant NDF, $\Lambda$ only depends on variable $a$
  (with $\mathbf{s}$ replaced by either $\mathbf{v}$ or $\mathbf{l})$:
  $$
  a = \frac{\mathbf{n} \cdot \mathbf{s}}{\alpha \sqrt{1 - (\mathbf{n} \cdot \mathbf{s})^2}}
  $$

Isotropic Normal Distribution Functions:
* Beckmann NDF p338 (first microfacet model developed by the optics community)
  is chosen for the Cook-Torrance BRDF. The NDF is _shape-invariant_.
* Blinn-Phong NDF p339 (less expensive to compute than others)
  was derived by Blinn as a modification of the (non physically based) Phong shading model.
  $$
  D(\mathbf{m}) =
  \chi^+(\mathbf{n} \cdot \mathbf{m})
  \frac{\alpha_p + 2}{2 \pi}
  (\mathbf{n} \cdot \mathbf{m})^{\alpha_p}
  $$
  * $\alpha_p$ is the roughness parameter of the Blinn-Phong NDF (higher is smoother).
    * Since it is visual impact is highly non-uniform, its is often mapped non-linearly
      to a user-manipulated parameter, such as $s \in [0, 1]$ in $\alpha_p = m^s$
      with constant $m$ being the upper-bound (e.g. 8192).
    * Equivalent values for the Beckmann and Blinn-Phong roughnesses are found using
      $ \alpha_p = 2 \alpha_b^{-2} - 2$.
  * It is not shape-invariant, and an analytic form for its $\Lambda$ function does not exist.
    * Walter et al. suggest using the Beckmann $\Lambda$ with the parameter equivalent function above.
* _Throwbridge-Reitz distribution_ NDF p340(recommended by Blinn in a 1977 paper)
  was rediscovered by Walter et al. who named it _GGX distribution_,
  which is now the common name.
  $$
  D(\mathbf{m}) =
  \frac{
     \chi^+(\mathbf{n} \cdot \mathbf{m})
     \alpha_g^2
  }
  {
     \pi
     (1 + (\mathbf{n} \cdot \mathbf{m})^2 (\alpha_g^2 - 1))^2
  }
  $$
  * $\alpha_g$ roughness control is similar to that provided by Beckmann $\alpha_b$.
    * In the Disney principled shading model, the user-manipulated roughness parameter $r \in [0, 1]$ maps as $\alpha_g = r^2$. It has a more linear visual effect, and is widely adopted.
  * It is shape-invariant, with a relatively simple $\Lambda$:
  $$
  \Lambda(a) =
  \frac{-1 + \sqrt{1 + \frac{1}{a^2}}}{2}
  $$
    * note: $a$ only appears squared, which avoids a squareroot to compute it.
  * the popularity of GGC distribution and Smith masking-shadowing function has lead to several optimizations for combination of the two p341.
* _generalized Throwbridge-Reitz_ (GTR) p342 NDF goal is to allow more control over the NDF's shape (specifically the tail). It is not shape-invariant.
* _Student's t-distribution_ STD and _exponential poweer distribution_ EPD p343 are shape invariant, and quite new (unclear if they will find use atm).

An alternative to increasing NDF complexity is to use multiple specular lobes p343.


###### Anisotropic NDF p343

TODO what is $\theta_m$ p343? Is it the elevation (polar) angle of the microfacet compared to the macrosurface normal $\mathbf{n}$?

Rough outline:
* The tangent and bitangent have to be perturbed by the normal map (and an optional _tangent map_) to obtain the TBN frame where $\mathbf{m}$ is expressed. p344
* Obtain an anisotropic version of the NDF, presented for **Beckmann NDF** and **GGX NDF** p345
* Optionally use a custom parameterizaion for both roughness $\alpha_x$ and $\alpha_y$. Presented for **Disney principled shading model** and **Imageworks**.

##### Multiple-bounce surface reflection

Imageworks combines elements from previous work to create a multiple-bounce specular term **that can be added** to the specular BRDF.
$$
f_{\text{ms}}(\mathbf{l}, \mathbf{v}) =
\frac
{
    \overline{F} \text{ } \overline{R_{\text{sF1}}}
}
{
    \pi
    (1 - \overline{R_{\text{sF1}}})
    (1 - \overline{F} (1 - \overline{R_{\text{sF1}}}))
}
(1 - R_{\text{sF1}}(\mathbf{l}))
(1 - R_{\text{sF1}}(\mathbf{v}))
$$

* $R_{\text{sF1}}$ is the directional albedo of $f_{\text{sF1}}$
  * $f_{\text{sF1}}$ is the specular BRDF term with $F_0$ set to 1
  * $R_{\text{sF1}}$ depends on $\alpha$ and $\theta$, it can be precomputed numerically (32x32 texture is enough according to Imageworks).
* $\overline{F}$ and $\overline{R_{\text{sF1}}}$ are the cosine weighted averages over the hemisphere.
    * $\overline{F}$ closed forms are provided p346
    * $\overline{R_{\text{sF1}}}$ can be precomputed in a 1D texture (or curve-fitted), it depends only on $\alpha$, see p346

#### BRDF models for subsurface scattering p347

TODO: Surface reflection are equaled to specular reflection, understand that...

Scoped to local subsurface scattering (diffuse surface response in opaque dielectrics)

Subsurface albedo $\rho_\text{ss}$ is the ratio of energy (of light) that escape a surface compared to the energy entering the interior of the material.
It is modeled as an RGB value (a spectral distribution).
It is related to the scattering albedo (chapter 14)

For dielectrics, it is usually brighter than the specular color $F_0$.
Subsurface albedo results from a different physical process (absorption in the interior, I guess what escape is the complement) than specular color (Fresnel reflectance at the surface).
Thus it typically has a different spectral distribution than $F_0$.

**Important**: Not all values in the sRGB color gamut are plausible subsurface albedos, p349.

Some BRDF models for local subsurface scattering take roughness into account by using a diffuse micro-BRDF $f_\mu$ (microfacet theory), some do not:
* If microgeometry irregularities are larger than subsurface scattering distances, microgeometry-related effects such as retroreflection occur.
A rough-surface diffused model is used, typically treating ss as local to each microfacet, thus only affectiong the micro-BRDF $f_\mu$.
* If scattering distances are larger than microgeometry irregularities, the surface should be considered flat when modeling subsurface scattering (retroreflection does not occur).
Subsurface scattering is not local to a microfacter, thus cannot be modeled via microfacet theory.
A smooth-surface diffuse model should be used.


##### Smooth-surface subsurface models p351

Diffuse shading is not directly affecty by roughness.

TODO: why is the flat mirror using $F(\mathbf{n}, \mathbf{l})$ and the microfacet specular term using $F(\mathbf{h}, \mathbf{l})$? I suppose this changed appeared earlier in the chapter.

See section for a selection of available equations.


##### Rough-surface subsurface models p353

* _Disney diffuse_ model p353
 * by default, use the same roughness as the specular BRDF, which limits materials.
   But a separate "diffuse roughnes" could be easily added
 * Apparently not derived from microfacet theory
* Oren-Nayar BRDF (the most-well known) p354
* Derivations from the isotropic GGX NDF:
  * Gotanda p355. Does not account for interreflections, has a complext fitted function.
  * Hammon p355. Uses interreflections, fairly simple fitter function.
    * Fundamentally assumes that irregularities are larger than scattering distances, which may limit materials it can model.

#### Cloth p356

#### Wave optics BRDF models p359

_Wave optics_, or _physical optics_, by opposition to _geometrical optics_,
is required to model the interaction of light with
_nanogeometry_: irregularities in the range [1-100] wavelength.

No diffraction occurs below 1 wavelength, and above 100 the angle between diffracted light and specularly reflected is so small it becomes negligible.

##### Diffraction models p360

Nanogeometry causes _diffraction_.

TODO: define diffraction.

##### Thin-film interference p361
_Thin-film interference_ is a wave optics phenomenon occuring when light paths reflect
from the top and bottom of a thin dielectric layer and interfere with each other.
The path has to be short (so the film has to be thin) because of _coherence length_:
the maximum distance a copy of a light wave can be displaced and still interfere coherently
(this length is inversely proportional to the bandwidth of the light).
For the human visual system (400-700 nm), coherence length is about 1μm.

#### Layered material p363

Simple and visually significant case of layering is _clear coat_:
a smooth transparent layer over a substrate of a different material.

Visual effect of clear-coat:
* Double reflection (clear-coat and underlying substrate).
  Most notable when the difference in IOR is large (e.g. metal substrate).
* Might be tinted (by absorption in the dielectric)
* In the general case, layers could have different surface normals (but uncommon for real-time rendering).

#### Blending and filtering materials p365

Ideally, when blending between materials (when staking, or at mask soft-boundaries),
the corect approach would be to compute each material, then blend linearly between them.
The same result is achieved by blending the linear parameters and computing the material,
but blending non-linear BRDF parameters is theretically unsound.
Yet, in real-time rendering, this is the usual approach, and the results are satisfying.

Blending normal-maps requires special consideration
(e.g. treating as a blend between equivalent height-maps), see p366.

"Material filtering is a topic closely related to material blending." TODO: understand why

_specular antialiasing_ techniques mitigate flickering highlights due to specular aliasing.
(e.g. due to linearly filtering normals, or BRDF roughness,
which have an non-linear relationship to the final color).
There are other potential artifacts (e.g. unexpected changes in gloss with viewer distance), butspecular aliasing is usually the most noticeable issue.


##### Filtering normals and normal distributions p367

Illustrate the issue of naive linear filtering of normal map and roughness.

Propose some solution for better filtering.
* Toksvig's method (intended for Blinn-Phong NDF, but usable with Beckmann):
  $$
  \alpha_p' =
  \frac
  {
    ||\overline{\mathbf{n}}|| \alpha_p
  }
  {
    ||\overline{\mathbf{n}}|| + \alpha_p (1 - ||\overline{\mathbf{n}}||)
  }
  $$
  * Usable with GGX with translation function for roughness, even though it does not have good theoretical foundation for it.
  * Works with the simplest normal mipmapping scheme: linear averaging without normalization.
  * Does not work well with compression techniques for normal, which require normals being unit length.

There is another family of mapping techniques (based on mapping the covariance matrix of the normal distribution): _LEAN_, _CLEAN_, _LEADR_. p370 (I do not understant it atm).

TODO: understand what are the variance mapping techniques.
_variance mapping_ family of techniques is commonly used for static normal-maps predominent in real-time rendering.
