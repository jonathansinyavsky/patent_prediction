# CS210 Final Project — Modeling Patent Impact and Examination Timelines

This project analyzes large-scale United States patent data to predict two outcomes that matter in intellectual property strategy:
- Whether a granted patent will become high-impact (measured by forward citations).
- How long a patent takes to move from filing to grant (USPTO examination time).

The work combines SQL-based preprocessing, feature engineering, and CatBoost machine learning models to create an end-to-end prediction pipeline.
```
Repository Structure
patent_prediction/
│
├── data_sql/
│   ├── schema.sql                   # SQL table definitions for all PatentsView tables
│   └── feature_engineering_script.sql   # SQL script building ml_patent_base
│
├── notebooks/
│   ├── high_impacts_models.ipynb        # High-impact patent classification pipeline
│   └── time_to_issue_models.ipynb       # Time-to-issue regression + bucket models
│
├── .gitignore                         # Virtual environment & system file exclusions
└── README.md

```

## Data Source

All datasets come from PatentsView, a public USPTO-backed resource containing millions of patent records.
Raw TSV files (≈150M rows across tables) were imported into PostgreSQL for preprocessing.

## SQL Pipeline

The SQL scripts:
- Create all required tables (schema.sql)
- Construct a cleaned, feature-rich modeling table (ml_patent_base) containing:
    - inventor and assignee counts
    - CPC and USPC code statistics
    - citation counts
    - date part extractions
    - CPC breadth, early CPC flag
    - assignee type indicators
    - influence score
    - time-to-issue in days

The final table contains >9M rows and ~34 engineered features per patent.

## Python Workflow
- The modeling workflow includes:
    - Chunked loading of the 9M-row dataset
    - Cleaning and validation
    - Feature engineering (complexity, team size, CPC diversity, timing variables)
    - CatBoost models for:
        - High-impact classification
        - Regression for exam timelines
        - 3- and 4-bucket timeline classification models

## Modeling Results

High-Impact Classifier
- AUC: 0.791
- Recall for high-impact patents: 87%
- Key predictors: filing year, CPC section, WIPO kind, CPC breadth, claim count

Time-to-Issue Regression
- MAE: 313 days
- RMSE: 405 days
- R²: 0.34

Bucketed Timeline Models
- 4-bucket accuracy: 0.53
- 3-bucket accuracy: 0.68

These models reveal meaningful structural patterns even though the USPTO timeline is highly variable.

## Limitations
- PatentsView includes only granted patents, so approval prediction is not possible.
- Missing and inconsistent metadata required extensive cleaning.
- Examination time depends on many unobserved factors (examiner workload, legal disputes, etc.).

## How to Reproduce
1. Load PatentsView TSV files into PostgreSQL.
2. Run schema.sql to create tables.
3. Run feature_engineering_script.sql to generate ml_patent_base.
4. Export the table to CSV.
5. Run the Jupyter notebooks in the notebooks/ directory.
