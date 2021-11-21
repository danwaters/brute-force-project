# Coupon Purchase Prediction
### CSCE 5300.002 - Fall 2021

This repository ([github.com/danwaters/brute-force-project](https://www.github.com/danwaters/brute-force-project)) contains code and artifacts for Team "BruteForce" and our final project submission. This README serves as the project report, as well as the instructions and links for reproducing our work.

Looking at the GitHub repository is encouraged, because GitHub has a very good .ipynb viewer. 

---
**NOTE** 

It is very important to run the notebooks in Google Colab, as the data files are hosted in Google Drive and acquired via Colab's built-in `!gdown` command.

---

Team members are listed in the separately submitted presentation deck to protect privacy. 

## About the project
The challenge is simple: using the data and the parameters of [this Kaggle competition](https://www.kaggle.com/c/coupon-purchase-prediction), with a goal of predicting which coupons a user is likely to purchase given some training data. 

**Exploratory data analysis, data structures, assumptions, and more are covered extensively in our [presentation](https://github.com/danwaters/brute-force-project/blob/main/BruteForce%20-%20Coupon%20Purchase%20Prediction.pdf).** This report contains more high level information and instructions to execute the code. 

## The team
A list of team members, tasks, and assignees is available in our [project management sheet](https://github.com/danwaters/brute-force-project/blob/main/BruteForce%20-%20Project%20Management.xlsx).

## Effort since the presentation
The highest score yet was achieved by augmenting 91 empty coupon lists with the Gradient Boosted Trees predictions for those users. This previously brought the mean average precision down by quite a lot, since the AP for those users was zero. 

## The data
The data is provided by Recruit Ponpare. The tables, relationships, and field descriptions are described in detail [here](https://www.kaggle.com/c/coupon-purchase-prediction/data). Exploratory data analysis is described in our [presentation document](https://github.com/danwaters/brute-force-project/blob/main/BruteForce%20-%20Coupon%20Purchase%20Prediction.pdf). 

### Japanese -> English translation
The original data had several fields in Japanese. Since nobody on our team speaks Japanese, it presented a problem for understanding the data and EDA tasks. Because place names don't work so well with machine translation, it took a couple of iterations to get the translation right. 

In the original [translation pipeline notebook](https://github.com/danwaters/brute-force-project/blob/main/notebooks/ipynb/BruteForce%20-%20Coupon%20Translation%20Engine.ipynb), two different translation libraries are used ([txtai](https://github.com/neuml/txtai) and [pykakasi](https://github.com/miurahr/pykakasi)) are used, to some comic effect. In the [v2 pipeline notebook](https://github.com/danwaters/brute-force-project/blob/main/notebooks/ipynb/BruteForce%20-%20Coupon%20Translation%20Engine%20v2.ipynb), a simple mapping is used instead. While tedious, it resulted in far more readable and accurate place names, genres, and capsule texts. There are not many translations required; most are repeated, so it was worth the time to do this manually.

### Downloading the data
The translated data is temporarily stored on Google Drive (caution: it will be deleted at the end of the semester after grades are put in). The data can be easily downloaded with the `!gdown` commands found in the notebooks. 

### Preprocessing
Preprocessing (for everything but the decision tree / random forest approach) consisted of one-hot encoding categorical variables, replacing NaN values with 0 (as these were not necessary for training), uniting user data with coupon data, and dropping unnecessary columns. The one-hot encoding expanded eight columns to over 140 features, which were then fed to logistic regression and cosine similarity algorithms. 

[TensorFlow Decision Forests and Gradient Boosted Trees](https://github.com/danwaters/brute-force-project/blob/main/notebooks/ipynb/BruteForce%20-%20Gradient%20Boosted%20Trees.ipynb) thankfully did not require any preprocessing other than feature selection. These approaches also required far less code, and were faster to iterate with. The [TFDF blog post](https://blog.tensorflow.org/2021/05/introducing-tensorflow-decision-forests.html) is a great place to start to understand this great library.

Cosine Similarity and Logistic Regression approaches employed feature selection to facilitate hybrid collaborative filtering - those coupons which match a user based on demographics or content.

## Evaluation
The Kaggle competition submission is a .csv file. Each row contains a user ID hash and a space-delimited list of 10 coupons. Kaggle evaluates these submissions against a held-out test data set containing coupons that a user actually purchased during the test period. 

The evaluation criteria is Mean Average Precision (k=10), taking the average precision of the 10 coupons for each user, averaged across all ~22K users.

## The solutions
First, a baseline model of random selection was created with the following code. You can also find this code about halfway down in the [Coupon EDA notebook](https://github.com/danwaters/brute-force-project/blob/main/notebooks/ipynb/BruteForce%20-%20Individual%20Coupon%20EDA.ipynb).
```
# create random baseline
user_list = df_users['USER_ID_hash']

rows = []
for u in user_list:
  coupon_list = df_c_list_test.sample(n=10, replace=False)['COUPON_ID_hash']
  coupon_list_str = ' '.join(coupon_list)
  
  row = {'USER_ID_hash': u, 'PURCHASED_COUPONS': coupon_list_str}
  rows.append(row)
  
df_pred = pd.DataFrame.from_dict(rows)
df_pred.to_csv('sample_submission.csv', header=True, index=False)
```
|Solution|Notebook URL|Mean Avg Precision @ 10|Notes|
|--------|------------|-----------------------|-----|
|Random Selection Baseline|[ipynb](https://github.com/danwaters/brute-force-project/blob/main/notebooks/ipynb/BruteForce%20-%20Individual%20Coupon%20EDA.ipynb), [pdf](https://github.com/danwaters/brute-force-project/blob/main/notebooks/pdf/BruteForce%20-%20Individual%20Coupon%20EDA.pdf)|**0.00051**|Randomly assign coupons|
|Logistic Regression #1 (Browsing Data)|Evolved into #2. [ipynb](https://github.com/danwaters/brute-force-project/blob/main/notebooks/ipynb/BruteForce%20-%20Logistic%20Regression%20and%20Hybrid%20Collaborative%20Filtering.ipynb), [pdf](https://github.com/danwaters/brute-force-project/blob/main/notebooks/pdf/BruteForce%20-%20Logistic%20Regression%20and%20Hybrid%20Collaborative%20Filtering.pdf)|<span style="color:red">*0.00028*</span>|Worse than baseline performance due to class imbalance.|
|Logistic Regression #2 (Browsing Data)|[ipynb](https://github.com/danwaters/brute-force-project/blob/main/notebooks/ipynb/BruteForce%20-%20Logistic%20Regression%20and%20Hybrid%20Collaborative%20Filtering.ipynb), [pdf](https://github.com/danwaters/brute-force-project/blob/main/notebooks/pdf/BruteForce%20-%20Logistic%20Regression%20and%20Hybrid%20Collaborative%20Filtering.pdf)|0.00061|Slightly better with class imbalance corrections (stratified splits, scaling, bias tuning, etc)|
|Cosine Similarity (Purchase History)|[ipynb](https://github.com/danwaters/brute-force-project/blob/main/notebooks/ipynb/BruteForce%20-%20Cosine%20Similarity%20between%20Purchased%20Coupons.ipynb), [pdf](https://github.com/danwaters/brute-force-project/blob/main/notebooks/pdf/BruteForce%20-%20Cosine%20Similarity%20between%20Purchased%20Coupons.pdf)|<span style="color:green">0.00269</span>|Great score, computationally expensive|
|TF Random Forest (Browsing Data) (also tried CART and untuned Boosted Trees with same result)|Very minor code changes from the one below; not included here|<span style="color:red">*0.00047*</span>|Easy configuration, needs tuning|
|TF Gradient Boosted Trees (Browsing Data)|[ipynb](https://github.com/danwaters/brute-force-project/blob/main/notebooks/ipynb/BruteForce%20-%20Gradient%20Boosted%20Trees.ipynb), [pdf](https://github.com/danwaters/brute-force-project/blob/main/notebooks/pdf/BruteForce%20-%20Gradient%20Boosted%20Trees.pdf)|<span style="color:green">0.00263</span>|Amazing results for imbalanced browsing data|
|**New since presentation:** Cosine Similarity backfilled with Gradient-Boosed Trees|[ipynb](https://github.com/danwaters/brute-force-project/blob/main/notebooks/ipynb/BruteForce%20-%20Merge%20Cosine%20with%20GBT.ipynb), [pdf](https://github.com/danwaters/brute-force-project/blob/main/notebooks/pdf/BruteForce%20-%20Merge%20Cosine%20with%20GBT.pdf)|<span style="color:green">**0.00283**</span>|Best results so far.|

## Our Conclusions
* Browsing history has less-than-expected correlation with purchasing habits. Class imbalance did not help.
* Past purchasing habits are better indicators of future purchases.
* Hybrid collaborative filtering measured using cosine similarity on purchase data far outperforms logistic regression (as a binary classifier) on browsing data, again, mostly due to class imbalance.

HOWEVER:
* TFDF's Gradient Boosted Trees algorithm, with appropriate hyperparameter tuning, performs at the same level as cosine similarity, though they are trained on browsing data. This merits further investigation.

## Resources
* [Our Presentation](https://github.com/danwaters/brute-force-project/blob/main/BruteForce%20-%20Coupon%20Purchase%20Prediction.pdf): Includes more detailed information about exploratory data analysis, assumptions, future enhancements etc. in a well-organized flow. 
* [The Kaggle Competition](https://www.kaggle.com/c/coupon-purchase-prediction): to learn more about the target problem and the data which was provided. 
* Bonus: [Interview with competition winner Halla Yang](https://machinelearningmastery.in/2020/08/10/recruit-coupon-purchase-winners-interview-2nd-place-halla-yang-by-kaggle-team-kaggle-blog/). We discovered this interview *after* doing our project (unfortunately), but we learned that we are on the right track with gradient boosting. Halla did far more work on the feature selection than we did. Halla incorporated windowing, locality, and more, with a focus on time series problems, which is not how we approached the problem. It is very interesting to see the process of a competitions grandmaster approaching this exact problem.
