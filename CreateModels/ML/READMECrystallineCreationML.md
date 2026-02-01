<h1>CreateSaveModelCrystallineML.ipynb</h1> 

This code requires that CrystallineFileLectureCreate.ipynb has been run. If you are confident that the model structures are correct (go to TestAFolder to test the structures) then this code creates the model. It trains on ALL experiments that were processed with CrystallineFileLectureCreate.ipynb 

__________________________________________________________________________________________

OUTPUTS OF THE CODE: 

1. **CrystallineLogFileModelCreation.txt**
A log file with every step that the algorithm has followed


2. **CrystallineExecution\_times.txt**
It times how long the code took to create each model (1/2->1 minutes aproximately per model)


3. **ModelsCrystalline**
For each Complexity and each num_augmentations (a.k.a, for each model structure and data organization) a folder is created.

    3.1 **Model\_{Complexity}\_{num\_augmentations}** **THESE ARE THE FOLDERS YOU NEED TO COPY AND PASTE INSIDE THE PREDICTING CODE FOLDERS. THE WHOLE FOLDER NOT JUST THE CONTENTS**
Inside these folders you have the entire model and the scalers all ready to use for prediction. 
Note: The corrections are dependent on the static parameters and not the model or the training routine. Therefore these models will not give you the corrected predictions. (Do not worry, in the prediction files these corrections are automatically done if you don't change the flag that activates them)
 
        3.1.1 Model_{Complexity}_{num_augmentations}.keras The model file

        3.1.2 scaler_static_{Complexity}_{num_augmentations}_PolarizationD3.pkl The scaler for the static parameters (initial and final polarizations included) used in this isolated-experiment iteration)

        3.1.3 scaler_time_{Complexity}_{num_augmentations}_PolarizationD3.pkl The scaler for the time evolution used in this isolated-experiment iteration

        3.1.4 scaler_y_{Complexity}_{num_augmentations}.pkl The scaler for the polarization values used in this isolated-experiment iteration


__________________________________________________________________________________________

Process:

It takes two lists o
By changing the lists Names and Augmentations you can pick what model you create. Just write on the two lists your model names and the number of augmentations and it will create models combining them 
As an example,

Names = ["ModelA","ModelB","ModelC"]

Augmentations = [5,9,2]

will create the models "ModelA_5","ModelB_9" and "ModelC_2"

In order to do that, the code first prepares the folder structure and the folder where the model will be stored.
It then loops (if told to) over all polarization columns to create models for both.
It extracts all information from the data base, scales it and saves those scalers
Finally it trains the model and saves it in a distinct folder. That folder contains the scalers (in order to decipher how to scale it back to physical units) and the trained model.

Here are some explanations of what the code does in a more detiled manner:

1. **load\_experiments**. It uses the directory where the array (numerical values) and parameter files resides, picks one polarization column and prepares a list of experiments to feed the ML algorythm. The output is as follows:

    *encoded\_experiments=[(static\_values, Deltatime, PolarizationD3, ErrPolarizationD3)$_{experiment 1}$,(static\_values, Deltatime, PolarizationD3, ErrPolarizationD3)$_{experiment 2}$,...]* 
    
    Note that the Cell Id is Hot Encoded. The type of cell did not affect greatly the predictions. However in the future, it may be useful to give more information to Cell_ID. As of 2025, these strings are Hot Encoded which means that the code finds all the different types, creates columns for each type and writes 0 or 1 (bool) depending of whether the cell was from one type or the other. This is the standard procedure to feed categorical variables to ML.
    The size of the output list depends on the number of pairs of .txt files present in the folder. This means that it also works for isolated experiments.
    
2. **augment_experiments** is a function that takes the original data list (as load\_experiments returns it) and augments them _num\_augmentations_ times. The final legth of the list will be (num\_augmentations +1) times the original length. Due to the reduced size of the data base (Not more than 30 experiments before 2026), techniques like augmentation are required. Due to the stochastic nature of neutron detection, most measurements, if repeated under the same conditions, will yield different results. Of course, all measurements converge (with uncertainty) to what we can say is "the true value". Therefore, we can duplicate the experiments adding noise to the measurements to obtain "hypothetical" measurements that expand the data base. It was decided that the uncertainty won't be modified. To decide what type of noise could be applied two possibilities were considered. The first one was to suppose that sensor detection follows a Poisson distribution (law of rare events). The second one was to suppose that it follows a Normal Distribution of mean the measured value and width the uncertainty. As a Poisson distribution converges stochastically (in probability and distribution but not almost sure nor in $L^p$) to a normal distribution, it is safe to assume that they converge to the same result so the gaussian approach was selected. Also, the measurements are not raw counts but a function of them.
$P=\frac{n^+-n^-}{n^++n^-}$
We have no direct data of the number of counts (the parameter of a Posissonian distribution) but a rough estimate points to them being (worst case scenario) within the order of the millions (note that after 100 counts, poisson distributions vary very little from normal distributions). Threfore, count detection can be approximated to a normal distribution and a linear combination of random variables distributed as normal distributions is also a random variable with a normal distribution. Therefore, it is safe to assume that sensor measurements can be modelled after a gaussian distribution.


