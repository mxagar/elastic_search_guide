# Elastic Search: Command Catalogue

This document compiles the most important commands of Elastic Search.

For a more detailed guide on Elastic Search, check [`README.md`](./README.md).

Table of contents:

- [Elastic Search: Command Catalogue](#elastic-search-command-catalogue)
  - [Basic](#basic)
  - [Manage Documents](#manage-documents)
  - [Analyzers](#analyzers)
  - [Mappings](#mappings)
  - [Search](#search)
  - [Joining Queries](#joining-queries)
  - [Controlling Query Results](#controlling-query-results)
  - [Aggregations](#aggregations)
  - [Improving Search Results](#improving-search-results)


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

```json
// --- Create and delete indices

// Create an index
PUT /pages

// Delete an index
DELETE /pages

// Create an index with specific properties
PUT /products
{
  "settings": {
    "number_of_shards": 2
  }
}

// --- Create/Index, delete and get documents

// Simple JSON Document indexed = created
// An _id is automatically assigned if not provided
// As we can see in the returned JSON
// Also, we see we have 2 shards
POST /products/_doc
{
  "name": "Coffee Maker",
  "price": 64,
  "in_stock": 10
}

// Here, we index a Document = we create one
// but we force it to be of id 100
// Note that the HTTP method is PUT, not POST!
// We can use the same code
// to replace entirely a Document
PUT /products/_doc/100
{
  "name": "Toaster",
  "price": 49,
  "in_stock": 4
}

// Delete the document with ID 100
DELETE /products/_doc/100

// Retrieve Document by ID
GET /products/_doc/100

// Get all Documents in the index products
// The result is a JSON in which result['hits']['hits']
// contains a list of al Document JSONs
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}

// Delete all the documents of an index
POST /products/_delete_by_query
{
  "query": {
    "match_all": {}
  }
}

// If the index was not created and we index
// a Document, the index will be created automatically
// and the Document added to it
PUT /products_test/_doc/1
{
  "price": 7.4
}

// --- Update documents: modify fields, add fields, scripts, upsert (insert or create)

// Update an existing field
// To update a Document, 
// we need to pass a JSON with an object "doc"
// The returned JSON contains "result": "updated"
POST /products/_update/100
{
  "doc": {
    "in_stock": 3
  }
}

// Add a new field: "tags": ["electronics"]
POST /products/_update/100
{
  "doc": {
    "tags": ["electronics"]
  }
}

// Go to Document with ID 100
// Decrease by one unit the field in_stock
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock--"
  }
}

// Go to Document with ID 100
// Assign the value 10 to the field in_stock
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock = 10"
  }
}

// Go to Document with ID 100
// Modify field in_stock with the values in params
POST /products/_update/100
{
  "script": {
    "source": "ctx._source.in_stock -= params.quantity",
    "params": {
      "quantity": 4
    }
  }
}

// Go to Document with ID 100
// If in_stock == 0, perform no operation
// Else, decrease in_stock in one unit
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

// Go to Document with ID 100
// If in_stock < 0, delete the Document (product)
// Else, decrease in_stock in one unit
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

// Upsert: Update or Insert/Create
// If the product 101 doesn't exist, this creates it
// Else, it increases in_stock in one unit
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

// --- Optimistic Concurrency Control: dealing with parallel updates

// Get Document with ID 100
// _primary_term and seq_no are in the metadata
// We take them to formulate the conditioned update request
GET /products/_doc/100

// Update Document/product with ID 100 
// if _primary_term == 1 and seq_no == 5 (reference values obtained in previous query)
// If _primary_term and seq_no don't match, we get an error
POST /products/_update/100?if_primary_term=1&if_seq_no=5
{
  "doc": {
    "in_stock": 123
  }
}

// --- Update and Delete by Query: Update/Delete a filtered set of documents

// Update a set of filtered Documents: _update_by_query
// Write the update in "script"
// Write the filterin "query"
// Errors/conflicts can occur; by default the request is aborted
// but we can specify to proceed if we want
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

// Delete all Documents that match the query
// in this case ALL DOCUMENTS!
// If errors/conflicts occur, the request is aborted
// unless we specify "conflicts": "proceed"
POST /products/_delete_by_query
{
  "conflicts": "proceed",
  "query": {
    "match_all": { }
  }
}

// --- Batch/Bulk processing

// Create new Documents
// We can list many (action, source) pairs
// Always newline after a line, also in last JSON
// Four actions: create, index, update, delete
// Each action needs a source (the Document fields), except delete
POST /_bulk
{ "index": { "_index": "products", "_id": 200 } }
{ "name": "Espresso Machine", "price": 199, "in_stock": 5 }
{ "create": { "_index": "products", "_id": 201 } }
{ "name": "Milk Frother", "price": 149, "in_stock": 14 }

// Update Documents + Delete
POST /_bulk
{ "update": { "_index": "products", "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_index": "products", "_id": 200 } }

// If all actions are for the same index,
// we can specify it in the API commad
POST /products/_bulk
{ "update": { "_id": 201 } }
{ "doc": { "price": 129 } }
{ "delete": { "_id": 200 } }

// Get all Documents in an index
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}

```

## Analyzers

```json
// Here a text string is analyzed
// with the standard analyzer:
// no char filter, standard tokenizer, lowercase token filter
POST /_analyze
{
  "text": "2 guys walk into   a bar, but the third... DUCKS! :-)",
  "analyzer": "standard"
}

// Analyzer components explicitly defined:
// character filer, tokenizer, token filters
// This call produces the same results are before
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
- Field Aliases
- Index Templates
- Configuring Dynamic Mappings
- Dynamic templates
- Custom analyzers
- Adding/Updating Analyzers to/from Existing Indices

```json
// --- Adding explicit mappings

// We create a mapping for the index reviews
// For simple types, we define their type key-value
// For object types, we need to nest a properties key again
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

// Now we index the first Document
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

// Objects can be defined flattened
// by using object_name.field_name keys 
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

// --- Retrieving mappings

// Retrieving mappings for the `reviews` index
GET /reviews/_mapping

// Retrieving mapping for the `content` field
GET /reviews/_mapping/field/content

// Retrieving mapping for the `author.email` field
// using dot-notation
GET /reviews/_mapping/field/author.email

// --- Extending mappings to existing indices: adding new fields

// Here, we have an Index reviews
// and we add a new field to it: 
// "created_at": { "type": "date" }
// However, it is usually not possible to change/modify 
// existing mappings or their fields. 
// The alternative is to create new mappings 
// and `_reindex` the old index to the new one.
PUT /reviews/_mapping
{
  "properties": {
    "created_at": {
      "type": "date"
    }
  }
}

// --- Dates

// Supplying only a date: "yyyy-mm-dd"
// It will be converted and stored as milliseconds since epoch
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

// Supplying both a date and time: "yyyy-mm-ddThh:mm:ss"
// It will be converted and stored as milliseconds since epoch
// ISO 8601 must be used: 
// - time separated with T
// - Z for UTC time zone, no offset (Greenwich), or offset as "+hh:mm"
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

// Specifying the UTC offset
// "yyyy-mm-ddThh:mm:ss++hh:mm"
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

// Supplying a timestamp (milliseconds since the epoch)
// Equivalent to 2015-07-04T12:01:24Z
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

// --- Reindexing: creating new indices because we want to change a field

// First, we get the mapping of an index
// We copy the output to paste it in the
// next command, which creates a new index
GET /reviews/_mappings

// This is the new index
// We paste the mapping obtained before
// and change the field(s) we want:
// "product_id": {"type": "integer"} -> {"type": "keyword"}
// PUT /reviews_new
// {
//   ... paste the mappings content here
// }
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

// After the new index is created
// we add documents to it by reindexing
// from the old index.
// Re-indexing is much cheaper than indexing
// the documents anew!
// We specify in the source and dest
// the old and the new indices, respectively
POST /_reindex
{
  "source": {
    "index": "reviews"
  },
  "dest": {
    "index": "reviews_new"
  }
}
// However, this does not change the _source objects
// That is not a problem, but if we want
// to be consistent, we can add a script
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

// Check that everything run ok
GET /reviews_new/_search
{
  "query": {
    "match_all": {}
  }
}

// We can also reindex only a subset
// of the documents.
// To that end, we need to change the match_all
// query by a more specific one,
// e.g., in this example, only
// reviews greater than 4.0
// are reindexed
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

// To remove fields when reindexing
// we apply source-filtering,
// i.e., we specify the fields from _source
// that we want to copy -- the rest is ignored!
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

// Renaming field names during reindexing
// However, reindexing to rename a field
// is a bed idea -- instead, use field aliases!
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
      // Rename "content" field to "comment"
      ctx._source.comment = ctx._source.remove("content");
    """
  }
}

// Ignore reviews with ratings below 4.0
// However, it is better in general to use a query
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
        ctx.op = "noop"; // Can also be set to "delete"
      }
    """
  }
}

// --- Field Aliases

// Reindexing to rename a field is a bad idea;
// instead, we can use field aliases.
// Here, we
// add `comment` alias pointing to the `content` field,
// so content is an alias of comment.
// Alias fields can be used as regular fields
// and the original fields are unaffected
PUT /reviews/_mapping
{
  "properties": {
    "comment": {
      "type": "alias",
      "path": "content"
    }
  }
}

// --- Multi-field mappings

// The most common multi-field mapping
// is the one where a field is both text and keyword.
// To that end:
// We add `keyword` mapping to a `text` field
// This effectively creates an additional index:
// ingredients.keyword
// In contrast to the ingredients field,
// which allows non-exact term search
// the new (sub-)field allows exact search
PUT /multi_field_test
{
  "mappings": {
    "properties": {
      "description": {
        "type": "text"
      },
      "ingredients": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      }
    }
  }
}

// Querying the `text` mapping
// (non-exact match/search)
// (assuming multi_field_test has been filled)
GET /multi_field_test/_search
{
  "query": {
    "match": {
      "ingredients": "Spaghetti"
    }
  }
}

// Querying the `keyword` mapping (exact match)
// (assuming multi_field_test has been filled)
// A new index `ingredients.keyword` has been created
// apart from `ingredients`, and here we search
// for exactly matching keywords in `ingredients.keyword`
// Thus, multi-fields allow different search types
// but bear in mind that in reality multiple indices
// are created under the hood
GET /multi_field_test/_search
{
  "query": {
    "term": {
      "ingredients.keyword": "Spaghetti"
    }
  }
}

// --- Index Templates

// We can create a reusable index template
// by specifying its settings and mappings
// Example index template: access-logs-*
PUT /_index_template/access-logs
{
  "index_patterns": ["access-logs-*"],
  "template": {
    "settings": {
      "number_of_shards": 2,
      "index.mapping.coerce": false
    },
    "mappings": {
      "properties": {
        "@timestamp": { "type": "date" },
        "url.original": { "type": "wildcard" },
        "url.path": { "type": "wildcard" },
        "url.scheme": { "type": "keyword" },
        "url.domain": { "type": "keyword" },
        "client.geo.continent_name": { "type": "keyword" },
        "client.geo.country_name": { "type": "keyword" },
        "client.geo.region_name": { "type": "keyword" },
        "client.geo.city_name": { "type": "keyword" },
        "user_agent.original": { "type": "keyword" },
        "user_agent.name": { "type": "keyword" },
        "user_agent.version": { "type": "keyword" },
        "user_agent.device.name": { "type": "keyword" },
        "user_agent.os.name": { "type": "keyword" },
        "user_agent.os.version": { "type": "keyword" }
      }
    }
  }
}

// Then, we add a doc to a non-existing index
// but whose template is already created (above)
// - Index access-logs-2023-01 will be created automatically
// - doc will be indexed
POST /access-logs-2023-01/_doc
{
  "@timestamp": "2023-01-01T00:00:00Z",
  "url.original": "https://example.com/products",
  "url.path": "/products",
  "url.scheme": "https",
  "url.domain": "example.com",
  "client.geo.continent_name": "Europe",
  "client.geo.country_name": "Denmark",
  "client.geo.region_name": "Capital City Region",
  "client.geo.city_name": "Copenhagen",
  "user_agent.original": "Mozilla/5.0 (iPhone; CPU iPhone OS 12_1 like Mac OS X) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0 Mobile/15E148 Safari/604.1",
  "user_agent.name": "Safari",
  "user_agent.version": "12.0",
  "user_agent.device.name": "iPhone",
  "user_agent.os.name": "iOS",
  "user_agent.os.version": "12.1.0"
}

// We can also manually create an index when a template exists
// and can even extend the definition of the template,
// e.g., by adding a new field - here, url.query (keyword) is added
PUT /access-logs-2023-02
{
  "settings": {
    "number_of_shards": 1
  },
  "mappings": {
    "properties": {
      "url.query": {
        "type": "keyword"
      }
    }
  }
}

// Get/retrieve an index
GET /access-logs-2023-01
GET /access-logs-2023-02

// Retrieving an index template
GET /_index_template/access-logs

// Deleting an index template
DELETE /_index_template/access-logs

// --- Configuring Dynamic Mappings

// Disable dynamic mapping ("dynamic": false)
// New fields are ignored, if we try to insert them
// but still continue being part of _source
// Remember _source is not part of the search
// data structures.
PUT /people
{
  "mappings": {
    "dynamic": false,
    "properties": {
      "first_name": {
        "type": "text"
      }
    }
  }
}

// Disable strictly dynamic mapping ("dynamic": "strict"),
// i.e., if we try to insert new fields an error occurs.
PUT /people
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "first_name": {
        "type": "text"
      }
    }
  }
}

// The dynamic setting is inherited to all the fields in
// the mapping, but we can overwrite it in any field.
// Here other is an object an we set "dynamic": true,
// so we can extend it, even though we cannot extend 
// the index outside from other, because it's set to "dynamic": "strict"!
PUT /computers
{
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "name": {
        "type": "text"
      },
      "specifications": {
        "properties": {
          "cpu": {
            "properties": {
              "name": {
                "type": "text"
              }
            }
          },
          "other": {
            "dynamic": true,
            "properties": {  }
          }
        }
      }
    }
  }
}

// We can set "numeric_detection": true
// So that new fields will be forced to be
// numbers if possible, 
// even when they are input as text
PUT /computers
{
  "mappings": {
    "numeric_detection": true
  }
}
// New string fields are indexed
// as numbers, because it's possible
// and "numeric_detection": true
POST /computers/_doc
{
  "specifications": {
    "other": {
      "max_ram_gb": "32", // long
      "bluetooth": "5.2" // float
    }
  }
}

// --- Dynamic Templates

// dynamic_templates can be used to define the parsing
// of concrete values to a given type + parameters, among others
// Example: Map whole numbers to `integer` instead of `long`
PUT /dynamic_template_test
{
  "mappings": {
    "dynamic_templates": [
      {
        "integers": {
          "match_mapping_type": "long", // every JSON field with a whole number...
          "mapping": {
            "type": "integer" // ... will be parsed as integer
          }
        }
      }
    ]
  }
}

// One common use case would be to modify
// the way strings are mapped by default; 
// instead of creating for a string a `text` and `keyword` field, 
// we might want to create just a `text` field,
// or limit the length of the keyword with `ignore_above`.
// In the example, we modify default mapping for strings:
// We set `ignore_above` to 512,
// so all strings will get a text field and a keyword field
// but the maximum length of the keyword will be 512
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings": {
          "match_mapping_type": "string",
          "mapping": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword",
                "ignore_above": 512
              }
            }
          }
        }
      }
    ]
  }
}

