# OpenGL

Notes and reminders aroud my understanding of modern OpenGL (4.2+)

> A program (the **client**) issues commands, and these commands are interpreted and processed
by the GL (the **server**).

## TODOs

* Find a spec definition of _Command Stream_

* Find a spec defintion of what it means for a command to "reach the GL server".
(In particular, does it mean it is actually on the GPU,
or could it just reach an internal driver buffer in main memory?)

* I'd like to get a formal definition of what the spec exactly means by:
  * _vertex array_ (e.g. p362 sec 10.3.9)
    > Additionally, each vertex array has an associated binding so there is a buffer object binding for each of the vertex attribute arrays.
    The initial values for all buffer object bindings is zero. (p84 bottom)
  * _vertex attribute array_ e.g. first line p365
    > "If an enabled vertex attribute array is instanced"

* Understand _derived state_:
  > Some of [the state] [...] is derived state visible only by the effect it has on how OpenGL operates. (p3 top)

* Complete the list of fixed-function stages

* Complete compute rendering command section

## Conventions

> "GLSL matrices are always column-major" (https://www.khronos.org/opengl/wiki/Interface_Block_(GLSL))

Shader code use pre-multiplication: `tansformation * vector`.

Prefixes in the C language:
* functions: `gl`
* constants: `GL_`
* types: `GL` (e.g. `GLfloat`)

## Data flow

Figure 3.1: Block diagram of the GL pipeline
![](images/opengl/gl46-pipeline_data_flow.png)

## Execution model

The application (client) issue commands executed by the GL (server).
**Commands** are always processed in the order they are received.

Implementation can buffer **commands** in a **command queue**.


## Context and Object model

> A server may maintain a number of **GL contexts**, each of which
is an encapsulation of current **GL state** and **objects**.
A client may choose to be made _current_ to any one of these contexts.

p3 1.2.2:
> OpenGL contains [...] many **types of objects** representing _programmable shaders_ and the data they consume and generate,
as well as other **context state** controlling _non-programmable aspects_ of OpenGL.


### Context state

**Context state** belongs to the GL context **as a whole**, rather than to specific instances of GL _objects_.
It controls **fixed-function** stages of the GPU and  specifies **bindings** of objects to the context.
There are two types **server state** (majority) and **client state**.
(p27 2.5)


Fixed function stages:
* Clipping
* Rasterization
* Framebuffer clear
* (TODO)


### Generalities on objects

OpenGL API gives access to **objects**, which are identified by their **name**,
Each type of object has a distinct **name space**, where names are a `GLuint` value (not actual text).

_Names_ are distinct from the underlying _objects_:
`glGen*()` functions reserve and return previously unused _names_,
but the _objects_ are created and bound to _names_ later, through other GL functions
(e.g. when names are bound to the context with `glBind*()`).
_Objects_ are modeled as a _state vector_, creating the object notably creates this state vector.

GL defines many types of _objects_, applications operate on _instances_ of those _objects_. Specific _instances_ are _bound_ to a _context_.

**Binding** _objects_ to the context determines which _objects_ are used during commands execution.

### "Standard" OpenGL Objects operations

The _object model_ describes how most types of objects are managed by the application.

Available _names_ are obtained and reserved with `glGen*()`.
Underlying _object_ is created and associated to a given _name_ with first call to `glBind*()` for this name: it creates a **state vector** for the _object_ with default values by the spec.
Additionnaly, the _bound_ object is the one affected by further GL operations/queries for this type.
Some type of _objects_ have a **target** (such as [Buffer Objects](#buffer_objects),
in which case there is one distinct _binding point_ for each _target_.

_Object_ might be created and associated to a new name at once with `glCreate*()`.

_Names_ are marked as unused with `glDelete*()`, but the underlying _object_ and its state are not deallocated **until it is not longer in use**.

**Query commands** start with `glGet`.

### Event Model

#### Fence

A **Fence Object** is associates with a **Fence Command** placed in the GL command stream.
The Fence Object is initially `UNSIGNALED`, and becomes `SIGNALED` at some point after the _Fence Command_ is completed.
_Fence command_ completion guarantees all preceding commands in the same command stream did complete.

#### Queries

**Query Objects** give access to _asynchronous queries_:
a mechanism to return information about the processing of a sequence of _GL commands_.

_Query object_ names are reserved with `glGenQueries()`.
A _query object_ is associated to the name with `glBeginQuery*()` (or `glQueryCounter()` for timer query), for provided _target_.
The state is queried via `glGetQuery*Object*()`, notably the result with `QUERY_RESULT`.

There is a **query result buffer** (buffer target `GL_QUERY_BUFFER`) to place query results in server memory.

##### Timing

* `gl[Begin|End]Query(GL_TIME_ELAPSED)`: _time elapsed query_
* `glQueryCounter()`: _timer query_

Both record time after all previous commands have been fully realized (Note: there is an error in the API reference of `glQueryCounter()`).

**Note**: The synchronous command `glGetInteger*(TIMESTAMP)` returns the GL time after all previous commands have reached the GL server,
but have not yet necessarily completed (by opposition to a _timer query_, providing time after completion).
This distinction **allows to measure latency**.

### <a name="buffer_objects"/> Buffer Objects

**Buffer objects** contain a _data store_ holding a fixed-size allocation of server memory,
and provide a mechanism to allocate, initialize, destroy, read from, and write to such memory.

The data store of a buffer object is created via:
* `gl*BufferStorage()` to obtain an immutable-size buffer (content is mutable).
* `gl*BufferData()` for a mutable buffer.

Buffer data can also be:
* modified
* mapped (for access in client address space),
* invalidated (`glInvalidateBuffer*Data()`, data then have undefined value)
* copied
* queried (`glGet*BufferParameter*()` for parameters,
`glGet*BufferPointerv()` for mapped pointer, `glGet*Buffer*Data()` for data)

Buffer usage is dictated by the **target** to which it is currently bound.

>Buffer objects created by binding a name returned by `glGenBuffers` to any of the valid targets
are formally equivalent.

(i.e. a buffer can be bound to distinct target at different points).

* _Array buffer_ is a buffer from which _generic vertex attributes_ are fetched. (i.e. the buffer target is `GL_ARRAY_BUFFER`)
* _Element buffer_ (or _Index buffer_) is the buffer from which vertex indices are fetched (target is `GL_ELEMENT_ARRAY_BUFFER`).
* _Uniform Buffer Objects_ are buffer objects used by an application to store uniform data for a shader program.
The shader program access the storage via GLSL _uniform blocks_ (the GLSL grouping of uniforms)
(target is `GL_UNIFORM_BUFFER`).
The link is made via a _uniform buffer binding location_.
* _Shader Storage Buffer Objects_ are accessed via GLSL _shader storage block_
(target is `GL_SHADER_STORAGE_BUFFER`).

#### Indexed targets

Some GL buffer targets are _indexed_.
For **indexed targets**, there is an array of buffer object **binding points**,
as well as the usual single **general binding point** (like for non-indexed targets).

A buffer can be bound to the _binding point_ at _binding index_ with
`glBindBufferBase(target, index, buffer)` (or `glBindBufferRange()`).
_Binding index_ is the index into an indexed buffer object target. Not to be confused with the index of the resources in GLSL programs (e.g. _block index_ for an interface block)

Indexed targets:
* `GL_ATOMIC_COUNTER_BUFFER`
* `GL_TRANSFORM_FEEDBACK_BUFFER`
* `GL_UNIFORM_BUFFER`
* `GL_SHADER_STORAGE_BUFFER`

**Notes**: Binding to an _indexed binding point_ is required for specific use by GL
(notably, to access the buffer in a shader program),
and also binds to the _general binding point_ (thus unbinding previously bound buffer).
Non-indexed binding (`glBindBuffer()`) can also be used for _indexed targets_,
it only binds to the _general binding point_.

In both case, the general binding point allows to use general buffer object manipulation functions,
whereas indexed binding usually means that the buffer content will be used (rather than just modified).

### <a name="VAO"/> Vertex Array Object

**Vertex Array Object** (or **VAO**) capture state for rendering.
A _VAO_ represent a collection of sets of _vertex attributes_,  each set values (the actual data)
are stored outside the _VAO_, as an array in a _buffer object_ data store
(TODO is the buffer called **vertex attribute array**?).

Binding the _VAO_ restore all captured state at once:
* _Vertex format_
  * enabled arrays (`glEnableVertex[AttribArray|ArrayAttrib]()`).
  * data type, element count, normalization
  * relative offset (added to the binding point _base offset_, thus allowing _interleaving_ attributes)
  * associated buffer binding point (binding index <-> vertex attribute index)
* _Buffer binding point_ state, **shared by all vertex attributes** pulled from this binding point
  * source _buffer object_ (buffer object <-> binding index)
  * base offset into the buffer
  * byte stride
  * [Instance divisor](#divisor), for instanced rendering
* _Index Buffer_ binding (The last _buffer object_ bound to `GL_ELEMENT_ARRAY_BUFFER` while the _VAO_ was bound)


**Important**: if the array corresponding to an attribute (required by a _Vertex Shader_) is not enabled, the corresponding element is taken from the current _attribute state_.
(`glVertexAttrib*()` family of functions allow to set the default state for an attribute, which is used when the attribute array is not enabled)


### Shader Objects & Program Objects

**Shader objects** to be used by one/several of the _programmable stages_ are linked together into a **Program object**. Shader programs executed by these stages are called **executable**, all information necessary for defining each _executable_ is encapsulated in a _program object_.

#### Interface blocks

**Interface blocks** are GLSL side constructs. They group input, output, uniform (in uniform blocks), _buffer variables_ (in shader storage blocks).

```
storage_qualifier[in / out / uniform / buffer] block_name
{
  <members>
} instance_name;
```

`in`, `out` can only be used **between** shaders stages (not as vertex shader input or fragment shader output).
_Uniform blocks_ and _shader storage blocks_ are collectively called **buffer-backed blocks**: the storage for their content come from a _Buffer Object_. Those buffers are indexed.

#### Program Interfaces & Introspection

An _executable_ communicate with other pipeline stages or the application through a variety of _interfaces_.
Each _interface_ has a list of _active resources_ (built at compile time).

GL provides functions to query properties of the _interfaces_ of a _program object_:
* `glGetProgramInterfaceiv()` queries properties of the _interface_ itself (e.g. number of active resources, maximal resource name length...)
* `glGetProgramResourceiv()` queries properties of the _active resource_ at `index` in `programInterface`.
* `glGetProgramResourceName()` queries the name of active resource at `index` in `programInterface`.

For interface blocks, the _active resources_ are the blocks themselves, not the variables they group:
* `GL_UNIFORM_BLOCK` active variables are accessible via the `GL_UNIFORM` program interface.
* `GL_SHADER_STORAGE_BLOCK` active variables are accessible via the `GL_BUFFER_VARIABLE` program interface.
  (consistent with the name _buffer variable_ for individual variable in a _shader storage blocks_.)

#### Program Pipeline Objects

A **program pipeline object** composes a pipeline by using shader stages from several _program objects_,
which must have the `GL_PROGRAM_SEPARABLE` parameter set. (`glProgramParameteri()`).
Note: they build **on top** of _program objects_, since they take shader stages from them.
_Program pipeline objects_ allow to combine stages without requiring one _program object_ instance for each combination.



### Textures

* `Pack`: read from OpenGL
* `Unpack`: write to OpenGL

`glTexStorage*()` specifies the internal format, i.e. the format OpenGL will use to store the texture data.
**Note**: For the `internalformat` parameter, use **sized formats** to be explicit (instead of letting implementation decide).

`glTex*Image*()` is used to load data into an allocated texture. The format and type are describing the data provided by the application.

_Textures_ have a **target**. Unlike _buffers_, a _texture_ must always be bound to the same _target_ it was initially created with.

#### Using textures with GLSL samplers

_Texture names_ (the `GLuint` returned by `glGenTextures()`) have to be bound to **texture image units** (sometimes just called _texture unit_, warning: _image unit_ are distinct, corresponding to image uniforms below),
Binding a texture with `glBindTexure()` also binds it to the currently active _texture image unit_ set with `glActiveTexture()`.
GLSL shader programs can _sample_ the texure via `sampler` uniform variables (whose type must match the texture target), set to the _texture image unit_ with `glUniform()`.
**Warning**: A `sampler` uniform variable is a GLSL type, distinct from a _Sampler Object_, which is an application object encapsulating sampling parameters.

#### Image load/store

**image variables** are GLSL uniform variables allowing arbitrary read/write to/from image within a Texture.
A given _level_ ("image") in a _texture_ is bound to an _image unit_ (distinct from _texture image unit_) with `glBindImageTexture()`. The uniform value of an `image` GLSL variable (of matching type) is set to the same _image unit_.


### Framebuffers

A **framebuffer** is a collection of logical buffers (color, depth, stencil) used as destination of rendering operations, or source of readback operations.

_Framebuffer **Objects**_ (_FBOs_) are user-created. Their logical buffers reference individual **framebuffer--attachable images**, either from **textures** (_render to texture_) or from **renderbuffers** (_off-screen_ rendering), attached to a set of **attachment points**.
An _FBO_ must be **complete** to be used for rendering.

The **default framebuffer** is provided by the system upon _context_ creation.
It usually consists of multiple colors buffers ([front|back][left|right] buffer), _depth buffer_ and _stencil buffer_.

GL has 2 active _framebuffers_:
* **draw framebuffer** (destination for **rendering operations**)
* **read framebuffer** (source for **readback operations**).

The separation between _Draw_ and _Read_ framebuffers exists so the Read frambebuffer can be blitted to the Draw framebuffer.

#### Attachment

Framebuffer object has a set of attachment points (`MAX_COLOR_ATTACHMENTS`, + depth, +stencil).
The color attachment points are named `COLOR_ATTACHMENT0` to `COLOR_ATTACHMENTn`.

The state is the FBO state (spec 23.24) + array of attachment point states (spec 23.25).
Attachment point state notably defines the attached object _type_ and _name_.

FBO logical buffers are idenfified by attachment point names.
* A Renderbuffer is attached to a FBO attachment point with `glFramebufferRenderbuffer()`.
* A level of a texture object is attached as a logical buffer of a FBO with `glFramebufferTexture()`.

#### Shader mapping

Output fragment colors, as defined in fragment shaders,
are mapped to FBO color attachment with `glDrawBuffers(color_attachments[])`.

The index of the `color_attachment` array correspond to the output fragment color `location`
(e.g. `layout(location = 1) out colorBis;`),
the value at this index is the FBO's color attachment mapped to this location.

#### Buffers

* Color buffers can have up to 4 color components (RGBA) per pixel.
  All _framebuffers_ might have multiple colors images that could be read/written.
* Depth buffer pixels are a single value (unsigned int or floating point)
* Stencil buffer pixel are single int value.

#### Renderbuffer

**Renderbuffer** objects contain images, intended to be used with _Framebuffer Objects_.
Unlike textures, they are optimized for use as _render targets_ (i.e. when there is no need to sample the image), and natively accomodate _multisampling_.

#### Layered rendering

**TODO** (the GS send specific primitives to different layers of a layered framebuffer)

## Vertex specification

glspec 10 p337
> The process of specifying attributes of a vertex and passing them to a shader is referred to as _transferring_ a vertex to the GL.

Vertices are specified with on or more _generic vertex attributes_ (1, 2, 3 or 4 scalar values).
_Vertex shaders_ access an array of 4-components _generic vertex attributes_, this array size is `MAX_VERTEX_ATTRIBS`.
They are loaded into the _shader attribute variable_ bound to the generic attribute _index_ (`[0..MAX_VERTEX_ATTRIBS[`).

glspec 10.2.1 p348
> Matrices are loaded into these slots in column major order. Matrix columns are loaded in increasing slot numbers.

_Current attribute values_ are transferred for a vertex when there is not vertex array enabled for a required attribute.
The value can be changed by `glVertexAttrib*()`, and queried with `glGetVertexAttrib*()`.

### Vertex Arrays

Vertex data can be placed into arrays (Buffers with target `GL_ARRAY_BUFFER`).
All the state to represent the _vertex arrays_ is stored in a [_Vertex Array Object_](#VAO):
it encapsulates all state related to the definition of data used by the _vertex processor_, and notably collects the _buffer objects_ to be used by the _vertex stage_.

`glEnableVertexAttribArray()` ([DSA](#DSA) variant: `glEnableVertexArrayAttrib()`) enables an individual _generic vertex attribute array_ in the selected _VAO_.

#### Separate format and data source

Specifying the organization of data store of _generic vertex attribute_ for a _VAO_:<br/>
`glVertexAttrib*Format(attribindex, size, type, normalized, relativeoffset)` ([DSA](#DSA) variant: `glVertexArrayAttrib*Format()`).<br/>
This stores in the VAO that the _generic vertex attribute_ at index `attribindex` will be fetched from an array of described format,
starting at `relativeoffset` (allow **interleaving** different attributes in a single buffer).

Associating a vertex buffer object to a VAO's _buffer binding index_:<br/>
`glBindVertexBuffer(bindingindex, buffer, offset, stride)` ([DSA](#DSA) variant: `glVertexArrayVertexBuffer()`).<br/>
This provides a `stride` and `offset` for a buffer and attach it to the specified _binding index_ of selected _VAO_.

Associating a vertex attribute to a _VAO_'s _buffer binding index_:<br/>
`glVertexAttribBinding(attribindex, bindingindex)` ([DSA](#DSA) variant: `glVertexArrayAttribBinding()`).<br/>
The _vertex attribute_ at `attribindex` will be fetched from the buffer currently attached to `bindingindex` of the selected _VAO_.

#### <a name="divisor"/> Vertex Attribute Divisor

A _generic attribute_ is **instanced** when it **divisor** is non-zero.
In this situation, the attribute advances once per _divisor_ number of instances (implies [instanced rendering](#draw_instanced)).
By opposition, attributes with the default _divisor_ 0 advance **once per vertex**.

_Divisor_ is set with `glVertex*BindingDivisor()`.


## Rendering commands

**Rendering commands** perform rendering into a _framebuffer_. They are:
* [_drawing commands_](#drawing_commands)
* `glBlitFramebuffer()`
* `glClear()`, `glClearBuffer*()`
* [`glDispatchCompute*()`](#compute)


### <a name="drawing_commands"/> Drawing commands

see: glspec 10.4

**Drawing commands**, of the form `gl*Draw*()`, send vertices through OpenGL to be rendered.

Some parameters are shared by most commands:
- `GLenum` parameter `mode` is the primitive type.
- `GLsizei` parameter `count` is the number of vertices that will be tranferred.

`gl[?Multi]Draw[Arrays|Elements][?Indirect]()`<br/>
`gl[?Multi]DrawElementsBaseVertex()`<br/>
`glMultiDraw[Arrays|Elements]IndirectCount()`

- `Arrays|Elements`:
  - `Array`: fetches in contiguous sequence from enabled vertex arrays.
  - `Elements`: fetches from enabled vertex arrays at indices provided by the buffer bound to `GL_ELEMENT_ARRAY_BUFFER`.
    Parameter `type` indicates the integral type of the value in the element array buffer.
  - In both cases, the index of the current array element may be read by a VS via `gl_VertexID`, **including** `BaseVertex`.
- `Multi`: is equivalent to issuing parameter `drawcount` draw commands at once, parameters becoming pointers into arrays of `drawcount` elements.
  - VS can read the index of the draw via `gl_DrawID`.
- `BaseVertex`: the value of `GLint` parameter `basevertex` is added to the indices read from the element array buffer **before** pulling the vertex data. It allows to load vertices for distinct models in the same `GL_ARRAY_BUFFER` without requiring to re-index subsequent models.
(Note: this is only available for `Element` variant which accesses an _element buffer_. `Array` variant does not fetch indices from a buffer, but provides `first` parameter to specify starting index in arrays).
  - `gl_BaseVertex` built-in give shader access to this parameter.
- `Count`: `drawcount` is now a byte offset into the (server space) buffer bound to `GL_PARAMETER_BUFFER`, where a single `GLsizei` value is read to provide the actual draw count.
- [`Indirect`](#draw_indirect): read some of the draw parameters from server space memory, from the buffer bound to `GL_DRAW_INDIRECT_BUFFER`, offset by parameter `indirect` bytes.

#### <a name="draw_instanced"/> Instanced

`gl[DrawArraysInstanced[?BaseInstance]()`<br/>
`glDrawElementsInstanced[?BaseVertex]}[?BaseInstance]()`

- `Instanced`: the sequence of vertices will be drawn parameter `instance` times.
  Instanced vertex attribute array ([non-zero `divisor`](#divisor)) will advance by one every `divisor` instances.
  VS may read the current instance index via `gl_InstanceID` (**not** including `BaseInstance` offset).
- `BaseInstance`: parameter `baseinstance` specifies the first element within instanced vertex attributes (i.e. an index offset).
  - `gl_BaseInstance` built-in give shader access to this parameter.

#### <a name="draw_indirect"/> Indirect

`GL_DRAW_INDIRECT_BUFFER` should contain data matching the corresponding struct (as tightly packed 32-bit values):


* `gl*DrawArraysIndirect*()`:
  ```
  struct DrawArraysIndirectCommand
  {
      uint count;
      uint instanceCount;
      uint first;
      uint baseInstance;
  };
  ```

* `gl*DrawElementsIndirect*()`:
  ```
  struct DrawElementsIndirectCommand
  {
      uint count;
      uint instanceCount;
      uint firstIndex;
      int baseVertex;
      uint baseInstance;
  }
  ```

In case of `glMultiDraw[Arrays|Elements]Indirect()`, the buffer, offset by `indirect` bytes, should contain at least `drawcount` instances of the struct.
Each instance being `stride` bytes appart (0 meaning tightly packed).

#### Range

`glDrawRangeElements[?BaseVertex]()`

**Note**: not usefull anymore, because the vertex data and indices live in server memory.

- `Range`: all element (indices) read from the element array buffer must have a value between `start` and `end` parameters (**prior** to adding `basevertex`).

### Compute

**TODO**

`glDispatchCompute[?Indirect]()`

- `Indirect`: read parameters from buffer bound to `GL_DISPATCH_INDIRECT_BUFFER`.

## Extensions

* `ARB`: Architecture Review Board (those are still considered extensions, not core)
* `KHR`: Khronos (I think those are core)

## Glossary

* <a name="DSA"/> DSA: Direct State Access. The ability to modify the GL object states without binding them. Introduced in OpenGL 4.5
* EGL: Mobile and Embedded Device Bindings
* FS: Fragment Shader
* GLX: X Window system Bindings
* MRT: Multiple Render Targets
* Render Target: Although not defined by the GL spec, render target  is the collective term for either a render buffer or a texture being used as the target for rendering
  (see: https://stackoverflow.com/a/27186611/1027706)
* TCS: Tessellation Control Shader
* TES: Tessellation Evaluation Shader
* VAB: Vertex Attribute Binding (ARB_vertex_attrib_binding),
  the ability to change the mapping between _vertex attribute_ and _vertex buffer binding_, and to specify _vertex attribute format_ separately.
* Varyings: older name for the shader _input_ and _output_ variables between the vertex and fragment stages.
* Vertex Attribute: user-defined inputs for _vertex shaders_. They are passed via _Vertex Arrays_ from data stored in _Buffer Objects_.
* VAO: Vertex Array Object is a list of the buffer objects used by the vertex stage, and how they map to _Vertex Attributes_ in the Vertex Shader. Warning: "Vertex arrays"?? or "Vertex attribute arrays" seem to refer to the buffer storage for the vertex data, not the VAO!
* VBUM: Vertex Buffer Unified Memory (NV_vertex_buffer_unified_memory), a generalization of bindless texture to buffers.
* VS: Vertex Shader
* WGL: Microsoft Windows Bindings
