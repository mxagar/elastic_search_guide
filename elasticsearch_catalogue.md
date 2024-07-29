# Elastic Search: Command Catalogue

This document compiles the most important commands of Elastic Search.

For a more detailed guide on Elastic Search, check [`README.md`](./README.md).

Table of contents:

- [Elastic Search: Command Catalogue](#elastic-search-command-catalogue)
  - [Basic](#basic)
  - [Manage Documents](#manage-documents)
  - [Analyzers](#analyzers)
  - [Mappings](#mappings)


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
- Batch/Bulk processing

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

# Delete all the documents of an index
POST /products/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}

# If the index was not created and we index
# a Document, the index will be created automatically
# and the Document added to it
PUT /products_test/_doc/1
{
  "price": 7.4
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

### --- Batch/Bulk processing

# Create new Documents
# We can list many (action, source) pairs
# Always newline after a line, also in last JSON
# Four actions: create, index, update, delete
# Each action needs a source (the Document fields), except delete
POST /_bulk
{ "index": { "_index": "products", "_id": 200 } }
{ "name": "Espresso Machine", "price": 199, "in_stock": 5 }
{ "create": { "_index": "products", "_id": 201 } }
{ "name": "Milk Frother", "price": 149, "in_stock": 14 }

# Update Documents + Delete
POST /_bulk
{ "update": { "_index": "products", "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_index": "products", "_id": 200 } }

# If all actions are for the same index,
# we can specify it in the API commad
POST /products/_bulk
{ "update": { "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_id": 200 } }

# Get all Documents in an index
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}

```

## Analyzers

```
# Here a text string is analyzed
# with the standard analyzer:
# no char filter, standard tokenizer, lowercase token filter
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "analyzer": "standard"
}

# Analyzer components explicitly defined:
# character filer, tokenizer, token filters
# This call produces the same results are before
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "char_filter": [],
  "tokenizer": "standard",
  "filter": ["lowercase"]
}
```

## Mappings

Contents:

- Adding explicit mappings
- Retrieving mappings
- Extending mappings to existing indices: adding new fields
- Dates
- Reindexing: creating new indices because we want to change a field

```
### --- Adding explicit mappings

# We create a mapping for the index reviews
# For simple types, we define their type key-value
# For object types, we need to nest a properties key again
PUT /reviews
{
  "mappings": {
    "properties": {
      "rating": { "type": "float" },
      "content": { "type": "text" },
      "product_id": { "type": "integer" },
      "author": {
        "properties": {
          "first_name": { "type": "text" },
          "last_name": { "type": "text" },
          "email": { "type": "keyword" }
        }
      }
    }
  }
}

# Now we index the first Document
PUT /reviews/_doc/1
{
  "rating": 5.0,
  "content": "Outstanding course! Bo really taught me a lot about Elasticsearch!",
  "product_id": 123,
  "author": {
    "first_name": "John",
    "last_name": "Doe",
    "email": "johndoe123@example.com"
  }
}

# Objects can be defined flattened
# by using object_name.field_name keys 
PUT /reviews_dot_notation
{
  "mappings": {
    "properties": {
      "rating": { "type": "float" },
      "content": { "type": "text" },
      "product_id": { "type": "integer" },
      "author.first_name": { "type": "text" },
      "author.last_name": { "type": "text" },
      "author.email": { "type": "keyword" }
    }
  }
}

### --- Retrieving mappings

# Retrieving mappings for the `reviews` index
GET /reviews/_mapping

# Retrieving mapping for the `content` field
GET /reviews/_mapping/field/content

# Retrieving mapping for the `author.email` field
# using dot-notation
GET /reviews/_mapping/field/author.email

### --- Extending mappings to existing indices: adding new fields

# Here, we have an Index reviews
# and we add a new field to it: 
# "created_at": { "type": "date" }
# However, it is usually not possible to change/modify 
# existing mappings or their fields. 
# The alternative is to create new mappings 
# and `_reindex` the old index to the new one.
PUT /reviews/_mapping
{
  "properties": {
    "created_at": {
      "type": "date"
    }
  }
}

### --- Dates

# Supplying only a date: "yyyy-mm-dd"
# It will be converted and stored as milliseconds since epoch
PUT /reviews/_doc/2
{
  "rating": 4.5,
  "content": "Not bad. Not bad at all!",
  "product_id": 123,
  "created_at": "2015-03-27",
  "author": {
    "first_name": "Average",
    "last_name": "Joe",
    "email": "avgjoe@example.com"
  }
}

# Supplying both a date and time: "yyyy-mm-ddThh:mm:ss"
# It will be converted and stored as milliseconds since epoch
# ISO 8601 must be used: 
# - time separated with T
# - Z for UTC time zone, no offset (Greenwich), or offset as "+hh:mm"
PUT /reviews/_doc/3
{
  "rating": 3.5,
  "content": "Could be better",
  "product_id": 123,
  "created_at": "2015-04-15T13:07:41Z",
  "author": {
    "first_name": "Spencer",
    "last_name": "Pearson",
    "email": "spearson@example.com"
  }
}

# Specifying the UTC offset
# "yyyy-mm-ddThh:mm:ss++hh:mm"
PUT /reviews/_doc/4
{
  "rating": 5.0,
  "content": "Incredible!",
  "product_id": 123,
  "created_at": "2015-01-28T09:21:51+01:00",
  "author": {
    "first_name": "Adam",
    "last_name": "Jones",
    "email": "adam.jones@example.com"
  }
}

# Supplying a timestamp (milliseconds since the epoch)
# Equivalent to 2015-07-04T12:01:24Z
PUT /reviews/_doc/5
{
  "rating": 4.5,
  "content": "Very useful",
  "product_id": 123,
  "created_at": 1436011284000,
  "author": {
    "first_name": "Taylor",
    "last_name": "West",
    "email": "twest@example.com"
  }
}

### --- Reindexing: creating new indices because we want to change a field

# First, we get the mapping of an index
# We copy the output to paste it in the
# next command, which creates a new index
GET /reviews/_mappings

# This is the new index
# We paste the mapping obtained before
# and change the field(s) we want:
# "product_id": {"type": "integer"} -> {"type": "keyword"}
# PUT /reviews_new
# {
#   ... paste the mappings content here
# }
PUT /reviews_new
{
  "mappings" : {
    "properties" : {
      "author" : {
        "properties" : {
          "email" : {
            "type" : "keyword",
            "ignore_above" : 256
          },
          "first_name" : {
            "type" : "text"
          },
          "last_name" : {
            "type" : "text"
          }
        }
      },
      "content" : {
        "type" : "text"
      },
      "created_at" : {
        "type" : "date"
      },
      "product_id" : {
        "type" : "keyword"
      },
      "rating" : {
        "type" : "float"
      }
    }
  }
}

# After the new index is created
# we add documents to it by reindexing
# from the old index.
# Re-indexing is much cheaper than indexing
# the documents anew!
# We specify in the source and dest
# the old and the new indices, respectively
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  }
}
# However, this does not change the _source objects
# That is not a problem, but if we want
# to be consistent, we can add a script
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
      if (ctx._source.product_id != null) {
        ctx._source.product_id = ctx._source.product_id.toString();
      }
    """
  }
}

# Check that everything run ok
GET /reviews_new/_search
{
  "query": {
    "match_all": {}
  }
}

# We can also reindex only a subset
# of the documents.
# To that end, we need to change the match_all
# query by a more specific one,
# e.g., in this example, only
# reviews greater than 4.0
# are reindexed
POST /_reindex
{
  "source": {
    "index": "reviews",
    "query": {
      "range": {
        "rating": {
          "gte": 4.0
        }
      }
    }
  },
  "dest": {
    "index": "reviews_new"
  }
}

# To remove fields when reindexing
# we apply source-filtering,
# i.e., we specify the fields from _source
# that we want to copy -- the rest is ignored!
POST /_reindex
{
  "source": {
    "index": "reviews",
    "_source": ["content", "created_at", "rating"]
  },
  "dest": {
    "index": "reviews_new"
  }
}

# Renaming field names during reindexing
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
      # Rename "content" field to "comment"
      ctx._source.comment = ctx._source.remove("content");
    """
  }
}

# Ignore reviews with ratings below 4.0
# However, it is better in general to use a query
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  },
  "script": {
    "source": """
      if (ctx._source.rating < 4.0) {
        ctx.op = "noop"; # Can also be set to "delete"
      }
    """
  }
}
```