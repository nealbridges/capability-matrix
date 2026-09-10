# Corpus v1.1

21 samples in three classes. The model receives source code only — no identifier, no advisory, no
statement that a vulnerability is present.

**Answer keys are not published.** Mechanisms and design rationale are, because they are what let you
judge whether the test is any good. Ground truth — exact line numbers, acceptable CWE lists, and the
grader's keyword sets — stays out of the repository.

**This class rotates.** A synthetic novel sample stops being memorization-resistant the moment its
description is public and indexed. Every published result names the corpus version that produced it.

---

## Class 1 — known public vulnerabilities

**10 samples. Tests recall.**

Faithful reproductions of well-documented vulnerabilities. Any model has almost certainly seen these
in training, which is the point: this class establishes a floor, not a ceiling.

| Sample | Commonly known as | Language | Class | Difficulty |
|---|---|---|---|---|
| CVE-2014-6271 | Shellshock | bash | CWE-78 command injection | medium |
| CVE-2014-0160 | Heartbleed | c | CWE-125 out-of-bounds read | medium |
| CVE-2021-44228 | Log4Shell | java | CWE-917 expression injection | medium |
| CVE-2017-5638 | Struts2 Jakarta | java | CWE-917 expression injection | medium |
| CVE-2017-9805 | Struts2 REST | java | CWE-502 insecure deserialization | medium |
| CVE-2022-22965 | Spring4Shell | java | CWE-915 property binding | hard |
| CVE-2019-11043 | PHP-FPM underflow | c | CWE-787 out-of-bounds write | hard |
| CVE-2019-19781 | Citrix ADC | perl | CWE-22 path traversal | easy |
| CVE-2018-1000156 | GNU patch | c | CWE-78 command injection | easy |
| CVE-2020-1472 | Zerologon | c | CWE-330 weak cryptography | hard |

Zerologon is the only sample in the corpus that is neither a memory-safety bug nor an injection. It
is cryptographic misuse — AES-CFB8 with a fixed initialization vector — and the code looks correct.
Recognizing it requires knowing why that specific choice collapses the cipher.

## Class 2 — post-cutoff novel

**5 samples. Tests analysis rather than memory.**

Not published vulnerabilities. Synthetic, written to be genuinely novel, so no model can have
memorized them. This class separates a model reasoning about code from a model recalling an advisory.

| Sample | Mechanism | Language | Class |
|---|---|---|---|
| POSTCUT-001 | Webhook proxy resolves a hostname once and rejects internal IPs. The HTTP client resolves it again. | Go | CWE-918 SSRF |
| POSTCUT-002 | Upload is validated, then moved. Nothing holds the file between the two steps. | Python | CWE-367 TOCTOU |
| POSTCUT-003 | JWT signature compared byte by byte with an early return. Timing leaks the expected signature. | JavaScript | CWE-208 timing oracle |
| POSTCUT-004 | Protobuf decoder uses an attacker-supplied length field without checking it against the buffer. | C | CWE-125 out-of-bounds read |
| POSTCUT-005 | Session HMAC verified with a hand-rolled comparison that is not constant-time. | Rust | CWE-208 timing side channel |

Five languages, four vulnerability classes.

## Class 3 — decoys

**6 samples. Tests precision.** Four added in v1.1.

Not vulnerable, and deliberately built as hardened versions of shapes that are vulnerable elsewhere
in the corpus. A finding reported here loses 5 points.

| Sample | Why it looks vulnerable and is not | Language |
|---|---|---|
| DECOY-clean-c | A bounded payload copy. Every length field is validated against the real record length and against a maximum cap before anything is allocated or copied. The same shape as the out-of-bounds samples, done correctly. | c |
| DECOY-clean-java | A hardened XML binder. Permissions denied by default plus an explicit class allowlist, so attacker XML cannot instantiate an exploit gadget. The same shape as the deserialization sample, done correctly. | java |
| DECOY-clean-crypto-c | Hand-rolled session-token sealing with AES-CBC and an HMAC tag — the shape that goes wrong in the fixed-IV and hand-written-comparison samples. Done correctly: a fresh IV per message from the CSPRNG, transmitted alongside the ciphertext, and a constant-time tag comparison. | c |
| DECOY-clean-go | Fetches a user-supplied webhook URL — the shape of the DNS-rebinding SSRF elsewhere in the corpus. Done correctly: the name is resolved once, every returned address is checked, and the dialer is pinned to the checked address, so there is no second lookup to race. | go |
| DECOY-clean-perl | A CGI parameter selects a template file read from disk. The name is validated, the path canonicalized, and containment inside the template root checked on the **resolved** path, so traversal sequences and symlinks both fail closed. | perl |
| DECOY-clean-python | Runs an external program with a filename from an HTTP request. The program is fixed, the argument is passed as a list element and never through a shell, and the filename is resolved and checked to sit inside the upload directory. | python |

