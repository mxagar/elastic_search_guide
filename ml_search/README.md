# Search Methods in Machine Learning

This folder contains some Python examples related to search methods.

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

Notebook: [`search_data_structures.ipynb`](search_data_structures.ipynb):

- B-Trees for range searches of numerical and date values-
- Inverted Index for searching relevant documents based on text.
- Doc Values (Columnar data representatios) for fast field/column-wise aggregation operations. 

For more contenxt, see [../README.md](../README.md) - **Building and Index**.

