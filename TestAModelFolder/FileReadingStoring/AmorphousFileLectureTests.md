<h1>AmorphousFileLectureCreate.ipynb</h1> 

Reads from D3Files all the files it is going to process

Outputs the following folders and files:

1. AmorphousLog_Reading_Creation.txt
Logs all the prints and every step the code does. If you trust the code, it is irrelevant. If you don't trust it or want to change it then this txt file will tell you how each experiment file has been processed and where there might have been issues.


2. Amorphous_CellID
Contains the Cell IDs found on all the files. Needed for the ML code.


3. AmorphousPlotResults
This folder will store the graphs of all the experiments that were accepted. Not needed for anything but it is nice to see the files that will be fed to the model. For each experiment you can find the following files:

3.1 {base_name}_ExtendedArea.png 
It shows the same pot and also the range y_min=mx+n-1.3*N<y<mx+n+1.3*N<y_max. The points outside the green area will be discarded as they are considered to be too off to be considered correct.


4. AmorphousMLDataBase
Contains the .txt files NECESSARY for the ML algorithm. There are two per experiment

4.1 {base_name}.txt 
Contains DeltaTime (the time of the measurement measured from the first VALID polarization measurement), PolarizationD3, SoftPolarizationD3 (the polarization after using a Savitzky-Golay filter) and ErrPolarizationD3 (the uncertainty)

4.2 {base_name}_Parameters.txt
Contains the CellID, Pressure, LabPolarization (the polarization measured at the lab) and LabTimeCellID (the time when it was measured)


5. AmorphousFailuresTest
Contains the plots 3.2 and 3.4 but for the experiments that failed the overall decreasing test. Check them if you can to see if any of your experiments has been placed there by mistake



6. AmorphousDataBase
Contains all the .fli files that were attempted to be read


7. AmorphousBadFiles
Contains all the .fli separated in experiment sets folders that were rejected (not enough points, negative polarizations, etc.)


_________________________________________________________________________________________

Process it follows:

1. REMOVAL OF PREVIOUS ITERATIONS
To avoid leaks and duplications, all files are erased before running the code file

2. ZIP FOLDER TREATMENT
The code will take all the zip files, extract them, remove duplicates using the name AND hash sha256.

3. SEPARATION OF FLI FILES ACCORDING TO EXPERIMENTS

Some fli files have the wrong structure (they are not polarization measurementes) and if they are polarization files they can have more than one experiment per file.
For evey fli file we will read the contents and try to find the header (two strings in two consecutive lines). This symbolizes the beginning of an experiment
If there are numerical values before the first header, that means that the process of saving the file occured before changing something of the experiment. These data rows will be skipped
A correct fli file will have the following structure:
polariser cell info ge18004 pressure/init. polar 2.30 0.79 initial date/time 21 11 23 @ 12:45
analyser cell info sic1402 pressure/init. polar 2.00 0.79 initial date/time 21 11 23 @ 12:45
   40661   3.000   3.000   3.000 21/11/23 12:50:35       0.00        0.7890    0.0020    8.4795    0.0897     120.00
   40662   3.000   3.000   3.000 21/11/23 12:54:43       0.00        0.7851    0.0020    8.3048    0.0867     120.00
    ...

Which corresponds to the following information:
    String:'polariser cell info', PolariserID, String:'pressure/init. polar', PolariserPressure(unknown units), InitialLabPolarization, String:'date/time', Day, Month, Year, String:'@', Hour:Minute
    String:'analyser cell info', AnalyserID, String:'pressure/init. polar', AnalyserPressure(unknown units), InitialLabPolarization(always the same as the polariser cell), String:'date/time', Day, Month, Year, String:'@', Hour:Minute
    Measurement Number, First Miller Index, Second Miller Index, Third Miller Index, Date Of Measurement, Time Of Measurement, Unkown Number, Polarization, Polarization uncertainty, Flipping Ratio, FlippingRatio Uncertainty, Duration of the measurement

