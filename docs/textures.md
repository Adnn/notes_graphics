
## Compression formats:

* ASTC (vbr)
  * Available in OpenGL via `GL_KHR_texture_compression_astc_hdr`.
* BC3 (Block Compression 3), also DXT5: good general-purpose copression for data requiring 4 channels.
* BC4 (Block Compresison 4) (4 bpp). Designed for single-channel textures.
  * Available from core OpenGL 3.0, and with `GL_ARB_texture_compression_rgtc` extension.
    * BC4u: `GL_COMPRESSED_RED_RGTC1`
    * BC4s: `GL_COMPRESSED_SIGNED_RED_RGTC1`
* BC5 (Block Compression 5), also 3Dc+, RGTC2 (Red-Green Texture Compression 2): high-quality 2-channels compression, specifically intended for normals.
  * Available from core OpenGL 3.0, and with `GL_ARB_texture_compression_rgtc` extension.
    * BC5u: `GL_COMPRESSED_RG_RGTC2`
    * BC5s: `GL_COMPRESSED_SIGNED_RG_RGTC2`
* BC7 (Block Compression 7) (8bpp), also BPTC (Block Partitioned Texture Compression).
  Specifically, BPTC is the specific compression technique used by BC7, which is the compression format (part of a set of texture compression format standardized by MS).
  BC7 is the DirectX nomenclature, while BPTC is OpenGL/general industry term.
  * Available from OpenGL 4.2 or `GL_ARB_texture_compression_bptc` extension.
    * `GL_COMPRESSED_RGBA_BPTC_UNORM`
    * `GL_COMPRESSED_SRGB_ALPHA_BPTC_UNORM`
* BC6 (Block Compression 6). Tailored for compressing HDR textures. Also part of BPTC family in OpenGL.
  * BC6H (Block Compression 6 HDR) (8 bpp) is the variant used in practice.
    * BC6H_UF16: Unsigned 16 bits floating-point, commonly used in HDR textures.
    * BC6H_SF16: Signed 16 bits floating-point, useful in some scenarios (e.g. scientific data).
  * Available from OpenGL 4.2 or `GL_ARB_texture_compression_bptc` extension.
    * `GL_COMPRESSED_RGB_BPTC_UNSIGNED_FLOAT`
    * `GL_COMPRESSED_RGB_BPTC_SIGNED_FLOAT`
* ETC2 (Ericson Texture Compression 2)
* S3TC (S3 Texture Compression), know as DXT in DirectX. Family of of lossy texture compression formats. DXT 1-5.
  * DXT 1-5:
      * DXT1: S3TC-1, BC1. RGB, optional 1-bit alpha
      * DXT3: S3TC-2, BC2. RGBA, explicit alpha
      * DXT5: S3TC-3, BC3. RGBA interpolated alpha
      * DXT2 and DXT4: not directly mapped to BC format and less commonly used. Similar tto DXT3 and DXT5 but handle premultiplied-alpha.
  * Available in OpenGL via `GL_EXT_texture_compression_s3tc`, which is an **ubiquitous extension**

### Recommended formats:

* Diffuse color map:
  * BC7: high-quality, broad support on modern hardware
  * ASTC: high quality, can handle HDR, not well supported on older systems
  * ETC2: Core in OpenGL ES 3.0, so widely supported on mobile and desktop, lower quality
* Normal map (tangent space):
  * BC5: high-quality, but requires reconstruction of third component in shader.
  * BC3: more-memory than BC5 and potentially more artifacts
  * ASTC: can be focused to preserve quality on specific channels, but support is not as wide.
* Ambient-occlusion, Roughness, Metalness:
  * BC7
  * BC3
  * ASTC

## Glossary

* ASTC: Adaptive Scalable Texture Compression
* BCn: Block Compression 1-7
* ORM/ARM texture: Ambient-occlusion (Red), roughness (Green), and metalness (Blue).

