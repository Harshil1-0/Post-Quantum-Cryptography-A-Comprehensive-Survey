# Post-Quantum Cryptography: A Comprehensive Survey

Group project for **MA2209 – Number Theory and Cryptography** (May 2026). The project traces how Shor's algorithm breaks the number-theoretic assumptions behind classical public-key cryptography, surveys the five main post-quantum (PQC) families and their hard problems, summarises the NIST standards, and evaluates the strengths, shortcomings and outlook of PQC.

**Team**

| Member | Roll No. |
|--------|----------|
| Gnanada | SE24UCAM003 |
| Anusha | SE24UCAM006 |
| Neharika | SE24UCAM018 |
| Pearl | SE24UCAM043 |
| Harshil | SE24UCAM051 |
| Dhruv | SE24UCAM068 |

> Math is written in LaTeX, which renders on GitHub. Items marked *(extension)* are added here to illustrate the report's points and are not in the original documents.

---

## 1. Classical Hardness Assumptions

| Problem | Statement | Used by |
|---------|-----------|---------|
| Integer Factorisation (IFP) | Given $N = pq$, recover $p, q$ | RSA |
| Discrete Logarithm (DLP) | Given $g, p, y = g^x \bmod p$, recover $x$ | Diffie–Hellman, DSA |
| Elliptic-Curve DLP (ECDLP) | Given $P$ and $Q = kP$, recover $k$ | ECDH, ECDSA |

**RSA.** For $n = pq$, $\varphi(n) = (p-1)(q-1)$ and $a^{\varphi(n)} \equiv 1 \pmod n$ whenever $\gcd(a,n)=1$.

- Encryption: $c = m^e \bmod n$
- Decryption: $m = c^d \bmod n$
- Signature: $\sigma = H(m)^d \bmod n$

Anyone who factors $n$ gets $\varphi(n)$ and hence the private exponent $d$.

**Best classical attack (GNFS)** has sub-exponential complexity

$$L_n\!\left[\tfrac13,\ \left(\tfrac{64}{9}\right)^{1/3}\right] = \exp\!\Big(\big(\left(\tfrac{64}{9}\right)^{1/3} + o(1)\big)(\ln n)^{1/3}(\ln\ln n)^{2/3}\Big),$$

with $(64/9)^{1/3}\approx 1.923$, which is infeasible for a 2048-bit modulus.

## 2. Shor's Algorithm

Shor (1994) reduces factoring to **order finding**: find the least $r > 0$ with $a^r \equiv 1 \pmod n$. A quantum computer uses a superposition and the **Quantum Fourier Transform** to extract $r$ efficiently, and $r$ then yields the factors. It runs in polynomial time and breaks RSA, DH, DSA, ECDH and ECDSA.

| Algorithm | Type | Complexity |
|-----------|------|------------|
| Trial division | Classical | $O(\sqrt n)$ |
| GNFS | Classical | $L_n[1/3, c]$ (sub-exponential) |
| Shor | Quantum | $O\big((\log n)^2 \log\log n\cdot\log\log\log n\big)$ |

Symmetric primitives are only weakened: **Grover's algorithm** gives a quadratic speed-up, so AES and SHA survive by doubling key or output length (for example, hash outputs of at least 256 bits).

**Harvest now, decrypt later:** adversaries record encrypted traffic today to decrypt it once a cryptographically relevant quantum computer exists, so long-lived secrets are already at risk.

### Worked Example of the Order-Finding Step *(extension)*

Factor $N = 15$ with $a = 7$. The powers of 7 mod 15 are $7, 4, 13, 1$, so the order is $r = 4$ (even). Then

$$\gcd(7^{2}-1,\,15) = \gcd(48,15) = 3, \qquad \gcd(7^{2}+1,\,15) = \gcd(50,15) = 5,$$

and $15 = 3\cdot 5$. On a quantum computer only the period-finding is quantum; this gcd step is classical.

## 3. The Five PQC Families

| Family | Hard problem | Strength | Weakness | Examples |
|--------|--------------|----------|----------|----------|
| **Lattice** | SVP, LWE, MLWE | Efficient and versatile; NIST standards | Moderate key/ciphertext size | ML-KEM, ML-DSA, FN-DSA |
| **Code** | Syndrome decoding | 40+ year security record | Very large public keys | Classic McEliece, HQC |
| **Hash** | Preimage / collision resistance | Minimal assumptions | Large signatures | SLH-DSA (SPHINCS+) |
| **Multivariate** | MQ polynomial systems | Very small, fast signatures | Structural breaks | Rainbow (broken 2022) |
| **Isogeny** | Isogeny problems on supersingular curves | Smallest key sizes | SIKE broken 2022 | SQIsign |

