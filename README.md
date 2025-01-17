# Overview
This repository holds the source code and raw experimental results of our ICDE 2020 paper "Active Model Selection for Positive Unlabeled Time Series Classification". This repository has the following four folders.

Code: all our source code. We will later show how to use it.

Data: a sample UCR dataset (GunPointMaleVersusFemale), as well as the ECG data (201_A_pyramid and 205_A_pyramid) we used in the case study in arrhythmia detection. We will elaborate on the datasets later.
    
Results: This folder consists of all our raw results. There are three types of files in this folder:
- fscore_0.xx.xlsx: raw average F1-scores on the UCR datasets, which are used to plot the critical difference diagrams in our paper. The "0.xx" in the file names indicate the percentatage of queried U examples. For instance, "fscore_0.1.xlsx" consists of the raw results of all model selection methods on all datasets with 10% of the U examples queried. Note that the critical difference diagrams are actually plotted using "one minus F1-score" values so as to keep the better performing methods on the right of the diagrams. These values are equal to Van Rijsbergen's effectiveness measure with alpha = 0.5. See https://en.wikipedia.org/wiki/F1_score for more.
- fscore_MITDB.xlsx: raw average F1-scores on the 201_A_pyramid and 205_A_pyramid datasets (two spreadsheets in a single file) with the number of queried U examples ranging from 1 : |U|.
- responseTimes.xlsx: average and maximum user interaction response time of all model selection methods on UCR datasets.
- fscores_non_AL.xlsx: raw average F1-scores of the 7 non-AL based baselines on the UCR datasets.
- fscores_PUSh_ST-1NN.xlsx: raw average F1-scores of the ST-1NN and PUSh on the UCR datasets. For ST-1NN, in each run on each dataset we use the result of the best of the 1271 models on that run. For PUSh, due to limited time and the fact that PUSh is too memory-inefficient to run on certain large datasets, only some results are available.
- All_Experimental_Figures.pdf: all experimental figures, including those in our revised paper, and additional ones that are only mentioned in the paper.

Seeds: seed files used by the code, including the initial PL sets used to run the candidate ST-1NN models, and sampling orders for model selection methods using the random sampling (Rand) strategy.

# How to use the code
Please take the following steps to obtain PUTSC model selection results.
1. Use GPU-DTW.cu to calculate the DTW distances.
2. Use ST1NN.cpp to run the candidate ST-1NN models
3. Use modelSelection.cpp to run the model selection methods.

Input parameters of GPU-DTW.cu:
- datasetName: name of dataset, for example "GunPointMaleVersusFemale"
- numTrain: number of examples in the dataset, for example "135"
- tsLen: time series length, for example "150"
- deviceId: the device ID of the selected NVIDIA GPU, default 0. If you have multiple NVIDIA GPUs, you can alter this value to the ID of the device you wish to run the code on.
- minIntWarp, maxIntWarp, intWarpStep: parameters related to DTW warping windows, which are minimum warping window * 100, maximum warping window * 100, and warping window step * 100, respectively. Do NOT change their default settings as it can lead to mistakes in subsequent outputs.
- maxThreadsPerBlock: maximum number of GPU threads per block, default 256. You can change this to lower values for large datasets.
- maxBlocksPerGrid: maximum number of GPU blocks per grid, default 256. You can change this to lower values for large datasets.
- datasetPath: path to the dataset folder
- dtwPath: output path of DTW distance matrices

Input parameters of ST1NN.cpp
- datasetName: name of dataset, for instance "GunPointMaleVersusFemale"
- numTrain: number of examples in the dataset, for instance "135"
- numP: number of positive examples, for instance "71". Note that we use the label "1" as the positive class.
- tsLen: time series length, for instance "150"
- minIntWarp, maxIntWarp, intWarpStep: parameters related to DTW warping windows, which are minimum warping window * 100, maximum warping window * 100, and warping window step * 100, respectively. Do NOT change their default settings as it can lead to mistakes in subsequent outputs.
- minNumIters, maxNumIters: minimum and maximum number of ST-1NN iterations allowed. We advise that you keep them as their default settings.
- datasetPath: path to the dataset folder
- dtwPath: path to DTW distance matrices
- seedPath: path to the file consisting of initial PL example indices for the multiple runs of ST-1NN. For our example datasets, we have attached the seed file we used in our experiments. You are welcome to use genST1NNSeeds.m to generate your own seeds, but keep in mind that the generated seeds can OVERWRITE the orignal seed files we have provided.
- outputPath: output path of ST-1NN results

