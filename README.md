# RISER: Responsible Identification of Stakeholders with Ethical Risks

An end-to-end pipeline for automated stakeholder identification, evaluation, clustering, and visualization in the context of AI systems. Designed to support ethics requirements engineering by surfacing who is affected by an AI system, how salient they are, and how they relate to one another.

---

## Overview

Given a natural-language description of an AI system, the pipeline:

1. **Generates** a deduplicated set of stakeholders and vulnerable groups by querying multiple LLMs in parallel
2. **Evaluates** each stakeholder's salience using Mitchell's Stakeholder Salience Model adapted for AI Ethics (Legitimacy, Power, Urgency)
3. **Clusters** stakeholders by semantic and salience similarity then cluster representatives against an 11-category stakeholder taxonomy
4. **Visualizes** the results as an interactive cluster explorer with bubble chart, split, and grid views

---


## Requirements

Tested on Google Colab (Python 3.10). Install dependencies in the first cell:

```bash
pip install sentence-transformers transformers torch scikit-learn tqdm pandas numpy scipy
```

### API Keys

The following API keys are required and configured in **Section 2** of the notebook:

| Provider | Used for |
|----------|----------|
| DeepInfra | Stakeholder generation (most models) + Evaluation + Taxonomy labelling |
| OpenAI | Stakeholder generation (GPT models) |
| Anthropic | Stakeholder generation (Claude models) |
| Google | Stakeholder generation (Gemini models) |

---

## Usage

1. Open `Stakeholder_Pipeline.ipynb` in Google Colab
2. Set `PROJECT_NAME` and `PROJECT_DESCRIPTION` in **Section 2**
3. Add your API keys in **Section 2**
4. Run sections sequentially, each section saves its output CSV for the next
5. To resume from a specific stage, upload the relevant CSV to the Colab session and run from that section

> **Resuming from Stage 3:** Upload `live_api_results_detailed.csv` and `frequency_table_with_rejected.csv` before running the clustering section.

---

## Configuration Reference

| Parameter | Default | Description |
|-----------|---------|-------------|
| `MAX_REPEATS_PER_MODEL` | 3 | Max API calls per model/temperature combination |
| `COST_CAP` | $5.00 | Generation budget in USD |
| `ITEM_CAP` | 200 | Max accepted stakeholders |
| `SIM_THRESHOLD` | 0.8 | Cosine similarity threshold for deduplication |
| `SCORE_WEIGHT` | 0.3 | Weight of salience scores vs. embeddings in feature matrix |
| `MIN_CHILD_SIZE` | 10 | RHRC ST1: minimum child cluster size |
| `MIN_PARENT_SIZE` | 20 | RHRC ST2: minimum parent cluster size before splitting |
| `PATIENCE` | 2 | RHRC: levels to continue after ST3 fires |

---

## Output Columns Reference

### `stakeholder_salience_evaluated.csv`

| Column | Description |
|--------|-------------|
| `Item` | Canonical stakeholder role |
| `legitimacy_before/after` | Legitimacy score (0–5) before/after calibration |
| `power_before/after` | Power score (0–5) before/after calibration |
| `urgency_before/after` | Urgency score (0–5) before/after calibration |
| `redundancy_after` | Redundancy score (0–5) relative to other stakeholders |
| `classification_after` | `Valid`, `Hallucination`, or `redundant` |

### `stakeholder_clustered.csv`

Includes all columns from evaluation plus:

| Column | Description |
|--------|-------------|
| `Cluster` | Cluster ID assigned by RHRC |
| `Is_Representative` | `True` if selected as cluster representative |
| `Frequency` | Total mentions from the frequency table |
| `K_Used` | Final number of clusters for this project |
| `Cluster_Compactness` | Mean intra-cluster cosine distance |

### `stakeholder_labelled.csv`

Includes all columns from clustering plus:

| Column | Description |
|--------|-------------|
| `taxonomy_labels` | JSON list of taxonomy category IDs e.g. `[1, 5]` |
| `taxonomy_label_names` | JSON list of category names e.g. `["System Users", "Vulnerable Stakeholders"]` |
| `taxonomy_reasoning` | One-sentence justification from the model |

---