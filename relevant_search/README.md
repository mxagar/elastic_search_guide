# Relevant Search: Notes

These are my notes of the book [Relevant SEARCH, by Doug Turnbull and John Berryman](https://www.manning.com/books/relevant-search).

Table of contents:

- [Relevant Search: Notes](#relevant-search-notes)
  - [Chapter 1: The search relevance problem](#chapter-1-the-search-relevance-problem)
  - [Chapter 2: Search under the hood](#chapter-2-search-under-the-hood)
  - [Chapter 3: Debugging your first relevance problem](#chapter-3-debugging-your-first-relevance-problem)
  - [Chapter 4: Taming tokens](#chapter-4-taming-tokens)
  - [Chapter 5: Basic Multi-field search](#chapter-5-basic-multi-field-search)
  - [Chapter 6: Term-centric search](#chapter-6-term-centric-search)
  - [Chapter 7: Shaping the relevance function](#chapter-7-shaping-the-relevance-function)
  - [Chapter 8: Providing relevance feedback](#chapter-8-providing-relevance-feedback)
  - [Chapter 9: Designing a relevance-focused search application](#chapter-9-designing-a-relevance-focused-search-application)
  - [Chapter 10: The relevance-centered enterprise](#chapter-10-the-relevance-centered-enterprise)
  - [Chapter 11: Semantic and personalized search](#chapter-11-semantic-and-personalized-search)

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

## Chapter 3: Debugging your first relevance problem



## Chapter 4: Taming tokens



## Chapter 5: Basic Multi-field search



## Chapter 6: Term-centric search



## Chapter 7: Shaping the relevance function



## Chapter 8: Providing relevance feedback



## Chapter 9: Designing a relevance-focused search application



## Chapter 10: The relevance-centered enterprise



## Chapter 11: Semantic and personalized search


