---
title: "Which Startups Close? A Temporal Analysis of Funding History"
subtitle: "AI and Big Data Analytics — Weeks 4 to 8 Assessment"
author: "Student name · Student ID · Group"
---

# 1 Dataset and research question

The data come from Kaggle's *Startup Success Prediction* set, an extract of Crunchbase records. Each row is one US startup. The file has 923 rows and 49 columns. Firms were founded between 1984 and 2013. Every firm has a resolved outcome: 597 were acquired and 326 closed. None is still operating.

The intended user is an early-stage investor or accelerator analyst. The decision is simple: which firms deserve extra due diligence before more money goes in. A missed failure is the expensive mistake. So the positive class is **closed**.

**Main question.** Can a startup's funding and network history, available before exit, identify which firms will close rather than be acquired?

**Supporting questions.** (1) Which pre-exit features are most strongly associated with closure? (2) Do startups form stable funding profiles, and do closure rates differ between them?

Three limits were clear before any analysis. The data hold only resolved outcomes. Counts such as relationships and funding rounds grow while a firm is alive. And several variables, such as `is_top500`, have no documented definition.

# 2 Data audit and evaluation design

The file loaded with the expected 923 × 49 shape. The audit found six issues.

One company, Redwood Systems, appeared twice with identical values except its coordinates. I kept one row, leaving 922. Forty-six firms showed first funding before their founding date. The median gap was three months, which points to a late-registered founding date. I set those ages to missing. All 152 firms with no milestone age also had zero milestones, so that gap is structural. It was imputed inside the pipeline with an indicator. Funding ran from $11 thousand to $5.7 billion, with skewness of 23.5. The extremes are real firms such as Clearwire, so I kept them and used a log scale. Four columns were export debris or copies and were dropped.

Leakage was the larger risk. `labels` is a copy of `status`. `closed_at` is filled for all 326 closed firms. The ages at *last* funding and *last* milestone measure how long a firm stayed active, which closure cuts short. `is_top500` has no stated date or source. All were excluded. Cumulative counts stayed in the model, flagged as the main limitation. The final feature set has 7 numeric and 18 binary variables.

The split is temporal. Firms were sorted by founding date. The oldest 64% (590 firms, 1984 to April 2007) form the training set. The next 16% (147 firms, to the end of 2008) form validation. The youngest 20% (185 firms, 2009 to 2013) form the test set. This mirrors use: a model learns from the past and screens newer firms. The closed rate rises from 0.327 in training to 0.454 in test, so the test is harder than the past.

All imputers, scalers, PCA and models were fitted on training or development data only. Cross-validation (five stratified folds, seed 42) ran on the 737 development firms. The test set was used once.

# 3 Exploratory analysis and PCA

Three figures each answer one question.

*Is funding skewed, and does it differ by outcome?* Yes to both. On a raw axis a handful of firms dominate. On a log scale the curve is close to bell-shaped. Median funding was $18.3 million for acquired firms and $10.6 million for closed ones in training. The two curves overlap heavily.

*How does the closed rate change with network size and milestones?* Steeply. Among firms with 0–2 relationships, 70% closed (n = 120). With 11 or more, 9% closed (n = 145). With no milestone, 62% closed; with three or more, 18%. These counts also grow with time alive, so part of the gap is exposure.

*Which features move together?* Funding rounds and total funding (Spearman ρ = 0.60). Relationships and milestones (ρ = 0.59). Closure is most tied to relationships (ρ = −0.45). The largest variance inflation factor is 3.2, so redundancy is moderate.

PCA was fitted on the seven numeric features of the training set. Four components are needed to pass 80% of variance (83.6%). PC1 (32%) is **track record**: relationships, rounds and milestones load positively, late first funding negatively. PC2 (23%) is **late, large, syndicated rounds**: milestone age, investors per round, funding and first-funding age. On the two-component map closed firms gather at low PC1, but the classes overlap. The map drops 45% of numeric variance and every binary feature. It does not prove separation or clusters.

# 4 Predictive analysis and errors

The baseline always predicts "acquired". In cross-validation its accuracy is 0.672. On test it falls to 0.546, and its recall is zero. It shows how hollow accuracy is under imbalance.

The primary model is logistic regression with balanced class weights inside one pipeline: median imputation with missing indicators, scaling, then the classifier. Cross-validation chose regularisation C = 0.03 (ROC-AUC 0.797, SD 0.032). A random forest served as a comparison.

The threshold was fixed on validation before the test was touched. The rule: the highest threshold that keeps recall for closed at 0.80 or more. That gave 0.45. Maximising F2 was rejected because it chose 0.30, the edge of the grid, where precision fell below 0.45.

| Model | Partition | Accuracy | Precision | Recall | F1 | ROC-AUC |
|---|---|---|---|---|---|---|
| Baseline | CV, development (n = 737) | 0.672 | 0.000 | 0.000 | 0.000 | 0.500 |
| Logistic regression | CV, development (n = 737) | 0.719 | 0.559 | 0.703 | 0.621 | 0.797 |
| Random forest | CV, development (n = 737) | 0.750 | 0.615 | 0.645 | 0.628 | 0.803 |
| Baseline | Test (n = 185) | 0.546 | 0.000 | 0.000 | 0.000 | 0.500 |
| Logistic regression, t = 0.45 | Test (n = 185) | 0.692 | 0.615 | 0.857 | 0.716 | 0.822 |
| Random forest, t = 0.45 | Test (n = 185) | 0.719 | 0.643 | 0.857 | 0.735 | 0.830 |

On the test set logistic regression caught 72 of 84 closed firms and missed 12. It also flagged 45 of 101 acquired firms. About four in ten flags are false alarms. For a screening tool that cost is acceptable: a false alarm costs a second look. The random forest is close, but a gain of 0.008 in ROC-AUC does not justify losing readable coefficients.

