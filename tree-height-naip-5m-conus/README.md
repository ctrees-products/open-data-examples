# CTrees CONUS Canopy Height & Cover at 5 m

A guided tour of CTrees' **CONUS canopy tree height and cover at 5 m**, derived from NAIP
aerial imagery and published as an [Icechunk](https://icechunk.io) repository on
[Arraylake](https://docs.earthmover.io) (`ctrees/tree-height-naip-5m-conus`).

**The question this notebook answers:** *Can we see the 2021 Caldor Fire in this dataset,
and how much canopy did it destroy?* The notebook is never told where the fire was. It
differences two years of canopy height over a loose rectangle and lets the burn scar
appear on its own.

## Contents

| File | Description |
|---|---|
| `get-to-know-a-dataset.ipynb` | The tutorial notebook |
| `environment.yml` | Conda environment (`ctrees-tree-height-env`) |
| `aoi/caldor_fire_aoi.shp` | Rectangular AOI around the Caldor Fire region (used by the notebook) |
| `aoi/dixie_fire_shape.shp` | Alternative AOI around the Dixie Fire, to try on your own |

## Getting started

```sh
conda env create -f environment.yml
conda activate ctrees-tree-height-env

al auth login      # opens a browser; caches an Arraylake session locally
```

`al auth login` is the only authentication step. The notebook never reads an API key;
`arraylake.Client()` picks up the cached session automatically. Then open
`get-to-know-a-dataset.ipynb` and run it top to bottom.

## Dataset at a glance

- **One Zarr group per state** (48 CONUS states), each with its **own `time` axis**,
  because NAIP flies on a state-by-state cycle. California has 2020 and 2022.
- **Variables:** `tree_height` (metres, **median** of the native 60 cm predictions),
  `tree_cover` (0 = non-forest, 1 = forest) and `tree_cover_percentage` (whole percent,
  0–100, with no scale factor).
- **Grid:** EPSG:4326, step `0.00004°` (~4.4 m), chunked `512 x 512`, sharded `4096 x 4096`.
- **Fill value:** `255`. xarray decodes it to `NaN` on open. Zero is real (bare ground).
- **Coverage is partial** within each state's bounding box. Missing regions read as fill.

## What you do in this tutorial

1. **Set up and connect**
   - Connect to Arraylake and open a read-only Icechunk session pinned to one snapshot.
   - List the state groups and open California lazily with `chunks={}`.

2. **Understand the data**
   - Inspect dtypes, fill values, chunking and units for each variable.
   - Learn why `tree_height` is a median, and how that affects comparisons.
   - Overview of Zarr v3 sharding, Icechunk versioning, and the multiscale overview pyramids.

3. **Load an area of interest**
   - Read the AOI shapefile with GeoPandas and subset the California array to its bounds.
   - Select 2020 and 2022, and keep only pixels valid in **both** years.

4. **Visualize the fire**
   - Block-average for display only (using factors that divide the chunk size).
   - Plot 2020, 2022 and the height difference. The Caldor burn scar shows up in red.

5. **Quantify canopy loss**
   - Compute true ground area with the spherical cell-area formula, and see why
     counting pixels on a lat/lon grid overstates area.
   - Delineate the burn scar from the data (canopy pixels losing more than 5 m).
   - Compare statistics inside the scar against the whole window.
   - Plot the distribution of height change with a streamed `dask` histogram.
   - All statistics come from a single `dask.compute` pass over the data.

6. **Explore further**
   - Discusses the caveats: optical heights, median statistics, and canopy loss vs. fire
     (salvage logging).
   - Proposes an open question: can canopy height change map burn severity better than dNBR?
   - Starter code aggregates loss onto ~1 km blocks and exports a GeoTIFF for joining to
     MTBS severity polygons.

> **Heads up:** the Caldor window is about 1 billion pixels per variable-year. The
> statistics cell reads that data and takes a few minutes. If you are resource-constrained,
> shrink the AOI.

Overall: **open the state group → subset to an AOI → difference two years → visualize →
compute area-correct statistics** for canopy height and cover.

## Related

See the sibling [30 m dataset](../tree-height-naip-30m-conus), which adds
`tree_height_mean` and `tree_height_std` at roughly 1/40th the pixel count. The two grids
are not pixel-nested, so comparing them requires an explicit resample.
