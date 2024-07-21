# Elastic Search: Command Catalogue

This document compiles the most important commands of Elastic Search.

For a more detailed guide on Elastic Search, check [`README.md`](./README.md).

## Basic

```
GET /_cluster/health

GET /_cat/nodes?v

GET /_cat/indices?v

GET /_cat/shards?v
```

## Manage Documents

```
# Create an index
PUT /pages

# Delete an index
DELETE /pages

# Create an index with specific properties
PUT /products
{
  "settings": {
    "number_of_shards": 2
  }
}

# Simple JSON Document indexed = created
# An _id is automatically assigned if not provided
# As we can see in the returned JSON
# Also, we see we have 2 shards
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}

# Here, we index a Document = we create one
# but we force it to be of id 100
# Note that the HTTP method is PUT, not POST!
# We can use the same code
# to replace entirely a Document
PUT /products/_doc/100
{
  "name": "Toaster",
  "price": 49,
  "in_stock": 4
}

# Delete the document with ID 100
DELETE /products/_doc/100

# Retrieve Document by ID
GET /products/_doc/100

# Update an existing field
# To update a Document, 
# we need to pass a JSON with an object "doc"
# The returned JSON contains "result": "updated"
POST /products/_update/100
{
  "doc": {
    "in_stock": 3
  }
}

# Add a new field: "tags": ["electronics"]
POST /products/_update/100
{
  "doc": {
    "tags": ["electronics"]
  }
}

# Go to Document with ID 100
# Decrease by one unit the field in_stock
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}

# Go to Document with ID 100
# Assign the value 10 to the field in_stock
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock = 10"
  }
}

# Go to Document with ID 100
# Modify field in_stock with the values in params
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}

# Go to Document with ID 100
# If in_stock == 0, perform no operation
# Else, decrease in_stock in one unit
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock == 0) {
        ctx.op = 'noop';
      }
      
      ctx._source.in_stock--;
    """
  }
}

# Go to Document with ID 100
# If in_stock < 0, delete the Document (product)
# Else, decrease in_stock in one unit
POST /products/_update/100
{
  "script": {
    "source": """
      if (ctx._source.in_stock < 0) {
        ctx.op = 'delete';
      }
      
      ctx._source.in_stock--;
    """
  }
}

# Upsert: Update or Insert/Create
# If the product 101 doesn't exist, this creates it
# Else, it increases in_stock in one unit
POST /products/_update/101
{
  "script": {
    "source": "ctx._source.in_stock++"
  },
  "upsert": {
    "name": "Blender",
    "price": 399,
    "in_stock": 5
  }
}
```