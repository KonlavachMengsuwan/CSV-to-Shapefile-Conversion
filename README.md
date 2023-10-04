# CSV-to-Shapefile-Conversion

Google Colab: https://colab.research.google.com/drive/18b8hnDZITyc4508KTY6mEeMIg_5lxDaK?usp=sharing

```
# Import necessary libraries
import pandas as pd
import geopandas as gpd
import re
from shapely.geometry import Point
import zipfile

# -----------------------------------------------------------------------------
# PART 1: Latitude and Longitude Conversion Functions
# -----------------------------------------------------------------------------

# Convert DMS (Degrees, Minutes, Seconds) format to Decimal Degrees format
def dms_to_dd(dms):
    degrees, minutes, seconds, direction = re.split('[°\'"]+', dms)
    dd = float(degrees) + float(minutes)/60 + float(seconds)/3600
    if direction in ('S', 'W'):
        dd *= -1
    return dd

# Convert Decimal Minutes format to Decimal Degrees format
def dm_to_dd(dm):
    degrees, minutes, direction = re.split('[°\']+', dm)
    dd = float(degrees) + float(minutes) / 60
    if direction in ('S', 'W'):
        dd *= -1
    return dd

# Determine the format of the coordinate and convert to Decimal Degrees format
def convert_to_dd(coord):
    if '"' in coord:
        return dms_to_dd(coord)
    else:
        return dm_to_dd(coord)

# -----------------------------------------------------------------------------
# PART 2: Read the CSV and Convert Latitude and Longitude Formats
# -----------------------------------------------------------------------------

# Load the CSV file into a DataFrame
df = pd.read_csv('path_to_your_file.csv')

# Convert the Latitude and Longitude columns to Decimal Degrees format
df['Latitude_DD'] = df['Latitude'].apply(convert_to_dd)
df['Longitude_DD'] = df['Longtitude'].apply(convert_to_dd)

# Save the updated DataFrame to a new CSV file
df.to_csv('updated_file.csv', index=False)

# -----------------------------------------------------------------------------
# PART 3: Convert DataFrame to GeoDataFrame and Save as Shapefile
# -----------------------------------------------------------------------------

# Convert the DataFrame to a GeoDataFrame
geometry = [Point(xy) for xy in zip(df['Longitude_DD'], df['Latitude_DD'])]
geo_df = gpd.GeoDataFrame(df, geometry=geometry)

# Set the coordinate reference system (CRS) to WGS 84 (EPSG:4326)
geo_df.crs = "EPSG:4326"

# Save the GeoDataFrame as a shapefile
shapefile_name = 'latitude_longitude_points'
geo_df.to_file(f"{shapefile_name}.shp")

# -----------------------------------------------------------------------------
# PART 4: Zip all components of the shapefile for easy sharing
# -----------------------------------------------------------------------------

with zipfile.ZipFile(f"{shapefile_name}.zip", 'w') as zipf:
    for ext in [".shp", ".shx", ".dbf", ".prj"]:
        zipf.write(f"{shapefile_name}{ext}")

print("Conversion and shapefile creation complete!")

```

|index|Latitude\_DD|Longitude\_DD|
|---|---|---|
|0|51\.81143333333333|14\.408266666666666|
|1|51\.80878333333333|14\.408133333333334|
|2|51\.80863333333333|14\.409016666666666|
|3|51\.80843333333333|14\.409966666666667|
|4|51\.808366666666664|14\.410516666666666|
