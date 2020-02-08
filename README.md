# All transit

All transit in the continental US, as reported by <transit.land>. Like [_All
Streets_](https://benfry.com/allstreets/map5.html), but for transit.

```bash
git clone https://github.com/kylebarron/all-transit
cd all-transit
pip install transitland-wrapper
mkdir -p data

# All operators
transitland operators --geometry data/gis/states/states.shp > data/operators.geojson

# All routes
transitland routes --geometry data/gis/states/states.shp > data/routes.geojson

# All stops
transitland stops --geometry data/gis/states/states.shp > data/routes.geojson
```

Make into vector tiles:
```bash
tippecanoe \
    `# tileset name` \
    -n 'Transit routes' \
    `# attribution` \
    --attribution '<a href="https://transit.land/" target="_blank">© Transitland</a>' \
    `# Description` \
    --description 'Transit routes from Transitland API' \
    `# Define layer name: routes` \
    --layer='routes' \
    `# Read input in parallel` \
    -P \
    `# Include only the following attributes:` \
    --include='onestop_id' \
    --include='color' \
    --include='vehicle_type' \
    --include='name' \
    `# Apply feature filter from file` \
    -J feature_filter.json \
    `# Guess appropriate max zoom` \
    -zg \
    `# overwrite` \
    --force \
    `# Export path` \
    -o data/routes.mbtiles \
    `# Input geojson` \
    data/routes.geojson
```
