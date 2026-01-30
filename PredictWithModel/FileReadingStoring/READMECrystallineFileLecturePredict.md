<h1>CrystalineFileLecturePredict.ipynb</h1>

Reads from D3Files all the files it is going to process

# WE REQUEST THE USER TO GIVE THE FILE WITH ONLY TWO ROWS PER POLARISER CELL USED

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
### How to prepare the files
1. Take the raw .fli file and open it in a text reader app (The Note Bloc in Windows opens them like .txt files so it works). 
2. Choose a Miller index combination and find the first row that has as polarization directions (+z,+z)
3. Find the last appearance of that Miller index combination with polarization direction (+z,+z)
4. Erase all but those two lines and keep the header too.
5. Repeat for every cell used (for every 'polariser cell info' row)

The reason why this process has not been automated is to allow the user more freedom when predicting. Perhaps an automatic routine will not take into account what Miller index combination is adequate. Also, the user might have files that have already the correct structure and they don't need the whole pipeline. Either way, if the ILL staff wants this automated routine (using the same logic as the other Reading.ipynb files) please contact Gonzalo and he will happily help you.

__________________________________________________________________________________________

Outputs the following folders and files:

1. **CrystallineLog_Predicting_Creation.txt**
Logs all the prints and every step the code does. If you trust the code, it is irrelevant. If you don't trust it or want to change it then this txt file will tell you how each experiment file has been processed and where there might have been issues.


2. **Crystalline_CellID**
Contains the Cell IDs found on all the files. Needed for the ML code.


3. **CrystallineSeparatedFolder**
It will create a folder for each subfolder where .fli files, that could be used, were found. Only the numerical files are saved, a.k.a, *{base_name}.txt*. They contain DeltaTime (the time of the measurement measured from the first VALID polarization measurement), PolarizationD3, SoftPolarizationD3 (the polarization after using a Savitzky-Golay filter) and ErrPolarizationD3 (the uncertainty). The parameter files are directly sent to the next folder. Despite having duplicates of the information, this folder has been saved as it also has any intermediate file that has not been fully processed. If one of the .fli files has any issues, the file is saved, as is, before running to the issue. 

4. **ML/CrystallinePredictFiles**
This folder is not inside the folder you are currently working on (FileReadingStoring) but on another folder at the same level as FileReadingStoring called ML. It contains the files needed for the ML predictions.

    4.1 **{base\_name}.txt**
Contains DeltaTime (the time of the measurement measured from the first VALID polarization measurement), PolarizationD3, SoftPolarizationD3 (the polarization after using a Savitzky-Golay filter) and ErrPolarizationD3 (the uncertainty).

    4.2 **{base\_name}\_Parameters.txt**
Contains the CellID, Pressure, LabPolarization (the polarization measured at the lab) and LabTimeCellID (the time when it was measured)


5. **CrystallineDataBase**
Contains all the .fli files that were attempted to be read before manipulating them


6. **CrystallineBadFiles**
Contains all the .fli separated in experiment sets folders that were rejected (not enough points, negative polarizations, etc.)

7. **PolarizationTimeReference**
The models where trained using relative time only. This means that the first valid polarization measurement has been considered as time zero for each experiment and the rest of measurements have their associated time values as a variation of time since that reference. This way all experiments have the same structure. However this reference time is not absolute and the measurements of the diffractograms may have an absolute time reference different from the ones used in the models. By saving the string "Year-Month-Day Hour:Minute:Second" we can safely change the time reference


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

