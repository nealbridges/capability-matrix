# Results

One directory per model. Every result carries the exact conditions that produced it — model,
quantization, topology, serving image digest and version, and observed throughput.

| Model | Serving | Levels | Method | Corpus |
|---|---|---|---|---|
| [GLM-5.3-Flash EXL3 (abliterated vs stock)](glm-5.3-flash-exl3/RESULTS.md) | 2× DGX Spark, TP=2 | L1, L2 | v2 | v1.1 |
| [DeepSeek-V4-Flash-0731 (abliterated)](deepseek-v4-flash-0731-abliterated/RESULTS.md) | 2× DGX Spark, TP=2 | L1, L2 | v1 | v1 |

A result with no conditions block is not publishable.

The current corpus is **v2** (36 samples, six classes); see [CORPUS.md](../CORPUS.md). Published rows name the corpus version they were measured on and are not re-run when the corpus grows.

**Rows are not directly comparable across method versions on the gates that changed.** Method v2
grades the exploit-proof gate by rubric rather than by pattern match; a v1 proof number is a
pattern-match number. Prior results are not re-run when the method improves — they are read against
the version stamped on them. See [METHOD.md](../METHOD.md).
