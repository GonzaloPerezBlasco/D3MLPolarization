# D3MLPolarization

There are four folders each responsible for a different task. If you just want to obtain the predicted polarization decay you only have to use PredictWithModel. If you want to add measurements (after a reactor cycle) to the data base then you also need to use CreateModels. In case you want to see how well a model performs, use TestAModelFolder. At the end of this README there is a very short summary to use it if you don't need to understand what is happening.
Before using the code files inside the ML folders, the code files inside FileReadingStoring must have been run at least once. There is no manual work needed unless specified in this README. If the explanation here is not thorough enough, there are two showcase videos, one in english and the other in spanish available through this link to Google Drive (that may or may not be accesible after two or more years if the University of Zaragoza account is erased)
ENGLISH: https://drive.google.com/file/d/1BeD356JblyybWiLf6zkPYyUM6mXkUMU8/view?usp=sharing
SPANISH: https://drive.google.com/file/d/13WnhcQqzGiN4jHNVd4pMpd0Ootr1qKG6/view?usp=sharing
________________________________________________________________________________

1.D3Files

Inside the D3Files folder you can have all the "processed" from the Data Base of D3 (ILL Data-Base). This is just a log to store them in case they are needed to retrain or if a code file erases a copy).


_______________________________________________________________________________

2.PredictWithModel

This is the code that uses the models that have been created and creates the prediction for the polarization decay in any experiment. There are two folders inside 

2.1 FileReadingStoring
	
It is responsible for preparing the outputs from D3 for the experiments you want to predict. To do it, enter FileReadingStoring and inside the subfolder D3Files paste the "processed" folder you can download from the ILL-DataBase. Normally if you download the "processed" folder from the ILL-DataBase it comes in a zip file that you can just paste inside D3Files. The code files will unzip it and find the correct files and I think it also works with unzipped folders (not tested) so the only thing that needs to be done is to paste your zip file inside D3Files.

Afterwards run either CrystalineFileLecturePredict.ipynb or AmorphousFileLecturePredict.ipynb depending if the experiment uses the Amorphous Substance settings or the Monocrystal/Powder settings. To run it, open them with JupyterLab and launch the only cell present. If you want more information on what the program does and what folders and files it creates, read the README on this folder.

The only real check that needs to be done is to open the folders BadFiles and PlotResults. If there is an experiment in BadFiles please check the LogFiles the .ipynb file produces to see the reason why it was rejected. In PlotResults you should find the experiments that are processed

2.2 ML

The contents of this folder will do the actual predictions.

First check that there are models inside the folder ModelsCrystaline or ModelsAmorphous. Inside you should find folders called Model_{Complexity}_{num_augmentations}. If they are there go to the next step. If there aren't any or you want to change them go to CreateModels->ML->ModelsCrystal or CreateModels->ML->ModelsAmorphous, copy the model folders that you want and paste them to their analogous folders here inside the PredictWithModel subfolders. If there are any, then please read the instructions to launch the code files inside CreateModels.

Then run either PredictCrystal.ipynb or PredictAmorphous.ipynb to do the calculations.

After using them, a new folder called PredictionsFolder will get created. Inside you will find for each model and each experiment (a.k.a each cell that has been used) two files, a PredictedData_*.txt file and a .png file. INside the txt file you will find three columns, GridTime which is the time of each prediction (the reference, t=0, is always the first polarization measurement at the beginning of the experiment), PredictedPolarizationD3 and ErrPredictedPolarizationD3. The .png file contains two scatter plots, the red one is the raw ML prediction and the green one (the one stored at the .txt file) is the ML prediction corrected to force the initial and final points.

 === WARNINGS FOR PREDICTWITHMODEL ===

1. IF YOU RUN THE CODE AND YOU GET AN ERROR SAYING THAT SOMETHING IS NOT DEFINED THERE IS A 99% CHANCE THAT TJE ISSUE IS THAT YOUR VERSION OF JUPYTERLAB DOESN'T HAVE DOWNLOADED THAT LIBRARY. ADDING A NEW CELL WRITING pip install _________ SHOULD FIX THE ISSUE. HOWEVER ASK CHATGPT FOR THE EXACT LINE pip install ______ THAT YOU NEED TO USE

2. THE FILEREADING.IPYNBs FILES EXPECT THE STRUCTURE OF THE FLI FILES TO BE LIKE THIS FOR CRYSTALS:

polariser cell info ge18004 pressure/init. polar 2.30 0.78 initial date/time 06 06 25 @ 15:15
   59685   0.000   0.000   2.000 06/06/25 16:20:21     228.88  +z +z     0.8460    0.0007   11.9879    0.0558     180.00
   60273   0.000   0.000   2.000 10/06/25 15:36:07     170.95  +z +z     0.5454    0.0023    3.3998    0.0215      60.00
polariser cell info ge18012 pressure/init. polar 2.30 0.78 initial date/time 10 06 25 @ 15:45
   60275   0.000   0.000   2.000 10/06/25 15:47:23     170.14  +z +z     0.8363    0.0011   11.2157    0.0764      60.00
   60343   0.000   0.000   2.000 11/06/25 14:30:33     169.83  +z +z     0.8016    0.0021    9.0804    0.1047      60.00

AND THIS FOR AMORPHOUS SUBSTANCES:

polariser cell info ge18012 pressure/init. polar 3.10 0.77 initial date/time 24 11 23 @ 12:10
analyser cell info sic1403 pressure/init. polar 2.00 0.77 initial date/time 24 11 23 @ 12:10
   40944   3.000   3.000   3.000 24/11/23 13:09:50       0.00        0.5383    0.0021    3.3319    0.0198     120.00
   40976   3.000   3.000   3.000 25/11/23 12:06:27       0.00        0.3985    0.0027    2.3247    0.0146     120.00


WE EXPECT THE FLI FILES TO HAVE FOR EACH CHARACTER STRING ONLY TWO NUMERICAL ROWS AND THE MILLER INDICES TO BE THE SAME FOR BOTH ROWS FOR EACH SET OF DATA. If you ever need a more universal code or anything ask, but now this is a requirement.

3. IF YOU EVER USE A NEW TYPE OF CELL (WITH A DIFFERENT CELL ID OR ANALYSER CELL ID) THAN THESE FOR CRYSTALS:
ge18012
ge18004
ge18006
 
AND THESE FOR AMORPHOUS SUBSTANCES:
ge18004,sic1402
ge18011,sic1402
ge18012,sic1403

THE CODE PREDICTION CODE WILL BREAK DUE TO TWO REASONS. FIRST, THE MODEL WON'T BE ABLE TO IMAGINE WHAT A NEW CELL TYPE WILL DO, SO IT NEEDS TO BE TRAINED WITH MORE THAN ONE EXPERIMENT TO BE ABLE TO START PREDICTING. SECOND, THERE IS A VARIABLE CALLED __________ AT LINES _________ IN PREDICTCRYSTAL.IPYNB AND AT LINES _________ IN PREDICTAMORPHOUS.IPYNB THAT DEPENDS ON THE CELLIDs. I RAN OUT OF TIEM TO AUTOMATE THE UPDATE PROCESS SO IT HAS BEEN MANUALLY FIXED. IF THERE ARE MORE CELLIDs IN THE FUTURE THESE LIST NEEDS TO BE UPDATED. TO UPDATE IT GO TO THE MODEL CREATION IPYNBS, FIND THAT SAME VARIABLE AND ADD PRINT(). READ THE OUTPUTS OF THE CODE TO FIND IT AND PASTE IT HERE. Sorry for any inconveniences that this situation may create

________________________________________________________________________________


3. CreateModels

If you want to create a model then you need to run the code files present here.

3.1 FileReadingStoring

The process is identical to the one in PredictWithModel, put on D3 all zip files you want you model to be trained with, run either AmorphousFileLectureCreate.ipynb or CrystalineFileCreate.ipynb and, optional, check if the BadFiles folder to see if the experiments you have added after each reactor cycle are treated incorrectly

3.2 ML Run CreateSaveModelAmorphous.ipynb or CreateSaveModelCrystal.ipynb to obtain the models in ModelsAmorphous or ModelsCrystal. Inside these two folders you will find the folders that you can copy and paste to the Model folder inside PredictWithModel. Copy the entire folder (for example Model_Average_3) not just the contents or the prediction code will break.

What if do I need to do if I want to create a new model?