// For each dynamic template, 
// we have conditions that can be specified 
// with **`match` and/or `unmatch`** parameters:
// - `"match": "text_*"`: all fields with a name that matches `text_*`;
// we can apply regex here!
// - `"unmatch": "*_keyword"`: except all fields with a name that matches `*_keyword`;
// we can apply regex here!
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "strings_only_text": {
          "match_mapping_type": "string",
          "match": "text_*",
          "unmatch": "*_keyword",
          "mapping": {
            "type": "text"
          }
        }
      },
      {
        "strings_only_keyword": {
          "match_mapping_type": "string",
          "match": "*_keyword",
          "mapping": {
            "type": "keyword"
          }
        }
      }
    ]
  }
}

POST /test_index/_doc
{
  "text_product_description": "A description.", // first template is matched -> type: text
  "text_product_id_keyword": "ABC-123" // second template is matched -> type: keyword
}

// In the context of dynamic mapping definitions,
// we also have the **`path_match` and `path_unmatch`** parameters,
// which refer to the dotted field name, i.e., `field_name.subfield_name`.
PUT /test_index
{
  "mappings": {
    "dynamic_templates": [
      {
        "copy_to_full_name": {
          "match_mapping_type": "string",
          "path_match": "employer.name.*",
          "mapping": {
            "type": "text",
            "copy_to": "full_name"
          }
        }
      }
    ]
  }
}

