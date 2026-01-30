<h1>CrystalineFileLectureCreate.ipynb</h1>

Reads from D3Files all the files it is going to process

Outputs the following folders and files:

1. **CrystallineLog_Reading_Creation.txt**
Logs all the prints and every step the code does. If you trust the code, it is irrelevant. If you don't trust it or want to change it then this txt file will tell you how each experiment file has been processed and where there might have been issues.


2. **Crystalline_CellID**
Contains the Cell IDs found on all the files. Needed for the ML code.


3. **CrystallinephousPlotResults**
This folder will store the graphs of all the experiments that were accepted. Not needed for anything but it is nice to see the files that will be fed to the model. For each experiment you can find the following files:

    3.1 **Crystalline_Reading_Summary.txt** Shows the Score of each experiment. If you use the default criteria for deciding if a file is adequate then all files under 1.4 are rejected and the files that have been forcefully accepted or rejected appear as well indicated here. **IF YOU RECKON AN EXPERIMENT SHOULD BE ACCEPTED DESPITE BEING SHOWN HERE AS REJECTED PLEASE FIND THE LIST force\_reject\_files AND ADD YOUR ECPERIMENT USING THE SAME STRUCTURE. IDEM FOR ACCEPTED FILES THAT SHOULD NOT BE ACCEPTED**. As a side note, in order to know visually if a file is good or not, open the associated graph and just check that it doesn't do any funky business (see the examples already present to get a feel on what the word "funky" means)

    3.2 **{base\_name}\_N\_{N}.png**
It shows the values of the experiment in black, a linear fit in blue and the area needed for 75% of the points to be inside the rectangular region (the area inside $y_{min} = m x + n - N < y < m x + n + N < y_{max}$). Here you can find a good estimate on how good the overall decreasing tendencies are.

    3.3 **{base_name}_ExtendedArea.png** 
It shows the same pot and also the range $y_{min} = m x + n  - 1.3N < y < m x + n + 1.3N < y_{max}$. The points outside the green area will be discarded as they are considered to be too off to be considered correct.

    3.4 **{base_name}_Derivatives.png**
It shows the number of negative slopes between points and how steep they are


4. **CrystallineMLDataBase**
Contains the .txt files **NECESSARY** for the ML algorithm. There are two per experiment

    4.1 **{base_name}.txt** 
Contains DeltaTime (the time of the measurement measured from the first VALID polarization measurement), PolarizationD3, SoftPolarizationD3 (the polarization after using a Savitzky-Golay filter) and ErrPolarizationD3 (the uncertainty)

    4.2 **{base_name}_Parameters.txt**
Contains the CellID, Pressure, LabPolarization (the polarization measured at the lab) and LabTimeCellID (the time when it was measured)


5. **CrystallineFailuresTest**
Contains the plots 3.2 and 3.4 but for the experiments that failed the overall decreasing test. Check them if you can to see if any of your experiments has been placed there by mistake



6. **CrystallineDataBase**
Contains all the .fli files that were attempted to be read


7. **CrystallineBadFiles**
Contains all the .fli separated in experiment sets folders that were rejected (not enough points, negative polarizations, etc.)


_________________________________________________________________________________________


Some parts of the code might use data from different sessions. It is safer to erase them and create all files from scratch everytime. This is not a big deal because this code file should only be run once unless the data base changes. It also clears all previous ML predictions to avoid mixing up information between experiments

Some experiments did not pass the filtering methods of the previous functions despite looking very promising. Also, some experiments were not adequate yet they passed all of the filtering process. That is why we will store the names of those files manually.
The code will take all zipped folders from the folder _D3Files_ and prepare them to get their .fli files extracted.

First, it will check if there are duplicate zip folders. To check it it will compare the folder name and the hash sha256. Duplicate folders will be erased. For more information about hash sha256 check for example:
>Wikipedia contributors. (2026, January 2). SHA-2. In Wikipedia, The Free Encyclopedia. Retrieved 10:49, January 17, 2026, from https://en.wikipedia.org/w/index.php?title=SHA-2&oldid=1330753870

Second, it will copy the contents of the zipped folders and create a new folder with the name of the experiment inside _D3Files_. No more zipped folders are erased and information gets duplicated.

Third, it will try to find all .fli files inside all the unzipped folders whether if they come from a zipped file or not. It will then send them to the newly created folder _CrystallineDataBase_. If, for each experiment proposal there are more than one .fli files, they get a numeric suffix (\_1, \_2,...) to distinguish them. Afterwards, all unzipped folders get erased leaving behind only the non-duplicated zipped folders.

