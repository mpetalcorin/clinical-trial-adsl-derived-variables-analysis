# Clinical Trial ADSL Derived Variables Analysis

End-to-end clinical trial programming workflow demonstrating synthetic source-domain generation, ADaM-style ADSL derivations, independent QC, safety analyses, statistical modelling, and publication-quality visualisation using parameters benchmarked to peer-reviewed journal-indexed literature.

## Overview

This repository contains a reproducible Jupyter notebook that simulates realistic clinical-trial source datasets and derives subject-level analysis variables commonly encountered in pharmaceutical programming workflows.

The project follows the conceptual pipeline:

```text
Synthetic source data
DM + EX + VS + AE + DS
        |
        v
ADSL derivations
        |
        v
Independent QC and traceability checks
        |
        v
Descriptive and exploratory analyses
        |
        v
TLG-style visualisations and interpretation
```

The primary goal is to demonstrate how clinical-programming rules can be translated into transparent, testable, and reproducible code while preserving traceability from source records to analysis-ready variables.

> **Important:** All patient-level data in this project are synthetic. Literature values are used only as simulation calibration anchors. The analyses and model outputs are not clinical evidence and should not be interpreted as results from a real therapeutic study.

## Main Notebook

```text
adsl_derived_variables_pubmed_benchmarked_analysis.ipynb
```

The notebook includes:

- simulation of `DM`, `EX`, `VS`, `AE`, and `DS` domains;
- derivation of subject-level ADSL variables;
- one-record-per-subject validation;
- source-to-derived QC checks;
- literature-benchmark calibration;
- age and treatment-start analyses;
- longitudinal systolic blood-pressure analysis;
- adverse-event subject incidence and event-frequency analysis;
- cardiac adverse-event analysis;
- last-known-alive-date derivation;
- exploratory logistic regression;
- disposition analysis;
- publication-quality figures;
- interpretation and implications after each major analysis;
- an audit table linking source variables, derivation rules, and QC risks.

## Derived Variables

### `AGEGR9`

Age group derived from `DM.AGE`.

```text
<18
18 - 50
>50
```

### `AGEGR9N`

Numeric representation of `AGEGR9`.

| AGEGR9 | AGEGR9N |
|---|---:|
| `<18` | 1 |
| `18 - 50` | 2 |
| `>50` | 3 |

### `TRTSDTM`

Treatment start date-time derived from the first valid exposure record for each subject.

A valid exposure is defined in the assessment as:

```text
EXDOSE > 0
OR
EXDOSE == 0 AND EXTRT contains "PLACEBO"
```

The first valid exposure record after sorting by treatment start date-time is used.

### `TRTSTMF`

Treatment start time imputation flag.

Assessment rules implemented:

```text
Missing time
    -> impute 00:00:00

Missing hour or minute
    -> impute missing component with 00

Missing seconds only
    -> impute seconds as 00
    -> do not populate the imputation flag
```

The purpose of the flag is to preserve transparency between observed and imputed information.

### `ITTFL`

Intent-to-treat population flag.

Assessment rule:

```text
ITTFL = "Y" if DM.ARM is populated
ITTFL = "N" otherwise
```

This is an assessment-specific programming rule, not a universal definition of the ITT population. In a production trial, the population definition would follow the protocol, Statistical Analysis Plan, and ADaM specification.

### `ABNSBPFL`

Abnormal supine systolic blood-pressure flag.

Assessment rule:

```text
ABNSBPFL = "Y" if any SYSBP value in mmHg satisfies:

VSSTRESN < 100
OR
VSSTRESN >= 140

ABNSBPFL = "N" otherwise
```

The thresholds are implemented because they were specified in the exercise and are not presented here as a universal clinical definition of abnormal blood pressure.

### `LSTALVDT`

Last known alive date.

Derived as the maximum qualifying date available from:

1. vital signs, `VS`;
2. adverse-event start dates, `AE`;
3. disposition dates, `DS`;
4. valid exposure dates, `EX`.

The variable represents the latest documented date on which the study data provide evidence that the participant was alive.

### `CARPOPFL`

Cardiac adverse-event population flag.

Assessment rule:

```text
CARPOPFL = "Y"
if any AE record has:
AESOC == "CARDIAC DISORDERS"

CARPOPFL = NA otherwise
```

## Synthetic Source Domains

The notebook generates five source-level datasets.