For every succesful experiment we will output:
1. Image:  **"PolarizationD3\_{folder\_name}\_{DD}/{MM}/{YY}\_{i}\_MillerIndex\_{PrettyCombination}\_Multiplier={Multiplier}.png"** in PlotResults. Shows the plot with the extended area with the raw data
2. Image: **"PolarizationD3\_{folder\_name}\_{DD}/{MM}/{YY}\_{i}\_MillerIndex\_{PrettyCombination}\_Multiplier={Multiplier}\_Soft.png"** in PlotResults. Shows the plot with the extended area with the filtered data
3. Image: **"PolarizationD3\_{folder_name}\_{DD}/{MM}/{YY}\_{i}\_MillerIndex\_{PrettyCombination}\_Filtered.txt\_plot\_Derivatives.png"** in PlotResults. Shows the evolution of the "derivatives". I apologize for the hideous names. Unless it results in a fatal error, I am scared to change the code.
4. Image: **"PolarizationD3\_{folder_name}\_{DD}/{MM}/{YY}\_{i}\_MillerIndex_{PrettyCombination}\_Filtered.txt\_N\_{N}\_ManualInterval.png"** in PlotResults. Shows the plot with the non-exteded area
5. Txt: **"PolarizationD3\_{folder\_name}\_{DD}/{MM}/{YY}\_{i}\_MillerIndex\_{PrettyCombination}.txt"** in MLDataBase. It contains the four data columns (DeltaTime, PolarizationD3, SoftPolarizationD3, ErrPolarizationD3)
6. Txt: **"PolarizationD3\_{folder\_name}\_{DD}/{MM}/{YY}\_{i}\_MillerIndex\_{PrettyCombination}\_Parameters.txt"** in MLDataBase. It contains the parameters (CellID, Pressure, LabPolarization, LabTime)


The plots are not necessary but are saved for the user to know what all the files look like. The txt files are fundamental for the rest of the pipeline. 
The files that are wrong or useless when all is done are the folowing. They are kept for  debug purposes (to see files with differents structures, why they fail,etc).

1. Txt: **"{folder\_name}\_Arrays\_{i}.txt"** in SeparatedFolder/{folder_name}. It still has the header and useless columns. It is the fli file of evey chunk, of every recorded experiment (correct or incorrect)
2. Txt: **"PolarizationD3\_{folder\_name}\_{DD}/{MM}/{YY}\_{i}\_MillerIndex\_{PrettyCombination}.txt"** in SeparatedFolder/{folder_name}. It is the same as the one in MLDataBase (a duplicate)
3. Image: **"PolarizationD3\_{folder\_name}\_{DD}/{MM}/{YY}\_{i}\_MillerIndex\_{PrettyCombination}.png"** in SeparatedFolder/{folder_name}. It plots (with error bars) PolarizationD3
4. Image: **"PolarizationD3\_{folder\_name}\_{DD}/{MM}/{YY}\_{i}\_MillerIndex\_{PrettyCombination}\_Combined.png"** in SeparatedFolder/{folder_name}. It plots (with error bars) both PolarizationD3 and SoftPolarizationD3
5. Image: **"PolarizationD3\_{folder\_name}\_{DD}/{MM}/{YY}\_{i}\_MillerIndex\_{PrettyCombination}\_Softened.png"** in SeparatedFolder/{folder_name}. It plots (with error bars) SoftPolarizationD3
6. Folder: **"FailuresTest"** contains all the graphs of the data sets that were considered not worthy but had more points that the ones saved. Check them if your experiment was not properly added
7. Folder: **"DataBase"** has the raw fli files. Once the code has been used they are no longer important (if you don't find the folder I may have added a line of code to erase it. Sorry in advance for any inconveniences)  


It erases all intermediate files and prepares the remaining ones for the ML pipeline

1. Removes all .fli files that have been created.
2. Removes empty folders
3. Collects all unique polariser–analyser ID pairs
 
As a result, the only useful files are _Crystalline_CellID.txt_ and the folder _ML/CrystallineToPredictFiles_
   60567   2.000   0.000   0.000 13/06/25 09:49:03     179.66  +z +z     0.8345    0.0018   11.0877    0.1322      60.00
   60886   2.000   0.000   0.000 15/06/25 07:23:31      75.02  +z +z     0.5979    0.0036    3.9735    0.0443      60.00
polariser cell info ge18004 pressure/init. polar 2.30 0.79 initial date/time 15 06 25 @ 10:40
   60888   2.000   0.000   0.000 15/06/25 10:49:27      75.02  +z +z     0.8467    0.0019   12.0495    0.1627      60.00
   60908   2.000   0.000   0.000 16/06/25 05:42:08      75.02  +z +z     0.8059    0.0024    9.3026    0.1256      60.00
