## Experiment 2 — ML Attack on BB84-Encoded PUF Responses

### 2.1 Aim

Investigate the same modelling attack when the classical target response is no longer directly revealed. Instead, the response bit is encoded into a non-orthogonal BB84 state and the adversary receives exactly one copy of each response state.

### 2.2 Construct the BB84 Response

Create a second independent 4-XOR Arbiter PUF to determine the basis bit:

```python
puf_basis = XORArbiterPUF(
    n=n,
    k=k,
    seed=2,
    noisiness=0
)
```

For each challenge $c_i$, compute $b_i = f_b(c_i)$ and $\theta_i = f_\theta(c_i)$, then conceptually return the BB84 state $|\psi_{(b_i,\theta_i)}\rangle$ according to the mapping in Section 0.

### 2.3 Threat Model

The adversary receives:

$$
D_Q = \{(c_i, |\psi_{(b_i,\theta_i)}\rangle)\}_{i=1}^{q}
$$

Assume that the adversary:
- knows the PUF architecture and the BB84 encoding rule;
- does not know the hidden physical parameters of $f_b$ or $f_\theta$;
- receives randomly selected challenges and one corresponding quantum response state per challenge;
- has arbitrary quantum measurement capability, but only one copy of each state;
- cannot obtain repeated copies of the same training response state; and
- wants to build a classical ML model that predicts the underlying target response bit $b$ for unseen challenges.

> **Critical assumption:** The one-copy restriction is essential. With repeated copies of the same BB84 state, the adversary could improve state discrimination, so a fixed 14.64% effective label-error model would no longer represent the attack.

### 2.4 From One-Copy Quantum Measurement to Noisy ML Labels

If $b=0$, the emitted state is equally likely to be $|0\rangle$ or $|+\rangle$. If $b=1$, it is equally likely to be $|1\rangle$ or $|-\rangle$. The adversary therefore distinguishes the two mixed states:

$$
\rho_0 = \frac{1}{2}\big(|0\rangle\langle 0| + |+\rangle\langle +|\big), \qquad
\rho_1 = \frac{1}{2}\big(|1\rangle\langle 1| + |-\rangle\langle -|\big)
$$

For unbiased bits, optimal single-copy discrimination gives:

$$
p_{guess} = \frac{1}{2} + \frac{1}{2\sqrt{2}} \approx 0.853553, \qquad
p_{error} = 1 - p_{guess} = \frac{1}{2} - \frac{1}{2\sqrt{2}} \approx 0.146447
$$

**Interpretation:** The ~14.64% is not physical noise introduced by the PUF or by BB84 preparation. It is the effective classical label-error rate after the optimal adversarial measurement of one copy of the encoded state.

### 2.5 Why No Quantum Simulator Is Required

For this ML experiment, the quantum state-preparation and optimal-measurement stages can be replaced by their analytically known classical statistics:

$$
b \rightarrow |\psi\rangle \rightarrow \text{optimal measurement} \rightarrow \tilde{b}
$$

$$
\Pr[\tilde{b} = b] = 0.853553, \qquad \Pr[\tilde{b} \neq b] = 0.146447
$$

Therefore, students can simulate the adversary's effective training database by independently corrupting response labels with probability $p_{error}$.

### 2.6 Generate the Adversary's Noisy Training Database

```python
clean_responses = train_responses.copy()

p_guess = 0.5 + 1/(2*np.sqrt(2))
p_error = 1 - p_guess
print('p_guess =', p_guess)
print('p_error =', p_error)

rng = np.random.default_rng(12345)
measured_responses = clean_responses.copy()
flip = rng.random(len(measured_responses)) < p_error
measured_responses[flip] *= -1

observed_error = np.mean(
    measured_responses.reshape(-1) != clean_responses.reshape(-1)
)
print('Observed extraction error:', observed_error)
```

For a large training database, the observed extraction error should be close to 0.1464.

### 2.7 Train Exactly the Same ML Attacker

```python
N = 30000
quantum_crps = ChallengeResponseSet(
    train_challenges[:N],
    measured_responses[:N]
)
attack_q = LRAttack2021(
    quantum_crps,
    seed=100,
    k=k,
    bs=1000,
    lr=0.001,
    epochs=100,
    stop_validation_accuracy=1.0
)
model_q = attack_q.fit()
```

The learner sees only $(c_i, \tilde{b}_i)$. It never sees the clean response $b_i$ for its training challenges.

### 2.8 Evaluate Against the Genuine PUF

```python
predicted_q = model_q.eval(test_challenges)
accuracy_q = np.mean(
    predicted_q.reshape(-1) == test_responses.reshape(-1)
)
print('BB84-interface attack accuracy:', accuracy_q)
```

> **Do not test against noisy labels:** The attack objective is to reproduce the underlying physical PUF, so predictions must be compared with the clean test responses $f_b(c^*)$, not with simulated measurement outcomes.

### 2.9 Build the BB84 Learning Curve

Use exactly the same training sizes as Experiment 1. For each $q$:

1. use the same $q$ training challenges as the classical experiment;
2. use their genuine $f_b$ responses as hidden ground truth;
3. independently flip each training label with probability 0.146447;
4. train the same LR attacker with the same hyperparameters;
5. evaluate against the same clean 10,000-response test set; and
6. repeat with several independent corruption masks and ML seeds, then report the mean.

| One-copy quantum responses | Effective noisy CRPs | Accuracy vs true PUF |
|---|---|---|
| 1,000 | 1,000 | |
| 5,000 | 5,000 | |
| 10,000 | 10,000 | |
| 20,000 | 20,000 | |
| 30,000 | 30,000 | |
| 50,000 | 50,000 | |
| 75,000 | 75,000 | |
| 100,000 | 100,000 | |