| Domain | Meaning | Example information |
|---|---|---|
| `DM` | Demographics | age, sex, treatment arm, reference start date |
| `EX` | Exposure | treatment, dose, treatment start date-time |
| `VS` | Vital Signs | systolic blood pressure and assessment dates |
| `AE` | Adverse Events | event term, system organ class, severity, dates |
| `DS` | Disposition | completion, discontinuation, and disposition dates |

These datasets are synthetic but designed to resemble the structures encountered in clinical-programming workflows.

## Independent QC and Traceability

The notebook checks that:

- ADSL contains one record per `USUBJID`;
- subject identifiers remain unique after merges;
- `AGEGR9` and `AGEGR9N` reproduce their source-based derivations;
- `TRTSDTM` corresponds to the first valid exposure;
- `TRTSTMF` reflects the programmed imputation rule;
- `ABNSBPFL` agrees with the underlying `VS` records;
- `CARPOPFL` agrees with the underlying `AE.AESOC` records;
- `LSTALVDT` can be reproduced from qualifying dates across source domains;
- population denominators remain consistent;
- subject incidence is distinguished from event frequency.

The guiding principle is:

> **Successful code execution does not by itself prove clinical correctness.**

Clinical correctness also requires specification compliance, source-to-target traceability, appropriate QC, and review of clinical meaning.

## Analysis Sections

### 1. Cohort and literature calibration

Compares simulated values with prespecified literature anchors for:

- age;
- sex;
- hypertension prevalence;
- placebo adverse-event prevalence;
- adverse-event-related discontinuation;
- higher-grade placebo adverse events;
- context-specific cardiac adverse-event incidence.

### 2. Age distribution

Evaluates:

- continuous age distributions;
- treatment-arm balance;
- `AGEGR9`;
- `AGEGR9N`.

### 3. Treatment start and imputation

Examines:

- first valid treatment exposure;
- placebo handling at `EXDOSE = 0`;
- frequency of complete versus imputed treatment start times;
- traceability of `TRTSTMF`.

### 4. Longitudinal systolic blood pressure

Evaluates:

- mean systolic BP by visit and treatment arm;
- longitudinal confidence intervals;
- assessment-defined abnormal BP thresholds;
- prevalence of `ABNSBPFL=Y` across age groups.

### 5. Adverse events

Distinguishes two fundamentally different quantities:

```text
Subject incidence
= number of unique participants with >=1 event

Event frequency
= total number of AE records
```

A subject may experience the same event more than once, so these measures should not be used interchangeably.

### 6. Cardiac adverse events

Evaluates:

- cardiac AE subject incidence;
- 95% confidence intervals;
- `CARPOPFL`;
- treatment-arm comparisons.

### 7. Last known alive date

Evaluates:

- cross-domain date reconciliation;
- follow-up duration;
- traceability of `LSTALVDT`.

### 8. Exploratory logistic regression

The notebook fits an exploratory model of the form:

```text
CARDIAC_BIN ~ AGE + C(ARM) + C(SEX) + ABN_BP_BIN
```

The model is used only to demonstrate statistical-programming workflow.

It is **not** a validated clinical prediction model. A production clinical model would require, among other things:

- prespecification;
- adequate sample size;
- model diagnostics;
- calibration;
- discrimination assessment;
- internal and external validation;
- clinical governance;
- documented intended use.

### 9. Disposition

Summarises final study disposition by treatment arm and examines adverse-event-related discontinuation.

## Visualisations

The notebook produces 11 high-resolution figures:

```text
01_benchmark_calibration.png
02_age_distribution.png
03_time_imputation.png
04_longitudinal_sbp.png
05_abnormal_sbp_by_age.png
06_ae_soc_incidence.png
07_ae_severity_distribution.png
08_cardiac_ae_incidence.png
09_followup_duration.png
10_cardiac_ae_logistic_forest.png
11_final_disposition.png
```

Figures are written to:

```text
outputs_adsl_simulation/figures/
```

Synthetic datasets are written to:

```text
outputs_adsl_simulation/data/
```

## Repository Structure

A minimal repository can be organised as:

