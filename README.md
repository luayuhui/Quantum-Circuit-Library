# Quantum Circuit Library

**Author:** Lua Yu Hui (Department of Physics, Universiti Teknologi Malaysia)  
**Framework:** Python 3.9+ | Qiskit | Qiskit Aer | Matplotlib | Tkinter  

---

## 📌 Project Overview
The **Quantum Circuit Library** is a GUI application developed to simulate and demonstrate fundamental quantum mechanics concepts using IBM's Qiskit SDK and the `AerSimulator` backend.

![Uploading quantum_circuit_library_GUI.png…]()


The application allows users to explore:
1. **Superposition** (1-qubit Hadamard transformation)
2. **Bell State** ($|\Phi^+\rangle$ 2-qubit maximal entanglement)
3. **GHZ State** ($|\text{GHZ}\rangle$ 3-qubit multi-qubit entanglement)

---

## 🛠️ Module Structure & Reusable Functions

The program follows a modular architecture using the required reusable functions:

* `create_superposition()`: Initializes a 1-qubit circuit and applies a Hadamard gate ($H$) to put the qubit into equal superposition.
* `create_bell_state()`: Initializes a 2-qubit circuit and applies an $H$ gate followed by a Controlled-NOT ($CNOT$) gate to entangle two qubits.
* `create_ghz_state()`: Extends entanglement across 3 qubits using an $H$ gate and two sequential $CNOT$ gates.
* `show_statevector(circuit)`: Extracts the state vector array of the quantum circuit **prior** to measurement collapse using `qiskit.quantum_info.Statevector`.
* `run_circuit(circuit, shots)`: Appends measurement gates and executes the circuit on `qiskit_aer.AerSimulator` across 1,024 shots.
* `show_histogram(counts, title, parent_frame)`: Embeds an interactive Matplotlib measurement histogram into the Tkinter application UI.

---

## ⚛️ Quantum Concepts Demonstrated

### 1. Superposition
- **State Preparation:** $H|0\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle) = |+\rangle$
- **Measurement Outcome:** Equal $\approx 50\%$ measurement probability for $|0\rangle$ and $|1\rangle$.

### 2. Bell State ($|\Phi^+\rangle$)
- **State Preparation:** $(CNOT_{0\to1})(H \otimes I)|00\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$
- **Measurement Outcome:** Perfectly correlated state measurements yielding only $|00\rangle$ and $|11\rangle$.

### 3. GHZ State ($|\text{GHZ}\rangle$)
- **State Preparation:** $(CNOT_{1\to2})(CNOT_{0\to1})(H \otimes I \otimes I)|000\rangle = \frac{1}{\sqrt{2}}(|000\rangle + |111\rangle)$
- **Measurement Outcome:** Tripartite entanglement yielding state measurements of $|000\rangle$ and $|111\rangle$.

---

## 📸 Screenshots
*(Include screenshots of your running GUI interface here before submitting on GitHub)*