Input parameters of modelSelection.cpp
- datasetName: name of dataset, for instance "GunPointMaleVersusFemale"
- numTrain: number of examples in the dataset, for instance "135"
- numP: number of positive examples, for instance "71". Note that we use the label "1" as the positive class.
- tsLen: time series length, for instance "150"
- numRandRuns: the number of runs for random sampling based methods. Do NOT change its default value as it can lead to mistakes in subsequent outputs.
- minIntWarp, maxIntWarp, intWarpStep: parameters related to DTW warping windows, which are minimum warping window * 100, maximum warping window * 100, and warping window step * 100, respectively. Do NOT change their default settings as it can lead to mistakes in subsequent outputs.
- datasetPath: path to the dataset folder 
- ST1NNPath: path to ST-1NN results
- seedPath: path to files containing the sampling orders for the random sampling (Rand) strategy. You are welcome to use genRandSampSeq.m to generate your own seeds, but keep in mind that the generated seeds can OVERWRITE the orignal seed files we have provided.
- outputPath: output path of final model selection results

Final output of modelSelection.cpp includes the following.
- F1-score files. The naming format is "\[dataset\]\_fscoreByRound\_rand10\_\[Sampling strategy\]\_\[Evaluation strategy\]\_\[ST-1NN seed ID\]\_.txt", for instance "GunPointMaleVersusFemale_fscoreByRound_rand10_Dis_IFE_3.txt". The ST-1NN seed ID is among 0-9 which correspond to the ten runs with different initial PL set. Each file consists of numU datapoints where numU is the number of U examples. The _i_-th data point is the F1-score of the selected model when the number of queried U example is _i_. If there are ties between multiple selected models, we use their average F1-score, as this is the expected F1-score for randomly picking a selected model with equal probability.
- User interaction response time files. The naming format is "\[dataset\]\_avgTimeBetweenQueries_rand10\_\[ST-1NN seed ID\]\_.txt", for instance "GunPointMaleVersusFemale_avgTimeBetweenQueries_rand10_3.txt". Each file consists of only one datapoint, which is the average user interaction response time (in seconds) for the current ST-1NN seed.

# On the datasets
In our paper, we have used two data sources: the UCR archive and MIT-BIH Arrhythmia Database (MITDB). Their references and web links can be found in our paper.

As with UCR datasets, we have used the original data without further editing. The complete datasets are available at http://www.cs.ucr.edu/~eamonn/time_series_data_2018/. Note that for datasets with missing values and variable time series lengths, the UCR archive has provided officially preprocessed versions of them, which is the data we have used in our experiments.

As with MITDB data, we have used leads A of Record 201 and 205. The original data is available at https://physionet.org/content/mitdb/1.0.0/. We have preprocessed the data following the practice of this paper below. 

    J. He, L. Sun, J. Rong, H. Wang, and Y. Zhang, "A pyramid-like model for heartbeat classification from ECG recordings," PLOS ONE, vol. 13, pp. 1–19, 11 2018

For the data preprocessing source code, see https://github.com/SamHO666/A-Pyramid-like-Model-for-Heartbeat-Classification. For user convenience, we have attached the preprocessed dataset (in UCR format, see 201_A_pyramid_TRAIN.tsv and 205_A_pyramid_TRAIN.tsv in the Data folder). Note that the data has been relabeled such that the VEB class is labeled as "1", while all other data is labeled as "0".

Note that for UCR datasets, we only used the training sets. This is because in transductive PUTSC, there are no testing sets in the conventional sense. Therefore, following the practice of the paper listed below, we only used the training sets in our experiments.

    M. G. Castellanos, C. Bergmeir, I. Triguero, Y. Rodríguez, J. M. Benítez, "On the stopping criteria for k-Nearest Neighbor in positive unlabeled time series classification problems," Inf. Sci. 328, pp. 42-59, 2016
    
As with MITDB data, we used the entire records (a small proportion of the data is discarded in the preprocessing phase) which was not separated into training and testing sets by the original contributors. The files "201_A_pyramid_TRAIN.tsv" and "205_A_pyramid_TRAIN.tsv"in the Data folder were named this way only to facilitate file reading by our code. It does NOT correspond to an actual training/testing set.

# Is ST-1NN the state-of-the-art PUTSC paradigm?

This question is a little weird, because ST-1NN is a paradigm, namely a category of models, not a single model. Usually, it is more appropriate to discuss whether a model is SOTA or not. However, since we are dealing with PUTSC model selection and in this paper we have focused on model selection for ST-1NN, this question becomes more relevant. To rephrase it, can any of the existing non-ST-1NN models significantly outperform the best possible performance by existing ST-1NN based models? 

