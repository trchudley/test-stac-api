# Testing the construction of a STAC and STAC API

This repository tests the production of a STAC catalogue for a simple hosted dataset, and contains notes as to how to make the STAC 'dynamic' (i.e. accessible via a local STAC API) using a Docker environment.

## Test data

200 m resolution crevasse fraction map of Greenland, split into six regions (Chudley, 2022, https://doi.org/10.5281/zenodo.6779087)

## Construct metadata STAC

See `make_stac.ipynb`

## Get API running

### Install Docker

Using `brew` within MacOS environment.

### Structure directory

```
stac-api/
├── docker-compose.yml
└── data/
    └── stac-catalog/
        └── (your STAC files go here)
```

Do **not** put Docker files inside your STAC.

###  Create `docker-compose.yml`

Create `stac-api/docker-compose.yml` with containing instructions.

### Run the docker.

```docker compose up```

(To shut down, `docker compose down`)

From here, the STAC (once built) will be available at http://localhost:8080, and the GUI interface at http://localhost:8081.

### Load the collections

In a seperate terminal, run

```zsh
for collection in ./data/stac-catalog/*/collection.json; do
  pypgstac load collections "$collection" \
    --dsn postgresql://username:password@localhost:5439/postgis
done

for item in ./data/stac-catalog/*/*/*.json; do
    pypgstac load items "$item"  \
    --dsn postgresql://username:password@localhost:5439/postgis
done
```

NB that pypgstac and associated dependencies need to be installed.

This is worryingly slow for even six `Item`s - there must be a better way but it works for now.

### Update the collection

Add a method flag:

 - `--method insert` - Add new only (fails if exists) - default
 - `--method upsert` - Add new or update existing - recommended for updates
 - `--method ignore` - Skip if exists, add if new

Upsert is most general and safe.

```zsh
for collection in ./data/stac-catalog/*/collection.json; do
  pypgstac load collections "$collection" \
    --dsn postgresql://username:password@localhost:5439/postgis \
    --method upsert
done

for item in ./data/stac-catalog/*/*/*.json; do
    pypgstac load items "$item"  \
    --dsn postgresql://username:password@localhost:5439/postgis \
    --method upsert
done
```
