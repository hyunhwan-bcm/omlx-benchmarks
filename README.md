# oMLX benchmarks — Qwen3.8-27B-4bit on Apple Silicon

Benchmark results for a 27B model quantized to 4-bit, served locally by
[oMLX](https://github.com/jundot/omlx) on an Apple M2 Max (30 GPU cores, 32 GB)
at roughly 19 tokens/second.

**Site:** https://hyunhwan-bcm.github.io/omlx-benchmarks/

## Results

| benchmark | result |
|---|---|
| MMLU | 87.6% (219/250) |
| ARC-Challenge | 97.6% (244/250) |
| TruthfulQA MC1 | 91.5% (183/200) |
| GSM8K | 96.0% (144/150) |
| HumanEval | 95.7% (157/164) — see note |
| FLORES-200 en→ko | COMET 0.904, chrF++ 34.03 |

Every number was verified by re-running the underlying code or scoring, not by
trusting the harness output.

### Two findings worth reading

**HumanEval: 12.2% → 95.7%.** oMLX's code extractor handles only a fenced code
block or a response starting with `def`. With thinking enabled this model opens
with reasoning prose, so the harness fell through to grading *prompt + entire
reasoning trace* as Python. That does not compile, so nearly everything failed.
Re-executing the model's real code against the official tests: 135 of 144
"failures" were correct.

**Translation metrics disagree sharply.** chrF++ says 34.03; COMET says 0.904.
The lowest chrF++ sentence is one where the model's translation is *more
faithful than the FLORES reference*. Character-overlap metrics punish valid
paraphrase, which Korean produces constantly.

## Files

- `index.html` — results, methodology, sample translations
- `csv-tool.html` — a browser CSV/XLSX editor **written by the model**, including
  a from-scratch ZIP reader and writer with CRC32. Runs offline, no libraries.
- `data.json` — the underlying numbers

## The CSV tool

Asked for a browser CSV editor with XLSX export, the model produced a working
one in 6 agent turns. A real `.xlsx` is a ZIP of XML parts, so this required
implementing ZIP local headers, the central directory, and CRC32 by hand. The
output opens correctly in openpyxl. It later wrote the reader too, handling
DEFLATE and shared strings.

Verified by running the code: CSV round-trips through quoted commas, escaped
quotes and embedded newlines; the CRC32 matches the known vector
`crc32("123456789") == 0xCBF43926`; generated archives pass `unzip -t`.
