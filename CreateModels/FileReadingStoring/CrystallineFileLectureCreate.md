CrystalineFileLectureCreate.ipynb

Reads from D3Files all the files it is going to process

Outputs the following folders and files:

1. CrystallineLog_Reading_Creation.txt
Logs all the prints and every step the code does. If you trust the code, it is irrelevant. If you don't trust it or want to change it then this txt file will tell you how each experiment file has been processed and where there might have been issues.


2. Crystalline_CellID
Contains the Cell IDs found on all the files. Needed for the ML code.


3. AmorphousPlotResults
This folder will store the graphs of all the experiments that were accepted. Not needed for anything but it is nice to see the files that will be fed to the model. For each experiment you can find the following files:

3.1 Crystalline_Reading_Summary.txt Shows the Score of each experiment. If you use the default criteria for deciding if a file is adequate (a.k.a method FilteringMethodInt = 12) then all files under 1.4 are rejected and the files that have been forcefully accepted or rejected appear as well indicated here. IF YOU RECKON AN EXPERIMENT SHOULD BE ACCEPTED DESPITE BEING SHOWN HERE AS REJECTED PLEASE FIND THE LIST force_reject_files AND ADD YOUR ECPERIMENT USING THE SAME STRUCTURE. IDEM FOR ACCEPTED FILES THAT SHOULD NOT BE ACCEPTED. As a side note, in order to know visullay if a file is good or not, open the associated graph and just check that it doesn't do any funky business (see the examples already present to get a feel on what the word "funky" means)

3.2 {base_name}_N_{N}.png 
It shows the values of the experiment in black, a linear fit in blue and the area needed for 75% of the points to be inside the rectangular region (the area inside y_min=mx+n-N<y<mx+n+N<y_max). Here you can find a good estimate on how good the overall decreasing tendencies are.

3.3 {base_name}_ExtendedArea.png 
It shows the same pot and also the range y_min=mx+n-1.3*N<y<mx+n+1.3*N<y_max. The points outside the green area will be discarded as they are considered to be too off to be considered correct.

3.4 {base_name}_Derivatives.png
It shows the number of negative slopes between points and how steep they are


4. CrystallineMLDataBase
Contains the .txt files NECESSARY for the ML algorithm. There are two per experiment

4.1 {base_name}.txt 
Contains DeltaTime (the time of the measurement measured from the first VALID polarization measurement), PolarizationD3, SoftPolarizationD3 (the polarization after using a Savitzky-Golay filter) and ErrPolarizationD3 (the uncertainty)

4.2 {base_name}_Parameters.txt
Contains the CellID, Pressure, LabPolarization (the polarization measured at the lab) and LabTimeCellID (the time when it was measured)


5. CrystallineFailuresTest
Contains the plots 3.2 and 3.4 but for the experiments that failed the overall decreasing test. Check them if you can to see if any of your experiments has been placed there by mistake



6. CrystallineDataBase
Contains all the .fli files that were attempted to be read


7. CrystallineBadFiles
Contains all the .fli separated in experiment sets folders that were rejected (not enough points, negative polarizations, etc.)


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


