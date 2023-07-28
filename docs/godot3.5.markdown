---
title: GODOT 3.5
---


TODO understand the different "render priority" mechanisms:
* depth_layer (is it related to SORT_KEY_OPAQUE_DEPTH_LAYER_SHIFT?)
* SORT_KEY_PRIORITY_SHIFT

TODO What is depth_layer / layer_mask interaction?

TODO What are the semantics of enumerators in VisualServer::ShadowCastingSetting

TODO whas it LightInstance::ShadowTransform, and why are there 4

## Ideas and notes


* The GLSL shaders do not show-up in the IDE, but they are at godot/drivers/gles3/shaders
* Have a "Render" object counting stats
(draw call, material switches, surface switch, shader rebind, vertices count, shader compilations)
* Instances keep track of the last render pass that rendered them (i.e. not culled)
* A list of shaders for dedicated purposes is harcoded in `RasterizerSceneGLES3::State`.
* Regarding shaders, it seems Godot has a single "logical" `SceneShader` instance, used to render any instance.
  This `SceneShader` then generate variations (i.e. distinct OpenGL shader compilations) based on several mechanisms:
  * Conditionals (conditional code compilation via macros)
  * `SanderGLES3::CustomCode`. TODO how does that work?
* `Mesh` contain a collection of `Surface` (which is-a `Geometry`).
* Distinct `Material` can be applied per surface. There is a "[material precedence](https://github.com/godotengine/godot/blob/3.5/drivers/gles3/rasterizer_scene_gles3.cpp#L2243)" logic:
  1. `InstanceBase::material_override`
  2. `InstanceBase::material[$surface_index]`
  3. `Geometry::material`
  4. `RasterizerSceneGLES3::default_material`
* `RenderList` use a pre-allocated pool of elements, adding non-alpha from front, alpha from back.
* `Material`, which contains a `Shader *`, has a `next_pass` RID to a next `Material`.
* `RasterizerSceneGLES3::State` seems to have several functions:
  * Cache a subset of the GL state, such as blen mode, line width, depth test
  * Host the different shaders (scene_shader, resolve_shader, ssr_shader, ssao_shader, ...)
  * Host the UBOs
* `Surface` host separate array, index and instancing (but not vertex) for wireframe. // TODO how is it populated?
* Regarding `VisualServer::InstanceType`: `INSTANCE_MESH` is used for a single "instance" (i.e. not GL instanced drawing), wherease `INSTANCE_MULTIMESH` does use `glDrawElementsInstanced()`.
* `Camera` has an optional `Environment`, taking precedence over `Scenario`'s environment.
* UBOs:
  * `SceneDataUBO`: everything that remain constant for a scene?
* When rendering depth prepass, disable writing to color buffers with `glDrawBuffers(0, nullptr)`. Is it the cannonical approach?
* It seems Godot's **camera transform** is the transformation from view space to world space,
  which is the inverse of what I would expect from the name (I expect the view transform).
  Same for **light transform**. I suppose Godot means the camera(or light) **model** transform.
* `Light` can have a shadow color.
* Filling a `RenderList` (`_fill_render_list()`) being separate from rendering (`_render_list()`)
  is probably to decouple both, so the same render list can be rendered multiple times
  (i.e. different passes of the Frame graph). This looks like an interesting solution.
  It is then either sorted by `sort_key` or by `depth`.
* Use of an (integer) `sort_key` to sort quickly the `RenderList`.
  Sort key is composed by bit manipulation.

* Texture image units:
  * -8 screen texture (color buffer?)
  * -7 sky irradiance
  * -6 shadow atlas
  * -5 directional shadow
  * -3 (if use_texture_array_environment) or -2 sky radiance
  * -2 see -3
  * -1 skeleton texture
* UBO binding points:
  * 0 scene data
  * 1 Material.ubo_id
  * 2 environment radiance
  * 4 omni light array
  * 5 spot light array
  * 6 reflection array

### Multithreading

`visual_server_wrap_mt.h`, which wrap each function of `VisualServer` interface in a macro
that checks if the current thread is the server thread to make a direct call, otherwise make a
call via command queue (and sync wait when there is a return value).

## Questions

