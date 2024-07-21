# Elastic Search Notebooks

This folder contains basic Python interactions with an Elastic Search cluster.

Check [`../README.md`](../README.md) to know how to get an Elastic cluster up and running.

Table of contents:

- [Elastic Search Notebooks](#elastic-search-notebooks)
  - [Setup](#setup)
  - [Notebooks](#notebooks)
    - [Elastic Search Usage Introduction](#elastic-search-usage-introduction)
    - [Data Structures for Search](#data-structures-for-search)

## Setup

Create a Python environment, e.g, with Conda:

```bash
# Create environment + activate it
conda env create --file conda.yaml
conda activate elastic

# If we add packages to the YAML
conda env update --name elastic --file conda.yaml --prune
```

## Notebooks

### Elastic Search Usage Introduction

Notebook: [`elastic_intro.ipynb`](./elastic_intro.ipynb).  
Contents:

- Connect to a cluster using its REST API via `requests`.
- Basic usage of the package `elasticsearch`.
- Basic usage of the package `elasticsearch-dsl`.

### Data Structures for Search

Notebook: [`search_data_structures.ipynb`](search_data_structures.ipynb):

- B-Trees for range searches of numerical and date values-
- Inverted Index for searching relevant documents based on text.
- Doc Values (Columnar data representatios) for fast field/column-wise aggregation operations. 

For more contenxt, see [../README.md](../README.md) - **Building and Index**.

