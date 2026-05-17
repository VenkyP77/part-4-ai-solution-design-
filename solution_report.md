# AI Solution Document: Predictive Quality Control in Manufacturing

## Task 1: Choose a Business Domain

**Answer:** Manufacturing Domain

---

## Task 2: Define the Business Problem

### What problem is being solved?
**Answer:** In manufacturing plants, defective products often slip through the production line undetected until final inspection or worse, until they reach the customer. This leads to costly rework, material waste, customer complaints, and brand damage. The goal is to **predict product defects early in the production process** so that corrective action can be taken before a batch is ruined.

### Who are the users or stakeholders?
**Answer:** The key stakeholders / users and their role in the process is as follows:
1. Quality Control Engineers - They monitor products for quality, identify defects and take corrective actions
2. Production Line Managers - They are responsible for the production process and quality of product manufactured from their line. They make changes to the process and machine settings as needed, to address production problems or quality issues.
3. Plant Managers - They track overall production and quality KPIs. They target reduction in waste costs across production lines across the plant.
4. Maintenance Teams - These teams are responsible for corrective and preventive maintenance of all machinery in the plant. They conduct periodic maintenance, but also respond to immediate issues. 
5. End Customers - They are the final consumers of the product. They expect a product with the best quality and zero defects.

### What is the current manual or traditional process?
**Answer:*** Today, the manufacturing plants rely on:
1. **Visual inspection** by human operators at the end of the production line
2. **Random sampling** where only a small percentage of products are tested
3. **Rule-based thresholds** on machine parameters (e.g., if temperature exceeds X, raise an alarm)
4. **Post-production lab testing** which takes hours or days to return results. Also, for some types of quality checks, this testing is destructive.

### What are the limitations of the current process?
**Answer:** The following are the key limitations of the current process:
1. **Too slow** — defects are caught after hundreds or thousands of units are already produced
2. **Inconsistent** — human inspectors get tired, miss subtle defects, and vary in judgment
3. **Reactive, not proactive** — problems are found after they happen, not prevented
4. **Low coverage** — random sampling misses defects in untested units
5. **Expensive** — rework and scrap costs add up quickly when defects go undetected

---

## Task 3: Identify the AI Task Type

### Classification: **Anomaly Detection**
This problem is best classified as **Anomaly Detection** (with elements of binary classification).

### Why this AI task type is suitable?
**Answer:** Following are the key reasons:
1. **Defects are rare events** — In a well-run plant, most products are good. Defective items are the exception (often less than 2-5% of production). Anomaly detection is specifically designed for such imbalanced scenarios.
2. **Pattern recognition** — Defects often correlate with unusual combinations of sensor readings (temperature spikes, vibration changes, pressure drops) that deviate from normal operating patterns.
3. **Early warning capability** — By learning what "normal" production looks like, the model can flag deviations before a full defect forms.
4. **No need for large defect datasets** — Unlike pure classification, anomaly detection can learn primarily from normal data, which is abundantly available.

---

## Task 4: Data Requirement Plan

### Type of data needed

|# | Data Category | Examples |
|--|------------|----------|
|1.| Sensor data | Temperature, pressure, vibration, humidity, speed |
|2.| Machine parameters | RPM, torque, feed rate, cycle time |
|3.| Raw material properties | Thickness, composition, supplier batch ID |
|4.| Environmental conditions | Ambient temperature, humidity on shop floor |
|5.| Quality inspection results | Pass/fail labels, defect type codes |

### Structured or unstructured data

1. **Primarily structured data** — time-series sensor readings stored in tabular format
2. **Some unstructured data** — inspection images (optional, for visual defect confirmation)

### Input features

1. Machine sensor readings (10-50 sensors depending on equipment)
2. Production speed and cycle time
3. Material batch properties
4. Time-based features (shift, hour of day, days since last maintenance)
5. Cumulative machine run hours

### Target variable or labels

1. **Primary target**: Binary label — Defective (1) or Non-defective (0)
2. **Secondary target**: Defect type category (crack, dimensional error, surface finish, etc.)

### Data collection method

1. **IoT sensors** already installed on production equipment (PLC/SCADA systems)
2. **MES (Manufacturing Execution System)** logs for production parameters
3. **Quality management system** records for historical inspection results
4. **ERP system** for material and batch information (e.g. SAP ERP)
5. Minimum **3-6 months of historical data** will be needed for training

### Data quality risks

