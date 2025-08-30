# 311 NYC Services - SLA Breach Prediction
## Project Description
When I started this project, my goal was to build a predictive model that could identify potential SLA (Service Level Agreement) breaches in NYC 311 non-emergency service requests. SLA breaches are rare but costly events as they signal inefficiencies, resource strain, and risk public trust in city services. So even though most requests are resolved on time, the ability to predict which ones are at risk of breaching allows decision makers to intervene early.
 
## Preparing the Data
 - Removed rows with nulls in key columns to ensure data quality and avoid noise.
 - Created the target variable ‘SLA_Breach’
   - If ‘Resolution Time’ (In hours) > ‘SLA Hours’ → SLA Breach (1)
   - Else → No Breach (0).
 - The target distribution showed imbalance. Most service requests resolve on time while SLA breaches are the minority. This actually matches real-world operations: SLA breaches are exceptions but the ones that matter most.
 - Converted ‘Created Date’ to datetime and filtered service requests after 2024-08-08. Since the downloaded dataset only started from August 1, using dates 1–7 wouldn’t provide a valid 7-day history. Beginning from August 8 ensured the lagging feature ‘Requests Past Week’ was meaningful and kept the model forward-looking and actionable.
 - Decided to keep both individual features and interaction features (like Borough × Agency). Tree-based models can learn nonlinear splits and interactions naturally. From a business perspective, keeping Borough × Agency is valuable because it reflects joint accountability because certain borough-agency pairs underperform consistently.
 - Stratified the data by the target variable and sampled 50,000 rows to balance computational feasibility with representativeness of the underlying process.
 - The sample data was then split 80/20 into train/test. I then separated numeric and categorical features for preprocessing.
 
## Modeling Setup
 - Two preprocessing pipeline:
   - Logistic & Neural → StandardScaler + OneHotEncoder.
   - Tree-based models (Decision Tree, Random Forest, XGBoost) → ordinal Encoder.
   
     For XGBoost, newer versions support categorical features directly (enable_categorical=True), so the encoded integers are treated as categories rather than ordered numbers. Ordinal encoding was used here because SMOTE requires all features to be numeric, and enabling categorical in XGBoost ensures no artificial ordering bias.
 - Wrapped preprocessing + SMOTE-NC (oversampling for categorical imbalance) + classifier into a pipeline. SMOTE-NC was used to balance the target class while properly handling categorical features, improving model learning for the minority SLA breach cases.
 - I tested five baseline models:
   - Logistic Regression, Neural Network, Decision Tree, Random Forest, XGBoost.
 - Results:
   - Logistic & XGBoost were more stable.
   - Neural Network, Decision Tree, Random Forest overfit badly (training >> testing).
   
   Overfit models would create too many false alarms by predicting breaches that aren’t real. For real-world use, I prioritized stability and interpretability and chose XGBoost as the best candidate.
 
## Model Development
- I ran Randomized Search hyperparameter tuning over:
     - n_estimators, max_depth, learning_rate, subsample.
- The best parameters were applied to train the final XGBoost pipeline.
 
## Feature Importance
I used Permutation Importance to evaluate the model.
The top features identified by permutation importance validate business intuition, but the low scores show that no single feature dominates. Instead, the model learns from a balanced mix of features making its predictions more reliable than relying on one driver alone.
I applied a 0.02 threshold to remove noisy features like ‘Address Type’ and ‘Open Data Channel’.
- All features:
        - Train Acc: 0.878 | Train F1: 0.825
        - Test Acc: 0.849 | Test F1: 0.785
- After feature selection:
        - Train Acc: 0.868 | Train F1: 0.811
        - Test Acc: 0.841 | Test F1: 0.775
The drop in accuracy/F1 was minimal, but the model became simpler and more interpretable. In real world use, this tradeoff is often preferred.
 
## Threshold Tuning
By default, classifiers use a 0.5 probability threshold. But in imbalanced problems, adjusting this can improve performance.
- At 0.5 threshold:
     - TP = 1487, TN = 6926
- At optimal threshold (via F1 tuning):
     - TP = 1598, TN = 6762
This adjustment improved recall for breaches by predicting more breaches, even though the F1 changed only slightly.
Threshold tuning lets the city choose:
 - Lower threshold → catch more breaches early (at the cost of more false alarms).
 - Higher threshold → fewer false alarms, but risk missing breaches.
This flexibility is critical for budget allocation and resource planning.
 
## Key Business Uses
1.	Early warning system → Predict SLA breaches in advance → agencies can reallocate staff.
2.	Resource optimization → Identify Borough & Agency hotspots → prioritize interventions.
3.	Policy insight → Feature importance shows timing & geography drive breaches more than reporting channel.
4.	Customer trust → Reducing SLA breaches improves citizen satisfaction with 311.
5.	Operational dashboards → Predictions can power real-time monitoring.
   
## Conclusion 
Through careful data preparation, model selection, feature reduction, and threshold tuning, I built a predictive system that balances performance with stability and interpretability.
I intentionally kept things focused on accuracy and F1 score, while leaving precision/recall tradeoff exploration as a future step. The insights already demonstrate clear operational value: city managers can anticipate SLA risks to optimize resources and ultimately deliver better service to residents.
 

