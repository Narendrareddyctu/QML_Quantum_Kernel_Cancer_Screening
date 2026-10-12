# Quantum Kernels for Early Cancer Screening (Qiskit)

4-qubit quantum-kernel SVM on the Breast Cancer Wisconsin dataset.

Run:  pip install -r requirements.txt  &&  jupyter notebook QML_Quantum_Kernel_Cancer_Screening.ipynb

Pipeline: StandardScaler -> PCA(4) -> Pauli feature map (Z, ZZ, linear entanglement) -> |<psi(x)|psi(x')>|^2 kernel -> SVC(kernel="precomputed").
Includes: circuit-vs-statevector kernel check, bandwidth sweep, 5-fold CV vs classical SVMs, shot noise, gate-noise emulation.

Results (5-fold CV, simulator): tuned quantum kernel 93.6 +/- 2.3 % vs classical RBF 95.4 +/- 4.0 %; default (untuned) quantum kernel 70.8 %.
Limits: no quantum advantage claimed; simulator only.