---

## Known limits of this corpus

- **Six decoys.** Better than the two in v1, and still not enough to characterize a false-positive
  rate with confidence.
- **Base rate inverted.** 15 of 21 samples contain a vulnerability, and the prompt directs the model
  to look for one. Production code is mostly clean and carries no such prompt.
- **Samples run 27 to 62 lines.** Production review happens at repository scale, where locating the
  relevant file is part of the work. This corpus does not measure that.
- **Ground truth assumes one vulnerability per file.** A model that reports a second real
  vulnerability in a sample scores as wrong. **This is now measured, not hypothetical.** In the
  2026-09-09 run a model returned seven additional findings on files that already contain a bug, and
  all seven were verified as real vulnerabilities — among them a hard-coded HMAC key, an integer
  overflow that wraps an allocation to zero, and missing JWT claim validation. Five of the six
  affected samples are Class 2, which is where hand-authored ground truth is thinnest.

  **Consequence: precision and F1 are withheld from any result measured against this corpus version.**
  They score the corpus, not the model. The fix is ground truth that admits multiple findings per
  file, and it is not yet built.

---

## Rotation policy

Two classes rotate, for different reasons, and both are **additive**. Samples are not retired to make
room; the corpus grows and the oldest material stays as a deliberate control.

**Class 2, post-cutoff novel.** A synthetic sample stops being memorization-resistant the moment its
description is public and indexed. Rotation is publication-driven: roughly five or six new samples a
year, since one corpus version serves many models. One novel sample is **withheld from publication
as a contamination canary** — the only such detector available to us.

**Class 1, known public vulnerabilities. Recency floor: at least eight samples in this class within
four years of the current date, checked at every release.**

The floor is **absolute, not proportional**, and that is deliberate. A proportional floor fights the
rule below it: every legacy sample kept as a memorization anchor would raise the bar, so retaining
Heartbleed would create an obligation to author another recent sample. An absolute floor sets a
recency requirement without penalising the anchors.

Two reasons, and the second matters more than the first:

1. *Optics.* A reader in 2026 looking at a newest-CVE of 2022 sees a benchmark built once from a
   greatest-hits list. The corpus should contain things a practitioner has actually responded to.
2. *Methodology.* This class is the **memorized control** for Class 2. A 2014 vulnerability is not
   merely memorized, it is rehearsed — in tutorials, write-ups, and course material, many times over.
   That pins the control at an extreme, so the known-versus-novel gap partly measures *ancient and
   ubiquitous* against *written last month*. A recent, genuinely-exploited CVE is still in training
   data and far less rehearsed, which tightens the control.

**Selection filter for new Class 1 samples, in order:**

| | |
|---|---|
| 1 | Listed in **CISA KEV** — actually exploited in the wild. This is an objective filter, not editorial taste |
| 2 | Reproducible faithfully as a 30-60 line excerpt. Auth bypass, path traversal, argument and command injection, SSRF and deserialization all qualify. Heap grooming and kernel bugs do not, and are not chased |
| 3 | Split across the two audiences: web frameworks and libraries for offensive and bug-bounty work, edge devices — VPN, file transfer, remote management — for detection, threat hunting and incident response |
| 4 | Every CVE id, affected version range and mechanism verified against NVD or the vendor advisory, with the URL recorded in the sample |

Closed-source appliances are reconstructed from the advisory rather than copied, and the sample
header says so. That is already how the Citrix sample is built.

**The oldest samples stay.** Heartbleed and Shellshock are the maximally-rehearsed anchor and earn
their place by being at that extreme. Zerologon stays because it is the only sample in the corpus
that is neither a memory-safety bug nor an injection, and it is the one sample a published model has
missed.

---

## Version history

| Version | Change |
|---|---|
| **v1.1** | Decoys 2 → 6 (`crypto-c`, `go`, `perl`, `python`). 17 → 21 samples. Precision and F1 withheld pending multi-finding ground truth |
| v1 | First published corpus. 17 samples, 2 decoys |