Unfortunately, our best answer is that there is no sufficient evidence to show that any non-ST-1NN based models can significantly outperform it. This is because there are no comprehensive experiments on a large batch of datasets to compare the two. However, we can confidently say that our own PU-Shapelets (PUSh) is significantly weaker than ST-1NN. Concretely, we have compared PUSh with ST-1NN on the 122 UCR datasets used in our paper. For ST-1NN, in each run on each dataset we use the result of the best of the 1271 models on that run (this is effectively the Optimal method in our paper). We use this because we want to simulate the case where ST-1NN's potential is fully unleashed. For PUSh, due to the limited time we have and the fact that PUSh is too memory-inefficient to run on certain large datasets, for now only results on 94 datasets are available (see fscores_PUSh_ST-1NN.xlsx). 

We use one-tail Wilcoxon signed rank test to see if PUSh is weaker than ST-1NN. Giving PUSh the edge, we set its F1-scores on datasets where the actual result is not available to the perfect (and unlikely) 1. Even with this unfair advantage, the p-value is 0.0021 < 0.05, which indicates that PUSh is significantly weaker than ST-1NN at best. 

# Issues with Critical Difference Diagrams

To plot the critical difference diagrams, we use Friedman test (alpha = 0.05) to decide if there are significant differences among the 18 methods. We then use two-sided Wilcoxon signed rank test (alpha = 0.05) with Holm correction to divide them into groups such that methods in the same group have no significant difference. In the diagrams, methods to the right have higher average rankings than those to the left according to Friedman test. Methods in the same clique (marked by solid bars) have no significant difference according to Wilcoxon signed rank test. However, there are two issues that we have not been able to fully solve.

The first issue concerns Holm correction. Since signed rank test is a pairwise test, to control family-wise error rate (FWER), some correction method must be used when doing multiple such tests. Such correction often has to do with the number of comparisons (i.e. number of hypotheses, or the number of p-values) k. For example, let us consider Bonferroni correction, a more conservative yet simpler correction than Holm. For k comparisons, Bonferroni correction times each of the k p-values by k to obtain the adjusted p-values. These adjusted (rather than the original) p-values are then used to compare with the pre-set alpha (say 0.05) to decide whether to reject each hypothesis. 

In our case, suppose there are m methods to compare (for example, for AL-based methods, m = 18 in our case), thus we need to perform m(m-1)/2 two-sided signed rank tests. If we use Bonferroni correction, intuitively we should time each p-value by m(m-1)/2. Similar procedures can be done for Holm correction. However, in the following paper, 

    Alessio Benavoli, Giorgio Corani, Francesca Mangili: Should We Really Use Post-Hoc Tests Based on Mean-Ranks? JMLR 17: 5:1-5:10 (2016)
    
the authors suggested that we should instead time m(m-1), seemingly because of the two-sided nature of the signed rank tests. Personally, we find it hard to understand the rationale behind this, and we do not know how to take into account the two-sided nature for Holm correction. We have contacted the authors of the JMLR paper for their advice, and have also posted relevant questions on online forums such as Cross Validated and Research Gate (https://www.researchgate.net/post/How_to_conduct_Holms_procedure_for_two-sided_Wilcoxon_signed_rank_test_when_conducting_pairwise_comparisons_for_multiple_classifiers), but has so far not been able to obtain a conclusive answer as to how to address this. Therefore, in our paper, for all Holm corrections we have used the value m(m-1)/2, disregarding this issue. We have chosen to do so because it seems that this conforms with the R implementation of such a procedure. However, we are open to suggestions on potentially better practices.

The second issue concerns the conflict between Friedman Test and Wilcoxon signed rank test. In the diagram, the methods are displayed in order of their average rankings obtained in Friedman test, higher (i.e. smaller) rankings are to the right. However, when forming cliques, signed rank test is used. This presents a potential conflict. Suppose we have m' (3 <= m' <= m) methods, among which are A, B and C, where A ranks higher than B, and B ranks higher than C. In this case, due to the difference between Friedman and signed rank tests, there is a chance that the signed rank p-value (after adjustment by Holm correction) between A and C is larger than that between A and B. It may even be the case that signed rank test dictates that A and C should be in the same clique, while A and B should not. In this case, there is no way to plot the clique in the diagram to accurately reflect this. Our current way of handling this is that as long as A and C are in the same clique, A is not in the same clique as the method ranked just below C, and C is not in the same clique as the method ranked just above A, we add a clique (black bar) from A to C, covering all methods between them. This may not be an ideal solution. However, we suspect if there is a better one. It may be the case that the critical difference diagram is innately unable to handle this. However, since as far as we know it is the best visualization tool for comparisons of multiple methods on multiple datasets, we currently settle on this solution. We welcome any suggestion on potentially better practices.
    

