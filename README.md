# All transit

[![Build Status](https://travis-ci.org/kylebarron/all-transit.svg?branch=master)](https://travis-ci.org/kylebarron/all-transit)

All transit in the continental US, as reported by <https://transit.land>.
Inspired by [_All Streets_](https://benfry.com/allstreets/map5.html).

```bash
git clone https://github.com/kylebarron/all-transit
cd all-transit
pip install transitland-wrapper
mkdir -p data

# All operators
transitland operators --geometry data/gis/states/states.shp > data/operators.geojson

# All routes
transitland routes --geometry data/gis/states/states.shp > data/routes.geojson

# All operator `onestop_id`s
cat data/operators.geojson | jq '.properties.onestop_id' | uniq |  tr -d \" > data/operator_onestop_ids.txt

# All stop `onestop_id`s for those routes:
cat data/routes.geojson | jq '.properties.stops_served_by_route[].stop_onestop_id' | uniq | tr -d \" > data/stop_onestop_ids.txt

# All route stop patterns `onestop_id`s for those routes:
cat data/routes.geojson | jq '.properties.route_stop_patterns_by_onestop_id[]' | uniq | tr -d \" > data/route_stop_patterns_by_onestop_id.txt

# All stops (hopefully faster)
rm data/stops.geojson
cat data/operator_onestop_ids.txt | while read operator_id
do
    transitland stops \
        --served-by $operator_id --per-page 1000 >> data/stops.geojson
done

# All route-stop-patterns (completed relatively quickly, overnight)
transitland onestop-id --file data/route_stop_patterns_by_onestop_id.txt > data/route-stop-patterns.json

# All schedule-stop-pairs
mkdir -p data/ssp/
cat data/operator_onestop_ids.txt | while read operator_id
do
    transitland schedule-stop-pairs \
        --operator-onestop-id $operator_id --per-page 1000 --active | gzip > data/ssp/$operator_id.json.gz
done
```

### Put into vector tiles

```bash
# Writes mbtiles to data/routes.mbtiles
# The -c is important so that each feature gets output onto a single line
cat data/routes.geojson | jq -c -f code/jq/routes.jq | bash code/tippecanoe/routes.sh

# Writes mbtiles to data/operators.mbtiles
bash code/tippecanoe/operators.sh data/operators.geojson

# Writes mbtiles to data/stops.mbtiles
# The -c is important so that each feature gets output onto a single line
cat data/stops.geojson | jq -c -f code/jq/stops.jq | bash code/tippecanoe/stops.sh
```

Combine into single mbtiles
```bash
tile-join \
    -o data/all.mbtiles \
    --no-tile-size-limit \
    --force \
    stops.mbtiles operators.mbtiles routes.mbtiles
```