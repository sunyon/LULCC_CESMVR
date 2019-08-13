# LULCC_CESMVR

Repository containing driving scripts and model configuration to set up CESM2 simulations used for land use land cover change experiments on NERSC CORI. 

The patch and model configuration settings to enable SE grids in CESMv2.0.0 is obtained from Colin Zarzycki. The input datasets used for the CESM experiments are obtained from NCAR. All input data that the namelists point to are available on NERSC at the specified paths.

The tutorial detailed here is obtained from Colin Zarzycki, and adapted for CORI by Anjana Devanand.

Model version used: CESMv2.0.0

## Repository structure

---scripts | ---user_mods | ---namelists | --postprocessing

## Tutorial to set up CESM-SE on CORI
### Grids used: ne30_ne30 | ne30-ne240 | ne240-ne240
We provide detailed notes on performing CESM simulations on NERSC CORI Haswell nodes using the SE dynamical core. The model may be configured to use any of the three grid combinations  
    1. ne30 - ne30    : coarse atm - coarse lnd  
    2. ne30 - ne240   : coarse atm - fine lnd  
    3. ne240 - ne240  : fine atm - fine lnd  
    
### Set user directory path
```
export BASE_DIR=<dir-of-choice>
```
### Checkout CESMv2.0.0 source code
```
cd $BASE_DIR
git clone -b release-cesm2.0.0 https://github.com/ESCOMP/cesm.git
cd cesm
./manage_externals/checkout_externals
```
### Checkout the LULCC_CESMVR configuration scripts and namelists
```
cd $BASE_DIR
git clone git@github.com:IMMM-SFA/LULCC_CESMVR.git
```

### Apply patch to make SE operational
Copy and apply patch to add grids and override checks in CAM to make SE operational. We need to move to the directory where cesm/ lives, copy a patch file, and apply the patch file. Colin has set patch to verbose, there should be ~7 "Hunk succeeded" messages for 5 files. Note that to patch, the destination folder needs to be named "cesm." It can be renamed after the patch is applied (if desired).

```
cd $BASE_DIR
cp $BASE_DIR/LULCC_CESMVR/user_mods/vr-cesm.cesm2.patch .
patch --verbose -p0 < vr-cesm.cesm2.patch
```

### Copy and replace the cime/config/cesm/config_grids.xml
```
cd $BASE_DIR
cd cesm/cime/config/cesm/
cp $BASE_DIR/LULCC_CESMVR/user_mods/config_grids.xml .
```
_Note_: config_grids.xml contains edits for both the ne240-ne240 (Colin) and ne30-ne240 grids (Anjana).

### Creating a new case with the cesm directory
_Note_: _user_mods/env_mach_specific.xml_ has been updated as of July 2019. The modules would have changed after a major OS upgrade on CORI during the first week of Aug 2019. This machine file has to be updated to setup and build a case using the new modules on CORI.  

Use the LULCC_CESMVR scripts to create and build a new case on the desired grid
```
cd $BASE_DIR
export year=<year-of-landuse>
./LULCC_CESMVR/scripts/cesm_create_case_script_<ne30_ne30/ne30_ne240/ne240_ne240>.sh
```

### Postprocessing model output

https://www.ncl.ucar.edu/Applications/Scripts/homme_1.ncl  
https://github.com/zarzycki/ncl-zarzycki/tree/master/projects/regridding  
NCL scripts provided in _postprocessing_ folder may be used to regrid the ne240 CLM outputs

## Additional notes about CESM-VR - from on Colin Zarzycki's help document
NUMNODES edits env_mach_pes.xml. A rough starting point is ~1000 CPUs, Model has been run as low as 288 and as high
as 4,608 on Cheyenne. The model should theoretically scale to ~20k CPUs, but probably not ideal to push it above ~8k.  

EPS_AAREA is a setting fix approved by Mariana Vertenstein that reduces the error tolerance associated with distorted grid elements. This does not impact conservation in F compsets.  

ATM_NCPL can be modified to control the CAM/CLM timestep, but 144/day (600s) is what I
used for our ASD runs here at NCAR.  

## Who do I talk to?
    maoyi.huang at pnnl.gov
    anjanadevanand at iitb.ac.in

## Reference:
Devanand, A., Huang, M., Lawrence, D. M., Zarzycki, C. M., Feng, Z. & Lawrence, P. J. Land use and land cover changes strongly modulate warm-season precipitation over the Central United States in CESM2-VR. _[under prep]_

## Acknowledgment
U.S. Department of Energy (DOE), Office of Science, as part of research in Multi-Sector Dynamics, Earth and Environmental System Modeling Program

