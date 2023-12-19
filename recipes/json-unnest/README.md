# Unnest array values in JSON documents

> In this recipe we'll learn how to unnest/explode values in nested JSON documents.

<table>
  <tr>
    <td>Pinot Version</td>
    <td>0.10.0</td>
  </tr>
  <tr>
    <td>Schema</td>
    <td><a href="config/schema.json">config/schema.json</a></td>
  </tr>
    <tr>
    <td>Table Config</td>
    <td><a href="config/table.json">config/table.json</a></td>
  </tr>
      <tr>
    <td>Ingestion Job</td>
    <td><a href="config/job-spec.yml">config/job-spec.yml</a></td>
  </tr>
</table>


This is the code for the following recipe: https://dev.startree.ai/docs/pinot/recipes/json-unnest

***

Clone this repository and navigate to this recipe:

```bash
git clone git@github.com:startreedata/pinot-recipes.git
cd pinot-recipes/recipes/json-unnest
```

Spin up a Pinot cluster using Docker Compose:

```bash
docker compose up
```

Open another tab to add the `movie_ratings` table:

```bash
docker run \
   --network json \
   -v $PWD/config:/config \
   apachepinot/pinot:1.0.0 AddTable \
     -schemaFile /config/schema.json \
     -tableConfigFile /config/table.json \
     -controllerHost "pinot-controller-json" \
    -exec
```

Import [data/movies.json](data/movies.json) into Pinot:

```bash
docker run \
   --network json \
   -v $PWD/config:/config \
   -v $PWD/data:/data \
   apachepinot/pinot:1.0.0  LaunchDataIngestionJob \
  -jobSpecFile /config/job-spec.yml
```

Navigate to http://localhost:9000/#/query and run the following query:

```sql
select * 
from movie_ratings 
limit 10
```
