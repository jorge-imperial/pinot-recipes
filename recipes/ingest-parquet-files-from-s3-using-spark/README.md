# Ingest Parquet Files from a S3 Bucket into Pinot Using Spark

> In this recipe we'll learn how to ingest Parquet formatted data from an AWS S3 bucket into a Pinot cluster.

<table>
  <tr>
    <td>Pinot Version</td>
    <td>0.9.3</td>
  </tr>
  <tr>
    <td>Schema</td>
    <td><a href="config/events_schema.json">config/events_schema.json</a></td>
  </tr>
    <tr>
    <td>Table Config</td>
    <td><a href="config/events_table.json">config/events_table.json</a></td>
  </tr>
      <tr>
    <td>Ingestion Job</td>
    <td><a href="config/events-s3.yml">config/events-s3.yml</a></td>
  </tr>
</table>

This is the code for the following recipe: https://dev.startree.ai/docs/pinot/recipes/ingest-parquet-files-from-s3-using-spark

***

Clone this repository and navigate to this recipe:

```bash
git clone git@github.com:startreedata/pinot-recipes.git
cd pinot-recipes/recipes/ingest-parquet-files-from-s3-using-spark
```

Launch Pinot Cluster

```bash
bin/quick-start-batch.sh
```

Launch Spark Shell

```bash
<SPARK_HOME>/bin/spark-shell
```

Convert CSV File to Parquet Format

```scala
val df = spark.read.format("csv").option("header", true).load("/path/to/events.csv")

df.write.option("compression","none").mode("overwrite").parquet("/path/to/output")

:q // to exit
```

Upload Parquet Files


```bash
aws s3 cp /path/to/output s3://pinot-spark-demo/rawdata/ --recursive
```

Create Table and Schema

```bash
./pinot-admin.sh AddTable \
  -tableConfigFile /path/to/events_table.json \
  -schemaFile /path/to/events_schema.json -exec
```

Ingest Parquet

```bash
export PINOT_VERSION=0.8.0
export PINOT_DISTRIBUTION_DIR=/path/to/apache-pinot-0.8.0-bin
export SPARK_HOME=/path/to/spark-2.4.8-bin-hadoop2.7

cd ${PINOT_DISTRIBUTION_DIR}

${SPARK_HOME}/bin/spark-submit \
  --class org.apache.pinot.tools.admin.command.LaunchDataIngestionJobCommand \
  --master "local[2]" \
  --deploy-mode client \
  --conf "spark.driver.extraJavaOptions=-Dplugins.dir=${PINOT_DISTRIBUTION_DIR}/plugins -Dlog4j2.configurationFile=${PINOT_DISTRIBUTION_DIR}/conf/pinot-ingestion-job-log4j2.xml" \
  --conf "spark.driver.extraClassPath=${PINOT_DISTRIBUTION_DIR}/plugins/pinot-batch-ingestion/pinot-batch-ingestion-spark/pinot-batch-ingestion-spark-${PINOT_VERSION}-shaded.jar:${PINOT_DISTRIBUTION_DIR}/lib/pinot-all-${PINOT_VERSION}-jar-with-dependencies.jar:${PINOT_DISTRIBUTION_DIR}/plugins/pinot-file-system/pinot-s3/pinot-s3-${PINOT_VERSION}-shaded.jar:${PINOT_DISTRIBUTION_DIR}/plugins/pinot-input-format/pinot-parquet/pinot-parquet-${PINOT_VERSION}-shaded.jar:${PINOT_DISTRIBUTION_DIR}/plugins/pinot-file-system/pinot-hdfs/pinot-hdfs-${PINOT_VERSION}-shaded.jar" \

local://${PINOT_DISTRIBUTION_DIR}/lib/pinot-all-${PINOT_VERSION}-jar-with-dependencies.jar \
  -jobSpecFile '/Users/dunith/Work/ingestion-specs/events-s3.yml'
```

Query Pinot

```
select ToDateTime(ts, 'yyyy-MM-dd') as t_date, count(distinct userId) as DAU
from events
where ts > now() - 86400000*30*3
group by t_date
order by 1 asc
```
