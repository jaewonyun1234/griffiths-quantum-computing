

# **Project Master Plan: Spectral Solvers for Continuous Variables**

**Goal:** Benchmark three quantum algorithms (VQE/VQD, SSVQE, qEOM) on a particle in a well, using a custom "Sinc-DVR" discretization and "Gray Code" encoding to minimize noise.

---

### **Week 1: The "Physics Engine" (The Core Novelty)**

*This is the most unique part of your project. Qiskit cannot do this.*

#### **1.1. Sinc-DVR Matrix Generator**

* **Task:** Create the N \times N matrices for Kinetic Energy (T) and Potential Energy (V).
* **Action:** **CODE FROM SCRATCH.**
* **Why?** Qiskit has no `SincDVR` class. You must implement the specific mathematical formula: T_{ij} = \frac{(-1)^{i-j}}{dx^2} \dots
* **Library Support:** Use `numpy` for creating the arrays.

#### **1.2. Gray Code Transformer**

* **Task:** Create a mapping function `index_to_gray(i)` that converts integer `2` to bitstring `011` (instead of binary `010`).
* **Action:** **CODE FROM SCRATCH.**
* **Why?** Standard computer science libraries use Binary. You need Gray Code to ensure adjacent grid points only differ by 1 bit flip (physics requirement).

#### **1.3. Hamiltonian to Pauli Converter**

* **Task:** Convert your N \times N matrix into a string of Pauli operators (e.g., `0.5 * XII + 0.1 * ZZI`).
* **Action:** **CODE THE LOGIC / USE LIBRARY FOR STORAGE.**
* **Your Code:** Write the loop that iterates through your matrix, applies the Pauli decomposition formula, and outputs coefficients and strings.
* **Library (Qiskit):** Once you have the strings, load them into `qiskit.quantum_info.SparsePauliOp`. **Do not** write your own class to multiply matrices.

---

### **Week 2: The Circuit & VQE Baseline**

*Building the "Car" that runs on the engine.*



#### **2.1. The Hamiltonian Variational Ansatz (HVA)**

* **The Concept:** Instead of random gates, you build the circuit by alternating "Kinetic Layers" and "Potential Layers" (Trotterization).
* Layer 1: e^{-i H_{potential} \theta_1}
* Layer 2: e^{-i H_{kinetic} \theta_2}


