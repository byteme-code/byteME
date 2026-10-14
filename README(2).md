# [Team name] · Qiskit Fall Fest 2026 · Track 3: Quantum Simulation

| | |
|---|---|
| **Team** | [Team name] |
| **Members** | [Full name, email], [Full name, email] (as registered; replace with actual members) |
| **Track** | 3 · Quantum Simulation |
| **Level reached** | Intermediate |
| **Code files** | `intermediate.ipynb` |
| **Slides** | none |

## Summary

We simulated real-time dynamics of a 12-spin transverse-field Ising chain and studied how hardware-like noise affects the average magnetizations \(M_z(t)\) and \(M_x(t)\). Using Qiskit's `FakeFez` noise model, readout correction, and zero-noise extrapolation (ZNE), we compared noisy and mitigated results with an exact classical reference and noiseless Trotter evolution. Exponential ZNE after readout correction gave the lowest mean absolute error among the noisy methods tested.

## Approach

- **Model:** Open-chain transverse-field Ising Hamiltonian, \(H=-\sum_{i=1}^{N-1} Z_iZ_{i+1}-\sum_{i=1}^{N}X_i\), with \(N=12\), \(J=h=1\), and initial state \(|0\rangle^{\otimes 12}\).
- **Time evolution:** Product-formula (Trotter) circuits with final time \(T=2.0\), 8 steps, and \(\Delta t=0.25\). The noisy measurements were taken at steps \(k=2,4,6,8\).
- **Circuit depth:** First-order Trotterization was used for \(M_z\) and second-order Suzuki–Trotter for \(M_x\). Circuits were transpiled to a connected, low-error line of 12 physical qubits with optimization level 3.
- **Noise:** Qiskit Aer simulated the `FakeFez` backend noise model using 2,000 shots per circuit. No real QPU was used.
- **Mitigation:** Per-qubit readout calibration/correction and global circuit folding for noise scales 1 and 3. Zero-noise estimates used exponential or linear extrapolation to noise scale zero.

## Results

Mean absolute errors are averaged over the four noisy time points (\(t=0.5,1.0,1.5,2.0\)) and measured against the exact classical reference.

| Method | \(M_z\) mean error | \(M_x\) mean error |
|---|---:|---:|
| Noisy (raw) | 0.0308 | 0.0850 |
| Readout correction | 0.0285 | 0.0805 |
| ZNE exponential (raw) | 0.0188 | 0.0214 |
| Readout correction + ZNE linear | 0.0175 | 0.0309 |
| **Readout correction + ZNE exponential** | **0.0164** | **0.0160** |
| Noiseless Trotter | 0.0146 | 0.0082 |

The exact reference at \(t=2.0\) was \(M_z=0.0451\) and \(M_x=0.4722\). Exponential ZNE combined with readout correction improved both observables over the raw noisy results, especially \(M_x\). The noiseless Trotter curve remained the more accurate approximation overall.

## How we checked our results

- Compared the dynamics with a classical exact reference computed using sparse-matrix time evolution (`scipy.sparse.linalg.expm_multiply`) for the same 12-spin Hamiltonian and time points.
- Compared first- and second-order Trotter results and checked that the \(M_z\) curves agree to numerical precision for the chosen model.
- Ran an ideal Aer simulation to validate measurement bit order, the \(X\)-basis rotation, and circuit folding.
- Evaluated mean and maximum absolute errors for each noisy/mitigated method at \(t=0.5,1.0,1.5,2.0\).

## Hardware runs

None. All noisy runs used the Qiskit Aer simulator configured with the `FakeFez` backend noise model; no QPU job IDs or QPU seconds apply.

## Limitations

- ZNE uses only two noise scales (1 and 3), so extrapolation can be sensitive to statistical fluctuations and the assumed fit model.
- Readout correction uses per-qubit calibration and does not model correlated readout errors.
- Results are from a fake-backend simulator, not real quantum hardware.
- The noisy curves are sampled at four nonzero time points, while the exact reference is evaluated more densely for plotting.
- Larger systems, longer times, more shots, and additional noise scales increase simulation cost. Next steps would be to test additional noise scales, compare mitigation strategies, and validate on a real QPU if access is available.

## How to run

- **Python version:** Python 3.x (tested package versions are listed below; the notebook does not pin a Python version).
- **Install:**
  ```bash
  pip install "qiskit>=1.2" qiskit-aer qiskit-ibm-runtime scipy matplotlib nbformat
  ```
- **Run:** Open `intermediate_cells(2).ipynb` in Jupyter Notebook or JupyterLab and run the cells from top to bottom. Set `QUICK = True` in the settings cell for a faster smoke test (6 spins, 500 shots); leave `QUICK = False` for the reported 12-spin, 2,000-shot run. The full run took about 7 minutes in the recorded environment; runtime varies with hardware and software versions.
- **Recorded package versions:** Qiskit 2.5.2, Qiskit Aer 0.17.2, Qiskit IBM Runtime 0.50.0, SciPy 1.18.1.

