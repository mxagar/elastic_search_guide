# Questions and Ideas about ElasticSearch

## Questions

- How does the k-NN plugin work? What is the underlying data structure? What is the set of algorithms used to compute it?

- How does fuzzy-matching work? (to handle typos and spelling variations)

- Currently, the standard way of integrating ES is to duplicate the DB data into ES; isn't that a waste of resources?

- Have they considered going beyond the REST API style as 

## Ideas

- An assistant, copilot: chatbot + suggestions

### Ideas: What an Assistant Cold Do

Low hanging fruits:

- Suggest performance improvement parameter modifications:
  - disable `doc_values`
  - disable `norms`
  - disable `index`

- Suggest field aliases instead of reindexing if the only purpose of reindexing is renaming fields

- Suggest typical types for fields; show pros & cons of each type

## Key Points

- Multiple data structures, not just one table for each mapping; each field has a/several data structure(s) behind:
  - Inverted indices: terms, text; for fast non-exact search/matches.
  - BKD Trees: numerical data; for fast range-based search/matches.
  - Doc Values or columnar data: numerical data; for fast aggregation operations.
  - ...
- Flexibility: indices can be dynamically created, no schedule jobs needed.
