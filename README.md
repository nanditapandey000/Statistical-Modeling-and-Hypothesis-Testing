# Statistical-Modeling-and-Hypothesis-Testing

# Week 3 — Statistical Modeling Report

**Project:** Predicting LendingClub Loan Defaults
**Focus:** Hypothesis testing, logistic regression, OLS diagnostics
**Dataset:** LendingClub LoanStats3a (42,535 loans, 2007–2011)
**Report date:** November 2026

---

## 1. Objective

Week 3 asked the question: *which borrower and loan attributes carry statistically
significant signal about default risk, and how strong is that signal?* The deliverable
is a formal statistical-modeling report that frames five hypotheses, tests each with
an appropriate inferential method, and reports effect sizes, p-values, and verdicts.

The week also delivered a baseline logistic-regression model whose coefficients and
diagnostics feed directly into Week 4's machine-learning pipeline.

---

## 2. Deliverables

| File | Location | Purpose |
|------|----------|---------|
| **Week3_Statistical_Modeling_Report.pdf** | `download/` | Main narrative report (cover + body) |
| **week3_statistical_modeling.ipynb** | `download/` | Reproducible analysis notebook |
| 10 diagnostic figures | `data/week3_figures/` | ROC, PR, confusion matrix, coefficient plots, OLS diagnostics, VIF |
| `week3_stats.json` | `data/` | Machine-readable test results |
| `week3_modeling.py` | `scripts/` | Pipeline that runs all hypothesis tests |
| `build_week3_body.py` | `scripts/` | ReportLab body PDF generator |
| `week3_cover.html` / `.pdf` | `scripts/` | Cover page (Playfair Display + Inter) |

---

## 3. Dataset

- **Source:** LendingClub LoanStats3a, the cleaned dataset from Week 1.
- **Rows:** 42,535 loans originated between June 2007 and December 2011.
- **Target:** `is_default` (binary) — any loan status other than Fully Paid.
- **Default rate:** 15.12% (6,431 defaults).
- **Features used in logistic regression:** 23 (numeric + one-hot encoded categoricals).
- **Features used in OLS:** 9 (a reduced numeric set for coefficient interpretation).

---

## 4. Hypotheses Tested

| ID | Hypothesis | Test | Statistic | p-value | Verdict |
|----|------------|------|-----------|---------|---------|
| H1 | Loan amount ↑ ⇒ default ↑ | Point-biserial | r = −0.107 | 0.002 | **Not supported** (sign flipped) |
| H2 | Grade dominates interest rate | Spearman ρ | ρ = 2.195 | < 0.001 | **Supported** |
| H3 | DTI ↑ ⇒ default odds ↑ | Logit coefficient | β = 0.018 | 0.231 | **Not supported** |
| H4 | Income ↓ interest rate (controlling grade) | Partial correlation | r = −0.005 | 0.734 | **Not supported** |
| H5 | Purpose adds signal beyond grade & DTI | Likelihood-ratio χ² | — | < 0.001 | **Supported** |

Two of five hypotheses were supported. The negative correlation in H1 is the surprising
finding that drives much of the narrative: in this vintage, larger loans defaulted
*less* often, because larger loans were concentrated in higher grades (A–C). The
grade signal dominates so strongly that it reverses the naive intuition.

---

## 5. Modeling Summary

### Logistic regression (baseline classifier)

- **Input:** 23 features (numeric + one-hot).
- **AUC-ROC:** ~0.69 on the held-out test set.
- **AUC-PR:** ~0.29 (reflects the 15% base rate — precision is hard at this imbalance).
- **Top positive coefficients:** grade_ord, interest rate, inquiries in last 6 months.
- **Top negative coefficients:** annual income, credit history length.

### OLS regression (interest-rate decomposition)

- **Dependent variable:** interest rate.
- **R²:** ~0.93 — grade alone explains almost all the variance in the rate.
- **Diagnostics:** residuals roughly normal, no severe heteroskedasticity, VIF all
  below 5 (no crippling multicollinearity in the reduced set).

---

## 6. Key Figures

| File | Content |
|------|---------|
| `m01_roc_curve.png` | ROC curve for the logistic-regression baseline |
| `m02_pr_curve.png` | Precision-recall curve (more informative than ROC at 15% base rate) |
| `m03_confusion_matrix.png` | Confusion matrix at the default 0.5 threshold |
| `m04_logit_coefs.png` | Logistic-regression coefficients, sorted, with confidence intervals |
| `m05_ols_diagnostics.png` | OLS residual QQ-plot, residuals-vs-fitted, scale-location |
| `m06_vif.png` | Variance-inflation-factor bar chart |
| `m07_actual_vs_predicted.png` | Actual vs predicted interest rate (OLS) |
| `m08_ols_coefs.png` | OLS coefficients with confidence intervals |
| `m09_default_by_grade.png` | Default rate by grade — the headline visual |
| `m10_intrate_by_grade.png` | Interest rate by grade — confirming grade→rate mapping |

---

## 7. How to Reproduce

```bash
# From the repository root
python3 -I scripts/week3_modeling.py        # runs all tests, writes week3_stats.json and figures
python3 -I scripts/build_week3_body.py      # builds scripts/body_week3.pdf
python3 scripts/pdf.py code.sanitize scripts/build_week3_body.py   # optional: re-sanitize source

# Final report assembly (cover + body)
python3 -c "
from pypdf import PdfReader, PdfWriter
w = PdfWriter()
w.add_page(PdfReader('scripts/week3_cover.pdf').pages[0])
for p in PdfReader('scripts/body_week3.pdf').pages: w.add_page(p)
w.write(open('download/Week3_Statistical_Modeling_Report.pdf','wb'))
"
```

**Notebook:** upload `download/cleaned_dataset.csv` to the notebook working directory and Run All.

---

## 8. Findings 


1. **Grade is the dominant signal.** Its logistic coefficient is an order of magnitude
   larger than any other feature, and it explains ~93% of interest-rate variance in
   the OLS model. The Week-4 pipeline must preserve grade as a feature and consider
   whether interaction terms with grade add incremental lift.
2. **Multicollinearity is present but manageable.** `int_rate` and `grade` are
   collinear (r ≈ 0.95); the Week-4 feature-selection step drops `int_rate` to avoid
   leaking the target-proxy.
3. **The baseline AUC is ~0.69.** Week 4's machine-learning models are benchmarked
   against this number; gradient-boosting ensembles are expected to beat it modestly,
   primarily by capturing the interaction effects that the linear logistic model misses.

---

## 9. Limitations

- The hypothesis tests treat the 2007–2011 vintage as a static sample. Default rates
  in later LendingClub vintages (post-2014) differ materially.
- Logistic-regression coefficients are reported on the log-odds scale; their
  magnitude is not directly comparable to OLS coefficients.
- The OLS model uses a reduced feature set (9 variables) to keep coefficients
  interpretable; the full logistic model uses 23. The two models answer different
  questions and should not be compared directly.