### Lattices

A lattice with basis $B = (b_1,\dots,b_n)$ is

$$\Lambda(B) = \Big\{\textstyle\sum_{i=1}^n z_i b_i \;:\; z_i \in \mathbb{Z}\Big\}.$$

- **SVP:** find a shortest non-zero vector, of length $\lambda_1(\Lambda) = \min_{v\in\Lambda\setminus\{0\}}\lVert v\rVert$.
- **LWE:** given $A \in \mathbb{Z}_q^{m\times n}$ and $b = As + e \pmod q$ with small noise $e$, recover $s$. Regev (2005) showed average-case LWE is at least as hard as worst-case lattice problems.
- **MLWE** is the module (structured) variant used by ML-KEM and ML-DSA.

### Codes

**Syndrome Decoding Problem (NP-hard):** given a parity-check matrix $H$ and syndrome $s = Hx \pmod 2$, find $x$ of Hamming weight at most $t$. McEliece (1978) hides a structured code (for example a Goppa code) behind a random-looking disguise $G' = SGP$.

### Hash-based

Signatures are built from one-time signatures aggregated with **Merkle trees**; security reduces to preimage and collision resistance of the hash function alone.

### Multivariate

The **MQ problem:** find $(x_1,\dots,x_n)$ over $\mathbb{F}_q$ satisfying $m$ quadratic equations $P_i(x_1,\dots,x_n)=0$. Rainbow was broken in 2022 by a rank attack exploiting its oil-and-vinegar structure.

### Isogenies

