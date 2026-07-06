<h1>AmorphousFileLectureCreate.ipynb</h1>

<h2>Objective of program:</h2>

It takes all polarization files from D3Files and predicts the remaining points from those files. If the file has more than two points (initial and final) it will ignore the intermediate ones. The files created will be processed by this notebook: *PredictWithModel/ML/PredictAmorphous.ipynb*

<h2>Input:</h2>

It looks in the folder *PredictWithModel/FileReadingStoring/D3Files* for zipped files with .fli files directly downloaded from the ILL Cloud (just download the entire experiment folder, zipped)
<h2>Output: </h2>

1. **AmorphousBadFiles**
Contains all the .fli separated in experiment sets folders that were rejected (negative polarizations, etc.)

2. **AmorphousDataBase**
Contains the extracted .fli files

3. **AmorphousSeparatedFolder**
Contains the extracted .fli files and the processed files

4. **AmorphousCell_ID.txt**
    Placeholder txt file
5.  **AmorphousLog\_Predicting\_Creation.txt**
Logs all the prints and every step the code does. If you trust the code, it is irrelevant. If you don't trust it or want to change it then this txt file will tell you how each experiment file has been processed and where there might have been issues.

6. **AmorphousPolariserAndAnalyser_IDs**
Contains the Cell IDs found on all the files. Needed for the ML code.

7. **PolarizationTimeReference**
The models where trained using relative time only. This means that the first valid polarization measurement has been considered as time zero for each experiment and the rest of measurements have their associated time values as a variation of time since that reference. This way all experiments have the same structure. However this reference time is not absolute and the measurements of the diffractograms may have an absolute time reference different from the ones used in the models. By saving the string "Year-Month-Day Hour:Minute:Second" we can safely change the time reference

8. **PredictWithModel\ML\AmorphousToPredictFiles**
    Contians the files used for prediction. They are automatically moved to the ML folder

   8.1 **{base\_name}.txt** 
Contains DeltaTime (the time of the first VALID polarization measurement), PolarizationD3, SoftPolarizationD3 (the polarization after using a Savitzky-Golay filter) and ErrPolarizationD3 (the uncertainty)

    8.2 **{base\_name}\_Parameters.txt**
Contains the CellID, Pressure, LabPolarization (the polarization measured at the lab) and LabTimeCellID (the time when it was measured) for both cells (analyser and polariser)
   



_________________________________________________________________________________________

## Explanations


First, it will check if there are duplicate zip folders. To check it it will compare the folder name and the hash sha256. Duplicate folders will be erased. For more information about hash sha256 check for example:
>Wikipedia contributors. (2026, January 2). SHA-2. In Wikipedia, The Free Encyclopedia. Retrieved 10:49, January 17, 2026, from https://en.wikipedia.org/w/index.php?title=SHA-2&oldid=1330753870

Second, it will copy the contents of the zipped folders and create a new folder with the name of the experiment inside _D3Files_. No more zipped folders are erased and information gets duplicated.

Third, it will try to find all .fli files inside all the unzipped folders whether if they come from a zipped file or not. It will then send them to the newly created folder _AmorphousDataBase_. If, for each experiment proposal there are more than one .fli files, they get a numeric suffix (\_1, \_2,...) to distinguish them. Afterwards, all unzipped folders get erased leaving behind only the non-duplicated zipped folders.

Note: All unzipped folders in D3Files will be explored, however they will get erased at the end of the pipeline. If you want them to persist for future runs of the code, they should be zipped first. **For ILL users, when navigating the ILL Cloud, the easiest way to prepare the zip files is to download the _processed_ folder for each experiment proposal. The code is "smart" enough to only process .fli files with polarization information. Therefore, there is no need to manually prepare anything.**


Some fli files have the wrong structure which means that they are not polarization measurements and if they are polarization files they may have used more than one polarization cell. Therefore, we need to remove all the non-polarization sets of data and separate the good fli files depending of the type of polarization cell they used.

