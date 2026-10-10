# Data Taming in R

Coursework from **Data Taming**, part of my Master of Data Science at the University of Adelaide (Trimester 1, 2024). There are three assignments, all in R and the tidyverse. They move from cleaning messy data, to linear regression, to a logistic regression classifier evaluated on held-out data.

| # | Topic | Dataset | Techniques |
|---|-------|---------|------------|
| 1 | [Data tidying](assignment-1-data-tidying) | Archery practice sessions | Wide-to-long reshaping, string parsing, type conversion, feature creation |
| 2 | [Linear regression](assignment-2-linear-regression) | Gadget sales by city | Feature engineering, skewness, Box-Cox transformation, OLS, assumption checks, prediction intervals |
| 3 | [Logistic regression](assignment-3-logistic-regression) | Customer survey, 50,000 respondents | Train/test split, dummy encoding, interaction terms, backward stepwise selection, confusion matrix, ROC/AUC |

**Tools:** R, tidyverse (dplyr, tidyr, ggplot2, forcats, stringr, lubridate), tidymodels, rsample, modelr, car, caret, moments, inspectdf, R Markdown

---

## Assignment 1: Tidying archery data

The dataset records archers' practice sessions in wide format. Each session column holds a free-text result such as *"Target hit 7 out of 10"*.

- Loaded the data while treating `XX` as missing, then took a reproducible random sample of 100 archers
- Reshaped the six session columns from wide to long with `gather()`
- Extracted numeric `shots` and `hits` from each free-text result with regex (`stringr::str_match`), and reduced the session label to an ordered session number
- Converted variables to "tame data" conventions: Yes/No recoded to TRUE/FALSE, counts to integers, start dates parsed with `lubridate`, session as an ordered factor
- Engineered `days_experience` and `accuracy` (hits ÷ shots)
- Filtered to experienced archers and plotted accuracy against experience

Files: [`a1_data_tidying.Rmd`](assignment-1-data-tidying/a1_data_tidying.Rmd) (source), [`a1_report.pdf`](assignment-1-data-tidying/a1_report.pdf) (knitted output)

## Assignment 2: Does advertising drive sales? (linear regression)

A company wants to know how advertising spend relates to sales of its "Gadget" across cities, and what sales to expect at different budgets.

- Created `sales_pct` (the percentage of a city's population that bought the gadget) and `adv_exp_pp` (advertising spend per person)
- Took a random sample of cities, then explored the relationship with a scatter plot and summary statistics
- Found `sales_pct` was skewed and the relationship curved, so applied a **Box-Cox transformation** to the response
- Fitted a linear model on the transformed scale, with R² ≈ 0.93
- Checked the four linear-model assumptions (linearity, constant variance, normality, independence) using residual, scale-location and Q-Q plots, and discussed a plausible violation of independence: buyers move between cities
- Back-transformed **90% prediction intervals** for low, medium ($3.14/person) and high ($6/person) advertising spend

Files: [`a2_report.pdf`](assignment-2-linear-regression/a2_report.pdf)

## Assignment 3: Who will recommend the product? (logistic regression)

A survey of 50,000 customers asks whether they would recommend "Gadget 2". The predictors are age, socio-economic status (SES), brand awareness, social media use, multiple purchases, and whether a gadget malfunctioned.

- Tamed the data: snake_case names, logical flags, `recommend` as a yes/no factor, SES as a factor. Each variable was classified as nominal, ordinal, discrete or continuous, with justification
- Split the data into training and testing sets (`rsample`, seeded)
- Fitted a main-effects logistic model with `tidymodels` (glm engine), then used `model_matrix()` to show how SES is dummy-coded (reference level: `high`)
- Fitted a model with all second-order interactions and used `car::Anova()` to keep the terms significant at the 99.9% level: **age × malfunction** and **multi-purchase × SES**
- Ran **backward stepwise selection** to the 95% level, dropping `company_aware` and then `social_media`
- Interpreted the final model's log-odds, why its lines are not parallel, and plausible real-world explanations for each interaction

**Results on the test set (10,000 respondents)**

| Metric | Value |
|--------|-------|
| AUC | **0.883** |
| Specificity | 0.928 |
| Sensitivity | 0.656 |

The model is good at identifying customers who won't recommend the product, and weaker at catching those who will.

Files: [`a3_report.pdf`](assignment-3-logistic-regression/a3_report.pdf), [`data/survey_2.csv`](assignment-3-logistic-regression/data/survey_2.csv)

---

## Repository structure

```
data-taming/
├── assignment-1-data-tidying/
│   ├── a1_data_tidying.Rmd
│   ├── a1_report.pdf
│   └── data/                  # archery_0.csv (not included, see data/README.md)
├── assignment-2-linear-regression/
│   ├── a2_report.pdf
│   └── data/                  # gadget_1.csv (not included, see data/README.md)
└── assignment-3-logistic-regression/
    ├── a3_report.pdf
    └── data/
        └── survey_2.csv
```

## Running the code

Open `assignment-1-data-tidying/a1_data_tidying.Rmd` in RStudio, place `archery_0.csv` in that folder's `data/` directory, and knit. The required packages are listed in the setup chunk:

```r
install.packages(c("tidyverse", "inspectdf", "lubridate", "tinytex"))
```

Assignments 2 and 3 are included as their knitted reports, which show the R code alongside its output.

## Note

Each student received a dataset variant chosen by their student number, which is why the code computes a file number from it. This repository contains my own submitted work only.
