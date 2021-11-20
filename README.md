# Coupon Purchase Prediction
### CSCE 5300 - Fall 2021
This repository contains code and artifacts for Team "BruteForce" and our final project submission. 

Team members are listed in the separately submitted presentation deck to protect privacy. 

## About the project
The challenge is simple: using the data and the parameters of [this Kaggle competition](https://www.kaggle.com/c/coupon-purchase-prediction), with a goal of predicting which coupons a user is likely to purchase given some training data. Exploratory data analysis, data structures, assumptions, and more are covered extensively in our presentation. This report contains more information about how to execute the code. 

## Effort since the presentation
The highest score yet was achieved by augmenting 91 empty coupon lists with data from browsing history. This previously brought the mean average precision down by quite a lot, since the AP for those users was zero. 

## The data
The original data had several fields in Japanese. Because place names don't work so well with machine translation, it took a couple of iterations to get the translation right. 

### Japanese -> English translation
In the original [translation pipeline](), real translations are used, to some comic effect. In the [v2 pipeline](), a simple mapping is used instead, which resulted in far more readable place names, genres, and capsule texts. There are not many translations required; most are repeated, so it was worth the time to do this manually.

The translated data is checkpointed and temporarily stored on Google Drive (caution: it will be deleted at the end of the semester). The data can be easily downloaded with the `!gdown` commands found in the notebooks. 

### Preprocessing
Preprocessing (for everything but the decision tree / random forest approach) consisted of one-hot encoding categorical variables, replacing NaN values with 0 (as these were not necessary for training), and dropping unnecessary columns. The one-hot encoding blew out eight columns to over 140 features, which were then fed to logistic regression and cosine similarity algorithms. 

[TensorFlow Decision Forests]() and [Gradient Boosted Trees]() did not require any preprocessing. These approaches also required far less code, and were faster to iterate with. 

Cosine Similarity and Logistic Regression approaches employed feature selection to facilitate hybrid collaborative filtering - those coupons which match a user based on demographics or content.

## The Solutions
|Solution|Notebook URL|Mean Avg Precision @ 10|Notes|
|--------|------------|-----------------------|-----|
|Random Selection Baseline||**0.00051**|Randomly assign coupons|
|Logistic Regression #1 (Browsing Data)||<span style="color:red">*0.00028*</span>|Worse than baseline performance due to class imbalance.|
|Logistic Regression #2 (Browsing Data)||0.00061|Slightly better with class imbalance corrections (stratified splits, scaling, bias tuning, etc)|
|Cosine Similarity (Purchase History)||<span style="color:green">0.00269</span>|Great score, computationally expensive|
|TF Random Forest (Browsing Data) (also tried CART and untuned Boosted Trees with same result)||<span style="color:red">*0.00047*</span>|Easy configuration, needs tuning|
|TF Gradient Boosted Trees (Browsing Data)||<span style="color:green">0.00263</span>|Amazing results for imbalanced browsing data|
|**New since presentation:** Cosine Similarity backfilled with Gradient-Boosed Trees||<span style="color:green">**0.00283**</span>|Best results so far.|