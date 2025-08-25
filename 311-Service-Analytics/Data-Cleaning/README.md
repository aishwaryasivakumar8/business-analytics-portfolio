## Data Cleaning & Feature Engineering
This folder contains all the work related to data cleaning, standardization, and feature engineering for the NYC 311 Service Requests dataset.

### Contents
cleaning.ipynb – Jupyter notebook implementing all data cleaning and preprocessing steps.
description.md – Detailed explanation of the data cleaning steps, including the business rationale behind each decision.

### Overview
- Missing and irrelevant data were removed to focus on predictive features.
- Columns with high cardinality were standardized and grouped to reduce noise.
- New features were created to capture temporal context and workload indicators.
- The dataset is prepared for SLA breach prediction and subsequent analysis.

More details: [Data Cleaning Description](Data-Cleaning/Description.md)
Code: [Data Cleaning Python Notebook](Data-Cleaning/311_NYC_SR_Data_Cleaning.ipynb)