## Credits

- *Track tutorial:* Adapted from the BasQ Qiskit Fall Fest 2026 tutorials by Benjamin Tirado (text CC BY 4.0, code Apache 2.0). The TFIM Hamiltonian and Trotter-circuit helper functions were reused from the tutorial; the additional analysis was developed for this challenge.
- Research papers:&#x20;

  Assi, I., Vogl, M., Kumari, M., & LeBlanc, J. (2025). Beyond trotterization: Variational product formulas for quantum simulation. Physical Review B. [https://doi.org/10.1103/941r-ql3g](https://doi.org/10.1103/941r-ql3g)

  Avtandilyan, A. A., & Pogosov, W. (2024). Optimal-order Trotter–Suzuki decomposition for quantum simulation on noisy quantum computers. Quantum Information Processing, 24. [https://doi.org/10.1007/s11128-024-04627-z](https://doi.org/10.1007/s11128-024-04627-z)

  Bosse, J. L., Childs, A. M., Derby, C., Gambetta, F., Montanaro, A., & Santos, R. A. (2024). Efficient and practical Hamiltonian simulation from time-dependent product formulas. Nature Communications, 16. [https://doi.org/10.1038/s41467-025-57580-5](https://doi.org/10.1038/s41467-025-57580-5)

  Cervera-Lierta, A. (2018). Exact Ising model simulation on a quantum computer. Quantum, 2, 114. [https://doi.org/10.22331/q-2018-12-21-114](https://doi.org/10.22331/q-2018-12-21-114)

  Dreyer, H., Bejan, M., & Granet, E. (2021). Quantum computing critical exponents. Physical Review A. [https://doi.org/10.1103/physreva.104.062614](https://doi.org/10.1103/physreva.104.062614)

  Kirmani, A., Pelofske, E., Bärtschi, A., Eidenbenz, S., & Zhu, J.-X. (2025). Variational Quantum Simulations of a Two-Dimensional Frustrated Transverse-Field Ising Model on a Trapped-Ion Quantum Computer. [https://doi.org/10.48550/arxiv.2505.22932](https://doi.org/10.48550/arxiv.2505.22932)

  Layden, D. (2021). First-Order Trotter Error from a Second-Order Perspective.. Physical review letters, 128 21, 210501. [https://doi.org/10.1103/physrevlett.128.210501](https://doi.org/10.1103/physrevlett.128.210501)

  Le, I. N. M., Sun, S., & Mendl, C. (2025). Riemannian quantum circuit optimization based on matrix product operators. Quantum, 9, 1833. [https://doi.org/10.22331/q-2025-08-27-1833](https://doi.org/10.22331/q-2025-08-27-1833)

  Lee, Y. (2023). Symmetric Trotterization in digital quantum simulation of quantum spin dynamics. Journal of the Korean Physical Society, 82, 479-485. [https://doi.org/10.1007/s40042-023-00722-z](https://doi.org/10.1007/s40042-023-00722-z)

  Lee, S.-J., Ahn, K., Cheon, S., Park, M. J., & Kim, Y. (2026). Constant-Depth Multi-Product Formula for Trotter Error Mitigation in Near-Term Digital Quantum Simulation. [https://doi.org/10.48550/arxiv.2607.12225](https://doi.org/10.48550/arxiv.2607.12225)

  Pelofske, E., Bärtschi, A., & Eidenbenz, S. (2023). Simulating Heavy-Hex Transverse Field Ising Model Magnetization Dynamics Using Programmable Quantum Annealers. [https://doi.org/10.48550/arxiv.2311.01657](https://doi.org/10.48550/arxiv.2311.01657)

  Pillay, S., David, I. J., Sinayskiy, I., & Petruccione, F. (2026). Optimising Trotter-Suzuki simulations of Markovian open quantum systems via classical search. Quantum Information Processing, 25. [https://doi.org/10.1007/s11128-026-05267-1](https://doi.org/10.1007/s11128-026-05267-1)

  Prayaga, C. S. (2026). From Exact Diagonalization to DMRG: A Complete Numerical Study of the Transverse-Field Ising Model. [https://doi.org/10.48550/arxiv.2607.28471](https://doi.org/10.48550/arxiv.2607.28471)

  Sharma, R. (2026). Quantum Phase Transitions in the Transverse-Field Ising Model: A Comparative Study of Exact, Variational, and Hardware-Based Approaches. [https://doi.org/10.48550/arxiv.2601.17515](https://doi.org/10.48550/arxiv.2601.17515)

  Tripathi, A., Mathur, N., & Tripathi, V. (2026). Ans"atz Expressivity and Optimization in Variational Quantum Simulations of Transverse-field Ising Model Across System Sizes. [https://doi.org/10.48550/arxiv.2604.20961](https://doi.org/10.48550/arxiv.2604.20961)
- *AI tools used:* Claude for coding, chatgpt for documentation, Consensus AI for research paper assistance.&#x20;