```text
clinical-trial-adsl-derived-variables-analysis/
|
|-- README.md
|-- adsl_derived_variables_pubmed_benchmarked_analysis.ipynb
|
`-- outputs_adsl_simulation/
    |-- data/
    |   |-- dm.csv
    |   |-- ex.csv
    |   |-- vs.csv
    |   |-- ae.csv
    |   `-- ds.csv
    |
    `-- figures/
        |-- 01_benchmark_calibration.png
        |-- 02_age_distribution.png
        |-- ...
        `-- 11_final_disposition.png
```

The `outputs_adsl_simulation` directory is generated automatically when the notebook runs.

## Requirements

The notebook uses:

```text
Python 3
NumPy
pandas
Matplotlib
SciPy
statsmodels
Jupyter
```

Install the main dependencies with:

```bash
python -m pip install numpy pandas matplotlib scipy statsmodels jupyter
```

Alternatively, with Conda:

```bash
conda install numpy pandas matplotlib scipy statsmodels jupyter
```

## Key Clinical-Programming Principles Demonstrated

### Specification-driven derivation

The programmer implements the documented rule rather than inventing a clinical definition.

### One subject, one ADSL record

ADSL is subject-level. Duplicate subject rows can lead to incorrect denominators, double counting, and downstream analysis errors.

### No silent imputation

Imputed values must remain distinguishable from observed values.

### Subject counts are not event counts

One participant can contribute multiple AE records.

### Severity is not seriousness

`MILD`, `MODERATE`, and `SEVERE` describe intensity. Regulatory seriousness is a separate concept based on outcomes and consequences.

### Traceability

The analysis preserves the conceptual lineage:

```text
Source data
   ->
standardised/structured source domains
   ->
ADSL derivations
   ->
analysis
   ->
tables and figures
```

### QC is more than successful execution

Code can run successfully while still implementing the wrong derivation, using the wrong denominator, creating duplicate subjects, or misinterpreting source data.

## Literature-Based Simulation Anchors

The simulation parameters were informed by peer-reviewed journal-indexed publications. These values are used only as plausibility anchors rather than as a pooled representation of a single therapeutic population.

1. **Li C, Liang W, et al.** Sleep and risk of hypertension in general American adults: the National Health and Nutrition Examination Surveys, 2015-2018. *Journal of Hypertension*. PMID: 36129105.  
   DOI: https://doi.org/10.1097/HJH.0000000000003299

2. **Howick J, Webster R, Kirby N, Hood K.** Rapid overview of systematic reviews of nocebo effects reported by patients taking placebos in clinical trials. *Trials*. 2018;19:674. PMID: 30526685.  
   DOI: https://doi.org/10.1186/s13063-018-3042-4

3. **Chac√≥n MR, et al.** Incidence of placebo adverse events in randomized clinical trials of targeted and immunotherapy agents in the adjuvant setting. *JAMA Network Open*. PMID: 30646278.  
   DOI: https://doi.org/10.1001/jamanetworkopen.2018.5617

4. **Swain SM, et al.** Cardiac tolerability of pertuzumab plus trastuzumab plus docetaxel in patients with HER2-positive metastatic breast cancer in CLEOPATRA. *The Oncologist*. 2013. PMID: 23475636.  
   DOI: https://doi.org/10.1634/theoncologist.2012-0448

5. **Siddiqui O.** Statistical methods to analyze adverse events data of randomized clinical trials. *Journal of Biopharmaceutical Statistics*. 2009;19(5):889-899. PMID: 20183450.  
   DOI: https://doi.org/10.1080/10543400903105463

## Interpretation

The notebook demonstrates how a clinical programmer can move from source-level records to analysis-ready subject-level variables while preserving transparent derivation logic and QC.

The main technical implications are:

- reusable derivation logic improves consistency;
- source-to-derived reconciliation improves traceability;
- explicit imputation flags prevent derived information from being mistaken for observed data;
- denominator control is essential for correct AE interpretation;
- subject-level and event-level analyses answer different questions;
- synthetic data are useful for testing software and statistical workflows without exposing patient information;
- exploratory models should be clearly separated from validated clinical decision tools.

## Intended Use

This repository is intended for:

- clinical programming demonstrations;
- ADaM/ADSL training;
- statistical-programming practice;
- pharmaceutical data-science portfolios;
- reproducible workflow development;
- interview and technical-assessment preparation.

It is **not intended for patient care, regulatory submission, medical decision-making, or clinical risk prediction**.

## License
MIT 

## Author

**Mark I. R. Petalcorin**

Molecular biology, biochemistry, clinical data analytics, machine learning, and AI.

