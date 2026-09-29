# QPUF Labs

This repository contains a two-part lab sequence on **Physically Unclonable Functions (PUFs)** and their vulnerability to machine-learning-based modeling attacks. The labs progress from a classical XOR Arbiter PUF (XOR-APUF) attack to a hybrid quantum-classical extension (QR-PUF) based on BB84 encoding, allowing direct, consistent comparison of results across labs.

## Overview

| | Lab 1: Classical XOR Arbiter PUF | Lab 2: QR-PUF (BB84-Encoded Responses) |
|---|---|---|
| **Goal** | Train an ML model that predicts a classical PUF's responses with high accuracy, and check the PUF's quality with standard tests | See how much harder that same attack becomes once the PUF's responses are protected by quantum encoding instead of sent as plain bits |
| **PUF simulation** | A simulated 64-bit hardware PUF built from 5 combined chains | The same PUF as Lab 1, plus a second independent one used to pick the quantum encoding basis |
| **Attack model** | The same logistic-regression-based ML attack in both labs | The same logistic-regression-based ML attack in both labs |
| **Quantum step** | — | The quantum measurement's effect is reproduced two ways: a quick mathematical shortcut, and an actual simulated quantum circuit — both give the same result |
| **Reference** | Rührmair et al., additive delay model | Quantum Lock paper (arXiv:2110.09469); Škorić, Quantum Readout of PUFs |

---

## Setup

### Recommended: Use a Virtual Environment

Both labs install a fairly specific `pypuf`/`tensorflow`/`numpy` combination (see the compatibility notes below), and Lab 2's optional Qiskit section adds another set of packages on top. Installing all of this into your system Python can conflict with other projects, so a dedicated virtual environment is recommended.

**Linux / macOS:**
```bash
python3 -m venv qpuf-env
source qpuf-env/bin/activate
python -m pip install --upgrade pip
```

**Windows (PowerShell):**
```powershell
python -m venv qpuf-env
qpuf-env\Scripts\Activate.ps1
python -m pip install --upgrade pip
```

**Windows (cmd.exe):**
```cmd
python -m venv qpuf-env
qpuf-env\Scripts\activate.bat
python -m pip install --upgrade pip
```

Then select the **"Python (qpuf-env)"** kernel from within Jupyter (or the VS Code Jupyter extension) before running either notebook. To leave the environment later, run `deactivate`.

> **Google Colab:** Colab provides its own managed environment, so none of the above is needed there — just run the `pip install` cells directly, as already included at the top of each notebook.

### Requirements
**Note:** the following commands are also included as the first code cell in each `.ipynb` file.

Core stack (both labs):
```bash
pip install pypuf==3.2.1
pip install tensorflow
pip install "numpy<2.0"
```

Additional, only for Lab 2's optional Qiskit section (Section 7):
```bash
pip install qiskit qiskit-aer
pip install numpy scipy
```

| Package | Used for |
|---|---|
| `pypuf==3.2.1` | XOR Arbiter PUF simulation (`XORArbiterPUF`), CRP generation (`random_inputs`), quality metrics (`reliability`, `uniqueness`), and the `LRAttack2021`/`ChallengeResponseSet` attack API. |
| `tensorflow` | Backend for `LRAttack2021`'s neural-network-based logistic regression attack |
| `numpy<2.0` | Required for compatibility with `pypuf==3.2.1`'s internal array handling |
| `matplotlib` | Learning-curve and result plots |
| `qiskit`, `qiskit-aer` | Lab 2, Section 7 only: single-qubit BB84 state-preparation circuits and `AerSimulator`-based measurement |
| `jupyter` (or an equivalent notebook runner) | Running the `.ipynb` files — see Known Limitations |

> **Note on installing Qiskit alongside pypuf/TensorFlow:** `qiskit`/`qiskit-aer` can pull in a different `numpy`/`scipy` than the `pypuf`/`tensorflow` stack expects. Lab 2 reinstalls `numpy` and `scipy` immediately after installing Qiskit (Section 7.1) to keep both stacks working in the same kernel.

### Known `pypuf` / TensorFlow Compatibility Issue

