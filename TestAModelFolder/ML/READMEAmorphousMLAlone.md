<H1> AmorphousMLAlone.ipynb </H1>

**Warning. When loading a single experiment and Hot-encoding CellID the program won't know it it should encode one or three values. (brings up an error of shape mismatch)
WARNING VERY VERY IMPORTANT, IF A NEW CELL IS ADDED, TWO EXPERIMENTS WITH THAT CELL ARE REQUIRED OR HOT ENCODING WILL BREAK (AND THE PROGRAM WON'T PREDICT WITH ZERO EXAMPLES OF THAT CELL)**

This code requires that AmorphousFileLectureTests.ipynb has been run. As the data base is extremely small, the effects of overfitting and underfitting are very prone to happening. To test if a model performs as expected it should try to predict files that have not been used in the process of training and validation. As we can't extract too many files form the data base, the solution is to extract only one experiment, train a model with a given structure with all the other experiments, try to predict this isolated experiment (it is a blind prediction where we know what the output looks like) and repeat this entire process for all experiments. To compare the results of different models please use the code files inside LTestVisualCheck

__________________________________________________________________________________________

OUTPUTS OF THE CODE: 

1. **AmorphousLog\_Testing\_ML.txt**
A log file with every step that the algorithm has followed


2. **AmorphousExecution\_times.txt**
It times how long the code took to loop for each model to loop over all the isolation 


3. **AmorphousAllTestsFolder\_{Complexity}\_{num_augmentations}** 
For each Complexity and each num_augmentations (a.k.a, for each model structure and data organization) a folder is created. Inside there are folders for each isolated experiment iteration

    3.1 Missing{base\_name}
For each experiment in the data base (AmorphousMLDataBase) a folder is created and inside there are all the files that the code has created when training and predicting
 
        3.1.1 Difference\_{base\_name}.jpg
For each experimental point, we obtain the prediction and we obtain the difference between prediction and experimental values. It may be helpful to detect an overall deficiency when predicting

        3.1.2 Difference\_{base\_name}.txt
Has the same information as the last .jpg but written on a .txt file

        3.1.3 Missing\_{base\_name}.jpg
This is the important graph. It shows in black the experimental values with their uncertainty, in red the pure ML predictions (without the correction) with the uncertainty bands in a transparent red, in green the final predictions (ML + linear correction) with uncertainty as the green transparent band and in blue the corrected predictions for the time points where experimental values are known

        3.1.4 PredictedData\_PolarizationD3\_Missing{base\_name}.txt
The green part of the graph but in a .txt file

        3.1.5 PredictedPoints\_PolarizationD3\_Missing{base\_name}.txt
The blue part of the graph but in a .txt file

        3.1.6 RawData\_PolarizationD3\_Missing{base\_name}.txt
The black part of the graph but in a .txt file. It is supposed to be a duplicate of the file in AmorphousMLDataBase but it is stored to make sure that the scaling process is being done accurately (if it doesn't fit, be very very careful)

        3.1.7 model_PolarizationD3.keras
The model used in this isolated-experiment iteration

        3.1.8 scaler_static_PolarizationD3.pkl
The scaler for the static parameters (initial and final polarizations included) used in this isolated-experiment iteration

        3.1.9 scaler_time_PolarizationD3.pkl
The scaler for the time evolution used in this isolated-experiment iteration

        3.1.10 scaler_y_PolarizationD3.pkl
The scaler for the polarization values used in this isolated-experiment iteration


Here are all funtions and a overall guide of what the code does:

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


1. **load\_experiments**. It uses the directory where the array (numerical values) and parameter files resides, picks one polarization column and prepares a list of experiments to feed the ML algorithm. The output is as follows:
    
    *encoded\_experiments=[(static\_values, Deltatime, PolarizationD3, ErrPolarizationD3)$_{experiment 1}$,(static\_values, Deltatime, PolarizationD3, ErrPolarizationD3)$_{experiment 2}$,...]* 
    Note that the Cell Ids (polariser and analyser) are both Hot Encoded. The type of cell did not affect greatly the predictions. However in the future, it may be useful to give more information to the Cell_IDs. As of 2025, these strings are Hot Encoded which means that the code finds all the different types, creates columns for each type and writes 0 or 1 (bool) depending of whether the cell was from one type or the other. This is the standard procedure to feed categorical varaibles to ML.
    The size of the output list depends on the number of pairs of .tx files present in the folder. This means that it also works for isolated experiments
    
2. **augment_experiments** is a function that takes the original data list (as load_experiments returns it) and augments them _num\_augmentations_ times. The final legth of the list will be (num_augmentations +1 ) times the original length. Due to the reduced size of the data base (Not more than 30 experiments before 2026), techniques like augmentation are required. Due to the stochastic nature of neutron detection, most measurements, if repeated under the same conditions, will yield different results. Of course, all measurements converge (with uncertainty) to what we can call "the true value". Therefore, we can duplicate the experiments adding noise to the measurements to obtain "hypothetical" measurements that expand the data base. It was decided that the uncertainty won't be modified. To decide what type of noise could be applied two possibilities were considered. The first one was to suppose that sensor detection foloows a Poissonian distribution (law of rare events). The second one was to suppose that it follows a Normal Distribution of mean the measured value and width the uncertainty. As a poissonian distribution converges stochastically (in probability and distribution but not almost sure nor in L^p) to a normal distribution, it is safe to assume that they converge to the same result so the gaussian approach was selected. Also, the measurements are not raw counts but normalized values. Therefore the
    uncertainty is not the square root of the mean. As, we hope, that polarization measurements were done with more than 50 counts, the error made should be negligible.  

3. **build_dataset** is a function that prepares the data base to be fed directly into a ML model for training and validation. There are a few import decisions taken here. The way this function is set up, it removes 2 rows of data per polariser cell studied. The reason why it was done is because we want the ML model to be able to predict polarization decay when given the specs of the cell (the static parameters) and the initial and final polarization values (with their associated time values). The reason why those two values are considered "known values" for each cell is beacuse they can be easily measured experimentally and give as a good estimate about the overall behaviour. In some experiments, the environment of the studied sample is too fragile to move to place the Si crystal for polarization measurements. Therefore, a working ML algorithm whose inputs are the specs of the cell and the initial and final polarizations (measured before and after the sample is in place) would enable experiments that could not be done before without proper polarization efficiency corrections. 
Also, we remove those two values per polariser cell from the training arrays (Xt, y and err). The reason why it is done is to avoid data leaks in the model. If we give those values as training data and also give them as parameters, the ML algorithm can run into the risk of memorizing those pairs
(over-fitting) and worsen any new predicitions. 
Uncertainties for the first and last polarization measurements are not added to the static features. This was a decision taken to avoid giving too
much weight to two variables that don't have value on their own (they compliment the polarization values but if the ML architecture is as shallow as
the one used here, they can be considered independent variables and reduce the weight and importance of the other variables). 
Another consideration taken here was that the static features get duplicated in Xs a lot of times. One could think that using a similar method  that of the augmentation of the data sets could also diversify the data base. However, we wanted all measurements of a same session and cell to be coherent 
and it wouldn't make sense to have different static features. Therefore, the safest approach was to duplicate these values. For the augmented 
experiments the only parameters that are changed are the initial and final times and polarizations. There is no incompatibility here to what we
have just said as these work as "hypothetical independent experiments". This is why sugmentation is done before this function gets used.
         

4. **nll_loss** ML algorithms require a way to tell the algorithm if it is learning or not. The most standard practice is with a Loss function. If the loss value goes down that means that the algorithm is learning and, if a step increases the loss, then it is punished and tries other approaches. When using uncertainties when teaching the model, the most common loss function is the NLL or Negative log-likelihood of a normal distribution 
NLL $=\frac{1}{2}$log$(σ^2)+\frac{(y−μ)^2}{2σ^2}$
where $\sigma$ is the uncertainty in the predictions, $y$ is the measured value and $\mu$ the predicted value. Instead of predicting $σ^2$ directly, we obtain its logarithm to have a more stable process (and avoid accounting precision as $\sigma^{-2}$ which is numerically unstable when uncertainties are low).

However we want to avoid uncertainties that drift too far from the overall model predictions. To achieve that, we can get a rough estimate on what the uncertainty of a set predictions looks like.
Let $\vec{\mu}=\left(\mu_1,\ldots,\mu_N\right)^T$ be the vector of $N$ predicted values. It can be considered as a random vector of variance: $Var(\vec{\mu})=\frac{1}{N}\sum_{i=1}^N\left(\mu_i-E(\vec{\mu})\right)$
where $E(\vec{\mu})$ is the mean of the predicted values. We then have two different variances, one obtained as the sparseness of the predictions, (denoted as $Var\left(\vec{\mu}\right)$, and one obtained as a result of the internal ML calculations (denoted as $\sigma^2$). A penalty can be added to the loss functions to force the model to try to reduce this differences. An easy way to model it is to obtain the difference of those variances and square the result (taking the absolute value was also a good estimate, but using squared values punished big discrepancies in a stronger way).
$StrayPenalty = B \cdot \left[\log\left(\sigma^2\right)-\log\left(\mathrm{Var}\left(\vec{\mu}\right)\right)\right]^2$
where $B$ is a constant used to control the weight of this penalty. The reason why $Var\left(\vec{\mu}\right)$ was used and not $Var\left(\vec{y}\right)$ (with $\vec{y}$ the vector of measured values) was to avoid noise in the original data to tamper with the loss function. It would be physically clearer to use measured values sparseness as a way to guide the model but some experimental uncertainties are clearly underestimated and that would cause this penalty to dominate the loss and obscure the main loss protocol, the NLL.
Also, we also want to punish the model if it tries overestimating $\sigma$. If the model is unable to minimize $y-\mu$, in order to lower NLL, it increases $\sigma$. If no precautions are taken, this "escape solution" achieves bad predictions with inflated uncertainties that simulate a low loss value. A new penalty was added that punishes overestimation of the uncertainties more than underestimation (which never happened). The slope correction done further on the pipeline can "fix" this issue but what the model returns then is closer to a poorly calculated linear fit.
Therefore, an addition penalty was added.
$OverestimatePenalty= C \cdot \max\left(0, \log\left(\sigma^2\right)-\log\left(\mathrm{Var}(\vec{\mu})\right)\right)$
where $C$ is a constant used to control the weight of this penalty

5. **Define\_Complexity**. Given the name of a model it defines the model function of the ML algorithm. To be precise, it defines a function called _build_model_ every time _Define\_Complexity_ runs. If _Define\_Complexity_ gets run another time, it will define (possibly) another _build_model_ if the _Complexity_ variable changes. To find the best model, first two models were obtaines, one that overfits and another one that underfits. Then a series of intermediate models are built and tested. As ML model training has an important random aspect, there is no continuous transition from models with similar characteristics. Therefore, 185 models were tested whose names represent the main feature that defined them (NaifTwice1D have two Conv1D layers, SimpleNoDroput was the Simple model without the Dropout layer, etc). The numbers represent the size of the layers or a property of those layers (num_kernel for example).
                Complex, Simple, Naif, Naif834, Naif838, Naif8316, Naif8332, Naif854, Naif858, Naif8516,
                Naif8532, Naif8104, Naif8108, Naif81016, Naif81032, Naif1634, Naif1638, Naif16316, Naif16332,
                Naif1654, Naif1658, Naif16516, Naif16532, Naif16104, Naif16108, Naif161016, Naif161032, Naif2434,
                Naif2438, Naif24316, Naif24332, Naif2454, Naif2458, Naif24516, Naif24532, Naif24104, Naif24108,
                Naif241016, Naif241032, NaifTwice1D883316, NaifTwice1D883332, NaifTwice1D883516, NaifTwice1D883532,
                NaifTwice1D885316, NaifTwice1D885332, NaifTwice1D885516, NaifTwice1D885532, NaifTwice1D843316,
                NaifTwice1D843332, NaifTwice1D843516, NaifTwice1D843532, NaifTwice1D845316, NaifTwice1D845332
                NaifTwice1D845516, NaifTwice1D845532, NaifTwice1D483316, NaifTwice1D483332, NaifTwice1D483516,
                NaifTwice1D483532, NaifTwice1D485316, NaifTwice1D485332, NaifTwice1D485516, NaifTwice1D485532,
                NaifTwice1D443316, NaifTwice1D443332, NaifTwice1D443516, NaifTwice1D443532, NaifTwice1D445316,
                NaifTwice1D445332, NaifTwice1D445516, NaifTwice1D445532, NaifTwiceDense414, NaifTwiceDense418
                NaifTwiceDense4116, NaifTwiceDense4124, NaifTwiceDense814, NaifTwiceDense818, NaifTwiceDense8116,
                NaifTwiceDense8124, NaifTwiceDense1614, NaifTwiceDense1618, NaifTwiceDense16116, NaifTwiceDense16124,
                NaifTwiceDense2414, NaifTwiceDense2418, NaifTwiceDense24116, NaifTwiceDense24124, NaifTwiceDense424,
                NaifTwiceDense428, NaifTwiceDense4216, NaifTwiceDense4224, NaifTwiceDense824, NaifTwiceDense828,
                NaifTwiceDense8216, NaifTwiceDense8224, NaifTwiceDense1624, NaifTwiceDense1628, NaifTwiceDense16216,
                NaifTwiceDense16224, NaifTwiceDense2424, NaifTwiceDense2428, NaifTwiceDense24216, NaifTwiceDense24224,
                SimpleNoDropout444, SimpleNoDropout448, SimpleNoDropout484, SimpleNoDropout488, SimpleNoDropout4164,
                SimpleNoDropout4168, SimpleNoDropout844, SimpleNoDropout848, SimpleNoDropout884, SimpleNoDropout888,
                SimpleNoDropout8164, SimpleNoDropout8168, SimpleNoDropout1644, SimpleNoDropout1648, SimpleNoDropout1684,
                SimpleNoDropout1688, SimpleNoDropout16164, SimpleNoDropout16168, Simple414414, Simple414418,
                Simple414424, Simple414428, Simple414814, Simple414818, Simple414824, Simple414828, Simple418414,
                Simple418418, Simple418424, Simple418428, Simple418814, Simple418818, Simple418824, Simple418828,
                Simple424414, Simple424418, Simple424424, Simple424428, Simple424814, Simple424818, Simple424824,
                Simple424828, Simple428414, Simple428418, Simple428424, Simple428428, Simple428814, Simple428818,
                Simple428824, Simple428828, Simple814414, Simple814418, Simple814424, Simple814428, Simple814814,
                Simple814818, Simple814824, Simple814828, Simple818414, Simple818418, Simple818424, Simple818428,
                Simple818814, Simple818818, Simple818824, Simple818828, Simple824414, Simple824418, Simple824424,
                Simple824428, Simple824814, Simple824818, Simple824824, Simple824828, Simple828414, Simple828418,
                Simple828424, Simple828428, Simple828814, Simple828818, Simple828824, Simple828828



6. **model\_fitting**. It is a function that logs and runs model.fit() on a two-input Keras model and returns the training history. It needs **static, time and polarization variables scaled**. Training is not done using the uncertainties of the data as it was decided that uncertainty information is encoded in the augmentations. Note: No validation is done anywhere in the code. Here are some of the reasons:
    
    6.1. The data base is very small. The amorphous data base contains only 199 points while the crystalline one contains 251. Removing a small percentage of those points for validation might leave the data base too small and underfitting might worsen the result more than fine tuning parametrs with validation. However this has not been tested. A set of models should be ran with and without validation to check if it affects positively, negatively or if it doesn't matter. **Possible optimization of the algorithm**
    
    6.2. A randomized validation split may be physically wrong. Therefore it should be chronological, not shuffled. However, in crystalline experiments, there are decay experiments that have four or five intermediate points. Even removing one point for validation is a massive hit on the experiment. Therefore, it is risky to add validation
    
    6.3. To find good models, a Leave-one-out approach was used. For a certain model structure, an experiment gets removed and the model trains on all the remaining experiments. Then, the model tries to predict this isolated experiment. Afterwards, the experiment is returned and a new one becomes isolated. This process loops for all experiments and an overall score of the model is computed. This process was done for 498 models for amorphous materials. This is a stronger (and more expensive) method than validation as it is not dependent on the validation splits and avoids possible information leaks.

7. **model\_prediction**. It is a funtion that predicts with a given model. It needs **static and time variables scaled**. This scaling must be coherent to the one done in the rest of the funtions.

8. **train**. This function is the one responsible of scaling the inputs and training the model (it uses **model\_fitting**)

    8.1. It creates the independent arrays with all the encoded experiments (augmented or not) using build_dataset
    
    8.2. Then it scales the data. ML algorithms work better when the inputs and outputs are normalized. The reason why we don't normalize inside the function is to have those scaler defined globally and not locally
    
    8.3. It builds the model depending on the use_uncertainty bool. (It changes the loss function and the output).
    
    8.4. It trains the model and returns its history (the trained model)
    
9. **align_static_vectors**. It converts the columns not present on an isolated experiment to zeros.
    
10. **model_predict_sloped** It substracts a linear function to the predicted values. If done correctly this makes it so that the polarization predictions at the time points stored as static features are the same as the measured polarization values at those times. This fixes a vertical shift and also an overall slope. As it is a correction done with experimental values, the algorithm is still universal. However we can't fully say that it is a pure ML algorithm. The 'correctness' of this method is subjective. It is a warning in the ML front that there is an issue with the data base but it is a valid fix for experimentalists.

11. **isolated_experiments**. It does the entire Leave-One-Out logic and uses all the previous functions
