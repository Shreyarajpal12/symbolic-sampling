# Evidence Before Sampling: Interpretable Implicit Negative Candidate Discovery for Recommendation

[![arXiv](https://img.shields.io/badge/arXiv-2610.07708-b31b1b.svg)](https://arxiv.org/abs/2610.07708)

Official code for the paper **Evidence Before Sampling: Interpretable Implicit
Negative Candidate Discovery for Recommendation**
([arXiv:2610.07708](https://arxiv.org/abs/2610.07708)).

## Overview

A missing user-item interaction does not mean the user is not interested.
This method finds symbolic rules over user features that show when an
unobserved interaction is likely a true negative. Each rule is checked with
four criteria: Scope, Evidence Strength, Hierarchical Contribution and Product
Relevance. The selected candidates can be used directly as negatives or
combined with existing samplers such as Random, BNS and DNS.

This repository contains the public-dataset experiments (KuaiRec, KuaiRand,
MIND, RetailRocket and Santander). The industrial dataset is proprietary and
not included.

## Layout

| Folder | Contents |
|---|---|
| `symbolic/` | rule discovery and the four criteria |
| `prep/` | data preparation for the five datasets |
| `conditions/` | sparse and missing-positive versions of KuaiRec |
| `models/` | LightGCN, DeepFM, DNS |
| `experiments/` | samplers, recommender training and LLM review |
| `analysis/` | scripts for the result tables |

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
python -m conditions.sparse_positives     # KuaiRec with 5%, 2% and 1% positives
python -m conditions.missing_positives    # KuaiRec with 75% of positives hidden
```

## Running

Set the dataset (`kuairec`, `mind` or `retailrocket`). For KuaiRec, you can
also set a condition (`pos05`, `pos02`, `pos01` or `pos05_mnar75`).

```bash
export NEGCAND_DATASET=kuairec NEGCAND_CONDITION=pos05_mnar75
python -m experiments.negative_rules
python -m experiments.positive_rules
for SD in 42 1 2; do NEGCAND_SEED=$SD python -m experiments.train_recommender; done
```

Then build the result tables:

```bash
python -m analysis.summarize_natural
python -m analysis.summarize_sparse_mnar
python -m analysis.negative_precision
python -m analysis.negative_precision_report
```

Candidate-quality experiments (appendix):

```bash
NEGCAND_DATASET=kuairec  python -m experiments.negative_rules
NEGCAND_DATASET=kuairand python -m experiments.negative_rules
python -m experiments.santander_candidate_quality
python -m experiments.santander_baselines
```

The LLM domain review (`experiments.llm_domain_review`) is optional and
needs an `OPENAI_API_KEY` and a category-name file (`NEGCAND_CATEGORY_NAMES`).
Results are written to `results/`.

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
