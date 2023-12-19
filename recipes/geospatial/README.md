# Geospatial Querying

> In this recipe we'll learn how to store and query geospatial objects.

<table>
  <tr>
    <td>Pinot Version</td>
    <td>1.0.0</td>
  </tr>
  <tr>
    <td>Schema</td>
    <td><a href="config/schema.json">config/schema.json</a></td>
  </tr>
    <tr>
    <td>Real-Time Table Config</td>
    <td><a href="config/_table.json">config/table.json</a></td>
  </tr>
</table>

This is the code for the following recipe: https://dev.startree.ai/docs/pinot/recipes/geospatial

***

```bash
git clone git@github.com:startreedata/pinot-recipes.git
cd pinot-recipes/recipes/geospatial
```

Spin up a Pinot cluster using Docker Compose:

```bash
docker-compose up
```

Ingest data into Kafka:

```bash
python datagen.py --sleep 0.0001 2>/dev/null |
jq -cr --arg sep ø '[.uuid, tostring] | join($sep)' |
kcat -P -b localhost:9092 -t events -Kø
```

Add tables and schema:

```bash
docker run \
   --network geospatial \
   -v $PWD/config:/config \
   apachepinot/pinot:1.0.0 AddTable \
     -schemaFile /config/schema.json \
     -tableConfigFile /config/table.json \
     -controllerHost "pinot-controller-geospatial" \
    -exec
```

Sample queries:

```sql
select ST_Within(point, polygon) AS inPolygon, 
       ST_AsText(polygon) AS polygon,
       ST_AsText(multiPolygon), 
       ST_AsText(point) AS point
from events 
WHERE ST_Within(point, polygon) = 1
limit 3
```

```sql
SELECT uuid, ST_DISTANCE(point, ST_Point(-122, 37, 1))
FROM events
WHERE ST_DISTANCE(point, ST_Point(-122, 37, 1)) < 50000
LIMIT 10
```
