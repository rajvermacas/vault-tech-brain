# Challenges of Building an In-House News Classification System

## Objective

Build and maintain an internal system that classifies financial news into categories such as:

- Planned Price-Sensitive
- Unplanned Price-Sensitive
- Not Price-Sensitive

At first glance this appears to be a machine learning problem. In practice, the model itself is often the smallest part of the effort. Most complexity lies in data, governance, evaluation, maintenance, and business alignment.

---

# 1. Taxonomy Definition Challenges

Before any model can be trained, the organization must agree on the definitions.

Examples:

| News Event | Planned? | Unplanned? |
|------------|-----------|------------|
| Dividend declaration | Usually Planned | |
| Large order win | ? | ? |
| Acquisition announcement | ? | ? |
| Regulatory approval | ? | ? |
| CEO retirement announced months in advance | ? | ? |
| Board-approved merger | ? | ? |

Common issues:

- Different business users interpret categories differently.
- Definitions evolve over time.
- Edge cases become a significant portion of discussions.
- Historical labels may become inconsistent.

---

# 2. Labeling Challenges

Supervised models require labeled examples.

Requirements:

- Thousands to tens of thousands of news items.
- Consistent labeling standards.
- Subject matter experts to review classifications.

Risks:

- Labeling is expensive.
- Labeling quality directly determines model quality.
- Different reviewers may disagree.
- Business rules change over time.

Example:

A model trained on inconsistent labels will learn inconsistent behavior.

---

# 3. Data Quality Challenges

Financial news data often contains:

- Duplicate stories
- Rewritten stories from different vendors
- Headlines with little context
- Incomplete information
- Vendor-specific terminology

Examples:

- "Company secures strategic agreement"
- "Receives Letter of Award"
- "Board approves restructuring plan"

These may require business interpretation rather than keyword matching.

---

# 4. Evaluation Challenges

Building a model is not enough.

Questions that must be answered:

- What accuracy is acceptable?
- What false positive rate is acceptable?
- What false negative rate is acceptable?
- Which mistakes are most costly?

Example:

Missing a major fraud event may be significantly worse than incorrectly classifying a dividend announcement.

---

# 5. Hidden Model Lifecycle Costs

A production classifier requires:

- Training pipeline
- Validation pipeline
- Model versioning
- Monitoring
- Retraining process
- Rollback process

The effort continues long after initial deployment.

---

# 6. Domain Drift

Financial markets evolve.

New event types appear regularly:

- Cyber incidents
- AI-related partnerships
- New regulatory disclosures
- Novel financing structures

A model trained today may degrade over time.

This requires:

- Monitoring
- Re-labeling
- Retraining

---

# 7. Explainability Requirements

Business users frequently ask:

- Why was this classified as Unplanned?
- Why was this classified as Price Sensitive?
- Why did the model change its opinion?

Traditional ML models often require additional tooling for explanation.

---

# 8. Infrastructure Requirements

A production-grade solution typically requires:

- Data storage
- Feature generation
- Model serving
- Monitoring dashboards
- Audit trails
- Security controls

These components often require more engineering effort than the model itself.

---

# 9. Governance and Auditability

Banks typically require:

- Model approval processes
- Change management
- Audit trails
- Validation documentation
- Periodic reviews

These requirements increase operational complexity.

---

# 10. Long-Term Ownership

Questions to address:

- Who owns the taxonomy?
- Who reviews classification errors?
- Who retrains models?
- Who monitors drift?
- Who approves model changes?

Without clear ownership, model quality tends to deteriorate over time.

---

# 11. Cost Underestimation Risk

A common misconception:

    News -> ML Model -> Classification

Actual reality:

    News
      -> Taxonomy Definition
      -> Labeling
      -> Data Preparation
      -> Feature Engineering
      -> Training
      -> Evaluation
      -> Deployment
      -> Monitoring
      -> Retraining
      -> Governance
      -> Audit

The machine learning model is often a small fraction of the total effort.

---

# Alternative Approach

A modern LLM-based approach can significantly reduce initial investment:

    News
      -> LLM
      -> Structured Classification

Benefits:

- No initial labeled dataset required
- Faster time-to-market
- Reduced ML specialization required
- Immediate business validation possible

A common strategy is:

1. Start with LLM-based classification.
2. Store all classifications and business feedback.
3. Build a labeled dataset over time.
4. Evaluate whether an internal classifier is justified later.

This reduces upfront risk while allowing the organization to learn from real-world usage.



https://chatgpt.com/share/6a20f805-fc90-8321-a561-8e18307a7e3e