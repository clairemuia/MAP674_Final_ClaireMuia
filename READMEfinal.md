# Claire Muia
# MAP 674 Final Project
# Summary File

## Introduction

### Brief description of question of interest

For this project, I decided to explore questions related to white oak (Quercus alba) presence, abundance, and distribution in the US and then Kentucky. 
White oaks are a huge topic here in the FNR Department at UK, and I'm particularly interested in them (and overall forest biomass) as there is a very direct and influential connection between white oaks, forest habitats, egde habitats, and wildlife. Their populations aren't in horrendous shape, but there is an overall decline and also struggle to regenerate and recruit into the forest canopies where they occur.
White oaks are massively important to not only the timber and bourbon industries of Kentucky and other states, but also are a critical component of the diets and habitat structures of a number of animal species, from microscopic (forest invertebrates) to quite large (my thesis study animal, the elk) and numerous in between. 

Some wider questions I thought of when considering this topic included: where should white oak sustainability, preservation, or planting efforts be most prioritized? Where should funding, foresters, training, and public education on the species be most allocated for highest positive impact on communities, industries, and ecosystems?
Where should this be increased? Where is this being allocated currently, and where would it potentially have a larger impact elsewhere in the US and/or Kentucky to make the most use of limited conservation, sustainability, and education dollars?

### Spatial data analysis selected with reasoning for selection

To go about this spatial analysis/exploration, I chose to utilize both polygon and raster datasets, similarly to my Module 6 Data Combination. This spatial analysis involves rasterizing polygon data, joining it with the raster data, performing zonal statistics, and then mapping using those statistics as visual filler. 

I chose to utilize this method as I feel it is very effective informatively and visually. As I mentioned in my Module 6 assignment, it makes for an easily understandable visualization(s) when complete for any audience. The method is logical and robust for this type of data (continuous forest biomass data) in particular, and zonal statistics and then mapping can be a stepping stone to many further explorations or questions. The zonal summaries using mean (I use counties and states) are easy to interpret, and, as I mentioned before, doing this analysis by the specific "zones" of US counties and states provides a more governmental/legislative scale of where to focus efforts, since that's how laws/funds/efforts are allocated, foresters are placed, and sustainability outcomes are measured in our country. 

### Spatial question of interest

My more specific research questions to answer with this analysis included: what's the distribution of the important Quercus alba in the US, by states and counties? Where is this species limited to? Where is it's maximum presence, and it's locations of more high presence in the US? Where is the more minimum presence within its range?

I didn't go further into answering the wider questions of interest here, but I plan to and would use the outputs of my work below to hopefully answer some of those larger questions. 

## Dataset Information

### Dataset identified to help address the spatial question of interest

US Forest Service FIA White oak US biomass raster (tons/acre)

TIGER US Counties polygon dataset (tigris package)

TIGER US States polygon dataset (tigris package)

### Dataset name

Hosted_AGB_0802_2018_WHITE_OAK_06022023131242.tif

tigris R package: states() and counties()

### Link to the dataset

https://data.fs.usda.gov/geodata/rastergateway/bigmap/index.php

https://cran.r-project.org/web/packages/tigris/index.html

https://github.com/walkerke/tigris

### Geographic coordinate system used in the dataset

Raster: NAD 83/Albers Conic Equal Area

TIGER datasets: NAD 83/EPSG:4269  (reprojected these in the workflow to match WO raster)

### Short description of the dataset

The US Forest Service FIA (Forest Inventory Analysis) BIGMAP Tree Species Aboveground Biomass layers represent estimates in tons/acre for total aboveground biomass for hundreds of tree species at 30m pixel resolution (I altered this). FIA inventorying efforts combined with Landsat data collection and nearest neighbor modelling (also taking into account climatic and topographic data) create raster layers such as this one which show the distribution and biomass amounts of US tree species. 

The TIGER datasets include feature geometries on all US (and territories) counties and states from US Census data. It's in units of meters and is in polygon form with the legal government boundaries updated through 2024. 

## Spatial data analysis, with step-by-step description

Full R code steps for the below operations found in the accompanying final project text file. 

### Preparing R for workflow

These TIGER datasets do not require download, just simply installing the 'tigris' package from the "Install" function within the "Packages" tab of R, and then calling the package via library(tigris) when loading your packages before the workflow (as below). 

When on the US Forest Service FIA BIGMAP webpage (linked above), you can utilize ctrl+F to locate the "Quercus alba" species layer from the large list, and press download. Once this large zip file is downloaded, you can extract it to your working directory folder on your device, from which it will be pulled in the workflow below. At least that's how I bring my datasets in. 

