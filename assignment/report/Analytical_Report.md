AI and Big Data Analytics, Weeks 4 to 8 Assessment

Student: \_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_ ID / Group:
\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_\_

# 1. Dataset and research question

I used the Startup Success Prediction dataset from Kaggle. It is an
extract of Crunchbase records. Each row is one US startup. The file has
923 rows and 49 columns, and the firms were founded between 1984 and
2013. Every firm in the file already has an outcome: 597 were acquired
and 326 closed. No firm is still operating.

I wrote the analysis for an early-stage investor or an accelerator
analyst who has to decide which startups need extra due diligence before
more money goes in. For this user, missing a firm that later closes is
the costly mistake. For that reason I made closed the positive class.

Main question: can a startup's funding and network history, available
before exit, show which firms will close rather than be acquired?
Supporting questions: which pre-exit features are most linked to
closure, and do startups form stable funding profiles with different
closure rates?

# 2. Data audit and evaluation design

The file loaded with the expected 923 rows and 49 columns. Redwood
Systems appeared twice with the same values except the coordinates, so I
kept one row and worked with 922 firms. In 46 firms the first funding
came before the founding date. The median gap was about three months, so
I set these ages to missing. All 152 firms without a milestone age also
had zero milestones. I imputed this inside the pipeline with an
indicator.

Total funding ranged from \$11 thousand to \$5.7 billion, with a
skewness of 23.5. The largest values belong to real companies such as
Clearwire, so I kept them and used a log transform. I dropped four
columns that were left over from the export or duplicated other columns.

Leakage needed more care than cleaning. The labels column is a copy of
status. closed_at is filled for all 326 closed firms, so it gives the
answer away. The age at last funding and the age at last milestone show
how long a firm stayed active, and closure cuts that time short.
is_top500 has no definition or date. I removed all of these. I kept the
cumulative counts but treated them as the main limitation. The final
model uses 7 numeric and 18 binary features.

I split the data by founding date because the model is supposed to
screen newer firms. The oldest 590 firms (1984 to April 2007) are the
training set, the next 147 (up to the end of 2008) are validation, and
the youngest 185 (2009 to 2013) are the test set. That is roughly 64%,
16% and 20%. The closed rate rises from 0.327 in training to 0.454 in
test. All preprocessing and models were fitted on training or
development data, cross-validation used five folds on the 737
development firms (seed 42), and the test set was used once.

# 3. Exploratory analysis and PCA

Figure 1 checks whether funding is skewed and whether it differs by
outcome. On a log scale it looks close to normal. In the training set
the median funding was \$18.3 million for acquired firms and \$10.6
million for closed firms, but the two distributions overlap a lot.

Figure 2 looks at the closed rate by network size and by milestones. The
drop is steep. Of the firms with 0 to 2 relationships, 70% closed (n =
120). Of those with 11 or more, only 9% closed (n = 145). Firms with no
milestone closed in 62% of cases, firms with three or more in 18%. Both
counts also grow with age, so some of this gap is just time.

Figure 3 is a Spearman correlation matrix. Funding rounds and total
funding move together (0.60), and so do relationships and milestones
(0.59). Closure is most strongly linked to relationships (-0.45). The
highest VIF is 3.2, so the features overlap, but not badly.

I fitted PCA on the seven numeric features of the training set. Four
components are needed to reach 80% of the variance (83.6%). The first
component (32%) looks like a track record: relationships, rounds and
milestones load positively and late first funding loads negatively. The
second component (23%) describes firms that raised large rounds with
many investors, but later in life. On the two-dimensional plot closed
firms sit more to the left, but the classes overlap. The plot leaves out
45% of the variance, so it does not prove clusters or class separation.

# 4. Predictive analysis and errors

As a baseline I used a model that always predicts acquired. Its accuracy
is 0.672 in cross-validation and 0.546 on the test set, and it never
finds a closed firm. Accuracy alone means little here.

My main model is logistic regression with balanced class weights.
Imputation, scaling and the model are in one pipeline. Cross-validation
picked C = 0.03 (ROC-AUC 0.797, SD 0.032). I also fitted a random forest
for comparison. I chose the threshold on the validation set before
touching the test set. The rule was to take the highest threshold that
still keeps recall for closed at 0.80 or more, which gave 0.45. I did
not use the F2 maximum, because it landed on 0.30, the lowest value I
tried, where precision was under 0.45.

Table 1. Model results

| **Model**                     | **Partition**             | **Accuracy** | **Precision** | **Recall** | **F1** | **ROC-AUC** |
|-------------------------------|---------------------------|--------------|---------------|------------|--------|-------------|
| Baseline                      | CV, development (n = 737) | 0.672        | 0.000         | 0.000      | 0.000  | 0.500       |
| Logistic regression           | CV, development (n = 737) | 0.719        | 0.559         | 0.703      | 0.621  | 0.797       |
| Random forest                 | CV, development (n = 737) | 0.750        | 0.615         | 0.645      | 0.628  | 0.803       |
| Baseline                      | Test (n = 185)            | 0.546        | 0.000         | 0.000      | 0.000  | 0.500       |
| Logistic regression, t = 0.45 | Test (n = 185)            | 0.692        | 0.615         | 0.857      | 0.716  | 0.822       |
| Random forest, t = 0.45       | Test (n = 185)            | 0.719        | 0.643         | 0.857      | 0.735  | 0.830       |

On the test set the logistic regression found 72 of the 84 closed firms
and missed 12. It also flagged 45 of the 101 acquired firms, so about
four flags in ten are false alarms. For a screening tool this is
acceptable, since a false alarm only means a second look. The random
forest is very close, but 0.008 more ROC-AUC is not worth losing
coefficients that can be read.

