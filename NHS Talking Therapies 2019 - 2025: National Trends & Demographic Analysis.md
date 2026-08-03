# NHS Talking Therapies 2019 - 2025: National Trends & Demographic Analysis

**A data analysis project examining service demand, treatment outcomes, demographic representation, and geographic equity in NHS Talking Therapies (IAPT) services across England.**

---

## Overview

NHS Talking Therapies (formerly IAPT) is England's flagship programme for treating anxiety and depression through evidence-based psychological therapies. This project analyses whether the service access and treatment outcomes are equitable across different population groups and geographic areas, using publicly available NHS Digital data.

The analysis is structured as three linked notebooks, moving from raw data to a national-level statistical picture:

| Notebook | Focus | Key Techniques |
|---|---|---|
| [`01_data_cleaning.ipynb`](notebooks/01_data_cleaning.ipynb) | Cleaning and structuring raw NHS Digital datasets | Data validation, handling missing values, schema standardisation |
| [`02_national_trends_eda.ipynb`](notebooks/02_national_trends_eda.ipynb) | National-level trend exploration | Time series analysis, referral/recovery rate trends |
| [`03_demographic_geographic_eda.ipynb`](notebooks/03_demographic_geographic_eda.ipynb) | Demographic and geographic breakdown | Chi-square tests, correlation analysis, representation ratios, geospatial mapping |

---

## Key Questions Explored

- How have referral volumes, access rates, and recovery rates changed over time nationally?
- Are certain demographic groups (gender, ethnicity) over/under represented compared to the general population?
- Does deprivation (IMD) correlate with variation in service access and/or outcomes?
- Are there meaningful geographic disparities in service access/outcome across England's 42 Integrated Care Boards (ICBs)?

---

## Methodology Highlights

- **Geospatial matching pipeline**: Matched 42 ICBs to ONS population estimates using prefix-based matching to handle inconsistent naming conventions across datasets.
- **Representation ratios**: Constructed using working-age, England-only population denominators to ensure demographic comparisons were methodologically sound rather than misleading.
- **Statistical testing**: Applied chi-square tests and correlation analysis, with explicit attention to the distinction between statistical significance and practical/real-world significance — findings are only flagged as noteworthy where both are present.

---

## Sample Outputs



```
![National referral trend](assets/national_trend.png)
![ICB representation ratio map](assets/icb_map.png)
```

---

## Key Code Sample

*(Pick 1–2 short, representative snippets that show your analytical thinking — e.g. the representation ratio calculation. This lets a technical reviewer judge code quality even if they don't open the notebook.)*

```python
# Example: representation ratio calculation
# (replace with your actual snippet)
df['representation_ratio'] = (
    df['service_users_pct'] / df['working_age_population_pct']
)
```

---

## Data Source

All datasets were drawn from publicly available sources. All data are aggregated, publicly published statistics; no patient-level or identifiable data is used.

* **NHS Talking Therapies data:** [NHS Digital, Psychological Therapies Annual Reports 2019/20–2024/25](https://digital.nhs.uk/data-and-information/publications/statistical/nhs-talking-therapies-for-anxiety-and-depression-annual-reports)

  *This is the cleaned, merged version of six annual [NHS Digital datasets](https://github.com/Annatheflyinggoldfish/Data-Analysis-Projects/blob/main/2019-2025%20NHS%20Therapy%20Analytics%20-%20Datasets/NHS_Talking_Therapies_Cleaned_All_Years.csv). See `01 NHS Talking Therapies - Cleaning.ipynb` for the full cleaning process.*
* **ICB geographic boundaries:** [Integrated Care Boards (April 2023) Boundaries EN BSC](https://geoportal.statistics.gov.uk/datasets/ons::integrated-care-boards-april-2023-boundaries-en-bsc/about)
* **Ethnicity reference data:** [Ethnic group, England and Wales - Office for National Statistics](https://www.ons.gov.uk/peoplepopulationandcommunity/culturalidentity/ethnicity/bulletins/ethnicgroupenglandandwales/census2021#ethnic-groups-in-england-and-wales)
* **ONS population by ICB:** [ONS, Mid-2022 revised (Nov 2025) to Mid-2024: Integrated Care Boards (2024 geography edition)](https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/populationestimates/datasets/clinicalcommissioninggroupmidyearpopulationestimates/mid2022revisednov2025tomid2024integratedcareboards2024geography)

---

## Tools Used

`Python` · `pandas` · `NumPy` · `SciPy` (chi-square, correlation) · `Plotly` 

---

## Repository Structure

```
├── README.md
├── notebooks/
│   ├── 01_data_cleaning.ipynb
│   ├── 02_national_trends_eda.ipynb
│   └── 03_demographic_geographic_eda.ipynb
├── data/
│   └── (raw/processed data or a note on data sourcing, if not included)
└── assets/
    └── (screenshots referenced above)
```

---

## About This Project

This project was built as part of my transition into data analysis, with a particular interest in public sector and health data. I'm currently seeking junior data analyst roles in the NHS, NGO, and public sector space.

**Contact**: [your email] · [your LinkedIn]