D3 uses two polariser cells, one between the reactor and the sample and a second between the sample and the sensor. The first one guarantees that only neutrons with the correct spin direction interacts with the sample. The second one guarantees that only the neutrons that have unchanged spin direction after interacting with the sample are detected by the sensor. The same happens with the analyser cells. The reason why this values are smaller than the ones measured with crystals or crystalline powders is that this polarizations are the multiplication of two polarizations. We have considered that temperature is not a relevant factor and the flipping ratio has no new information that polarization alrady posseses.
First, the code will first locate the first header (ignoring eveything before) and save all the data afterwards (until the next header or end of the document) in a file with the suffix Array_{i} (i is the number of headers already processed in that fli file).
Second it will try to find a combination of the strings with polariser and anlyser (there have been files where one comes before the other, others the other way around and some where the lab polarization.
changes in the same cell in mere minutes. As there is not necessarily an absolute order, the code has become flexible to fix these issues)
Third, it will save the headers as a file with the suffix Parameters.
Fourth, the header row and the columns of Measurement Number, Unkown Number, Flipping Ratio, FlippingRatio Uncertainty and Time Between Measurements will be erased
Fifth, not all data from all Miller Index combinations are polarization measurements. Even some of the ones that are polarization measurements are tampered (playing with magnetic fields for example).
This means that there needs to be a way to select the correct combination. For starters, irrational Miller indices are not used for measurements with the samples (they need to be discarded)
The integer Miller indices combination will be put to the test by all the functions defined before.
For evey succesful experiment we will output:
    Image:  "PolarizationD3_{folder_name}_{DD}/{MM}/{YY}_{i}_MillerIndex_{PrettyCombination}_ExtendedArea.png" in AmorphousPlotResults. Shows the plot with the extended area with the raw data
    Txt:    "PolarizationD3_{folder_name}_{DD}/{MM}/{YY}_{i}_MillerIndex_{PrettyCombination}.txt" in AmorphousMLDataBase. It contains the four data columns (DeltaTime, PolarizationD3, SoftPolarizationD3, ErrPolarizationD3)
    Txt:    "PolarizationD3_{folder_name}_{DD}/{MM}/{YY}_{i}_MillerIndex_{PrettyCombination}_Parameters.txt" in AmorphousMLDataBase. It contains the parameters (CellID, Pressure, LabPolarization, LabTime)
These plots are not necessary but are saved for the user to know what all the files look like.
The files that are wrong or useless when all is done are the folowing:
    Txt:    "{folder_name}_Arrays_{i}.txt" in SeparatedFolder/{folder_name}. It still has the header and useless columns. It is the fli file of evey chunk, of every recorded experiment (correct or incorrect)
    Folder: "BadTest" contains all the graphs of the data sets that were considered not worthy but had more points that the ones saved. Check them if your experiment was not properly added.

No scores are obtained to say if a file is correct or not
Seventh, it will try each Miller index combination for a set i value, apply a filter, and return filtered df + PrettyCombination. it will only add the filtered column if enough points & data changes significantly. If it doesn't change too much, the column PolarizationD3 will be duplicated with the new name
Eight, it repeats the process of obtaining the area in m*x+n-N < y < m*x+n+N where 75% of the points are inside the area. It also multiplies the value of N by a factor AcceptableMultiplier and erases all points outside this bigger area. The enlarged area will be plotted and saved in PlotResults only for the normal data (not the softened one as the files were already adequately saved and filtered without the Savitzky–Golay filter). As uncertainties are clearly underestimated I tried to make them reasonable (looking at the dispersion of the points it is clear there is systematic uncertainties. Under the hypothesis that the polarization curve should be a soft curve (at least C^1) we will try to use χ^2 to add a provisional uncertainty margin fitting to a linear expression. This is a very inaccurate uncertainty increase but it is an improvement of the underestimated uncertainties (and the lack of ways to quantify the systematic uncertainty sources)
    
4. CellID SAVING
It will safely store in a txt file all the polariser and analyser cell ids so that the code in ML can use them

5. DUPLICATION REMOVAL
It will check if the files created for the ML algorithm are duplicates and erases them in that case
