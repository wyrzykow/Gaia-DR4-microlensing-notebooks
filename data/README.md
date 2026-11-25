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

## Optional Gaia DR3 density background

Notebook 05 works without a comparison sample. To add a density background,
download a public Gaia DR3 reference using the following ADQL and save it as
`data/gaia_dr3_cmd_reference_extinction.vot.gz` (or change `REFERENCE_PATH`).
Then set `USE_REFERENCE_BACKGROUND = True`.

```sql
SELECT TOP 150000
    source_id, bp_rp, phot_g_mean_mag,
    parallax, parallax_error, parallax_over_error,
    ag_gspphot, ebpminrp_gspphot
FROM gaiadr3.gaia_source
WHERE random_index < 10000000
  AND parallax_over_error > 3
  AND parallax > 0
  AND bp_rp IS NOT NULL
  AND phot_g_mean_mag IS NOT NULL
  AND ag_gspphot IS NOT NULL
  AND ebpminrp_gspphot IS NOT NULL
```

This is a bounded comparison sample, not a complete Gaia DR3 catalogue.
Keep its query, job ID and download date with the local file.
