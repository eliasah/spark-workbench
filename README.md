# Spark Workbench

A Dockerized Spark 3.5.1 + Jupyter workspace for local data exploration.  
The repository contains only infrastructure — large data files and Python wheel
archives are external assets that each developer supplies on their own machine.

---

## Architecture

| Layer | Where it lives | How it gets there |
|---|---|---|
| Code & notebooks | this repo (`notebooks/`, `src/`) | committed |
| Docker image | built locally from `spark/Dockerfile` | `docker compose build` |
| Python packages | `wheels/` (gitignored) | `pip download` — see below |
| Data | arbitrary host path | `.env` → Docker volume mount |

Because `wheels/` and data are not committed, every developer downloads them
once and configures their own local path.  CI servers or new machines reproduce
the environment by following the same two-step process.

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
Different machines can point to different paths; the committed `.env.example`
stays clean.

---

### 2 — Download Python wheels (once per machine)

Wheels must be downloaded **before** the first `docker compose build` because
the Docker build runs fully offline after this step.

**Linux / macOS:**

```bash
pip download \
  --dest wheels \
  --platform manylinux_2_17_x86_64 \
  --implementation cp \
  --python-version 38 \
  --abi cp38 \
  --only-binary=:all: \
  pandas==2.0.3 pyarrow==17.0.0 pyspark==3.5.1 notebook
```

**Windows (PowerShell):**

```powershell
pip download --dest wheels --platform manylinux_2_17_x86_64 --implementation cp `
  --python-version 38 --abi cp38 --only-binary=:all: `
  pandas==2.0.3 pyarrow==17.0.0 pyspark==3.5.1 notebook
```

> `notebook` provides the Jupyter server.  The remaining packages (`notebook`
> and its transitive dependencies) are resolved automatically by pip and stored
> in `wheels/`.  The `wheels/` directory is gitignored — do not commit it.

---

### 3 — Build the image

```bash
docker compose build
```

The build copies the contents of `wheels/` into the image and installs all
packages with `--no-index --find-links`, so no outbound network access is
needed at build time.

---

### 4 — Run

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

Both commands are on PATH inside the container.  Open a shell into a running
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

`/datamart` is available inside the shell because the same volume mount
applies to every process in the container.

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
├── spark/
│   └── Dockerfile          # image definition
└── wheels/                 # wheel cache (gitignored — download locally)
```

---

## Package versions

| Package | Version |
|---|---|
| Apache Spark | 3.5.1 |
| pyspark | 3.5.1 |
| Scala | 2.12.18 |
| Python | 3.8 |
| pandas | 2.0.3 |
| pyarrow | 17.0.0 |
| JDK | OpenJDK 11 |
