# Verification status and numerical acceptance plan

**Status:** The repository was reviewed statically against `Final_code.py` and the checked-in input CSV files. The numerical solver was **not rerun or independently benchmarked** for this documentation update. Existing checks for mass-matrix symmetry/positive definiteness do not establish a physically correct FEM formulation.

## Code-level observations

| Area | Current implementation | Impact / recommended action |
|---|---|---|
| 2D shape-function derivatives | `compute_B_matrix_and_Jacobian()` constructs `B` using isolated inverse-Jacobian components and nonstandard strain rows | Correct chain-rule transformation and compare to analytical derivatives on a rectangle |
| Elastic stiffness | `E * B.T @ B` times thickness and quadrature weight | Define plane stress/strain and assemble the appropriate 3×3 constitutive matrix |
| Element mass | `rho * area * thickness * np.eye(8)` | Replace with consistent or correctly lumped mass and check total element mass |
| Support constraints | Insert artificial diagonal entries for fixed-node DOFs | Use exact free-DOF submatrices `K_ff`, `M_ff` before eigensolution |
| Numerical regularization | Adds `1e-10` to every diagonal mass entry | Avoid arbitrary regularization in a validated model; diagnose singularity instead |
| Frequency units | Prints `sqrt(eigenvalue)` as rad/s | Correct angular-frequency unit; optionally report `f=omega/(2*pi)` in Hz |
| Input paths | Hard-coded `C:/Users/karti/OneDrive/Desktop/RJ&SC` | Use a path relative to the script or an explicit user-selected input directory |
| Eigenmodes | Displays first three scaled modes via Matplotlib | Do not interpret mode-shape scale as physical displacement amplitude |
| Input values | `reinforced_concrete_properties.csv` contains additional values not used in assembly | Review unused entries/units before extending the material formulation |

## Proposed tests before reporting validated frequencies

1. **Shape-function consistency:** Check $\sum_i N_i=1$, $\sum_i\nabla N_i=0$, and a positive Jacobian determinant at all 2×2 Gauss points.
2. **Affine element patch test:** For a rectangular element, compare computed $\nabla N_i$ and strain-displacement interpolation with hand-calculated gradients.
3. **Stiffness verification:** Test symmetry, rigid-body null modes before boundary constraints, positive elastic energy after appropriate constraints, and a simple linear patch test.
4. **Mass verification:** Confirm $\mathbf 1^T\mathbf M_{xx}\mathbf 1=\rho At$ for each element; compare consistent and lumped formulations.
5. **Constrained eigenproblem:** Eliminate fixed degrees of freedom and check strictly positive, physically admissible eigenvalues with no artificial constraint modes.
6. **Eigenpair quality:** Report a normalized $\|K_{ff}\phi-\lambda M_{ff}\phi\|$ and mass orthogonality $\Phi^T M_{ff}\Phi\approx I$ after appropriate normalization.
7. **Mesh convergence:** Compare first natural frequencies across at least three spatial discretizations. State your convergence threshold before treating results as mesh-independent.
8. **Benchmark validation:** Compare to an analytically tractable beam/plate test or a reputable CAE package using matching units, dimensions, support conditions, and material assumptions.

No numerical pass/fail claims are made until these checks have been executed and saved with software versions, input files, mesh details and solver commit SHA.

## Presentation guidance

The model image and current interactive plots can illustrate the **workflow**. They should not be captioned as experimentally or analytically validated natural frequencies. This distinction makes the portfolio more credible to CAE and computational-mechanics reviewers.
