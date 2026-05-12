# Spin-wave ground state of the Kagome lattice Fermi-Hubbard model 
![Parameters as function of U](figure_result.png)

This project analytically derives the band structure and gradients of the Kagome lattice Fermi-Hubbard model using mean-field approximation and spin-wave assumptions. At $2/3$ filling, the magnetization $m_A,m_B,m_C$ and spin-wave vectors $q_{Ax},q_{Bx},q_{Cx},q_{Ay},q_{By},q_{Cy}$ of the ground state are iteratively solved by gradient descent, yielding the order parameters as functions of the interaction strength $U$.

---

## Physical background

The unit cell of the Kagome lattice contains three inequivalent atoms A, B, C. At the mean-field level, the Fermi‑Hubbard model can be parametrized by introducing magnetization order $m_i$ and spin-wave vectors $Q_i=(q_{ix},q_{iy})$. The non-interacting three bands $\varepsilon_i(\mathbf{k})$ are given by tight-binding. Including spin waves leads to six quasiparticle bands:

$$
E_i^\pm(\mathbf{k}) = \frac{1}{2}\big[\varepsilon_i(\mathbf{k}) + \varepsilon_i(\mathbf{k}+Q_i)\big] \pm \sqrt{\frac{1}{4}\big[\varepsilon_i(\mathbf{k}) - \varepsilon_i(\mathbf{k}+Q_i)\big]^2 + (U m_i)^2},
$$

where $i=A,B,C$, and $U$ is the interaction strength. The total energy is the sum over all occupied states:

$$
E_{\text{total}} = \sum_{\mathrm{occ}} E_n(\mathbf{k}) + 2U\sum_{i}m_i^2 .
$$

At $2/3$ filling, only $4N_{\text{cell}}$ of the $6N_{\text{cell}}$ single-particle states are occupied ($N_{\text{cell}}$ is the number of unit cells). Using analytically derived gradients, gradient descent simultaneously optimizes the dependence of the total energy on $m_i$ and $Q_i$ to obtain the ground-state order parameters.

The core problems solved by this code are:  
1. Analytically derive the partial derivatives of the six bands with respect to $m_i,q_{ix},q_{iy}$ and package them as array operations;  
2. Under the $2/3$ occupancy constraint, use gradient descent to find the parameter set that minimizes the total energy, and investigate the $U$-driven phase transition behavior.

---

## Numerical method

### 1. Analytical derivatives and discretization

- **Band expressions**:  
  The non-interacting bands $\varepsilon_i(\mathbf{k})$ ($i=A,B,C$) are evaluated on an $L\times L$ momentum grid. The shifted bands $\varepsilon_i(\mathbf{k}+Q_i)$ and their derivatives with respect to $Q_i$, $\partial\varepsilon_i/\partial q_{ix},\partial\varepsilon_i/\partial q_{iy}$, are also computed.

- **Six-band spectrum and gradients**:  
  $E_i^\pm(\mathbf{k})$ and its partial derivatives with respect to $m_i,q_{ix},q_{iy}$ are written analytically as three arrays `dEdm`, `dEdqx`, `dEdqy`, each of shape $(6,L,L)$, facilitating vectorized operations.

### 2. Gradient descent for the ground state

- **Initialization**:  
  For each $U$, initialize with $m_i = 0.1U$, $q_{ix}=q_{iy}=0.8\pi U/3$ plus small random noise. When scanning continuously using the result from the previous $U$, no special linking is needed.

- **Update iteration**:  
  1. Compute the six bands $E$ and their gradients for the current parameters.  
  2. Flatten all $6N_{\text{cell}}$ energy values, sort them in ascending order, and take only the lowest $4N_{\text{cell}}$ levels as occupied states.  
  3. Sum the gradients over the occupied states to obtain the gradient of the total energy with respect to $m_i$ and $Q_i$, including the additional $2U m_i$ contribution.  
  4. Update the parameters with a fixed learning rate $\alpha=0.1$:  

$$
m_i \leftarrow m_i - \alpha \frac{\partial E_{\text{total}}}{\partial m_i}, \quad q_{i\nu} \leftarrow q_{i\nu} - \alpha \frac{\partial E_{\text{total}}}{\partial q_{i\nu}}.
$$
  
- **Convergence criterion**:  
  Stop when the maximum update step among the three components is less than $10^{-4}$. No hard limit on the number of iterations; convergence typically occurs within tens to a few hundred steps.

### 3. Scanning and post-processing

Repeat the optimization process for a series of $U$ values (from $0.2$ to $6.4$ in steps of $0.1$), recording the converged $m_i,q_{ix},q_{iy}$. Finally plot the order parameters as functions of $U$ and mark the known transition points $U_{c1}\approx2.6$ and $U_{c2}\approx5.35$.

---

## Code structure

- Main program file (single script or Jupyter Notebook):  
  - Parameter settings: lattice size $L$, $U$ list, learning rate $\alpha$;  
  - Momentum-space basis vectors $\mathbf{b}_1,\mathbf{b}_2$ and grid generation;  
  - Calculation of the non-interacting bands $\varepsilon$ (three bands);  
  - Analytical expressions for the six-band spectrum $E$ and gradients `dEdm`, `dEdqx`, `dEdqy`;  
  - Main gradient descent loop: energy sorting, occupied state selection, gradient accumulation, parameter update;  
  - Storage arrays `m_g`, `qx_g`, `qy_g`;  
  - Plotting and data saving with `np.savez`.

---

## Dependencies

The following Python libraries are required to run the code:

- `numpy`
- `matplotlib`

Python 3.7 or higher is recommended.

---

## Quick start

1. **Clone the repository** (or download the files):
   ```bash
   git clone https://github.com/chaoranyang/QuantumManyBody_FromZreo.git
   cd QuantumManyBody_FromZreo/kagomeFH_Grad
2. **Dependencies:**
    ```bash
    pip install numpy matplotlib
---

## Output results

- **Order parameters as functions of $U$**:  
  Three subplots showing the evolution of $m_i$, $q_{ix}$, $q_{iy}$ with interaction strength $U$.

  The first subplot shows that magnetization starts to appear spontaneously at the critical $U_{c1}\approx2.6$;  
  The second subplot shows that $q_{ix}$ jumps to a plateau near $2.199$ at $U_{c1}$, and changes again at $U_{c2}\approx5.35$;  
  The third subplot shows the behavior of $q_{iy}$, consistent with the symmetry of the system.

  The numerical results clearly reproduce the phase transition features of the spin-density wave state in the Kagome Hubbard model.

## References
- **Patrik Fazekas**, *Lecture Notes on Electron Correlation and Magnetism*, in *Series in Modern Condensed Matter Physics*, Vol. 5, World Scientific, 1999, pp. 1-796.
