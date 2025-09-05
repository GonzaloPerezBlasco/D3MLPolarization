<H1>CreateSaveModelCrystallineML</H1>

This code requires that CrystallineFileLectureCreate.ipynb has been run. If you are confident that the model structures are correct (go to TestAFolder to test the structures) then this code creates the model. It trains on ALL experiments that were processed with CrystallineFileLectureCreate.ipynb 

__________________________________________________________________________________________

OUTPUTS OF THE CODE: 

1. CrystallineLogFileModelCreation.txt
A log file with every step that the algorithm has followed


2. CrystallineExecution_times.txt
It times how long the code took to create each model (0.5-1 mins aprox. per model)


3. ModelsCrystalline 
For each Complexity and each num_augmentations (a.k.a, for each model structure and data organization) a folder is created.

3.1 Model_{Complexity}_{num_augmentations}
THESE ARE THE FOLDERS YOU NEED TO COPY AND PASTE INSIDE THE PREDICTING CODE FOLDERS. THE WHOLE FOLDER NOT JUST THE CONTENTS
Inside these folders you have the entire model and the scalers all ready to use for prediction. 
Note: The corrections are dependent on the static parameters and not the model or the training routine. Therefore this models will not give you the corrected predictions. (Do not worry, in the prediction files these corrections are automatically done if you don't change the flag that activates them)
 
3.1.1 Model_{Complexity}_{num_augmentations}.keras
The model file

3.1.2 scaler_static_{Complexity}_{num_augmentations}_PolarizationD3.pkl
The scaler for the static parameters (initial and final polarizations included) used in this isolated-experiment iteration

3.1.3 scaler_time_{Complexity}_{num_augmentations}_PolarizationD3.pkl
The scaler for the time evolution used in this isolated-experiment iteration

3.1.4 scaler_y_{Complexity}_{num_augmentations}.pkl
The scaler for the polarization values used in this isolated-experiment iteration


__________________________________________________________________________________________


Process:

1. Read the {base_name}.txt and {base_name}_Parameters.txt from CrystallineMLDataBase (running CrystallineFileLectureCreate.ipynb in its folder is COMPULSARY)

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

4. Augment the experiments used for training. Augmentation of experiments is the process by which we use the uncertainty of the points of our experiment to create "fake experiments" which could have been measured. Knowing a measurement with uncertainty can be interpreted statistically as a probability of measuring a value inside the error bars. If we were to measure again the same point it is probable that the result would be different (but inside the uncertainty range). Therefore another experiment where our measurements are slightly different but inside the error margins is as valid as our original one. This way we can create copies of the experiment increasing the number of points the ML algorythm can learn from. 
There is one disadvantage of doing augmentation and that is that if the data is augmented too many times, the algorithm will always overfit (5-10 is said to be ok but better safe than sorry). Augment each experiment by adding Gaussian noise to polarization using uncertainty. Returns combined list of original + augmented experiments.

6. Train the model with all experiments and save the scalers and model. The model uses the uncertainty ONLY during augmentation.

10. Repeat the process for all four models currently available (you can add new ones by creating a new one on Define_Complexity and adding it to the list on the 'for Complexity' loop) and the four number of augmentations ["Naif", "Simple", "Average","Complex"], [3,5,7,9] (for example)

11. The time it takes is also recorded
