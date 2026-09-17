Master Development Plan — v3

Building, Validating & Verifying Three Quantum Algorithms

Algorithms in Scope:
1. Quantum Tensor Networks (MPS-based quantum circuit simulation & compression)
2. Quantum Spectral Clustering (QPCA + eigenvector extraction for clustering)
3. Quantum Reinforcement Learning (VQC-based agent)

Revision Basis: Independent technical review of v2 (score 8/10). v3 closes 5 conditions for execution readiness:

- C1: QPCA hardware qubit budget made consistent (explicit system/ancilla split; iterative QPE; tolerance tied to Trotter steps)
- C2: `SPEC_QPCA_KERNEL.md` — explicit mathematics of the swap-test / quantum-kernel pipeline
- C3: Pre-registration frozen at G1 (Week 12), not Week 24
- C4: Compute budget re-estimated to 3,500 CPU-hours base
- C5: GridWorld action encoding specified; latency audited for all algorithms; G3 fallback path pre-declared

Plan Horizon: 40 Weeks (10 Months)

---

Part 1 — Definitions, Scope & Success Criteria

1.1 Objective
Design, implement, and scientifically verify three quantum algorithms through a staged pipeline:
Classical Simulation → Noisy Simulation → Real Quantum Hardware → Formal Benchmarking.

1.2 Explicit Scope of Each Algorithm

Algorithm	In Scope	Out of Scope	
Tensor Networks	(a) MPS simulation of quantum circuits; (b) MPS/TT compression of classical weight matrices; (c) execution of one depth-reduced compressed circuit on hardware	Classical-only compression without quantum circuit execution	
QPCA	Eigenvalues and clustering-usable outputs of a low-rank covariance matrix on 8 qubits hardware budget (5 system + 3 ancilla, iterative QPE; see §4.B and SPEC_QPCA_KERNEL.md)	Full-rank PCA; regimes beyond 8 qubits; claims of asymptotic advantage	
QRL	VQC agent on CartPole-v1 (simulator only) and 4×4 GridWorld with 2-qubit one-hot action encoding (simulator + hardware)	Continuous-control tasks on real hardware; real-time closed-loop control on hardware	

1.3 Definition of "Verified"
- V1 (Correctness): Output matches classical reference within tolerance at specified shot budget and confidence level.
- V2 (Reproducibility): ≥ 10 fixed seeds; bootstrap 95% CI; std below per-algorithm threshold.
- V3 (Noise Bounds): Performance vs. noise documented under the extended model §2.4.
- V4 (Resource Audit): Qubits, two-qubit depth, total shot count, parameter count, transpiled gates, and wall-clock latency for all algorithms.
- V5 (Ablation): Each component shown necessary via component-specific ablations (§4 per-track tables).

1.4 No Quantum Advantage Claims
All results are proof-of-concept benchmarks on NISQ hardware. Any speedup claim requires a classical baseline under identical accuracy constraints, run on the same datasets.

---

Part 2 — Verification & Testing Standards

2.1 Test Environment (pinned)

Item	Standard	
Language	Python 3.11.x (exact patch pinned)	
Classical	numpy 2.x, scipy 1.x, matplotlib, networkx	
Quantum SDKs	Qiskit 1.3.x + qiskit-aer; PennyLane 0.39.x; Cirq 1.5.x	
Tensor libs	Quimb 1.8.x, TensorNetwork (google)	
Simulators	`aer_simulator_statevector`, `aer_simulator`, `qsim`, `lightning.qubit`	
CI	GitHub Actions, weekly scheduled re-test	
Testing	`pytest` + `hypothesis` + custom benchmark harness	

2.2 Fixed, Versioned Datasets (committed; SHA-256 in `DATASETS.md`)

Task	Dataset	
Clustering	`data/synth_circles_v1.npz`; `data/mnist1k_v1.npz`	
RL	`CartPole-v1` (sim only); `GridWorld4x4_v1` (custom wrapper, deterministic seeds)	
Tensor Networks	`data/tn_states_v1/` (GHZ, W, random-MPS with known entanglement spectra)	

