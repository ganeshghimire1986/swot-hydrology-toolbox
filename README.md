# SWOT Hydrology Toolbox.

Copyright (C) 2018-2020 Centre National d'Etudes Spatiales

This software is released under open source license LGPL v.3 and is distributed WITHOUT ANY WARRANTY, read LICENSE.txt for further details.

## Background 

SWOT (Surface Water and Ocean Topography) is an innovative radar altimetry satellite mission projected for launch in 2021, prepared jointly by NASA’s Jet Propulsion Laboratory (JPL) and Centre National d’Etudes Spatiales (CNES), with contributions from the UK Space Agency (UKSA) and the Canadian Space Agency (CSA). SWOT features a high-rate (HR) data mode for continental hydrology, and a low-rate (LR) data mode dedicated mainly to oceanography. For more information, refer to https://swot.cnes.fr/en/ or https://swot.jpl.nasa.gov/. 

## Objectives 
* Provide open-source tools that, together with JPL’s RiverObs tool (https://github.com/SWOTAlgorithms/RiverObs.git), enable end-users to generate virtually all SWOT HR level-2 (L2) products with fairly (but not fully) representative characteristics (see section on caveats below)
  * Get familiar with product content and formats, use the data to conduct studies...
* Give end-users access to the L2+ HR processing prototypes 
  * Validate methodology, propose improvements...
* As far as possible let the toolbox correspond directly to the processing prototypes that are evolving towards operational processing chains 
  * Coded in high-level programming language (Python 3), for simple distribution, installation and use

```
Note that both algorithms and products are still under development and will be updated regularly.
```

## Content 
* SISIMP: Large scale simulator of L2_HR_PIXC products (with orbit selection tool)
* LOCNES: Generation of L2_HR_LakeTile, L2_HR_LakeSP and L2_HR_PIXCVec products 
* Improved geolocation library (used by RiverObs and LOCNES)
* Module to generate L2_HR_Raster products (under development, not yet included)
* Overall script permitting to run all steps consecutively (with example dataset)
* Tools for file format conversion etc.
* Potentially other modules in the future (for ex. floodplain DEM generation)

## Associated RiverObs version
develop branch

commit 1c13a9fc2f2bdc257db84a3ee51194f6bf6dec46 Author: Cassie Stuurman cassie.stuurman@jpl.nasa.gov Date: Mon Nov 23 19:19:21 2020 -0800

updated batch validation tool for compatibility with plot reach

River database SWORD v6 available here:
http://gaia.geosci.unc.edu/SWORD/SWORD_v06.zip

Don't forget to modify parameter_river.rdf
reach_db_path (-) = /work/ALT/swot/swotpub/BD/BD_rivers/SWORD_v06/Reaches_Nodes/netcdf

You can also try to use a more recent RiverObs version, but don't forget to use the associated SWORD version.

## Caveats
Although the large-scale simulator included in the toolbox provides fairly representative statistical errors, several simplifications or approximations are made:
* Spherical Earth approximate geolocation equations (loss of accuracy at high latitudes, >60°)
* No topography is taken into account 
  * Radar geometry grid constructed on sphere
  * No geometric distortions, no layover 
* Simplified representation of water height (several options) 
  * Spatially constant height for each water body (but possibility to vary height over time, cycle)
  * Spatially random correlated heights, and 2D polynomial model (with synthetic slopes)
  * Also possible to inject “true” heights from models (after simulation)
  * Random effective instrument noise added to height (and propagated to geolocation)
* Idealized pixel cloud processing 
* Synthetic "dark water" model (correlated random fields used to simulate low reflectivity areas)
* Geoid (mean tide corrected EGM-2008), tropospheric and cross-over residual errors simulated

When to use the simulator:
* To familiarize with SWOT HR products, if needed over large areas and over time (multitemporal series)
* To study the inpact of the geometrical shapes of the water bodies on River and Lake processing 
* When a simplified representation of phenomenology, hydrological characteristics and errors is sufficient (e.g. no layover, artifical water slope, basic error models)  

```
If a higher degree of realism is necessary to conduct a study, lower-level simulators and processors need to be employed. 
These are not publicly available, but SWOT Science Team members can contact the SWOT Algorithm Development Team for support. 
```

Product formats and algorithms:
* The product formats correspond to the current official versions, but are likely to evolve. Some data fields are at this stage void (various flags, some uncertainty indicators…).
* The processing algorithms will also continue to evolve, as today's prototypes are progessively refined into operational software. 

Last modifications:
In the large scale simulator:
* Uncertainties of geolocated heights added
* Multilook adaptive averaging implemented
* Land pixels around water bodies added (label 1 and 2 in classification field)
* Near-range computaton improved
* Some fields added or made more realistic

In the processing chain
* Lake tile processing improved, new product format (three shapefiles)
* Lake single pass processing added (cf leman and france_pekel new dataset to test it, can't be pushed on github, but can't be share through CNES cluster if needed)
```bash
% python ../../scripts/laketile_to_lakesp.py output/lakesp rdf/multi_lake_sp_command.cfg output/lake/lake-annotation_*
```


## Installation procedure

### Get the repository
1. Clone __swot_hydrology_toolbox__ repo

```bash
% git clone https://github.com/cnes/swot-hydrology-toolbox.git
```
The repository __swot_hydrology_toolbox__ should be assignated to the __SWOT_HYDROLOGY_TOOLBOX__ variable.

```bash
% export SWOT_HYDROLOGY_TOOLBOX=your_installation_path/swot-hydrology-toolbox
```

2. Clone __RiverObs__ repo

```bash
% git clone https://github.com/SWOTAlgorithms/RiverObs.git
```

The repository __RiverObs__ should be assignated to the __RIVEROBS__ variable.

```bash
% export RIVEROBS=your_installation_path/RiverObs
```

### Dependencies

The dependencies of swot-hydrology-toolbox are:
* GDAL
* netcdf
* proj4
* libspatialindex
* CGAL (optional, if using HULL_METHOD=1.0)
* and the following Python modules:
  * numpy
  * scipy
  * matplotlib
  * scikit-learn
  * scikit-image
  * lxml
  * netCDF4
  * xarray
  * dask
  * distributed
  * pyproj
  * jupyter
  * notebook
  * statsmodels
  * pysal
  * pandas
  * pytables
  * Shapely
  * Fiona
  * sphinx
  * numpydoc
  * rtree
  * mahotas
  * utm
  * pygeodesy

### Python environment installation

#### Setting up a conda environment

To create a conda environment, execute

```bash
cd $SWOT_HYDROLOGY_TOOLBOX
conda env create -f environment.yml
```

To activate this environment, if the first option was used, type
```bash
conda activate swot-env
```

To deactivate this environment, type
```bash
conda deactivate
```

## Execute the toolbox

After activating your Python environment, you have to set your PYTHONPATH variables:
```bash
export PYTHONPATH=$SWOT_HYDROLOGY_TOOLBOX/processing/src/:$RIVEROBS/src:$PYTHONPATH
```

An example dataset showing how to configure and run simulation and processing is available under /test.

The needed input for the overall chain include:
* An orbit file (provided)
* A water mask in shapefile format covering the area you want so simulate (see example under /test)
* A river database in shapefile format (e.g. GRWL)
* A lake database in shapefile format
* Various configuration files (examples provided)