Isogenies $\phi: E_1 \to E_2$ between supersingular elliptic curves over $\mathbb{F}_{p^2}$, whose endomorphism rings are non-commutative quaternion algebras. SIKE fell to a 2022 classical attack using its auxiliary torsion-point information (via Kani's lemma). SQIsign, based on the Deuring correspondence between isogenies and quaternion ideals, avoids this flaw and gives signatures of about 177 bytes.

## 4. NIST Standards

| Standard | Algorithm | Basis | Role |
|----------|-----------|-------|------|
| FIPS 203 | ML-KEM (Kyber) | Module lattice | Key establishment |
| FIPS 204 | ML-DSA (Dilithium) | Module lattice | Digital signatures |
| FIPS 205 | SLH-DSA (SPHINCS+) | Hash-based | Backup signature scheme |
| FIPS 206 | FN-DSA (Falcon) | Lattice | Compact signatures (in development) |
| HQC | Code-based KEM | Codes | Selected March 2025 as backup KEM; draft expected 2026–2027 |

FIPS 203–205 were published in 2024. NIST encourages **hybrid** (classical + PQC) deployment during the transition and **crypto-agility**.

## 5. Analysis Methods

**Security reductions.** A reduction shows that breaking a scheme is as hard as a well-studied problem. ML-KEM reduces worst-case SVP to average-case MLWE, with proofs extended to quantum adversaries in the Quantum Random Oracle Model (QROM).

**NIST security levels** are matched to exhaustive search on AES: Level 1 ≈ AES-128, Level 3 ≈ AES-192, Level 5 ≈ AES-256.

**Core-SVP model.** Lattice security is set by the cost of the BKZ algorithm with block size $\beta$; a quantum BKZ attack costs about

$$2^{0.265\,\beta}.$$

ML-KEM-768 uses $n = 256$, $q = 3329$, $k = 3$ and gives about 164 bits of quantum security, comfortably NIST Level 3.

**Mathematical tools**

- **Number Theoretic Transform (NTT)** for $O(n\log n)$ polynomial multiplication
- **Gaussian heuristic:** $\lambda_1(\Lambda) \approx \sqrt{\tfrac{n}{2\pi e}}\ \mathrm{vol}(\Lambda)^{1/n}$
- **Gröbner bases** for analysing MQ systems

**Attack models:** quantum (Grover reduces hash security by half; there is no Shor-like exponential speed-up known for SVP, LWE or syndrome decoding), structural breaks (SIKE, Rainbow), and implementation attacks (timing, cache, power analysis of NTT butterflies, fault injection), which require constant-time implementations.

## 6. Size and Speed

| Scheme | Family | Public key | Private key | Sig / ciphertext | NIST level |
|--------|--------|-----------|-------------|------------------|-----------|
| RSA-2048 | IFP | 256 B | ~1,200 B | 256 B | n/a |
| ECDSA P-256 | ECDLP | 64 B | 32 B | 64 B | n/a |
| ML-KEM-768 | Lattice | 1,184 B | 2,400 B | 1,088 B | 3 |
| ML-DSA-65 | Lattice | 1,952 B | 4,000 B | 3,293 B | 3 |
| FN-DSA-512 | Lattice | 897 B | 1,281 B | 666 B | 1 |
| SLH-DSA-128f | Hash | 32 B | 64 B | 17,088 B | 1 |
| Classic McEliece | Code | 261,120 B | 6,492 B | 128 B | 1 |
| SQIsign | Isogeny | ~64 B | ~782 B | ~177 B | 1 |

Approximate speeds in thousands of cycles (x86-64 with AVX2):

| Scheme | KeyGen | Enc / Sign | Dec / Verify |
|--------|--------|-----------|--------------|
| ML-KEM-768 | ~47 | ~56 | ~59 |
| ML-DSA-65 | ~125 | ~195 | ~122 |
| FN-DSA-512 | ~8,000 | ~900 | ~43 |
| SLH-DSA-128f | ~900 | ~10,000 | ~420 |
| Classic McEliece | ~1,500,000 | ~120 | ~130 |

## 7. Shortcomings

| Shortcoming | Core issue | Severity |
|-------------|-----------|----------|
| Size and performance overhead | Large keys and ciphertexts, higher compute cost | High (near-term) |
| New attack surfaces | Extra algebraic structure enables new attacks | Medium–High |
| Implementation risks | Side channels, constant-time difficulty | High |
| Diversity risk | Standards concentrated on lattices | Medium (long-term) |
| Migration complexity | Retrofitting infrastructure, hybrid deployment | Critical |
| Theoretical gaps | Security rests on unproven assumptions | Medium–High |

Key point: no security proof rules out efficient classical or quantum algorithms for LWE, Ring/Module-LWE or SIS. The collapse of SIKE shows that young assumptions can fail suddenly.

## 8. Future Prospects

- **Standards:** finalising FN-DSA and HQC, plus NIST's on-ramp for non-lattice signatures (SQIsign, MAYO, UOV)
- **Constructions:** fully homomorphic encryption (BFV, CKKS, TFHE, built on Ring-LWE), post-quantum zero-knowledge proofs (STARKs from hash commitments, lattice-based schemes), SQIsign
- **Engineering:** hardware NTT acceleration, formal verification (HACL*, EverCrypt in F*), layering with QKD
- **Mathematics:** the quantum hardness of SVP is the central open question. A QMA-hardness proof would strengthen confidence in lattices, while a sub-exponential quantum algorithm for structured ideal lattices would force parameter changes.

## 9. Notes on the Source Documents

- **BKZ block size:** the report states $\beta = 177$ for ML-KEM-768, but $2^{0.265\cdot177}\approx 2^{47}$, not the quoted ~164 bits. Reaching 164 bits needs $\beta \approx 164/0.265 \approx 620$. Check this before citing it.
- **Two definitions of "level":** the report defines NIST levels by AES-equivalent cost, while the slides list 96 quantum bits for Level 3 as a halving of the classical 192. They are different conventions and should not be mixed.
- **FIPS 206 status:** the report lists FN-DSA under "in development", but the slides list it among current standards. Treat it as in development unless you confirm a final publication.
- The timing chart in the slides (Kyber, McEliece, SIDH in milliseconds) looks illustrative and is not reproduced here. The cycle counts in Section 6 come from the report.

## 10. Repository Contents

```
.
├── README.md
├── Crypto-3-1.pdf                  # Group project report (12 pages)
└── Post-Quantum-Cryptography.pdf   # Presentation slides (22 slides)
```

## 11. References

1. NIST, FIPS 203/204/205, August 2024. https://csrc.nist.gov/projects/post-quantum-cryptography
2. O. Regev, "On Lattices, Learning with Errors, Random Linear Codes, and Cryptography," *J. ACM* 56(6), 2009.
3. R. J. McEliece, "A Public-Key Cryptosystem Based On Algebraic Coding Theory," DSN Progress Report, 1978.
4. P. W. Shor, "Algorithms for Quantum Computation: Discrete Logarithms and Factoring," *FOCS*, 1994.
5. W. Castryck and T. Decru, "An Efficient Key Recovery Attack on SIDH," *EUROCRYPT*, 2023.
6. W. Beullens, "Breaking Rainbow Takes a Weekend on a Laptop," *CRYPTO*, 2022.
7. C. Peikert, "A Decade of Lattice Cryptography," *Foundations and Trends in TCS* 10(4), 2016.
8. NSA, "Commercial National Security Algorithm Suite 2.0," CNSSP No. 15, 2022.
