# D3MLPolarization

**LINK TO THE PUBLISHED PAPER:**
The software is designed to function as a black-box tool for routine use on D3. In standard operation, users require only the predicted polarization decay for data correction and do not need to interact with the internal ML architecture. The prediction module can therefore be executed independently. For advanced applications, such as testing alternative model architectures or defining new ones, dedicated testing and model-creation modules are provided in the repository.

The procedure described at the methodology section of the paper is implemented as the testing module (left column), which evaluates combinations of model architectures and dataset augmentations. Specifically, Subsection about datasets corresponds to the *TestPreprocess* routine; Subsections Models and Renormalization are implemented in *TestML*; and Subsection Ranking is realized through the *Ranking* routine.

After selecting the optimal model set, the creation module (center column) retrains the chosen architectures for deployment. This stage follows the same pre-processing and training steps as the testing module, except that LOOCV is not applied. All available datasets are used for training without isolating individual experiments.

The prediction module (right column) applies the trained models to new datasets and returns the corresponding polarization decay curve. When executed in standalone mode, the production module uses the default ML models identified in this work, thereby preserving the intended black-box functionality. Re-execution of the testing and creation modules is required only if the training database is significantly expanded, for example through additional polarized neutron experiments on D3.


There are four folders each responsible for a different task. If you just want to obtain the predicted polarization decay you only have to use PredictWithModel. If you want to add measurements (after a reactor cycle) to the data base then you also need to use CreateModels. In case you want to see how well a model performs, use TestAModelFolder. At the end of this README there is a very short summary to use it if you don't need to understand what is happening.
In this presentation you can see how to use the programs:

[https://docs.google.com/presentation/d/1BqC0SIO63Y6Gf9Ebw5jM6ON_GJMAgEgx/edit?usp=sharing&ouid=103533666238364010404&rtpof=true&sd=true](https://docs.google.com/presentation/d/1ej2dq06_DOG1woB3bfhDSEdd2fq4od1S/edit?usp=sharing&ouid=103533666238364010404&rtpof=true&sd=true)



Brief Summary.

1. Go to PredictWithModel->FileReadingStoring->D3Files and paste your zip file with the "processed" data (straight from the ILL-DataBase) that you want to predict. Run AmorphousFileLecturePredict.ipynb if you used an amorphous substance or CrystalineFileLecturePredict.ipynb if it was an crystal or powder. If there is an error, install the python libraries.
2. Go to PredictWithModel->ML->ModelsAmorphous or ModelsCrystalline and check if there are models. If there are and they are the ones you want (we recommend Average_3 for both types of experiments), proceed to step 6
3. Go to CreateModels->FileReadingStoring (update D3files after each reactor cycle if there are new polarization decay experiments) and run AmorphousFileLectureCreate.ipynb or CrystalineFileLectureCreate.ipynb
4. Go to CreateModels->ML and run CreateSaveModelAmorphous.ipynb or CreateSaveModelCrystal.ipynb
5. Go to CreateModels->ML->ModelsAmorphous or ModelsCrystalline, copy the folders and paste them at PredictWithModel->ML->ModelsAmorphous or ModelsCrystalline
6. Go to PredictWithModel->ML and run PredictAmorphous.ipynb or PredictCrystalline.ipynb. The text file with the predicted polarization and a graph with the time evolution (green curve) will appear in PredictionsFolder inside each model subfolder and experiment sub-subfolder
7. Correct your data with the predicted polarizations (not part of this pipeline at the moment)
