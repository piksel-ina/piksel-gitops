## Commands with argo-workflows template

### Onetime Index

1. **Landsat STAC to DC Indexing**

```bash
argo submit --from workflowtemplate/datacube-stac-indexer \
  --parameter catalog="https://landsatlook.usgs.gov/stac-server/" \
  --parameter collections="landsat-c2l2-sr" \
  --parameter datetime="2024-01-01/2024-01-30" \
  --parameter arguments="--rename-product=ls9_c2l2_sr --options=query={\\\"platform\\\":{\\\"in\\\":[\\\"LANDSAT_9\\\"]}} --url-string-replace=https://landsatlook.usgs.gov/data,s3://usgs-landsat" \
  --parameter bbox="112,-10,118,-6" \
  -n argo-workflows \
  --serviceaccount argo-workflows-executor
```

2. **Sentinel 2 STAC to DC Indexing**

```bash
argo submit --from workflowtemplate/datacube-stac-indexer \
  -n argo-workflows \
  -p catalog="https://earth-search.aws.element84.com/v1/" \
  -p bbox="114,-9,116,-7" \
  -p collections="sentinel-2-c1-l2a" \
  -p datetime="2025-01-01/2025-01-31" \
  -p arguments="--rename-product='s2_l2a'"
```
