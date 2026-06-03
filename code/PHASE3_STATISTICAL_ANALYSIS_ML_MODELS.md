# PHASE 3: RISK DETECTION & ANALYSIS - STATISTICAL TESTING & ML MODELS

## STEP 3.1: PATTERN MINING & ANOMALY DETECTION

### SQL: Cohort Analysis - High-Risk Seller Patterns

```sql
-- Identify cohorts with elevated violation rates
SELECT 
    s.seller_country,
    EXTRACT(QUARTER FROM s.account_created_date) as onboarding_quarter,
    EXTRACT(YEAR FROM s.account_created_date) as onboarding_year,
    COUNT(DISTINCT s.seller_id) as seller_count,
    COUNT(DISTINCT v.violation_id) as total_violations,
    ROUND(COUNT(DISTINCT v.violation_id) / NULLIF(COUNT(DISTINCT s.seller_id), 0), 2) as violations_per_seller,
    ROUND(SUM(CASE WHEN v.violation_type = 'COUNTERFEIT' THEN 1 ELSE 0 END) / 
          NULLIF(COUNT(DISTINCT v.violation_id), 0) * 100, 2) as counterfeit_pct,
    ROUND(AVG(DATEDIFF(DAY, s.account_created_date, v.violation_date)), 1) as avg_days_to_first_violation
FROM sellers s
LEFT JOIN violations v ON s.seller_id = v.seller_id
GROUP BY seller_country, onboarding_quarter, onboarding_year
HAVING violations_per_seller > 0.3
ORDER BY violations_per_seller DESC;

-- Time Series: Trend Analysis
SELECT 
    DATE_TRUNC(CURRENT_DATE(), WEEK) as week,
    COUNT(DISTINCT v.violation_id) as violations_detected,
    COUNT(DISTINCT CASE WHEN v.violation_status = 'OVERTURNED' THEN v.violation_id END) as appeals_overturned,
    ROUND(COUNT(DISTINCT CASE WHEN v.violation_status = 'OVERTURNED' THEN v.violation_id END) / 
          NULLIF(COUNT(DISTINCT v.violation_id), 0) * 100, 2) as overturn_rate_pct,
    COUNT(DISTINCT v.seller_id) as unique_sellers
FROM violations v
WHERE v.violation_date > DATE_SUB(CURRENT_DATE(), INTERVAL 12 WEEK)
GROUP BY week
ORDER BY week DESC;

-- Anomaly Detection: Seller Transaction Spike
SELECT 
    t.seller_id,
    CURRENT_DATE() as detection_date,
    COUNT(DISTINCT t.transaction_id) as transactions_today,
    AVG(daily_counts.daily_transaction_count) as avg_daily_transactions,
    STDDEV_POP(daily_counts.daily_transaction_count) as transaction_stddev,
    ROUND((COUNT(DISTINCT t.transaction_id) - AVG(daily_counts.daily_transaction_count)) / 
          NULLIF(STDDEV_POP(daily_counts.daily_transaction_count), 0), 2) as z_score
FROM transactions t
JOIN (
    SELECT seller_id, DATE(transaction_date) as txn_date, 
           COUNT(DISTINCT transaction_id) as daily_transaction_count
    FROM transactions
    WHERE transaction_date > DATE_SUB(CURRENT_DATE(), INTERVAL 30 DAY)
    GROUP BY seller_id, txn_date
) as daily_counts ON t.seller_id = daily_counts.seller_id
WHERE DATE(t.transaction_date) = CURRENT_DATE()
GROUP BY t.seller_id
HAVING z_score > 2.5
ORDER BY z_score DESC;
```

---

## STEP 3.3: STATISTICAL HYPOTHESIS TESTING & ML MODEL DEVELOPMENT

### Python: Statistical Testing Framework