There are two steps to do so. First go to the sixth step, 6. Model Architecture,  in CreateSaveModelAmorphous.ipynb or CreateSaveModelCrystal.ipynb and add another model using this structure.
elif Complexity == "________":
	def build_model(input_dim_static, use_uncertainty=False):
		(...)
		return model
You can create a custom model but the inputs and outputs must be the same as in the other models (if in doubt, ask an AI like ChatGPT to help when writing a model structure)
Second, go to 12. MAIN CODE and add your new Complexity type in the fist loop and select the number of augmentations you want to use in the nested second loop. It will create models using all possible combinations of the two lists. If you do not want some of the models that are automatically created, remove them from the lists.  === WARNINGS FOR CREATEMODEL ===
AS OF NOW THE MODEL STRUCTURE CAN'T INCLUDE NON-KERAS ARCHITECTURES OR DOUBLE INPUT MODELS. THIS MIGHT BE SOMETHING INTERESTING TO PLAY WITH IN THE FUTURE

Here you have a list of the training times for each model using the data base until the first cycle of 2025 (included).
Time it took each model to be trained (68 Crystal experiments and 22 Amorphous experiments)
Execution time=60.299263 seconds for Crystal Naif 3
Execution time=80.102365 seconds for Crystal Naif 5
Execution time=101.088829 seconds for Crystal Naif 7
Execution time=119.885529 seconds for Crystal Naif 9
Execution time=77.859478 seconds for Crystal Simple 3
Execution time=106.144997 seconds for Crystal Simple 5
Execution time=143.909518 seconds for Crystal Simple 7
Execution time=173.053109 seconds for Crystal Simple 9
Execution time=56.425388 seconds for Crystal Average 3
Execution time=79.558593 seconds for Crystal Average 5
Execution time=103.525498 seconds for Crystal Average 7
Execution time=127.568648 seconds for Crystal Average 9
Execution time=87.121417 seconds for Crystal Complex 3
Execution time=122.914268 seconds for Crystal Complex 5
Execution time=161.588065 seconds for Crystal Complex 7
Execution time=201.641090 seconds for Crystal Complex 9
Execution time=26.961958 seconds for Amorphous Naif 3
Execution time=33.078888 seconds for Amorphous Naif 5
Execution time=39.718516 seconds for Amorphous Naif 7
Execution time=48.783417 seconds for Amorphous Naif 9
Execution time=34.927928 seconds for Amorphous Simple 3
Execution time=40.110364 seconds for Amorphous Simple 5
Execution time=52.122849 seconds for Amorphous Simple 7
Execution time=57.954138 seconds for Amorphous Simple 9
Execution time=26.117254 seconds for Amorphous Average 3
Execution time=34.822092 seconds for Amorphous Average 5
Execution time=37.940874 seconds for Amorphous Average 7
Execution time=43.588653 seconds for Amorphous Average 9
Execution time=35.430177 seconds for Amorphous Complex 3
Execution time=45.733330 seconds for Amorphous Complex 5
Execution time=54.983139 seconds for Amorphous Complex 7
Execution time=68.275803 seconds for Amorphous Complex 9
_________________________________________________________________________________

4. TestAModelFolder

If you want to know if your model is adequate (it doesn't overfit, underfit, gives a reasonable curve, etc.) then a LeaveOneOut training method can be used. If you are confident the models you already have are enough then there is no need to try these code files. What the following code files will do is to take a given model architecture, train the model using all but one of the experiments os the data base and then try to predict the missing experiment. As the experiment used for prediction has ever been seen by the model, there is no risk of information leackage or memorization on the test experiment. This process can be made for each experiment (create a model trained with all but that experiment) and the results for each missing experiment can be compared between models to decide which ones perform the best. To compare them, as the real values are known for the known experiments, a metric like chi-squared or MAE can be obtained per experiment and an overall score can be computed. We can also visually compare the performance for each isolated experiment. The time these tests require far exceeds the traing time of one single moment (if there are 12 models and 68 experiments, 12*68 models have to be created and each of them may take more or less time. For 12*68+12*25 models using the ILL internship computers it took around four days to run the code. Run this tests if the models don't perform well and you want to see how it performs using more experiments or if you create a new model architecture to test it. If you just run the one inside LTestVisualCheck if you can see the difference between models using the data-set up until 2025)

4.1 FileReadingStoring

As always, copy and paste the experiments that you want to train and isolate (zip folders format) inside D3Files and run either AmorphousFileLectureTests.ipynb or AmorphousFileLEctureTests.ipynb. 

