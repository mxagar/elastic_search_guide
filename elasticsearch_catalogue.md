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

Contents:

- Create and delete indices
- Create/Index, delete and get documents
- Update documents: modify fields, add fields, scripts, upsert (insert or create)
- Optimistic Concurrency Control: dealing with parallel updates
- Update and Delete by Query: Update/Delete a filtered set of documents

```
### --- Create and delete indices

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

### --- Create/Index, delete and get documents

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

# Get all Documents in the index products
# The result is a JSON in which result['hits']['hits']
# contains a list of al Document JSONs
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}

### --- Update documents: modify fields, add fields, scripts, upsert (insert or create)

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

### --- Optimistic Concurrency Control: dealing with parallel updates

# Get Document with ID 100
# _primary_term and seq_no are in the metadata
# We take them to formulate the conditioned update request
GET /products/_doc/100

# Update Document/product with ID 100 
# if _primary_term == 1 and seq_no == 5 (reference values obtained in previous query)
# If _primary_term and seq_no don't match, we get an error
POST /products/_update/100?if_primary_term=1&if_seq_no=5
{
  "doc": {
    "in_stock": 123
  }
}

### --- Update and Delete by Query: Update/Delete a filtered set of documents

# Update a set of filtered Documents: _update_by_query
# Write the update in "script"
# Write the filterin "query"
# Errors/conflicts can occur; by default the request is aborted
# but we can specify to proceed if we want
POST /products/_update_by_query
{
  "conflicts": "proceed",
  "script": {
    "source": "ctx._source.in_stock--"
  },
  "query": {
    "match_all": {}
  }
}

# Delete all Documents that match the query
# in this case ALL DOCUMENTS!
# If errors/conflicts occur, the request is aborted
# unless we specify "conflicts": "proceed"
POST /products/_delete_by_query
{
  "conflicts": "proceed",
  "query": {
    "match_all": { }
  }
}
```