```python
import pandas as pd
import numpy as np
from scipy import stats
from scipy.stats import chi2_contingency, ttest_ind, mannwhitneyu
import statsmodels.api as sm
from statsmodels.formula.api import logit
import logging

logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class StatisticalAnalyzer:
    """Hypothesis testing and statistical analysis for risk operations"""
    
    @staticmethod
    def mcnemar_test(legacy_predictions: np.array, optimized_predictions: np.array) -> dict:
        """
        McNemar's Test: Compare two paired classifiers
        Tests if optimized pipeline catches significantly more violations than legacy
        
        Args:
            legacy_predictions: Array of 0/1 (violation detected by legacy system)
            optimized_predictions: Array of 0/1 (violation detected by optimized system)
        
        Returns:
            Dictionary with test statistic and p-value
        """
        # Create contingency table
        # [no, no]: both systems missed
        # [no, yes]: legacy missed, optimized caught
        # [yes, no]: legacy caught, optimized missed
        # [yes, yes]: both caught
        
        nn = np.sum((legacy_predictions == 0) & (optimized_predictions == 0))
        ny = np.sum((legacy_predictions == 0) & (optimized_predictions == 1))
        yn = np.sum((legacy_predictions == 1) & (optimized_predictions == 0))
        yy = np.sum((legacy_predictions == 1) & (optimized_predictions == 1))
        
        # McNemar test statistic: (n10 - n01)^2 / (n10 + n01)
        mcnemar_stat = (ny - yn) ** 2 / (ny + yn) if (ny + yn) > 0 else 0
        p_value = 1 - stats.chi2.cdf(mcnemar_stat, df=1)
        
        return {
            'test_name': 'McNemar Test',
            'test_statistic': mcnemar_stat,
            'p_value': p_value,
            'is_significant': p_value < 0.05,
            'contingency_table': {
                'both_missed': nn,
                'legacy_missed_optimized_caught': ny,
                'legacy_caught_optimized_missed': yn,
                'both_caught': yy
            },
            'improvement': f"+{ny} additional violations caught (p={p_value:.2e})"
        }
    
    @staticmethod
    def disparate_impact_analysis(enforcement_data: pd.DataFrame, protected_groups: list) -> dict:
        """
        Disparate Impact Analysis: Check if enforcement is fair across demographics
        
        Regulatory floor: 0.80 (80/20 rule)
        If one group affected at <80% rate of another, enforcement is biased
        
        Args:
            enforcement_data: DataFrame with enforcement_action, demographic_group columns
            protected_groups: List of demographic group names
        
        Returns:
            Dictionary with disparate impact ratios
        """
        results = {'test_name': 'Disparate Impact Analysis', 'results': {}}
        
        # Calculate enforcement rate for each group
        group_enforcement_rates = {}
        for group in protected_groups:
            group_data = enforcement_data[enforcement_data['demographic_group'] == group]
            enforcement_rate = group_data['enforcement_action'].sum() / len(group_data) if len(group_data) > 0 else 0
            group_enforcement_rates[group] = enforcement_rate
        
        # Calculate disparate impact ratios
        max_rate = max(group_enforcement_rates.values())
        for group, rate in group_enforcement_rates.items():
            ratio = rate / max_rate if max_rate > 0 else 1.0
            results['results'][group] = {
                'enforcement_rate': rate,
                'disparate_impact_ratio': ratio,
                'passes_regulatory_floor': ratio >= 0.80
            }
        
        results['regulatory_floor'] = 0.80
        results['is_compliant'] = all(r['passes_regulatory_floor'] for r in results['results'].values())
        
        return results
    
    @staticmethod
    def chi_square_test(df: pd.DataFrame, var1: str, var2: str) -> dict:
        """
        Chi-Square Test: Test association between two categorical variables
        Example: "Is seller country associated with fraud rate?"
        """
        contingency_table = pd.crosstab(df[var1], df[var2])
        chi2, p_value, dof, expected = chi2_contingency(contingency_table)
        
        return {
            'test_name': 'Chi-Square Test',
            'variables': f'{var1} vs {var2}',
            'chi2_statistic': chi2,
            'p_value': p_value,
            'degrees_of_freedom': dof,
            'is_significant': p_value < 0.05,
            'interpretation': f"Variables {'ARE' if p_value < 0.05 else 'are NOT'} associated (p={p_value:.4f})"
        }
    
    @staticmethod
    def ttest_two_groups(group1_data: np.array, group2_data: np.array) -> dict:
        """
        T-Test: Compare means of two groups
        Example: Compare fraud rates between KYC-verified vs unverified sellers
        """
        t_stat, p_value = ttest_ind(group1_data, group2_data)
        
        return {
            'test_name': 't-test (Independent Samples)',
            'group1_mean': np.mean(group1_data),
            'group2_mean': np.mean(group2_data),
            'group1_std': np.std(group1_data),
            'group2_std': np.std(group2_data),
            't_statistic': t_stat,
            'p_value': p_value,
            'is_significant': p_value < 0.05,
            'effect_size': (np.mean(group1_data) - np.mean(group2_data)) / np.sqrt((np.var(group1_data) + np.var(group2_data)) / 2)
        }
    
    @staticmethod
    def mann_whitney_u_test(group1_data: np.array, group2_data: np.array) -> dict:
        """
        Mann-Whitney U Test: Non-parametric alternative to t-test
        Better for non-normally distributed data
        """
        u_stat, p_value = mannwhitneyu(group1_data, group2_data, alternative='two-sided')
        
        return {
            'test_name': 'Mann-Whitney U Test',
            'u_statistic': u_stat,
            'p_value': p_value,
            'is_significant': p_value < 0.05,
            'group1_median': np.median(group1_data),
            'group2_median': np.median(group2_data)
        }


class MLModelDeveloper:
    """Develop and validate ML models for risk prediction"""
    
    @staticmethod
    def prepare_training_data(df: pd.DataFrame, feature_cols: list, target_col: str, test_size: float = 0.2):
        """Prepare data for ML model training"""
        from sklearn.model_selection import train_test_split
        
        X = df[feature_cols].fillna(0)
        y = df[target_col]
        
        X_train, X_test, y_train, y_test = train_test_split(
            X, y, test_size=test_size, random_state=42, stratify=y
        )
        
        return {
            'X_train': X_train, 'X_test': X_test,
            'y_train': y_train, 'y_test': y_test,
            'feature_cols': feature_cols
        }
    
    @staticmethod
    def train_logistic_regression(X_train, y_train):
        """Train logistic regression model"""
        from sklearn.linear_model import LogisticRegression
        from sklearn.preprocessing import StandardScaler
        
        scaler = StandardScaler()
        X_scaled = scaler.fit_transform(X_train)
        
        model = LogisticRegression(max_iter=1000, random_state=42)
        model.fit(X_scaled, y_train)
        
        return {'model': model, 'scaler': scaler, 'model_type': 'LogisticRegression'}
    
    @staticmethod
    def train_random_forest(X_train, y_train):
        """Train Random Forest model"""
        from sklearn.ensemble import RandomForestClassifier
        
        model = RandomForestClassifier(n_estimators=100, max_depth=15, random_state=42, n_jobs=-1)
        model.fit(X_train, y_train)
        
        return {'model': model, 'model_type': 'RandomForest', 'feature_importances': model.feature_importances_}
    
    @staticmethod
    def train_xgboost(X_train, y_train):
        """Train XGBoost model"""
        import xgboost as xgb
        
        model = xgb.XGBClassifier(n_estimators=100, max_depth=6, learning_rate=0.1, random_state=42, n_jobs=-1)
        model.fit(X_train, y_train)
        
        return {'model': model, 'model_type': 'XGBoost', 'feature_importances': model.feature_importances_}
    
    @staticmethod
    def evaluate_model(model, X_test, y_test):
        """Evaluate model performance"""
        from sklearn.metrics import precision_score, recall_score, f1_score, roc_auc_score, confusion_matrix
        
        y_pred = model.predict(X_test)
        y_pred_proba = model.predict_proba(X_test)[:, 1] if hasattr(model, 'predict_proba') else None
        
        results = {
            'accuracy': model.score(X_test, y_test),
            'precision': precision_score(y_test, y_pred, zero_division=0),
            'recall': recall_score(y_test, y_pred, zero_division=0),
            'f1_score': f1_score(y_test, y_pred, zero_division=0),
            'roc_auc': roc_auc_score(y_test, y_pred_proba) if y_pred_proba is not None else None,
            'confusion_matrix': confusion_matrix(y_test, y_pred).tolist(),
            'true_negatives': confusion_matrix(y_test, y_pred)[0, 0],
            'false_positives': confusion_matrix(y_test, y_pred)[0, 1],
            'false_negatives': confusion_matrix(y_test, y_pred)[1, 0],
            'true_positives': confusion_matrix(y_test, y_pred)[1, 1]
        }
        
        return results
    
    @staticmethod
    def hyperparameter_tuning(X_train, y_train, model_type: str = 'xgboost'):
        """Hyperparameter tuning with GridSearchCV"""
        from sklearn.model_selection import GridSearchCV
        
        if model_type == 'xgboost':
            import xgboost as xgb
            model = xgb.XGBClassifier(random_state=42)
            param_grid = {
                'n_estimators': [50, 100, 200],
                'max_depth': [3, 6, 9],
                'learning_rate': [0.01, 0.1, 0.3]
            }
        elif model_type == 'random_forest':
            from sklearn.ensemble import RandomForestClassifier
            model = RandomForestClassifier(random_state=42, n_jobs=-1)
            param_grid = {
                'n_estimators': [50, 100, 200],
                'max_depth': [10, 15, 20],
                'min_samples_split': [5, 10]
            }
        
        grid_search = GridSearchCV(model, param_grid, cv=5, scoring='f1', n_jobs=-1)
        grid_search.fit(X_train, y_train)
        
        return {
            'best_model': grid_search.best_estimator_,
            'best_params': grid_search.best_params_,
            'best_score': grid_search.best_score_,
            'cv_results': grid_search.cv_results_
        }


# ============================================================================
# USAGE EXAMPLE
# ============================================================================

if __name__ == "__main__":
    # Example: McNemar's Test
    legacy_system = np.array([0, 1, 0, 1, 0, 0, 1, 1, 0, 0])
    optimized_system = np.array([0, 1, 1, 1, 0, 1, 1, 1, 1, 0])
    
    analyzer = StatisticalAnalyzer()
    mcnemar_result = analyzer.mcnemar_test(legacy_system, optimized_system)
    print("McNemar Test Results:")
    print(f"  p-value: {mcnemar_result['p_value']:.2e}")
    print(f"  Is Significant: {mcnemar_result['is_significant']}")
    print(f"  Improvement: {mcnemar_result['improvement']}")
    
    # Example: Disparate Impact Analysis
    enforcement_df = pd.DataFrame({
        'demographic_group': ['USA', 'USA', 'EU', 'EU', 'APAC', 'APAC'] * 100,
        'enforcement_action': np.random.binomial(1, 0.3, 600)
    })
    
    disparate_impact = analyzer.disparate_impact_analysis(
        enforcement_df, 
        protected_groups=['USA', 'EU', 'APAC']
    )
    print("\nDisparate Impact Analysis:")
    print(f"  Compliant: {disparate_impact['is_compliant']}")
    for group, result in disparate_impact['results'].items():
        print(f"  {group}: {result['disparate_impact_ratio']:.2f}")
```