Test ROC-AUC (0.822) sits slightly above cross-validation (0.797). That does not show better generalisation. Partitions differ in firms and base rates. What it shows is that the ranking held up on younger firms.

Coefficients are associations inside the model. One standard deviation more relationships goes with about half the odds of closure (odds ratio 0.51). Investors per round (0.80) and total funding (0.81) point the same way. Later first funding (1.22) and no milestone (1.31) point toward closure. None of this is causal.

I inspected the 20 most confident test errors: all 12 false negatives and the 8 strongest false positives. Three patterns appeared. First, **healthy-looking failures**. The missed closures had a median of 7.5 relationships and a Series A round. Color Labs raised $41 million and still closed. Funding data cannot see product or market failure. Second, **small acquisitions**. False positives had median funding of $1.5 million and four relationships. Many were likely bought for the team. "Acquired" is not the same as "succeeded". Third, **short histories**. Three quarters of the false negatives were founded in 2010 or later. Young firms had less time to build the counts the model rewards.

# 5 Clustering and stability

Clustering asks whether startups fall into funding profiles. I used six features: log funding, rounds, log relationships, milestones, investors per round and age at first funding. The outcome and identifiers were excluded. Features were median-imputed and standardised on the training set, so Euclidean distance weights each one equally. Logs tame the funding and network tails that would otherwise drive K-means.

| k | Silhouette | Calinski-Harabasz | Davies-Bouldin | Smallest cluster | Bootstrap ARI |
|---|---|---|---|---|---|
| 2 | 0.214 | 178.5 | 1.634 | 239 | 0.763 |
| 3 | 0.198 | 158.3 | 1.578 | 161 | 0.625 |
| 4 | 0.187 | 144.5 | 1.518 | 120 | 0.700 |
| 5 | 0.182 | 134.5 | 1.487 | 96 | 0.602 |
| 6 | 0.188 | 125.7 | 1.447 | 17 | 0.630 |

No index gives a clear answer. Silhouette stays near 0.2 for every k, which means weak, overlapping structure. Silhouette and Calinski-Harabasz favour k = 2. Davies-Bouldin keeps improving, but k = 6 produces a cluster of 17 firms. I chose **k = 4**. It has the second-best stability, a lower Davies-Bouldin than k = 2 or 3, no cluster under 120 firms, and profiles that differ clearly in original units. The simpler k = 2 remains a defensible fallback.

| Cluster | Size | Medians in original units | Name |
|---|---|---|---|
| 0 | 126 | $7.5M, 1 round, 2 relationships, first funding at 4.5 years | Thin late starters |
| 1 | 192 | $10.0M, 2 rounds, 8 relationships, first funding at 1.4 years | Networked early starters |
| 2 | 152 | $24.3M, 2 rounds, 4 investors per round, first funding at 4.2 years | Late syndicated raisers |
| 3 | 120 | $38.9M, 4 rounds, 13 relationships, 3 milestones | Serial-funded leaders |

Stability was tested three ways. Twenty bootstrap refits gave a mean adjusted Rand index of 0.70 (minimum 0.46). Ten single-start seeds gave 0.62 (minimum 0.30). Ward clustering agreed with K-means at only 0.24. The solution is moderately stable within K-means, but its borders depend on the method.

Only after clustering did I look at outcomes. In training, 67.5% of thin late starters closed, against 18.3% of serial-funded leaders. On the test set, 88% of the 50 thin late starters closed. The segments carry real signal. Yet 102 of 185 test firms fall into cluster 1 and only 11 into cluster 3. Young firms have not yet built long histories, which repeats the exposure problem.

The solution is interpretable and useful for describing a portfolio. It is not proof of natural groups.

# 6 Method suitability, claim audit and conclusions

**Text.** The dataset has no genuine unstructured text. Its non-numeric columns are names, places, dates and a category code. Running TF-IDF on them would only re-encode dummies. A valid extension would collect each firm's description as written at first funding, encode it, and test late fusion under the same temporal split.

**Neural model.** A one-layer MLP with 16 ReLU units used the same preprocessing and boundary. Validation loss bottomed at epoch 38 (0.472). By epoch 200 training loss had fallen to 0.395 while validation loss rose to 0.661. That is overfitting. Refitted to 38 epochs, the network scored test ROC-AUC 0.742 against 0.822 for logistic regression. Extra capacity added nothing.

**Claim audit.** I gave an LLM the compact result table and asked for five statements. One was supported: the 0.822 test ROC-AUC against a 0.500 baseline. One was contradicted: random forest did not win on every metric. One was overstated: recall of 0.857 describes 84 firms founded 2009–2013, not "startups that will close". Two were unsupported: an accuracy comparison across partitions with different base rates, and a causal claim about networks. Fluent text was not evidence.

**Conclusions.** Pre-exit funding history ranks startups by risk of closure, and the ranking held on younger firms. Network size is the strongest single signal. Four funding profiles emerge with moderate stability and closure rates from 18% to 68%. The most important limitation is exposure: cumulative counts grow while a firm survives, so part of the signal may be time alive rather than quality. The data also come from one source, hold only resolved outcomes, and treat every acquisition as success.

The model should be used as a screening flag, not a verdict. Flagged firms go to manual review. The next test is to rebuild every feature at a fixed firm age, such as two years after founding, and check whether the ROC-AUC holds.

## References

- Kaggle. *Startup Success Prediction* (manishkc06). https://www.kaggle.com/datasets/manishkc06/startup-success-prediction
- Pedregosa, F. et al. (2011). Scikit-learn: Machine Learning in Python. *JMLR* 12, 2825–2830.
- Course practice notebooks, Weeks 4–8 (classification, clustering, neural learning curves).