`pypuf==3.2.1`'s `LRAttack2021` was written against older TensorFlow/NumPy versions and fails with a modern TensorFlow install:

```
TypeError: Expected int8, but got 0.5 of type 'float'.
```

This happens inside `LRAttack2021`'s internal `loss`/`accuracy` functions when they multiply an `int8` response tensor by a Python float. **Both notebooks include a fix for this** as a dedicated cell right after the first attack attempt — run it once (it monkey-patches `LRAttack2021.loss`, `LRAttack2021.accuracy`, and `LRAttack2021.keras_to_pypuf` at runtime) if you hit this error, then re-run the attack cell above it. No changes to installed package files are needed; the patch only needs to be re-applied once per fresh kernel/session. Lab 2's Section 7 re-applies the same patch independently so that section can be run on its own.

---

## Lab 1: Classical XOR Arbiter PUF Attack

**Notebook:** `QPUF_LAB1.ipynb`

### What It Covers
- The additive delay model behind delay-based PUFs, and why it makes logistic regression such an effective attack
- Building a 64-bit, 5-XOR Arbiter PUF in pypuf
- Standard PUF quality metrics: reliability, uniqueness, uniformity
- A modeling attack via `LRAttack2021`, trained on up to 200,000 CRPs

---

## Lab 2: QR-PUF (BB84-Encoded Response) Attack

**Notebook:** `QPUF_LAB2.ipynb`

### What It Covers
- The QR-PUF concept — keeping the PUF classical but carrying its responses over BB84 quantum states
- Encoding each response as one of four non-orthogonal qubit states, using a second independent PUF to choose the basis
- Modeling the one-copy quantum adversary's effective label noise (`p_error ≈ 0.1465`), analytically and via an actual Qiskit circuit simulation
- Re-running the same `LRAttack2021` attack on the resulting noisy labels, and comparing against the clean Lab 1 result

---

## References

- U. Rührmair et al., "Modeling Attacks on Physical Unclonable Functions," ACM CCS 2010.
- B. Škorić, "Quantum Readout of Physical Unclonable Functions," AFRICACRYPT 2010.
- W. Mao et al., "Quantum Lock: A Provable Quantum Communication Advantage," [arXiv:2110.09469](https://arxiv.org/abs/2110.09469)
- pypuf documentation: https://pypuf.readthedocs.io/
- pypuf documentation — Logistic Regression Attack: https://pypuf.readthedocs.io/en/latest/attacks/lr.html
- Qiskit documentation: https://docs.quantum.ibm.com/

## Known Limitations

- The PyPI release of `pypuf==2.2.0` ships with an empty `attack` submodule (`LRAttack2021` cannot be imported at all). Both labs instead install `pypuf==3.2.1` — see Setup above.
- `LRAttack2021` in `pypuf==3.2.1` fails against modern TensorFlow/NumPy with a `TypeError` inside its internal loss/accuracy functions; both notebooks include a runtime monkey-patch to fix this (see "Known `pypuf` / TensorFlow Compatibility Issue" above).
- Modeling a k-XOR Arbiter PUF with `LRAttack2021` is a non-convex optimization; a given random seed can occasionally converge to a poor local optimum (near-chance accuracy). Lab 2's Section 7 addresses this with a restart helper; if you see this in Lab 1 or Lab 2's main attack cell, simply re-run with a different `seed`.
- Lab 2's core sections (4–6) model the BB84 single-copy measurement analytically rather than through a circuit simulation; Section 7 covers the actual Qiskit-based circuit but only for the one-copy, single-qubit-per-challenge case — it does not cover multi-copy adversaries or the lockdown/challenge-reuse countermeasure from the Quantum Lock paper.
- Lab 1 references an image asset (`Pictures/PUFscheme.png`) that must be present alongside the notebook for that cell to render correctly.
- To run these notebooks, you need Jupyter Notebook, the VS Code Jupyter extension, or an equivalent (e.g. Google Colab). If using a local Jupyter install, make sure it's running inside the virtual environment above (or that the environment is registered as a kernel — see the venv instructions).