POST /test_index/_doc
{
  "employer": {
    "name": {
      "first_name": "John",
      "middle_name": "Edward",
      "last_name": "Doe"
    }
  }
}

// --- Custom analyzers

// Analyze the text with the standard analyzer
// HTML characters are tokenized one by one...
// We need to create our own analyzer...
POST /_analyze
{
  "analyzer": "standard",
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}

// Creation of a custom analyzer
// able to process a text with HTML tags
// and handle special characters
// Note that we create it within an index: analyzer_test
// In other words, it is created when creating the index.
// It can be added later too, though, as shown below.
PUT /analyzer_test
{
  "settings": {
    "analysis": {
      "analyzer": {
        "my_custom_analyzer": {
          "type": "custom",
          "char_filter": ["html_strip"],
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop",           // remove stop words
            "asciifolding"    // convert special symbols to ASCII equivalent
          ]
        }
      }
    }
  }
}

// Analyze the text with the custom analyzer
// Note that we need to run it within the index
// where it was defined
POST /analyzer_test/_analyze
{
  "analyzer": "standard",
  "text": "I&apos;m in a <em>good</em> mood&nbsp;-&nbsp;and I <strong>love</strong> açaí!"
}

// --- Adding/Updating Analyzers to/from Existing Indices

// First, close the index
// This is necessary because
// the analyzer setting is static
// not dynamic, i.e., we need to
// stop any operations in the index
// before we change it
POST /analyzer_test/_close

// Then, create/add a custom analyzer
// The syntax is the same as when
// we create and configure an index
// with a custom analyzer
// Updating and existing one is done also with
// the same call, but we additionally need to
// call the _update_by_query API, shown below
PUT /analyzer_test/_settings
{
  "analysis": {
    "analyzer": {
      "my_second_analyzer": {
        "type": "custom",
        "tokenizer": "standard",
        "char_filter": ["html_strip"],
        "filter": [
          "lowercase",
          "stop",
          "asciifolding"
        ]
      }
    }
  }
}

// Re-Open the index again
// If an index is closed, no queries are accepted
// no indexing of Documents can happen
POST /analyzer_test/_open

// If we have updated the analyzer,
// and not created a new one,
// we need to update the Documents
// to be processed by the new analyzer,
// otherwise the indexed values are inconsistent.
// This call re-indexes all Documents again.
POST /analyzer_test/_update_by_query?conflicts=proceed
```

## Search

Contents:

- Basic search: search all
- Term-level search
- Search documents by ID
- Range searches
- Searching with Prefixes, Wildcards, Regex
- Querying by Field Existence
- Match Query: Full-Text Query
- Searching Multiple Fields
- Phrase Searches
- Bool Compound Queries
- Boosting Queries
- Disjoint Max
- Querying Nested Fields

```json
// --- Basic search: search all

// Search all documents in an index (products)
GET /products/_search
{
  "query": {
    "match_all": {}
  }
}


// --- Term-level search

// Term-Level queries can be used
// to perform exact searches.
// They should be used with *keywords*, *numbers*, *booleans*, *dates*
// but never for *text* fields, since no analyzer is used,
// which transforms the queried word.
// Example: Term query in which we find for all product Documents
// which contain the tag "Vegetable"; the tag "vegetable"
// won't be found.
// Exception: when we query using prefixes, wildcards and regex in the term query.
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": "Vegetable"
    }
  }
}

// Same query as before,
// but with explicit syntax
// The advantage is that we can add
// more parameters, like case_insensitive
GET /products/_search
{
  "query": {
    "term": {
      "tags.keyword": {
        "value": "Vegetable",
        "case_insensitive": true
      }
    }
  }
}

// Term-search query for multiple values
// Note the field is plural: terms
GET /products/_search
{
  "query": {
    "terms": {
      "tags.keyword": ["Soup", "Meat"]
    }
  }
}

// Term-search for 
// numbers, booleans, dates, timestamps
GET /products/_search
{
  "query": {
    "term": {
      //"in_stock": 1
      //"created": "2007/10/14"
      //"created": "2007/10/14 12:34:56"
      "is_active": true
    }
  }
}

// --- Search documents by ID

// Retrieve documents by IDs
// Not all IDs we specify are required to match!
// Simply, the ones that are matched are returned
GET /products/_search
{
  "query": {
    "ids": {
      "values": ["100", "200", "300"]
    }
  }
}

// --- Range searched

// SELECT * FROM products WHERE in_stock >= 1 AND in_stock <= 5
// Also, we can use (gt, lt) = (>, <>) 
// instead of (gte, lte) = (>=, <=)
GET /products/_search
{
  "query": {
    "range": {
      "in_stock": {
        "gte": 1,
        "lte": 5
      }
    }
  }
}

// Dates can take days, day+time, etc.
//   "gte": "2020/01/01 00:00:00",
//   "lte": "2020/01/31 23:59:59"
// By default, dates should be specified as in the mapping
// otherwise, if we change the format, we can specify it in the query:
//   "format": "dd/MM/yyyy",
//   "gte": "01/01/2020", ...
// It's even possible to specify a timezone parameter:
//   "time_zone": "+01:00",
//   "gte": "2020/01/01 01:00:00", ...
// If no time zone provided, it's assumed the date is in UTC
GET /products/_search
{
  "query": {
    "range": {
      "created": {
        "gte": "2020/01/01",
        "lte": "2020/01/31"
      }
    }
  }
}

// --- Searching with Prefixes, Wildcards, Regex

// Prefixes, Wildcards and Regex
// can be used in ter-level queries performed on keywords.
// "prefix"
// "wildcard" (use them as suffix):
//    ?: any single character
//    *: any set of characters once or several times
// "regexp"
//    "Bee(f|r)+": f or r one or more times
//    "Bee(f|r){1}": f or r once
//    "Bee[a-zA-Z]+": any continuation, i.e. "Bee" is a prefix
// We can also add "case_insensitive": true
GET /products/_search
{
  "query": {
    "prefix": {
      "name.keyword": {
        // This matches only a keyword which starts with "Past"
        "value": "Past"
      }
    }
  }
}

GET /products/_search
{
  "query": {
    "wildcard": {
      "name.keyword": {
        "value": "Bee*"
      }
    }
  }
}

GET /products/_search
{
  "query": {
    "regexp": {
      "tags.keyword": {
        "value": "Bee(f|r)+"
      }
    }
  }
}

// --- Querying by Field Existence

// SELECT * FROM products WHERE tags IS NOT NULL
// Notes: 
// - empty arrays [] are like NULL, so NOT indexed -> non existent
// - empty strings "" are NOT NULL -> they're indexed!
// - if the field contains `null_value`, it is indexed!
// - if we set index = False, the field is not indexed! (e.g., time series)
GET /products/_search
{
  "query": {
    "exists": {
      "field": "tags.keyword"
    }
  }
}

// SELECT * FROM products WHERE tags IS NULL
GET /products/_search
{
  "query": {
    "bool": {
      "must_not": [
        {
          "exists": {
            "field": "tags.keyword"
          }
        }
      ]
    }
  }
}

// --- Match Query: Full-Text Query

