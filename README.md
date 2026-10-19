# Evidence Before Sampling: Interpretable Implicit Negative Candidate Discovery for Recommendation

[![arXiv](https://img.shields.io/badge/arXiv-2610.07708-b31b1b.svg)](https://arxiv.org/abs/2610.07708)

Official code for the paper **Evidence Before Sampling: Interpretable Implicit
Negative Candidate Discovery for Recommendation**
([arXiv:2610.07708](https://arxiv.org/abs/2610.07708)).

## Overview

An unobserved user-item interaction is not necessarily a negative. We first
test whether an unobserved interaction is a defensible negative, using
symbolic rules over user features and four separate criteria: Scope, Evidence
Strength, Hierarchical Contribution and Product Relevance. Validated
candidates fill a tiered negative budget, and samplers such as Random, BNS
and DNS can operate inside the admitted set. Positive Symbolic applies the
same evidence in reverse to protect likely positives.

This repository reproduces the public-dataset experiments in the paper on
KuaiRec, KuaiRand, MIND, RetailRocket and Santander. The industrial dataset
used in the paper is proprietary and is not included.

## Layout

| Folder | Contents |
|---|---|
| `symbolic/` | rule evaluation, the four criteria, rule discovery, negative ladder |
| `prep/` | data preparation for the five datasets |
| `conditions/` | KuaiRec sparse and MNAR training conditions |
| `models/` | LightGCN, DeepFM, DNS |
| `experiments/` | rule discovery, samplers, recommender training, LLM review, Santander |
| `analysis/` | result tables |

## Setup

```bash
pip install -r requirements.txt
```

Place the raw data under `data/`:

| Dataset | Source | Location |
|---|---|---|
| KuaiRec 2.0 | kuairec.com | `data/kuairec/` |
| KuaiRand-Pure | kuairand.com | `data/KuaiRand-Pure/data/` |
| MIND-small | msnews.github.io | `data/mind/MINDsmall_train/` |
| RetailRocket | Kaggle `retailrocket/ecommerce-dataset` | `data/retailrocket/` |
| Santander | Kaggle `santander-product-recommendation` | downloaded by `prep/download_santander.sh` |

```bash
python -m prep.kuairec_kuairand
python -m prep.mind
python -m prep.retailrocket
bash prep/download_santander.sh
python -m conditions.sparse_positives     # KuaiRec 5%, 2%, 1% positives
python -m conditions.missing_positives    # KuaiRec 5% + MNAR-75
```

## Running

Each run uses a dataset (`kuairec`, `mind`, `retailrocket`) and, for KuaiRec,
an optional condition (`pos05`, `pos02`, `pos01`, `pos05_mnar75`).

```bash
export NEGCAND_DATASET=kuairec NEGCAND_CONDITION=pos05_mnar75
python -m experiments.negative_rules
python -m experiments.positive_rules
for SD in 42 1 2; do NEGCAND_SEED=$SD python -m experiments.train_recommender; done
```

After all runs, build the result tables:

```bash
python -m analysis.summarize_natural
python -m analysis.summarize_sparse_mnar
python -m analysis.negative_precision
python -m analysis.negative_precision_report
```

Appendix candidate quality:

```bash
NEGCAND_DATASET=kuairec  python -m experiments.negative_rules
NEGCAND_DATASET=kuairand python -m experiments.negative_rules
python -m experiments.santander_candidate_quality
python -m experiments.santander_baselines
```

The optional LLM domain review (`experiments.llm_domain_review`) needs
`OPENAI_API_KEY` and a category-name file set in `NEGCAND_CATEGORY_NAMES`.

Samplers: `random`, `bns`, `bns_smooth`, `dns`, `negative_symbolic`,
`positive_symbolic`, `symbolic_random`, `symbolic_bns`,
`symbolic_bns_smooth`, `symbolic_dns`. Outputs go to `artifacts/` and
`results/`.

## Citation

If you use this code, please cite our paper:

```bibtex
@misc{rajpal2026evidencesamplinginterpretableimplicit,
      title={Evidence Before Sampling: Interpretable Implicit Negative Candidate Discovery for Recommendation},
      author={Shreya Rajpal and Sonia Sharma and Swapnil Parekh and Lisa Li and Jeyendran Balakrishnan and Nagaraj Janardhana and Andrew Mattarella-Micke},
      year={2026},
      eprint={2610.07708},
      archivePrefix={arXiv},
      primaryClass={cs.AI},
      url={https://arxiv.org/abs/2610.07708},
}
```
