<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:FF9900,100:232F3E&height=160&section=header&text=Amazon%20Book%20Review%20Analytics&fontSize=30&fontColor=ffffff&fontAlignY=50&desc=Big%20Data%20Pipeline%20%7C%20Sentiment%20Analysis%20%7C%20Hadoop%20%2B%20PySpark%20%7C%203.3M%2B%20Reviews&descAlignY=72&descSize=13" width="100%"/>

</div>

<div align="center">

![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)
![Hadoop](https://img.shields.io/badge/Hadoop_HDFS-FFCE00?style=for-the-badge&logo=apachehadoop&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-0ea5e9?style=for-the-badge&logo=docker&logoColor=white)
![Kaggle](https://img.shields.io/badge/Dataset-Kaggle-20BEFF?style=for-the-badge&logo=kaggle&logoColor=white)

</div>

---

## Overview

A full-scale big data pipeline for processing and analyzing the Amazon Books Reviews dataset using Hadoop and PySpark. The pipeline covers the entire ML lifecycle — from distributed data ingestion and EDA to feature engineering, model training, hyperparameter tuning, and writing results back to HDFS.

Three core prediction tasks are addressed: **sentiment classification**, **star rating prediction**, and **review helpfulness prediction** — across a dataset of 3.3M+ user-generated reviews spanning 209,456 unique book titles.

---

## Dataset

| Attribute | Detail |
|-----------|--------|
| Source | Kaggle — Amazon Books Reviews |
| Files | `books.csv` (metadata) + `rating.csv` (user ratings) |
| Final Merged Records | 3,320,245 rows · 19 columns |
| Unique Book Titles | 209,456 |
| Features | Review text, star rating, helpfulness votes, price, publisher, categories, review timestamp |

---

## Pipeline

```
books.csv + rating.csv  (Kaggle)
          │
          ▼
   HDFS Ingestion
   (DataIngestion.java → JAR → Hadoop Filesystem API)
          │
          ▼
   PySpark EDA
   (Schema inference, missing value analysis,
    summary stats, distribution plots)
          │
          ▼
   Data Cleaning & Preprocessing
   ├── Missing value imputation (mean / placeholder defaults)
   ├── Outlier handling (IQR for ratingsCount, price capping at $500)
   ├── Data type conversion (Unix timestamp → datetime, cast numerics)
   └── Merge books.csv + rating.csv → single cleaned Parquet
          │
          ▼
   Feature Engineering
   ├── Text: Tokenizer → StopWordsRemover → HashingTF → IDF (TF-IDF)
   ├── Numeric: Price, ratingsCount, review/score (Imputer)
   └── VectorAssembler → unified feature vector
          │
          ▼
   Spark ML — Model Training (80/20 train/test split)
   ├── Logistic Regression
   ├── Support Vector Machine (LinearSVC)
   ├── Random Forest Classifier (100 trees, maxDepth=10)
   └── Multinomial Logistic Regression
          │
          ▼
   Hyperparameter Tuning
   (CrossValidator + ParamGridBuilder, 3–5 fold CV)
          │
          ▼
   Results written back to HDFS
   (model pipeline, predictions, metrics → Parquet)
```

---

## Model Results

### Before Hyperparameter Tuning

| Model | Accuracy | Precision | Recall | F1-Score |
|-------|----------|-----------|--------|----------|
| Logistic Regression | 86.92% | 87.74% | 86.92% | 87.18% |
| Support Vector Machine | 81.12% | 84.74% | 81.12% | 73.88% |
| Random Forest Classifier | 60.88% | 60.58% | 60.88% | 46.91% |
| **Multinomial Logistic Regression** | **86.92%** | **87.74%** | **86.92%** | **87.18%** |

### After Hyperparameter Tuning

| Model | Accuracy | F1-Score | Tuning Strategy |
|-------|----------|----------|-----------------|
| **Logistic Regression (Tuned)** | **95.01%** | **93.45%** | regParam · elasticNetParam · 3-fold CV |
| SVM (Tuned) | 88.64% | 86.81% | regParam [0.8–1.5] · 5-fold CV |
| Random Forest (Tuned) | 60.86% | 46.97% | numTrees · maxDepth · 5-fold CV |

**Best model: Tuned Logistic Regression — 95.01% accuracy**, demonstrating that linear models outperform tree-based ensembles on this TF-IDF + numeric feature space.

---

## Key Findings

**Feature Importance**
- `review/score_imputed` was the dominant feature in Random Forest — indicating strong reliance on prior rating signals
- Logistic Regression highlighted stylistic words (`setting`, `prose`, `comes`), while Random Forest emphasized sentiment-charged words (`waste`, `good`, `worst`)

**Misclassification Analysis**
- 8,759 misclassified samples identified from the Multinomial Logistic Regression model
- Most errors occurred on borderline reviews with sarcasm, mixed sentiment, or context-dependent language
- Largest misclassifications involved scores predicted 3 points off from the true label (e.g., true: 4.0 → predicted: 1.0)

**Sentiment by Book Category**
- Fiction and Mystery dominate review volume; Fiction leads in total positive reviews
- Positive reviews overwhelmingly outweigh neutral and negative across all 20 top categories
- Romance, Sci-Fi, and Children's genres show similarly high positive sentiment rates

**Temporal Trends (1995–2015)**
- Review volume grew sharply post-late 1990s, with a significant spike around 2013 — coinciding with the rise of Kindle and self-publishing
- Positive sentiment dominated throughout all periods; neutral and negative reviews remained consistently low

---

## EDA Highlights

- Rating distribution is heavily right-skewed — most reviews cluster at 4–5 stars, introducing class imbalance considerations for modeling
- Most book prices fall below $20; extreme outliers capped at $500 during preprocessing
- Average ratings per book: **56.32** with a high standard deviation of **329.21** — a small number of titles are extremely popular while most receive few ratings
- Review helpfulness ratio is bimodal — users tend to rate reviews as either very helpful (1.0) or not helpful at all (~0)
- **Fiction** is the most common book category; **Science** the least

---

## Repository Structure

```
amazon-bigdata-review-analytics/
├── main.ipynb      # Full PySpark pipeline — EDA, preprocessing, ML, evaluation
├── Report.pdf      # Detailed project report (Phase 1 + Phase 2)
└── code.pdf        # Code walkthrough and implementation details
```

---

## Tech Stack

![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=flat-square&logo=apachespark&logoColor=white)
![Hadoop](https://img.shields.io/badge/Hadoop_HDFS-FFCE00?style=flat-square&logo=apachehadoop&logoColor=black)
![Docker](https://img.shields.io/badge/Docker-0ea5e9?style=flat-square&logo=docker&logoColor=white)
![Spark MLlib](https://img.shields.io/badge/Spark_MLlib-E25A1C?style=flat-square&logoColor=white)
![TF-IDF](https://img.shields.io/badge/TF--IDF-7c3aed?style=flat-square&logoColor=white)
![Java](https://img.shields.io/badge/Java-ED8B00?style=flat-square&logo=openjdk&logoColor=white)
![Python](https://img.shields.io/badge/Python-0ea5e9?style=flat-square&logo=python&logoColor=white)

---

<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:232F3E,100:FF9900&height=80&section=footer" width="100%"/>

</div>
