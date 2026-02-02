<h1>PredictCrystalline.ipynb </h1>

It takes the files from CrystallineFileLecturePredict.ipynb and predicts with them. There is now only one function that can be used for prediction, Predict.
**IMPORTANT, IN THE FOLDER CrystallineModels YOU HAVE TO MANUALLY PASTE THE FOLDERS WITH THE MODELS. IT IS A MANUAL PROCESS. THE REST OF THE PROCESS SHOULD BE AUTOMATED EXCEPT FOR THE TIME CONDITIONS THAT ARE ASKED TO THE USER**

Here is a diagram of the folder output structure:
        A folder structure with the predictions:
        
        - PredictCrystalline.ipynb
        - CrystallinePredictionsFolder
          |
          |_Model_{NameOfModel 1}
          |    |
          |    |_Experiment_{NameOfExperiment 1}
          |    |   |
          |    |   |_Graph with predictions and original data (.png)
          |    |   |
          |    |   |_Text file with the results. First column is the internal program time
          |    |   |                             Second column is the time in real life of those predictions (YY-MM-DD HH:mm:SS
          |    |   |                             Third column is the predicted polarization
          |    |   |                             Fourth column is the uncertainty
          |    |   |
          |    |   |_PolarizationD3_{Name}_Paramteres.txt (original parameter file)
          |    |   |_PolarizationD3_{Name}.txt (original data file)
          |    |   
          |    |_Experiment_{NameOfExperiment 2}
          |    |   |
          |    |   |_Graph with predictions and original data (.png)
          |    |   |
          |    |   |_Text file with the results.  
          |    |   |
          |    |   |_Parameter and array original files
          |    |
          |    |_(...)
          |
          |_Model_{NameOfModel 2}
          |    |
          |    |_Experiment_{NameOfExperiment 1}
          |    |   |
          |    |   |_Graph with predictions and original data (.png)
          |    |   |_Text file with the results. First column is the internal program time
          |    |   
          |    |_Experiment_{NameOfExperiment 2}
          |    |   |
          |    |   |_Graph with predictions and original data (.png)
          |    |   |_Text file with the results.   
          |    |   |_Parameter and array original files
          |    |
          |    |_(...)
          |
          |_Combined model
          |    |
          |    |_Experiment_{NameOfExperiment 1}
          |    |   |
          |    |   |_Graph with predictions and original data (.png)
          |    |   |_Text file with the results. First column is the internal program time
          |    |   
          |    |_Experiment_{NameOfExperiment 2}
          |    |   |
          |    |   |_Graph with predictions and original data (.png)
          |    |   |
          |    |   |_Text file with the results.  
          |    |   |
          |    |   |_Parameter and array original files   
          |    |
          |    |_(...)
          |_(...)



The inputs of the code are:
1. The point in the past to start predicting (e.g. if you don't want to start your predictions at the time of the first measurement you can write -123 and the predictions will start 123 seconds before the first experimental measurement). Accepts positive and negative values and I believe even floats (not tested for non-integer values but I don't think it makes sense to ask for time values with a precision smaller than what the files from D3 have)
2. The time between steps (the time interval between predictions, e.g, if you want predictions every 4 seconds, write as the second input 4).
3. The point in the future to stop predicting (e.g. if you don't want to stop at the last experimental measurement you can write 1024 and the last polarization will be the last step before t_final+1024). Accepts positive and negative values




___________________________________________________________________________________________

 
OUTPUTS OF THE CODE (the important ones)

1. **CrystallineLogFile_PredictingML.txt**
It is a log with all the important steps that the code has done

2. **AbsoluteTimes.txt**
It is only created if you use Predict (not with PredictWithPolarizationTimeReference). It stores for each experiment (and model, if I have time I will remove those duplications) the name of the experiment, the time where the first polarization measurement was recorded (in an absolute format like "2023-12-08 13:06:39") and the time value that was used for the Predict function (it is just DiffractogramAbsoluteTime that was added so there is a txt file with all the information needed to change from time intervals with random reference points to absolute time strings if needed)

3. **CrystallineExecution_times.txt**
It records the time it took each individual experiment to be fully predicted. The time depends on the amount of points and model but 48 full predictions with 20 second gaps took less than 90 seconds (hopefully it is fast enough) 

4. **CrystallinePredictionsFolder**
Stores all the predictions. The first subdivision of folders separate the information using the model

    4.1 **Model\_{Complexity}\_{num\_augmentations}**
Depending of the model (characterized using two strings) you can obtain predictions for all the experiments. In each Model_{Complexity}_{num_augmentations} folder you will find folders for each experiment

        4.1.1 Experiment_{base_name}
For each model and {base_name} (or experiment name) you can find one of these folders. The contents contain the predictions

            4.1.1.1 {Complexity}_{num_augmentations}_{base_name}.jpg
Contains the pure ML predictions in red and the correction (in green) using the first and final polarization measurements. For more information read the explanation of what the code does.
 
            4.1.1.2 PredictedData_{Complexity}_{num_augmentations}_{base_name}
Contains the same information as the green curve but on a txt file. On the first column you have the relative time where t=0 corresponds to the first correct polarization measurement. On the second column you have the absolute time (in a format datetime.strptime("2023-12-08 13:06:39", "%Y-%m-%d %H:%M:%S") ) and on the second and third columns you can find the polarization and uncertainty of polarization associated to those time values



___________________________________________________________________________________________


Information about the parts



Here we have functions that train, validate and fit the models. Some models require the variables to be scaled or will scale them. Extra precautions need to be taken into account
1. _PrintDebug_ is a flag that allows the code to output on screen all the steps. If it is set to false, it won´t show anything. However, all information will be properly logged whether this flag is set to true or false. The name of the log is determined by the variable *log_file_path*. The code runs faster if it is set to False.

2. _ShowPlot_ is a similar flag that allows the code to show on screen all plots that are being produced. They are all stored independently of whether this flag is True or False. The code runs faster if it is set to False.

3. _LogNoise_ is another flag that allows numerical values in the log. Most of these values are not worth keeping but if you want to see if there are no NaNs or zeros you can turn it on


4. **log_message** is a function used for writting on the log file

5. **win_long_path** is a function that "fixes" directory paths

6. **load\_experiments**. It uses the directory where the array (numerical values) and parameter files resides, picks one polarization column and prepares a list of experiments to feed the ML algorythm. The output is as follows:

    *encoded\_experiments=[(static\_values, Deltatime, PolarizationD3, ErrPolarizationD3)$_{experiment 1}$,(static\_values, Deltatime, PolarizationD3, ErrPolarizationD3)$_{experiment 2}$,...]* 
    
    Note that the Cell Id is Hot Encoded. The type of cell did not affect greatly the predictions. However in the future, it may be useful to give more information to Cell_ID. As of 2025, these strings are Hot Encoded which means that the code finds all the different types, creates columns for each type and writes 0 or 1 (bool) depending of whether the cell was from one type or the other. This is the standard procedure to feed categorical variables to ML.
    The size of the output list depends on the number of pairs of .txt files present in the folder. This means that it also works for isolated experiments.
    
7. **build_dataset** is a function that prepares the data base to be fed directly into a ML model for training and validation. There are a few import decisions taken here. The way this function is set up, it removes 2 rows of data per polariser cell studied. The reason why it was done is because we want the ML model to be able to predict polarization decay when given the specs of the cell (the static parameters) and the initial and final polarization values (with their associated time values). The reason why those two values are considered "known values" for each cell is beacuse they can be easily measured experimentally and they give as a good estimate about the overall behaviour. In some experiments, the environment of the studied sample is too fragile to move and place the Si crystal for polarization measurements. Therefore, a working ML algorithm whose inputs are the specs of the cell and the initial and final polarizations (measured before and after the sample is in place) would enable experiments that could not be done before without proper polarization efficiency corrections. 
Also, we remove those two values per polariser cell from the training arrays (Xt, y and err). The reason why it is done is to avoid data leaks in the model. If we give those values as training data and also give them as parameters, the ML algorithm can run into the risk of memorizing those pairs (over-fitting) and worsen any new predicitions. Uncertainties for the first and last polarization measurements are not added to the static features. This was a decision taken to avoid giving too much weight to two variables that don't have value on their own (they compliment the polarization values but if the ML architecture is as shallow as
the one used here, they can be considered independent variables and reduce the weight and importance of the other variables). 
Another consideration taken here was that the static features get duplicated in Xs a lot of times. One could think that using a similar method that the one used for augmentations of the data sets could also diversify the data base. However, we wanted all measurements of a same session and cell to be coherent 
and it wouldn't make sense to have different static features. Therefore, the safest approach was to only duplicate these values. For the augmented experiments the only parameters that are changed are the initial and final times and polarizations. There is no incompatibility here to what we have just said as these work as "hypothetical independent experiments". This is why augmentation is done before this function gets used.
         

8. **nll_loss** ML algorythms require a way to tell the algorythm if it is learning or not. The most standard practice is with a Loss function. If the loss value goes down that means that the algorythm is learning and, if a step increases the loss, then it is punished and tries other approaches. When using uncertainties when teaching the model, the most common loss function is the NLL or Negative log-likelihood of a normal distribution 
NLL$=\frac{1}{2}$log$(σ^2)+\frac{(y−μ)^2}{2σ^2}$
where $\sigma$ is the uncertainty in the predictions, $y$ is the measured value and $\mu$ the predicted value. Instead of predicting $σ^2$ directly, we obtain its logarithm to have a more stable process (and avoid accounting precision as $\sigma^{-2}$ which is numerically unstable when uncertainties are low).

However we want to avoid uncertainties that drift too far from the overall model predictions. To achieve that, we can get a rough estimate on what the uncertainty of a set predictions looks like.
Let $\vec{\mu}=\left(\mu_1,\ldots,\mu_N\right)^T$ be the vector of $N$ predicted values. It can be considered as a random vector of variance:
$Var(\vec{\mu})=\frac{1}{N}\sum_{i=1}^N\left(\mu_i-E(\vec{\mu})\right)$
where $E(\vec{\mu})$ is the mean of the predicted values. We then have two different variances, one obtained as the sparseness of the predictions, (denoted as $Var\left(\vec{\mu}\right)$, and one obtained as a result of the internal ML calculations (denoted as $\sigma^2$). A penalty can be added to the loss functions to force the model to try to reduce this differences. An easy way to model it is to obtain the difference of those variances and square the result (taking the absolute value was also a good estimate, but using squared values punished big discrepancies in a stronger way).

$StrayPenalty = B \cdot \left[\log\left(\sigma^2\right)-\log\left(\mathrm{Var}\left(\vec{\mu}\right)\right)\right]^2$
where $B$ is a constant used to control the weight of this penalty. The reason why $Var\left(\vec{\mu}\right)$ was used and not $Var\left(\vec{y}\right)$ (with $\vec{y}$ the vector of measured values) was to avoid noise in the original data to tamper with the loss function. It would be physically clearer to use measured values sparseness as a way to guide the model but some experimental uncertainties are clearly underestimated and that would cause this penalty to dominate the loss and obscure the main loss protocol, the NLL.

Also, we also want to punish the model if it tries overestimating $\sigma$. If the model is unable to minimize $y-\mu$, in order to lower NLL, it increases $\sigma$. If no precautions are taken, this "escape solution" achieves bad predictions with inflated uncertainties that simulate a low loss value. A new penalty was added that punishes overestimation of the uncertainties more than underestimation (which never happened). The slope correction done further on the pipeline can "fix" this issue but what the model returns then is closer to a poorly calculated linear fit
Therefore, an addition penalty was added.

$OverestimatePenalty= C \cdot \max\left(0, \log\left(\sigma^2\right)-\log\left(\mathrm{Var}(\vec{\mu})\right)\right)$
where $C$ is a constant used to control the weight of this penalty

9. **model\_fitting**. It is a function that logs and runs model.fit() on a two-input Keras model and returns the training history. It needs **static, time and polarization variables scaled**. Training is not done using the uncertainties of the data as it was decided that uncertainty information is encoded in the augmentations. Note: No validation is done anywhere in the code. Here are some of the reasons:
    
    9.1. The data base is very small. The amorphous data base contains only 199 points while the crystalline one contains 251. Removing a small percentage of those points for validation might leave the data base too small and underfitting might worsen the result more than fine tuning parametrs with validation.
    
    9.2. A randomized validation split may be physically wrong. Therefore it should be chronological, not shuffled. However, in crystalline experiments, there are decay experiments that have only four or five intermediate points. Even removing one point for validation is a massive hit on the experiment. Therefore, it is risky to add validation
    
    9.3. To find good models, a Leave-one-out approach was used. For a certain model structure, an experiment gets removed and the model and it trains on all the remaining experiments. Then, the model tries to predict this isolated experiment. Afterwards, the experiment is returned and a new one becomes isolated. This process loops for all experiments and an overall score of the model is computed. This process was done for 498 models for crystalline materials. This is a stronger (and more expensive) method than validation as it is not dependent on the validation splits and avoids possible information leaks.

Also, eight randomly picked models were tested with and without validation and with and without an asymetric uncertainty-overestimated penalizing loss. The result showed that the Loss update was an improvement and validation did not increase performance (without validation, the results were slightly better).

10. **model\_prediction**. It is a funtion that predicts with a given model. It needs **static and time variables scaled**. This scaling must be coherent to the one done in the rest of the funtions.

111. **train**. This function is the one responsible of scaling the inputs and training the model (it uses **model\_fitting**)

     11.1. It creates the independent arrays with all the encoded experiments (augmented or not) using build_dataset
    
     11.2. Then it scales the data. ML algorithms work better when the inputs and outputs are normalized. The reason why we don't normalize inside the function is to have those scaler defined globally and not locally
    
     11.3. It builds the model depending on the use\_uncertainty bool. (It changes the loss function and the output).
    
     11.4. It trains the model and returns its history (the trained model)
    
12. **align_static_vectors**. It converts the columns not present on an isolated experiment to zeros.
    
13. **model_predict_sloped** It substracts a linear function to the predicted values. If done correctly this makes it so that the polarization predictions at the initial and final time points are the same as the measured polarization values at those times. This fixes a vertical shift and also an overall slope. As it is a correction done with experimental values, the algorithm is still "universal". However we can't fully say that it is a pure ML algorithm. The "correctness" of this method is subjective. It is a warning in the ML front that there is an issue with the data base but it is a valid fix for experimentalists.
___________________________________________________________________________________________

Process it follows:

The steps it takes are explained here:
1. Obtain the absolute time references for the experiments (year, month, day, hour, minute and second)
2. Loop over all models that will be used
3. Loop over all two-numerical-rowed experiments. It copies them (does't move them) to the final folders
4. Build the structure that the ML model was trained with and scale all the experiment using the scalers that the model used in training
5. Predict, correct the slope if Correction=True, plot everything and write the results on a .txt file

Finally, it runs the _Predict_ function. The code will not progress until the user manually inputs the time values. It asks for three values, t_initial (which means that predictions start _t_initial_ seconds later (it can be negative)), _t_final_ (it changes the final time point for predictions by t_final seconds) and _t_step_ (the time steps between predictions)
It also takes all previous predictions of all models and averages them using a weighted mean. That way we have a single prediction file that should be the best that ML can currently do.
For every experiment, the code finds all the experiment files created by each model. Then, for every line it performs the following calculation. Let $P_1\left(t\right),\ldots,P_n\left(t\right)$ be the polarization predictions of models $1,\ldots,n$ at time $t$ with uncertainties $\sigma_1\left(t\right),\ldots,\sigma_n\left(t\right)$. Then the weighted mean is 

$P\left(t\right)=\frac{\sum_{i=1}^n P_i\left(t\right)\sigma_i^{-2}\left(t\right)}{\sum_{i=1}^n \sigma_i^{-2}\left(t\right)}$

with

$\sigma^2\left(t\right)=\frac{1}{\sum_{i=1}^n \sigma_i^{-2}\left(t\right)}$
