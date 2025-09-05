<h1>AmorphousFileLectureCreate.ipynb</h1>

Reads from D3Files all the files it is going to process

Outputs the following folders and files:

1. AmorphousLog_Predicting_Creation.txt
Logs all the prints and every step the code does. If you trust the code, it is irrelevant. If you don't trust it or want to change it then this txt file will tell you how each experiment file has been processed and where there might have been issues.

2. ML/AmorphousPredictFiles
This folder is not inside the folder you are currently in (FileReadingStoring) but on another one at the same level called ML. It contains the files needed for the ML 

2.1 {base_name}.txt 
Contains DeltaTime (the time of the measurement measured from the first VALID polarization measurement), PolarizationD3, SoftPolarizationD3 (the polarization after using a Savitzky-Golay filter) and ErrPolarizationD3 (the uncertainty)

2.2 {base_name}_Parameters.txt
Contains the CellID, Pressure, LabPolarization (the polarization measured at the lab) and LabTimeCellID (the time when it was measured)


6. AmorphousDataBase
Contains all the .fli files that were attempted to be read


7. AmorphousBadFiles
Contains all the .fli separated in experiment sets folders that were rejected (not enough points, negative polarizations, etc.)

8. PolarizationTImeReference
The models where trained using relative time only. This means that the first valid polarization measurement has been considered as time zero for each experiment and the rest of measurements have their associated time values as a variation of time since that reference. This way all experiments have the same structure. However this reference time is not absolute and the measurements of the diffractograms may have an absolute time reference different from the ones used in the models. By saving the string "Year-Month-Day Hour:Minute:Second" we can safely change the time reference
_________________________________________________________________________________________

Process it follows:

1. REMOVAL OF PREVIOUS ITERATIONS
To avoid leaks and duplications, all files are erased before running the code file

2. ZIP FOLDER TREATMENT
The code will take all the zip files, extract them, remove duplicates using the name AND hash sha256.

3. It finds all the chunks with the correct information
   
    
4. CellID SAVING
It will safely store in a txt file all the polariser and analyser cell ids so that the code in ML can use them

5. DUPLICATION REMOVAL
It will check if the files created for the ML algorithm are duplicates and erases them in that case

________________________________________________________________________________________

VERY VERY IMPORTANT

The other code files should not have an issue with the structure inside the fli files as long as you download them from the online data base and keep them zipped. However here we expect a very specific structure. PLEASE HAVE THE FILES INSIDE THE ZIP FILES HAVE THE FOLLOWING STRUCTURE (perhaps in the future we can change it so it is also automatic).
1. Only two rows of numerical values per 'polariser cell info' and 'analyser cell info' row (not even the order or number of them matters, the code is bulletproof in this aspect)
2. The same miller indices for those two rows
Here you have a (real) example:

analyser cell info sic1403 pressure/init. polar 2.00 0.77 initial date/time 08 12 23 @ 13:00
polariser cell info ge18012 pressure/init. polar 3.10 0.77 initial date/time 08 12 23 @ 12:00
   41433   3.000   3.000   3.000 08/12/23 13:05:39       0.00        0.5573    0.0020    3.5181    0.0202     120.00
   41468   3.000   3.000   3.000 09/12/23 10:48:40       0.00        0.4575    0.0656    2.6866    0.4456     120.00
analyser cell info sic1402 pressure/init. polar 2.00 0.79 initial date/time 09 12 23 @ 10:55
polariser cell info ge18008 pressure/init. polar 3.10 0.77 initial date/time 09 12 23 @ 10:55
analyser cell info sic1402 pressure/init. polar 2.00 0.77 initial date/time 09 12 23 @ 10:55
   41488   3.000   3.000   3.000 09/12/23 15:36:16       0.00        0.5222    0.0022    3.1860    0.0191     120.00
   41526   3.000   3.000   3.000 10/12/23 10:53:09       0.00        0.3935    0.0026    2.2976    0.0142     120.00

