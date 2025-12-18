# **Project Master Plan: Spectral Solvers for Continuous Variables**

**Goal:** Benchmark three quantum algorithms (VQE/VQD, SSVQE, qEOM) on a particle in a well, using a custom **Sinc-DVR** discretization and **Gray Code** encoding to minimize noise.

---

### **Week 1: The "Physics Engine" (The Core Novelty)**

*This is the most unique part of your project. Qiskit cannot do this.*

#### **1.1. Sinc-DVR Matrix Generator**

* **Task:** Create the N \times N matrices for Kinetic Energy (T) and Potential Energy (V).
* **Action:** **CODE FROM SCRATCH.**
* **Why?** Qiskit has no `SincDVR` class. You must implement the specific mathematical formula: T_{ij} = \frac{\hbar^2}{2m \Delta x^2} \dots
* **Library Support:** Use `numpy` for creating the arrays.

#### **1.2. Gray Code Transformer**

* **Task:** Create a mapping function `index_to_gray(i)` that converts integer `2` to bitstring `011` (instead of binary `010`).
* **Action:** **CODE FROM SCRATCH.**
* **Why?** Standard computer science libraries use Binary. You need Gray Code to ensure adjacent grid points only differ by 1 bit flip (physics requirement).

#### **1.3. Hamiltonian to Pauli Converter**

* **Task:** Convert your N \times N matrix into a string of Pauli operators (e.g., 0.5 \times XII + 0.1 \times ZZI).
* **Action:** **CODE THE LOGIC / USE LIBRARY FOR STORAGE.**
* **Your Code:** Write the loop that iterates through your matrix, applies the Pauli decomposition formula, and outputs coefficients and strings.
* **Library (Qiskit):** Once you have the strings, load them into `qiskit.quantum_info.SparsePauliOp`.

---

### **Week 2: The Circuit & VQE Baseline**

*Building the "Car" that runs on the engine.*

#### **2.1. The Hamiltonian Variational Ansatz (HVA)**

* **The Concept:** Instead of random gates, you build the circuit by alternating "Kinetic Layers" and "Potential Layers" (Trotterization).
* **Layer 1:** e^{-i H_{potential} \theta_1}
* **Layer 2:** e^{-i H_{kinetic} \theta_2}
* **Status:** **100% YOU (Logic) / 0% Library (Template)**
* **CODE THIS:**
* **The Potential Layer:** In grid basis, the potential V(x) is diagonal. Use a layer of R_z gates.
* **The Kinetic Layer (The Hard Part):** The kinetic energy operator in Gray Code is complex. You must write the custom sub-circuit that implements e^{-i T \theta}.


* **USE LIBRARY:** Use Qiskit's `ParameterVector` to handle the \theta values.

#### **2.2. Multigrid VQE Loop**

* **Task:** Solve for the Ground State by starting with 2 qubits, optimizing, then upscaling to 3.
* **Action:** **CODE FROM SCRATCH.**
* **Why?** Qiskit has no concept of "resizing" a problem. You must write the Python loop that extracts parameters from the 2-qubit result and maps them to the 3-qubit ansatz.
* **Library (Qiskit/Scipy):** Use `Estimator` for energy and `scipy.optimize.minimize` for the optimization.

---

### **Week 3: The Solvers (The Benchmark)**

#### **3.1. Algorithm A: VQD (Variational Quantum Deflation)**

* **Task:** Find the 1st and 2nd excited states by penalizing overlap with the ground state.
* **Action:** **CODE THE LOOP.**
* **Your Code:** Write the cost function: Cost = E(\theta) + \beta |\langle \psi_0 | \psi(\theta) \rangle|^2.
* **Library:** Use Qiskit's `Sampler` or `ComputeUncompute` primitive to calculate the overlap |\langle \psi_0 | \psi(\theta) \rangle|^2.

#### **3.2. Algorithm B: SSVQE (Subspace Search VQE)**

* **Task:** Find all states simultaneously using orthogonal input states (e.g., |00\rangle, |01\rangle).
* **Action:** **CODE FROM SCRATCH.**
* **Your Code:** Write a cost function that sums the energies: Cost = w_0 \langle \psi_{00}|H|\psi_{00} \rangle + w_1 \langle \psi_{01}|H|\psi_{01} \rangle.

#### **3.3. Algorithm C: Custom qEOM (Quantum Equation of Motion)**

* **Task:** Calculate excited states by solving the generalized eigenvalue problem on the operator pool.
* **Your Code:**
1. **Operator Pool:** Define the operators (e.g., X, Y, Z on specific qubits) representing "momentum kicks."
2. **Commutator Logic:** Calculate the double commutators [H, [H, O_\mu]] for the qEOM matrices.


* **Library (NumPy):** Use `numpy.linalg.eig` to solve the final system.

---

### **Summary Table**

| Component | **CODE IT** (Your Brain/Novelty) | **IMPORT IT** (The Tool/Plumbing) |
| --- | --- | --- |
| **Hamiltonian** | Sinc-DVR math, Gray Code mapping, Decomposition loop. | `SparsePauliOp`, `numpy`. |
| **HVA Ansatz** | Manual Trotter layers (e^{-iV} and e^{-iT}) for Gray Code. | `QuantumCircuit`, `Rz`, `Parameter`. |
| **VQE (Ground)** | The "Multigrid" loop (resizing parameters). | `Estimator`, `scipy.optimize`. |
| **VQD (Algo A)** | The "Penalty" cost function and deflation loop. | `Sampler` (for overlap calc). |
| **SSVQE (Algo B)** | The "Weighted Sum" cost function. | `Estimator`, `Optimizer`. |
| **qEOM (Algo C)** | The Operator Pool and Commutator construction. | `numpy.linalg.eig`. |
