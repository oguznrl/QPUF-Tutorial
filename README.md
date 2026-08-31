# QPUF Labs

This repository contains a two-part lab sequence on **Physically Unclonable Functions (PUFs)** and their vulnerability to machine-learning-based modeling attacks. The labs progress from a purely classical Arbiter PUF (A-PUF) attack to a hybrid quantum-classical extension (QR-PUF / HLPUF) based on the BB84 protocol, allowing direct, consistent comparison of results across labs.


## Overview

| | Lab 1: Classical A-PUF | Lab 2: QR-PUF (HLPUF/BB84) |
|---|---|---|
| **Goal** | Show that a classical Arbiter PUF can be cloned with high accuracy using logistic regression | Show how encoding PUF responses into BB84 qubits changes the modeling-attack problem |
| **PUF simulation** | `pypuf.simulation.ArbiterPUF` | Same `ArbiterPUF`, output encoded into BB84 qubits |
| **Attack model** | `sklearn.linear_model.LogisticRegression` | Same `LogisticRegression` class |
| **Reference** | Rührmair et al., additive delay model | Quantum Lock paper (arXiv:2110.09469) |

## Pedagogical Principle

Both labs deliberately avoid black-box attack libraries (e.g. pypuf's built-in `LRAttack2021`, which is non-functional in pypuf 2.2.0). Instead, the φ-parity feature transformation and the logistic regression attack are implemented explicitly, so students see exactly what is being learned at each step. Lab 2 reuses the identical feature transform and model class from Lab 1, isolating the effect of quantum encoding on ML learnability from any confounding implementation differences.

---

## Setup

### Requirements
**Note that** following commands also exist in .ipynb file.

```bash
pip install pypuf qiskit qiskit-aer numpy scikit-learn matplotlib jupyter pylatexenc
```

Some environments require the `--break-system-packages` flag:

```bash
pip install pypuf qiskit qiskit-aer numpy scikit-learn matplotlib jupyter pylatexenc --break-system-packages
```

| Package | Used for |
|---|---|
| `pypuf` | Classical Arbiter PUF simulation (`ArbiterPUF`, `random_inputs`) |
| `qiskit` | BB84 encoding/measurement circuits (`QuantumCircuit`, `transpile`) |
| `qiskit-aer` | Circuit-based measurement simulation (`AerSimulator`) |
| `numpy` | Vectorized computation, φ-transform, RNG |
| `scikit-learn` | Logistic regression attack, accuracy metrics |
| `matplotlib` | Learning-curve and result plots |
| `jupyter` | Running the `.ipynb` notebooks |
| `pylatexenc` | Required by Qiskit for circuit diagrams (`qc.draw('mpl')`) |

---

## Lab 1: Classical Arbiter PUF Attack

**Notebook:** `QPUF_LAB1.ipynb`

### What It Covers
- The additive delay model of a k-stage Arbiter PUF
- Deriving the $$\phi$$ feature vector from the delay model, and why raw challenge bits are not linearly separable without it
- Simulating a 64-bit A-PUF with `pypuf.simulation.ArbiterPUF`
- Training a hand-coded logistic regression attack (`sklearn.linear_model.LogisticRegression`) on the $$\phi$$ -transformed challenges
- Evaluating cloning accuracy as a function of the number of CRPs used in training
- Standard PUF quality metrics: reliability, uniqueness, uniformity

---

## Lab 2: QR-PUF (HLPUF/BB84) Attack

**Notebook:** `QPUF_LAB2.ipynb`

### What It Covers
- The Quantum-Readout PUF (QR-PUF) concept and the Hybrid Locked PUF (HLPUF) construction from the Quantum Lock paper (arXiv:2110.09469)
- Encoding each pair of A-PUF response bits into one BB84 qubit (one bit selects the basis, the other the encoded value), implemented as real Qiskit circuits (`QuantumCircuit`, `AerSimulator`)
- Simulating Eve's measurement strategy: a randomly guessed basis, run through the same Qiskit circuits, producing noisy labels
- Replaying the Lab 1 logistic regression attack on the noisy labels, and comparing the number of CRPs required to reach a 

### Scale Note
Running Qiskit circuits for very large CRP counts (tens of thousands) is significantly slower than a pure numpy simulation due to per-circuit transpile/job overhead. The notebook batches circuit execution to reduce this cost, and recommends validating the Qiskit-based measurement against a numpy vectorized equivalent at small `N` before scaling up the full learning-curve experiment.

## References

- U. Rührmair et al., "Modeling Attacks on Physical Unclonable Functions," ACM CCS 2010.
- B. Škorić, "Quantum Readout of Physical Unclonable Functions," AFRICACRYPT 2010.
- Quantum Lock: A Provable Quantum Communication Advantage, [arXiv:2110.09469](https://arxiv.org/abs/2110.09469)
- pypuf documentation: https://pypuf.readthedocs.io/
- Qiskit documentation: https://docs.quantum.ibm.com/

## Known Limitations

- `pypuf` 2.2.0's built-in attack module (`LRAttack2021`) is non-functional in some environments; both labs use a manual `sklearn` implementation instead.
- Qiskit circuit-based simulation does not scale efficiently to very large CRP counts; see the Scale Note above.
- To complete this lab exercise, you need to install Jupyter Notebook or the Visual Studio Jupyter Notebook extension. Alternatively, you can use Google Colab.