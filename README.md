# fish-summarizer
README File

File name: Fish_summarizer_function.Rmd

Created by Anela K. Akiona, aakiona@ucsd.edu


## Description
Fish_summarizer_function.Rmd is an R markdown function that calculates density and biomass of fish transect count data, using user-provided length-weight parameters, and produces summary files and plots. The motivation for developing this function was to provide a one-stop-shop for colleagues and collaborators to utilize the length-weight parameters published in Akiona et al. *(In prep)*. This function was conceived and created by members of the Sandin Lab at Scripps Institution of Oceanography at UC San Diego, and as such the function is tailored for their belt transect methods, though it is meant to be customizable to other styles of belt transects.

## Arguments

- fishdata: The CSV file containing the field observation data of fish counts and lengths.

- LWfile: The CSV file containing the species-specific length-weight parameters.

- filename: Unique identifier to be included in output file names.
  
- sp_data: Name of the column in fishdata which contains the species names. The values in this column must match the values in sp_lw.
  
- sp_lw: Name of the column in LWfile which contains the species names. The values in this column must match the values in sp_data.
- integrate: If TRUE, uses the integral method to calculate biomass. If FALSE, biomass is calculated using the given length.
- size_correct: Number to be subtracted from the total length of each fish prior to biomass calculation. Cannot be used if integrate = TRUE.
- bin: Bin size used for length measurements. This can either be a single number or vector of numbers.
- max_size: If bin is a single number, this is the maximum size to which biomass should be calculated.
- size_threshold: Fish length at which belt transect width changes (cm). If belt width is the same for the entire transect, enter 0.
- width_below: Width of transect for fish smaller than size_threshold (m).
- width_above: Width of transect for fish larger than size_threshold (m). If belt width is the same for the entire transect, enter it here.
- transect_length: Length of transect (m).

## Outputs
Data files:
- fish_summary_station_'filename'.csv
	- Biomass and density for each station.
  
- fish_summary_group_'filename'.csv
	- Biomass and density by trophic group at each station.
 
- fish_summary_species_'filename'.csv
	- Biomass and density by species at each station.
 
Plots:
- fish_summary_density_'filename'.png
	- Stacked bar chart of density (#/m2) by trophic group at each station.

- fish_summary_biomass_'filename'.png
	- Stacked bar chart of biomass (g/m2) by trophic group at each station.
 

## Details
_Data file requirements_

This code calls specific column names to conduct the calculations, and as such the data files passed to the function must contain those column names. Below are the required column names for the respective data files. Please note that unless indicated otherwise, the names must be identical, including capitalization.

_Required columns in the fish survey data file (with order of columns not constrained):_

1) Data_Type -- For all count data, record as "QUAN" (differentiates from "PRES" used to document presence in larger swath for remote island surveys). Only data with Data_Type = QUAN will be included in the calculations,
	
2) Date -- Part of metadata for record,
   
4) Transect -- Metadata (continued), noting from which replicate transect the data were collected,
   
6) Diver -- Metadata (continued), noting unique identifier of diver collecting data,
   
8) Station -- Name of the station (needs to be unique within the data file, but can be repeated across dates, though the data will be lumped by station),
   
10) sp_data -- Unique species codes or taxon names which must match sp_lw (see below). This column can have any name, but it must be passed to the function as an argument,
    
12) Number -- Records the number of fish of the particular size observed in the particular transect,
    
14) Size -- Records the size of fish in cm.

_Required columns in length-weight file (with order of columns not constrained):_

1) sp_lw -- Unique species codes or taxon names. The list in this column must include all species identifiers used in sp_data. This column can have any name, but it must be passed to the function as an argument,
   
3) a_cm -- Best estimate of alpha from length-weight relationship,
   
5) b_cm -- Best estimate of beta from length-weight relationship,
   
7) LTLRat -- Length conversion factor linking field-estimated length (in Total Length) to length type for length-weight relationship (can be Standard, Fork, or Total Length),
Weight (g)=a_cm*(LTLRat*fish$Size[field estimate of TL in cm])^(b_cm)

9) Trophic -- Trophic level classification for each species. Any types of unique categorizations will work.

_Examples_
```
## Using the integral method
fish_summary(fishdata = "Pohnpei_2018-08_Fish_Data.csv", 
            LWfile = "Pacific_LW_params.csv",
            filename = "Pohnpei 2018",
            sp_data = "Species", 
            sp_lw = "NewName",
            integrate = TRUE, 
            bin = c(2, 5, 10, 15, 25, 50, 75, 100, 150, 200), 
            size_threshold = 20,
            width_below = 2,
            width_above = 4,
            transect_length = 25)

## Using size bins with size correct
fish_summary(fishdata = "Pohnpei_2016-08_Fish_Data.csv", 
            LWfile = "Pacific_LW_params.csv",
            filename = "Pohnpei 2016",
            sp_data = "Species", 
            sp_lw = "Taxon",
            integrate = FALSE,
            size_correct = 2,
            bin = 5,
            max_size = 300,
            size_threshold = 0, 
            width_above = 2,
            transect_length = 30)
```

_Notes on size correcting and the integral method of biomass calculation_

When estimating fish lengths in the field, it is often prudent to classify individual fish into size bins, rather than attempting to estimate length to the nearest 1 or 0.1 cm. While this is a commonplace and practical solution to the challenge of estimating the length of often rapidly moving organisms, it can introduce bias when calculating biomass – using the length estimated in the field assumes that every individual is at the maximum of the size bin, and thus biomass calculations can become artificially inflated. One such way to correct this bias is the use of a ‘size correct,’ which subtracts a user-defined amount from each fish size (e.g. 2 cm off every length for fish in 5 cm bins). This, however, can be quite arbitrary and may not be applicable when bin size changes with fish length.

Here, we standardize the correction using the mean value theorem for integrals, which states that if $f(x)$ is a continuous function on the closed interval [a, b], then there exists a number $c$ in the closed interval such that

$$\int_{a}^{b} f(x) \mathrm{d}x = f(c) * (b - a)$$

Where f(c) is the average value of the function from a to b. For the length-weight relationship, this can be written as,

$$\int_{x_1}^{x_2} a_i L^{b_i} \mathrm{d}L = \bar{W_i} * (x_2 - x_1)$$

Where $x_1$ and $x_2$ are the lower and upper limits of the size bin, respectively, and $\bar{W_i}$ is the mean weight of the size bin. This can be rearranged to give,

$$\bar{W_i} = \frac{\int_{x_1}^{x_2} a_i L^{b_i} \mathrm{d}L}{x_2 - x_1}$$

This method corrects for potential bias introduced in the belt survey methods and will work regardless of the width of the size bin. When integrate=TRUE, the function will calculate biomass using the mean value theorem for integrals. When integrate=FALSE, the function will calculate biomass using the given length, i.e. the maximum value for that size bin.

## Reference
Akiona, A.K., B.J. Zgliczynski, M.M. Agarwal, B.J. French, N. Hanna Holloway, K.A. Lubarsky, M.E. Shirley, C.J. Sullivan, S.A. Sandin. 2025. A database of life history parameters for Pacific coral reef fish. Scientific Data, 12(1425). https://doi.org/10.1038/s41597-025-05788-x 