2.3 Statistical Protocol (mandatory)
- Seeds: ≥ 10 fixed seeds per experiment, recorded in `seeds.json`.
- Shots: every shot-based result reports its shot budget; convergence-vs-shots curves mandatory for QPCA and QRL.
- Confidence: bootstrap 95% CI on the primary metric; mean ± std reported.
- Pre-registration: frozen at G1 (Week 12) in `PREREGISTRATION.md`: hypotheses, primary metrics, thresholds, fallback rules, and deviation-log procedure. No post-G1 changes except via written deviation log.

2.4 Extended Hardware Noise Model

Parameter	Value	
T1 / T2	100 µs / 150 µs	
1q / 2q gate error	1e-3 / 1e-2	
Readout error	2%	
Gate durations	1q: 30 ns, 2q: 300 ns	
Topology	Fixed 127-qubit heavy-hex coupling map; all circuits transpiled under real connectivity	
Crosstalk	Nearest-neighbor 2q error inflation +50% (documented simplification; sensitivity sweep ±50% applied)	
Drift	±20% parameter variation over simulated 24 h window	
Measurement latency	1 µs readout; latency audited for all algorithms as wall-clock per experiment (V4)	

2.5 Numeric Tolerances (shot- and seed-aware)

Quantity	Tolerance	
MPS reconstruction error (exact)	≤ 1e-10 (10 qubits)	
Compressed TN relative error @ target bond dim	≤ 5% (10 seeds, CI half-width ≤ 1%)	
QPCA eigenvalue error (statevector sim)	≤ 1e-4	
QPCA eigenvalue error (noisy sim, ITE-QPE, m=3, r=2 Trotter steps)	≤ 0.08 (10 seeds)	
QPCA eigenvalue error (hardware, ≥ 200k shots)	≤ 0.10 vs. noisy-sim reference, within G3 margin	
QPCA clustering ARI vs. classical spectral clustering (sim)	≥ 90% of classical ARI	
QPCA clustering ARI (hardware)	≥ 85% of classical ARI	
QRL (sim): success over last 100 episodes	≥ 90% (10 seeds)	
QRL (hardware GridWorld): success	≥ 80%; within 15% of noisy-sim (fallback rule §2.6)	
Hardware vs. noisy-simulator primary metric	within 15% (G3); if in 15–25%, fallback analysis §6.3	

2.6 Pre-Declared Fallbacks (in PREREGISTRATION.md)

Condition	Pre-declared response	
G3 margin in (15%, 25%]	Accept with root-cause analysis (leakage, correlated/non-Markovian noise, calibration drift) documented	
G3 margin > 25%	Re-scope: reduce qubit count (QPCA → 5q system only / QRL → 4q); re-run; document	
QPCA ITE-QPE fails tolerance at Week 22	Increase Trotter steps r (pre-registered values r = 2, 4, 8); if still failing, QPCA hardware stage reduced to eigenvalue-only demo + classical eigenvectors	
Crosstalk model uncertainty material	Report results at crosstalk ±50% as sensitivity band	

2.7 Go / No-Go Gates

Gate	Timing	Criterion	
G0	Week 4	Team, pinned env, hashed datasets, compute budget signed	
G1	Week 12	V1–V2 met (≥ 10 seeds) on classical simulator for all 3 tracks; PREREGISTRATION.md frozen	
G2	Week 24	Extended-noise report; ≥ 80% of ideal; QPCA kernel pipeline validated on simulator	
G3	Week 34	Hardware runs within 15% of noisy sim (fallback §2.6)	
G4	Week 40	Verification matrix signed; pre-registered analysis executed with deviation log	

---

Part 3 — Phase 0: Foundations (Weeks 1–4)

Week	Task	Deliverable	Acceptance	
1	Team setup	Roles doc	—	
1–2	Study: spectral theory/SVD; QM; QI theory	Reading log	Quiz ≥ 80%	
2–3	Pinned environment + CI	`environment.yml`, CI badge	CI green on clean machine	
3–4	Datasets hashed; `seeds.json`; SPEC_QPCA_KERNEL.md drafted; compute budget	repo artifacts	SPEC approved internally; budget signed	

SPEC_QPCA_KERNEL.md (normative summary — full math in repo)
Pipeline (Lloyd–Mohseni–Rebentrost style, adapted to 8 qubits):

