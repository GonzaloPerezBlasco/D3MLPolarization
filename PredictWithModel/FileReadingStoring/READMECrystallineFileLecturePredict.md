<h1>CrystalineFileLecturePredict.ipynb</h1>

<h2>Objective of program:</h2>

It takes all polarization files from D3Files and predicts the remaining points from those files. If the file has more than two points (initial and final) it will ignore the intermediate ones. The files created will be processed by this notebook:  *PredictWihModel/ML/PredictCrystalline.ipynb*

<h2>Input:</h2>

It looks in the folder *PredictWithModel/FileReadingStoring/D3Files* for zipped files with .fli files directly downloaded from the ILL Cloud (just download the entire experiment folder, zipped)

<h2>Output: </h2>

1. **CrystallineBadFiles**
Contains all the .fli separated in experiment sets folders that were rejected (negative polarizations, etc.)

2. **CrystallineDataBase**
Contains all the .fli files that were attempted to be read

3. **Crystalline_CellID**
Contains the Cell IDs found on all the files. Needed for the ML code.

4. **CrystallineLog_Predicting_Creation.txt**
Logs all the prints and every step the code does. If you trust the code, it is irrelevant. If you don't trust it or want to change it then this txt file will tell you how each experiment file has been processed and where there might have been issues.

5. **PredictWithModel\MLCrystallineToPredictFiles**
Contains the .txt files **NECESSARY** for the ML algorithm. There are two per experiment


    5.1 **{base\_name}.txt** 
Contains DeltaTime (the time of the first VALID polarization measurement), PolarizationD3, SoftPolarizationD3 (the polarization after using a Savitzky-Golay filter) and ErrPolarizationD3 (the uncertainty)

    5.2 **{base\_name}\_Parameters.txt**
Contains the CellID, Pressure, LabPolarization (the polarization measured at the lab) and LabTimeCellID (the time when it was measured)
Reads from D3Files all the files it is going to process

6. **PolarizationTimeReference**
The models where trained using relative time only. This means that the first valid polarization measurement has been considered as time zero for each experiment and the rest of measurements have their associated time values as a variation of time since that reference. This way all experiments have the same structure. However this reference time is not absolute and the measurements of the diffractograms may have an absolute time reference different from the ones used in the models. By saving the string "Year-Month-Day Hour:Minute:Second" we can safely change the time reference


_________________________________________________________________________________________
## Explanations

We should expect something like this:

|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|
polariser cell info |ge18004 |pressure/init. |polar |2.29 |0.79 |initial |date/time |17 |09 |23 |@ |10:39
|37391 |4.000 |0.000 |1.000 |18/09/23 |06:20:44 |155.03 |+z |+z |0.8391 |0.0156 |11.4270 |1.2031 |120.00
|37417 |4.000 |0.000 |1.000 |18/09/23 |10:31:59 |155.29 |+z |+z |0.8120 |0.0187 |9.6406 |1.0613 |120.00
polariser cell info |ge18012 |pressure/init. |polar |2.27 |0.79 |initial |date/time |18 |09 |23 |@ |10:33
|37418 |4.000 |0.000 |1.000 |18/09/23 |10:37:52 |155.29 |+z |+z |0.9101 |0.0107 |21.2483 |2.6375 |120.00
|37434 |4.000 |0.000 |1.000 |18/09/23 |14:16:07 |155.33 |+z |+z |0.8784 |0.0129 |15.4409 |1.7409 |120.00
polariser cell info |ge18004 |pressure/init. |polar |2.28 |0.79 |initial |date/time |22 |09 |23 |@ |09:45
|37462 |4.000 |0.000 |1.000 |22/09/23 |10:06:36 |0.00 |+z |+z |0.8670 |0.0427 |14.0333 |4.8278 |10.00
|37521 |4.000 |0.000 |1.000 |23/09/23 |09:51:06 |0.00 |+z |+z |0.7598 |0.0211 |7.3276 |0.7333 |120.00
|  ...  |       |       |       |          |          |      |        |        |        |        |        | | |

You can have as many sets of three lines as you desire, but they have to be a header and two regular rows per experiment. 





_________________________________________________________________________________________

Some parts of the code might use data from different sessions. It is safer to erase them and create all files from scratch everytime. This is not a big deal because this code file should only be run once unless the data base changes. It also clears all previous ML predictions to avoid mixing up information between experiments

The code will take all zipped folders from the folder _D3Files_ and prepare them to get their .fli files extracted.

First, it will check if there are duplicate zip folders. To check it, it will compare the folder name and the hash sha256. Duplicate folders will be erased. For more information about hash sha256 check for example:
>Wikipedia contributors. (2026, January 2). SHA-2. In Wikipedia, The Free Encyclopedia. Retrieved 10:49, January 17, 2026, from https://en.wikipedia.org/w/index.php?title=SHA-2&oldid=1330753870

Second, it will copy the contents of the zipped folders and create a new folder with the name of the experiment inside _D3Files_. No more zipped folders are erased and information gets duplicated.

