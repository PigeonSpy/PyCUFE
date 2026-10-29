# PyCUFE

A Python/SageMath implementation of **Ciphertext-Updatable Functional Encryption (CUFE)** for inner products, built on lattices (Learning With Errors). It follows the construction in Cini et al., [*(Inner Product) Functional Encryption with Updatable Ciphertexts*](https://eprint.iacr.org/2022/1284.pdf) (IACR ePrint 2022/1284).

Written for my BSc Computer Science dissertation at the University of Sheffield.

> **Research prototype, not secure.** This is a "toy" construction for learning and experimentation. It uses small parameters and a non-cryptographic random number generation, and it has not been audited. **Do not use it to protect real data.** See [Known limitations](#known-limitations).

---

## What is this?

**Functional encryption** lets you encrypt data so that other people can learn only a specific calculation on it without revealing the underlying (and whole) message in plain

In *inner-product* functional encryption:

1. Someone encrypts a vector **x**.
2. An authority issues a *function key* for a vector **y**.
3. Whoever holds that key can decrypt to get **⟨x, y⟩** (the inner product) and nothing else about **x**.

**Ciphertext-updatable** adds a second ability. A ciphertext created under one *tag* (think of it as a label or epoch) can be moved to a new tag using an *update token*, without decrypting it. In this implementation each ciphertext can be updated **once**.

PyCUFE implements the pieces needed to run that scheme end to end:
**Setup → KeyGen → Encrypt → Decrypt**, plus **TokGen → Update**.

The scheme depends on lattice machinery (trapdoor generation, a gadget matrix, discrete-Gaussian preimage sampling), all implemented here on top of Sage's matrix and sampling primitives.

---

## Status

| Component | State |
|---|---|
| Setup / trapdoor generation | Implemented. Structural check (`A·R = G`) passes. |
| Gadget matrix and inversion | Implemented and self-tested. |
| KeyGen, Encrypt, Decrypt | Implemented. Correctness depends on the parameters (see below). |
| TokGen and Update (one update per ciphertext) | Implemented. |
| Parameter-bound enforcement | **Not yet implemented.** |
| Automated tests | Script-based checks only. A pytest suite is planned. |

**Decryption is correct only when the accumulated noise stays below the signal spacing `⌊q/K⌋/2`.** With some parameter choices the noise drowns the message and the output is wrong. This is expected for a first implementation and is the main thing I am fixing next. See [Known limitations](#known-limitations) and the [Roadmap](#roadmap).

---

## Repository layout

```
CUFE.ipynb   # the scheme (class CUFE_scheme), helpers, test runners, and experiments
LICENSE      # MIT
README.md
```

---

## Quick start

Tested on **Windows 10 + WSL (Ubuntu)** only. I have not tested other setups.

**Requirements:** Python 3.12 (the code uses nested-quote f-strings), SageMath, and Jupyter.

1. Install WSL and open the project from VS Code: <https://learn.microsoft.com/en-us/windows/wsl/tutorials/wsl-vscode> and <https://code.visualstudio.com/docs/remote/wsl>
2. Install SageMath inside WSL: <https://doc.sagemath.org/html/en/installation/index.html>
3. Clone this repo inside WSL, and install the VS Code Python and Jupyter extensions.
4. Open `CUFE.ipynb` and select the kernel **"Python 3.12.12 (Sage)"**.
5. Run the cells top to bottom.

At the bottom of the notebook:

- `single_run()` runs one encrypt/decrypt round trip and prints the result and error.
- `multi_run(n)` runs `n` cycles and reports timing and error statistics.
- Set `testing_enabled = True` to run the scripted checks (gadget inversion, token update, update on a mismatched ciphertext, double update, wrong function key).

<!-- TODO: paste the output of a passing single_run() here once parameters are fixed -->

---

## Using the scheme

```python
scheme = CUFE_scheme(m, n, q, K, security_parameter, sigma)   # runs Setup

y   = vector([...])                       # secret/function vector (length m)
x   = vector([...])                       # message vector (length m)

sk  = scheme.key_gen("tag-1", y)          # function key for y under tag-1
ct  = scheme.encrypt(x, "tag-1")          # ciphertext = (ct1, ct2)
scheme.ciphertexts.append(ct)             # register it (required before update)

scheme.decrypt(*ct, y, sk)                # approximately <x, y>

# Move the ciphertext to a new tag (one update per ciphertext)
tok1, tok2 = scheme.tok_gen("tag-1", "tag-2")
ct_new     = scheme.update(tok1, tok2, *ct, "tag-2")
scheme.updated.append(ct_new)             # mark as updated so decrypt picks the right key

sk_new = scheme.key_gen("tag-2", y)
scheme.decrypt(*ct_new, y, sk_new)
```

Note that `scheme.ciphertexts` and `scheme.updated` are bookkeeping lists maintained by the caller. See [Known limitations](#known-limitations).

### Parameters

| Parameter | Meaning |
|---|---|
| `m` | Lattice dimension. Message and secret vectors have length `m` in this code. |
| `n` | Row dimension of the LWE matrices. |
| `q` | Modulus. |
| `K` | Upper bound on the inner product. Messages are scaled by `⌊q/K⌋`, so results must lie in `[0, K)`. |
| `security_parameter` | Accepted but not yet used. |
| `sigma` | Accepted but currently ignored. Gaussian widths are hard-coded (σ=2, ρ=2, μ=0.5). |

**Constraint:** the trapdoor construction needs `m > n·⌈log₂ q⌉`. Parameter sets that violate this give a degenerate trapdoor.

---

## Results

<!-- TODO: paste results from multi_run() for the parameter sets that work, e.g. a table:
| n | m | q | K | runs | exact | within 2.5% | outside | encrypt (s) | decrypt (s) |
-->

---

## Known limitations

I would rather list these than hide them.

- **Noise growth.** With small moduli and some parameter choices, accumulated error can exceed the signal spacing and `decrypt` returns the wrong inner product.
- **Parameter bounds are not enforced.** Invalid combinations (for example `m ≤ n·⌈log₂ q⌉`) are not rejected, and the Gaussian widths are hard-coded rather than derived from the paper's constraints.
- **Not cryptographically secure randomness.** Secrets, the trapdoor and the sampling matrices use Python's `random`, not a CSPRNG. The random oracles are instantiated as SHA-256-seeded PRNG expansions rather than a standard XOF.
- **Caller-side state.** Whether a ciphertext is fresh or updated is tracked in in-memory lists.
- **Single trusted authority.** One party holds the master secret key. Distributing or hardening that role is out of scope.
- **Toy scale.** Small dimensions with no side-channel considerations.
- **Limited platform testing.** WSL + Python 3.12.12 only.

---

## Roadmap

- [ ] Enforce parameter bounds and raise clear errors for invalid parameter sets
- [ ] Use the `sigma` argument, and derive Gaussian widths from the paper's constraints
- [ ] Replace `random` with `secrets` / OS randomness
- [ ] Split the notebook into a small package (parameters, sampling, scheme) with a thin demo notebook
- [ ] Add a `pytest` suite: round-trip correctness for valid parameters, rejection of invalid ones, update and failure-case tests
- [ ] Publish parameter sets that decrypt reliably, with measured error and runtimes
- [ ] One-command setup (Docker or devcontainer with Sage)

---

## Code map

| Paper component | Where |
|---|---|
| Setup / trapdoor | `setup`, `trap_gen` |
| Gadget matrix and its inverse | `build_gadget`, `bit_decomp`, `gadget_inverse` |
| Left and preimage sampling | `sample_left`, `sample_pre` (plus `_as_matrix` variants) |
| Hash-to-matrix (H1, H2, H3) | `H`, `H1`, `H2`, `H3` |
| Function key generation | `key_gen` |
| Update token generation | `tok_gen` |
| Encryption, update, decryption | `encrypt`, `update`, `decrypt` |

---

## References

- Main CUFE construction: <https://eprint.iacr.org/2022/1284.pdf> (*(Inner Product) Functional Encryption with Updatable Ciphertexts*)
- `sampleLeft`: <https://link.springer.com/chapter/10.1007/978-3-642-13190-5_28> (*Efficient Lattice (H)IBE in the Standard Model*)
- Gadget matrix: <https://advancedcrypto.github.io/Lecture4a.pdf> (Fully Homomorphic Encryption lecture notes)
- Trapdoor sampling: <https://people.csail.mit.edu/vinodv/6876-Fall2015/L16.pdf> (Sampling Lattice Trapdoors lecture notes)

## Acknowledgements

Thank you to my supervisor, Behzad Abdolmaleki for their support and direction, and to the authors of the papers above for the basis of the construction.

## Licence

MIT. See [LICENSE](LICENSE).
