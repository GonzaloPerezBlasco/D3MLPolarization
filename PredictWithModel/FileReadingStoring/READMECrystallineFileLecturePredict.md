<h1>CrystalineFileLecturePredict.ipynb</h1>

Reads from D3Files all the files it is going to process

Outputs the following folders and files:

1. CrystallineLog_Predicting_Creation.txt
Logs all the prints and every step the code does. If you trust the code, it is irrelevant. If you don't trust it or want to change it then this txt file will tell you how each experiment file has been processed and where there might have been issues.


2. Crystalline_CellID
Contains the Cell IDs found on all the files. Needed for the ML code.


3. CrystallineSeparatedFolder
It will create a folder for each subfolder where .fli files that could be used were found. Only the numerical giles are saved, a.k.a, {base_name}.txt. They contain DeltaTime (the time of the measurement measured from the first VALID polarization measurement), PolarizationD3, SoftPolarizationD3 (the polarization after using a Savitzky-Golay filter) and ErrPolarizationD3 (the uncertainty). The parameter files are directly sent to the next folder. Despite having duplicates of the information, this folder has been saved as it also has any intermediate file that has not been fully processed. If one of the .fli files has any issues, the file is saved as is before running to the issue. 

4. ML/CrystallinePredictFiles
This folder is not inside the folder you are currently in (FileReadingStoring) but on another one at the same level called ML. It contains the files nneded for the ML predictions.

4.1 {base_name}.txt
Contain DeltaTime (the time of the measurement measured from the first VALID polarization measurement), PolarizationD3, SoftPolarizationD3 (the polarization after using a Savitzky-Golay filter) and ErrPolarizationD3 (the uncertainty).

4.2 {base_name}_Parameters.txt
Contains the CellID, Pressure, LabPolarization (the polarization measured at the lab) and LabTimeCellID (the time when it was measured)


5. CrystallineDataBase
Contains all the .fli files that were attempted to be read before manipulating them


6. CrystallineBadFiles
Contains all the .fli separated in experiment sets folders that were rejected (not enough points, negative polarizations, etc.)

7. PolarizationTImeReference
The models where trained using relative time only. This means that the first valid polarization measurement has been considered as time zero for each experiment and the rest of measurements have their associated time values as a variation of time since that reference. This way all experiments have the same structure. However this reference time is not absolute and the measurements of the diffractograms may have an absolute time reference different from the ones used in the models. By saving the string "Year-Month-Day Hour:Minute:Second" we can safely change the time reference


_________________________________________________________________________________________

Process it follows:

1. REMOVAL OF PREVIOUS ITERATIONS
To avoid leaks and duplications, all files are erased before running the code file

2. ZIP FOLDER TREATMENT
The code will take all the zip files, extract them, remove duplicates using the name AND hash sha256.

3. SEPARATION OF FLI FILES ACCORDING TO EXPERIMENTS

Some fli files have the wrong structure (they are not polarization measurementes) and if they are polarization files they can have more than one experiment per file.
For evey fli file we will read the contents and try to find the header (a string in an entire line). This symbolizes the beginning of an experiment
If there are numerical values before the first header, that means that the process of saving the file occured before changing something of the experiment. These data rows will be skipped
A correct fli file will have the following structure:
    polariser cell info ge18004 pressure/init. polar 2.29 0.79 initial date/time 17 09 23 @ 10:39
    37391   4.000   0.000   1.000 18/09/23 06:20:44     155.03  +z +z     0.8391    0.0156   11.4270    1.2031     120.00
    37392   4.000   0.000   1.000 18/09/23 06:26:49     155.05  +x +x     0.8255    0.0110   10.4610    0.7211     300.00
    ...

Which corresponds to the following information:
    String:'polariser cell info', CellID, String:'pressure/init. polar', Pressure(unknown units), InitialLabPolarization, String:'date/time', Day, Month, Year, String:'@', Hour:Minute
    Measurement Number, First Miller Index, Second Miller Index, Third Miller Index, Date Of Measurement, Time Of Measurement, Temperature [Kelvin],
                        Direction Of Polarization In The First Polarizer Cell (Direction of the quantum operator S_x,S_y,S_z), Direction Of Polarization In The Second Polarizer Cell,
                        Polarization, Polarization uncertainty, Flipping Ratio, FlippingRatio Uncertainty, Duration of the measurement

