# FastText Related Experiments #

## Summary ##
This folder contains files related to  
1. the __"Supervised FastText"__,  
2. the __"Unsupervised FastText + RidgeCV"__,  
3. the __"Unsupervised FastText + Random Forest"__  
experiments of RWP multi-label theme classification  

## Data ##
The experiments use the development set of RWP dataset. It contains 29912 records.  

## Norms and Symbols Commonly Used ##
### Experiment ID's ###
id1: Supervised FastText  
id2: Unsupervised FastText + RidgeCV  
id3: TFIDF + RidgeCV  
id4: Unsupervised FastText + Random Forest  
id5: TFIDF + Random Forest
  
This doc focuses on id1, 2, 4.  
  
file suffix:  
"id_" refers to results obtained from 10-fold cross validation (each time training on 90% dev set and test on 10% dev set). Results usually have dimension 29912.  
"id_x" or "id_x_10" refers to results obtained from training on 90% dev set and test on 10% dev set. Results usually have dimension 2991.  

### Abbrevations ###
dev: develoment set  
ft: FastText  
rf: Random Forest  
lbl: label  
mlb: multilabel

## Module 1: Flow of "Supervised FastText" ##
### Step 0: Define Input and Output ###
Input: Raw doc in training set, and their labels.  
Output: Labels and Proba of each label of each doc in test set, as well as the result for on-demand evaluation metrics.  

### Step 1: Get the Suprevised FastText model ###
The example command line to obtain the FastText model:
```
"~/fasttext supervised -input data.train.txt -output model_wn1_lr1_ws5 -wordNgrams 1 -lr 1.0 -ws 5"
```
In which, the __data.train.txt__ is formatted input file that contains training set documents and their associated labels. It is dynamically generated based on what record indices you want them to serve as the training set. Related files are:  
1. __"X_data_labeled.json"__: A list of string, dim 29912 * 1. It contains all the labeled development set data in FastText input format. The string format is basically "text + label". This labeled dataset is used for training, and contains combined components of (2) and (3). Please check FastText documentations for detailed format.  
2. __"X_data_unlabeled.json"__: A list of string, dim 29912 * 1. Compared to the above one, its strings only contains texts, but no labels. This unlabeled dataset is used for testing.  
3. __"y_rawlabels_dev.json"__: A list of string, dim 29912 * 1. Its strings only contains labels, but no texts. Used for testing.  
4. __"ids_raw_dev.json"__ or __"id_list_all.json"__: A list of string, dim 29912 * 1. It records the sequence of document id's in dev set. For example, the 0th position of (1) (2) (3) corresponds to record with 0th position of (4).  
5. __"train_index_ft_90.json"__: A list of integer, dim 26921 * 1. It identifies what are the training record indices in (1) (2) (3) (4). The training set is 90% of dev set, and the indices range is 0 - 29911.  
6. __"test_index_ft_10.json"__: A list of integer, dim 2991 * 1. It identifies what are the test record indices in (1) (2) (3) (4). The test set is 10% of dev set, and the indices range is 0 - 29911.  
  
The __"data.train.txt"__ in command line is obtained by picking out (1) with indices in (5).  
The __"model_wn1_lr1_ws5"__ in command line is the name of output model. The model will be in the format of __.bin__ and __.vec__. Please check FastText documentation for details.  
  
The specified parameters are obtained from grid search.  

### Step 2: Get Predicted Labels Based on Trained Model ###
```
"~/fasttext predict-prob model_wn1_lr1_ws5.bin test.texts.txt 19 > lbl_wn1_lr1_ws5_correct.txt"  
```
In which, the __"test.texts.txt"__ is formatted output file that contains unlabeled test set documents. It is dynamically generated based on what record indices you want them to serve as the test set. It is obtained by picking out (2) with indices in (6).  
The __"19"__ is total number of labels.  
The __"lbl_wn1_lr1_ws5_correct.txt__" is FastText output result. It contains test set documents' labels and their proba.  

### Step 3: Process the Raw Output into Organized Python Data Structures ###
After processing, the result will look like the file __"result_lbl_n_prob_best_FastText_sup_id1x_10.json"__. One typical line:  
```
[[["Water_Sanitation_Hygiene", 0.208984], ["Shelter_and_Non-Food_Items", 0.183594]...]] 
```
It is a list of list of list. The outer list has dim 2991 with each item as a record. The inner list has dim 19 with each item as a binary list of [label, proba]. Sorted by proba from high to low.  

### Step 4: Get Evaluations ###
Put what obtained from Step 3 and the truth (Step 1 item (3) with indices in (6)) as the first two arguments of multilabel evaluator (the __MLB_Evaluation__ class or the __evaluation__ method in it), then you can get on-demand evaluation metric results.  
  
The file __"result_ndcg_best_FastText_sup_id1x_10.json"__ contains NDCG result for each record of the 10% test set, dim 2991 * 1.  


## Module 2: Flow of "Unsupervised FastText + Classifier" ##
### Step 1: Get the Unsupervised FastText Model ###
The example command line to obtain the FastText model:  
```
~/fasttext skipgram -input data.unsup_all.txt -output model -wordNgrams 3 -lr 0.05 -ws 8"
```
In which, the __"data.unsup_all.txt"__ is Module 1 - Step 1 - (2) in FastText input format.  
  
The __"model"__ in command line is the name of output model. The model will be in the format of __.bin__ and __.vec__. Please check FastText documentation for details.  
  
The specified parameters are obtained from grid search.  

### Step 2: Get the Embeddings ###
The example command line to obtain the embeddings of all docs:  
```
~/fasttext print-sentence-vectors model.bin < data.unsup_all.txt > raw_wn3_lr05_ws8.txt")
```
It means applying the model on texts and putting results into a file.  

### Step 3: Process the Embeddings into Train & Test ###
Process the raw txt format embeddings into Python data structures. Then prepare training and test set based on Module 1 - Step 1 - (5) and (6).  

### Step 4: Apply ML models ###
Apply RidgeCV or Random Forest Models on prepared data and get predicted data that contain proba.   

### Step 5: Get Evaluations ###
Put what obtained from Step 4 and the truth as arguments of multilabel evaluator (the __MLB_Evaluation__ class or the __evaluation__ method in it), then you can get on-demand evaluation metric results.  

## JSON Result files ##
### NDCG files, each number is NDCG for a record ###
result_ndcg_best_FastText_sup_id1x_10  
result_ndcg_best_FastText_sup_id1  
  
result_ndcg_best_FastText_RidgeCV_id2x_10  
result_ndcg_best_FastText_RidgeCV_id2  
  
result_ndcg_best_FastText_rf_id4x_10  
result_ndcg_best_FastText_rf_id4  

### Proba files, each list is [lbl,proba] for a record ###
result_lbl_n_prob_best_FastText_sup_id1x_10  
result_lbl_n_prob_best_FastText_sup_id1  

result_lbl_n_prob_best_FastText_RidgeCV_id2x_10  
result_lbl_n_prob_best_FastText_RidgeCV_id2  

result_lbl_n_prob_best_FastText_rf_id4x_10  
result_lbl_n_prob_best_FastText_rf_id4  



