* `RID` vs `RID_Data`? (`RID` contains a pointer to an `RID_Data`)
* What is a [`Scenario`](https://github.com/godotengine/godot/blob/3.5/servers/visual/visual_server_scene.h#L256)?
* Understand the different high level objects (Storage, Scene, Canvas)
* How are `uint64_t sort_key` made and used?
* What is overdraw? (there is a overdraw material and shader in RasterizerSceneGLES3)
* What are `RasterizerSceneGLES3` `render_pass` and `scene_pass` uses?

## Call outline for frame rendering

Entry point:

[(visual_server_raster.cpp L97)](https://github.com/godotengine/godot/blob/3.5/servers/visual/visual_server_raster.cpp#L97) -> `VisualServerRaster::draw()`

[(rasterizer_gles3.cpp L194)](https://github.com/godotengine/godot/blob/3.5/drivers/gles3/rasterizer_gles3.cpp#L194) -> `begin_frame()`

```
VisualServerRaster::draw(frame_step)
    RasterizerGLES3::begin_frame(frame_step)
        RasterizerStorageGLES3::update_dirty_resources()
            // multimeshes, skeleton, shaders, materiales, particles, captures
        RasterizerSceneGLES3::iteration()
            // Seem to get "globals" for shadow filtering mode, subsurface scattering, lightmapping
            ShaderGLES3::_set_conditional(int p_which, bool p_value)
                // Set bitflags for things such as skeleton, instancing, lighting,, depth, pcf, lightmpap...
                // Is it a way to introduce static shader permutations?

    VisualServerScene::update_dirty_instances()
        RasterizerStorageGLES3::update_dirty_resources()
        for instance in _instance_update_list : _update_dirty_instance(Instance *)
            VisualServerScene::_update_instance_aabb(Instance *)
            // A complicated dance with materials and surfaces
        // Get spatial partioning scene from scenario
        SpatialPartitioningScene_BVH::update()

    VisualServerViewport::draw_viewports()
        for viewport in sorted(active_viewports)
            RasterizerGLES3::set_current_render_target(viewport.render_target)
                glViewport()
            RasterizerStorageGLES3::render_info_begin_capture() // assign info.render to info.snap
            VisualServerViewport::_draw_viewport(viewport, ARVRInterface::Eyes = EYE_MONO)
                RasterizerGLES3::clear_render_target(Color) // set frame.clear_request and color
                VisualServerViewport::_draw_3d(viewport)
                    VisualServerScene::render_camera(camera, scenario, viewport_size, shadow_atlas)
                        // set camera_matrix, with optional interpolation
                        VisualServerScene::_prepare_scene()
                            BVH_Manager::cull_convex()
                                _cull_convex_iterative()
                            for culled in instance_cull_result
                                // Behaviour depending on the instance type and shadow casting
                            for ins in ligh_cull_result
                                // Setup shadow maps (coverage, ...)
                                VisualServerScene::_light_instance_update_shadow(ins)
                                    instance_shadow_cull_result = _cull_convex_from_point()
                                    RasterizerSceneGLES3::render_shadow(instance_shadow_cull_result )
                                        RasterizerSceneGLES3::_fill_render_list()
                                            for surface in each InstanceBase in instance_shadow_cull_result :
                                                RasterizerSceneGLES3::_add_geometry(surface)
                                                    // select material
                                                    RasterizerSceneGLES3::_add_geometry_with_material()
                                                        if depth path, override material to default_material
                                                        if alpha : RasterizerSceneGLES3::RenderList::add_element_alpha
                                                        else : RasterizerSceneGLES3::RenderList::add_element
                                                        // populate Element
                                                        // register render_pass to geometry.last_pass, inc current_geometry to geometry.index. TODO understand
                                                        // generate element.sort_key by bitwise ops
                                                        // register render_pass to material.last_pass, inc current_material to material.index. TODO understand
                                                    for each material.next_pass : _add_geometry_with_material() // recursively add the next pass
                                                    // do the same recursive logic if there is a material_overlay
                                        render_list.sort_by_depth(Non-alpha)
                                        // Set GL state (depth, framebuffer, blend, masks...)
                                        // glClear()
                                        // set RasterizerSceneGLES3::State values
                                        RasterizerSceneGLES3::_setup_environment(Environment *, Camera &)
                                            // populate RasterizerSceneGLES3::State ubo_data, env_radiance_data, by reading data from Environment.
                                            // Bind directional shadow texture
                                            // fill and bind scene and environment radiance UBOs
                                        state.scene_shader.set_conditional(SceneShaderGLES3::RENDER_DEPTH)
                                        RasterizerSceneGLES3::_render_list() // see below for callgraph
                            // calculate each instance depth from the camera
                        VisualServerScene::_render_scene()
                            // Select Environment following priority logic (camera, scenario, scenario fallback)
                            RasterizerSceneGLES3::render_scene()
                                // increment object_count
                                // if used, bind shadow atlas, bind reflection_atlas
                                // populate SceneDataUBO (sahdow pixel size, viewport size, ...)
                                RasterizerSceneGLES3::_setup_environment() // see above

                                if use_depth_prepass:
                                    // setup pipeline state, bind framebuffer, set viewport
                                    // Clear depth bufer
                                    _fill_render_list() // see above
                                    render_list.sort_by_depth(Non-alpha)
                                    // set shader conditional RENDER_DEPTH
                                    _render_list() // see below
                                    // increment render_pass

                                RasterizerSceneGLES3::_setup_lights()
                                    for each light:
                                        // write to a temporary ubo_data, and copy it state.spot_array_tmp
                                        // assign render_pass to light.last_pass
                                RasterizerSceneGLES3::_setup_reflections()
                                RasterizerSceneGLES3::_fill_render_list()
                                // setup GL state
                                // bind FBO (glBindFramebuffer) and specify color buffer to write (glDrawBuffers)
                                // blend state
                                render_list.sort_by_key(Non-alpha)
                                render_list()

                                if environment sky:  RasterizerSceneGLES3::_draw_sky()
                                    // fill vertex_buffer, bind vertex_array
                                    // bind shader RasterizerStorageGLES3::Shaders.copy
                                    glDrawArrays()

                                if State.used_screen_texture:
                                    // glBlitFramebuffer from Frame.current_rt.buffers.fbo
                                    //    to Frame.current_rt.effects.mip_pmaps[0].sizes[0].fbo
                                    RasterizerSceneGLES3::_blur_effect_buffer() // separatable convolution gaussian blur

                                // -- Alpha from here
                                render_list.sort_by_reverse_depth_and_priority(alpha instances)
                                    // sort by priority, and within same priority by instance depth

                                // Attention: special handling if there are multiple directional lights
                                if directional_light:
                                    RasterizerSceneGLES3::_setup_directional_light()

                                _render_list()
                                // -- Alpha till here

                                if probe:
                                    return
                                else:
                                    if dof_enabled:
                                        RasterizerSceneGLES3::_prepare_depth_textures()
                                    RasterizerSceneGLES3::_post_process()
                                        // DOF, FXAA, Bloom, Tonemap, Adjustments

                // draw canvas (2D)
            RasterizerStorageGLES3::render_info_end_capture()
                // store (info.render - info.snap) in info.snap

            // Copy viewport to screen
                // glBindFramebuffer(DRAW, 0 /*default framebuffer*/)
                // glBindFramebuffer(READ, viewport.rendertarget.fbo)
                // glBlitFramebuffer()

    VisualServerScene::render_probes() // visual_server_scene.cpp L4241

    RasterizerGLES3::end_frame()
        // advance async shader compilation
        SwapBuffers()
```

```
RasterizerSceneGLES3::_render_list()
    // bind state.scene_ubo
    // bind environment radiance / irradiance
    // set conditionals on state.scene_shader
    // Increment (RasterizerStorageGLES3::Info) storage.info draw_call_count by the element count.

    for element in p_elements:
        // Extra shading "bitset" from element.sort_key

        if ! shadow_pass:
            if shading != prev_shading:
                // N x set_conditional() to use shading

        // Compare opaque_prepass, use_instancing, prev_skeleton, octahedral_compression
        // against prev values, and update if different.

        if material != prev_material || rebind:
            // increment material_switch_count
            RasterizerSceneGLES3::_setup_material()
                // Set GL state: glEnable/Disable and write to RasterizerSceneGLES3::State
                ShaderGLES3::set_custom_shader() // set new_conditional_version.code_version to Material.Shader.custom_code_id
                ShaderGLES3::_bind()
                    // set current conditional_version to new_conditional_version

                    ShaderGLES3::get_current_version()
                        // Retrieve CustomCode and Version for current conditional_version.key
                        // Assemble the shader preamble, by pushing strings (header, #defines)
                        // Assign CustomCode.version to Version.code_version, mark uniforms NOT ready.
                        Version.ids.main = glCreateProgram()
                        // Assemble vertex & fragment shader bodies
                        //   by interleaving code segments ShadelGLES3::vertex_codeX and CustomCode segments
                        // Construct cache key from GL vendor, renderer, version, shader code
                        if key hit cache :
                            return from cache
                        else :
                            // glCreateShader() glShaderSource(), for vertex and fragment
                        // add Version.version (i.e. the conditional bitflags) to the Set CustomCode.versions

                    ShaderGLES3::_process_program_state() // ad-hoc loop-switch to handle different states of compilation and potential async compilation
                        ShaderGLES3::_complete_compile()
                            glAttachShader()
                            for each attribute : glBindAttribLocation
                            glLinkProgram()
                        ShaderGLES3::_complete_link()
                    glUseProgram(version.ids.main)
                    ShaderGLES3::_setup_uniforms()
                        // uniforms, textures, uniform blocks
                    // write this as the global ShaderGLES3::active shader
                // if shader was changed : increment shader_rebind_count
                // bind material UBO
                // N x active texture unit and bind texture
            // increment rebind count

        if shaded && not shadow && ...
            RasterizerSceneGLES3::_setup_light() // TODO dig

        if changed(instance.base_type) || changed(geometry) ...
            RasterizerSceneGLES3::_setup_geometry()
                if blend_shapes:
                    RasterizerStorageGLES3::mesh_render_blend_shapes()
                        // use transform feedback
                glBindVertexArray()
            // increment surface_switch count

        RasterizerSceneGLES3::_set_cull() // control GL culling

        RasterizerSceneGLES3::_render_geometry()
            glDrawElements() / glDrawElementsInstanced()
            // more advanced logic for INSTANCE_IMMEDIATE or INSTANCE_PARTICLES
            // increment vertices count

        // Update all local prev_ values, for comparison during next iteration

```


## Data model

The `VisualServer` instance is retrieved via `VisualServer::getSingleton()`,
its concrete type is a `VisualServerWrapMT` wrapping a `VisualServerRaster`.

Other high level object are accessed as global via static members in `VisualServerGlobals`:
* `RasterizerStorage`
* `RasterizerCanvas`
* `RasterizerScene`
* `Rasterizer`
* `VisualServerCanvas`
* `VisualServerViewport`
* `VisualServerScene`

`VisualServer` operations (such as `draw()`) are then implemented by accessing those globals.

### High level objects

```
struct RasterizerStorageGLES3
{
    struct Config
    {
        // GPU capacities and max values (e.g. max texture image units), supported extensions
        // Hight level options such as depth_prepass, anisotropic level, force vertex shading
    } config

    struct Shaders
    {
        // instance of shaders: Copy, CubemapFilter, BlendShape, Particles
        // plus ShaderCompiler, ShaderChach, queues, ShaderCompiler::IdentifierActions
    } shaders

    struct Info
    {
        // memcount Texture and Vertex
        struct Render
        {
            // counts (vertices, object, drawcall, material switch, surface switch, ...)
        } render, render_final, snap
    } info

    struct Frame
    {
        RenderTarget * current_rt
        // Clear request flag and clear color
        // Time and frame count?
    } frame
};
```

```
struct RasterizerSceneGLES3
{
    struct Environment
    {
        // sky, ambient color, ssr, ssao, glow, dof, fog
    };
    RID_Owner<Environment> environment_owner;

    struct LightDataUBO
    {
        // Many parameters to control lights
    }

    struct LightInstance
    {
        // shadow_transform
    }
    RID_Owner<LightInstance> light_instance_owner

    struct RendererList
    {
        struct Element
        {
            // InstanceBase *, Geometry *, Material *, GeometryOwner *
            uint64_t sort_key
        }
        Element ** elements
    } render_list

    struct ReflectionProbeInstance
    {}
    RID_Owner<ReflectionProbeInstance> reflection_probe_instance_owner

    struct ReflectionProbeDataUBO
    {}

    struct State
    {
        // Current setup sub-state of the GL server (blend mode, line_width, depth_test...)

        // Shaders: SceneShaderGLES3, cubeToDp, Resolve, SSR, EffectBlur, SSS, SSAO, Exposure, Tonemap

        struct SceneDataUBO
        {
            // projection, camera, ambient, time, resolution, shadow pixel size, fog params...
        } ubo_data

        struct EnvironmentRadianceUBO
        {
            // transform, ambient_contribution
        } env_radiance_data

        GLuint env_radiance_ubo

        // GLuint names for UBOs: directional, spot_array, omni_array, reflection_array, ...
        // byte buffers for light data (to transfer into LightDataUBO)
        // GLuint names for other shared resources (e.g. sky vertex buffer and VAO)
    }
};
```

```
struct VisualServerRaster : public VisualServer
{
    // Texture, sky, shader, common material, mesh, multimesh, interpolation, immediate,
    // skeleton, light, reflection probe, GI probe, particles, camera, viewport, environment,
    // interpolation, scenario, instancing, portal, roomgroups, occluders, canvas(2D), black bars
    // even queuing, status info, testing & misc (boot image, clear color, time scale)

    draw() (in event queuing)
}
```

```
struct VisualServerScene
{
    struct InterpolationData
    {
        // double buffer transform_update_lists for instance and camera
        // teleport list for instance and camera
        bool interpolation_enabled
    } _interpolation_data

    struct Camera
    {}
    RID_Owner<Camera> camera_owner

    struct Scenario
    {
        SpatialPartitioningScene * sps;
        RID environment;
        // list of directional lights
        // selflist of Instance
    };
    RID_Owner<Scenario> scenario_owner;

    struct Instance // see below
    RID_Owner<Instance> instance_owner;
}
```

```
struct VisualServerViewport {
    struct Viewport
    {
        RID scenario
        RID camera
        RID render_target
        RID shadow_atlas
        // a lot of members
    }

    RID_Owner<Viewport> viewport_owner;
};
```

### Instancing

[(rasterizer.h L86)](https://github.com/godotengine/godot/blob/3.5/servers/visual/rasterizer.h#L86)
```
struct RasterizerScene::InstanceBase : RID_Data
{
    // type (mesh, multimesh, light, particle, probe, ...)
    // IDs to skeleton, material override, material overlay
    // transform
    // depth layer, layer mask
    // Vectors materials, light instances, probes
    // Flags visual: cast shadows, receive shadows, mirror, visible, baked light
    // Flags regarding interpolation
    // depth
}
```

[(visual_server_scene.h L293)](https://github.com/godotengine/godot/blob/3.5/servers/visual/visual_server_scene.h#L293)
```
struct VisualServerScene::Instance : public RasterizerScene::InstanceBase
{
    // AABB
    // LOD
    // Occlusion handle
    // spatial partition ID & scenario

    InstanceBaseData * base_data
};
```

`InstanceBaseData` is a common empty type that is specialized base on the `BaseInstance` type,
e.g. `InstanceGeometryData`, `InstanceLightData`, ...

### Geometry related

[(rasterizer_storage_gles3.h L728)](https://github.com/godotengine/godot/blob/3.5/drivers/gles3/rasterizer_storage_gles3.h#L728)
```
struct RasterizerStorageGLES3::Mesh : public RasterizerStorageGLES3::GeometryOwner
{
  // active
  // vector of Surface *
  // list of multimesh
};
```

[(rasterizer_storage_gles3.h L226)](https://github.com/godotengine/godot/blob/3.5/drivers/gles3/rasterizer_storage_gles3.h#L226)
```
struct RasterizerStorageGLES3::GeometryOwner : public RasterizerStorageGLES3::Instantiable
{};
```

[(rasterizer_storage_gles3.h L201)](https://github.com/godotengine/godot/blob/3.5/drivers/gles3/rasterizer_storage_gles3.h#L201)
```
struct RasterizerStorageGLES3::Instantiable
{
   // list of RasterizerScene::InstanceBase
};
```

[(rasterizer_storage_gles3.h L643)](https://github.com/godotengine/godot/blob/3.5/drivers/gles3/rasterizer_storage_gles3.h#L643)
```
/// \brief Looks like the base drawable geometry
struct RasterizerStorageGLES3::Surface : public RasterizerStorageGLES3::Geometry
{
    // Array of attribs (shader attributes parameters, index, size, type, stride offset)
    // GL ids to array (VAO), instancing array, vertex, and index
    // separate GL ids for wireframe (but not for vertex)
    // primitive type
    // bool active
};
```

[(rasterizer_storage_gles3.h L229)](https://github.com/godotengine/godot/blob/3.5/drivers/gles3/rasterizer_storage_gles3.h#L229)
```
struct RasterizerStorageGLES3::Geometry : public RasterizerStorageGLES3::Instantiable
{
    // RID material
    // Index
};
```

### Material

[(rasterizer_storage_gles3.h L570)](https://github.com/godotengine/godot/blob/3.5/drivers/gles3/rasterizer_storage_gles3.h#L570)
```
struct RasterizerStorageGLES3::Material
{
    Shader * shader
    // ubo id & size
    // map: param name -> variant
    Vector<RID> textures
    int render_priority
    RID next_pass // seems to be the ID to a "next material", providing a new pass with its shader

    // last_pass, set to render_pass during _add_geometry_with_material(). TODO what is it for?
    // render_priority, used to sort. TODO what for?
}
```

[(rasterizer_storage_gles3.h L413)](https://github.com/godotengine/godot/blob/3.5/drivers/gles3/rasterizer_storage_gles3.h#413)
```
struct RasterizerStorageGLES3::Shader
{
    // mode (SPATIAL, CANVAS_ITEM, PARTICLES)
    ShaderGLES3 * shader
    String code
    // map: name -> Uniform

    // Specialized structs per mode
    struct Spatial
    {
        bool uses_world_coordinates
        // lot of other toggles
    }
}
```

[(shader_gles3.h L55)](https://github.com/godotengine/godot/blob/3.5/drivers/gles3/shader_gles3.h#L55)
```
ShaderGLES3
{
    static ShadlerGLES3 * active;

    union VersionKey
    {
        struct {
            uint32_t version; // bitflags for SceneShaderGLES3::Conditionals, controlled via set_conditional().
            uint32_t code_version; // key into custom_code_map, to retrieve CustomCode for this VersionKey.
                                   // set to the current version of a given CustomCode via set_custom_shader()
        }
        uint64_t key
    }

    VersionKey conditional_version // TODO: confirm if currently active (bound) version ?
    VersionKey new_conditional_version // target for modifications from set_conditionals()

    struct Version
    {
        VersionKey version_key

        struct Ids {
            // GLuint OpenGL names of main program, vertex and fragment shaders
        } ids

        uint32_t code_version // the CustomCode.version
    }
    Version * version

    _bind()
    {
        // assign new_conditional_version to conditional version
        // glUseProgram(version.ids.main)
        // _setup_uniforms(CustomCode associated to conditional_version.code_version)
        // assign `this` to `active`
    }

    CustomCode
    {
        uint32_t version // incremented each time new custom code is provided via set_custom_shader_code()
                         // i.e. this is the actual version of the custom code
        Set<uint32_t> versions // all VersionKey.version (conditional bitflags) that were
                               // get_current_version() with this custom code
    }

    HashMap<uint32_t, CustomCode> custom_code_map // I think it is used as a plain array
    uint32_t last_custom_code // incremented each time client request new custom code

    ProgramBinary
    {
        // byte array of compiled shader, and GLenum format
    } program_binary

    const char * vertex_code, * fragment_code
    CharString vertex_code0..1, fragment_code0..4 // assembled in ShaderGLES3::setup()
                                                  // from static array such as SceneShaderGLES3::_vertex_code
}
```
