# Startup outcomes: acquired or closed?

AI and Big Data Analytics, Weeks 4–8 assessment.

## Source

Kaggle, *Startup Success Prediction* (`manishkc06`), derived from Crunchbase:
https://www.kaggle.com/datasets/manishkc06/startup-success-prediction

## Files

| File | Content |
|---|---|
| `startup_success_analysis.ipynb` | Main notebook. All cells executed with outputs, figures and interpretations. |
| `data/startup_data_original.csv` | Original file as downloaded (923 × 49). |
| `data/startup_data_analytical.csv` | Final analytical dataset written by the notebook (922 × 46, with `split` column). |
| `data/data_dictionary.md` | Meaning and use of every variable. |
| `report/Analytical_Report.docx` (+ `.md`) | Analytical report, about 2,000 words (Times New Roman 12, 1.5 spacing). |
| `report/Completed_Working_Form.docx` | Student working form with every table filled. |
| `figures/` | Figures saved by the notebook. |
| `AI_use_statement.md` | Generative AI use statement. |

## Software

Python 3.11+ with pandas, numpy, scikit-learn (≥ 1.3), matplotlib, seaborn and statsmodels. Runs in Google Colab without extra installs.

## Execution order

1. Keep `data/startup_data_original.csv` next to the notebook, in a `data/` folder. In Colab, the first cell offers an upload if the file is missing.
2. Run `startup_success_analysis.ipynb` from top to bottom (Runtime → Run all). Runtime is under two minutes on CPU.
3. The notebook writes `data/startup_data_analytical.csv` and the PNG files in `figures/`.

## Reproducibility

- Random seed: **42** (`RANDOM_STATE`) for CV folds, models, bootstrap and PCA.
- Split: temporal by `founded_at`. Train 64%, validation 16%, test 20%.
- All imputers, scalers, PCA, clustering and models are fitted on training or development data only. The test set is evaluated once.
