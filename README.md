# QPUF Labs

This repository contains a two-part lab sequence on **Physically Unclonable Functions (PUFs)** and their vulnerability to machine-learning-based modeling attacks. The labs progress from a classical XOR Arbiter PUF (XOR-APUF) attack to a hybrid quantum-classical extension (QR-PUF) based on BB84 encoding, allowing direct, consistent comparison of results across labs.

## Overview

| | Lab 1: Classical XOR Arbiter PUF | Lab 2: QR-PUF (BB84-Encoded Responses) |
|---|---|---|
| **Goal** | Show that a classical XOR Arbiter PUF can be cloned with high accuracy using logistic regression, and evaluate it against standard PUF quality metrics | Show how encoding PUF responses into BB84 states changes the modeling-attack problem, by turning single-copy quantum measurement into an equivalent classical label-noise model |
| **PUF simulation** | `pypuf.simulation.XORArbiterPUF` (n=64, k=4) | Same `XORArbiterPUF`, output encoded into BB84 qubits |
| **Attack model** | `pypuf.attack.LRAttack2021` (TensorFlow-based logistic regression) | Same `LRAttack2021`, trained on BB84-corrupted labels |
| **Reference** | Rührmair et al., additive delay model | Quantum Lock paper (arXiv:2110.09469); Škorić, Quantum Readout of PUFs |

## Pedagogical Principle

Both labs use pypuf's own `LRAttack2021` (TensorFlow-based) as the modeling attack, applied consistently to a 64-bit, 4-XOR Arbiter PUF in both labs. Lab 1 first builds the mathematical case for *why* this attack works (the additive delay model and its φ-parity linearity), then evaluates the target PUF against standard quality metrics (reliability, uniqueness, uniformity) before running the attack. Lab 2 reuses the exact same PUF instance, challenges, and attack configuration from Lab 1, changing only what information reaches the adversary — this isolates the effect of BB84 encoding on modeling accuracy from any confounding implementation differences.

---

## Setup

### Requirements
**Note:** the following commands are also included as the first code cell in each `.ipynb` file.

```bash
pip install pypuf==3.2.1 --no-deps --break-system-packages
pip install tensorflow --break-system-packages
pip install "numpy<2.0" --break-system-packages
```

| Package | Used for |
|---|---|
| `pypuf==3.2.1` (`--no-deps`) | XOR Arbiter PUF simulation (`XORArbiterPUF`), CRP generation (`random_inputs`), quality metrics (`reliability`, `uniqueness`), and the `LRAttack2021`/`ChallengeResponseSet` attack API. Installed with `--no-deps` because its default `tensorflow~=2.4.0` pin has no wheels for modern Python; a current TensorFlow is installed separately instead. |
| `tensorflow` | Backend for `LRAttack2021`'s neural-network-based logistic regression attack |
| `numpy<2.0` | Required for compatibility with `pypuf==3.2.1`'s internal array handling |
| `matplotlib` | Learning-curve and result plots |
| `jupyter` (or an equivalent notebook runner) | Running the `.ipynb` files — see Known Limitations |

### Known `pypuf` / TensorFlow Compatibility Issue

`pypuf==3.2.1`'s `LRAttack2021` was written against older TensorFlow/NumPy versions and fails with a modern TensorFlow install:

```
TypeError: Expected int8, but got 0.5 of type 'float'.
```

This happens inside `LRAttack2021`'s internal `loss`/`accuracy` functions when they multiply an `int8` response tensor by a Python float. **Both notebooks include a fix for this** as a cell near the end of the modeling-attack section — run it once (it monkey-patches `LRAttack2021.loss`, `LRAttack2021.accuracy`, and `LRAttack2021.keras_to_pypuf` at runtime) if you hit this error, then retrain. No changes to installed package files are needed; the patch only needs to be re-applied once per fresh kernel/session.

---

## Lab 1: Classical XOR Arbiter PUF Attack

**Notebook:** `QPUF_LAB1.ipynb`