// For term-level searches we use 
// the `query` object `term`.
// For full-text searches we use 
// the `query` object `match`
// This returns many products (12)
// which have a name containing "pasta"
// or a derivate.
GET /products/_search
{
  "query": {
    "match": {
      "name": "pasta"
      //"name": "PASTA" 
      // This would result in the same, 
      // because of lowercasing in the analysis...
    }
  }
}

// We can search for documents
// with multiple words simply
// by specifying them in a string
// Recall that the string is analyzed: tokenized, etc.
GET /products/_search
{
  "query": {
    "match": {
      "name": "pasta chicken"
    }
  }
}
// If we have several search tokens
// the default operator is OR,
// i.e., not all tokens need to appear
// We can change that with "operator": "and"
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "pasta chicken",
        // require all tokens to appear in results
        "operator": "and"
      }
    }
  }
}


// --- Searching Multiple Fields

// We can use `multi_match` 
// to search in more than one field.
// However, by default the score of the
// highest matching field is used for the document;
// exception: using a tie_breaker (see below)
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable",
      "fields": ["name", "tags"]
    }
  }
}

// We can modify the relevance scores
// by boosting the relevance score per field
// field_a^2: relevance scores of matches in field_a
// and doubled (x2). Again, by default the 
// highest matching field is used for the document;
// exception: using a tie_breaker (see below)
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable",
      "fields": ["name^2", "tags"]
    }
  }
}

// Here a multi_match is performed in 2 fields
// and a tie_breaker is added; as a consequence,
// the score of the document is a sum of
// - the score of the maximum matching field
// - and the scores of the other matching fields multiplied by tie_breaker
GET /products/_search
{
  "query": {
    "multi_match": {
      "query": "vegetable broth",
      "fields": ["name", "description"],
      "tie_breaker": 0.3
    }
  }
}


// --- Phrase Searches

// The `match` query searches for 
// any of the tokens in any order in the fields.
// Meanwhile, with `match_phrase` we 
// require for all tokens to appear
// in the correct order and without other
// tokens in between.
GET /products/_search
{
  "query": {
    "match_phrase": {
      "name": "mango juice" // it does not appear, but "juice mango" does
    }
  }
}


// --- Bool Compound Queries

// Compound queries perform several leaf queries simultaneously.
// The most common compound query is `bool`
// which can accept several clauses:
// - must: required to contain
// - must_not: required not to contain
// - should: not required to contain, but relevance is improved
// - filter: required to contain, relevance unaffected

// Example: must clause: the presence is required
// SELECT * FROM products  WHERE tags IN ("Alcohol")
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ]
    }
  }
}

// Example: must_not clause: the non-presence is required
// SELECT * FROM products WHERE tags IN ("Alcohol") AND tags NOT IN ("Wine")
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "tags.keyword": "Wine"
          }
        }
      ]
    }
  }
}

// Example: should clause: not required to appear, but relevance is increased if it does
// If we compose our compound query only with should clauses
// at least one clause must match to get results.
// If should clauses appear among must / filter clauses
// should matches act as relevance boosters, unles we add
// the parameter `minimum_should_match` to the should clause,
// with which the should query becomes compulsory.
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "tags.keyword": "Wine"
          }
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        }
      ]
    }
  }
}

// Example: Another bool compound query with more sub-queries
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ],
      "must_not": [
        {
          "term": {
            "tags.keyword": "Wine"
          }
        }
      ],
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        },
        {
          "match": {
            "name": "beer"
          }
        },
        {
          "match": {
            "description": "beer"
          }
        }
      ]
    }
  }
}

// Example: should clause with minimum_should_match,
// which makes the query compulsory
GET /products/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ], 
      "should": [
        {
          "term": {
            "tags.keyword": "Beer"
          }
        },
        {
          "match": {
            "name": "beer"
          }
        }
      ],
      "minimum_should_match": 1
    }
  }
}

// Example: filter clause: similar to must,
// but they don't affect relevance scores.
// Additionally, the results are cached,
// so they increase speed/performance.
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "term": {
            "tags.keyword": "Alcohol"
          }
        }
      ]
    }
  }
}

// Example
// SELECT * FROM products WHERE tags IN ("Beer") AND (name LIKE '%Beer%' OR description LIKE '%Beer%') AND in_stock <= 100
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "in_stock": {
              "lte": 100
            }
          }
        },
        {
          "term": {
            "tags.keyword": "Beer"
          }
        }
      ],
      "should": [
        { "match": { "name": "Beer" } },
        { "match": { "description": "Beer" } }
      ],
      "minimum_should_match": 1
    }
  }
}
// Alternative, equivalent
GET /products/_search
{
  "query": {
    "bool": {
      "filter": [
        {
          "range": {
            "in_stock": {
              "lte": 100
            }
          }
        },
        {
          "term": {
            "tags.keyword": "Beer"
          }
        }
      ],
      "must": [
        {
          "multi_match": {
            "query": "Beer",
            "fields": ["name", "description"]
          }
        }
      ]
    }
  }
}

// --- Boosting Queries

// The `boosting` query
// allows to weight different sub-query matches differently.
// Example: Get products which are a juice
GET /products/_search
{
  "size": 20, // increase page size from 10 (default) to 20
  "query": {
    "match": {
      "name": "juice"
    }
  }
}
// Next version of the query but with boosting:
// - we prioritize juices
// - we penalize apple products
GET /products/_search
{
  "size": 20,
  "query": {
    "boosting": {
      "positive": {
        "match": {
          "name": "juice"
        }
      },
      "negative": {
        "match": {
          "name": "apple"
        }
      },
      "negative_boost": 0.5 // [0.0,1.0]
    }
  }
}

// Example: Pasta products, preferably without bacon
GET /recipes/_search
{
  "query": {
    "boosting": {
      "positive": {
        "term": {
          "ingredients.name.keyword": "Pasta"
        }
      },
      "negative": {
        "term": {
          "ingredients.name.keyword": "Bacon"
        }
      },
      "negative_boost": 0.5
    }
  }
}

// Example: anything without bacon
// We always need to have a positive boosting query
// so we add match_all by default
GET /recipes/_search
{
  "query": {
    "boosting": {
      "positive": {
        "match_all": {}
      },
      "negative": {
        "term": {
          "ingredients.name.keyword": "Bacon"
        }
      },
      "negative_boost": 0.5
    }
  }
}

// Example: Show me all products, but
// - prioritize pasta products
// - penalize bacon products
GET /recipes/_search
{
  "query": {
    "boosting": {
      "positive": {
        "bool": {
          "must": [
            { "match_all": {} }
          ],
          "should": [
            {
              "term": {
                "ingredients.name.keyword": "Pasta"
              }
            }
          ]
        }
      },
      "negative": {
        "term": {
          "ingredients.name.keyword": "Bacon"
        }
      },
      "negative_boost": 0.5
    }
  }
}

// --- Disjoint Max

// Disjoint max is a compound query:
// - We can add several queries within it
// - For a document to be a match, it's enough if one query is a match
// - If several queries give a match, the one with the highest relevance is used to compute the document relevance
// The `multi_match` query is broken down to a `dis_max` query internally.
GET /products/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "name": "vegetable" } },
        { "match": { "tags": "vegetable" } }
      ]
    }
  }
}
// Here, we add tie_breaker != 0.0, with which
// we take the lesser relevant scores weighted by it
GET /products/_search
{
  "query": {
    "dis_max": {
      "queries": [
        { "match": { "name": "vegetable" } },
        { "match": { "tags": "vegetable" } }
      ],
      "tie_breaker": 0.3
    }
  }
}

// --- Querying Nested Fields

// Array fields are not nested fields,
// so we cannot filter documents considering each
// element of the array independenty as an object.
// In this example, ingredients is an array, so all values within it
// are handled together; thus, if a filtering query
// is run on a field of ingredients,
// we won't get the instances within the array which satisfy it, 
// but the *complete array*, 
// if the array has at least one instance which satisfies the condition.
// WARNING: This query does not work with nested objects/types
// but ingredients is not nested yet...
GET /recipes/_search
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "ingredients.name": "parmesan"
          }
        },
        {
          "range": {
            "ingredients.amount": {
              "gte": 100
            }
          }
        }
      ]
    }
  }
}