For evey fli file we will read the contents and try to find the header (two strings in two consecutive lines). This symbolizes the installation of a new polarizer cell. If there are numerical values before the first header, that means that the process of saving the file occured before swapping the cell. Those data rows will be skipped. A correct fli file will have the following structure:

|  |  |  |  |  |  |  |  |  |  |  |  |
|--|--|--|--|--|--|--|--|--|--|--|--|
| polariser cell info | ge18004 | pressure/init. polar | 2.30 | 0.79 | initial date/time | 21/11/23 | @ | 12:45 |
| analyser cell info  | sic1402 | pressure/init. polar | 2.00 | 0.79 | initial date/time | 21/11/23 | @ | 12:45 |
| 40661 | 3.000 | 3.000 | 3.000 | 21/11/23 | 12:50:35 | 0.00 | 0.7890 | 0.0020 | 8.4795 | 0.0897 | 120.00 |
| 40662 | 3.000 | 3.000 | 3.000 | 21/11/23 | 12:54:43 | 0.00 | 0.7851 | 0.0020 | 8.3048 | 0.0867 | 120.00 |
|  ...  |       |       |       |          |          |      |        |        |        |        |        |

Which corresponds to the following information for the first two rows:
1. 'polariser cell info'/'analyser cell info' (str): Log of the installation of the first and second polariser cells
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
7. Unknown float, maybe Temperature
8. 'D3Polarization' (float): A float with the polarization measurement
9. 'ErrD3Polarization' (float): A float with the uncertainty of that polarization measurement
10. 'FlippingRatio' (float): The flipping ratio. Given either the flipping ratio or the polarization value, the other one is fully determined. Therefore, only one is needed and that is why we don´t work with the flipping ratio
11. 'ErrFlippingRatio' (float): The uncertainty of the flipping ratio
12. 'Elapsed time' (float): It is the time used to obtain the measurement (integration of the beam over that number of seconds)

Temperature did not seem to have an effect on the decay. Therefore, it has been eliminated in this code cell. Here is a summary of what the code does:

1. The code will go through the .fli files and find all combinations of consecutive rows with 'polariser cell info' and 'analyser cell info'. It doesn´t care about the order which makes the code more robust. We will consider that a polariser cell has been properly installed whrn both of these rows are present and that the cell has been changed once a new set of polariser and analyser rows are encountered. At the moment it ignores the experiments that use the 'magical box' as we are not sure if they are experiments compatible with the ones studied here
2. For evey cell change, a new .fli file is created storing all the information including both polariser and analyser rows and the measured data rows. Also, all cell IDs are recorded
3. For all .fli files the code now will:
    
- Remove unwanted rows
- Extract data form the header rows (polariser row and analyser row)
- Remove unwanted columns
- Set a time reference with the first measurement row. All other time values get referenced with respect to this moment in time and converted into seconds.
- Ignore all Miller index combinations that are not integers
- Run through all Miller index combinations until one passes all the filters defined in previous code cells
- Plot the succesful experiments
- Save two files for each experiment. One with the header rows and another one with just the numeric rows (with a new header that explains what each column has)

Finally, it erases all intermediate files and prepares the remaining ones for the ML pipeline

1. Removes all .fli files that have been created.
2. Removes empty folders
3. Collects all unique polariser–analyser ID pairs
 
As a result, the only useful files are _AmorphousPolariserAndAnalyser\_IDs.txt_ and the folder _\PredictWithModel\ML\AmorphousToPredictFiles_

Finally, it sends the files to the proper folders for the ML algorithm. This part is unique in all "Lecture" files. Clear AmorphousToPredictFiles, then move all files from AmorphousMLDataBase to AmorphousToPredictFiles,
finally erase AmorphousMLDataBase.  However, the next notebook that needs to run will automatically read them from this folder so **DO NOT MOVE THIS FILE AND FOLDER**. 

<h3>After running this notebook please go to PredictWithModel\ML and run PredictAmorphous.ipynb</h3>