1. Input encoding: data matrix X (N×d) row-normalized; state preparation circuit `U(x_i)` produces |x_i⟩ on n = 5 system qubits (d ≤ 32 features after classical PCA pre-compression to 5 principal components — pre-registered).
2. Density matrix: ρ = (1/N) Σ_i |x_i⟩⟨x_i| prepared as a mixed state via heralded preparation on the same 5 qubits.
3. Eigenvalues: Iterative QPE on m = 3 ancilla qubits (total 8), one eigenphase refinement at a time; unitary e^{−iρt} implemented by r = 2 Trotter steps of e^{−iρt/r} using the density-matrix exponentiation trick with copies of ρ (copy budget pre-registered: ≤ 8 copies per run).
   - Resolution: Δφ = 2π/2³ = 0.785 rad → Δλ ≈ Δφ/t with t = 2π → Δλ ≈ 0.125 raw; ITE refinement + r = 2 yields target Δλ ≤ 0.08 in simulation (validated empirically at Week 22; tolerance re-confirmed against measured Trotter error, not assumed).
4. Clustering-usable output — quantum kernel on data points (chosen method, pre-registered):
   - For the top-k eigenvectors of ρ (k = 2), compute kernel entries K_ij = ⟨x_i|ρ|x_j⟩ estimated via swap-test between |x_i⟩ and ρ|x_j⟩ (preparation of ρ|x_j⟩ via one copy of ρ).
   - What is measured: swap-test gives |⟨ψ|φ⟩|²; with φ = ρ|x_j⟩, K_ij is estimated from statistics of the ancilla measurement over S shots (S ≥ 200k hardware; convergence-vs-S curve mandatory).
   - Affinity matrix: A = K + Kᵀ (symmetrized); ARI computed against classical spectral clustering on the same Gram matrix (not against k-means on raw data).
   - Theoretical guarantee: K built from the exact ρ equals the classical Gram matrix of projected data; with shot noise and Trotter error, deviation bounded by (Trotter error + 1/√S) — bound computed and reported, guaranteeing the quantum pipeline approximates the same spectral embedding as the classical reference up to a stated error.
5. Explicitly rejected alternative (recorded): full tomography of eigenvectors — exponential cost in n, incompatible with 8-qubit budget and shot budget.
6. Ablation targets (V5): Trotter steps r ∈ {1, 2, 4, 8}; ancilla precision m ∈ {2, 3}; eigenvector-rank k ∈ {1, 2, 3}; kernel method vs. classical-Gram baseline.

---

Part 4 — Phase 1: Classical Simulation (Weeks 5–12)

Sequencing rule: parallel tracks; sequential gate reviews TN (Wk 10) → QPCA (Wk 11) → QRL (Wk 12). Verification resources follow that priority.

4.A Tensor Networks (MPS)

Week	Task	Verification Standard	
5–6	MPS via successive SVD from scratch	Rel. error ≤ 1e-10, 10–12 qubits; unit tests: canonical form, isometry	
7–8	Expectation values + entanglement entropy	Agreement with exact diagonalization ≤ 1e-9	
9–10	Bond-dim truncation; V5 ablation: bond dimension χ ∈ {4, 8, 16, 32}	≤ 5% error at target χ (10 seeds, CI ≤ 1%)	
11–12	Compress small LM layer + transpile one compressed circuit	Accuracy drop ≤ 2% at ≥ 50× compression; 2q depth logged	

4.B QPCA (per SPEC_QPCA_KERNEL.md)

Week	Task	Verification Standard	
5–6	Classical baseline: PCA-5 + spectral clustering on §2.2 datasets	ARI recorded as reference	
7–8	ρ preparation; ITE-QPE on simulator (statevector)	Eigenvalue error ≤ 1e-4 vs. `eigh` (n=5, m=3, r=2)	
9–10	Quantum-kernel pipeline: swap-test Gram estimation, A construction, clustering	Sim ARI ≥ 90% of classical ARI (statevector, exact K)	
11–12	Shot-noise version (S sweep) + integrated 8-qubit pipeline	Shot-convergence curve; shot-based ARI within pre-registered band	
V5 ablations	r, m, k sweeps (§3, item 6)	Each ablation table committed	

