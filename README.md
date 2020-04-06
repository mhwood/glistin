# resample_GLISTIN_DEMs
This set of codes is a Python toolbox to download and resample GLISTIN-A radar interferometer digital elevation models from NASA's Oceans Melting Greenland campaign. 

## Getting Started
Required Python packages: ```numpy```,```scipy```, ```argparse```, ```requests```, ```pyresample```, ```utm```, ```netCDF4```, ```xarray```, ```osr```

From a fresh anaconda environment, it is recommended to use this installation sequence:
```
pip install requests
conda install -c conda-forge pyresample
pip install netCDF4
pip install xarray
conda install gdal
pip install pyproj
pip install utm
```

After the installing the required packages,
1. Determine a directory on your local machine where the Level 1 data and resampled output products will be stored. This directory is referred to as "dataFolder" in the scripts provided.
2. Determine the indicies of the GLISTIN-A DEMs to downloaded. Refer to the map provided [HERE](https://github.com/mhwood/glistin/blob/master/GLISTIN-A_DEM_Index_Domains.pdf) which indicates the location of swath indices.

## Steps to Generate Resampled GLISTIN-A DEMs in NetCDF format

1. Download the Level-2 GLISTIN-A DEMs and associated metadata using **download_level_2_data.py**
2. (Optional) Download a geoid layer to reference the elevation measurements to mean sea level
3. Resample the Level-2 data using **resample_GLISTIN_DEMs.py**

### Step 1: Download the Level-2 data GLISTIN-A DEMs and associated metadata using download_level_2_data.py

Use **download_level_2_data.py** to download level 2 data from the [UAVSAR website](https://uavsar.jpl.nasa.gov/).

Keywords:
- dataFolder (-d): Directory where data will be downloaded. (Required)
- indexList (-i): List of swath indices to download. (Optional. Default is to download all 81 swaths)
- yearList (-y): List of years to download each swath. (Optional. Default is to download swaths in all available years (2016-2019))

Example command to download all years for swath index 52:
```
python download_level_2_data.py -d '/path/to/dataFolder' -i 52 
```

After downloading the data, you will have the following files and directory structure:
```
> cd /path/to/dataFolder
> find .
.
./Raw
./Raw/2017
./Raw/2017/Data
./Raw/2017/Data/greenl_17914_17037_011_170321_ALTTBB_HH_04.hgt.grd
./Raw/2017/Metadata
./Raw/2017/Metadata/greenl_17914_17037_011_170321_ALTTBB_HH_04_metadata.txt
./Raw/2019
./Raw/2019/Data
./Raw/2019/Data/greenl_17914_19022_009_190413_ALTTBB_HH_01.hgt.grd
./Raw/2019/Metadata
./Raw/2019/Metadata/greenl_17914_19022_009_190413_ALTTBB_HH_01_metadata.txt
./Raw/2018
./Raw/2018/Data
./Raw/2018/Data/greenl_17914_18014_005_180315_ALTTBB_HH_01.hgt.grd
./Raw/2018/Metadata
./Raw/2018/Metadata/greenl_17914_18014_005_180315_ALTTBB_HH_01_metadata.txt
./Raw/2016
./Raw/2016/Data
./Raw/2016/Data/greenl_17914_16037_013_160330_ALTTBB_HH_03.hgt.grd
./Raw/2016/Metadata
./Raw/2016/Metadata/greenl_17914_16037_013_160330_ALTTBB_HH_03_metadata.txt
```

### Step 2 (Optional): Download a geoid layer to reference the elevation measurements to mean sea level 

To include a geoid file with your resampling, you must include a directory called 'Geoid' within your specified dataFolder that contains your geoid file. These scripts are designed to use the GOCO05c geoid, obtained via the following steps:
1. Go to http://icgem.gfz-potsdam.de/calcgrid
2. Under Model Selection, choose Longtime Model -> GOCO05c.
3. Under Functional Selecation, choose geoid.
4. Under Geographic Selection, set longitude bounds to -75.9 to -9.8, latitude bounds to 55.2 to 86.8, and Grid Step to 0.1.
5. Leave all other parameters as their default valies, and 'start computation'.
6. When file is complete, click Download Grid and save to dataFolder/Geoid
7. Finally, convert this file to a netCDF file using the **geoid_grid_to_nc.py** function.

Example command to convert the geoid grid to a NetCDF file:
```
python geoid_grid_to_nc.py -d '/path/to/dataFolder' -g 'GOCO05c_383e72b1d9fbea44d4c550a7446ff8fcb6a57aba0bfdd6293a3e4b72f86030aa.gdf'
```

### Step 3: Resample the Level-2 data using **resample_GLISTIN_DEMs.py**

To resample data, use **resample_GLISTIN_DEMs.py**

Keywords:
- dataFolder (-d): Directory where data will be downloaded. (Required)
- resolution (-r): Resolution for the resampling. Default is 50 m.
- fileIndices (-i): List of swath indices to resample. (Optional. Default is to resample all 81 swaths)
- years (-y): List of years to resample each swath. (Optional. Default is to download swaths in all available years (2016-2019))
- projection (-p): The projection of the output DEM. Input as ESPG:XXXX, default is to use the UTM Zone that corresponds to the center lat/long of the grid.
- addGeoid (-g): Choose 1 if you would like to add a geoid correction to the file, otherwise choose 0.

Example command to resample the DEMs for swath index 52 in years 2018 and 2019:
```
python resample_GLISTIN_DEMs.py -d '/path/to/dataFolder' -i 52 -y 2018 2019 -g 1
```



