# Drift Mode Implementation Plan

## Overview
This document outlines the implementation plan for introducing a **Drift Mode** feedback mechanism in the PixIQ anomaly detection system. The drift mode uses statistical thresholds (95th and 99th percentiles) calculated from good samples during training to enable intelligent anomaly classification and automated decision-making at the edge.

---

## 1. Cloud Training Phase (PatchCore Model)

### 1.1 Objective
Calculate and store the 95th and 99th percentile anomaly scores from good samples during model training.

### 1.2 Concept
- After the PatchCore model is trained, run inference on all **good samples** from the training dataset
- Collect all anomaly scores from these good samples
- Calculate two key statistical thresholds:
  - **95th Percentile**: The score below which 95% of good samples fall
  - **99th Percentile**: The score below which 99% of good samples fall
- Store these threshold values alongside the trained model for later use

---

## 2. Backend (PixIQ Training Page)

### 2.1 Objective
Display the calculated threshold values on the training page for user reference.

### 2.2 Concept
- After training completes, display the calculated 95th and 99th percentile threshold values
- Show these values in the training results or summary page
- Users can view and note these threshold values
- These values will be manually entered into the PixIQ Edge configuration 

---

## 3. Frontend (PixIQ Training Page)

### 3.1 Objective
Display the calculated threshold values to users after training is complete.

### 3.2 Concept
 - Show the 95th and 99th percentile threshold values in the training results summary
 - Present only the two numeric threshold values:
   - **95th Percentile** — 95% of the good sample
   - **99th Percentile** — 99% of the good sample

---

## 4. Edge Deployment (PixIQ Edge)

### 4.1 Objective
Implement drift mode for real-time anomaly detection with intelligent decision-making based on the threshold values.

### 4.2 Drift Mode Classification Logic

The drift mode uses the two thresholds to create three distinct classification zones:

#### **Zone 1: Auto-Pass (Score < 95th Percentile)**
- **Interpretation**: The product is clearly good
- **Action**: Automatically write 0 "good" to PLC
- **Rationale**: 95% of known good products scored in this range, so high confidence of quality

#### **Zone 2: Review Required (95th ≤ Score < 99th Percentile)**
- **Interpretation**: The product shows some deviation but might still be acceptable
- **Action**: Send to HMI for human operator review
- **Rationale**: This is the "uncertainty zone" - some good products score here, but requires expert judgment

#### **Zone 3: High Anomaly (Score ≥ 99th Percentile)**
- **Interpretation**: The product shows significant anomaly
- **Action**: Configurable behavior:
  - **Option A**: Automatically write 1 "bad" to PLC (strict mode)
  - **Option B**: Send to HMI for human review (cautious mode)
- **Rationale**: Only 1% of good products score this high, indicating likely defect

### 4.3 Key Concepts

#### 4.3.1 Real-Time Processing Flow
1. Capture image from production line
2. Run PatchCore inference to get anomaly score
3. Compare score against the two thresholds
4. Route to appropriate action:
   - Direct PLC write for clear cases (Zone 1 and optionally Zone 3)
   - HMI review for uncertain cases (Zone 2 and optionally Zone 3)

#### 4.3.2 HMI Review Integration
- For images requiring review, send to HMI interface with:
  - Visual preview of the image
  - Calculated anomaly score
  - Which zone it falls into
  - Recommendation based on score proximity to thresholds
  - User decision buttons (Accept/Reject)

#### 4.3.3 PLC Communication
- Write results directly to PLC for automated decisions or Write user-confirmed results for review cases
- Maintain decision traceability for quality records

---

## 5. Frontend Configuration (PixIQ Edge)

### 5.1 Objective
Add "Drift Mode" as a new feedback mode option in the PixIQ interface.

### 5.2 Concept
### 5.2 Concept (concise)

- Add a single, simple selector option in the feedback-mode control: **Drift Mode**.

- Frontend responsibility is limited to:
  - Enabling/disabling Drift Mode for a device or deployment

---

## 6. Conclusion

The Drift Mode implementation introduces an intelligent, automated quality control mechanism that leverages statistical thresholds derived from training data to enable real-time decision-making at the edge. This approach offers several key benefits:

### 6.1 Operational Efficiency
- **Automated Decision-Making**: Clear-cut good products (Zone 1) are automatically passed, reducing manual review workload by up to 95%
- **Focused Human Attention**: Operators only review uncertain cases (Zone 2) and optionally high-anomaly cases (Zone 3)

### 6.2 Flexibility and Scalability
- **Configurable Behavior**: Zone 3 handling can be adjusted based on production requirements (strict vs. cautious mode)
- **Product-Specific Tuning**: Different products can have different thresholds based on their unique good-sample distributions

---


