# open-data-examples
Examples for access CTrees' Open Datasets

## AGB 100m 

**What you do in this tutorial:**

1. **Set up the environment**
   - Install required Python packages: `rioxarray`, `icechunk`, etc.
   - Import libraries: NumPy, pandas, GeoPandas, xarray, icechunk`, and rioxarray.

2. **Load the global AGB dataset**
   - Connect to the `ctrees/agb_100m_global` repository.
   - Open the `aboveground_biomass` Icechunk dataset with xarray.
   - Inspect the `agb` variable:
     - `time` dimension: years 2000-2025.
     - `x` and `y` dimensions: global grid coverage.
     - Metadata: projection (spatial reference), units, and scale factor.

3. **Clip data by a bounding box (WGS84)**
   - Define a bounding box `[minx, miny, maxx, maxy]`.
   - Subset `ds.agb` to that region.
   - Plot biomass maps for selected years (e.g., 2020 and 2025).

4. **Export a single year as a GeoTIFF**
   - Select a specific year from the clipped data.
   - Save as a GeoTIFF with `rioxarray` (`.rio.to_raster()`).
   - Download the GeoTIFF from the Colab “Files” panel.

5. **Clip data by a geometry (e.g., admin boundary)**
   - Load a polygon geometry (e.g., GeoJSON) with GeoPandas.
   - Use the geometry’s bounds to crop the raster.
   - Clip the raster to the polygon using `rio.clip`.
   - Plot biomass for the geometry for different years.

6. **Compute and plot time-series statistics**
   - Clean the data:
     - Mask no-data values (e.g., `-9999`).
     - Set negative values to 0.
     - Apply the scale factor (e.g., divide by 10).
   - Compute average AGB density over the clipped area for all years.
   - Convert results to a pandas DataFrame.
   - Plot a time series of `avg_agb_density` vs. `time`.

Overall, the notebook guides you through a complete workflow:
**load global AGB → subset (bounding box or polygon) → visualize → export → compute + plot time-series statistics** for Above Ground Biomass.