// The real question we want to ask is:
// Which recipies contain at least 100 units of Parmesan cheese?
// To achieve that, we need to
// - map the ingredients field as nested
// - reindex everything correctly again
// Therefore, we:
// 1. delete the index
// 2. define the mapping
// 3. bulk-process the JSON again

// 1. Delete recipes index
DELETE /recipes
// 2. Define new mapping with ingredients as type nested
// Each nested object will be stored internally 
// as a separete Lucene document.
// Additionally, we have a root document that points
// to the nested documents.
// And usually, we are interested in the root document
// but we apply queries to the nested ones
// if we want to filter using nested fields.
// IMPORTANT NOTE: while nested fields allow for
// filtering according to nested objects,
// they have the important limitation that they can contain
// only 10k instances. Thus, it is often better
// to create separate indices instead of nested fields,
// so that we can easily scale.
PUT /recipes
{
  "mappings": {
    "properties": {
      "title": { "type": "text" },
      "description": { "type": "text" },
      "preparation_time_minutes": { "type": "integer" },
      "steps": { "type": "text" },
      "created": { "type": "date" },
      "ratings": { "type": "float" },
      "servings": {
        "properties": {
          "min": { "type": "integer" },
          "max": { "type": "integer" }
        }
      },
      "ingredients": {
        "type": "nested",
        "properties": {
          "name": {
            "type": "text",
            "fields": {
              "keyword": {
                "type": "keyword"
              }
            }
          },
          "amount": { "type": "integer" },
          "unit": { "type": "keyword" }
        }
      }
    }
  }
}
// 3. Bulk-process JSON, done in the notebook
// NOTE: The bulk command is the same, only
// this time we don't let ELasticsearch infer the mapping
// but we use a manually & correctly defined one!

// Now, we can ask the correct question and the the correct answer:
// Which recipies contain at least 100 units of Parmesan cheese?
// The query for nested fields is `nest` and has the following syntax.
// Note that even we apply the query to the nested
// documents, only root documents are returned.
// We can modify how the scoring is transferred from child to root documents
// with the parameter `score_mode`,
// which can be: `avg` (default: average score of matching children), min, max, sum, none
GET /recipes/_search
{
  "query": {
    "nested": {
      "path": "ingredients", // we can use the dot notation if required
      "query": { // here, we can use the typical query as for not nested fields
        "bool": {
          "must": [
            {
              "match": {
                "ingredients.name": "parmesan"
              }
            },
            {
              "range": {
                "ingredients.amount": {
                  "gte": 100
                }
              }
            }
          ]
        }
      }
    }
  }
}

// In general, when we query 
// conditioning nested field/object values,
// the parent/root documents whose nested objects
// satisfy the conditions are returned.
// If we want to know more about 
// which nested objects/fields matched
// (not only the parent object),
// we can add the parameter `inner_hits`.
// Nested/children docs that match are displayed
// with their offset id according to relevance.
GET /recipes/_search
{
  "query": {
    "nested": {
      "path": "ingredients",
      "inner_hits": {}, // children/nested object that match will be shown in hits
      "query": {
        "bool": {
          "must": [
            {
              "match": {
                "ingredients.name": "parmesan"
              }
            },
            {
              "range": {
                "ingredients.amount": {
                  "gte": 100
                }
              }
            }
          ]
        }
      }
    }
  }
}

// We can further specify parameters to `inner_hits`:
// - name: the JSON field name for inner hits in the response
// - size: how many inner hits we want to get (default: 3)
GET /recipes/_search
{
  "query": {
    "nested": {
      "path": "ingredients",
      "inner_hits": {
        "name": "my_hits",
        "size": 10
      }, 
      "query": {
        "bool": {
          "must": [
            { "match": { "ingredients.name": "parmesan" } },
            { "range": { "ingredients.amount": { "gte": 100 } } }
          ]
        }
      }
    }
  }
}
```

## Joining Queries

Contents:

- Mapping Document Relationships
- Querying Related/Joined Documents
- Multi-level Relations
- Parent/Child Inner Hits
- Terms Lookup Mechanism

```json
// --- Mapping Document Relationships

// The department index is composed by
// - a department name
// - and a nested field of employees
PUT /department
{
  "mappings": {  
    "properties": {
      "name": {
        "type": "text",
        "fields": {
          "keyword": {
            "type": "keyword"
          }
        }
      },
      "employees": {
        "type": "nested"
      }
    }
  }
}

// To define document relationships, 
// we need to modify/add them to the mapping
// in the `join_field`
// There, we define parent-child relationships
// with key-value pairs.
// When adding documents, we can use that relationship
PUT /department/_mapping
{
  "properties": {
    "join_field": { 
      "type": "join",
      "relations": {
        // key-value pairs (parent-child); 
        // arbitraty strings
        // that don't need to match the index names!
        // We will refer to these strings (key-values)
        // when adding/indexing documents.
        // If we have several children, the value is an array
        "department": "employee"
      }
    }
  }
}

// New department document with id 1
// We assign the parent/key "department" to it
PUT /department/_doc/1
{
  "name": "Development",
  "join_field": "department"
}
// New department document with id 2
// We assign the parent/key "department" to it
PUT /department/_doc/2
{
  "name": "Marketing",
  "join_field": "department"
}
// New document with id 3
// This time it is an employee!
// We need to:
// - specify the id of the parent document (department): 1
// - add the same id in the routing, so that the employee and the department are in the same shard
// - assign the value/child entry to the join_field
// Note that dynamic mapping is applied,
// - no employees field is specified
// - age and gender are added
// Note: the id of the parent document is used to get the shard
PUT /department/_doc/3?routing=1
{
  "name": "John Doe",
  "age": 28,
  "gender": "M",
  "join_field": {
    "name": "employee",
    "parent": 1 // department doc id, same as in routing
  }
}

// --- Querying Related/Joined Documents

// Querying the children by parent ID
GET /department/_search
{
  "query": {
    "parent_id": {
      "type": "employee", // the type of relation for parent we'd like to get
      "id": 1 // the id of the parent document
    }
  }
}

// Querying the children by some query related to the parent
// It is possible to modify the relevance sorting with `score`
GET /department/_search
{
  "query": {
    "has_parent": {
      "parent_type": "department", // parent/key relation name
      "query": { // any query would work, e.g., a bool query
        "term": {
          "name.keyword": "Development"
        }
      }
    }
  }
}

// Qurying the parent(s) by some query related to the children
// It is possible to modify the relevance sorting with `score_mode`
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee", // relation type of the child: employee
      "query": { // any query related to the children fields
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}

// --- Multi-level Relations

// We can extend the relations by adding
// more parent-children pairs
// Now, we have:
// company -> 
//    department ->
//        employee
//    supplier
PUT /company
{
  "mappings": {
    "properties": {
      "join_field": { 
        "type": "join",
        "relations": {
          // We add new relations as a key-value pairs (parent-children)
          "company": ["department", "supplier"],
          "department": "employee"
        }
      }
    }
  }
}

// Adding a company (ID: 1)
PUT /company/_doc/1
{
  "name": "My Company Inc.",
  "join_field": "company"
}
// Adding a department (ID: 2)
PUT /company/_doc/2?routing=1
{
  "name": "Development",
  "join_field": {
    "name": "department",
    "parent": 1
  }
}
// Adding an employee (ID: 3)
// NOTE: the rounting and parent values are different now!
// - routing refers to the furtherst parent (company)
// - parent refers to the next parent (department)
PUT /company/_doc/3?routing=1
{
  "name": "Bo Andersen",
  "join_field": {
    "name": "employee",
    "parent": 2
  }
}

// Search: works similarly as before
// Queries are nested one inside the other.
// In this example, a company is returned
// which has a department which has an employee
// with the specified name
GET /company/_search
{
  "query": {
    "has_child": {
      "type": "department",
      "query": {
        "has_child": {
          "type": "employee",
          "query": {
            "term": {
              "name.keyword": "John Doe"
            }
          }
        }
      }
    }
  }
}

// --- Parent/Child Inner Hits

