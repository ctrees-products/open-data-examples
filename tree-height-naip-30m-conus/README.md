# CTrees CONUS Canopy Height & Cover at 30 m

A guided tour of CTrees' **CONUS canopy tree height and cover at 30 m**, derived from NAIP
aerial imagery and published as an [Icechunk](https://icechunk.io) repository on
[Arraylake](https://docs.earthmover.io) (`ctrees/tree-height-naip-30m-conus`).

**The question this notebook answers:** *Where is California's canopy structurally
complex, and where is it uniform?* The answer uses `tree_height_std`, the within-cell
standard deviation of the native 60 cm height predictions. It separates planted canopy
(orchards) from natural forest even where height and cover look alike.

## Contents

| File | Description |
|---|---|
| `get-to-know-a-dataset.ipynb` | The tutorial notebook |
| `environment.yml` | Conda environment (`ctrees-tree-height-env`) |

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
- **Variables:**

  | variable | dtype | fill | what it is |
  |---|---|---|---|
  | `tree_height` | uint8 | 255 | **median** height of the 60 cm predictions in the cell (m) |
  | `tree_height_mean` | float32 | -9999 | **mean** height (m) |
  | `tree_height_std` | float32 | -9999 | **standard deviation** of height (m) |
  | `tree_cover` | uint8 | 255 | 0 = non-forest, 1 = forest |
  | `tree_cover_percentage` | uint8 | 255 | whole percent, 0–100 |

  xarray decodes both fill values to `NaN` on open.
- **Grid:** EPSG:4326, step `0.00025°` (~27.6 m), chunked `512 x 512`, sharded `2048 x 2048`.
- **Not a downsampled 5 m product.** Both resolutions reduce the same 60 cm source
  independently, and the grids are not pixel-nested.
- **Coverage is partial** within each state's bounding box. Missing regions read as fill.

## What you do in this tutorial

1. **Set up and connect**
   - Connect to Arraylake and open a read-only Icechunk session pinned to one snapshot.
   - List the state groups and open California lazily with `chunks={}`.

2. **Understand the data**
   - Inspect dtypes, fill values, `cell_methods` and units for each variable.
   - Learn why the three height variables are three different statistics, and why the
     mean sits above the median wherever there is canopy.
   - Overview of Zarr v3 sharding, Icechunk versioning, and the overview pyramid.

3. **Load three contrasting regions** (2022)
   - Central Valley orchards near Fresno: planted, even-aged, uniform.
   - Sierra Nevada mixed conifer near Shaver Lake: natural, multi-layered.
   - Coast redwood in inland Humboldt County: tall and structurally complex.

4. **Visualize structure**
   - Map median height and within-cell standard deviation side by side. The orchards
     almost vanish in the standard-deviation row.

5. **Compare regions**
   - Summary table of height, variability, skew (`mean − median`) and cover per region.
   - Histograms of `tree_height_std` vs. `tree_height`, which show that structure
     separates the regions where height alone overlaps.
   - Scale up to a state-wide map of structural variability (~2.8 km blocks).

6. **Explore further**
   - Discusses the caveats: low variability is not unique to orchards, within-cell vs.
     between-cell variability, and area calculations on a lat/lon grid.
   - Proposes an open question: can within-cell structure power a national map of managed
     vs. natural canopy, and how much carbon sits in each?
   - Starter code builds the four-feature table a classifier would train on.

> **Heads up:** the state-wide map reads several GB of data and takes a few minutes. It
> streams chunk by chunk, so it will not exhaust memory. If you only want to look at it,
> the overview pyramid in the Earthmover viewer is much cheaper.

Overall: **open the state group → load small windows → compare structural statistics →
scale to a state-wide map** for canopy height and cover.

## Related

See the sibling [5 m dataset](../tree-height-naip-5m-conus) for sub-pixel structure such
as street trees and narrow riparian strips. Comparing the two resolutions requires an
explicit resample.
