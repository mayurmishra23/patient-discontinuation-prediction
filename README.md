# Patient Discontinuation Prediction (Point-in-Time Modeling)

## Problem Overview

Patients on specialty therapies often appear stable but may suddenly discontinue treatment due to:

- Prior Authorization (PA) denials  
- High out-of-pocket costs  
- Missed refills or shipment delays  
- Lack of timely intervention  

Field support teams (patient services, specialty pharmacy coordinators) have **limited bandwidth**, making it critical to:

> Identify patients at high risk of discontinuation *before it happens* and prioritize outreach.

---

## Data Description

This project uses **longitudinal patient-level healthcare data** across three datasets:

### 1. `events_status_shipments.csv`
Primary event-level dataset containing:
- Patient journey events (referrals, shipments, refills, denials, appeals)
- Timestamps for each activity
- Event types (e.g., `PA_Denied`, `Shipment_Released`, `Refill_Rejected`)
- Prediction anchor flag (`is_prediction_anchor = 1`)

This dataset is used to:
- Construct **time-based behavioral features**
- Identify **patient state at prediction moment**

---

### 2. `hcps_master.csv`
Site-level attributes:
- Site type and specialty
- Historical discontinuation rates
- Engagement characteristics

Used to capture **provider/site-level risk signals**

---

### 3. `payers_master.csv`
Payer/plan characteristics:
- Coverage type
- Step therapy requirements
- Cost-sharing structure
- Access friction indicators

Used to model **payer-driven barriers**

---

### Target Variable

- `discontinued_within_60d`  
Binary indicator of whether a patient discontinued therapy within 60 days after the prediction anchor.

---

## Point-in-Time Modeling Framework (Critical)

To simulate real-world deployment:

- Each patient has a **prediction anchor** (`is_prediction_anchor = 1`)
- All features are computed using **only data available up to that timestamp**
- No future information is used

This ensures:
- No data leakage  
- Realistic model performance  
- Production-ready logic  

---

## Feature Engineering

A total of **37 features** were engineered across key behavioral and operational dimensions:

---

### 1. Refill Cadence & Timing
Captures adherence patterns:
- Mean refill gap  
- Max refill gap  
- Refill gap variability  
- Days since last shipment  

Irregular refill behavior is a strong early warning signal.

---

###️ 2. Access Friction Signals
Captures barriers to therapy:
- Count of PA denials  
- Refill rejections  
- Appeals submitted  
- Delays in approval  

These represent **structural reasons for drop-off**

---

### 3. Patient Support Engagement
- Number of support calls  
- Interaction frequency  

High interaction often indicates **patient distress or issues**

---

### 4. Therapy Progression
- Fill number at anchor  
- Early-stage vs established patients  

Patients in **early therapy (Fill 1–2)** have highest dropout risk

---

### 5. Site-Level Risk
- Historical discontinuation rate of site  

Some providers inherently have higher drop-off rates

---

### 6. Payer Context
- Coverage restrictions  
- Cost-sharing level  
- Plan type  

Financial and policy barriers significantly impact continuation

---

## Modeling Approach

### Models Used:
- Logistic Regression (baseline)
- Gradient Boosting Model (final)

### Validation Strategy:
- 5-fold cross-validation  
- Out-of-fold predictions for unbiased evaluation  

---

## Model Performance

| Metric | Value |
|------|------|
| ROC-AUC | 0.969 |
| PR-AUC | 0.899 |
| Baseline AUC (LR) | 0.961 |
| Top Decile Lift | 4.1× |

---

### Business Interpretation

- Average discontinuation rate: **23.4%**
- Top decile: **~95.7% discontinuation**

The model effectively isolates high-risk patients for targeted intervention.

---

## Key Drivers of Discontinuation

1. **Refill Gap Variability**  
   → Strongest predictor (~4× higher risk)

2. **Support Call Volume**  
   → Indicates unresolved patient issues

3. **Early Therapy Stage**  
   → Highest drop-off occurs early

4. **Site-Level Risk**  
   → Provider behavior impacts outcomes

---

## Business Output

### Risk Segmentation

| Risk Tier | Patients | Action |
|----------|--------|--------|
| High Risk | 669 (22%) | Immediate outreach (24–48 hrs) |
| Medium Risk | 166 (6%) | Weekly follow-up |
| Low Risk | 2165 (72%) | Routine monitoring |

---

### Actionable Flags

For high-risk patients:

- **PA Denied / Refill Rejected**
  → Initiate appeals, coordinate with payer

- **High Cost Share**
  → Enroll in affordability programs

- **Early Therapy**
  → Provide onboarding support

- **Refill Due Soon**
  → Send proactive reminders

---

## Outputs

- `all_patients_scored.csv` → predicted probabilities  
- `high_risk_patients.csv` → prioritized intervention list  
- Decile-level performance tables  
- Feature importance plots  

---

## Key Learnings

- Temporal modeling is critical in healthcare ML  
- Data leakage can severely inflate performance if not handled  
- Behavioral + operational features outperform static demographics  
- Business-aligned metrics (lift) are more useful than AUC alone  

---

## ▶️ How to Run

```bash
pip install -r requirements.txt
jupyter notebook notebooks/discontinuation_prediction.ipynb