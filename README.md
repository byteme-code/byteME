# byteME · Qiskit Fall Fest 2026 · Track 3: Quantum Simulation

| **Field** | **Details** |
|---|---|
| **Team** | byteME |
| **Members** | Riddhi Singh [riddhi.official777@gmail.com], Dhruwaz Deswal [dhruwazofficial@gmail.com], Srishti Aggarwal [srishti2007aggarwal@gmail.com], Saurabh Yadav [saurabh.ydvconnect@gmail.com] |
| **Track** | Track 3 · Quantum Simulation |
| **Level Reached** | Intermediate (noisy simulator, no real QPU) |
| **Code Files** | `main.ipynb` (Beginner + helpers), `intermediate.ipynb` (Intermediate) |
| **Slides** | `slides.pdf` (up to 8 slides) |

## 1. Project Summary

We simulated the time evolution of a transverse-field Ising chain ($J=h=1$, initial state $|0\ldots0\rangle$, $T=2$) with Trotter circuits.

- **Beginner level:** 8 spins, noiseless, compared with the exact reference from `scipy.linalg.expm`.
- **Intermediate level:** 12 spins on an Aer simulator with the FakeFez noise model, with readout-error correction and zero-noise extrapolation (ZNE), compared with the exact reference from `expm_multiply`.

Readout correction combined with exponential ZNE gave the lowest error ($M_z$: 0.0308 → 0.0164, $M_x$: 0.0850 → 0.0160), close to the noiseless Trotter floor. No mitigation method was worse than the raw noisy result.

## 2. Approach

### Model

We simulate an open chain of spins governed by the Hamiltonian

$$
H=-J\sum_{i}Z_iZ_{i+1}
-h\sum_{i}X_i,
$$

with $J=h=1$, built as a `SparsePauliOp`, where $X_i$ and $Z_i$ denote Pauli operators acting on spin $i$. The initial state is $|\psi(0)\rangle=|0\ldots0\rangle$.

### Product Formulas

- **Lie–Trotter (`LieTrotter`):** first-order approximation to time evolution.
- **Suzuki–Trotter (`SuzukiTrotter`):** second-order product-formula approximation.

Circuits are decomposed so that Trotter error is present.

### Observables

- **Longitudinal magnetization**

$$
M_z(t)=\frac{1}{N}\sum_i\langle Z_i\rangle
$$

- **Transverse magnetization**

$$
M_x(t)=\frac{1}{N}\sum_i\langle X_i\rangle
$$

- **Spin–spin correlations** $\langle Z_0Z_r\rangle$ at $t=2$ for different distances $r$ (Beginner level).

### Beginner Level

- 8 spins, noiseless, $T=2$.
- We choose the largest $\Delta t$ with maximum error $\le 0.01$.
- Extra analysis: correlations, error scaling with $\Delta t$, and CX count.

### Intermediate Level

- **Settings:** $N=12$, $T=2$, $\Delta t=0.25$ (8 steps). This is a bigger step than Beginner's 0.17, because each extra step adds gates and noise.
- **Sampling:** noisy points at $t=0.5, 1.0, 1.5, 2.0$ with 2000 shots.
- **Depth reduction:** $M_z$ circuits use 1st order ($M_z$ is identical for both orders here, with about half the two-qubit gates). $M_x$ circuits use 2nd order. Circuits are transpiled with `optimization_level=3` onto the lowest-error line of 12 connected FakeFez qubits (physical qubits 141, 142, 143, 136, 123, 124, 125, 126, 127, 137, 147, 146).
- **Mitigation:**
  - Per-qubit readout correction (all-0 / all-1 calibration circuits).
  - ZNE with noise factors 1 and 3 (global folding $UU^\dagger U$), with linear and exponential fits.
- **Not done:** dynamical decoupling and Pauli twirling (see Limitations).

## 3. Results

### Beginner (8 spins, $T=2$, largest $\Delta t$ with max error $\le 0.01$)

| Formula | Observable | Largest $\Delta t$ | Steps $n$ | CX Gates | Max Error |
|---|---|---:|---:|---:|---:|
| Lie–Trotter (1st) | $M_z$ | 0.1667 | 12 | 168 | ≤ 0.01 |
| Lie–Trotter (1st) | $M_x$ | 0.0125 | 160 | 2240 | ≤ 0.01 |
| Suzuki–Trotter (2nd) | $M_z$ | 0.1667 | 12 | 314 | ≤ 0.01 |
| Suzuki–Trotter (2nd) | $M_x$ | 0.1667 | 12 | 314 | ≤ 0.01 |

### Intermediate (12 spins, mean |error| vs exact over $t=0.5, 1.0, 1.5, 2.0$)

