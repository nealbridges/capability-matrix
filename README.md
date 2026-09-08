# Capability Matrix

Security capability evaluation of locally served models.

Most published model comparisons measure throughput. This measures whether the model is any good at
the work — finding vulnerabilities, classifying them correctly, and staying quiet on code that is
fine.

## What is here

| | |
|---|---|
| **[METHOD.md](METHOD.md)** | How the numbers are made. Metric definitions, the scoring gates, and the rules a result has to follow. Read this first. |
| **[CORPUS.md](CORPUS.md)** | What is in the test. Sample classes, mechanisms, and why each class exists. |
| **[results/](results/)** | Per-model results, each with the exact conditions that produced it. |

## Levels

Three questions, three tests. Each answers something the level below it cannot see.

| Level | Question | State |
|---|---|---|
| **L1 — competency** | Does it code, call tools, and engage with security work at all? | Running |
| **L2 — recognition** | Shown code, does it find a real vulnerability, classify it, and stay quiet on clean code? | Running |
| **L3 — operation** | Can it find and exploit a real bug in a real codebase, unprompted, iterating against an executable oracle? | Does not exist |

Nobody should claim L3 capability from an L2 result. That includes us.

## Results

- [DeepSeek-V4-Flash-0731 (abliterated)](results/deepseek-v4-flash-0731-abliterated/RESULTS.md) — 2× DGX Spark, TP=2

## What is not here

**The evaluation code.** The method is documented in full and the instrument follows once it is
cleaned up for release. Until then these results are not independently reproducible, which is stated
plainly on every result page.

**The answer keys.** Sample mechanisms and design rationale are published. Ground truth is not, and
the post-cutoff class rotates, because a novel sample stops being novel the moment its answer is
public.

**Throughput comparisons.** Observed throughput appears inside each result's conditions block because
the harness settings derive from it. It is context for the result, not a hardware benchmark.

## Related

[**fenceline**](https://github.com/nealbridges/fenceline) — a standalone refusal-posture probe you
can point at your own model today. MIT, no dependencies beyond an OpenAI-compatible endpoint.

## License

MIT. See [LICENSE](LICENSE).

---

Neal Bridges · [@cybersec](https://x.com/cybersec)
