# Domain Name Generator — Dataset, Notebook, Results

Generates realistic, safe domain names from short business descriptions, trains LoRA-tuned LLMs (GPT-2, Zephyr, Llama), and evaluates with **TinyLlama** as an LLM-as-a-Judge. Success is measured by a strict pass/fail **Success Rate**.

---

## Repo layout

- `generated datasets/` — auto-generated CSV/JSONL data from the notebook (train/val/test).
- `notebook folder/` — Jupyter notebook with dataset generation, training, and evaluation.
- `tech_report.*` — the full technical report (LaTeX/PDF) summarizing method and results.
- `LICENSE` — Apache-2.0 (repo-wide).

> Tip: keep the notebook as the single entry point. It generates data into `generated datasets/` and runs training/eval end-to-end.

---

## Quick start

1. Open the notebook in `notebook folder/`.
2. Run **Dataset** cells → writes files to `generated datasets/` (seeded, reproducible).
3. Run **Training** cells → LoRA fine-tunes GPT-2, Zephyr, and Llama.
4. Run **Evaluation** cells → computes Success Rate with TinyLlama and prints an edge-case summary.

_Environment (typical):_ Python ≥3.10, PyTorch, `transformers`, `peft`, (optional) `bitsandbytes` for QLoRA.

---

## Strategy (how it works)

**Dataset (from code)**
- Rule-based combinatorics build short **business descriptions** (sector, modifiers, locale).
- Optional toggle to **use an LLM** to paraphrase/expand descriptions (then filtered the same way).
- Candidate domains are formed by blending description tokens with prefixes/suffixes and curated **TLDs**.
- **Safety filters** (banned words) + **format checks** (DNS rules) run during generation.
- Edge examples are purposely included (TLD traps, formatting pitfalls, semantic contradictions).
- Safe items go to training; unsafe triggers are used to **teach refusals**.

**Models**
- **GPT-2 (LoRA)** — compact baseline, benefits most from decoding constraints and validators.
- **Zephyr (LoRA)** — strong instruction following; good quality/compute trade-off.
- **Llama (LoRA)** — best overall capability and fluency.

**LoRA improvements**
- Targeted modules (attn proj; optionally MLP proj for GPT-2).
- Small grid over **rank r** and **alpha**, + **adapter dropout** for generalization.
- Optional **QLoRA** for memory efficiency; mixed precision + gradient clipping.
- Early stopping on **judge-measured** Success Rate (not just loss).

**Evaluation — LLM-as-a-Judge**
- **TinyLlama** scores each suggestion for: relevance, TLD fit, format validity, brandability, uniqueness-in-K, and safety.
- **Success Rate** per prompt:
  - Safe brief → **all K** suggestions pass.
  - Unsafe brief → model returns a **refusal** (no domains).
- Report = successful prompts / total prompts × 100%.

---

## Headline results

| Model (LoRA) | Before | After | Δ (pp) |
|---|---:|---:|---:|
| GPT-2 | 82% | **92%** | **+10** |
| Zephyr | 90% | **92%** | +2 |
| Llama | 96% | **98%** | +2 |

**Interpretation:** Biggest absolute gain for GPT-2 via stricter decoding + better filters + LoRA targeting. Zephyr/Llama already strong; safety curriculum and diversity nudges add consistent wins.

---

## Edge-case frequency (test set)

| Category        | % of prompts | Top root cause                 | Fix applied                                  |
|----------------|:------------:|--------------------------------|----------------------------------------------|
| TLD mismatch   | 7.8%         | weak sector↔TLD prior          | stricter TLD maps + judge penalty            |
| Format errors  | 4.1%         | len/hyphen rules not enforced  | tighter validator + decode tweaks            |
| Over-generic   | 6.3%         | bland stems                    | prefix/suffix tuning + brandability nudges   |
| Duplicates-in-K| 3.2%         | high token overlap             | dedupe pass + duplicate penalty              |
| Safety leaks   | 0.6%         | borderline phrases             | banned-list expansion + refusal demos        |

---

## Writing good descriptions (for dataset/prompts)

Keep it short (1–2 lines) and specific:
- **Sector/Niche**, **locale/market**, **audience**, key **product/service** cues, optional **tone**.

Examples:
- “Family-owned **coffee shop** in **Lyon** serving **single-origin espresso and pastries**; **warm, local** brand feel.”
- “**AI analytics consultancy** in **Paris** for **SMBs**; **modern, trustworthy** vibe.”

Avoid unsafe topics and irrelevant TLD hints (e.g., “use .ai” when not AI-related).

---

## License

This project is licensed under **Apache-2.0**. See `LICENSE`. Use of any third-party base models remains subject to their own licenses.

---

## Cite / Reference

See `tech_report.*` for the full methodology, LoRA details, the judge rubric, and expanded experiments.