---

## STEP 3.3: ML MODEL DEPLOYMENT & EXPLAINABILITY

### Python: SHAP-based Model Explainability

```python
import shap
import matplotlib.pyplot as plt

class ModelExplainability:
    """Generate explanations for model predictions using SHAP"""
    
    @staticmethod
    def compute_shap_values(model, X_train, X_test, model_type: str = 'xgboost'):
        """Compute SHAP values for model explanation"""
        
        if model_type == 'xgboost':
            explainer = shap.TreeExplainer(model)
        elif model_type == 'random_forest':
            explainer = shap.TreeExplainer(model)
        else:
            # For linear models
            explainer = shap.LinearExplainer(model, X_train)
        
        shap_values = explainer.shap_values(X_test)
        return {'explainer': explainer, 'shap_values': shap_values}
    
    @staticmethod
    def explain_prediction(model, X_test, sample_idx: int, feature_names: list):
        """Generate human-readable explanation for single prediction"""
        
        explainer = shap.TreeExplainer(model)
        shap_values = explainer.shap_values(X_test.iloc[[sample_idx]])
        
        prediction = model.predict_proba(X_test.iloc[[sample_idx]])[0, 1]
        
        # Get top contributing features
        feature_contributions = pd.DataFrame({
            'feature': feature_names,
            'shap_value': shap_values[0],
            'abs_shap_value': np.abs(shap_values[0])
        }).sort_values('abs_shap_value', ascending=False)
        
        return {
            'prediction': prediction,
            'top_features': feature_contributions.head(5).to_dict('records'),
            'explanation': f"Account flagged as high-risk (prob={prediction:.2%}) due to: "
                          + ", ".join([f"{row['feature']}={row['shap_value']:.3f}" 
                                      for _, row in feature_contributions.head(3).iterrows()])
        }
```

**✅ Phase 3 Complete - Production-Ready Statistical Analysis & ML Models**
