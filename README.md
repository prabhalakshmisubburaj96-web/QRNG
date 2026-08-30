# QRNG
Quantum Random Number Generator

# Quantum Random Number Generator (QRNG)


**Tools:** Qiskit, Python (NumPy, SciPy) &nbsp;|&nbsp; **Hardware:** IBM Quantum (Kingston, Fez)

A quantum random number generator built and evaluated end-to-end on real IBM Quantum
hardware. A qubit is prepared in superposition via a Hadamard gate and measured; because
real NISQ hardware is noisy, the raw output is biased, so this project goes beyond basic
bit generation to characterize that bias, remove it with purpose-built randomness
extractors, and test whether entanglement-based generation offers any advantage over
single-qubit generation.

## What this project does

1. **Bias & backend comparison** — surveys 12 physical qubits (6 on IBM Kingston, 6 on
   IBM Fez) to characterize raw measurement bias and identify hardware anomalies.
2. **Randomness extraction** — implements and benchmarks four extractors (Von Neumann,
   a custom windowed-XOR scheme, a Toeplitz universal-hash extractor, and a SHA-256
   cryptographic conditioner) against the worst-case biased qubit found in step 1.
3. **Entanglement vs. single-qubit generation** — prepares a Bell pair and compares the
   statistical quality of entanglement-based bit generation against plain single-qubit
   generation, on both backends.

## Repository contents

| File | Description |
|---|---|
| `QRNG_sanitized.ipynb` | Main notebook — all code for Tasks 1-3. Credentials and job IDs are placeholders (see **Setup** below). |
| `QRNG_Project_Report.docx` | Full write-up: literature survey, methodology, complete per-qubit results, statistical analysis, discussion, limitations, and references. |
| `QRNG_Project_Slides.pptx` | Slide-deck summary of the project and findings. |
| `QRNG_job_id_reference.md` | Reference table of original IBM Quantum Job IDs used to produce the results in the report (for the author's own reproducibility — see note below). |

## Setup

### 1. Install dependencies
```bash
pip install qiskit qiskit-ibm-runtime qiskit-aer numpy scipy
```

### 2. Configure IBM Quantum credentials
This notebook does **not** contain any API token or instance CRN — they were removed
before publishing. To run it yourself:

1. Create a free account at [IBM Quantum Platform](https://quantum.ibm.com/).
2. Copy your API token and CRN instance string from your account settings.
3. In the notebook's setup cell, replace the placeholders:
   ```python
   QiskitRuntimeService.save_account(
       token="YOUR_API_TOKEN_HERE",
       instance="YOUR_CRN_INSTANCE_HERE",
       channel="ibm_quantum_platform",
       overwrite=True,
       set_as_default=True
   )
   ```
   with your own token and instance string.

**Never commit real credentials to this repository.** If you accidentally do, revoke
and regenerate the token immediately from your IBM Quantum account settings.

### 3. Job IDs
Every `service.job("YOUR_JOB_ID_HERE")` call in the notebook retrieves a previously
submitted hardware job. These placeholders won't resolve to anything — you'll need to
either:
- Run the corresponding `sampler.run(...)` submission cell (commented out just above
  each retrieval line) to submit a fresh job under your own account, or
- Substitute your own previously submitted Job ID if you're re-running an existing job.

## Key findings

**Task 1 — Bias survey:** Kingston Q0 showed severe bias (21.34%, min-entropy 0.4872),
and Kingston Q1 a secondary anomaly (8.90%). All other qubits on both backends stayed
below 2% bias; Fez was consistently the cleaner backend overall.

**Task 2 — Extractor benchmark** (on the worst-case qubit, 10,000 raw bits):

| Extractor | Efficiency | Bias | Chi-square verdict |
|---|---|---|---|
| Von Neumann (baseline) | 20.4% | 0.54% | Pass |
| Custom windowed-XOR | 33.3% | 4.07% | Fail |
| Toeplitz-hash | 48.1% | 0.69% | Pass |
| SHA-256 block | 48.6% | 0.23% | Pass |

Adaptive extractors (Toeplitz, SHA-256) roughly double Von Neumann's efficiency while
preserving statistical uniformity, and approach ~97-99% efficiency on a clean qubit.

**Task 3 — Entanglement vs. single-qubit generation:** Bell-pair generation showed no
randomness advantage over single-qubit generation on either backend. An initially
extreme result on Kingston (8.13% marginal bias) was traced to the entangling circuit
accidentally using the anomalous Q0 from Task 1; once excluded, Kingston's result
(1.21% bias) closely matched Fez's (1.10%), consistent with ordinary two-qubit gate
error rather than any property of entanglement itself.

See `QRNG_Project_Report.docx` for full methodology, statistical analysis (confidence
intervals, significance tests), literature survey, and discussion.

## References

Key references are listed in full in the project report. Primary reference:

Herrero-Collantes, M. & Garcia-Escartin, J. C. (2017). Quantum random number
generators. *Reviews of Modern Physics*, 89, 015004.
https://doi.org/10.1103/RevModPhys.89.015004

## License

Add a license of your choice (e.g., MIT) before publishing, if you intend this
repository to be reused by others.