4.C QRL
GridWorld encoding (normative): 16 states → 4 qubits one-hot-ish binary encoding (|s⟩ with s ∈ {0..15}); actions: 2 qubits one-hot ({00,01,10,11} → {up,down,left,right}); policy/value head measured on the action register. Total circuit: 6 qubits (4 state + 2 action) on simulator; hardware version: action register reduced to 1 qubit + relabeled 2-action subset {up,right} (5 qubits total) — pre-registered simplification for hardware.

Week	Task	Verification Standard	
5–6	Classical DQN, CartPole-v1	Solved ≤ 500 episodes	
7–8	VQC design; Barren plateau screen: grad variance at init > 1e-6	Depth/params logged	
9–10	VQC value function, CartPole, statevector sim	Success ≥ 90% (10 seeds, CI reported)	
11–12	GridWorld VQC (6q sim; 5q hardware variant designed); A/B: NN vs VQC	Convergence, stability, param counts reported	
V5 ablations	Ansatz depth L ∈ {1, 2, 4}; encoding: amplitude vs. angle	Ablation tables committed	

Gate G1 (Week 12): V1 + V2 met; PREREGISTRATION.md frozen.

---

Part 5 — Phase 2: Noisy Simulation (Weeks 13–24)

Week	Task	Verification Standard	
13–14	Implement §2.4 (durations, topology, crosstalk ±50%, drift, latency)	Model cross-checked vs. IBM 127q datasheet	
15–16	QPCA: ITE-QPE + kernel under noise; Trotter-error measurement (r sweep under noise)	Measured eigenvalue error at r=2 reported; tolerance §2.5 re-confirmed or fallback §2.6 executed	
17–18	QRL: measurement latency + shots scaling (GridWorld)	Shot budget for 80% success identified	
19–20	TN: χ Pareto front under noise	Optimal χ documented	
21–22	Drift + crosstalk sensitivity sweeps (all 3)	Sensitivity tables; QPCA go/no-go for hardware re-confirmed	
23–24	Expected Performance Bounds Report	G2 board sign-off	

Gate G2 (Week 24): ≥ 80% of ideal under §2.4; QPCA kernel validated; hardware re-scoping decisions logged per pre-registered rules.

---

Part 6 — Phase 3: Real Hardware (Weeks 25–34)

Week	Task	Verification Standard	
25–26	IBM access; calibration (Bell, GHZ)	Bell fidelity ≥ 90% (100k shots)	
27–28	QPCA 8q (5 sys + 3 anc): ITE-QPE eigenvalues + kernel clustering	Eigenvalue error ≤ 0.10 vs. noisy-sim reference; ARI ≥ 85% of classical	
29–30	GridWorld 5q VQC (2-action subset) on hardware	Success ≥ 80%; within 15% of noisy-sim (fallback §2.6)	
31–32	Compressed TN circuit on hardware	2q depth, fidelity, latency logged	
33–34	V4 audit on real device; 24h drift re-run	Audit complete; rerun within pre-registered drift band	

Gate G3 (Week 34): within 15% of noisy-sim, or §2.6 fallback executed and documented.

---

Part 7 — Phase 4: Formal Verification & Benchmarking (Weeks 35–40)

7.1 Verification Matrix

Criterion	Tensor Networks	QPCA	QRL	
V1	≤ 5% error @ χ (CI ≤ 1%)	eigenvalue ≤ 0.08 (sim) / 0.10 (HW); ARI ≥ 90%/85% of classical	success ≥ 90% sim / 80% HW	
V2	10 seeds, bootstrap CI	10 seeds, bootstrap CI	10 seeds, bootstrap CI	
V3	Phase 2 report	Phase 2 report (incl. measured Trotter error)	Phase 2 report	
V4	qubits, 2q depth, params, latency	qubits, 2q depth, shots, copy budget, latency	qubits, 2q depth, shots, latency	
V5	χ ablation	r, m, k, kernel ablations	ansatz-depth, encoding ablations	

7.2 Deliverables