| Method | $M_z$ Mean Error | $M_x$ Mean Error |
|---|---:|---:|
| Noisy (raw) | 0.0308 | 0.0850 |
| Readout correction | 0.0285 | 0.0805 |
| ZNE exponential (raw) | 0.0188 | 0.0214 |
| Readout + ZNE linear | 0.0175 | 0.0309 |
| Readout + ZNE exponential | 0.0164 | 0.0160 |
| Noiseless Trotter (no noise) | 0.0146 | 0.0082 |

### Depth Reduction ($k=8$, $M_z$ circuit, 12 qubits)

| Formula | Optimization Level | Two-Qubit Gates | Depth |
|---|---:|---:|---:|
| Order 1 | 1 | 176 | 256 |
| Order 1 | 3 | 176 | 194 |
| Order 2 | 1 | 338 | 1710 |
| Order 2 | 3 | 322 | 1083 |

Plots: 
exact vs noiseless vs noisy vs mitigated $M_z(t)$ and $M_x(t)$ (intermediate cell 12)- "graph_plots/intermediate/The_M_z(t)_M_x(t)_plot.png"

error vs time and circuit size (cell 13)- "graph_plots/intermediate/The_error_vs_time_and_circuit_size_plot.png"

## 4. Key Findings

### Observable-dependent efficiency

- **For $M_z$:** both formulas give identical errors at the same $n$, so Lie–Trotter is cheaper (fewer CX per step).
- **For $M_x$:** Suzuki–Trotter needs 12 steps instead of 160, so it is far cheaper.

### Why the observables behave differently

$M_z$ commutes with the $ZZ$ terms, and the initial state is an eigenstate of them, which hides the first-order error. $M_x$ does not commute with them and shows the error fully.

### Mitigation

Readout correction plus exponential ZNE performed best on both observables. Mitigated $M_z$ stays close to the noiseless Trotter floor (0.0164 vs 0.0146), so the remaining $M_z$ error is mostly Trotter error, not noise.

## 5. Validation

- **Exact reference:** `scipy.linalg.expm` (8 spins) and `expm_multiply` (12 spins, 4096-dim).
- **Helper cross-check:** helpers agree with `StatevectorEstimator` (max difference $4.7\times10^{-16}$).
- **Error scaling:** measured error slopes on 4 spins matched theory (1, 2, 4 for orders 1, 2, 4).
- **Ideal Aer run:** a noise-free Aer run of the same measured circuits, including folded ones, matches noiseless Trotter within shot noise (e.g. $M_z$ at $k=4$: 0.2485 vs 0.2526; $M_x$: 0.4490 vs 0.4538). This validates bit ordering, the X-basis rotation, and folding.
- **Order check:** assertion that $M_z$ is identical for 1st and 2nd order at $N=12$.
- **Error separation:** noiseless Trotter error is reported beside the noisy errors, so Trotter error and hardware-style noise are separated.

## 6. Hardware Execution

**No quantum hardware runs were performed.** All results are from the FakeFez noisy simulator (calibration snapshot bundled with `qiskit-ibm-runtime`). Everything was run locally on a personal laptop using a locally hosted JupyterLab.

## 7. Limitations and Future Work

- **Noise model:** the simulated noise is a model, not the real device.
- **Shot noise:** about ±0.01 on $M_z$ or $M_x$ at 2000 shots; ZNE amplifies it. We did not compute error bars.
- **Sampling:** only 4 time points and 2 noise factors (runtime). The two-point exponential fit is fixed to decay to 0.
- **DD and twirling:** dynamical decoupling and Pauli twirling were not implemented. We are unsure the Aer noise model includes idle-delay noise, so DD might show no benefit.
- **Folding:** folding is global, so idle/decoherence noise scales only approximately.
- **Next steps:** more time points, error bars, 3-point ZNE, DD and twirling, a real QPU run.

## 8. How to Run

### Requirements

- Python version: 3.12.9
- Packages (tested): qiskit 2.5.2, qiskit-aer 0.17.2, qiskit-ibm-runtime 0.50.0, scipy 1.18.1, numpy, matplotlib.

```bash
pip install numpy matplotlib scipy qiskit qiskit-aer qiskit-ibm-runtime
```

### Instructions

1. `main.ipynb` (Beginner) and `intermediate.ipynb` (Intermediate) are self-contained, since the helpers are copied into the intermediate notebook.
2. Run either one on its own, top to bottom.
3. The intermediate noisy run took about 7.5 minutes (435 s). Set `QUICK = True` in its settings cell for a fast smoke test.

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

---

**Team byteME** · Qiskit Fall Fest 2026 · Track 3: Quantum Simulation
