<h1>PredictAmorphous.ipynb</h1>

It takes the files from AmorphousFileLecturePredict.ipynb and predicts with them. There is now only one function that can be used for prediction, Predict.
IMPORTANT, IN THE FOLDER AmorphousModels YOU HAVE TO MANUALLY PASTE THE FOLDERS WITH THE MODELS. IT IS A MANUAL PROCESS

The inputs of the function are:
1. The point in the past to start predicting (e.g. if you don't want to start your predictions at the time of the first measurement you can write -123 and the predictions will start 123 seconds before the first experimental measurement). Accepts positive and negative values and I believe even floats (not tested for non-integer values but I don't think it makes sense to ask for time values with a precision smaller than what the files from D3 have)
2. The time between steps (the time interval between predictions, e.g, if you want predictions every 4 seconds, write as the second input 4).
3. The point in the future to stop predicting (e.g. if you don't want to stop at the last experimental measurement you can write 1024 and the last polarization will be the last step before t_final+1024). Accepts positive and negative values


As we want to correct the Diffractogram, perhaps it is wiser to store the time variable using the zero time reference using the beginning of the Diffractogram. Predict will demand another input apart from the ones from PredictWithPolarizationTimeReference which will be the new time reference you want to use (it could be the beginning of the diffractogram or even a random time point). The predictions will be the same but GridTimes will be referenced with another time reference. To use it the new reference time must be written like
DiffractogramAbsoluteTime = datetime.strptime("2023-12-08 13:06:39", "%Y-%m-%d %H:%M:%S")


___________________________________________________________________________________________

 
OUTPUTS OF THE CODE

1. AmorphousLogFile_PredictingML.txt
It is a log with all the important steps that the code has done

2. AbsoluteTimes.txt
It is only created if you use Predict (not with PredictWithPolarizationTimeReference). It stores for each experiment (and model, if I have time I will remove those duplications) the name of the experiment, the time where the first polarization measurement was recorded (in an absolute format like "2023-12-08 13:06:39") and the time value that was used for the Predict function (it is just DiffractogramAbsoluteTime that was added so there is a txt file with all the information needed to change from time intervals with random reference points to absolute time strings if needed)

3. AmorphousExecution_times.txt
It records the time it took each individual experiment to be fully predicted. The time depends on the amount of points and model but 48 full predictions with 20 second gaps took less than 90 seconds (hopefully it is fast enough) 

4. AmorphousPredictionsFolder
Stores all the predictions. The first subdivision of folders separate the information using the model

4.1 Model_{Complexity}_{num_augmentations}
Depending of the model (characterized using two strings) you can obtain predictions for all the experiments. In each Model_{Complexity}_{num_augmentations} folder you will find folders for each experiment

4.1.1 Experiment_{base_name}
For each model and {base_name} (or experiment name) you can find one of these folders. The contents contain the predictions

4.1.1.1 {Complexity}_{num_augmentations}_{base_name}.jpg
Contains the pure ML predictions in red and the correction (in green) using the first and final polarization measurements. For more information read the explanation of what the code does.
 
4.1.1.2 PredictedData_{Complexity}_{num_augmentations}_{base_name}
Contains the same information as the green curve but on a txt file. On the first column you have the relative time where t=0 corresponds to the first correct polarization measurement. On the second column you have the absolute time (in a format datetime.strptime("2023-12-08 13:06:39", "%Y-%m-%d %H:%M:%S") ) and on the second and third columns you can find the polarization and uncertainty of polarization associated to those time values

___________________________________________________________________________________________




Process:

1. Read the {base_name}.txt and {base_name}_Parameters.txt from AmorphousMLDataBase (running AmorphousFileLectureCreate.ipynb in its folder is COMPULSARY)

2. Choose to use PolarizationD3 or SoftPolarizationD3 (always use the first one as it is a more natural approach)

3. Scaling. The code needs normalization to work so the parameters (or static vector), the time values and the polarization values need to be normalized. There are many ways to choose the normalization process. Especially since we are going to add the first and last polarization measurement to the static vector. Here is the reasoning:
When this algorythm is used, we will give it the same parameter types and the beginning and final polarization. Therefore the wisest approach is to teach the model with this
exact structure. Therefore it is needed that we extract the initial and final polarization from the files and add it into the parameters. This means that all files contribute 
two points less to the algorythm, hence why two point files are useless for training. If uncertainty is used the parameters will have an uncertainty assigned to each of them. In 
order for the model to avoid treating the uncertainties as independent values (when they are dependent to only one of them) we are removing them entirely. Technically the algorythm 
is able to determine if the parameters are useful and reduce the weight to them. As we have so few points, in case this weighting is not done correctly, I have decided to ignore them
Warning, the outputs are scaled (if you want them raw you will have to undo this scaling). Everything has been taken care of automatically
 Summary of Options
Option	Description
1. Leave as is	First/last points removed from time series, added to static params. No duplication. <- What we are doing
2. Duplicate	First/last points added to static, and remain in the time series. Duplicated info.
3. Perturb	First/last points added to static, and perturbed copies remain in time series.

 Option 1 – Leave as is (No duplication)
    What you do: First and last (time, polarization) points are removed from the time series and used only as static parameters.
    Pros:
        No risk of information leakage from duplication.
        Keeps static and dynamic features clearly separated.
        Avoids overfitting to repeated values.
        Cleanest conceptually.
    Cons:
        You're losing two data points per experiment — which may be relevant if datasets are small.
        Time series now has a gap at beginning and end.
        Might miss important boundary dynamics if those edges are meaningful (e.g. stabilization or edge effects).
 Best if you want a clean and leak-proof setup and you're okay with slightly reduced data volume.
 Option 2 – Duplicate first/last into both static and time series
    What you do: First and last values are added to static vector and remain in the time series.
    Pros:
        You retain all original data points, no loss in dataset size.
        You still have access to boundary behavior through static features.
        Easier to debug/troubleshoot models if you see consistency across static/dynamic behavior.
    Cons:
        Information leakage: same value appears in two places, and each is scaled separately.
        Risk of overfitting, especially if those boundary values dominate or are strongly correlated with labels.
        Can confuse models trained to learn relationships, if they see one value in two places with different scaling.
        Scaling separately can make the model learn inconsistent representations of the same quantity.
 Only choose this if your data is scarce and overfitting isn’t a concern, or if the duplicates are negligible in impact.
 Option 3 – Perturb duplicates
    What you do: First/last are added to static vector, and a slightly altered copy remains in the time series (e.g. polarization adjusted using Gaussian noise via uncertainty, small delta on time).
    Pros:
        Keeps all data points, like Option 2.
        Reduces leakage risk because values are no longer exactly the same.
        Still captures edge dynamics in the time series.
        Adds small noise robustness to the model.
    Cons:
        Still not completely clean — the model may still learn the similarity between static and perturbed values.
        If not carefully tuned, noise could introduce label mismatch or make the data noisier than real measurements.
        Slightly more complex to implement and explain.
        You now have synthetic data in your real dataset, which might affect interpretability.
 This is a clever compromise, especially if:
    You're worried about data size,
    You want to retain edge info,
    And you don’t want outright duplication.
Just make sure the noise scale matches real uncertainty — otherwise you risk corrupting the training signal.
 Final Recommendation
    If you value clean data splits and minimizing information leakage, go with Option 1.
    If you need every datapoint and the duplicates are unlikely to skew things, Option 2 is easier but riskier.
    If you want a smart middle ground with some noise robustness, Option 3 is the most nuanced but effective — just be careful with your noise scale.

That way the static vector (with initial and final points included), the polarizations and the time are scaled independently and the scaler is obtained 


5. Import the models and scalers.

6. Prepare the predictable experiment and obtain the linear correction. As our upmost priority is to obtain a prediction capable of substituting the experimental process, i.e. the ML prediction is not the be-all and end-all, an automatic correction is calculated. The first and final points are known so it makes sense that our final curve needs to pass through those two points. Therefore the pure ML prediction will get a linear curve subtracted so that these two points are fitted perfectly. It can be toggled on or off

7. Use the trained model to predict (with or without the correction) and store all the possible important information

8. Go to the next experiment and repeat the process

9. Repeat the process for the models saved on the models folder. If absolute time values are used (like in Predict) save also all necessary absolute time values.


