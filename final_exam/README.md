# Final Exam, AI and Big Data Analytics: Variant B, Bike Sharing

Aktan Seraliyev, 255522, DBAIT-2501

## Files

| File | What it is |
|---|---|
| `final_exam_bike_sharing.ipynb` | Main notebook (code only), all cells run, outputs visible |
| `Final_Exam_AIBD_Aktan_Seraliyev.docx` | Completed examination sheet |
| `data/hour.csv` | Original UCI file (17,379 rows) |
| `data/Readme.txt` | Original UCI description |
| `data/hour_clean.csv` | Clean analytical table written by the notebook (17,379 rows, 19 columns) |
| `figures/` | Figures saved by the notebook |
| `AI_use_statement.md` | AI use statement |

## Source

UCI Machine Learning Repository, Bike Sharing Dataset (Fanaee-T and Gama, 2013).
https://archive.ics.uci.edu/dataset/275/bike+sharing+dataset

## Software

Python 3.13, pandas 3.0.5, numpy 2.5.3, scikit-learn 1.9.1, scipy 1.18.1, statsmodels 0.15.0, matplotlib 3.11.2, seaborn 0.13.2.
It also runs in Google Colab with its default versions.

## How to run

1. Keep `hour.csv` in a `data/` folder next to the notebook. In Colab, the loading cell asks you to upload it if the file is missing.
2. Run all cells from top to bottom. On a normal laptop or in Colab this takes about 1 minute.
3. The notebook writes `data/hour_clean.csv` and the PNG files in `figures/`.

## Reproducibility

- Random seed: 42 (`RANDOM_STATE`), used for models, bootstrap, permutation importance and sampling.
- Split by date: train 2011-01-01 to 2012-06-30, validation 2012-07-01 to 2012-09-30, test 2012-10-01 to 2012-12-31.
- All preprocessing is fitted on training data only. The test quarter is used once.
