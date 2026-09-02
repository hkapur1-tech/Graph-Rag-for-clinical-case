# Hybrid Vector Search and Knowledge Graph Retrieval for Clinical Question Answering

Code and results for the paper. Evaluates four retrieval architectures on a textual
clinical QA benchmark and a medical visual QA benchmark.

## Results

| System | MedQA (n=1,273) | PathVQA (n=1,010) |
|---|---|---|
| Baseline (no retrieval) | 96.0% | 68.0% |
| Flat-Vector RAG | 96.5% (p=0.34) | 70.4% (p=0.21) |
| Static GraphRAG | 95.8% (p=0.72) | 69.9% (p=0.26) |
| **Hybrid GraphRAG** | **96.2% (p=0.72)** | **79.6% (p<0.001)** |

p-values from McNemar's test against the baseline within each benchmark.

On MedQA no architecture differs significantly from the baseline: the benchmark is
saturated for a frontier model. On PathVQA only the hybrid system clears significance —
neither flat-vector RAG nor static GraphRAG improves on the baseline alone, so the
benefit comes from combining keyword, dense-vector and graph retrieval rather than
from any single strategy.

## Verifying the reported numbers

The per-question outputs of every reported run are in `results/`. Running **section 8**
of the notebook against these files reproduces Tables I and II of the paper — every
accuracy, confidence interval and p-value — in a few seconds, with no API calls and
no cost.

| File | Reported as |
|---|---|
| `baseline_medqa_matched.csv` | MedQA baseline, 96.0% |
| `rag_opus5_results.csv` | MedQA Flat-Vector RAG, 96.5% |
| `graphrag_opus5_results.csv` | MedQA Static GraphRAG, 95.8% |
| `hybrid_graphrag_results.csv` | MedQA Hybrid GraphRAG, 96.2% |
| `pathvqa_results.csv` | PathVQA baseline / Static GraphRAG / Hybrid GraphRAG (one file, three columns) |
| `pathvqa_rag_results.csv` | PathVQA Flat-Vector RAG, 70.4% |
| `baseline_medqa_nothinking.csv` | Control experiment with mismatched decoding settings, 94.7% — **not used in the main results** |

Each row is one test question. `is_correct` is the boolean used by McNemar's test.

## Reproducibility

Sampling parameters are left at their API defaults, so re-running the experiments
produces statistically equivalent but not bit-identical output. The files in
`results/` are the record of the runs reported in the paper.

Section 8 of the notebook reproduces every number in Tables I and II from those files
exactly, with no API calls. The knowledge graphs in `graphs/` are committed for the
same reason: they are built by LLM extraction and would not rebuild identically.

## Running the experiments

Open `graphrag_clinical_qa.ipynb` in Google Colab and run top to bottom.

Add your Anthropic API key as a Colab Secret named `ANTHROPIC_API_KEY` (key icon in
the left sidebar). No key is stored in this repository.

Intermediate results checkpoint to Google Drive every 10 questions, so an interrupted
run resumes rather than restarting. A full reproduction is roughly 10,000 API calls.

### Matched decoding settings

All architectures within a benchmark use identical API settings — `max_tokens=2000`
for MedQA and `max_tokens=10` for PathVQA, with the model's default reasoning
configuration.

This matters. Evaluating the MedQA baseline with a reduced output limit and reasoning
disabled, while the retrieval architectures kept the default configuration, lowered the
baseline from 96.0% to 94.7% and made all three retrieval systems appear significantly
better. The effect was an artifact of decoding configuration, not retrieval. The
appendix cell of the notebook reproduces this control.

## Datasets

| Dataset | Source | Used |
|---|---|---|
| MedQA-USMLE | `GBaker/MedQA-USMLE-4-options` | 10,178 train / 1,273 test |
| PathVQA | `flaviagiammarino/path-vqa` | 19,654 train / 1,010 yes-no test |
| PubMedQA | `qiaojin/PubMedQA` (pqa_labeled) | 500 abstracts for graph construction |
| PrimeKG | Harvard Dataverse | filtered to 12,752 entities / 441,429 relations |

## Knowledge graphs

| Graph | Entities | Relations | Built from |
|---|---|---|---|
| MedQA | 1,330 | 1,190 | 500 training documents, LLM extraction |
| PubMedQA | 1,968 | 1,549 | 500 abstracts, LLM extraction |
| PrimeKG | 12,752 | 441,429 | Harvard Dataverse, clinically filtered |
| PathVQA | 6,614 | 19,613 | 19,654 training examples, rule-based |

The two LLM-extracted graphs are included in `graphs/` because rebuilding them costs
API calls. PrimeKG and the PathVQA graph are rebuilt by the notebook at no cost.

## Models

- `claude-opus-5` — textual QA and knowledge graph extraction
- `claude-haiku-4-5-20251001` — visual QA
- `all-MiniLM-L6-v2` — sentence embeddings for dense retrieval and graph node matching

## Repository layout

```
graphrag_clinical_qa.ipynb        experiments, analysis, figures
README.md
results/                          per-question outputs of every reported run
graphs/                           pre-built knowledge graphs (LLM-extracted only)
figures/                          figures as they appear in the paper
```
