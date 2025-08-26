# 311 Service Requests – Data Cleaning and Feature Engineering
This document explains the data cleaning, standardization, and feature engineering steps performed on the NYC 311 Service Requests dataset. The focus is on technical processing and business rationale behind each step.

## 311 Service Level Agreements 
- Rows in the SLA table with missing SLA Hours were removed as they were not managed by 311 and to ensure only eligible (Problem, Problem_Details) pairs are present.
- Some (Problem, Problem_Details) pairs had multiple SLA values because the actual SLA depends on Additional_Details variable, which is not present in the 311_Service_Requests dataset.
- To assign a single SLA per pair:
      - The majority SLA (most frequent value) was used when there was a clear majority.
      - For ties (equal frequency), the minimum SLA was selected to be strict and flagging potential SLA breaches.

This approach prevents row duplication and maintains a consistent, conservative dataset for SLA breach analysis and predictive modeling.

## 311 Service Requests
### 1. Missing Values
Variables with >80% missing values were removed. These variables add little predictive value and their information is often captured by features like 'Agency' or 'Location Type'.

### 2. Irrelevant Columns
Removed columns with high cardinality or limited relevance:
'X Coordinate (State Plane)', 'Y Coordinate (State Plane)', 'Park Facility Name', 'Park Borough', 'Incident Address', 'Street Name', 'Cross Street 1', 'Cross Street 2', 'Agency Name', 'Landmark', 'Intersection Street 1', 'Intersection Street 2', 'City', 'BBL', 'Community Board', 'Location', 'Resolution Action Updated Date', 'Resolution Description'

These are mostly text or address fields that add noise and do not improve SLA predictions.

### 3. Standardizing Columns
Variables with high cardinality but predictive relevance were standardized:
- 'Location Type' → consistent naming; grouped into 5 categories → 'Location Group'.
- 'Complaint Type' → standardized names; top categories retained; others labeled as “Other”.
- 'Open Data Channel Type' and 'Address Type' → unknown or miscellaneous values grouped as “Other”.

### 4. Feature Engineering
- Converted Created Date and Closed Date into datetime format.
- Created Resolution Time in hours to capture SLA adherence.
- Derived temporal features: Day of Week, Time of Day, Month.
- Created integrated features:
       - Day & Time
       - Borough Agency
       - Borough Location
       - Created lagging indicator: requests per week to measure past workload for Borough and Agency.

### 5. Merging with (Service Level Agreements) SLA Data
- Merged dataset with SLA tables on Complaint Type and Descriptor (Problem and Problem_Details in the SLA table).
- Standardized naming variations to match SLA records.
- Records without SLA information were left null to avoid assumptions.

### 6. Handling High Cardinality and Outliers
- Retained top 50 categories for Complaint Type and Location Type; remaining categories labeled “Other”.
- Removed records with negative 'Resolution Time'.
- Dropped 'Problem' and 'Problem_Details' after standardization.
- Removed records with:
    - Null Closed Date
    - 'Status' other than “Closed”
    - 'Borough' with "Unspecified" value
      
### 7. Business Perspective
   
- Each step ensures the dataset is clean, consistent, and predictive.
- Focused on features that matter for SLA breach prediction, while removing irrelevant or noisy data.
- Standardization and grouping reduce complexity without losing critical insights.
- Feature engineering adds temporal and integrated context to test for better predictive modeling.
