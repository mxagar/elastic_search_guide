# Elastic Search Notebooks

This folder contains basic Python interactions with an Elastic Search cluster.

Check [`../README.md`](../README.md) to know how to get an Elastic cluster up and running.

## Setup

Create a Python environment, e.g, with Conda:

```bash
# Create environment + activate it
conda env create --file conda.yaml
conda activate elastic

# If we add packages to the YAML
conda env update --name elastic --file conda.yaml --prune
```

## Notebook(s)

Notebook: [`elastic_intro.ipynb`](./elastic_intro.ipynb).  
Contents:

- Connect to a cluster using its REST API via `requests`.
- Basic usage of the package `elasticsearch`.
- Basic usage of the package `elasticsearch-dsl`.

