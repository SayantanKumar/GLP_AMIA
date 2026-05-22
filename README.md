# Temporally Phenotyping GLP-1RA Case Reports with Large Language Models

<div align="center">

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://python.org)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org)
[![scispaCy](https://img.shields.io/badge/scispaCy-UMLS-green.svg)](https://allenai.github.io/scispacy/)
[![lifelines](https://img.shields.io/badge/lifelines-survival%20analysis-8A2BE2.svg)](https://lifelines.readthedocs.io/)

</div>

## Overview

This repository contains analysis notebooks for the study **"Temporally Phenotyping GLP-1RA Case Reports with Large Language Models: A Textual Time Series Corpus and Risk Modeling"**.

The study converts PubMed Open Access single-patient case reports involving glucagon-like peptide-1 receptor agonists (GLP-1RAs) into clinical textual time series: event-time pairs where each clinical event is assigned a timestamp in hours relative to a case-specific reference point. The paper uses these timelines to evaluate LLM-based temporal extraction against expert annotations, characterize diagnosis patterns in the GLP-1RA PMOA cohort, and demonstrate downstream time-to-onset modeling for kidney, cardiovascular, and respiratory outcomes.

The notebooks in this release are primarily for cohort characterization, UMLS diagnosis normalization, threshold-sweep evaluation, and survival analysis after timeline extraction has already been run. The repository now includes `annotations.zip`, which expands to the released manual annotations, LLM-generated textual time series, demographic features, and diagnosis features used by the analyses. Raw PMOA full text and the broader PMOA annotation tree are not included.

## Figures

### Study Workflow

[![GLP-1RA PMOA textual time-series workflow](paper_plots/glp_workflow.png)](paper_plots/fig1_worflow_with_example.pdf)

### Example Textual Time Series

[![Example GLP-1RA textual time series](paper_plots/glp_tts_example.png)](paper_plots/fig1_worflow_with_example.pdf)

## Key Contributions

- **GLP-1RA Textual Time-Series Corpus**: Builds a medication-centered corpus from PubMed Open Access case reports by representing patient narratives as timestamped clinical events.
- **LLM Timeline Evaluation**: Compares LLM-extracted timelines against two clinically trained expert annotators using event match rate, temporal order concordance, and AULTC.
- **Diagnosis Characterization**: Uses scispaCy UMLS linking to normalize diagnosis strings and summarize cardiometabolic, kidney, respiratory, digestive, mental health, cancer, anemia, oral/dental, bone/joint, and sepsis categories.
- **Threshold Sensitivity Analysis**: Sweeps event-matching thresholds to study the tradeoff between semantic event coverage and temporal fidelity.
- **Time-to-Onset Modeling**: Constructs GLP-1RA treatment and comparison cohorts from textual timelines and fits Kaplan-Meier and Cox proportional hazards models for selected outcomes.

## Repository Structure

```text
code_github/
|-- annotations.zip                               # Released annotations archive; expands to annotations/
|-- annotations/                                  # Manual, LLM, diagnosis, and demographic annotations
|-- create_medspacy_diagnosis_cui_data.ipynb      # Extract dx2 annotations and link diagnoses to UMLS CUIs
|-- diagnosis_GLP_PMOA.ipynb                      # GLP/diabetes cohort selection and diagnosis analyses
|-- diagnosis_char_med_spacy.ipynb                # Exploratory full-PMOA UMLS diagnosis summaries
|-- diagnosis-characterization-prevalence.ipynb   # Disease-category prevalence plots vs US baselines
|-- threshold_sweep.ipynb                         # Event match, concordance, and AULTC threshold sweeps
|-- survival_analyses_glp.ipynb                   # Treatment/control construction and survival modeling
|-- paper_plots/                                  # Figures used in the paper and README
`-- README.md                                     # This file
```

## Workflow

The analysis proceeds in seven stages:

1. **Annotation archive setup**: `annotations.zip` expands to the manual timelines, LLM timelines, demographic table, diagnosis table, and precomputed match files.
2. **Diagnosis annotation ingestion**: `create_medspacy_diagnosis_cui_data.ipynb` can recreate UMLS-linked diagnosis pickles from a local sharded PMOA annotation directory when those upstream files are available.
3. **GLP-1RA and diabetes cohort setup**: `diagnosis_GLP_PMOA.ipynb` loads PMOA diagnosis annotations or the released diagnosis feature table, restricts to GLP-1RA case reports, identifies diabetes-related reports, samples comparison cases, and creates linked diagnosis pickles.
4. **Diagnosis normalization and prevalence**: `diagnosis_char_med_spacy.ipynb` and `diagnosis-characterization-prevalence.ipynb` aggregate UMLS canonical diagnoses and disease-category prevalence.
5. **Timeline quality evaluation**: `threshold_sweep.ipynb` consumes precomputed `annotations/**/matches/best_matches*.csv` files and computes event match rate, concordance, and AULTC across cosine-distance thresholds.
6. **Outcome extraction from textual time series**: `survival_analyses_glp.ipynb` searches LLM-extracted event-time CSVs, especially the GPT5 treatment/control folders, for cardiovascular, respiratory, and kidney outcome keywords with lightweight negation handling.
7. **Survival modeling**: `survival_analyses_glp.ipynb` fills censoring times from each timeline's last observed timestamp and fits Kaplan-Meier and age/sex-adjusted Cox models.

## Quick Start

### 1. Clone And Install

```bash
git clone <repo-url>
cd code_github

conda create -n glp-tts python=3.10
conda activate glp-tts

pip install jupyter pandas numpy matplotlib seaborn tqdm scikit-learn scipy statsmodels lifelines spacy scispacy
```

The UMLS normalization notebooks use the large scispaCy model:

```bash
pip install https://s3-us-west-2.amazonaws.com/ai2-s2-scispacy/releases/v0.5.4/en_core_sci_lg-0.5.4.tar.gz
```

The notebooks call:

```python
nlp = spacy.load("en_core_sci_lg")
nlp.add_pipe("scispacy_linker", config={"resolve_abbreviations": True, "linker_name": "umls"})
```

The first UMLS linker run may download additional knowledge-base files.

### 2. Unpack Annotation Archive

If the `annotations/` folder is not already present, unpack the archive from the repository root:

```bash
unzip annotations.zip
```

The archive expands to `annotations/`. If both the zip file and folder are present, the notebooks can use the folder directly.

### 3. Prepare Local Artifacts

The repository includes the GLP-1RA annotation release in `annotations.zip`, including model timelines, A1/A2 manual timelines, demographic features, diagnosis features, and precomputed match files.

To fully rerun every notebook from the original raw pipeline, you may still need local research artifacts that are not committed to this repository:

- PMOA diagnosis annotation folders with paths like `PMC000xxxxxx/anns/dx2/PMC*_body.txt.bsv.gz`
- GLP-1RA case report text files named like `PMC*_body.txt`
- Intermediate diagnosis pickle files such as `full_linked_diagnosis_data.pkl`, `glp_linked_diagnosis_data.pkl`, `diagnosis_case_diabetes_glp.pkl`, and `diagnosis_control_diabetes_nonglp.pkl`

Several of those intermediate pickles can be regenerated from the released CSV annotations plus the scispaCy/UMLS pipeline, but the full PMOA source corpus is not bundled here.

### 4. Update Paths

The notebooks currently contain absolute paths from the study environment, for example:

```text
/Users/kumars33/Desktop/Diabetes_LLM_annotation/
/Users/kumars33/Desktop/CHARM_MIMIC/
/Users/kumars33/Desktop/TTA/Textual_tabular_alignment-main/
```

Before running, edit the path cells near the top of each notebook to point to your local copies of the PMOA annotations, case reports, timeline CSVs, and match files. For many post-extraction analyses, these paths can be changed to the corresponding files under `annotations/`.

### 5. Run Notebooks

Open Jupyter and run only the notebooks needed for your analysis:

```bash
jupyter lab
```

Suggested order:

1. `create_medspacy_diagnosis_cui_data.ipynb`
2. `diagnosis_GLP_PMOA.ipynb`
3. `diagnosis-characterization-prevalence.ipynb`
4. `threshold_sweep.ipynb`
5. `survival_analyses_glp.ipynb`

`diagnosis_char_med_spacy.ipynb` is an exploratory companion for full-PMOA diagnosis summaries.

## Annotations

The annotation archive has this top-level layout:

```text
annotations/
|-- demo_glp_n140.csv                 # Demographics extracted for GLP-1RA cases
|-- dx_glp_n140.csv                   # Diagnosis feature annotations
|-- manual_annotations/
|   |-- best_performer_A1/            # Expert annotator A1 timelines
|   `-- 2nd_best_performer_A2/        # Expert annotator A2 timelines and A2-vs-A1 matches
`-- LLM_annotations/
    |-- l33_70b_tts/                  # Llama-3.3-70B-Instruct timelines
    |-- DS_l33_70b_tts/               # DeepSeek-R1-Distill-Llama-70B timelines
    |-- DSR1_0528_n140/               # DeepSeek R1 timelines
    |-- gpt5_tts/                     # GPT5 timelines plus treatment/control subsets
    |-- o1_tts/                       # O1 timelines
    |-- o3_tts/                       # O3 timelines
    `-- o4mini_tts/                   # O4mini timelines
```

Model-folder mapping:

| Folder | Model name in paper | Notes |
| --- | --- | --- |
| `l33_70b_tts/` | `Llama3.3-70B-Instruct` | Includes cleaned CSV/BSV timelines and match files. |
| `DS_l33_70b_tts/` | `DeepSeek-R1-Distill-Llama-70B` | Includes cleaned CSV variants and match files. |
| `DSR1_0528_n140/` | `DeepSeek R1` | Includes cleaned CSV/BSV timelines and match files. |
| `gpt5_tts/` | `GPT5` | Includes cleaned CSV/BSV timelines, match files, and the 82-case treatment/control subsets used in survival analysis. |
| `o1_tts/` | `O1` | Includes cleaned CSV/BSV timelines and match files. |
| `o3_tts/` | `O3` | Includes cleaned CSV/BSV timelines and match files. |
| `o4mini_tts/` | `O4mini` | Includes cleaned CSV/BSV timelines and match files. |

Manual annotations:

| Folder | Meaning | Format |
| --- | --- | --- |
| `manual_annotations/best_performer_A1/` | Expert annotator A1 timelines | CSV files with `event,time` columns. |
| `manual_annotations/2nd_best_performer_A2/` | Expert annotator A2 timelines | CSV files with `event,time` columns, plus match files comparing A2 to A1. |

Feature annotations:

| File | Description |
| --- | --- |
| `annotations/demo_glp_n140.csv` | Age, sex, ethnicity, source case path, and PMCID. |
| `annotations/dx_glp_n140.csv` | One row per case with `diagnosis_1`, `diagnosis_2`, ... fields. |

Timeline files are provided in two common formats:

```csv
event,time
35 years old,0
male,0
presented for medical weight-loss management,0
weight 118 kg,0
```

```text
35 years old | 0
male | 0
presented for medical weight-loss management | 0
weight 118 kg | 0
```

The archive also contains precomputed `matches/best_matches*.csv` files used by `threshold_sweep.ipynb`. These files store predicted/reference event alignments and similarity errors used to compute event match rate, concordance, and AULTC.

## Main Outputs

| Output | Produced By | Description |
| --- | --- | --- |
| `full_linked_diagnosis_data.pkl` | `create_medspacy_diagnosis_cui_data.ipynb` | UMLS-linked diagnosis dictionary for the broader PMOA case-report pool. |
| `glp_linked_diagnosis_data.pkl` | `diagnosis_GLP_PMOA.ipynb` | UMLS-linked diagnosis dictionary for GLP-1RA case reports. |
| `diagnosis_case_diabetes_glp.pkl` | `diagnosis_GLP_PMOA.ipynb` | Linked diagnoses for diabetes reports with GLP-1RA exposure. |
| `diagnosis_control_diabetes_nonglp.pkl` | `diagnosis_GLP_PMOA.ipynb` | Linked diagnoses for sampled diabetes comparison reports. |
| `paper_plots/top20_diagnoses_glp.pdf` | `diagnosis_GLP_PMOA.ipynb` | Top UMLS-normalized diagnoses in the GLP-1RA cohort. |
| `paper_plots/disease_prevalence.pdf` | `diagnosis-characterization-prevalence.ipynb` | Broad disease-category prevalence vs US adult baselines. |
| `paper_plots/concordance_vs_EMR_*.pdf` | `threshold_sweep.ipynb` | Concordance vs event match rate across matching thresholds. |
| `paper_plots/aultc_vs_EMR_*.pdf` | `threshold_sweep.ipynb` | AULTC vs event match rate across matching thresholds. |
| `paper_plots/survival_all_plots.pdf` | `survival_analyses_glp.ipynb` | Survival-analysis summary plots. |
| `paper_plots/*_cox_survival.pdf` | `survival_analyses_glp.ipynb` | Outcome-specific adjusted survival plots. |

## Data

This repository includes the released GLP-1RA annotation archive (`annotations.zip`) with expert timelines, LLM timelines, diagnosis features, demographic features, and precomputed matching outputs. It does not include the full PubMed Open Access source corpus or the broader intermediate PMOA annotation tree used to construct the initial candidate pool.

Expected timeline CSV format:

```csv
event,time
55-year-old man,0
started semaglutide,-720
admitted to hospital,0
acute kidney injury,72
discharged home,168
```

Times are stored in hours relative to the case reference point. Negative values indicate events before the reference time.

## Models

The paper evaluates multiple LLMs for textual time-series extraction, including DeepSeek R1, DeepSeek-R1-Distill-Llama-70B, Llama-3.3-70B-Instruct, GPT5, O1, O3, and O4mini. The notebooks in this archive do not serve or call those LLMs directly; instead, they analyze generated timeline outputs and match files included under `annotations/LLM_annotations/`.

For diagnosis normalization, the notebooks use:

- `en_core_sci_lg`
- `scispacy_linker`
- UMLS candidate filtering by score threshold, with top-ranked CUI/canonical-name summaries

For survival modeling, the notebooks use:

- `lifelines.KaplanMeierFitter`
- `lifelines.CoxPHFitter`
- age and sex covariate adjustment
- bootstrap uncertainty bands for adjusted survival curves

## Notes And Caveats

- Several notebooks are exploratory and contain repeated or commented analysis cells from different experiments.
- Many paths are absolute and must be changed before reuse.
- Some cells assume intermediate pickle files have already been created; match files are included in `annotations/**/matches/`.
- The `compare_tts/` folder in this archive is empty aside from system metadata; threshold-sweep logic is currently in `threshold_sweep.ipynb`.
- Outcome detection in the survival notebook is keyword-based and should be interpreted as time to first documented mention in the textual time series, not necessarily biological onset.
- Case reports are a biased clinical-informatics cohort, not a representative epidemiologic sample; hazard ratios in the paper are associative rather than causal.

## Citation

If you use this code or build on this work, please cite:

```bibtex
@inproceedings{kumar2026glp_tts,
  title = {Temporally Phenotyping GLP-1RA Case Reports with Large Language Models: A Textual Time Series Corpus and Risk Modeling},
  author = {Kumar, Sayantan and Weiss, Jeremy C.},
  booktitle = {AMIA Symposium},
  year = {2026},
  note = {Forthcoming}
}
```