// Including inner hits for the `has_child` query.
// Departments which have employees
// which match the specified requirements;
// inner hit employees are returned, too.
GET /department/_search
{
  "query": {
    "has_child": {
      "type": "employee",
      "inner_hits": {},
      "query": {
        "bool": {
          "must": [
            {
              "range": {
                "age": {
                  "gte": 50
                }
              }
            }
          ],
          "should": [
            {
              "term": {
                "gender.keyword": "M"
              }
            }
          ]
        }
      }
    }
  }
}

// Including inner hits for the `has_parent` query;
// inner hit departments are returned, too
// (in this case, it's trivial, i.e., only one Department)
GET /department/_search
{
  "query": {
    "has_parent": {
      "inner_hits": {},
      "parent_type": "department",
      "query": {
        "term": {
          "name.keyword": "Development"
        }
      }
    }
  }
}

// --- Terms Lookup Mechanism

// The idea behind is to use 
// the values from another document 
// to perform the search. 
// We could do it in other ways (e.g., with two queries), 
// but the way shown here is the optimum, 
// because the minimum amount of queries 
// are performed behind the hood.

// First, user and stories indices are created and filled:
// - users who have a name and can follow other users
// - and stories, which are posted by users.
PUT /users/_doc/1
{
  "name": "John Roberts",
  "following" : [2, 3]
}
PUT /users/_doc/2
{
  "name": "Elizabeth Ross",
  "following" : []
}
PUT /users/_doc/3
{
  "name": "Jeremy Brooks",
  "following" : [1, 2]
}
PUT /users/_doc/4
{
  "name": "Diana Moore",
  "following" : [3, 1]
}
PUT /stories/_doc/1
{
  "user": 3,
  "content": "Wow look, a penguin!"
}
PUT /stories/_doc/2
{
  "user": 1,
  "content": "Just another day at the office... #coffee"
}
PUT /stories/_doc/3
{
  "user": 1,
  "content": "Making search great again! #elasticsearch #elk"
}
PUT /stories/_doc/4
{
  "user": 4,
  "content": "Had a blast today! #rollercoaster #amusementpark"
}
PUT /stories/_doc/5
{
  "user": 4,
  "content": "Yay, I just got hired as an Elasticsearch consultant - so excited!"
}
PUT /stories/_doc/6
{
  "user": 2,
  "content": "Chilling at the beach @ Greece #vacation #goodtimes"
}

// Now, the term lookup query.
// The idea behind is to use the values
// from another document to perform the search.
// We could do it in other ways (e.g., with 2 queries), 
// but the way shown here is the optimum, 
// because the minimum amount of queries 
// are performed behind the hood.
// Query: Retrieve all stories posted by users 
// that user 1 is following.
GET /stories/_search
{
  "query": {
    "terms": {
      "user": {
        "index": "users",
        "id": "1",
        "path": "following"
      }
    }
  }
}
```

## Controlling Query Results

Contents:

- Specifying the Result Format
- Source Filtering
- Specifying the Result Size and Offset
- Sorting Results

```json
// -- Specifying the Result Format

// YAML format output
GET /recipes/_search?format=yaml
{
    "query": {
      "match": { "title": "pasta" }
    }
}

// Returning pretty JSON
// This is helpful when we're debugging
// in the Terminal.
GET /recipes/_search?pretty
{
    "query": {
      "match": { "title": "pasta" }
    }
}

// --- Source Filtering

// Sometimes the `_source` is not necessary at all,
// and we can decide to restrict 
// its returned content to increase performance.
// We can specify to return given keys/objects within the source
GET /recipes/_search
{
  "_source": false, // exclude source
  //"_source": "created", // only return field "created"
  //"_source": "ingredients.name", // only return "ingredients" object's key "name"
  //"_source": "ingredients.*", // return all object's keys 
  //"_source": [ "ingredients.*", "servings" ], // returns "ingredient" object's keys and "servings field"
  "query": {
    "match": { "title": "pasta" }
  }
}

// Also, we can be more selective, e.g.:
// Including all of the `ingredients` object's keys, except the `name` key
// (this query doesn't really make sense)
GET /recipes/_search
{
  "_source": {
    "includes": "ingredients.*",
    "excludes": "ingredients.name"
  },
  "query": {
    "match": { "title": "pasta" }
  }
}

// --- Specifying the Result Size and Offset

// Using a query parameter
GET /recipes/_search?size=2
{
  "_source": false,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}

// Using a parameter within the request body
GET /recipes/_search
{
  "_source": false,
  "size": 2,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}

// Specifying an offset with the `from` parameter
GET /recipes/_search
{
  "_source": false,
  "size": 2,
  "from": 2,
  "query": {
    "match": {
      "title": "pasta"
    }
  }
}

// --- Sorting Results

// The default behavior is sorting by score, 
// but we can alter that by specifying fields
// by which we would like to sort.
// Sorting by ascending order (implicitly) 
// of the field preparation_time_minutes.
// The result contains a sort value
// related to the sorted field
// so we can exclude the _source object
// since we're really interested only on the
// returned, sorted values of preparation_time_minutes
GET /recipes/_search
{
  "_source": false,
  "query": {
    "match_all": {}
  },
  "sort": [
    "preparation_time_minutes"
  ]
}

// Sorting by descending order
// of the field created.
// Since created is in milliseconds,
// we include the created field from _source
// which is a date in a human readable format,
// i.e., we do it for easier interpretation.
GET /recipes/_search
{
  "_source": "created",
  "query": {
    "match_all": {}
  },
  "sort": [
    { "created": "desc" }
  ]
}

// Sorting by multiple fields:
// sorting is done in order of specification.
// We add the sorted/searched fields to
// _source just for easier interpretation.
GET /recipes/_search
{
  "_source": [ "preparation_time_minutes", "created" ],
  "query": {
    "match_all": {}
  },
  "sort": [
    { "preparation_time_minutes": "asc" },
    { "created": "desc" }
  ]
}

// Multi-value fields (e.g., arrays) can also be used
// for sorting; with them, we can specify
// an aggregation mode for the field.
// Example: ratings is an array with several scores.
// Sorting by the average rating (descending)
GET /recipes/_search
{
  "_source": "ratings",
  "query": {
    "match_all": {}
  },
  "sort": [
    {
      "ratings": {
        "order": "desc",
        "mode": "avg"
      }
    }
  ]
}
```

## Aggregations

Contents:

- Metric Aggregations
- Bucket Aggregations
- Nested Aggregations
- Filtering Out Documents
- Bucket Rules with Filters
- Range Aggregations
- Histograms
- Global Aggregations
- Missing Field Values
- Aggregation of Nested Objects


```json
// --- Metric Aggregations

// Aggregations are similar to the ones in SQL
// Aggregations run always in a search query context, 
// which by default (implicitly) is `match_all`,
// i.e., if not other explicit search query used.
// Example: Calculating statistics with `sum`, `avg`, `min`, and `max` aggregations.
// We use the _search API,
// where we can specify any search query (e.g., range query);
// but we don't really need to write a search query,
// instead we can run an aggs query and specify 
// the field + operation (sum, avg, min, max).
// Without search query, all documents are considered.
GET /orders/_search
{
  "size": 0, // otherwise, used documents are also output
  "aggs": {
    "total_sales": { // arbitraty name of the aggregation
      "sum": { // type of aggregation operation
        "field": "total_amount" // field name to be aggregated with peration
      }
    },
    "avg_sale": {
      "avg": {
        "field": "total_amount"
      }
    },
    "min_sale": {
      "min": {
        "field": "total_amount"
      }
    },
    "max_sale": {
      "max": {
        "field": "total_amount"
      }
    }
  }
}

// Retrieving the number of distinct values
// with the operation cardinality.
// Watch out: approximate numbers produced...
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "total_salesmen": {
      "cardinality": {
        "field": "salesman.id"
      }
    }
  }
}

// Retrieving the number of values
// with value_count
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "values_count": {
      "value_count": {
        "field": "total_amount"
      }
    }
  }
}

// Using `stats` aggregation for common statistics:
// count, min, max, avg, sum
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_stats": {
      "stats": {
        "field": "total_amount"
      }
    }
  }
}

