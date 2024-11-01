# News Classification with FastText
This repository provides code and instructions for training a news classification model using FastText. The dataset used is the News Category Dataset, and the model classifies news headlines into various categories.
## Table of Contents
- Dataset Preparation
- Environment Setup
- Data Preprocessing
- Training the Model
- Evaluating the Model
- Hyperparameter tuning

## Dataset Preparation

1. **Download the Dataset**

   - Obtain the `News_Category_Dataset_v3.json` dataset from [Kaggle](https://www.kaggle.com/datasets/rmisra/news-category-dataset).

2. **Upload the Dataset**

   - Upload `News_Category_Dataset_v3.json` to your working directory in GitHub Codespaces or your local environment.
     
## Environment Setup
### Step 1: Update Packages and Install Dependencies

```bash
sudo apt-get update
sudo apt-get install -y build-essential cmake jq
```
### Step 2: Clone the FastText Repository
```bash
git clone https://github.com/facebookresearch/fastText.git
```
### Step 3: Build FastText

```bash
cd fastText
make
cd ..
```
## Data Preprocessing

### Step 1: Convert JSON to FastText Format

```bash
jq -r '. | "__label__" + (.category | gsub(" "; "_")) + " " + .headline' News_Category_Dataset_v3.json > data.txt
```
### Step 2: Split Data into Training and Test Sets

```bash
total_lines=$(wc -l < data.txt)
train_lines=$((total_lines * 8 / 10))
head -n $train_lines data.txt > train.txt
tail -n +$((train_lines + 1)) data.txt > test.txt
```
### Step 3: Preprocess the Data

```bash
awk '{
    label = $1;
    $1 = "";
    sub(/^[ \t]+/, "", $0);
    text = tolower($0);
    gsub(/[[:punct:]]+/, "", text);
    print label " " text;
}' train.txt > preprocessed_train.txt
```

```bash
awk '{
    label = $1;
    $1 = "";
    sub(/^[ \t]+/, "", $0);
    text = tolower($0);
    gsub(/[[:punct:]]+/, "", text);
    print label " " text;
}' test.txt > preprocessed_test.txt
```
## Training the Model

```bash
./fastText/fasttext supervised -input preprocessed_train.txt -output model_news 
```

## Evaluating the Model

```bash
./fastText/fasttext test model_news.bin preprocessed_test.txt
```

```bash
N       41906
P@1     0.46
R@1     0.46
```
## Hyperparameter tuning
### Epoch

```bash
./fasttext supervised -input preprocessed_train.txt -output model_news -epoch 10
Read 1M words
Number of words:  60183
Number of labels: 42
Progress: 100.0% words/sec/thread:  216909 lr:  0.000000 avg.loss:  1.162408 ETA:   0h 0m 0s
./fasttext test model_news.bin preprocessed_test.txt
N       41906
P@1     0.557
R@1     0.557
```
### Learning Rate

```bash
./fasttext supervised -input preprocessed_train.txt -output model_news -epoch 10 -lr 0.5
Read 1M words
Number of words:  60183
Number of labels: 42
Progress: 100.0% words/sec/thread:  220232 lr:  0.000000 avg.loss:  1.014158 ETA:   0h 0m 0s
./fasttext test model_news.bin preprocessed_test.txt
N       41906
P@1     0.516
R@1     0.516
```

```bash
./fasttext supervised -input preprocessed_train.txt -output model_news -epoch 10 -lr 1.0
Read 1M words
Number of words:  60183
Number of labels: 42
Progress: 100.0% words/sec/thread:  217262 lr:  0.000000 avg.loss:  2.029095 ETA:   0h 0m 0s
./fasttext test model_news.bin preprocessed_test.txt
N       41906
P@1     0.532
R@1     0.532
```

### Word N-grams

```bash
./fasttext supervised -input preprocessed_train.txt -output model_news -epoch 10 -lr 1.0 -wordNgrams 2
Read 1M words
Number of words:  60183
Number of labels: 42
Progress: 100.0% words/sec/thread:  176563 lr:  0.000000 avg.loss:  0.374077 ETA:   0h 0m 0s
./fasttext test model_news.bin preprocessed_test.txt
N       41906
P@1     0.535
R@1     0.535
```
### Multilabel Classification

```bash
./fasttext supervised -input preprocessed_train.txt -output model_news -epoch 10 -lr 1.0 -wordNgrams 2 -loss one-vs-all
Read 1M words
Number of words:  60183
Number of labels: 42
Progress: 100.0% words/sec/thread:  166258 lr:  0.000000 avg.loss:  0.752180 ETA:   0h 0m 0s
./fasttext test model_news.bin preprocessed_test.txt
N       41906
P@1     0.521
R@1     0.521
```






