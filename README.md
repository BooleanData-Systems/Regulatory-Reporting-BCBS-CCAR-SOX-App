# Regulatory Reporting Accelerator

AI-powered regulatory compliance intelligence for BCBS capital adequacy, CCAR stress testing, SOX compliance, and ML-driven risk prediction.

## Overview

This Native App provides a comprehensive Regulatory Reporting dashboard covering:

- **Executive Dashboard** — CET1 ratios, risk breaches, stress test pass rates, SOX effectiveness, geographic risk
- **BCBS Capital Adequacy** — Basel III capital requirements: CET1, Tier 1, Total Capital, Leverage ratios, RWA
- **CCAR Stress Testing** — Scenario analysis, projected losses, capital impact, post-stress CET1
- **SOX Compliance** — Internal controls, audit logs, approval workflows, change history, evidence management
- **ML Risk Intelligence** — Weighted risk scoring for loan default prediction using PD, LGD, LTV features

## Installation

After installing, configure the app by binding your data tables to the required references. Each reference expects a table (or view) with the columns described below.

## Configuration Steps

1. **Install the application** from the Snowflake Marketplace.
2. **Bind table references** — On first launch, the app will prompt you to bind each of the 15 required table references via the Snowsight permissions dialog. You must grant SELECT access to tables (or views) matching the schemas described below.
3. **Validate environment** — The app runs a pre-launch validation check ensuring all views are accessible and contain data.
4. **Launch the accelerator** — Once validation passes, click "Launch Accelerator" to access the full dashboard.
5. **Filter & explore** — Use the sidebar to filter by US Region and Regulatory Framework.

## Stored Procedures & UDFs

| Object | Type | Description |
|--------|------|-------------|
| `config.register_single_reference` | Procedure | Callback invoked by Snowsight when a consumer binds/unbinds a table reference |
| `config.setup_views` | Procedure | Creates internal views pointing to bound reference tables |

## Required Privileges

The app requests **SELECT** on each bound table. No other privileges are required. All analytics run in-place on your Snowflake warehouse — no data leaves your account.

## Example SQL Commands

After installing the app, you can manually bind references via SQL if preferred:

```sql
-- Grant access to your data
GRANT USAGE ON DATABASE <YOUR_DB> TO APPLICATION REG_REPORTING_ACCELERATOR_APP;
GRANT USAGE ON SCHEMA <YOUR_DB>.<YOUR_SCHEMA> TO APPLICATION REG_REPORTING_ACCELERATOR_APP;
GRANT SELECT ON TABLE <YOUR_DB>.<YOUR_SCHEMA>.<YOUR_TABLE> TO APPLICATION REG_REPORTING_ACCELERATOR_APP;

-- Bind a reference
CALL REG_REPORTING_ACCELERATOR_APP.config.register_single_reference(
  'CAPITAL_ADEQUACY_TABLE', 'ADD',
  SYSTEM$REFERENCE('TABLE', '<YOUR_DB>.<YOUR_SCHEMA>.<YOUR_TABLE>', 'PERSISTENT', 'SELECT')
);
```

Repeat the `register_single_reference` call for each of the 15 references.

## Required Tables

| Reference | Key Columns |
|-----------|-------------|
| CUSTOMERS_TABLE | CUSTOMER_ID, CUSTOMER_NAME, REGION, SEGMENT, RISK_RATING, ONBOARDING_DATE |
| TRANSACTIONS_TABLE | TRANSACTION_ID, CUSTOMER_ID, TRANSACTION_DATE, AMOUNT, TRANSACTION_TYPE, REGION |
| LOAN_EXPOSURES_TABLE | EXPOSURE_ID, CUSTOMER_ID, LOAN_TYPE, OUTSTANDING_BALANCE, INTEREST_RATE, COLLATERAL_VALUE, LTV_RATIO, PD_SCORE, LGD_ESTIMATE, RISK_WEIGHT, ORIGINAL_AMOUNT, REGION |
| RISK_METRICS_TABLE | METRIC_ID, REGION, METRIC_DATE, METRIC_TYPE, METRIC_VALUE, THRESHOLD, BREACH_FLAG |
| CAPITAL_ADEQUACY_TABLE | REPORT_DATE (DATE), REGION, CET1_RATIO, TIER1_RATIO, TOTAL_CAPITAL_RATIO, LEVERAGE_RATIO, CET1_CAPITAL, TIER1_CAPITAL, TIER2_CAPITAL, RISK_WEIGHTED_ASSETS |
| STRESS_TEST_RESULTS_TABLE | TEST_DATE (DATE), SCENARIO_ID, REGION, PROJECTED_LOSSES, CAPITAL_IMPACT, POST_STRESS_CET1, PASS_FAIL |
| SOX_CONTROLS_TABLE | CONTROL_NAME, CONTROL_CATEGORY, CONTROL_OWNER, FREQUENCY, LAST_TEST_DATE (DATE), TEST_RESULT, DEFICIENCY_TYPE, REMEDIATION_STATUS, RISK_LEVEL, REGION |
| FINANCIAL_CONTROLS_LOG_TABLE | LOG_DATE (DATE), EVENT_TYPE, DESCRIPTION, DISCREPANCY_AMOUNT, REVIEWED_BY, STATUS, REGION |
| STRESS_TEST_SCENARIOS_TABLE | SCENARIO_ID, SCENARIO_NAME, SCENARIO_TYPE, SEVERITY, GDP_SHOCK_PCT, UNEMPLOYMENT_RATE, INTEREST_RATE_SHOCK, MARKET_DECLINE_PCT, DEFAULT_RATE_MULT |
| REGULATORY_REPORTS_TABLE | REPORT_NAME, REPORT_TYPE, REGULATORY_BODY, STATUS, ACCURACY_SCORE, REPORTING_PERIOD, SUBMISSION_DEADLINE (DATE), REGION |
| SOX_AUDIT_LOG_TABLE | AUDIT_DATE (DATE), AUDITOR, FINDING, SEVERITY, STATUS, REGION |
| SOX_APPROVAL_WORKFLOW_TABLE | SUBMITTED_BY, APPROVED_BY, SUBMISSION_DATE (DATE), APPROVAL_DATE (DATE), STATUS, REGION |
| SOX_CONTROL_VERSIONS_TABLE | VERSION_NUMBER, CHANGE_DESCRIPTION, EFFECTIVE_DATE (DATE), CHANGED_BY, REGION |
| SOX_EVIDENCE_TABLE | EVIDENCE_TYPE, EVIDENCE_DESC, COLLECTED_DATE (DATE), COLLECTED_BY, STATUS, REGION |
| GEOGRAPHIC_RISK_SUMMARY_TABLE | REGION, TOTAL_EXPOSURE, AVG_RISK_WEIGHT, HIGH_RISK_COUNT, TOTAL_CUSTOMERS, AVG_PD_SCORE, REPORT_DATE (DATE) |