// --- Bucket Aggregations

// Bucket aggregations create sets (i.e., buckets) of documents instead of metrics.
// We use the "terms" operation within "aggs",
// that way we create a bucket of each of the possible
// terms/values in the field we specify.
// Bucket aggregations are similar to GROUP BY in SQL.
// WARNING: counts from "terms" can be approximate
// if we are using distributed shards and use small
// top-n queries with small n=size values.
// The default size is 10; we can in crease it
// if we want a higher accuracy.
// Example: Creating a bucket for each `status` value.
// The values are returned as
// - "key": one possible value
// - "doc_count": number of documents with the possible value
// WARNING: unless we change it, only the first unique 10 terms
// are use dto create the buckets.
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status"
      }
    }
  }
}

// Including `20` terms instead of the default `10`
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "size": 20
      }
    }
  }
}

// Aggregating documents with missing field (or `NULL`)
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "size": 20,
        "missing": "N/A"
      }
    }
  }
}

// Changing the minimum document count for a bucket to be created
// Usage example: here N/A does not really appear, so no bucket
// is created; however, if we set "min_doc_count": 0, then
// we will get a bucket of 0 documents also for the value "N/A"
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "size": 20,
        "missing": "N/A",
        "min_doc_count": 0
      }
    }
  }
}

// Ordering the buckets
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status",
        "size": 20,
        "missing": "N/A",
        "min_doc_count": 0,
        "order": {
          "_key": "asc"
        }
      }
    }
  }
}

// --- Nested Aggregations

// Nested aggregations are sub-aggregations, 
// i.e., bucket aggregations of bucket aggregations.
// The syntax is the same as before, 
// we simply we nest `aggs` within `aggs`.
// Retrieving statistics for each status
// First, we create buckets for each status
// then, we apply a metric aggregation for each bucket.
// As a result here, we get the stats for each bucket/group.
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status"
      },
      "aggs": {
        "status_stats": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}

// Narrowing down the aggregation context
// Implicitly, match_all is used, i.e., all documents
// are used, but we can run a search query which
// narrows down to a subset; e.g.: a range query.
// Recall: aggregations run always in a search query context
// which is match_all (i.e., all documents) if no other query specified.
// Here, a range query is used.
GET /orders/_search
{
  "size": 0,
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "aggs": {
    "status_terms": {
      "terms": {
        "field": "status"
      },
      "aggs": {
        "status_stats": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}

// --- Filtering Out Documents

// Instead of filtering in the search query
// and then running nested aggregations, 
// we can directly filter in the aggregation queries.
// Example: Filtering out documents with low `total_amount`,
// we get the doc count.
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "low_value": {
      "filter": {
        "range": {
          "total_amount": {
            "lt": 50
          }
        }
      }
    }
  }
}

// We can go beyond and add sub-aggregation queries.
// Example: Aggregating on the bucket of remaining documents
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "low_value": {
      "filter": {
        "range": {
          "total_amount": {
            "lt": 50
          }
        }
      },
      "aggs": {
        "avg_amount": {
          "avg": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}

// --- Bucket Rules with Filters

// Instead of using `aggs` and `terms`,
// we can use `aggs` and `filters`. 
// The `filters` aggregation query allows
// to specify any criteria for our bucket,
// it doesn't need to be constrained to the levels 
// (i.e., unique values) a field has.
// Example: Placing documents into buckets based on criteria
GET /recipes/_search
{
  "size": 0,
  "aggs": {
    "my_filter": {
      "filters": {
        "filters": {
          "pasta": {
            "match": {
              "title": "pasta"
            }
          },
          "spaghetti": {
            "match": {
              "title": "spaghetti"
            }
          }
        }
      }
    }
  }
}

// Example: Calculate average ratings for buckets
// We can define subaggregations within `filters`
GET /recipes/_search
{
  "size": 0,
  "aggs": {
    "my_filter": {
      "filters": {
        "filters": {
          "pasta": {
            "match": {
              "title": "pasta"
            }
          },
          "spaghetti": {
            "match": {
              "title": "spaghetti"
            }
          }
        }
      },
      "aggs": {
        "avg_rating": {
          "avg": {
            "field": "ratings"
          }
        }
      }
    }
  }
}

// --- Range Aggregations

// With `range` aggregation
// we can create buckets associated to a range of a field
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "range": {
        "field": "total_amount",
        "ranges": [ // we create 3 buckets, each associated with a range
          {
            "to": 50 // [0,50]
          },
          {
            "from": 50,
            "to": 100
          },
          {
            "from": 100 // [100, inf)
          }
        ]
      }
    }
  }
}

// The operation `date_range` aggregation
// is the same as `range`, but for dates.
// We can also 
// - define keys (i.e., names) for buckets (manually or automatically)
// - specify date formats
// - and add sub-aggregations
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "purchased_ranges": {
      "date_range": {
        "field": "purchased_at",
        "format": "yyyy-MM-dd",
        "keyed": true,
        "ranges": [
          {
            "from": "2016-01-01",
            "to": "2016-01-01||+6M",
            "key": "first_half"
          },
          {
            "from": "2016-01-01||+6M",
            "to": "2016-01-01||+1y",
            "key": "second_half"
          }
        ]
      },
      "aggs": {
        "bucket_stats": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}

// --- Histograms

// Histograms are a bucket aggregation 
// in which we simply define an interval 
// and the documents are grouped depending on 
// the interval they belong to.
// Example: Distribution of `total_amount` with interval `25`
// Buckets are created every 25
// and documents assigned to them.
// Note that we could have empty buckets.
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "histogram": {
        "field": "total_amount",
        "interval": 25
      }
    }
  }
}

// Requiring minimum 1 document per bucket
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "amount_distribution": {
      "histogram": {
        "field": "total_amount",
        "interval": 25,
        "min_doc_count": 1
      }
    }
  }
}

// Specifying fixed bucket boundaries
GET /orders/_search
{
  "size": 0,
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "aggs": {
    "amount_distribution": {
      "histogram": {
        "field": "total_amount",
        "interval": 25,
        "min_doc_count": 0,
        "extended_bounds": {
          "min": 0,
          "max": 500
        }
      }
    }
  }
}

// Aggregating by month with the `date_histogram` aggregation
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "orders_over_time": {
      "date_histogram": {
        "field": "purchased_at",
        "calendar_interval": "month"
      }
    }
  }
}

// --- Global Aggregations

