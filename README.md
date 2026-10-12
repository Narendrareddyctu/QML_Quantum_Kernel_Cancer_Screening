# Quantum Kernels for Early Cancer Screening

**Quantum Machine Learning Challenge · Built with Qiskit**

A 4-qubit quantum-kernel classifier that separates malignant from benign breast biopsies. A Qiskit circuit turns each patient sample into a quantum state, the overlap between two states is used as a similarity score (a *quantum kernel*), and a classical Support Vector Machine (SVM) is trained on that score.

The tuned quantum model reaches **93.6 % accuracy** (5-fold cross-validation, simulator), about two points below a classical RBF SVM (95.4 %), using a shallow circuit of **4 qubits, depth 11 and 6 CNOT gates**.

---

## Table of contents
1. [Problem and motivation](#1-problem-and-motivation)
2. [Quantum approach](#2-quantum-approach)
3. [Qiskit implementation](#3-qiskit-implementation)
4. [Working prototype](#4-working-prototype)
5. [Results and analysis](#5-results-and-analysis)
6. [Innovation](#6-innovation)
7. [Limitations (honest summary)](#7-limitations-honest-summary)
8. [Project structure](#8-project-structure)
9. [How to run](#9-how-to-run)
10. [Reproducibility notes](#10-reproducibility-notes)
11. [Next steps](#11-next-steps)

---

## 1. Problem and motivation
Doctors must decide whether a tumour is malignant or benign from measurements of cell nuclei taken from a biopsy. Missing a malignant case is the costly mistake, so a good classifier matters.

- **Data:** Breast Cancer Wisconsin (Diagnostic), loaded from scikit-learn. 569 samples, 30 numeric features each (212 malignant, 357 benign).
- **Task:** binary classification (malignant vs benign).
- **Question:** can a quantum feature space classify this data as well as a tuned classical model, with a circuit small enough to run on today's quantum hardware?

## 2. Quantum approach
The method is a **quantum kernel SVM**:

1. **Reduce:** standardise the 30 features, then use PCA to keep 4 components, one qubit per component. The values are scaled into the range [0, π].
2. **Encode:** a Pauli feature map (Z and ZZ terms, linear entanglement, 1 repetition) prepares a 4-qubit state |ψ(x)⟩ for each sample x.
3. **Compare:** the similarity of two samples is the overlap of their states:

   `k(x, x') = |⟨ψ(x)|ψ(x')⟩|²`

4. **Classify:** the matrix of these similarities is passed to a classical SVM (`SVC(kernel="precomputed")`).

Why a quantum kernel: the states live in a 16-dimensional Hilbert space, and the entangling gates mix features in ways that are costly to imitate classically. Whether that helps on a given dataset has to be tested, which is what this project does.

## 3. Qiskit implementation
- **Circuit:** built with `qiskit.circuit.library.pauli_feature_map`. After decomposition it has 4 qubits, depth 11, 11 single-qubit gates and 6 CNOTs.
- **Kernel:** computed from exact statevectors with `qiskit.quantum_info.Statevector`.
- **Circuit-based check:** the notebook also measures kernel entries the way hardware would. It applies the feature map for one sample, then the inverse feature map for the other, and records how often the result is `0000` on `AerSimulator`. This matches the statevector kernel to within 0.008.
- **Noise tests:**
  - finite-shot sampling (1024 shots per kernel entry, emulated),
  - a depolarizing gate-noise model on `AerSimulator` (0.1 % on single-qubit gates, 1 % on two-qubit gates).

## 4. Working prototype
The notebook `QML_Quantum_Kernel_Cancer_Screening.ipynb` is runnable end to end and already contains its outputs. Its sections:

| Section | What it does |
|---|---|
| 1 | Preprocessing: 30 features to 4 qubits |
| 2 | Draws the feature-map circuit and prints its size |
| 3 | Defines the quantum kernel and checks it against the circuit-based version |
| 4 | Sweeps the kernel bandwidth using inner cross-validation on training data only |
| 5 | 5-fold cross-validation: quantum vs classical models |
| 6 | Kernel heatmap and confusion matrix on one held-out fold |
| 7 | Gate-noise emulation on a small subset |

## 5. Results and analysis
Evaluation: 5-fold stratified cross-validation, 140 training and 100 test samples per fold, simulator. Malignant recall is the share of malignant cases that were correctly found.

| Model | Accuracy % | Malignant recall % |
|---|---|---|
| Classical RBF SVM | 95.4 ± 4.0 | 90.7 |
| Classical linear SVM | 94.8 ± 2.5 | 89.6 |
| Quantum kernel, default (bandwidth 1, 2 repetitions) | 70.8 ± 3.1 | 46.8 |
| Quantum kernel, tuned (bandwidth 0.1, 1 repetition) | 93.6 ± 2.3 | 85.8 |
| Quantum kernel, tuned, 1024 shots | 92.6 ± 3.4 | 83.7 |

Noisy-gate check (50 training and 40 test samples, one fold): both the noiseless and the noisy kernel scored 85 %. This subset is small, so treat it as a quick sanity check, not a precise measurement.

**What the results show**
- The default quantum kernel performs poorly (about 71 %) because it is too expressive: almost every pair of samples looks equally dissimilar.
- Shrinking the input scale (bandwidth 0.1) smooths the kernel and lifts accuracy to 93.6 %.
- Finite-shot sampling costs about one point of accuracy, so the tuned kernel tolerates measurement noise.
- The tuned quantum model is competitive with, but not better than, the classical baselines.

## 6. Innovation
- **Bandwidth-tuned, shallow kernel:** instead of using the standard feature map as is, we tune its bandwidth and find the one-repetition version is both more accurate and half as deep as the default circuit.
- **No test leakage:** the bandwidth is chosen by cross-validation inside the training data only.
- **Hardware-aware evaluation:** the kernel is checked with real circuits (compute-uncompute), with finite shots, and with a gate-noise model, not only with ideal statevectors.
- **Honest reporting:** the untuned failure case and the gap to classical models are reported, not hidden.

## 7. Limitations (honest summary)
- No quantum advantage is claimed. The tuned quantum kernel is about two points below the classical RBF SVM.
- Malignant recall (85.8 %) is lower than the RBF SVM (90.7 %).
- All results come from simulators. No run on real quantum hardware has been done yet.
- The 30 features are compressed to 4 PCA components, so some information is lost, and only one public dataset is used.
- The noisy-gate test uses a small subset and a simple depolarizing model.

## 8. Project structure
```
QML_Project/
├── QML_Quantum_Kernel_Cancer_Screening.ipynb   # full project, with outputs
├── requirements.txt                            # Python packages
└── README.md                                   # this file
```

## 9. How to run
Requires Python 3.10 or newer.

```bash
pip install -r requirements.txt
jupyter notebook QML_Quantum_Kernel_Cancer_Screening.ipynb
```

Run all cells in order. The full notebook, including the optional noise test, takes a few minutes on a laptop. No IBM account or internet connection is needed, since everything runs on local simulators.

## 10. Reproducibility notes
- Random seeds are fixed (`default_rng(7)`, `StratifiedKFold(random_state=1)`, `PCA(random_state=0)`).
- The shot-noise and gate-noise results involve sampling, so the last decimal may vary slightly between runs.
- The notebook was built with Qiskit 2.x. It uses `pauli_feature_map`, which replaces the deprecated `PauliFeatureMap` class.

## 11. Next steps
- Run the tuned circuit on IBM quantum hardware with error mitigation.
- Use a class-weighted SVM to raise malignant recall.
- Try more qubits, other feature maps, and other datasets.
- Compare against more classical baselines, such as gradient boosting.