|# | Risk | Impact | Mitigation |
|--|------|--------|------------|
|1.| Missing sensor readings | Gaps in time-series data | Implement data imputation; add sensor health monitoring |
|2.| Incorrect labels | Model learns wrong patterns | Cross-validate labels with multiple inspection methods |
|3.| Sensor drift over time | Gradual accuracy degradation | Regular sensor calibration schedule |
|4.| Class imbalance | Model biased toward "no defect" | Use oversampling (SMOTE) or anomaly detection approach |
|5.| Data silos | Incomplete picture of production | Integrate data from MES, SCADA, and quality systems |

---

## Task 5: Model Recommendation

### Recommended Model: **LSTM Autoencoder (with Isolation Forest as baseline)**

### Architecture Overview

```
Input (sensor time-series) → LSTM Encoder → Compressed Representation → LSTM Decoder → Reconstructed Output
                                                    ↓
                                        Reconstruction Error → Anomaly Score → Defect Prediction
```

### Why this model is appropriate:

1. **LSTM handles time-series data naturally** — Manufacturing sensor data is sequential. An LSTM can capture patterns like "temperature rose gradually over 10 minutes before the defect occurred." Standard models would miss these temporal dependencies.

2. **Autoencoder learns normal behavior** — The model is trained on normal production data to reconstruct it accurately. When a defective pattern appears, the reconstruction error spikes, signaling an anomaly. This is ideal since we have abundant normal data but limited defect examples.

3. **Isolation Forest as a simpler baseline** — Start with this tree-based anomaly detector for quick wins. It requires no deep learning infrastructure and works well with tabular sensor snapshots.

4. **Practical advantages**:
   - The model can be retrained incrementally as new data arrives
   - It outputs interpretable anomaly scores (not just yes/no)
   - It will works even when new, when previously unseen defect types appear

---

## Task 6: Evaluation Plan

### Technical metrics

|# | Metric | Target | Why it matters |
|--|--------|--------|----------------|
|1.| Recall (Sensitivity) | > 90% | We must catch most defects — missed defects are costly |
|2.| Precision | > 70% | Too many false alarms cause alert fatigue |
|3.| F1-Score | > 0.80 | Balanced measure of recall and precision |
|4.| AUC-ROC | > 0.85 | Overall model discrimination ability |
|5.| False Positive Rate | < 15% | Operators should trust the alerts |

### Business metrics

|# | Metric | Current | Target |
|--|--------|---------|--------|
|1.| Defect escape rate | 3-5% | < 1% |
|2.| Scrap/rework cost | Baseline | 40-60% reduction |
|3.| Customer complaint rate | Baseline | 30% reduction |
|4.| Inspection time per unit | Manual baseline | 70% reduction |
|5.| Mean time to detect defect | End of line | Within 5 minutes of occurrence |

### Possible failure cases

1. **Concept drift** — Production conditions change (new material, new product line) and the model's learned "normal" becomes outdated
2. **Sensor failure** — Model receives garbage data and makes unreliable predictions
3. **Novel defect types** — A completely new failure mode that doesn't match any learned pattern

### Human review and validation process

1. **Daily review**: Quality engineers review all flagged anomalies and confirm/reject them
2. **Weekly calibration**: Compare model predictions against actual inspection results
3. **Monthly model audit**: Data science team checks for drift and retrains the model, if needed
4. **Quarterly business review**: Assess whether business metrics are improving - Value realization from the model
5. **Feedback loop**: Every confirmed or rejected prediction feeds back into retraining data

---

## Task 7: Responsible AI Considerations

### Incorrect predictions

1. **False negatives (missed defects)**: Defective products reach customers, causing safety risks or brand damage.
2. **False positives (false alarms)**: Production is halted unnecessarily, causing delays and loss of trust in the system.
**Mitigation**: Set conservative thresholds initially; tune based on feedback over time.

### Bias in data

1. **Historical bias**: If past inspectors were less thorough on night shifts, the training data may under-represent defects from those periods. The model could learn to under-flag night production.
2. **Supplier bias**: If defect labels correlate with specific suppliers, the model might unfairly penalize certain material sources without considering other factors.
**Mitigation**: Audit training data across shifts, suppliers, and product lines for balanced representation.

### Over-reliance on AI

1. **De-skilling risk**: If operators stop paying attention because "the AI will catch it," human expertise degrades over time.
2. **Automation complacency**: Trusting the system blindly during edge cases it was never trained for.
**Mitigation**: Position AI as a decision-support tool, not a replacement. Maintain manual inspection for critical safety components.

### Need for human oversight

1. **Critical decisions stay with humans**: The AI flags; a human decides whether to halt production.
2. **Override capability**: Operators must be able to override or dismiss predictions with documented reasoning.
3. **Escalation path**: If the model consistently disagrees with human judgment, trigger a review rather than forcing either side.

---