// When we run a search query, 
// we set a document context composed of the documents
// that match that query. 
// Then, if we apply an aggregation, 
// it is carried out on those selected documents.
// However, we can break that selection
// and apply the aggregation to all the documents
// by specifying the objecy `global`.
// Example: Break out of the aggregation context
GET /orders/_search
{
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "size": 0,
  "aggs": {
    "all_orders": {
      "global": { },
      "aggs": {
        "stats_amount": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    }
  }
}

// The whole global exemption makes sense
// if we want to create aggregation buckets
// with different scopes together/simultaneously.
// Example: Adding aggregation without global context.
// Check how the counts for each bucket are different
GET /orders/_search
{
  "query": {
    "range": {
      "total_amount": {
        "gte": 100
      }
    }
  },
  "size": 0,
  "aggs": {
    "all_orders": {
      "global": { },
      "aggs": {
        "stats_amount": {
          "stats": {
            "field": "total_amount"
          }
        }
      }
    },
    "stats_expensive": {
      "stats": {
        "field": "total_amount"
      }
    }
  }
}

// --- Missing Field Values

// Aggregating documents with missing field value
// This aggregation with `missing`
// finds all documents with a status field value null
GET /orders/_search
{
  "size": 0,
  "aggs": {
    "orders_without_status": {
      "missing": {
        "field": "status"
      }
    }
  }
}

// --- Aggregation of Nested Objects

// Example: This query returns a bucket with all the employees
// because no filtering is applied.
GET /department/_search
{
  "size": 0,
  "aggs": {
    "employees": {
      "nested": {
        "path": "employees" // higher level object
      }
    }
  }
}

// We can use any sub-aggregation with
// the buckets we have created.
GET /department/_search
{
  "size": 0,
  "aggs": {
    "employees": {
      "nested": {
        "path": "employees"
      },
      "aggs": {
        "minimum_age": {
          "min": {
            "field": "employees.age"
          }
        }
      }
    }
  }
}
```

## Improving Search Results

Contents:

- Proximity Searches
- Affecting Relevance Scoring with Proximity
- Fuzzy Queries
- Adding Synonyms
- Adding Synonyms from File
- Highlighting Matches in Fields
- Stemming

```json
// --- Proximity Searches

// If we are looking for phrases,
// all terms need to appear
// and the word order matters, e.g., if we look for "spicy sauce"
// - "Spicy Sauce" will be matched
// - "Spicy Tomato Sauce" won't be matched, but we want it to match!
// Solution: Adding the `slop` parameter to a `match_phrase` query
// slop refers to how many positions a term can be moved,
// it also allows different order of terms, i.e., it's the edit distance.
// Also: the closest the terms are (proximity), the higher is the relevance score,
// but there's no guarantee that the documents with the closest phrases
// will rank the best, because the relevance can be affected by other factors.
GET /proximity/_search
{
  "query": {
    "match_phrase": {
      "title": {
        "query": "spicy sauce", // matches also "Spicy Tomato Sauce"
        "slop": 1
      }
    }
  }
}

// --- Affecting Relevance Scoring with Proximity

// One easy way of affecting the relevance scores
// is to create bool queries that contain
// several optional queries.
// For instance, match looks for the terms
// and match_phrase
// boosts relevance based on proximity
// but allows some flexibility with the `slop` parameter.
// Again, there's no guarantee that the most relevant
// documents will be the first, but it's a heuristic
// in that direction.
GET /proximity/_search
{
  "query": {
    "bool": { // Within bool we can have several queries
      "must": [
        {
          "match": {
            "title": {
              "query": "spicy sauce"
            }
          }
        }
      ],
      "should": [ // Optional, but if it matches, it boosts relevance
        {
          "match_phrase": {
            "title": {
              "query": "spicy sauce",
              "slop": 5
            }
          }
        }
      ]
    }
  }
}

// --- Fuzzy Queries

// We can deal with typos and related misspelling errors
// by using the `fuzziness` parameter in `match` queries. 
// Also we can use the `fuzzy` query. 
// Differences:
// - The `fuzziness` parameter in a `match` query is a full text query, 
// i.e., we use an analyzer.
// - The `fuzzy` query is a term-level query, i.e., 
// no analyzer is used and exact matches are targeted 
// (with some edit distances, depending on teh fuzziness level).
// Example: we want to search the term "lobster", but type "l0bster".
// We set `fuzziness` to `auto`.
// Fuzziness is implemented with the edit distance,
// i.e., the Levenshtein distance between strings.
// Generally, leaving "fuzziness": "auto" is the best approach.
// The maximum edit distance under the hood is computed
// depending on the term length; however, the maximum
// value is 2:
// a maximum number of 2 insertion/deletion/substitution/transpositions are allowed, 
// since these cover 80% of all misspellings.
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "l0bster",
        "fuzziness": "auto"
      }
    }
  }
}

// Fuzziness is applied per term here.
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "l0bster love",
        "operator": "and",
        "fuzziness": 1
      }
    }
  }
}

// Switching letters around with transpositions,
// one transposition is one edit distance unit.
// Transpositions can be disabled (enabled by default).
GET /products/_search
{
  "query": {
    "match": {
      "name": {
        "query": "lvie", // correct: "live"
        "fuzziness": 1
        //"fuzzy_transpositions": false
      }
    }
  }
}

// Apart from adding the fuzziness parameter
// we can also use the fuzzy query.
// BUT: the fuzzy query is a term query, i.e.
// it is not analyzed!
// Therefore, in this case the search won't yield
// any results, because the edit distance 
// lobster <-> LOBSTER
// it too high.
GET /products/_search
{
  "query": {
    "fuzzy": {
      "name": {
        "value": "LOBSTER", // correct "lobster"
        "fuzziness": "auto" // Optional
      }
    }
  }
}

// --- Adding Synonyms

// Defining synonyms can help improve our queries 
// because sometimes the same conecpt can be expressed
// with different words. 
// Synonyms are defined in the analyzer,
// i.e., we actually create a custom analyzer 
// which contains the synonyms in a `filter`.
// There are a couple of things we should consider:
// - The order/place in which the synonyms are inegrated matters: 
// if we first lowercase and the define synonyms with capital letters,
// they won't really work. 
// Similarly, synonyms need to be defined before stemming, 
// otherwise they won't be catched.
// - The syntax is `matched term(s) => replacement term(s)`.
// - If we define several matched terms with commas, 
// all will be replaced one by one to the replacement terms.
// - If we define several replacement terms, 
// all will be used to replace the original terms,
// and all will have the same position number.
// - If we define alist of words, they all take the same position.
// Example: Creating index with custom analyzer
PUT /synonyms
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym", 
          "synonyms": [
            "awful => terrible",
            "awesome => great, super", // one replaced by two, the two take same position
            "elasticsearch, logstash, kibana => elk", // all 3 replaced by elk
            "weird, strange" // if any present, both placed and in same position
          ]
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}

// Testing the analyzer (with synonyms)
POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "awesome" // great, super (both in position 0)
}
POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Elasticsearch" // elk
}
POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "weird" // weird, strange (both in position 0)
}
POST /synonyms/_analyze
{
  "analyzer": "my_analyzer",
  "text": "Elasticsearch is awesome, but can also seem weird sometimes."
}

// Adding a test document
POST /synonyms/_doc
{
  "description": "Elasticsearch is awesome, but can also seem weird sometimes."
}

// Searching the index for synonyms
// Both terms will be found, even though we introduced only awesome.
GET /synonyms/_search
{
  "query": {
    "match": {
      "description": "great"
      //"description": "awesome"
    }
  }
}

// --- Adding Synonyms from File

// Usually, we prefer to add synonyms to an analyzer using a file.
// To that end, we need to use the parameter `synonyms_path`; there we put
// - either an absolute path
// - or a relative path to the config directory.
// The syntax in the file is the same as when we specify the synonyms inline.
PUT /synonyms
{
  "settings": {
    "analysis": {
      "filter": {
        "my_synonym_filter": {
          "type": "synonym",
          "synonyms_path": "analysis/synonyms.txt"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "my_synonym_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}

// --- Highlighting Matches in Fields

// We can return an array of highlighted text parts
// where the searched tokens are matched.
// Example test document
PUT /highlighting/_doc/1
{
  "description": "Let me tell you a story about Elasticsearch. It's a full-text search engine that is built on Apache Lucene. It's really easy to use, but also packs lots of advanced features that you can use to tweak its searching capabilities. Lots of well-known and established companies use Elasticsearch, and so should you!"
}

// Highlighting matches within the `description` field.
// The returned matches have a highlight object
// which contains an array with the found tokens highlighted in <em> tags
// along with the surrounding words.
GET /highlighting/_search
{
  "_source": false,
  "query": {
    "match": { "description": "Elasticsearch story" }
  },
  "highlight": {
    "fields": { // fields we want to highlight
      "description" : {} // field we want to highlight
    }
  }
}


// --- Stemming

// We can improve the matches of the search queries
// by applying stemming. 
// If we create a custom analyzer to include synonyms, 
// we should consider adding a language-based stemmer.
// Stemmed words can be similarly highlighted, too.
// Example: Creating a test analyzer which has a stemmer.
PUT /stemming_test
{
  "settings": {
    "analysis": {
      "filter": {
        "synonym_test": {
          "type": "synonym",
          "synonyms": [
            "firm => company",
            "love, enjoy"
          ]
        },
        "stemmer_test" : {
          "type" : "stemmer",
          "name" : "english"
        }
      },
      "analyzer": {
        "my_analyzer": {
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "synonym_test",
            "stemmer_test"
          ]
        }
      }
    }
  },
  "mappings": {
    "properties": {
      "description": {
        "type": "text",
        "analyzer": "my_analyzer"
      }
    }
  }
}

```