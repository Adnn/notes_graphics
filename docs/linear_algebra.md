**Rigid-body transform**: Preserves length, angles, and handedness.

For an **orthogonal matrix**: $\mathbf{M}^{-1} = \mathbf{M}^{T}$

$$
(\mathbf{A} \times \mathbf{B})^{-1} =
\mathbf{B}^{-1} \times \mathbf{A}^{-1}
$$

$$
(\mathbf{A} \times \mathbf{B})^T =
\mathbf{B}^T \times \mathbf{A}^T
$$

The _determinant_ of matrix $\mathbf{M}$ is noted $|\mathbf{M}|$.

$$\DeclareMathOperator{\adj}{adj}$$
_Adjoint of a matrix_ $\adj(\mathbf{M})$ can be computed from
the _cofactors_ (or _subdeterminants_) $d_{ij}^M$ of said matrix.

The cofactor $d_{ij}^M$ is the determinant of the submatrix
obtained by removing row $i$ and column $j$ of $\mathbf{M}$,
e.g. (with $\mathbf{M}$ a 3x3 matrix):

$$
d_{02}^M =
\begin{vmatrix}
    m_{10} & m_{11} \\
    m_{20} & m_{21}
\end{vmatrix}
$$

Then, the components $a_{ij}$ of the adjoint are:
$$
[a_{ij}] = [(-1)^{(i+j)} d_{ij}^M]
$$

The _inverse_ of a matrix can be computed as
$\mathbf{M}^{-1} = \frac{\adj(\mathbf{M})}{|\mathbf{M}|}$

**Note**: if the determinant is zero, the matrix is _singular_, and **the inverse does not exist**.

_afaiu_: A _basis_ is a set of vectors, and a _frame of reference_ is a basis + an origin.

# Transformations

## Translation

rigid-body, affine.

$$
\mathbf{T}(\vec{\mathbf{t}}) =
\begin{pmatrix}
    1 & 0 & 0 & 0 \\
    0 & 1 & 0 & 0 \\
    0 & 0 & 1 & 0 \\
    t_x & t_y & t_z & 1 \\
\end{pmatrix}
$$

$$
\mathbf{T}^{-1}(\vec{\mathbf{t}}) = \mathbf{T}(-\vec{\mathbf{t}})
$$


## Rotation

rigid-body, linear, orthogonal

It can be used as the _orientation matrix_ of an entity in space. It defines the object _orientation_: its directions for up and forward.

$$
\mathbf{R}_x(\phi) =
\begin{pmatrix}
    1 & 0 & 0 & 0 \\
    0 & \cos{\phi} & \sin{\phi} & 0 \\
    0 & -\sin{\phi} & \cos{\phi} & 0 \\
    0 & 0 & 0 & 1
\end{pmatrix}
$$
$$
\mathbf{R}_y(\phi) =
\begin{pmatrix}
    \cos{\phi} & 0 & -\sin{\phi} & 0 \\
    0 & 1 & 0 & 0 \\
    \sin{\phi} & 0 & \cos{\phi} & 0 \\
    0 & 0 & 0 & 1
\end{pmatrix}
$$
$$
\mathbf{R}_z(\phi) =
\begin{pmatrix}
    \cos{\phi} & \sin{\phi} & 0 & 0\\
    -\sin{\phi} & \cos{\phi} & 0 & 0\\
    0 & 0 & 1 & 0 \\
    0 & 0 & 0 & 1
\end{pmatrix}
$$

$$
\mathbf{R}^{-1}(\phi) = \mathbf{R}(-\phi) (= \mathbf{R}^{T}(\phi))
$$

$$
\det(\mathbf{R}) = 1
$$

For any 3x3 rotation matrix: $\DeclareMathOperator{\tr}{tr} \tr(\mathbf{R}) = 1 + 2\cos{\phi}$

Rotation around a point $\mathbf{p}$ not at the origin can be implemented by a compound transformation:
$$
\mathbf{T}(-\vec{\mathbf{p}})
\mathbf{R}
\mathbf{T}(\vec{\mathbf{p}})
$$

## Scaling

$$

\mathbf{S}(\mathbf{s}) =
\begin{pmatrix}
    s_x & 0 & 0 & 0 \\
    0 & s_y & 0 & 0 \\
    0 & 0 & s_z & 0 \\
    0 & 0 & 0 & 1 \\
\end{pmatrix}
$$

$$
\mathbf{S}^{-1}(\mathbf{s}) = \mathbf{S}(\frac{1}{s_x}, \frac{1}{s_y}, \frac{1}{s_z})
$$