3. **build_dataset** is a function that prepares the data base to be fed directly into a ML model for training and validation. There are a few import decisions taken here. The way this function is set up, it removes 2 rows of data per polariser cell studied. The reason why it was done is because we want the ML model to be able to predict polarization decay when given the specs of the cell (the static parameters) and the initial and final polarization values (with their associated time values). The reason why those two values are considered "known values" for each cell is beacuse they can be easily measured experimentally and they give as a good estimate about the overall behaviour. In some experiments, the environment of the studied sample is too fragile to move and place the Si crystal for polarization measurements. Therefore, a working ML algorithm whose inputs are the specs of the cell and the initial and final polarizations (measured before and after the sample is in place) would enable experiments that could not be done before without proper polarization efficiency corrections. 
Also, we remove those two values per polariser cell from the training arrays (Xt, y and err). The reason why it is done is to avoid data leaks in the model. If we give those values as training data and also give them as parameters, the ML algorithm can run into the risk of memorizing those pairs (over-fitting) and worsen any new predicitions. Uncertainties for the first and last polarization measurements are not added to the static features. This was a decision taken to avoid giving too much weight to two variables that don't have value on their own (they compliment the polarization values but if the ML architecture is as shallow as
the one used here, they can be considered independent variables and reduce the weight and importance of the other variables). 
Another consideration taken here was that the static features get duplicated in Xs a lot of times. One could think that using a similar method that the one used for augmentations of the data sets could also diversify the data base. However, we wanted all measurements of a same session and cell to be coherent 
and it wouldn't make sense to have different static features. Therefore, the safest approach was to only duplicate these values. For the augmented experiments the only parameters that are changed are the initial and final times and polarizations. There is no incompatibility here to what we have just said as these work as "hypothetical independent experiments". This is why augmentation is done before this function gets used.
         

4. **nll_loss** ML algorythms require a way to tell the algorythm if it is learning or not. The most standard practice is with a Loss function. If the loss value goes down that means that the algorythm is learning and, if a step increases the loss, then it is punished and tries other approaches. When using uncertainties when teaching the model, the most common loss function is the NLL or Negative log-likelihood of a normal distribution 
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

5. **Define_Complexity** It consists of a single function called _Define\_Complexity_. Given the name of a model it defines the model function of the ML algorithm. To be precise, it defines a function called _build_model_ every time _Define\_Complexity_ runs. If _Define\_Complexity_ gets run another time, it will define (possibly) another _build_model_ if the _Complexity_ variable changes

Here we have functions that train, validate and fit the models. Some models require the variables to be scaled or will scale them. Extra precautions need to be taken into account

6. **model\_fitting**. It is a function that logs and runs model.fit() on a two-input Keras model and returns the training history. It needs **static, time and polarization variables scaled**. Training is not done using the uncertainties of the data as it was decided that uncertainty information is encoded in the augmentations. Note: No validation is done anywhere in the code. Here are some of the reasons:
    
    6.1. The data base is very small. The amorphous data base contains only 199 points while the crystalline one contains 251. Removing a small percentage of those points for validation might leave the data base too small and underfitting might worsen the result more than fine tuning parametrs with validation.
    
    6.2. A randomized validation split may be physically wrong. Therefore it should be chronological, not shuffled. However, in crystalline experiments, there are decay experiments that have only four or five intermediate points. Even removing one point for validation is a massive hit on the experiment. Therefore, it is risky to add validation
    
    6.3. To find good models, a Leave-one-out approach was used. For a certain model structure, an experiment gets removed and the model and it trains on all the remaining experiments. Then, the model tries to predict this isolated experiment. Afterwards, the experiment is returned and a new one becomes isolated. This process loops for all experiments and an overall score of the model is computed. This process was done for 498 models for crystalline materials. This is a stronger (and more expensive) method than validation as it is not dependent on the validation splits and avoids possible information leaks.

Also, eight randomly picked models were tested with and without validation and with and without an asymetric uncertainty-overestimated penalizing loss. The result showed that the Loss update was an improvement and validation did not increase performance (without validation, the results were slightly better).

7. **model\_prediction**. It is a funtion that predicts with a given model. It needs **static and time variables scaled**. This scaling must be coherent to the one done in the rest of the funtions.

8. **train**. This function is the one responsible of scaling the inputs and training the model (it uses **model\_fitting**)

    8.1. It creates the independent arrays with all the encoded experiments (augmented or not) using build_dataset
    
    8.2. Then it scales the data. ML algorithms work better when the inputs and outputs are normalized. The reason why we don't normalize inside the function is to have those scaler defined globally and not locally
    
    8.3. It builds the model depending on the use\_uncertainty bool. (It changes the loss function and the output).
    
    8.4. It trains the model and returns its history (the trained model)
    
9. **align_static_vectors**. It converts the columns not present on an isolated experiment to zeros.
    
10. **model_predict_sloped** It substracts a linear function to the predicted values. If done correctly this makes it so that the polarization predictions at the initial and final time points are the same as the measured polarization values at those times. This fixes a vertical shift and also an overall slope. As it is a correction done with experimental values, the algorithm is still "universal". However we can't fully say that it is a pure ML algorithm. The "correctness" of this method is subjective. It is a warning in the ML front that there is an issue with the data base but it is a valid fix for experimentalists.
