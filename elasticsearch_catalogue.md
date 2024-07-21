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

# Simple JSON Document
# An _id is automatically created
# As we can see in the returned JSON
# Also, we see we have 2 shards
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}

# Here, we index a Document
# but we force it to be of id 100
# Note that the HTTP method is PUT, not POST!
PUT /products/_doc/100
{
  "name": "Toaster",
  "price": 49,
  "in_stock": 4
}

# Retrieve Document by ID
GET /products/_doc/100
```