# D3MLPolarization

**LINK TO THE PUBLISHED PAPER:**

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
