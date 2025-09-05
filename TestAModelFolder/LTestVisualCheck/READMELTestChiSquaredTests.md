LTestChiSquaredTests.ipynb

This code is prepared so that it condenses all the information from the code files inside TestAModelFolder/ML. 

First copy and paste the folders that CrystallineMLAlone.ipynb and AmorphousMLAlone.ipynb have created inside LTestVisualCheck (at the same level as the ipynb files).
Then run the first cell of LTestChiSquaredTests.ipynb if you want to obtain information using the chi squared metric. This cell will create a subfolder called ChiSquared where only three files are important:
 - Chi2_rank_vertical.png For every experiment, the chi squared value of the preditions of each model is obtained. Then all chi squared values of the different models in each experiment is compared. The one that has the smaller chi squared value obtains a +1, the second smallest a +2 and so on. This process is repeated for each experiment and these integer values are summed. At the end we can say that the model with the smallest score has performed better overall
 - Chi2_AccumulativeScores.txt has the same information than the graph but on a txt file
 - Chi2_NumberOfExperimentsWhereEachModelOutperforms.txt as its name indicates counts only the number of experiment where each model obtains the smallest chi squared value. Sometimes, models that overfit perform better on this test (as they pass through all the points)

Run the second folder if you want the same information using a metric similar to the Mean Average Error (MAE). Instead of summing over the differences squared and divided by the uncertainty of the prediction squared, here we are summing over the absolute value of the differences and divided by the uncertainty (not squared). By running this cell you create the subfolder Lvalues that contains the same type of information (Lvalues_rank_vertical.png, Lvalues_TotalScores.txt with the information of the graph and Lvalues_WinnerScores which is the equivalent version of Chi2_NumberOfExperimentsWhereEachModelOutperforms.txt but with The L metric

If you run the third one it will automatically combine the predictions for twelve models for each experiment in a single image. This way it is easier to spot the differences between models in a quick look. The code also orders them in order of complexity (each consecutive row has a bigger complexity) and number of augmentations (each consecutive column ahs more augmentation). This way the models are organized from simpler (top left) to more complex (bottom right).

 === WARNINGS FOR PREDICTWITHMODEL ===

THE IMAGE COMBINATION CELL HAS BEEN CREATED TO EXPECT THE NAMES NAIF, SIMPLE, AVERAGE AND COMPLEX AND FOR THERE TO BE EXACTLY 12 MODELS. IF A DIFFERENT NAME IS USED OR THERE IS A DIFFERENT NUMBER OF MODELS, IT WILL NOT WORK. PLEASE CHANGE THE CODE ACCORDINGLY. 
THE IMAGE COMBINATION CELL ALSO DOES NOT REMOVE THE TITLE OF EACH GRAPH AND DUE TO THEIR LENGTH THEY OVERLAP. APOLOGIES IN ADVANCE