Scaling is **uniform** (or **isotropic**) if all components of $\mathbf{s}$ are equal, it is otherwise **nonuniform** (**anisotropic**).

Alternatively, for a _uniform_ scaling of $s$:
$$
\mathbf{S'} =
\begin{pmatrix}
    1 & 0 & 0 & 0 \\
    0 & 1 & 0 & 0 \\
    0 & 0 & 1 & 0 \\
    0 & 0 & 0 & 1/s \\
\end{pmatrix}
$$

**Important**: This will scale positions, but not displacements (since $w = 0$), and will require homogenization to take place.

### Negative components

* A negative value on a single component of $\mathbf{s}$ gives a type of _reflection matrix_ called a _mirror matrix_.
* A negative value on exactly two components will rotate $\pi$ radians.

**Important**: Reflection matrices might require special treatment, since they change winding order of triangles.

A reflection matrix can be detected by the negative determinant of its upper left 3x3 submatrix.

## Change of (orthonormal only?) basis

Orthogonal (only for an orthonormal basis?).

Given the orthonormal, right-oriented vectors $\mathbf{f}^x, \mathbf{f}^y, \mathbf{f}^z$, in the reference basis, then:
$$
\mathbf{F} =
\begin{pmatrix}
    \mathbf{f}^x & 0 \\
    \mathbf{f}^y & 0 \\
    \mathbf{f}^z & 0 \\
    \mathbf{0} & 1 \\
\end{pmatrix}
$$

changes vectors expressed in the basis defined by $\mathbf{f}$ vectors to the reference basis. _afaiu_, it is equivalent to a rotation transform that would rotate the reference basis to align it on the new basis.

The inverse, transforming vector components from the reference frame to the new basis, is the transpose:

$$
\mathbf{F}^-1 =
\begin{pmatrix}
    \mathbf{f}^x & \mathbf{f}^y & \mathbf{f}^z & \mathbf{0} \\
    0 & 0 & 0 & 1
\end{pmatrix}
$$

**mnemonic** to remember where to put each vector $\mathbf{f}$ in the change of basis matrix:\
when we multiply $\mathbf{f}^x$ with $\mathbf{F}^-1$, we must find $(1, 0, 0)$.
Having $\mathbf{f}^x$ in the first column allows the $1$, since $\mathbf{f}^x \cdot \mathbf{f}^x = 1$,
and having orthogonal vectors in other columns allow the $0$s.

## Shearing

non-rigid, area-preserving, volume-preserving.

Shear matrices, noted $\mathbf{H}_{ji}(s)$ (with $i \neq j$), are distorting geometry,
changing coordinate $j$ in proportion to the coordinate $i$.

$j$ and $i$ are indicating the position of the $s$ factor in the matrix (by following mathematical convention that $i$ is row and $j$ column):
$$
\mathbf{H}_{xz}(s) =
\begin{pmatrix}
    1 & 0 & 0 & 0 \\
    0 & 1 & 0 & 0 \\
    s & 0 & 1 & 0 \\
    0 & 0 & 0 & 1 \\
\end{pmatrix}
$$

$$
\mathbf{H}_{ji}^{-1}(s) = \mathbf{H}_{ji}(-s)
$$

$$|\mathbf{H}| = 1 \text{ (so it is volume-preserving)}$$

Another kind of shear matrix is shearing two coordinates (as subscripts) by the third coordinate (implicit):

$$
\mathbf{H}_{xy}'(s,t) =
\begin{pmatrix}
    1 & 0 & 0 & 0 \\
    0 & 1 & 0 & 0 \\
    s & t & 1 & 0 \\
    0 & 0 & 0 & 1 \\
\end{pmatrix}
$$

$$
\mathbf{H}_{j_aj_b}'(s,t) =
\mathbf{H}_{j_ai}(s) \mathbf{H}_{j_bi}(t)
\text{, where i is the implicit third coordinate}
$$

## Change of frame of reference

E.g. used to implement a look-at matrix.

This can be implemented as a composite transform,
first translating so the new frame origin $\mathbf{o}$ coincide with the reference origin,
then changing to the new basis $\mathbf{u}, \mathbf{v}, \mathbf{w}$:

$$
\mathbf{M} =
\mathbf{T}(-\mathbf{o})\mathbf{F}^{-1} =
\begin{pmatrix}
    u_{x}                        & v_{x}                        & w_{x}                        & 0 \\
    u_{y}                        & v_{y}                        & w_{y}                        & 0 \\
    u_{z}                        & v_{z}                        & w_{z}                        & 0 \\
    -\mathbf{o} \cdot \mathbf{u} & -\mathbf{o} \cdot \mathbf{v} & -\mathbf{o} \cdot \mathbf{w} & 1
\end{pmatrix}
$$

## Concatenation of transformations

Matrix multiplication is non-commutative, so concatenation of transforms is _order-dependent_, but it is _associative_ $\mathbf{MNO} = (\mathbf{MN})\mathbf{O}$.

$$
\mathbf{C} = \mathbf{SRT}
\newline
\text{ or in a pre-multiply framework: }
\mathbf{C} = \mathbf{TRS}
$$

## Rigid-body transform

Concatenations of only translations and rotations transformations are resulting in rigid-body transforms.

Any rigid-body transform $\mathbf{B}$ can be written as the concatenation of a rotation followed by a translation:
$$
\mathbf{X} =
\mathbf{R}\mathbf{T}(\vec{\mathbf{t}}) =
\begin{pmatrix}
    r_{00} & r_{01} & r_{02} & 0 \\
    r_{10} & r_{11} & r_{12} & 0 \\
    r_{20} & r_{21} & r_{22} & 0 \\
    t_x & t_y & t_z & 1 \\
\end{pmatrix}
$$

And the inverse is $\mathbf{X}^{-1} = \mathbf{T}(-\vec{\mathbf{t}})\mathbf{R}^T$

Or with $\bar{\mathbf{R}} = \begin{pmatrix}r_{,0} & r_{,1} & r_{,2}\end{pmatrix}$,
i.e. the upper-left 3x3 matrix representing the rotation:
$$
\mathbf{X} =
\begin{pmatrix}
    \bar{\mathbf{R}} & \mathbf{0}^T \\
    \vec{\mathbf{t}} & 1
\end{pmatrix}
$$

The inverse can be written:
$$
\mathbf{X}^{-1} =
\begin{pmatrix}
    \bar{\mathbf{R}}^T & \mathbf{0}^T \\
    - \vec{\mathbf{t}} \bar{\mathbf{R}}^T & 1
\end{pmatrix}
$$

## Normal transform

The same transform matrix can transform positions (points, lines, triangles, etc.) and tangent vectors.
Normals cannot use the same matrix in presence of non-uniform scaling (? other conditions).

* A traditional method for transforming the normal is to use the transpose of the inverse
$(\mathbf{M}^{-1})^T$ (does not preserve unit length). But the full inverse is not necesarry (and does not exist if $|\mathbf{M}| = 0$).
* The proper method is to use the transpose of the matrix's adjoint
$\adj(\mathbf{M})^T$ (does not preserve unit length).
  * Since the normal is a vector, it ignores translation, so when the _modeling transform_ is affine (not changing homogeneous $w$, i.e. no projection),
only the adjoint of the upper 3x3 submatrix is needed.
* When the transform matrix is composed exclusively by concatenation of translations, rotations and **uniform** scaling: the original transform can be used.
(Scalar division by the uniform scale factor maintains the norm. This factor might be embedded in the normal transform matrix so it produces normalized results).
  * Because the uniform scale changes the legnth of the normal, the translation has no effect, and the remaining transform is a net rotation, whose transpose of the inverse is the transpose of the transpose.
  * If there is no scaling, it means a unit normal will not need normalization after transform.

  ## Computing the inverse
  3 approaches:
  * If the matrix is a sequence of simple transforms of know parameter, the inverse can be computed by concatenating the inverses (seen above) in reverse order.
  * If the matrix is orthogonal (such as a sequence of rotations), the transpose is the inverse.
  * Use one of the numeric methods (adjoint method, Cramer's rule?, LU decomposition?, Gaussian elimination?)

  ## Orientation

  ### Euler angles

  Assuming that the oriented object local space is y-axis up, x-axis right, and minus z-axis front. With euler angles:
  * $h$ yaw (or head)
  * $p$ pitch
  * $r$ roll

$$
\mathbf{E}(h, p, r) =
\mathbf{R}_y(h) \mathbf{R}_x(p) \mathbf{R}_z(r)
$$

$$
\mathbf{E}(h, p, r)^{-1} = \mathbf{E}(h, p, r)^T =
\mathbf{R}_z^T(r) \mathbf{R}_x^T(p) \mathbf{R}_y^T(h)
$$

It is orthogonal

**Note**: For a given orientation, parameters are not unique.

_Gimbal lock_ occurs when a rotation around a given axis aligns 2 axeses: we lose one degree of freedom since the rotation around aligned axises are now reduced to a single resulting rotation.