I loaded all the packages I felt would be necessary for this project using library().

### Setting working directory

I always start by finding my current working directory with getwd() and then changing it with setwd() if its not the right one for my course/assignment. 

### Bringing in the TIGER polygon datasets, initial plotting

I utilized both states and counties included within the tigris package in R, from the TIGER US Census data. These were loaded using counties() and states(), viewed with glimpse(), the FIPS and ID codes changed to numeric variables with mutate(as.numeric()) to narrow to the contiguous US, narrowed down to the contiguous US for this assignment using filter() with those codes, the columns/variables renamed with rename() for clarification when plotted interactively later, and these datasets plotted initially with plot(). 

![counties_orig](counties_orig.png)

![states_orig](states_orig.png)

### Bringing in raster, initial plotting of original raster

I defined my object with the .tif file name exactly as it appears in my working directory and then used the rast() function on that object to ensure that it was in R as a raster dataset in the proper format for functions to be carried out on as I went. 

I also plotted this initial raster using plot(). 

![raster_orig](raster_orig.png)

This takes a long time to plot because of the size/fine resolution of the original raster. 

### Modifying original raster resolution for easier/quicker analysis

This raster is very large, even with much of it being No Data or 0 value cells. It was taking an extremely long time to load and do any kind of processing. 

I decided, for the purposes of this assignment, to make this raster "smaller," through first changing the resolution (# rows and columns) of it to more match the resolution of the imperiled species richness raster I used for Module 6, which was easier to handle in R. I did this by first defining new rows and new cols objects with the desired number, creating a template resolution using the new numbers of rows and columns, creating a template raster with that new resolution and the same extent with rast(), and resampling my original raster to that template with resample() using the bilinear method (which is appropriate for continuous data such as this). This took a long time to convert, and I then plotted the resampled raster with plot(). 

This was sort of an unnecessary step, as modifying resolution could have been combined with my other adjustments below that I did later. 

![resamp_raster](resamp_raster.png)

### Matching CRS of polygon datasets and this new resampled raster

Before proceeding any further, I wanted to ensure that the CRS of both datasets matched, so that I did not run into any projection issues as I moved on. 

I checked the CRS for all of the datasets using writeLines(st_crs()) and found that the USFS FIA raster data and US Census TIGER data have different CRS's (as above in the description). I then modified the CRS of the TIGER data so that it would match the raster by establishing new county and state datasets with st_transform() that had the white oak raster CRS. I used writeLines() again to confirm that the CRS's had changed. 

### Matching the extents of the rasters/extending the original raster using template, plotting

I then wanted to address the fact that the white oak raster was loading in to R with a different extent than the polygon layers (since white oaks do not occur on half the country, R was leaving out that half from the map). 

I created a template extent with ext() of the contiguous US using the extent of the narrowed down contiguous US counties polygon TIGER layer that I established before, created another template raster with that extent and the same resolution and CRS with rast(), extended the resampled white oak raster to that template with extend(), and then plotted this with plot() to confirm that the raster had been extended properly. 

![resampext_raster](resampext_raster.png)

### Changing to square cell shape/resolution

In yet another separate step that probably could have been carried out sooner, I was not happy with the new non-square cell shape that the prior change in resolution had created in this raster. I wanted to then alter the cell shape of this new extended raster so that the cells are squares. 

I researched what square resolution (width in meters x height in meters) was appropriate for continuous biomass data like this, and around 300 was an appropriate number. I decided to make this a bit finer (200m x 200m), without going too small that the raster became "large" and hard to handle again. 

I created a template extent using ext(), defined a new cell size, calculated the rows and columns for that cell size using ceiling() for another template raster to be made, created the templete with rast(), set the resolution to that of the defined size using res(), and resampled my extended raster from before to this new template using resample() and the bilinear method again. 

This created my final raster for use in the rest of the assignment, which I then renamed to an easier name and plotted with plot(). 

![resampextsq_raster](resampextsq_raster.png)

### Computing dataset summary statistics

I then computed summary statistics on my new white oak raster by first obtaining the dimensions of it with dim(), getting sample statistics using summary(), and then getting global stats on the whole raster with minmax() for minimum and maximum values, global(raster, mean) for mean, global(raster, median) for median, global(raster, sd) for standard deviation, global(raster, sum) for the sum of all of the raster values, and freq() to see the frequency of each cell value in the raster. 

![dim](dim.png)

![summary](summary.png)

![global](global.png)

![freq](freq.png)

For the contiguous US states polygon, I chose to do summary stats on the ALAND -> LandArea (total land area in square meters) variable that I renamed earlier. I made a summary object and used sumamrize() to obtain mean land area, median, total number of states, minimum land area, maximum land area, and standard deviation of land area for the contiguous US, and printed that using print(). 

![states_summ](states_summ.png)

I did this same thing for contiguous US counties. 

![counties_summ](counties_summ.png)

To obtain summary stats on just Kentucky counties, I used the counties() function from tigris again and specified state to KY. I could have done this differently, as discussed below. I changed the CRS of this subset of counties to match the narrowed down county_new object I had made prior, again using writeLines(st_crs()) and st_transform(). I then did the same thing as for US counties using summarize() and print() to obtain the stats. 

![ky_summ](ky_summ.png)

### Creating and evaluating histogram and boxplot

I attempted to create a raster histogram of the white oak raster using terra's hist(), which resulted in the below error and histogram. 

![terra_error](terra_error.png)

![raster_terrahist](raster_terrahist.png)

A way around this issue (which I assume just comes from the fact that most of this extended raster is NA values, which would then make up for the majority of the raster values) is to designate only non-NA values for creating the histogram. This was done using values() and !is.na(values()), and then hist(). 

![raster_nonnahist](raster_nonnahist.png)

The resulting histogram here is similar to the one created before with the error, but just without that error. I'm not entirely sure what the deal is with this, but I figured the histogram output itself seems to make sense. 

There are a number of zero values here in this countrywide raster, and then there seems to be a peak around maybe 1 ton of white oak biomass/acre, with a relatively steady decrease from there to a maximum of low 20's tons of biomass/acre. There's a very slight plateau right before 5 tons/acre. 

This all makes sense to me, as obviously there are not many areas left in the US where the biomass of any certain tree would be hugely significant, and the likelihood of higher biomass amounts would decrease as the amount increases for any tree. The peak around 1 ton per acre was interesting, perhaps this is the more common healthy distribution of adult white oaks (more dense white oak stands become less and less frequent due to outcompetition with each other). 

I also attempted a boxplot of the raster using boxplot(). This also came up with some similar sampling errors and the following boxplot.

![raster_box](raster_box.png)

I then made a boxplot instead with those same non-NA values as established for the histogram, which made a boxplot without errors that looked similar. 

![raster_nonnabox](raster_nonnabox.png)

The many outliers are visible, and the mean around 4 tons/acre is visible, with most data right around that value (creating the many nearby outliers). 

I also created a histogram and boxplot of contiguous states land area, contiguous US counties land area, and Kentucky counties land area (respectively, below) using ggplot(), geom_histogram(), and geom_boxplot(). 

Contiguous US states:

![states_hist](states_hist.png)

![states_box](states_box.png)

The weird distribution of contiguous US states sizes/areas can be seen in the histogram, where there is one major outlier in size and a decent grouping around the mean of 156223616735 square meters. The three actual higher outliers are seen in the boxplot. 

Contiguous US counties: 

![counties_hist](counties_hist.png)

![counties_box](counties_box.png)

The contiguous US counties are also weird, with abundant outliers above the mean of 2462382465 square meters. That's also seen in the extremely high peak that tapers off very quickly into the outliers with the higher values. 

Kentucky counties:

![ky_hist](ky_hist.png)

![ky_box](ky_box.png)

Kentucky counties, with a much smaller number of counties, has a distribution much more easily discerned from ist histogram and boxplot. The mean is visible with the IQR encompassing the large portion of the data (many of the counties are similar in size), and two outliers are visible. The distibution in the histogram does not have a taper to extreme outliers. 

### Mapping of polygon layers (for fun)

At this point I also wanted to go ahead and actually map out these state and county polygon layers, to compare the land area summary findings with a visual (unnecessary step). 

I did this by using ggplot() + geom_sf() and specifications. 

![states_map](states_map.png)

![counties_map](counties_map.png)

![ky_map](ky_map.png)

### Rasterizing county vector datasets to carry out selected spatial analysis

In order to carry out the selected spatial analysis from here (zonal stats and mapping), rasterizing the vector polygon states and counties datasets had to be done. 

I used rasterize(vect()) to do this on both the contiguous states and counties datasets I had created and used before, which resulted in these transformed into rasters. 

I looked at the dimensions and summaries of these too out of curiosity, but this was not necessary at all. 

### Zonal statistics (mean) using states and counties as zones 

To carry out the zonal statistics, I assigned a new object as the "zonals" for the zonal(white oak raster, new state raster) output. 

I looked at a summary for this, and renamed the second "variable" here from the initial raster title to a more meaningful name (State_MeanWOBiomass).

I did the same for counties, but using the new counties raster and renaming the variable to County_MeanWOBiomass. 

### Joining the rasters for mapping

I utilized left_join() to join first the contiguous states dataset with the new states as zones zonal() output, joining by the "fips" column which exists in both (state FIPS code). 

I did the same but with the counties dataset and the new counties as zones zonal() output. 

For just Kentucky counties, I did another left_join() of the county dataset and the counties zonal() output, then filtering to just StateFIPS = 21 for KY. I did a summary of this out of curiosity too, but again this was an unnecessary step for the assignment. 

These all create datasets which can now be plotted and visually show the zonal statistics by zone with fill color. 

### Ggplotting

I then used ggplot() with geom_sf(), scale_fill_distiller(), labs(), theme(), and annotation_scale(), along with other specifications to map the mean white oak biomass per contiguous US state, contiguous US county, and Kentucky county. 

![states_ggplot](states_ggplot.png)

![counties_ggplot](counties_ggplot.png)

![ky_ggplot](ky_ggplot.png)

### Tmapping

I also then mapped the states and Kentucky counties using tmap() as well, so that they could be turned into interactive maps using tmap_mode("view") and saved as html's. 

With tmap() I used tm_shape(), tm_fill(), tm_borders(), tm_title(), and tm_scalebar() to create the below static outputs.

![states_tmap](states_tmap-1.png)

![ky_tmap](ky_tmap.png)

### Interactive tmapping

To turn these tmaps into their interactive version, tmap_mode("View") was used, the maps called (which then displayed as interactive), and then tmap_save() with a .html filename was used so these would become accessible, interactive HTML's where states or Kentucky counties 

[state_wo_tmap](../../htmls/state_wo_tmap.html)

[ky_wo_tmap](../../htmls/ky_wo_tmap.html)

## Discussion

### Interpreting spatial analysis results

The areas of the country with the most (highest biomass) and most dense white oak (Quercus alba) are pretty clear visually from the US counties map, along with the areas with less or no white oak stands. 

WHite oaks need specific climatic conditions and elevations to occur/thrive. For this reason, the clear limitation on the species range is visible in the wide contiguous US maps I made. It is mainly limited to the eastern US, but absent in the highest elevations of the Appalachians and very notably absent in the Mississippi delta regions and Florida. 

From some research, the most optimum average annual conditions for the species are 55 degrees F, 40 in of precipitation, and snowfall from 15-20 in. It grows well in most soil types besides those that are shallow, dry, sandy, poorly-draining, or wet bottom land soil. It's normally found below 500 ft in elevation, but can exist as a smaller scrub tree at elevations up to 4,000 ft. 

Source: (https://www.srs.fs.usda.gov/pubs/misc/ag_654/volume_2/quercus/alba.htm)

These factors acount for the distribution of the species as we see in the map. 

In the US, it's clear that the highest occurrences of Quercus alba are to either side of the maxmimum elevation of the Appalachians (the foothills and such) and toward the central east coast, along with the clear highest occurrence in the lower Missouri and upper Arkansas areas. This makes sense given their optimal conditions, the availability of standing timber habitats in these areas, and a lack of human development and encroachment in most of those areas. 

As for Kentucky, despite white oak being a major part of the state's identity, it falls to third in mean biomass when it comes to biomass by state. It's mean biomass is still relatively high statewide for the areas in which it occurs, but is not the state with the highest mean at all. 

When zooming in then to just the Kentucky counties, more fine trends in the species' occurrence are seen regionally in the maps I made. Areas within KY with high biomass of white oak are clearly seen, notablly in Trigg and Lyon counties toward the western end of the state and also that same band diagonally across eastern Kentucky that I observed with my imperiled species mapping. These patterns relate to overall biodiversity and intact natural habitat patterns in Kentucky. As for Trigg and Lyon counties, I've learned that the Pennyrile region/plateau is very fertile and has optimal climatic, soil, geology, elevation, and other conditions for white oak growth along with lots of land preserves and intact natural areas. Similarly to the imperiled species assessment I did before, the band of higher biomass from southern KY to upper eastern KY coincides with areas lacking significant human presence and development along with lots of intact natural habitat and habitat recovering from prior surface mining and large logging projects (there still is some of this occurring, but this is much smaller now and mainly in the bottom right corner of KY). The hotspots for higher white oak (and probably all tree species biomass) in the state coincide with those bands of higher biomass that go across the Appalachian region in the foothills areas. 

Even though this mapping of tree biomass was done by legistalive/governmental boundaries (for reasons discussed) instead of scientific/random/research-oriented boundaries or grids, it's still visually informative and also indicates states/regions within states where foresters, silviculturalists, conservation/sustainability efforts, white oak research, etc. should be allocated and should focus. I personally had not heard of much white oak scientific or conservation activity happening in the Missouri and Arkansas areas, but this is clearly where lots of research into their regeneration, recruitment, and sustainability should be focusing largely. Obviously, research across a species whole range is ideal, but dollars and people for this are limited, so the most impact from any efforts would be desired. This mapping could help reframe some of those efforts potentially and create more beneficial and robust outcomes that can then be applied throughout the rest of the species range for it's benefit. 

### Description of how the map includes the spatial analysis results

These maps exhibit the resulting zones (states or counties) with the fill color of each zone representing the value of the mean white oak biomass found within each zone (tons/acre), which was calculated using the zonal() function, as given by the legend of each map. 

### Discussion of mappping decisions made that highlight the spatial results

I discuss these above and below. 

### Consider the questions: What worked well? What didn't? What could be tried in the future?

Plotting, data handling, and zonal statistics worked well once I reduced the size of the initial large WO raster. I'm glad I did this, even if it does alter the fineness and precision of the biomass data (helpful for this assignment/workflow/learning though). 

Mapping with the zonal statistics was cool and the visualizations turned out well I think. 

Small analytical fixes:

I would've added the ALAND and AWATER variables initially using mutate() to create a true total area variable for each county and state - I think the ALAND variable does not give total area, just land and excludes the water area (since it is a separate variable - I wasn't able to find confirmation of this online or not). 

I would've combined and narrowed down my workflow when going through the process of creating a new square resolution, resmapling, and extending my raster to make it easier to use. I mentioned this before in this file - the way I did it was not very logical and sort of jumbled and overly complicated. I also discuss this more below. 

I probably should've matched the CRS's of these datasets earlier in the workflow than I did. 

I would have changed the units for biomass (tons/acre) using formulas and mutate() so that they match the units used for the area of the states and counties (square meters, so biomass like weight/square meter or km). 

I should have subsetted my county_new object for just KY counties and made a KY counties object from that instead of making a whole new ky_counties the way I did, as the variable names were not updated like I had done prior (had to use ALAND) and I also had to rematch the CRS myself (dumb mistake here). 

Larger decisions:

I considered one aspect before choosing a raster dataset and starting this workflow: using zonal stats on a raster such as this (continuous data on a single species) is more logical/informative than using it on a raster data type as in my Module 6 with richness of multiple distinct species as the variable being plotted - in that case (which I didn't think about at the time), there would be overlap between species counted in neighboring zones, making the mapping less logical/informative when you aren't certain if the 5 imperiled species in one zone are all distinct, different imperiled species from those in a neighboring zone or those in a zone across the country, or if 4 of those 5 species are the same as another zone (if that makes sense). So I'm glad I thought of that, and used zonal statistics on data that made more logical sense to use it on for this project. 

I could have used minimum for the zonal statistics instead of mean perhaps, to maybe better account a little bit for overestimation by the FIA modelling techniques used by the USFS when making the biomass maps, but I think mean was generally fine and maybe best to use here. 

I think zonal statistics was generally appropriate to use for this data/these objectives, and that it resulted in informative visualizations for most audiences (but probably wouldn't be used much in a very scientific sense since I decided to perform the stats with governmental boundaries as zones and means within those). A different type of spatial analysis and subsequent mapping may be more informative for actual foresters/scientists, but I think zonal stats in this manner were appropriate for the project and the broader objectives and audiences I mentioned, along with conservation dollar/effort allocation to specific legislative areas in the US.  

I would have thought/planned my R workflow out more ahead of time, especially concerning modifying the original WO raster, in order to cut down on unnecessary steps and be more concise coding-wise. A lot of those steps can be combined if the process is thought out ahead of time (simultaneously making the resolution more coarse, changing the cell size to squares, and extending the new raster). I thought ahead and planned better for the mapping, so that part of the process was much more consice and less sort of "stream of consciousness" coding as I went along. I could have gone back and slimmed those resampling parts down, but I didn't want to push too close to the deadline of the assignment. 

### You don't have to work through lots of iterations of the analysis when you identify something that could be tried in the future. The purpose of the map discussion is to think critically about the analysis you worked through, the map decisions you made, and how those could be adapted in the future.