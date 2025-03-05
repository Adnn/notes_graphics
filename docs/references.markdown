---
title: References
layout: default
---

# References

## Webpages

* {% include link title="Forward vs Deferred vs Forward+ Rendering with DirectX 11" url="https://www.3dgep.com/forward-plus/" v="2022/10/01" %}
  : (#implementation #deferred #tiled-forward #intermediate)\
  Excellent introduction and tutorial for Forward, Deferred, and Forward+ (Tiled Forward), using DX12. Provide several good references too.

* {% include link title="What’s the Difference Between Ray Tracing and Rasterization?" url="https://blogs.nvidia.com/blog/2018/03/19/whats-difference-between-ray-tracing-rasterization/" v="2022/09/28" %}
  : (#introductory #raytracing)\
  A good introduction to ray tracing and its development history, with comparison to rasterization.

* https://registry.khronos.org/glTF/specs/2.0/glTF-2.0.html#appendix-b-brdf-implementation\
  A BRDF implementation given as appendix by th glTF specification.

## Books

### GPU Pro 360 Guide to 3D Engine Design

* {% include g3ded p=161 ch=10 %}
  : Aspect based game engine design. Seems like a precursor to ECS design (where entites are classic objects pointing to attributes), achieving some of its modularity. Aspects are equivalent to systems. An advantage is it keeps the core scene graph data structure.

* {% include g3ded p=179 ch=11 %}
  : Serve as an introduction/tutorial on Kinect abilities and programming. With a few use ideas at the end.
* {% include g3ded p=197 ch=12 %}
  : Describe an overall approach for authored damage and replacement geometry (bounded and unlimited regions count).
* {% include g3ded p=211 ch=13 %}
  : Quaternions in practice in the rendering engine, and the components it impacts. Also remove the vertex count increase drawback from chapter 7.
    * Yet I did not understand some constraints:
      - what about SQT being done in object space, how is it different from matrices?
      - skinning, what about blending final vertex pos instead of quats?
      - in quaternion format, what is a vertex quaternion, and a model transform (by oposition to an instance quaternion)?


## Papers


* {% include link title="Real Shading in Unreal Engine 4" url="https://cdn2.unrealengine.com/Resources/files/2013SiggraphPresentationsNotes-26915738.pdf" v="2025/03/04" %}
  : (#pbr #ibl #real-time #area-light #seminal #intermediate #implementation)\
  Pioneering paper for implementation of a real-time PBR renderer, with IBL and area light).
  Cover many practical details from implementation in UE 4.
