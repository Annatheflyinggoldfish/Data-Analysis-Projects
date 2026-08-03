# NHS Talking Therapies 2019 - 2025: National Trends & Demographic Analysis

**A data analysis project examining service demand, treatment outcomes, demographic representation, and geographic equity in NHS Talking Therapies (IAPT) services across England.**

### Tools Used

**`Python`· `pandas` · `NumPy` · `SciPy`· `Plotly`**


## Overview

NHS Talking Therapies (formerly IAPT) is England's flagship programme for treating anxiety and depression through evidence-based psychological therapies. This project analyses whether the service access and treatment outcomes are equitable across different population groups and geographic areas, using publicly available NHS Digital data.

**Key Questions Explored:**

- How have referral volumes, access rates, and recovery rates changed over time nationally?
- Are certain demographic groups (gender, ethnicity) over/under represented compared to the general population?
- Does deprivation (IMD) correlate with variation in service access and/or outcomes?
- Are there meaningful geographic disparities in service access/outcome across England's 42 Integrated Care Boards (ICBs)?

The analysis is structured as three linked notebooks, moving from raw data to a national-level statistical picture:

| Notebook | Focus | Key Techniques |
|---|---|---|
| [`01_data_cleaning.ipynb`](https://colab.research.google.com/drive/1LvEKX0ORIJt1LcAqN6YgoPYoXnbyyZmA#scrollTo=K2j33NjYn6zp) | Cleaning and structuring raw NHS Digital datasets | Data validation, handling missing values, schema standardisation |
| [`02_national_trends_eda.ipynb`](https://colab.research.google.com/drive/1SMcayvIqioYXsH-vVryktwTvnqZMyR15#scrollTo=hNVyojD7UT10) | National-level trend exploration | Time series analysis, referral/recovery rate trends |
| [`03_demographic_geographic_eda.ipynb`](https://colab.research.google.com/drive/1Ga34-NX5Gnakc6pC7JuKS13H8tPdXDp1#scrollTo=61WdBP8VV9Uf) | Demographic and geographic breakdown | Chi-square tests, correlation analysis, representation ratios, geospatial mapping |

---

## Methodology Highlights

- **Data standardisation with SQL**: Used DuckDB SQL queries to standardise column naming and structure across six years of data before merging into a single dataset.
- **Geospatial matching pipeline**: Matched 42 ICBs to ONS population estimates using prefix-based matching to handle inconsistent naming conventions across datasets.
- **Representation ratios**: Constructed using working-age, England-only population denominators to compare demographic and geographic population shares among service users against the general population — ensuring comparisons were methodologically sound rather than misleading.
- **Statistical testing**: Applied chi-square tests and correlation analysis, with explicit attention to the distinction between statistical significance and practical/real-world significance — findings are only flagged as noteworthy where both are present.

---

## Sample Outputs

**Note:** For **interactive versions** of these charts and the full analysis, see the Colab notebooks above.

---

### **Service Demand & Treatment Funnel Overview** - referral, access, and recovery volumes over time.

<details>
<summary>View code: Service Funnel Conversion Rates</summary>

```python
# Service Funnel Conversion Rates
conversion_df = pd.DataFrame({
    'Year': df_treatment['Year'],
    'Referral to Accessing (%)': df_treatment['Accessing Services'] / df_treatment['Referrals Received'] * 100,
    'Lost at Accessing (%)': 100 - df_treatment['Accessing Services'] / df_treatment['Referrals Received'] * 100,
    'Lost at Finishing (%)': (df_treatment['Accessing Services'] - df_treatment['Finished Course Treatment']) / df_treatment['Referrals Received'] * 100
}).set_index('Year')

print(f"Referrals Received --> Accessing Services: {conversion_df['Lost at Accessing (%)'].mean().round(1)}% loss.")
print(f"Accessing Services --> Finished Course Treatment: {conversion_df['Lost at Finishing (%)'].mean().round(1)}% loss.")
```

</details>

<img width="988" height="448" alt="image" src="https://github.com/user-attachments/assets/3b217cd0-10a1-496f-adcf-7ac992c6803f" />

### **Gender Breakdown Dashboard** — service use by gender relative to population share.

<details>
<summary>View code: representation ratio calculation</summary>

```python
# representation ratio calculation
# used across the gender, ethnicity, and IMD analyses
df['representation_ratio'] = (
    df['group_pct_of_users'] / df['group_pct_of_population']
)
```

</details>

<img width="962" height="552" alt="image" src="https://github.com/user-attachments/assets/54af85d6-62e1-4cb2-b7c3-de731a78a6b9" />

### **Ethnicity: Representation Ratio vs. Recovery Rate** — comparing access representation against treatment outcomes across ethnic groups.

<details>
<summary>View code: representation ratio calculation</summary>

```python
# representation ratio calculation
# used across the gender, ethnicity, and IMD analyses
df['representation_ratio'] = (
    df['group_pct_of_users'] / df['group_pct_of_population']
)
```

</details>

<img width="896" height="476" alt="image" src="https://github.com/user-attachments/assets/b42fa3f8-e515-44d0-883d-b26383bc2d7d" />

### **IMD (Index of Multiple Deprivation) Dashboard** — service access and outcomes by deprivation decile.

<details>
<summary>View code: representation ratio calculation</summary>

```python
# representation ratio calculation
# used across the gender, ethnicity, and IMD analyses
df['representation_ratio'] = (
    df['group_pct_of_users'] / df['group_pct_of_population']
)
```

</details>

<img width="1080" height="544" alt="image" src="https://github.com/user-attachments/assets/5d87250c-26d7-4d8e-ac29-4fdead489741" />


### **Geographic Representation Map** — access disparities across England's 42 Integrated Care Boards.

<details>
<summary>View code: representation ratio calculation</summary>

```python
# representation ratio calculation
# used across the gender, ethnicity, and IMD analyses
df['representation_ratio'] = (
    df['group_pct_of_users'] / df['group_pct_of_population']
)
```

</details>

<img width="700" height="455" alt="image" src="https://github.com/user-attachments/assets/b83a0671-6f0e-48f4-aacc-32ed58f2cce3" />

---

## Data Source

All datasets were drawn from publicly available sources. All data are aggregated, publicly published statistics; no patient-level or identifiable data is used.

* **NHS Talking Therapies data:** [NHS Digital, Psychological Therapies Annual Reports 2019/20–2024/25](https://digital.nhs.uk/data-and-information/publications/statistical/nhs-talking-therapies-for-anxiety-and-depression-annual-reports)

  *This is the cleaned, merged version of six annual [NHS Digital datasets](https://github.com/Annatheflyinggoldfish/Data-Analysis-Projects/blob/main/2019-2025%20NHS%20Therapy%20Analytics%20-%20Datasets/NHS_Talking_Therapies_Cleaned_All_Years.csv). See `01 NHS Talking Therapies - Cleaning.ipynb` for the full cleaning process.*
* **ICB geographic boundaries:** [Integrated Care Boards (April 2023) Boundaries EN BSC](https://geoportal.statistics.gov.uk/datasets/ons::integrated-care-boards-april-2023-boundaries-en-bsc/about)
* **Ethnicity reference data:** [Ethnic group, England and Wales - Office for National Statistics](https://www.ons.gov.uk/peoplepopulationandcommunity/culturalidentity/ethnicity/bulletins/ethnicgroupenglandandwales/census2021#ethnic-groups-in-england-and-wales)
* **ONS population by ICB:** [ONS, Mid-2022 revised (Nov 2025) to Mid-2024: Integrated Care Boards (2024 geography edition)](https://www.ons.gov.uk/peoplepopulationandcommunity/populationandmigration/populationestimates/datasets/clinicalcommissioninggroupmidyearpopulationestimates/mid2022revisednov2025tomid2024integratedcareboards2024geography)

---

## About This Project

This project was built as part of my transition into data analysis, with a particular interest in public sector and health data. I'm currently seeking junior data analyst roles in the NHS, NGO, and public sector space.

**Contact**: huanglp582@gmail.com · [LinkedIn](https://www.linkedin.com/in/huangliping)
