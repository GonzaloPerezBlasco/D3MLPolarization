<H1> AmorphousMLAlone.ipynb </H1>

<h2>Objective of program:</h2>

Trains all requested models for all requested augmentations. The results will automatically be compared when you run the notebook in *TestAModelFolder/LTestVisualCheck/TestVisualCheck.ipynb*

<h2>Input:</h2>

It looks in the folder *TestAModelFolder/FileReadingStoring/AmorphousMLDataBase* for all valid experiments. 

<h2>Output: </h2>

1. **AmorphousAllTestsFolder\_{Complexity}\_{num_augmentations}** 
For each Complexity and each num_augmentations (a.k.a, for each model structure and data organization) a folder is created. Inside there are folders for each isolated experiment iteration.

    1.1 Missing{base\_name}
For each experiment in the data base (AmorphousMLDataBase) a folder is created and inside there are all the files that the code has created when training and predicting.
 
        1.1.1 Difference\_{base\_name}.jpg
For each experimental point, we obtain the prediction and we obtain the difference between prediction and experimental values. It may be helpful to detect an overall deficiency when predicting.

        1.1.2 Difference\_{base\_name}.txt
Has the same information as the last .jpg but written on a .txt file

        1.1.3 Missing\_{base\_name}.jpg
This is the important graph. It shows in black the experimental values with their uncertainty, in red the pure ML predictions (without the correction) with the uncertainty bands in a transparent red, in green the final predictions (ML + linear correction) with uncertainty as the green transparent band and in blue the corrected predictions for the time points where experimental values are known

        1.1.4 model_PolarizationD3.keras
The trained file of the model used in this isolated-experiment iteration

        1.1.5 PredictedData\_PolarizationD3\_Missing{base\_name}.txt
The predicted values every 1000 seconds (fixed scale)

        1.1.6 PredictedPoints\_PolarizationD3\_Missing{base\_name}.txt
The predictions at the same time as the measured values

        1.1.7 RawData\_PolarizationD3\_Missing{base\_name}.txt
The black part of the graph but in a .txt file. It is supposed to be a duplicate of the file in AmorphousMLDataBase but it is stored to make sure that the scaling process is being done accurately (if it is not the same, be very very careful. I have thoroughly checked but you never know)

        1.1.8 scaler_static_PolarizationD3.pkl
The scaler for the static parameters (initial and final polarizations included) used in this isolated-experiment iteration

        1.1.9 scaler_time_PolarizationD3.pkl
The scaler for the time evolution used in this isolated-experiment iteration

        1.1.10 scaler_y_PolarizationD3.pkl
The scaler for the polarization values used in this isolated-experiment iteration

2. **AmorphousExecution\_times.txt**
It times how long the code took to loop for each model to loop over all the isolation

3. **AmorphousLog\_Testing\_ML.txt**
A log file with every step that the algorithm has followed



**Warning. When loading a single experiment and Hot-encoding CellID the program won't know it it should encode one or three values. (brings up an error of shape mismatch)
WARNING VERY VERY IMPORTANT, IF A NEW CELL IS ADDED, TWO EXPERIMENTS WITH THAT CELL ARE REQUIRED OR HOT ENCODING WILL BREAK (AND THE PROGRAM WON'T PREDICT WITH ZERO EXAMPLES OF THAT CELL)**

This code requires that AmorphousFileLectureTests.ipynb has been run. As the data base is extremely small, the effects of overfitting and underfitting are very prone to happening. To test if a model performs as expected it should try to predict files that have not been used in the process of training and validation. As we can't extract too many files form the data base, the solution is to extract only one experiment, train a model with a given structure with all the other experiments, try to predict this isolated experiment (it is a blind prediction where we know what the output looks like) and repeat this entire process for all experiments.

____________________________________________________________________________________


Here is an overall guide of what the code does:

    1. Define the model and decide the number of augmentations
    1. Creates the folder where the isolated files will temporally reside
    2. Loops for all pairs of array and parameter files
    3. Moves both, parameter and array, files to the isolated directory
    4. A subfolder on the main folder gets created with the name of the isolated experiment. All results will be stored here for this isolation iteration.
    5. Loop for all polarization modes to be processed. It can be SoftPolarizationD3, PolarizationD3 or both
    6. Load the rest of the files and augment them
    7. Obtain the time and polarization scalers. They are obtained using all time values and all polarization values including the soon-to-become static features
    8. Obtain the static features scaler.
    9. Save scalers locally
    10. Train the model. The data gets scaled during this step.
    11. Load the isolated experiment and align both sets of data (use align_static_vectors)
    12. Build manually the dataset for the isolated experiment
    13. Extract the initial and final points, add them as parameters and scale the entire static vector with the general scaler.
    14. Scale time and polarization arrays (only intermediate points). This is compatible with the scaling done to the rest of the experiments.
    15. Combine all structures so that the shape is correct for prediction.
    16. Check for shapes and create the final NumPy arrays of the experiment
    17. Extract the initial and final polarizations and keep a scaled and unscaled version of all. Then predict at those time points. 
    18. Prepare the linear correction and predict in the time points where we have data. The unscaled (physical units) outputs are pred_polar and pred_sigma.
    19. Plot the raw values (black), the predictions for those time points (blue) and predictions every 1000 seconds (red if there is no correction, green if there is correction)
    20. Save data
    21. Move all experiments back to the original folder

