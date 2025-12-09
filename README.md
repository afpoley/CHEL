## Spatial Join Example (Point-in-Hex Join)

import geopandas as gpd

# -------------------------------------------------------------------
# 1. Load participant location data (must be a GeoDataFrame of points)
# -------------------------------------------------------------------
# Example structure:
# participant_locations = [
#     {"participant_id": 1, "lat": 40.12, "lon": -75.33},
#     {"participant_id": 2, "lat": 34.05, "lon": -118.24},
# ]
#
# Convert list of dicts → GeoDataFrame
# (Replace this section with your actual participant data source)
import pandas as pd
from shapely.geometry import Point

participant_df = pd.DataFrame(participant_locations)
participant_gdf = gpd.GeoDataFrame(
    participant_df,
    geometry=gpd.points_from_xy(participant_df.lon, participant_df.lat),
    crs="EPSG:4326",
)

# -------------------------------------------------------------------
# 2. Load CHEL hexagons (ensure same CRS as participant points)
# -------------------------------------------------------------------
chel = gpd.read_file("CHEL_2022.gpkg")

# Reproject CHEL hexagons if needed
if chel.crs != participant_gdf.crs:
    chel = chel.to_crs(participant_gdf.crs)

# -------------------------------------------------------------------
# 3. Perform spatial join: assign each participant to a hexagon
# -------------------------------------------------------------------
joined = gpd.sjoin(
    participant_gdf,
    chel,
    how="left",       # keep all participants
    predicate="intersects"   # point-in-polygon
)

# -------------------------------------------------------------------
# 4. (Optional) Select final columns or rename CHEL ID
# -------------------------------------------------------------------
# Example: keep participant_id + hex_id
output = joined[["participant_id", "h3_hex_id", "geometry"]]

print(output.head())
