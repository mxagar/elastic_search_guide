# Relevant Search: Notes

These are my notes of the book [Relevant SEARCH, by Doug Turnbull and John Berryman](https://www.manning.com/books/relevant-search).

Table of contents:

- [Relevant Search: Notes](#relevant-search-notes)
  - [Setup](#setup)
  - [Chapter 1: The search relevance problem](#chapter-1-the-search-relevance-problem)
  - [Chapter 2: Search under the hood](#chapter-2-search-under-the-hood)
  - [Chapter 3: Debugging your first relevance problem](#chapter-3-debugging-your-first-relevance-problem)
    - [Key points](#key-points)
  - [Chapter 4: Taming tokens](#chapter-4-taming-tokens)
  - [Chapter 5: Basic Multi-field search](#chapter-5-basic-multi-field-search)
  - [Chapter 6: Term-centric search](#chapter-6-term-centric-search)
  - [Chapter 7: Shaping the relevance function](#chapter-7-shaping-the-relevance-function)
  - [Chapter 8: Providing relevance feedback](#chapter-8-providing-relevance-feedback)
  - [Chapter 9: Designing a relevance-focused search application](#chapter-9-designing-a-relevance-focused-search-application)
  - [Chapter 10: The relevance-centered enterprise](#chapter-10-the-relevance-centered-enterprise)
  - [Chapter 11: Semantic and personalized search](#chapter-11-semantic-and-personalized-search)

## Setup

Github repository with code: [o19s/relevant-search-book](https://github.com/o19s/relevant-search-book).

## Chapter 1: The search relevance problem

- Search is everywhere, seamlessly integrated.
- **Relevance**: ranking content for a search based on how much that content satisfies, among others, the *needs of the user*. Depending on the application, relevance varies:
  - Web search: trustworthiness is important, i.e., Google's PageRank.
  - E-Commerce: affordable products are important, but also profit for Amazon, i.e., properly using stock.
  - Expert search: doctors might use jargon and perform a practical search to know about how to help their patients or search to do research on illnesses.
  - Etc.
- Therefore, relevance is not related only to satisfying the *user needs*, it is also associated with:
  - Satisfying business needs (e.g., Amazon wants to sell and make profit, Google wants to show adds, etc.)
  - User experience / background / motivation, or application context (e.g., doctors looking for information for patients or research)
- **Information Retrieval (IR)**: academic discipline behind search relevance.
  - Relevance: related to most satisfying information for user.
  - IR: focused on ranking.
- Key elements of a search and relevance:
  - **Features**: properties of the items being searched, e.g., descriptive relevant characteristics like "T-Shirt", or its color and size. 
  - **Signals**: cues used to modify the ranking during search-time; they affect the ranking function, e.g., user ratings.
- Information **curation** from domain experts and **feedback** from users (explicit or even automatic, as I understand) are fundamental to improve search.

## Chapter 2: Search under the hood

- Documents
  - Fields; typed. Strings most relevant: strings are searched within.
  - Analysis: tokenization, stop-words, lowercase, stemming, filtering, etc.
  - Indexing of documents happens after analysis.
- Inverted index: mapping of ordered terms (lexicographical order) + list of documents (aka. postings). In Apache Lucene, they have other additional data:
  - Doc frequency: count of docs that contain a term.
  - Term frequency: count of times a term appears in a doc.
  - Term positions: term positions might yield relevant semantic information.
  - Term offsets: when results are highlighted in the text fields to the user, we need to have fast start-end slicing of the original text, done by using pre-stored offsets.
  - Payloads: additional information associated to text, used for ranking; e.g., part of speech, external term values, etc.
  - Stored fields: real source values, often stored somewhere else.
  - Doc values
- ETL process in search: Extract, Transform, Load
  - Extract: retrieve documents from source
    - Best case: fom relational DB
    - Worst case: MS Word, PDFs
    - Own the extraction process!
  - Transform: enrich, analyses
    - Enrich = augment with info for relevant search
      - Clean
      - Augment with ML features like sentiment, cluster
      - Merge external data, if available: metadata, etc.
    - Analysis: tokenization, filtering, etc.
      - We can tokenize also geolocations
      - Query and indexing analysis ust be the same
      - Be familiar with the analysis to improve search
      - Components of analysis
        - Character filtering: HTML tags
        - Tokenization: usually, standard tokenizer
        - Token filtering: stop words
      - Though: stemming maybe makes sense when you're creating inverted indices, but not so much when you're capturing the entire meaning; same for stop words
      - Payloads: metadata associated with tokens
        - Term positions
        - Term offsets: for fast highlights
        - Beware: it increases storage needs!
  - Load: index processed documents, i.e., place data into data structures
    - We need to decide which fields to index and which not!
      - Only indexed files will be searchable
    - Text fields are indexed in inverted indices
      - Rest of field types are indexed in other data structures
    - Non-indexed fields are stored, unaltered.
- Document search and retrieval
  - Boolean search: AND (intersection), OR (union), NOT
    - "dress shoe" -> "dress" AND "show", both need to appear
  - However, in Apache Lucene and related, instead of Boolean operator, other are used: MUST, MUST_NOT, SHOULD.
    - MUST: has to have a match in document
    - MUST_NOT: documents that match will be excluded, even if they comply with other clauses
    - SHOULD: might or might not have a match
  - Example:
    ```
    black +cat -dog (Lucene notation)
        SHOULD black MUST cat MUST_NOT dog
    (cat or (black AND cat)) AND NOT dog
        equivalent boolean notation
    ```
  - Positional and phrase matching: position of word might carry semantic meaning
    - We can perform a phrase query: "dress shoes"
    - First all documents with "dress" and "shoes" returned
    - Then those items that have no adjacent terms discarded
  - Sorting, ranked results and relevance
    - Users can often filter results: price, brand, etc.
    - Results are listed according to an order
      - Number-based
      - Lexicographical
      - Relevance: but what dictates relevance?
    - Relevance defined by a **ranking function**
      - What is important to the user? (e.g., hits for their search)
      - What is important to the company? (e.g., high margin products)
    - Example **ranking function**
      - We are looking for the movie "Back to the future".
      - We search for it: fields `title` and `description` are used to find the tokens.
        - Usually, in a search, several fields are used.
        - Sometimes, even several searches are performed under the hood: regular ones, and extra ones which incorporate common misspellings.
      - For each field, the TF-IDF of each set of tokens is computed.
      - All field TF-IDF values are summed and multiplied by a field weight.
      - Then, field sums are summed and multiplied by the value of the `popularity` field; i.e., a *boost*.
      - The resulting **score** is used to rank the document (i.e., movie), 


## Chapter 3: Debugging your first relevance problem

This chapter is about debugging two key aspects to **relevance**: *matching* and *ranking*.

It makes reference to code in the repository [`o19s/relevant-search-book`](https://github.com/o19s/relevant-search-book), which is the official repository of the book. However, the Python and notebook versions are quite old.

[The Movie Database (TMDB)](https://www.kaggle.com/datasets/tmdb/tmdb-movie-metadata) is used, which contains per movie:

- prose text: overviews, reviews, synopsis, etc.
- shorter text: title, director & actors name
- numerical attributes: user ratings, revenue, number of awards
- movie release dates

:warning: Note: the linked Kaggle version seems to be another one.

First, the REST API of Elasticsearch is used to (bluk) index the movie entries after being loaded.
Then, a search query is run using the `multi_match` query and `requests` for connecting to Elasticsearch:

```python
# We want to look for the movie "Space Jam", but we don't remember its name
usersSearch = "basketball with cartoon aliens"

# Query DSL
query = {
    'query': {
        'multi_match': { 
            'query': usersSearch,
            # Two fields are searched, overview is boosted to be 10x more important
            'fields': ['title^10', 'overview'],
        }
    }
}

# Run search via REST
search(query) # requests.get('http://localhost:9200/tmdb/movie/_search', data=json.dumps(query))
```

This query returns unexpected results, i.e., non-desired movies. The rest of the chapter is about what happened and how we can improve our search.

### Key points

To debug **query matching**, we need to check:

- **Query parsing**: how the query DSL is translated to the matching strategy
  - We can ask Elasticsearch to explain the parsed query: `/tmdb/movie/_search -> /tmdb/movie/_validate/query?explain`
  - That way, the underlying Lucene operations are shown
- **Analysis**: token creation of the query and the document text
  - The document and the query must be analyzed by the same tokenizer
  - We can run the analyzer with `/tmdb/_analyze?analyzer=standard&format=yaml`
  - We can see that the analyzer often takes into account the relative position of the terms and stores them to the inverted index
  - The relative position of some irrelevant tokens can lead to spurious results!
  - Common best practice: If the text is in English, use `english` analyzer instead of the `standard` to stemming and remove stop-words.
    ```python
    mappingSettings = {
        'movie': {
            'properties': {
                'title': {
                    'type': 'string',
                    'analyzer': 'english'
                },
                'overview': {
                    'type': 'string',
                    'analyzer': 'english'
                }
            }              
        }
    }
    ```

To debug **ranking**, we need to check:

- Calculation of **individual match scores**
  - We can add `explain` in the query to get a detailed and nested list of scores for each term; the outermost values are the sum of the innermost ones
    ```python
    query = {
        'explain': True, # <--
        'query': {
            'multi_match': { 
                'query': usersSearch,
                # Two fields are searched, overview is boosted to be 10x more important
                'fields': ['title^10', 'overview'],
            }
        }
    }

    ```
  - Relevance scoring uses theory-inspired heuristics that work well in practice.
  - For each document, document-term matrices can be built following the *bag-of-words* scheme.
    - Then, each document (i.e., movie), is represented by sparse vectors.
  - Similarly, properties could be derived for movies, like *alien-ness, cartoon-ness*, etc.
  - Then, the score is the dot-product between the query vector and the movie vectors, normalized: similarity.
  - In practice, a modified version of TF-IDF or BM25 are used to compute vectors
    - TF-IDF
      - TF: Term-Frequency: how frequently a term occurs in a field
      - IDF: Inverse Document Frequency: how rare (thus, valuable) a matched term is
      - The modification is applied by using 
        - `TF' = sqrt(TF)`
        - `IDF'`: `log` in the `IDF` formula
        - `TF-IDF = TF' * IDF' * (1/sqrt(fieldLength))`
    - BM25 is similar to TF-IDF, but more robust and usually the go-to; the TF part is changed:
      - A saturation point is added.
      - The average document length is considered: above average weighted down, below boosted.
- **How these match scores factor** into the document's overall relevance score with respect to the **query**
  - A query has its `queryWeight`, which is composed by two factors:
    - Boosting: the `^k` 
    - Normalization: it attempts to make scores of different fields comparable, but fails (often discussed to be removed)
  - Boosting places different weights to different terms, and it implicitly considers them comparable, BUT we should not compare scores of different fields, because they belong to different universes!
    - A boost of `[term_1^k, term_2]` doesn't mean `term_1` is `k x` more important, because their weights are different! Therefore, we should always first check the term weights and then decide a boosting strategy.
      - Factors that affect weights: important descriptions present, field length, etc.
    - In the example, it makes much more sense to use: `[term_1, term_2^(1/k)]`.

## Chapter 4: Taming tokens

- Tokenization is effectively the creation of useful features that capture user meaning/intent.
- Fundamental measures for relevance:
  - **Precision**: from detected and shown in the result, which are really relevant? (relevant in result)
  - **Recall** (**= Sensitivity**): from all items in corpus/index, which relevant ones are shown in the result? (relevant in index and shown in result)
  - An, also: **Specificity**: how well irrelevant documents are excluded; complement of recall for negative instances.
  - Precision and Recall are the fundamental, and are often at odds. They can be controlled by loosening/tightening the search criteria/terms, but they often compete against each other. 
- General strategy to get good Precision and Recall:
  - First, increase RECALL: This can be done in many ways, for instance:
    - Use stemming in the analysis: that way tokens are reduced to their basic form that captures the meaning; if the user queries 'running', documents that contain anything related to 'run' will be detected.
      - Thus, if the text is in English, try to use the `english` analyzer, which contains the `english_stemmer` and it removed the `english_stop` words.
      - Increasing the recall this way often decreases the precision, because irrelevant documents will be listed, BUT we'll deal with that later.
    - We could try the phonetic analyzer: this is even more extreme, and not always recommended; tokens are reduced to their spoken symbol, removing vowels (except the first) and double consonants.
      - Effect: misspellings can be handled; recall increases, precision decreases.
  - Second, increase PRECISION by improving the ranking score of the most relevant tokens
    - Documents that contain a search term several times score higher; stemming achieves this.
    - When several terms are in the query, multiple matches are more prominently scored.
- Analysis (tokenizing) strategies: they affect both precision and recall
  - Delimiters
    - Acronyms: words like `I.B.M` can be tokenized to `ibm` by using `word_delimiter` filter
    - Phone numbers can be parsed with regex patterns and tokenized to several terms, e.g., with/out country code, with/out region code
  - Synonyms: defining synonyms helps capture situations in which the user expresses themselves in another way
    - We can create and use a filter (in the specific field) which defiles a `synonyms` field, e.g. `"dress show,dress shoes => dress_shoe, shoe"`
  - In general, there's a common strategy which consists in using different analyzers during indexing and query: **asymmetric analysis**
    - During **indexing**, we can tokenize more versions of the term: the **concrete + general / related** (e.g., synonym)
      - Example: `fuji -> fuji, apple, fruit`
    - During **query**, we tokenize the **concrete** only
      - Example: `fuji -> fuji`
      - If our query is concrete (e.g., `fuji`), the matches will find the documents which contain it.
      - If our query is more general (e.g., `apple`), documents with both general and concrete terms will be delivered!
    - Consider that general terms/tokens appear in more documents, which increases increased index size and requires maybe some TF-IDF normalization.
    - Concrete and general terms create implicitly a taxonomy which can be exploited.
  - Be mindful of the DSL query methods we use:
    - `match` performs analysis
    - `term` does not perform analysis
    - `keyword` treats the text as-is
    - ... 
  - Not only text can be tokenized
    - Numerical data: dates are often tokenized using asymmetric tokenization/analysis:
      - `1945 -> 1, 19, 194, 1945`
    - Geographic data is tokenized using *Z-encoding*: map divided in ABCD quadrants recursively, and again asymmetric tokenization/analysis can be used:
      - `BCDBC -> BCDBC, BCDB, BCD, BC, B`
    - Melodies can be tokenized using the *Parsons Code for Melodic Contours*
      - Code (we would require an audio processing which encodes a whistle to this code):
        - First note `*`
        - If next same pitch `R` (repeat), upper `U`, lower `D` (down).
        - Example: `*URDR`
      - Then, tokenization happens in n-grams: take token and break it into all possible tokens of length n contained in the string.
      - Result: the whistling of one part can be detected in the index, which stores encoded songs!

## Chapter 5: Basic Multi-field search

TBD.

## Chapter 6: Term-centric search

TBD.

## Chapter 7: Shaping the relevance function

TBD.

## Chapter 8: Providing relevance feedback

TBD.

## Chapter 9: Designing a relevance-focused search application

TBD.

## Chapter 10: The relevance-centered enterprise

TBD.

## Chapter 11: Semantic and personalized search

TBD.

