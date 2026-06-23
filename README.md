# wqaf-kenya

Water Quality Assurance Fund — exploration and analysis code for piped water population dynamics.

Aim: characterize the communities Aquaya's piped water systems serve in Kenya,
consider "piped-water community" characteristics, then apply that characteristics
thresholdprofile to *all* communities in an area to estimate likely piped-water
community counts in new regions / counties.

Notebooks were built in Google Colab — they mount Google Drive and read shared data
from `drive/MyDrive/Colab Notebooks/Data/dd-afkenya/`

**Uasin Gishu** is used as a reference county to derive thresholds or distributions
of piped water systems from. These thresholds/distributions are then applied to counties:
- Nakuru, Kericho, Kwale, Kilifi, and Nyeri

## Code Environment

### Running on Google Colab

If running the notebooks on Google Colab, most packages are pre-installed. Each 
notebooks features `!pip install` commands for the packages it requires. *These* 
*cells should not be run locally!*

As mentioned above, if on Google Colab, a drive directory containing required data
files must be mounted - code cells for this are left inthe notebooks - simply do not
execute them if running locally.

### Running locally

A [uv](https://docs.astral.sh/uv/) project (Python ≥ 3.12). Build the environment from
the lockfile and launch Jupyter:

```bash
uv sync
uv run jupyter lab
```

Geospatial packages: `geopandas`, `shapely`, `pyproj`, `pyogrio`, `rasterio`. Plus
`pandas`, and `mapclassify`, `folium`, `matplotlib`, `plotly` for visualization.
`nbstripout` strips notebook outputs on commit.

## Main analysis flow (start to finish)

Two notebooks are the primary pipeline: **load all data → determine the
characteristics of communities that have piped water → apply those characteristics as a
model to new communities to derive likely piped-water community counts.** They are two
implementations of that flow, differing in the underlying "community" dataset.

### `AF_Kenya_Waterpoint_Community_Analysis_PopClusters.ipynb`  *(primary)*

The more developed version, built on the **PopClusters** dataset — a pre-computed
population-cluster product derived for Sub-Saharan Africa c. 2020 from population
rasters, night-lights, etc. Each cluster carries population, urban/peri-urban/rural
class, area, and electrified-population attributes.

1. **Load** admin boundaries (COD), HOTOSM populated places, Aquaya waterpoints & labs,
   and the Kenya PopClusters — all reprojected to a metric CRS (`EPSG:21037`) so
   distances are in meters.
2. **Characterize** communities with piped water: spatially join waterpoints to
   PopClusters (within 500 m buffers), and compare populated places / waterpoints /
   labs inside vs. outside clusters. Produces stats and population distributions for
   clusters *with* vs. *without* a waterpoint — what a served community looks like
   (notably a population floor around ~200).
3. **Re-cluster into communities**: raw PopClusters are fragmented, so the notebook
   groups clusters within 500 m (later 250 m) into connected components via BFS
   traversal — collapsing ~13.4k raw clusters into ~109 coherent communities — then
   re-checks where waterpoints land to support more informed counts.

### `AF_Kenya_Waterpoint_Community_Analysis_GRID3.ipynb`

The same flow on **GRID3 Settlement Extents** — polygon settlement footprints
classified as Built-Up Area / Small Settlement Area / Hamlet, with population — instead
of PopClusters. Loads a wider set of reference layers (COD & HOTOSM populated places and
polygons, GRID3 SE versions 1.1 / 3.0, GRID3 gridded population), then computes
waterpoint-vs-settlement stats: the share of waterpoints inside settlement extents (and
within 500 m), and the share of settlements containing a waterpoint, broken down by
type with average size and population. A settlement-polygon alternative to the
PopClusters approach.

This approach was abandoned due to excessive mass-grouping of small populated areas,
leading to infeasible actual community identification. FOR REFERENCE ONLY.

## Supporting notebooks

### `AF_Kenya_Explore_Gridded_Datasets.ipynb`

Exploration NB for loading and inspecting candidate **gridded (mostly raster) population
datasets** — GRID3 gridded population estimates, Meta Data-for-Good high-res population
density, GHSL (population / built-up surface / degree of urbanization), and Kontur
population hexagons — against the same admin / populated-place / waterpoint layers.
Includes an experiment grouping GRID3 ~100 m grid points (within 105 m) into
community-like polygons by buffer-union. Feeds candidate data sources into the main
flow.

### `AF_Kenya_PopClusters_Ghana_Cmp.ipynb`

Runs the PopClusters analysis on **Ghana** (Ahafo and Bono regions, CRS `EPSG:2136`)
with the Aquaya/AFPW piped-systems data, as a cross-country sanity check on the Kenya
approach. Mirrors the populated-place / waterpoint-in-cluster stats and population
distributions, and sketches the same connected-component cluster-merging.
