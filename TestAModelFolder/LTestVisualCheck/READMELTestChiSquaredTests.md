<h1> LTestChiSquaredTests.ipynb </h1>

<h2>Objective of program:</h2>

Takes the tested model folders and compares it using the chi squared metric. It also plots the results. To be precise, for every model (denoted by the subindex $k$ and each experiment (denoted by $i$) it computes the chi squared metric (let $j$ be each individual point in the experiment) $\chi^2_{k,i}=\sum_{j=1}^{N}\frac{(y_j-\sigma_j)^2}{\sigma_j^2}$. Then it compares it using two algorithms.

The best one is the normalized score:
It finds, for each experiment, the model with the smallest chi squared and normlizes all values of chi squared for that experiment. Each model then computes the fraction *minimum_chi_squared_of_an_experiment/own_chi_squared_of_an_experiment*. The best model for that experiment gets a 1 while the rest gets smaller numbers. Finally, all these fractions are summed (across all experiments) and the result divided by the number of experiments to "normalize them"

The other one, the acummulative one, just orders, for each experiment, all chi squared values and depending on their placement, it adds +1,+2,+3,... The higher the number, the worst score. This +1,+2,+3,... get added for all experiments

<h2>Input:</h2>

Manually copy all the folder files that were returned from the notebooks in the folder *ML*. For the amorphous configuration copy the folders like AmorphousAllTestsFolder_Naif_2 and for the crystalline one copy the folders like CrystallineAllTestsFolder_Naif_2. Paste them into the folder *Models_to_be_compared*

<h2>Output: </h2>

It will return everything in the folder *Results*

1. **ExperimentComparison**
   It is a folder that takes all images produced by *AmorphousMLAlone.ipynb* or *CrystallineMLAlone.ipynb* and combines them into a single image per experiment. That way it is easier to llok at. This folder is produced by running the last cell. If there are a lot of models it will produce very heavy images and take a lot of time.

2. **Chi2_AccumulativeScores.txt**
    Contains the results of the test where chi squared values are ordered and a +1,+2,+3,... score is given to each model

3. **Chi2_RelativeNormalizedScores.txt**
    Contains the results of the normalized score test. 
4. **Chi2_NumberOfExperimentsWhereEachModelOutperforms.txt**
    Self explanatory
5. **Chi2_rank_vertical.png**
    Shows as a histogram graph the results of **Chi2_AccumulativeScores.txt**
6. **Chi2_relative_vertical.png**
    Shows as a histogram the results of **Chi2_RelativeNormalizedScores.txt**
7. **Chi2_values_{experiment_folder_name}.txt**
    For each model inputted it will create a file where it stores all chi squared values of that model for all experiments
8. **FullHorizontalChi_ScoreOrdered.png**. Shows the normalized score in a horizontal manner but orders them using the scores, not the names

