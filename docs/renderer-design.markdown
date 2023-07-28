## Material system / Shader system

[ourmachinery](https://web.archive.org/web/20220529231010/https://ourmachinery.com/post/) has a 3 parts serie:
* [part 1](https://web.archive.org/web/20210711062143/https://ourmachinery.com/post/the-machinery-shader-system-part-1/)
  Describing the difficulties in designing such systems, and their stated goals. Describe their 2 tier (shader, materials) approach, and the low level implementation of shaders.
* [part2](https://web.archive.org/web/20220517143514/https://ourmachinery.com/post/the-machinery-shader-system-part-2/)
  The shader declaration structure and corresponding JSON front-end (data-driven), and the abstraction it provides for resources and constant bindings. Brief description of the Cpp side of binding.
* [part3](https://web.archive.org/web/20210804175728/https://ourmachinery.com/post/the-machinery-shader-system-part-3/)
  High level aspects, addressing shader variations, runtime selection, as well as providing resources and constants (both context and per instance) with a notion of `System`.
  Presentation of the `Material` concept (maybe too briefly).