* **Status:** **100% YOU (Logic) / 0% Library (Template)**
* **THE TRAP (Why you can't use `qiskit.circuit.library.HamiltonianGate` blindly):**
* If you just ask Qiskit to "exponentiate this giant matrix," it will generate a massive, inefficient circuit full of CNOTs that destroys the structure.
* Qiskit *does* have an `HVA` class, but it is designed for standard models (Ising/Heisenberg). It does not know how to efficiently decompose your **Gray Code Kinetic Energy** operator.


* **CODE THIS (Your "Novelty"):**
* **The Potential Layer:** In grid basis, the potential V(x) is diagonal. This is easy: just a layer of R_z gates. You write the loop to place them.
* **The Kinetic Layer (The Hard Part):** The kinetic energy operator in Gray Code is complex (it involves interactions between distant qubits). You must write the custom sub-circuit that implements e^{-i T \theta}.
* *Note:* This proves you understand "Trotterization" and "Unitary Decomposition," which is a huge interview plus.

* **USE LIBRARY:**
* **`Parameter`:** Use Qiskit's parameter binding so you don't have to rebuild the circuit every optimization step.


#### **2.2. Multigrid VQE Loop**

* **Task:** Solve for the Ground State by starting with 2 qubits, optimizing, then upscaling to 3.
* **Action:** **CODE FROM SCRATCH.**
* **Why?** Qiskit has no concept of "resizing" a problem. You must write the Python loop that extracts parameters from the 2-qubit result and maps them to the 3-qubit ansatz.
* **Library (Qiskit/Scipy):** Use `Estimator` to calculate energy and `scipy.optimize.minimize` (L-BFGS-B or COBYLA) to find the minimum.

---

### **Week 3: The Solvers (The Benchmark)**

*Implementing the three competing methods.*

#### **3.1. Algorithm A: VQD (Variational Quantum Deflation)**

* **Task:** Find the 1st and 2nd excited states by penalizing overlap with the ground state.
* **Action:** **CODE THE LOOP (Avoid `qiskit_algorithms.VQD`).**
* **Why?** The built-in class is a "black box" that hides the penalty logic and won't support your Multigrid approach easily.
* **Your Code:** Write the cost function: Cost = E(\theta) + \beta \times |\langle \psi_0 | \psi(\theta) \rangle|^2.
* **Library:** Use Qiskit's `Sampler` or `ComputeUncompute` primitive to calculate the overlap |\langle \psi_0 | \psi(\theta) \rangle|^2.

#### **3.2. Algorithm B: SSVQE (Subspace Search VQE)**

* **Task:** Find all states simultaneously using orthogonal input states (|00\rangle, |01\rangle).
* **Action:** **CODE FROM SCRATCH.**
* **Why?** No standard class exists in Qiskit.
* **Your Code:** Write a cost function that sums the energies: Cost = w_0 \langle \psi_{00}|H|\psi_{00} \rangle + w_1 \langle \psi_{01}|H|\psi_{01} \rangle.

#### **3.3. Algorithm C: Custom qEOM (Quantum Equation of Motion)**

* **Task:** Calculate excited states by solving the generalized eigenvalue problem on the operator pool.
* **Action:** **CODE THE PHYSICS / USE LIBRARY FOR SOLVER.**
* **Your Code:**
1. **Operator Pool:** Define the operators (e.g., X, Y, Z on specific qubits) that represent "momentum kicks" in your grid.
2. **Commutator Logic:** Calculate the double commutators [H, [H, O_\mu]] needed for the qEOM matrices.


* **Why?** Qiskit's qEOM is hard-coded for Fermions (electrons). You are doing Bosons (grid particles).
* **Library (NumPy):** Once you build the M and V matrices, use `numpy.linalg.eig` to get the final answers.

---

### **Week 4: The Analysis (The "Data Engineer" Part)**

* **Task:** Run the benchmarks and plot the results.
* **Action:** **USE LIBRARIES.**
* **Library:** Use `matplotlib` for plotting "Error vs. Qubit Count". Use `pandas` to store your results tables.

---

### **Summary Table for Reference**

| Component | **CODE IT** (Your Brain) | **IMPORT IT** (The Tool) |
| --- | --- | --- |
| **Hamiltonian** | Sinc-DVR math, Gray Code logic, Matrix-to-Pauli conversion loop. | `SparsePauliOp`, `numpy` arrays. |
| Component | **CODE IT** (Your Brain) | **IMPORT IT** (The Tool) |
| --- | --- | --- |
| **HVA Ansatz** | **The Trotter Decomposition.** You manually build the layers: "Rotate Z for Potential, Rotate Custom-Basis for Kinetic." | `QuantumCircuit`, `Rz`, `Rxx` (if needed), `Parameter`. |

| **VQE (Ground)** | The "Multigrid" loop (resizing parameters). | `Estimator` (simulator), `scipy.optimize`. |
| **VQD (Algo A)** | The "Penalty" cost function loop. | `Sampler` (for overlap calc). |
| **SSVQE (Algo B)** | The "Weighted Sum" cost function. | `Estimator`, `Optimizer`. |
| **qEOM (Algo C)** | The Operator Pool, The Commutator construction. | `numpy.linalg.eig` (linear solver). |

This plan is safe. You are relying on Qiskit for the complex "plumbing" (simulating quantum mechanics, optimizing gradients), but you are building the "architecture" (the physics model and the algorithms) yourself.

**Are you ready to write the first script: `sinc_dvr.py`?**
