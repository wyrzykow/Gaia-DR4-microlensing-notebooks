# Local data inputs

This directory holds generated epoch inputs and optional reference tables.
Only this README is intended for Git; downloaded measurements stay local.

## Epoch inputs for paper-like plots

Run notebook 06 for the source selected in notebook 07, including its final
reading/export cell. It copies the original epoch FITS members without changing
their data and writes:

```text
data/<source_id>/epoch_photometry.fits
data/<source_id>/epoch_astrometry.fits
data/<source_id>/provenance.json
```

The source's original product must have the standard photometric columns
(`g_obs_time`, `g_mag`, and supplied flux/error/flag columns) and vector-column
astrometry (`obs_time_tcb`, `scan_pos_angle`, `centroid_pos_al`,
`centroid_pos_error_al`, parallax factors and `used_by_agis_al`). The notebook
validates these columns. Some astrometry FITS files identify the source in a
`SOURCEID` header instead of a table column.

Notebook 07 also reads notebook 02's joined CSV and its `.provenance.json`
sidecar from `outputs/gaiadr4/`. It rejects incompatible schema/release labels,
wrong source IDs and changed input checksums. If a product is unavailable,
notebook 06 records it in `products_not_returned`; a failed download is not
interpreted as an absent product.

An optional `astrometry_supplement.ecsv` in the source directory can replace
missing ordinary catalogue astrometry for the matching source. It must belong
to the same release, contain `source_id`, and provide the astrometric columns
listed in notebook 07's supplement block. Its use and checksum are recorded.

## Optional publication event names

To attach names in notebook 03, provide `data/event_names.csv` with these columns:

```text
event,source_id
```

The alternative ID header `sourceid` is also accepted. Preserve IDs as exact
integers or decimal strings. Use the publication mapping for the same release,
with rows in the same baseline-G brightness order as the queried catalogue.
Notebook 03 requires an exact source-ID/order match before attaching names.
Without this table it exports brightness ranks, not official event names.
Notebooks 04 and 07 use this optional table for matching event labels only.

## Automatically downloaded Gaia DR4 CMD background

Notebook 05 downloads its own background from `gaiadr4.gaia_source` on first use.
No reference table needs to be supplied. The default queries all qualifying rows
within `0 <= random_index < 2_000_000`, applying explicit parallax and catalogue
quality cuts, then keeps up to 150,000 finite eligible stars without replacement
with a fixed seed. The actual count is printed and recorded; increase
`REFERENCE_RANDOM_INDEX_LIMIT` for a larger candidate pool.

Because `random_index` defines the random subset, this introduces no sky or
brightness cut. The CMD eligibility cuts still select the represented population:
positive parallax/error, parallax S/N > 5, RUWE < 1.4, at least nine visibility
periods, G flux S/N > 50, BP/RP flux S/N > 20, reliable G photometry, and finite
colour/magnitude. Dereddened diagrams additionally need finite DR4 `ag` and
`ebpminrp` estimates. This is not a completeness-corrected Galactic sample.

The download uses metadata checks, sequential resumable asynchronous queries,
independent counts, and TAP overflow/error validation. It retains raw VOTables,
ADQL and job ledgers under `data/dr4_cmd_reference/tap/`. Each processed sample
has its own directory:

```text
data/dr4_cmd_reference/samples/<request_hash>/sample.ecsv
data/dr4_cmd_reference/samples/<request_hash>/provenance.json
```

The request hash includes endpoint, schema, release, query, quality thresholds,
dereddening, seed and sample cap. A validated cache is reused without archive
access; changed settings create another sample, and interrupted submitted jobs
are resumed. The plot also records the reference provenance alongside its PNG,
PDF and selected-source CSV in `outputs/gaiadr4/`.

Set `USE_REFERENCE_BACKGROUND = False` to omit the background. These generated
downloads remain local and are excluded from Git by the repository's `.gitignore`.