4.2 ML

The LeaveOneOut routine can be run using the files CrystallineMLAlone.ipynb and AmorphousMLAlone.ipynb

4.3 LTestVisualCheck

First copy and paste the folders that CrystallineMLAlone.ipynb and AmorphousMLAlone.ipynb have created inside LTestVisualCheck (at the same level as the ipynb files).
Then run the first cell of LTestChiSquaredTests.ipynb if you want to obtain information using the chi squared metric. This cell will create a subfolder called ChiSquared where only three files are important:
 - Chi2_rank_vertical.png For every experiment, the chi squared value of the preditions of each model is obtained. Then all chi squared values of the different models in each experiment is compared. The one that has the smaller chi squared value obtains a +1, the second smallest a +2 and so on. This process is repeated for each experiment and these integer values are summed. At the end we can say that the model with the smallest score has performed better overall
 - Chi2_AccumulativeScores.txt has the same information than the graph but on a txt file
 - Chi2_NumberOfExperimentsWhereEachModelOutperforms.txt as its name indicates counts only the number of experiment where each model obtains the smallest chi squared value. Sometimes, models that overfit perform better on this test (as they pass through all the points)

Run the second folder if you want the same information using a metric similar to the Mean Average Error (MAE). Instead of summing over the differences squared and divided by the uncertainty of the prediction squared, here we are summing over the absolute value of the differences and divided by the uncertainty (not squared). By running this cell you create the subfolder Lvalues that contains the same type of information (Lvalues_rank_vertical.png, Lvalues_TotalScores.txt with the information of the graph and Lvalues_WinnerScores which is the equivalent version of Chi2_NumberOfExperimentsWhereEachModelOutperforms.txt but with The L metric

If you run the third one it will automatically combine the predictions for twelve models for each experiment in a single image. This way it is easier to spot the differences between models in a quick look. The code also orders them in order of complexity (each consecutive row has a bigger complexity) and number of augmentations (each consecutive column has more augmentation). This way the models are organized from simpler (top left) to more complex (bottom right).

 === WARNINGS FOR PREDICTWITHMODEL ===

THE IMAGE COMBINATION CELL HAS BEEN CREATED TO EXPECT THE NAMES NAIF, SIMPLE, AVERAGE AND COMPLEX AND FOR THERE TO BE EXACTLY 12 MODELS. IF A DIFFERENT NAME IS USED OR THERE IS A DIFFERENT NUMBER OF MODELS, IT WILL NOT WORK. PLEASE CHANGE THE CODE ACCORDINGLY. 
THE IMAGE COMBINATION CELL ALSO DOES NOT REMOVE THE TITLE OF EACH GRAPH AND DUE TO THEIR LENGTH THEY OVERLAP. APOLOGIES IN ADVANCE
_________________________________________________________________________________

Brief Summary.

1. Go to PredictWithModel->FileReadingStoring->D3Files and paste your zip file with the "processed" data (straight from the ILL-DataBase) that you want to predict. Run AmorphousFileLecturePredict.ipynb if you used an amorphous substance or CrystalineFileLecturePredict.ipynb if it was an crystal or powder. If there is an error, install the python libraries.
2. Go to PredictWithModel->ML->ModelsAmorphous or ModelsCrystalline and check if there are models. If there are and they are the ones you want (we recommend Average_3 for both types of experiments), proceed to step 6
3. Go to CreateModels->FileReadingStoring (update D3files after each reactor cycle if there are new polarization decay experiments) and run AmorphousFileLectureCreate.ipynb or CrystalineFileLectureCreate.ipynb
4. Go to CreateModels->ML and run CreateSaveModelAmorphous.ipynb or CreateSaveModelCrystal.ipynb
5. Go to CreateModels->ML->ModelsAmorphous or ModelsCrystalline, copy the folders and paste them at PredictWithModel->ML->ModelsAmorphous or ModelsCrystalline
6. Go to PredictWithModel->ML and run PredictAmorphous.ipynb or PredictCrystalline.ipynb. The text file with the predicted polarization and a graph with the time evolution (green curve) will appear in PredictionsFolder inside each model subfolder and experiment sub-subfolder
7. Correct your data with the predicted polarizations (not part of this pipeline at the moment)
