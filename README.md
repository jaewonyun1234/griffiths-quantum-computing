Here is your fully revised, "Hiring Manager Proof" Master Plan.

I have integrated the **HVA Truncation**, the **HEA Comparison**, and the **Noise Benchmarking** into a rigorous engineering roadmap. You can save this directly as `README.md` or `project_plan.md`.

---

# **Project Master Plan: Benchmarking Quantum Encodings & Ansatz Strategies for Continuous Variables**

**Objective:** Investigate the trade-offs between **Accuracy** (Sinc-DVR basis) and **Circuit Depth** (Noise Resilience) by benchmarking **Gray Code** vs. **Binary** encodings and **Physics-Informed** vs. **Hardware-Efficient** ansätze.

**Target System:** 1D Particle in a Double Well Potential (Tunneling Analysis).

---

### **Week 1: The "Physics Engine" & Hamiltonian Engineering**

*Goal: Build the control group and the experimental group Hamiltonians.*

#### **1.1. Sinc-DVR Matrix Generator (The "Exact" Operator)**

* **Task:** Implement the Sinc-DVR Kinetic (T) and Potential (V) matrices.
* **The Math:**
* Potential: V_{ij} = \delta_{ij} V(x_i) (Diagonal).
* Kinetic (Dense): 



* **Action:** **CODE FROM SCRATCH.**
* **The Twist:** Implement a **Double Well Potential** (V(x) = ax^4 - bx^2) to study tunneling splitting, which is harder than a simple square well.

#### **1.2. The "Truncator" (The Circuit Optimizer)**

* **The Problem:** The T matrix is dense (all-to-all connectivity). Implementing e^{-iT\theta} exactly requires O(N^2) gates, which is too deep for NISQ.
* **The Solution:** Create a function `truncate_hamiltonian(matrix, threshold)` that filters out long-range Kinetic interactions (where |i-j| > 1).
* **Use Case:** You will use the **Full Matrix** for energy measurement (Accuracy), but the **Truncated Matrix** for constructing the HVA Circuit (Speed).

#### **1.3. Encoding Manager (The A/B Test)**

* **Task:** Create a class that converts the N \times N matrix into Pauli Strings using two methods:
1. **Standard Binary** (Control Group): 0 \to 00, 1 \to 01, 2 \to 10, 3 \to 11.
2. **Gray Code** (Experimental Group): 0 \to 00, 1 \to 01, 2 \to 11, 3 \to 10.


* **Hypothesis:** Gray Code should minimize the Hamming weight of the truncated Kinetic operator, reducing the number of CNOTs required for nearest-neighbor interactions.

---

### **Week 2: The "Ansatz Battle" (Circuit Architecture)**

*Goal: Compare "Compression" (HEA) vs. "Physics" (HVA).*

#### **2.1. Strategy A: The Hardware Efficient Ansatz (HEA)**

* **Concept:** "Compress & Solve." Use Sinc-DVR to fit the problem on minimal qubits (e.g., 4-5), then use a generic, shallow circuit.
* **Implementation:** Use Qiskit’s `EfficientSU2` (Ry + CNOTs).
* **Pros:** Guaranteed to be shallow; immune to the "Dense Matrix" problem.
* **Cons:** Prone to "Barren Plateaus" (trains slowly); ignores the physics of the well.

#### **2.2. Strategy B: The Truncated Hamiltonian Variational Ansatz (HVA)**

* **Concept:** "Physics-Informed." Build the circuit layers based on the **Truncated** Kinetic Energy operator (Nearest-Neighbor only).
* **Layer Construction:**
* 


* **Implementation:** **CODE FROM SCRATCH.** You must manually map the Pauli terms of the truncated operator to quantum gates.
* **Pros:** Should train faster (fewer parameters); Gray Code makes the neighbor interactions local (fewer SWAPs).
* **Cons:** Circuit depth depends on how well you truncate.

#### **2.3. Depth Profiling**

* **Task:** Before training, transpile both ansätze (HEA vs. HVA) to a target backend (e.g., `FakeTorino`).
* **Metric:** Log the **CNOT count** and **Circuit Depth** for both strategies.

---

### **Week 3: The Benchmarks (Noise & Solvers)**

*Goal: Stress-test the methods under realistic conditions.*

#### **3.1. The "Noise Torture Chamber"**

* **Action:** Run VQE for both strategies on a **Noisy Simulator** (using `FakeProvider` or `AerSimulator` with a depolarization channel).
* **The Experiment:**
* **X-Axis:** Noise Rate (increase error probability).
* **Y-Axis:** Energy Error (distance from exact eigenvalue).
* **Success Metric:** Does Gray Code + HVA degrade *slower* than Binary + HEA?



#### **3.2. Excited States via qEOM**

* **Task:** Once the Ground State is found (using the best ansatz from 3.1), apply the **Quantum Equation of Motion (qEOM)**.
* **Goal:** Extract the **Tunneling Splitting** (Energy difference between Ground and 1st Excited state).
* **Why qEOM?** It avoids re-training the ansatz (unlike VQD), which saves expensive QPU time.

---

### **Summary of Comparisons**

| Component | Control Group (Baseline) | Experimental Group (Your Innovation) |
| --- | --- | --- |
| **Discretization** | Finite Difference (Algebraic) | **Sinc-DVR** (Exponential Convergence) |
| **Encoding** | Standard Binary | **Gray Code** (Hamming Locality) |
| **Ansatz** | HEA (`EfficientSU2`) | **Truncated HVA** (Local-only Physics) |
| **Hamiltonian** | Exact (Dense) | **Truncated** (Sparse for Circuit Generation) |

---

### **Tech Stack**

* **Linear Algebra:** `numpy` (Matrix generation, Eigensolvers for validation).
* **Quantum SDK:** `Qiskit` (Circuit building, `SparsePauliOp`).
* **Optimization:** `scipy.optimize` (COBYLA/L-BFGS-B).
* **Noise Models:** `qiskit_aer` (Depolarizing error, Readout error).