## Expected Categorical Column Values

The dashboard renders charts that group, color, or filter by categorical fields. For charts to display meaningful results, these columns **must** contain values from the sets listed below.

### Region Field

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `REGION` | All tables except STRESS_TEST_SCENARIOS | `Northeast`, `Southeast`, `Midwest`, `West`, `Southwest` |

### Capital & Risk Fields

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `BREACH_FLAG` | RISK_METRICS_TABLE | `TRUE`, `FALSE` (BOOLEAN) |
| `PASS_FAIL` | STRESS_TEST_RESULTS_TABLE | `PASS`, `FAIL` |
| `METRIC_TYPE` | RISK_METRICS_TABLE | Any string (e.g., `Credit Risk`, `Market Risk`, `Operational Risk`) |

### Stress Test Fields

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `SCENARIO_TYPE` | STRESS_TEST_SCENARIOS_TABLE | `Baseline`, `Adverse`, `Severely Adverse` |
| `SEVERITY` | STRESS_TEST_SCENARIOS_TABLE | `Low`, `Medium`, `High`, `Severe` |

### SOX Compliance Fields

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `TEST_RESULT` | SOX_CONTROLS_TABLE | `Effective`, `Ineffective`, `Needs Improvement` |
| `DEFICIENCY_TYPE` | SOX_CONTROLS_TABLE | `Material Weakness`, `Significant Deficiency`, `None` |
| `REMEDIATION_STATUS` | SOX_CONTROLS_TABLE | `Completed`, `In Progress`, `Not Started`, `Overdue` |
| `RISK_LEVEL` | SOX_CONTROLS_TABLE | `Low`, `Medium`, `High`, `Critical` |
| `CONTROL_CATEGORY` | SOX_CONTROLS_TABLE | Any string (e.g., `Financial Reporting`, `IT General Controls`, `Access Controls`, `Change Management`) |
| `FREQUENCY` | SOX_CONTROLS_TABLE | `Daily`, `Weekly`, `Monthly`, `Quarterly`, `Annual` |
| `STATUS` | SOX_AUDIT_LOG, SOX_APPROVAL_WORKFLOW, SOX_EVIDENCE, FINANCIAL_CONTROLS_LOG | `Open`, `Closed`, `Pending`, `Approved`, `Rejected` |
| `SEVERITY` (Audit) | SOX_AUDIT_LOG_TABLE | `Low`, `Medium`, `High`, `Critical` |

### Loan & ML Fields

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `LOAN_TYPE` | LOAN_EXPOSURES_TABLE | `Commercial Real Estate`, `C&I`, `Residential Mortgage`, `Consumer`, `SBA`, `Auto` |
| `SEGMENT` | CUSTOMERS_TABLE | `Corporate`, `SME`, `Retail`, `Institutional` |
| `RISK_RATING` | CUSTOMERS_TABLE | `AAA`, `AA`, `A`, `BBB`, `BB`, `B`, `CCC`, `Default` |

### Regulatory Reports Fields

| Column | Table(s) | Expected Values |
|--------|----------|-----------------|
| `REPORT_TYPE` | REGULATORY_REPORTS_TABLE | `Capital`, `Liquidity`, `Stress Test`, `Compliance`, `Risk` |
| `REGULATORY_BODY` | REGULATORY_REPORTS_TABLE | `Federal Reserve`, `OCC`, `FDIC`, `SEC`, `PCAOB` |
| `STATUS` | REGULATORY_REPORTS_TABLE | `Submitted`, `Pending`, `Draft`, `Approved`, `Rejected` |

## Permissions

The app requests SELECT access on bound tables. All analytics run in-place on your Snowflake warehouse. No data leaves your account.

## Version History

| Version | Date | Notes |
|---------|------|-------|
| 1.0.0 | 2026-05-21 | Initial marketplace release |