The direction +z is chosen to be pointing away from the ground.
The direction +x is the direction of the flow of neutrons, i.e, the direction of Scattering.
The direction +y is the orthogonal to both of them.
D3 uses two polariser cells, one between the reactor and the sample and a second between the sample and the sensor. The first one guarantees that only neutrons with the correct spin direction
interacts with the sample. The second one guarantees that only the neutrons that have unchanged spin direction after interacting with the sample are detected by the sensor. This is
the reason why the directions (+z,+y,+x,-z,-y,-x) appear twice.
We have considered that temperature is not a relevant factor and the flipping ratio has no new information that polarization alrady posseses.
First, the code will first locate the first header (ignoring eveything before) and save all the data afterwards (until the next header or end of the document) in a file with the suffix Array_{i} (i is the number of headers already processed in that fli file)
Second, it will save the header as a file with the suffix Parameters.
Third, the header row and the columns of Measurement Number, Temperature, Flipping Ratio, FlippingRatio Uncertainty and Time Between Measurements will be erased
Fourth, as all data measurement uses the +z,+z combination, all other combinations are erased
Fifth, not all data from all Miller Index combinations are polarization measurements. Even some of the ones that are polarization measurements are tampered (playing with magnetic fields for example).
This means that there needs to be a way to select the correct combination. For starters, irrational Miller indices are not used for measurements with the samples (they need to be discarded)
The integer Miller indices combination will be put to the test by all the functions defined before.
Sixth, It computes a score depending on how many derivatives are negative, (200 / (200 - percent_neg) - 1) to be precise. This is a normalized score (0-1) with a 1/x evolution. Also it computes a score depending of the size of N, 2 * (-0.5 + 1 / (1 + needed_N / 8.54e-2)) to be precise. 8.54e-2 is the maximum of the data set. If a new maximum is achieved, the score wont be normalized (0-1) but won't break. The final score combines both of these values (addition). Manually I have seen that 1.4 is a good threshold. If Score>1.4 the file will be accepted. If it is smaller it will be discarded by the main code (a False will be returned). It does the m<0 test, writes everything in Summary_txt (filename, Score associated with N, Score associated with Derivatives nad the total Score). If the file was chosen to be forcefully accepted or rejected, a string will appear in the .txt file. Finally it will save plots of both filters in PlotResults if it is correct and in FailureTest if it is considered a bad file. Again, if a new file is added it may be wise to check your experiment in these folders. For more info read the description of FilteringMethodInt = 0 inside the code
Seventh, it will try each Miller index combination for a set i value, apply a filter, and return filtered df + PrettyCombination. it will only add the filtered column if enough points & data changes significantly. If it doesn't change too much, the column PolarizationD3 will be duplicated with the new name
Eight, it repeats the process of obtaining the area in m*x+n-N < y < m*x+n+N where 75% of the points are inside the area. It also multiplies the value of N by a factor AcceptableMultiplier and erases all points outisde this bigger area. As uncertainties are clearly underestimated I tried to make them reasonable (looking at the dispersion of the points it is clear there is systematic uncertainties. Under the hypothesis that the polarization curve should be a soft curve (at least C^1) we will try to use χ^2 to add a provisional uncertainty margin fitting to a linear expression. This is a very inaccurate uncertainty increase but it is an improvement of the underestimated uncertainties (and the lack of ways to quantify the systematic uncertainty sources). The enlarged area will be plotted and saved in PlotResults for both the normal data and the softened data (Savitzky–Golay filter)    
    
4. CellID SAVING
It will safely store in a txt file all the cell ids so that the code in ML can use them

5. DUPLICATION REMOVAL
It will check if the files created for the ML algorithm are duplicates and erases them in that case


________________________________________________________________________________________

VERY VERY IMPORTANT

The other code files should not have an issue with the structure inside the fli files as long as you download them from the online data base and keep them zipped. However here we expect a very specific structure. PLEASE HAVE THE FILES INSIDE THE ZIP FILES HAVE THE FOLLOWING STRUCTURE (perhaps in the future we can change it so it is also automatic).
1. Only two rows of numerical values per 'polariser cell info' row
2. The same miller indices for those two rows
3. Direction +z,+z
Here you have an example:

polariser cell info ge18004 pressure/init. polar 2.30 0.78 initial date/time 06 06 25 @ 15:15
   59685   0.000   0.000   2.000 06/06/25 16:20:21     228.88  +z +z     0.8460    0.0007   11.9879    0.0558     180.00
   60273   0.000   0.000   2.000 10/06/25 15:36:07     170.95  +z +z     0.5454    0.0023    3.3998    0.0215      60.00
polariser cell info ge18012 pressure/init. polar 2.30 0.78 initial date/time 10 06 25 @ 15:45
   60275   0.000   0.000   2.000 10/06/25 15:47:23     170.14  +z +z     0.8363    0.0011   11.2157    0.0764      60.00
   60343   0.000   0.000   2.000 11/06/25 14:30:33     169.83  +z +z     0.8016    0.0021    9.0804    0.1047      60.00
polariser cell info ge18004 pressure/init. polar 2.30 0.78 initial date/time 11 06 25 @ 14:30
   60345   2.000   0.000   0.000 11/06/25 14:33:57     169.83  +z +z     0.8019    0.0021    9.0935    0.1039      60.00
   60565   2.000   0.000   0.000 13/06/25 09:43:00     179.66  +z +z     0.6776    0.0033    5.2033    0.0629      60.00
polariser cell info ge18012 pressure/init. polar 2.30 0.78 initial date/time 13 06 25 @ 09:44
   60567   2.000   0.000   0.000 13/06/25 09:49:03     179.66  +z +z     0.8345    0.0018   11.0877    0.1322      60.00
   60886   2.000   0.000   0.000 15/06/25 07:23:31      75.02  +z +z     0.5979    0.0036    3.9735    0.0443      60.00
polariser cell info ge18004 pressure/init. polar 2.30 0.79 initial date/time 15 06 25 @ 10:40
   60888   2.000   0.000   0.000 15/06/25 10:49:27      75.02  +z +z     0.8467    0.0019   12.0495    0.1627      60.00
   60908   2.000   0.000   0.000 16/06/25 05:42:08      75.02  +z +z     0.8059    0.0024    9.3026    0.1256      60.00