The test ROC-AUC (0.822) is a bit higher than in cross-validation
(0.797). The partitions have different base rates, so this only shows
that the ranking still works on younger firms.

The coefficients are associations inside the model, not causes. One
standard deviation more relationships goes together with about half the
odds of closure (odds ratio 0.51). Late first funding (1.22) and having
no milestone (1.31) go with higher odds of closure.

I went through the 20 most confident errors on the test set: all 12
false negatives and the 8 strongest false positives. The missed closures
often looked healthy on paper, with a median of 7.5 relationships and a
Series A. Color Labs, for example, raised \$41 million and still closed.
The false positives were mostly small firms with a median of \$1.5
million in funding and four relationships. Many were probably bought for
the team. Finally, three quarters of the false negatives were founded in
2010 or later. They had less time to build up the counts the model
rewards.

# 5. Clustering and stability

For clustering I used six features: log funding, funding rounds, log
relationships, milestones, investors per round and age at first funding.
The outcome and the identifiers were left out. I imputed and
standardised the features on the training set, so each feature has the
same weight in the Euclidean distance.

Table 2. K-means for k = 2 to 6 (training set, n = 590)

| **k** | **Silhouette** | **Calinski-Harabasz** | **Davies-Bouldin** | **Smallest cluster** | **Bootstrap ARI** |
|-------|----------------|-----------------------|--------------------|----------------------|-------------------|
| 2     | 0.214          | 178.5                 | 1.634              | 239                  | 0.763             |
| 3     | 0.198          | 158.3                 | 1.578              | 161                  | 0.625             |
| 4     | 0.187          | 144.5                 | 1.518              | 120                  | 0.700             |
| 5     | 0.182          | 134.5                 | 1.487              | 96                   | 0.602             |
| 6     | 0.188          | 125.7                 | 1.447              | 17                   | 0.630             |

The indices do not agree. Silhouette is around 0.2 for every k, which
means the groups are weak and overlap. Silhouette and Calinski-Harabasz
prefer k = 2. Davies-Bouldin keeps improving, but k = 6 creates a
cluster of only 17 firms. I chose k = 4 because it is the second most
stable solution, has a lower Davies-Bouldin than k = 2 and k = 3, has no
cluster smaller than 120 firms, and gives profiles that are easy to tell
apart. k = 2 would also be a reasonable choice.

Table 3. Cluster profiles (medians, original units)

| **Cluster** | **Size** | **Profile**                                                          | **Name**                 |
|-------------|----------|----------------------------------------------------------------------|--------------------------|
| 0           | 126      | \$7.5M, 1 round, 2 relationships, first funding at 4.5 years         | Thin late starters       |
| 1           | 192      | \$10.0M, 2 rounds, 8 relationships, first funding at 1.4 years       | Networked early starters |
| 2           | 152      | \$24.3M, 2 rounds, 4 investors per round, first funding at 4.2 years | Late syndicated raisers  |
| 3           | 120      | \$38.9M, 4 rounds, 13 relationships, 3 milestones                    | Serial-funded leaders    |

I checked stability in three ways. Twenty bootstrap refits gave a mean
adjusted Rand index of 0.70 (minimum 0.46). Ten runs with different
seeds and one start each gave 0.62 (minimum 0.30). Ward clustering
agreed with K-means at only 0.24. So the solution is fairly stable
inside K-means, but the borders change with the method.

I looked at the outcome only after the clusters were built. In the
training set 67.5% of the thin late starters closed, compared with 18.3%
of the serial-funded leaders. In the test set 88% of the 50 thin late
starters closed. Only 11 of the 185 test firms fell into cluster 3,
which again shows that young firms have short histories. The clusters
are useful for describing a portfolio, but they do not prove that
natural groups exist.

# 6. Method suitability, claim audit and conclusions

The dataset has no real unstructured text. The non-numeric columns are
names, places, dates and a category code, and running TF-IDF on them
would just repeat the dummy variables. A useful extension would be
company descriptions written at the first funding round, tested with the
same time split.

I also trained a small neural network (one hidden layer, 16 ReLU units)
on the same split. Validation loss was lowest at epoch 38 (0.472). By
epoch 200 training loss had dropped to 0.395 while validation loss rose
to 0.661, which is clear overfitting. At epoch 38 the network reached a
test ROC-AUC of 0.742, against 0.822 for logistic regression, so the
extra complexity did not help.

For the claim audit I gave an LLM the result table and asked for five
statements. One was supported: the test ROC-AUC of 0.822 against 0.500
for the baseline. One was contradicted, because the random forest did
not beat logistic regression on every metric. One was overstated: a
recall of 0.857 describes 84 firms founded in 2009 to 2013, not all
startups that will close. The last two were unsupported. One compared
accuracy across partitions with different base rates, and the other made
a causal claim about networks. All five sounded convincing.

My main finding is that pre-exit funding history can rank startups by
their risk of closing, and this ranking held on younger firms. Network
size is the strongest single signal. The four funding profiles are
moderately stable and their closure rates range from 18% to 68%. The
biggest limitation is exposure time: the counts grow while a firm
survives, so part of the signal may be how long the firm lived rather
than how good it was. The data also count every acquisition as a
success.

I would use the model as a first filter, not as a final verdict. The
next step is to rebuild all features at the same firm age, for example
two years after founding, and check whether the ROC-AUC stays at a
similar level.

# References

Kaggle. Startup Success Prediction (manishkc06).
https://www.kaggle.com/datasets/manishkc06/startup-success-prediction

Pedregosa, F. et al. (2011). Scikit-learn: Machine Learning in Python.
Journal of Machine Learning Research, 12, 2825-2830.

Course practice notebooks, Weeks 4 to 8.
