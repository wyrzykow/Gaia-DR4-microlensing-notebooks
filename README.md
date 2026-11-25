# Gaia DR4 microlensing notebooks

Worked examples for accessing, analysing and visualising Gaia DR4 microlensing data.
**The data are described in the Gaia DR4 publication by Wyrzykowski et al. (2026).**
Please cite that publication when using these data or the scientific methods illustrated here.

## Notebooks

| Notebook | What it does | Inputs and outputs |
| --- | --- | --- |
| [01 — Archive access](01_archive_access.ipynb) | Tests the Gaia Archive with a known-source query, inspects table metadata, and retrieves one source's coordinates, G magnitude and BP−RP. | Queries `gaiadr4.gaia_source`; retains the query and result locally. |
| [02 — Microlensing catalogue](02_download_microlensing_catalogue.ipynb) | Downloads every microlensing event and joins the Gaia-source columns. A left join retains events lacking a consolidated source row. Checks catalogue counts and duplicate source IDs. | Exports CSV, FITS or VOTable, with a checksummed provenance sidecar. Choose CSV for notebooks 04, 05 and 07. |
| [03 — Brightness-ordered catalogue](03_brightness_ordered_catalogue.ipynb) | Adds RA/Dec from `all_source_astrometry` and sorts events by baseline G magnitude, with source ID as a tie-breaker. Adds a brightness rank. Optionally verifies and attaches publication event names. | Saves an ordered CSV. Optional names come from `data/event_names.csv`; ranks alone are not official event names. |
| [04 — Single-event physical posteriors](04_single_event_physical_posteriors.ipynb) | Propagates one event's fitted uncertainties into microlensing parallax, lens mass, source/lens distances, proper motions and lens transverse velocity. Reports 16th/50th/84th percentiles and plots scalar and joint distributions. | Reads notebook 02's CSV. Uses 100,000 mass/distance draws and 50,000 kinematic draws with reproducible seeds. |
| [05 — Source colour–magnitude diagram](05_source_colour_magnitude_diagram.ipynb) | Builds a quality-filtered absolute colour–magnitude diagram using baseline source photometry and fitted source parallax, with optional extinction correction and an automatically downloaded random Gaia DR4 density background. | Reads notebook 02's CSV; writes PNG/PDF figures and a selected-source CSV. The DR4 background is downloaded and cached automatically; it can be disabled. |
| [06 — Epoch data and other products](06_download_epoch_data_and_products.ipynb) | Downloads the complete microlensing table, selected-source catalogue rows and all available DataLink products. Preserves complete FITS bundles, image HDUs and quality flags; validates files and records an inventory. | Starts with one source. Its final cell exports per-source epoch photometry and astrometry FITS files plus provenance to `data/<source_id>/` for notebook 07. |
| [07 — Paper-like plots](07_paper_like_plots.ipynb) | Produces paper-like G-band light curves and photometric/along-scan astrometric residual panels using the catalogue models. Shows CCD-to-transit averaging, explicit clipping masks and a full-range diagnostic. | Reads notebook 02's CSV and notebook 06's epoch exports. Writes figures, selected measurements, complete masks and checksummed provenance. |

## Installation

Use Python 3.11 or newer and open a terminal in this folder:

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m jupyter lab
```

The notebooks include their model helpers and use paths relative to this project.
No other local project or private checkout is required. The optional `cmasher`
package changes the plotting colour map; a standard Matplotlib map is used otherwise.

## Getting started

1. Run **01** to check archive access.
2. Run **02** with `OUTPUT_FORMAT = "csv"` to create
   `outputs/gaiadr4/gaia_dr4_microlensing_events_joined.csv`.
3. Use **03** for an ordered catalogue, **04** for one event's physical
   parameters, or **05** for the source colour–magnitude diagram.
4. For paper-like plots, set the same exact `SOURCE_ID` in **06** and **07**.
   Run **06**, including its final reading/export cell, then run **07**.
   Their default example is source `4048864055145726976`.

Notebook 06 initially downloads products for one source. To request every
microlensing source, set `DATALINK_SOURCE_IDS = []` and
`DATALINK_SOURCE_LIMIT = None`. Product availability varies: a missing product
is recorded separately from a failed request. A source needs usable photometry,
astrometry and the corresponding microlensing model parameters for notebook 07.

Optional reference inputs are explained in [data/README.md](data/README.md).
No catalogue downloads, epoch measurements or precomputed scientific results
are included in this repository.

## Archive configuration

```python
ARCHIVE_URL = "https://gea.esac.esa.int/"
TAP_SCHEMA = "gaiadr4"
DATA_RELEASE = "Gaia DR4"
AUTHENTICATE = False
```

The endpoint, TAP schema and DataLink release label are independent settings.
Public access uses no credentials. If login is required, enable authentication
and use your own account through the private password prompt. Optional Keychain
mode uses your own macOS entry; no account or password is supplied here.

Queries run sequentially and retain ADQL, job IDs, raw VOTables, timestamps and
release settings. Interrupted submitted jobs are resumed rather than replaced.
Authenticated mode checks account-wide capacity: at most five executing jobs
and 100 queued jobs across all sessions. Errors and truncated/overflow results
are rejected. Large jobs and DataLink downloads may take time; successful cached
results are reused after validation.

DataLink requests use `DATAMODEL_STANDARD`, `retrieval_type="ALL"`, FITS format
and `valid_data=False`. Rejected observations and their flags are retained for
explicit scientific filtering. Original FITS ZIPs remain the raw products.
All source IDs must remain decimal strings or exact 64-bit integers.

## Scientific conventions

Notebook 04 uses the microlensing-fit source astrometry. It samples reported
marginal errors independently because the full fit covariance is unavailable.
It illustrates uncertainty propagation without a Galactic mass or distance
prior; broad inverse-parallax tails need particular care.

Notebook 05 applies explicit fit and catalogue-quality cuts. Dereddening is
on by default and requires finite extinction estimates. Set `DEREDDEN = False`
for observed colours and magnitudes. The background also comes from Gaia DR4:
a bounded `random_index` window
provides an all-sky random subset, explicit CMD-quality cuts are applied, and a
seeded cap selects at most 150,000 eligible stars. The default window spans two
million random indices; fewer stars are used if fewer qualify. Parallax S/N > 5,
photometric/astrometric quality, and finite extinction estimates (when dereddening)
define the eligible subset. It is not a completeness-corrected Galactic sample.
The first run downloads the background; later runs reuse its checksummed ECSV
cache and provenance under `data/dr4_cmd_reference/`. No manual background file
is required. `USE_REFERENCE_BACKGROUND = False` makes a source-only diagram.

Notebook 07 averages finite CCD measurements into transits and applies one-pass
five-MAD-sigma clipping to astrometric lens-model residuals. It records flags,
rejected transits and figure display limits separately. The `used_by_agis_al`
flag is reported but is not a selection cut. Photometry has finite-value checks
without statistical clipping or a photometric flag cut. These are figure masks,
not a reconstruction of the internal CU7 fitting selection, and no refit is
performed after clipping. See Wyrzykowski et al. (2026) for the data description
and scientific context.

## Reproducibility

The supplied notebooks have cleared outputs and execution counts. Their Python
cells and notebook structure have been checked; the public archive downloads
have not been validated end-to-end during preparation. Live execution requires
an available Gaia DR4 archive and the advertised columns/products.

Downloads and generated figures are excluded by `.gitignore`. Keep provenance
sidecars and raw bundles with your local results. Notebook 07 checks source IDs,
release/schema and input checksums before plotting. Clear notebook outputs
before committing an executed copy.