Note: All unzipped folders in D3Files will be explored, however they will get erased at the end of the pipeline. If you want them to persist for future runs of the code, they should be zipped first. **For ILL users, when navigating the ILL Cloud, the easiest way to prepare the zip files is to download the _processed_ folder for each experiment proposal. The code is "smart" enough to only process .fli files with polarization information. Therefore, there is no need to manually prepare anything.**

Some fli files have the wrong structure which means that they are not polarization measurements and if they are polarization files they may have used more than one polarization cell. Therefore, we need to remove all the non-polarization sets of data and separate the good fli files depending of the type of polarization cell they used.

For evey fli file we will read the contents and try to find the header (two strings in two consecutive lines). This symbolizes the installation of a new polarizer cell. If there are numerical values before the first header, that means that the process of saving the file occured before swapping the cell. Those data rows will be skipped. A correct fli file will have the following structure:

|  |  |  |  |  |  |  |  |  |  |  |  |  |  |
|--|--|--|--|--|--|--|--|--|--|--|--|--|--|
| polariser cell info | ge18004 | pressure/init. polar | 2.29 | 0.79 | initial date/time | 17/09/23 | @ | 10:39 |  |  |
| 47391 | 4.000 | 0.000 | 1.000 | 18/09/23 | 06:20:44 | 155.03 | +z | +z | 0.8391 | 0.0156 | 11.4270 | 1.2031 | 120.00 |
| 37392 | 4.000 | 0.000 | 1.000 | 18/09/23 | 06:26:49 | 155.05 | +x | +x | 0.8255 | 0.0110 | 10.4610 | 0.7211 | 300.00 |
|  ...  |       |       |       |          |          |      |        |        |        |        |        | | |






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
7. Temperature 
8. 'Direction_1' (str): It is the direction of the polarization after the monochromator.The direction +z corresponds to the orthogonal with respect to the floor pointing away from it. +x is the direction of the beam (variable) and +y is the orthogonal (positive orthonormal basis) direction to +z and +x. 
8. 'Direction_2' (str): The direction of the polarization at the sensors. True polarization measurements are done **only** on the (+z,+z) direction.

8. 'D3Polarization' (float): A float with the polarization measurement
9. 'ErrD3Polarization' (float): A float with the uncertainty of that polarization measurement
10. 'FlippingRatio' (float): The flipping ratio. Given either the flipping ratio or the polarization value, the other one is fully determined. Therefore, only one is needed and that is why we don´t work with the flipping ratio
11. 'ErrFlippingRatio' (float): The uncertainty of the flipping ratio
12. 'Elapsed time' (float): It is the time used to obtain the measurement (integration of the beam over that number of seconds)

Temperature did not seem to have an effect on the decay. Therefore, it has been eliminated in this code cell. Here is a summary of what the code does:

1. The code will go through the .fli files and find all combinations of consecutive rows with 'polariser cell info' and 'analyser cell info'. It doesn´t care about the order which makes the code more robust. We will consider that a polariser cell has been properly installed whrn both of these rows are present and that the cell has been changed once a new set of polariser and analyser rows are encountered. At the moment it ignores the experiments that use the 'magical box' as we are not sure if they are experiments compatible with the ones studied here.
2. For evey cell change, a new .fli file is created storing all the information including both polariser and analyser rows and the measured data rows. Also, all cell IDs are recorded
3. For all .fli files the code now will:   

- Remove unwanted rows
- It removes any rows that don´t have polarization directions (+z,+z)
- Extract data form the header rows (polariser row)
- Remove unwanted columns (temperature, flipping ratio, counts, elapsed time,...)
- Set a time reference with the first measurement row. All other time values get referenced with respect to this moment in time and converted into seconds.
- Ignore all Miller index combinations that are not integers
- Run through all Miller index combinations until one passes all the filters defined in previous code cells. When one is found, any other combinations are voided as well
- Plot the succesful experiments
- Save two files for each experiment. One with the header rows and another one with just the numeric rows (with a new header that explains what each column has)

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

Finally, it erases all intermediate files and prepares the remaining ones for the ML pipeline

1. Removes all .fli files that have been created.
2. Removes empty folders
3. Collects all unique polariser–analyser ID pairs
 
As a result, the only useful files are _Crystalline_CellID.txt_ and the folder _CrystallineMLDataBase_
