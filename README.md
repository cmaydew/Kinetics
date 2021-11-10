# Kinetics
Fit double exponential model to kinetic data in pseudo first order conditions.
def Kinetics(Folder, individual = False, comparison = "Auto",key = False,Units = "Auto",S0 = "Auto", Normalize = True, Errorbar = True, overlay = False):

Inputs:
Folder: Parent folder containing all kinetic data. Should be grouped by replicates for an ion, which are grouped by pH. End of each replicate needs to have biological replicate and technical replicate listed last with either a period or hyphen between them, e.g. 1.2,1-2,Rep1.2, Rep1-2. Replicate folders should contain csv files named with concentration and a space immediately after. Concentrations need to be in the same unit. Files need to come from the stopped flow (ProData SX) and any csv file that starts with "Run" or "0" will be skipped. All folders called "Gluconate" or "Plots" are skipped.

comparison = "Auto", "pH", "Ion": automatically determines whether to compare between pH values for a single ion or between ions for a single pH. If a folder for a single pH is given as Folder, then sets comparison to "Ion" if folder containing multiple pH values are given then sets comparison to "pH"
key = False, dictionary: uses default key for colors of ions or colors of pHs, key value pair of ion(str), or pH(float) with color
Units = "Auto", str: Finds units based on the unit in the name of the csv file, unit must be separated by spaces and consistent between all files.
S0 = "Auto", int: Automatically determines the starting fluorescence by varying it for the best fit for each injection
Errorbar = True, plots include errorbars
Normalize = True, normalize every injection from 1 to 0 before averaging between injections
overlay = False, can be set to "All" to overlay all concentrations, separating by ion, pH and biological replicate, passes data to PlotOverlayAll() for plotting

This function relies on several functions, listed below. Overall result is the overlaid plot of pseudo first order kinetic constants either between ions for a single pH or between pH values for all ions (plotted one ion at a time, overlaying each pH for that ion).

Needs specific organization of files, example of path to one kinetic trace is "/For Publication/pH 6/Bromide/210803 GFPxm163 Br pH 6 Rep1.1/8 mM Br.csv". All kinetic traces for each replicate need to be grouped according to replicate. Kinetic traces need to be named with the numerical concentration in front, followed by a space, units need to be consistent between as well. "8 Br mM.csv" would work but "8mM Br" would not. Last term in folder needs to be biological replicate followed by technical replicate, with either a period or hyphen between. Value of technical replicate doesn't matter, but must be after biological replicate, which needs to be a number, 1 or 2.
Folder containing replicates needs to be name of ion, currently can use "Fluoride" "Chloride" "Bromide" "Iodide" "Nitrate" or "Nitrite"
Folder containing ions needs to contain "pH" and the pH value, both separated from anything else by spaces, and the pH value needs to be last.

As an example "/For Publication/" is a folder containing the folders "pH 6" "pH 7" "pH 6.5" So if "/For Publication" is specified, then it looks within each subdirectory (pH) for each ion and groups each ion for overlaying. If "/For Publication/pH 6" is specified then each ion of pH 6 is grouped for plotting.

Output: plots files in folder called "Plots" which should be located in the parent folder when comparing between ions for a pH or in the folder specified for comparing between pHs for ions, e.g. "/For Publication/Plots/"

Kinetic function (with comparison = "Auto") first uses the basename of the folder to determine whether to compare between ions or between pH for each ion. It splits the basename of the folder and if it finds "pH" then it compares between ions for that pH and if that is not found then it compares between pHs for each ion. Folders must be organized as specified above.
Plotting colors/markers/lines are then specified based on available ions names or pH values.




FolderFinder(Folder, comparison)

Uses Folder specified above with comparison (determined or specified by Kinetics()) to find and organize all replicate folders for each ion.

Output: with comparison = "Ion" a dictionary with key:value of ion:[Path to replicates for ion] orders based on keyorder, which is currently "Chloride, Bromide, Iodide, Nitrate"
with comparison = "pH" a dictionary with key:value of ion:d where d is a dictionary with key:value of pH: [path to replicates for pH]

Fitting(Data):
Integral transformation of raw data to fit linearly. Because this is unweighted and occasionally produces outliers, it is used for accurate initial parameters for scipy.curve_fit(), outputs the parameters calculated


DataProcessing(IonReplicates,S0 = "Auto",Units = "Auto", Normalize = True)
Function used within Kinetics() to organize and fit all data, relies on Fitting(Data) as initial estimate of parameters for weighted regression (scipy.curve_fit). Bounds are specified within as 15% on kobs1, 40% on kobs2 and physical bounds on other parameters. Also checks for outliers based on linear correlation, and replaces outliers with correlating parameters. Will also check if the two biological replicates correlate with each other.


IonReplicates: output of FolderFinder()
S0, Units, Normalize from Kinetics()

Output: IonsData: {final raw data after truncating dead time, normalizing, and averaging},
	     ddfavg: {kinetic parameters from exponential fit of IonsData}
	     Units: determined within function by ReadInCSV()

Feeds IonReplicates to ReadInCSV() to bring in averaged raw data for each csv file. Each concentration is averaged between technical replicates, propagating error based on sum of variances and variance between means at each time point. This is then outputted along with the exponential fits of these averaged traces. The biological replicates are kept separately.


ReadInCSV(directory, S0 = "Auto",Units = "Auto", Normalize = True)
Function used to read the raw csv files from the stopped-flow instrument

directory: string, folder containing csv files (e.g. "/Users/cadenmaydew/Research/GFPxm163/For Publication/pH 6/Bromide/210803 GFPxm163 Br pH 6 Rep1.1"

Searches for csv files within directory and ignores any files with "Run0000" (the default filename) or "0" (the control)

With Units = "Auto", uses the second term to set Units value to "mM" or "µM". CSV filename needs to be numerical concentration followed by space, with either "mM" "uM" or "µM" isolated by spaces "8 mM Br.csv" or "8 mM.csv" "8 Br mM.csv" would work, while "8mM Br.csv" or "8 mm Br.csv" would not work.

Files are read in, and must have multiple injections for reading in as it changes how the csv file is written. Finds table of data, truncates first 10 ms, normalizes each injection (if Normalize = True) based on end points (calculated based on fitting each injection), then averages and takes the standard deviation. This is then outputted along with the units. 



PlotOverlayAll(Data,Data2,output, units, size = (3.4,2.5), DPI = 600)

All variables are passed from Kinetics:
Data is the raw fluorescence data
Data2 is the fit parameters for the double exponential model
output is the general output directory
units are the units for concentrations
size is the figure size
DPI is the output dots per inch to save the files.

For each ion, pH, and biological replicate, all concentrations are overlaid with the highest concentration plotted in red. A subplot of the residuals from the fit is calculated and plotted, with the highest concentration also in red.







