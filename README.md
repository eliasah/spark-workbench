# Spark Workbench

![CI](https://github.com/eliasah/spark-workbench/actions/workflows/ci.yml/badge.svg)

A Dockerized Spark 3.5.1 + Jupyter workspace for local data exploration.  
The repository contains only infrastructure — large data files are external
assets mounted at runtime.

---

## Architecture

| Layer | Where it lives | How it gets there |
|---|---|---|
| Code & notebooks | this repo (`notebooks/`, `src/`) | committed |
| Docker image | built locally from `spark/Dockerfile` | `docker compose build` |
| Base runtime | Docker Hub (`apache/spark:3.5.1-…`) | pulled automatically |
| Data | arbitrary host path | `.env` → Docker volume mount |

The base image (`apache/spark`) is pulled from Docker Hub and already contains
Spark, Java 11, and Python 3. The Dockerfile only adds pandas, pyarrow,
notebook, and the standalone Scala REPL on top.

---

## Quick start

### 1 — Configure your data folder

```bash
cp .env.example .env
```

Edit `.env` and set `DATAMART_HOST_PATH` to the absolute path of your local
data directory:

```env
DATAMART_HOST_PATH=/path/to/local/datamart
```

That folder will be mounted into the container at `/datamart`.

---

### 2 — Build the image

```bash
docker compose build
```

Docker pulls the official Spark base image from Docker Hub, then installs
pandas, pyarrow, and notebook on top. No manual downloads required.

---

### 3 — Run

```bash
docker compose up
```

Stop with `Ctrl-C`; remove containers with `docker compose down`.

---

## Access

| Service | URL |
|---|---|
| Jupyter Notebook | http://localhost:8888 |
| Spark UI | http://localhost:4040 (available while a Spark session is active) |

No token is required — the notebook server starts without authentication for
local development convenience.

---

## PySpark example

Create a new notebook under `notebooks/` and paste:

```python
from pyspark.sql import SparkSession
import os

spark = SparkSession.builder \
    .appName("workbench") \
    .getOrCreate()

datamart = os.environ["DATAMART_ROOT"]   # /datamart

# --- read parquet ---
df = spark.read.parquet(f"{datamart}/my_table/")
df.printSchema()
df.show(5)

# --- read CSV ---
df_csv = spark.read.option("header", True).csv(f"{datamart}/my_file.csv")
df_csv.show(5)
```

---

## Scala REPL / spark-shell

The image ships two Scala entry points:

| Command | What it gives you |
|---|---|
| `spark-shell` | Spark Scala REPL — `SparkContext` and `SparkSession` pre-wired |
| `scala` | Standalone Scala 2.12 REPL (no Spark context) |

Both commands are on PATH inside the container. Open a shell into a running
container to use them:

```bash
# spark-shell (Spark context available as `spark` and `sc`)
docker compose exec spark-jupyter spark-shell

# standalone Scala REPL
docker compose exec spark-jupyter scala
```

Or start a one-off container without launching Jupyter:

```bash
docker compose run --rm spark-jupyter spark-shell
```

### spark-shell example

```scala
import spark.implicits._

val df = spark.read.parquet("/datamart/my_table/")
df.printSchema()
df.show(5)

// basic aggregation
df.groupBy("category").count().orderBy($"count".desc).show()
```

---

## Project structure

```
.
├── docker-compose.yml      # service definition + volume mounts
├── .env.example            # template — copy to .env and edit
├── .gitignore
├── README.md
├── notebooks/              # Jupyter notebooks (committed)
├── src/                    # shared Python source (committed)
└── spark/
    └── Dockerfile          # image definition
```

---

## Package versions

| Package | Version |
|---|---|
| Apache Spark | 3.5.1 |
| Scala | 2.12 |
| Java | 11 |
| pandas | 2.0.3 |
| pyarrow | 17.0.0 |
| Base image | `apache/spark:3.5.1-scala2.12-java11-python3-ubuntu` |