Week	Deliverable	
35–37	Full test pyramid re-run; fixes; code freeze	
38	Verification matrix signed (G4); pre-registered analysis executed; deviation log published	
39	Technical report + tagged release v3.0	
40	Executive summary + per-algorithm investment recommendation	

---

Part 8 — Compute Budget

Phase	Workload	Estimate	
Phases 1–2	Statevector ≤ 16q; noisy sims ≤ 10q; 10 seeds; sweeps (r, m, k, χ, L, S)	3,500 CPU-hours base; 1× mid-range GPU 300 h (qsim/lightning)	
Phase 3	IBM Quantum free tier; 200k shots/algorithm; fallback cloud cap	0; ≤ 500 if triggered	
Storage/CI	Artifacts, datasets, logs	50 GB	
Contingency	+25% on CPU line	875 CPU-hours	
Overrun policy	20% overrun at any Gate	scope reduction per priority TN → QPCA → QRL (pre-registered)	

---

Part 9 — Risk Register (quantitative triggers)

Risk	Likelihood	Trigger	Contingency	
Barren plateaus (VQC)	High	grad variance < 1e-6 at Wk 8	layer-wise / local-cost ansatz (pre-approved)	
Hardware access limits	Medium	queue > 48 h or quota exhausted	noisy-sim evidence; cloud cap 500	
ITE-QPE accuracy insufficient	Medium	eigenvalue error > 0.08 at r=2 (Wk 15–16)	r ∈ {4, 8} pre-registered; else eigenvalue-only hardware demo (§2.6)	
Crosstalk/drift exceeds model	Medium	G3 margin > 15%	§2.6 fallback path	
Copy budget for ρ exceeded	Medium	8 copies/run needed for kernel SNR	reduce k to 2; raise S within shot cap; document	
Team bandwidth	Medium	any track slips > 2 weeks	priority order TN → QPCA → QRL	
SDK drift	Medium	weekly CI failure	pinned versions; upgrades only at phase boundaries	

---

Part 10 — Milestone Summary

Milestone	Week	Evidence	
M0 Foundations	4	pinned env, hashed data, SPEC approved, budget signed	
M1 Classical prototypes + frozen pre-registration	12	V1–V2 (10 seeds); PREREGISTRATION.md frozen	
M2 Noise characterization	24	extended-noise report; measured Trotter error; QPCA go/no-go	
M3 Hardware execution	34	8q QPCA, 5q GridWorld, TN circuit; G3 margin documented	
M4 Full verification	40	signed matrix; pre-registered analysis + deviation log; release v3.0	

---

Appendix A — Changes v1 → v2
(unchanged from v2; see v2 document)

Appendix B — Changes v2 → v3
1. QPCA hardware budget fixed and made internally consistent: 5 system + 3 ancilla qubits (8 total); iterative QPE replaces full QPE; r = 2 Trotter steps; eigenvalue tolerance set to ≤ 0.08 (sim, measured not assumed) and ≤ 0.10 (hardware vs. noisy-sim reference); copy budget ≤ 8/run.
2. SPEC_QPCA_KERNEL.md added (Part 3): explicit pipeline — what is measured (swap-test overlaps), how the affinity matrix is built (symmetrized Gram from K_ij = ⟨x_i|ρ|x_j⟩), and the error bound guaranteeing equivalence to classical spectral clustering up to (Trotter error + 1/√S). Tomography explicitly rejected and recorded.
3. Pre-registration moved to G1 (Week 12); deviation-log procedure added; fallback rules pre-declared (§2.6) to prevent post-hoc rationalization.
4. Compute budget re-estimated: 3,500 CPU-hours base (+25% contingency) with documented workload basis.
5. GridWorld encoding specified: 4 state qubits + 2 action qubits one-hot (simulator); 5-qubit hardware variant with 2-action subset, pre-registered.
6. Latency audit (V4) extended to all three algorithms.
7. QPCA V5 ablations redefined: Trotter steps r, ancilla precision m, rank k, kernel-vs-classical-Gram — replacing the unclear "encoding vs. no-encoding".
8. G3 fallback path pre-declared: 15–25% margin → accept with root-cause analysis; > 25% → re-scope to fewer qubits and re-run.
9. Crosstalk simplification acknowledged with ±50% sensitivity band requirement.