Third, it will try to find all .fli files inside all the unzipped folders whether if they come from a zipped folder or not. It will then send them to the newly created folder _CrystalineDataBase_. If, for each experiment proposal there are more than one .fli files, they get a numeric suffix (\_1, \_2,...) to distinguish them. Afterwards, all unzipped folders get erased leaving behind only the non-duplicated zipped folders.

Note: All unzipped folders in D3Files will be explored, however they will get erased at the end of the pipeline. If you want them to persist for future runs of the code, they should be zipped first. For ILL users, when navigating the ILL Cloud, the easiest way to prepare the zip files is to download the _processed_ folder for each experiment proposal. The code is "smart" enough to only process .fli files with polarization information. Therefore, there is no need to manually prepare anything.
To be precise, this cell of code will take all the zip files, extract them nd remove duplicates using the file name And the hash sha256. 





An explanation of the information that all the contents in the .fli files give can be found here: 
1. 'polariser cell info' (str): Log of the installation of the polariser cells
2. 'PolariserID' (str): A string with the type of cell used
3. 'pressure/init. polar' (str): A string to introduce the $^\mathrm{3}$He gas pressure and the polarization measured at the creation lab.
4. 'PolariserPressure' (float): $^\mathrm{3}$He gas pressure in some units
5. 'InitialLabPolarization' (float): Polarization measured at the creation lab
6. 'initial date/time' (str): A string that introduces the day, month and year and the hour and minutes.
7. 'Date' (str): A string with the information DD/MM/YY
8. '@' (str): A string to separate date and time
9. 'time' (str): A string with the information HH:MM

And for the rest of the rows:
1. 'Measurement number' (int): The number index of the measurement.
2. 'First_Miller_Index' (float): The first Miller index of the crystal. Polarization is measured using a known Si Bragg crystal. For the source of the origin of the Si crystal see:
>Stunault, Anne & Vial, S & Pusztai, Laszlo & Cuello, Gabriel & Temleitner, László. (2016). Structure of hydrogenous liquids: separation of coherent and incoherent cross sections using polarised neutrons. Journal of Physics: Conference Series. 711. 012003. 10.1088/1742-6596/711/1/012003. 
3. 'Second_Miller_Index' (float)
4. 'Third_Miller_Index' (float):
5. 'Date' (str): A string with the information DD/MM/YY of that measurement
6. 'time' (str): A string with the information HH:MM:SS of that measurement
7. Temperature 
8. 'Direction_1' (str): It is the direction of the polarization after the monochromator.The direction +z corresponds to the orthogonal with respect to the floor pointing away from it. +x is the direction of the beam (variable) and +y is the orthogonal (positive orthonormal basis) direction to +z and +x. 
8. 'Direction_2' (str): The direction of the polarization at the sensors. True polarization measurements are done **only** on the (+z,+z) direction.

8. 'D3Polarization' (float): A float with the polarization measurement
9. 'ErrD3Polarization' (float): A float with the uncertainty of that polarization measurement
10. 'FlippingRatio' (float): The flipping ratio. Given either the flipping ratio or the polarization value, the other one is fully determined. Therefore, only one is needed and that is why we don´t work with the flipping ratio
11. 'ErrFlippingRatio' (float): The uncertainty of the flipping ratio
12. 'Elapsed time' (float): It is the time used to obtain the measurement (integration of the beam over that number of seconds)

Temperature did not seem to have an effect on the decay. Therefore, it has been eliminated in this code cell. Here is a summary of what the code does:

1. The code will go through the .fli files and find all rows with 'polariser cell info'. A cell change is considered once a new 'polariser cell info'. At the moment it ignores the experiments that use the 'magical box' as we are not sure if they are experiments compatible with the ones studied here.
2. For evey cell change, a new .fli file is created storing all the information including the header row and the measured data rows (in this case, just two). Also, all cell IDs are recorded
3. For all .fli files the code now will:   

    3.1 Remove unwanted rows (hopefully none)
    
    3.2 It removes any rows that don´t have polarization directions (+z,+z). (Should remove none if done correctly).
    
    3.3 Extract data form the header row.
    
    3.4 Remove unwanted columns (temperature, flipping ratio, counts, elapsed time,...).
    
    3.5 Set a time reference with the first measurement row. All other time values get referenced with respect to this moment in time and converted into seconds.
    
    3.6 Ignore all Miller index combinations that are not integers (hopefully no issues here).
    
    3.7 Save two files for each experiment. One with the header rows and another one with just the numeric rows (with a new header that explains what each column has).

It erases all intermediate files and prepares the remaining ones for the ML pipeline

1. Removes all .fli files that have been created.
2. Removes empty folders
3. Collects all unique polariser–analyser ID pairs
 
As a result, the only useful files are _Crystalline_CellID.txt_ and the folder _ML/CrystallineToPredictFiles_As a result, the only useful files are _Crystalline_CellID.txt_ and the folder _CrystallineMLDataBase_. However, the next notebook that needs to run will automatically read them from this folder so **DO NOT MOVE THIS FILE AND FOLDER**. 
<h3>After running this notebook please go to PredictWithModel\ML and run PredictCrystalline.ipynb</h3>
