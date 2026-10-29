# Finite-element formulation and mapping to the current solver

This note separates the **standard structural mechanics formulation** from the **expressions currently assembled by `Final_code.py`**. It is not a claim that the current implementation has been verified.

## 1. Physical model

The repository models a two-dimensional, undamped, small-amplitude free-vibration problem. The model contains 32 nodes, 15 four-node quadrilateral elements and two translational degrees of freedom at each node. Fixed-node IDs are `0`, `7`, `18` and `25`.

For an ideal linear-elastic system,

$$\mathbf M\ddot{\mathbf u}+\mathbf K\mathbf u=\mathbf 0.$$

Harmonic motion yields the symmetric generalized eigenproblem

$$\mathbf K_{ff}\boldsymbol\phi_i=\lambda_i\mathbf M_{ff}\boldsymbol\phi_i,\qquad
\omega_i=\sqrt{\lambda_i},\qquad f_i=\omega_i/(2\pi),$$

where `f` denotes unconstrained degrees of freedom and $\omega$ has units of rad/s.

## 2. Bilinear quadrilateral

For local coordinates $(\xi,\eta)\in[-1,1]^2$,

$$
\begin{aligned}
N_1&=\tfrac14(1-\xi)(1-\eta), & N_2&=\tfrac14(1+\xi)(1-\eta),\\
N_3&=\tfrac14(1+\xi)(1+\eta), & N_4&=\tfrac14(1-\xi)(1+\eta).
\end{aligned}
$$

With two in-plane components per node, the standard displacement interpolation matrix is

$$
\mathbf N_u=\begin{bmatrix}
N_1&0&N_2&0&N_3&0&N_4&0\\
0&N_1&0&N_2&0&N_3&0&N_4
\end{bmatrix}.
$$

The source builds the natural derivatives `N_dxi` and the Jacobian as
`J = N_dxi @ coords`, corresponding to a matrix whose rows are the natural-coordinate derivatives of $(x,y)`. The correct physical gradients for this convention satisfy

```python
dN_dxdy = np.linalg.solve(J, N_dxi)  # rows: dN/dx, dN/dy
```

The small-strain plane formulation then uses

$$
\mathbf B=\begin{bmatrix}
N_{1,x}&0&\cdots&N_{4,x}&0\\
0&N_{1,y}&\cdots&0&N_{4,y}\\
N_{1,y}&N_{1,x}&\cdots&N_{4,y}&N_{4,x}
\end{bmatrix}.
$$

**Current code:** `compute_B_matrix_and_Jacobian` assigns derivatives through individual entries of `invJ` instead of forming the full chain-rule transformation; it also places terms in inappropriate B-matrix components. The implementation should be corrected and unit-tested before using computed eigenfrequencies as reference values.

## 3. Elastic stiffness

For an isotropic *plane-stress* assumption, the standard constitutive matrix is

$$
\mathbf C=\frac{E}{1-\nu^2}
\begin{bmatrix}
1&\nu&0\\ \nu&1&0\\0&0&(1-\nu)/2
\end{bmatrix}.
$$

The corresponding element stiffness is

$$
\mathbf K_e=t\int_{-1}^1\int_{-1}^1
\mathbf B^T\mathbf C\mathbf B\,\det\mathbf J\,d\xi\,d\eta.
$$

The same general approach can be formulated for plane strain with a different constitutive matrix. The relevant assumption must be declared and tested.

**Current code:** `K_local += (B.T @ B) * young_modulus * thickness * detJ`. It does not assemble the isotropic constitutive matrix $\mathbf C$. Therefore, the source expression should not be presented as a mechanically validated plane-stress/plane-strain stiffness matrix.

## 4. Mass matrix

A consistent translational element mass matrix is

$$
\mathbf M_e=\rho t\int_{\Omega_e}\mathbf N_u^T\mathbf N_u\,d\Omega.
$$

For an equal-area four-node rectangular element, a conventional nodal lumped approximation allocates $\rho At/4$ to **each nodal translational component on the diagonal** (the total mass in each translation direction is $\rho At$).

**Current code:** `local_mass_matrix = density * area * thickness * np.eye(8)`. This allocates $\rho At$ to **each** of the eight component entries, substantially exceeding a conventional element's physical mass allocation. Correction and mass-conservation tests are required.

## 5. Boundary conditions and modal solve

The preferred constrained-system strategy is to form the free-DOF index set and solve

$$\mathbf K_{ff}\boldsymbol\phi_i=\lambda_i\mathbf M_{ff}\boldsymbol\phi_i,$$

then expand the eigenvectors to full-node arrays with zero values at constrained entries.

**Current code:** rows and columns for fixed nodes are zeroed and replaced with `K[ii,ii]=1` and `M[ii,ii]=1e-10`, followed by an additional diagonal regularization of `M`. The resulting generalized eigenproblem includes artificially introduced constrained-DOF eigenvalues and perturbed mass. It should be replaced by free-DOF reduction before quantitative modal analysis.

## 6. Scope

The equations in this document serve as a **verification and development reference**. They do not certify that the uploaded script already implements them. See [Verification](VERIFICATION.md) for planned tests.