### What It Covers
- **Additive delay model** — the general linear framework shared by all delay-based PUFs, and why it makes logistic regression an effective attack
- **XOR Arbiter PUF construction** — combining k independent Arbiter PUF chains via XOR (product of signs in the ±1 representation), and why this breaks the simple linear separability that makes plain Arbiter PUFs so vulnerable
- **PyPUF implementation** of a 64-bit, 4-XOR Arbiter PUF (`XORArbiterPUF(n=64, k=4)`)
- **Security analysis** using pypuf's built-in metrics:
  - **Reliability** — response stability under simulated noise (requires a `noisiness > 0` instance)
  - **Uniqueness** — pairwise response distinguishability across multiple simulated chip instances (different seeds)
  - **Uniformity** — 0/1 response balance for a single chip instance
- **Modeling attack** via `pypuf.attack.LRAttack2021`, trained on 50,000 CRPs and evaluated on a held-out test set

---

## Lab 2: QR-PUF (BB84-Encoded Response) Attack

**Notebook:** `QPUF_LAB2.ipynb`

### What It Covers
- **The QR-PUF concept** — keeping the underlying PUF classical while carrying challenge/response information over quantum (BB84) states, and why this exploits the no-cloning theorem and the indistinguishability of non-orthogonal states
- **BB84 qubit encoding** — mapping a (basis bit, value bit) pair to one of the four states `|0>, |1>, |+>, |->`, and why an adversary measuring in the wrong basis learns nothing about the value bit
- **Reuse of the Lab 1 XOR-APUF** — the same `XORArbiterPUF(n=64, k=4, seed=1)` instance and challenge set from Lab 1 is reused here as the target PUF
- **Modeling the one-copy quantum adversary analytically** — rather than simulating an actual quantum circuit, the notebook derives the optimal single-copy state-discrimination probability, `p_guess = 1/2 + 1/(2*sqrt(2)) ≈ 0.8536` and `p_error ≈ 0.1465`, and reproduces its effect by independently flipping each training response label with probability `p_error`. This is equivalent to the physical single-copy BB84 measurement without requiring a quantum simulator.
- **Modeling attack on the corrupted labels** — the same `LRAttack2021` configuration from Lab 1 is retrained on the noisy (BB84-equivalent) labels and evaluated against the *clean* test set, so the reported accuracy reflects how well the adversary recovers the true underlying PUF, not the noisy labels themselves

### Note on Scope
This lab models the **one-copy adversary** case only (each response state is measured exactly once, with no possibility of collecting repeated copies of the same state). The notebooks do not implement an actual Qiskit circuit simulation of the BB84 states — the analytical noise-injection approach above is mathematically equivalent for this threat model and avoids the performance overhead of per-qubit circuit simulation at scale.

---

## References

- U. Rührmair et al., "Modeling Attacks on Physical Unclonable Functions," ACM CCS 2010.
- B. Škorić, "Quantum Readout of Physical Unclonable Functions," AFRICACRYPT 2010.
- W. Mao et al., "Quantum Lock: A Provable Quantum Communication Advantage," [arXiv:2110.09469](https://arxiv.org/abs/2110.09469)
- pypuf documentation: https://pypuf.readthedocs.io/
- pypuf documentation — Logistic Regression Attack: https://pypuf.readthedocs.io/en/latest/attacks/lr.html

## Known Limitations

- The PyPI release of `pypuf==2.2.0` ships with an empty `attack` submodule (`LRAttack2021` cannot be imported at all). Both labs instead install `pypuf==3.2.1` with `--no-deps` plus a separately installed modern TensorFlow — see Setup above.
- `LRAttack2021` in `pypuf==3.2.1` fails against modern TensorFlow/NumPy with a `TypeError` inside its internal loss/accuracy functions; both notebooks include a runtime monkey-patch to fix this (see "Known `pypuf` / TensorFlow Compatibility Issue" above).
- Lab 2 models the BB84 single-copy measurement analytically (via label-flip probability `p_error`) rather than through an explicit Qiskit circuit simulation; it does not cover multi-copy adversaries or the lockdown/challenge-reuse countermeasure from the Quantum Lock paper.
- To run these notebooks, you need Jupyter Notebook, the VS Code Jupyter extension, or an equivalent (e.g. Google Colab).
