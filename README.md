# Elastic Search Guide

This are my notes on **ElasticSearch** and **search methods** with focus on Machine Learning.

I created most of the content in this `README.md` after following the course [Complete Guide to Elasticsearch (Udemy), by Bo Andersen](https://www.udemy.com/course/elasticsearch-complete-guide), but extended it mainly consulting the official Elastic documentation. The couser by Bo Andersen has a Github repository with a summary of all the commands used: [codingexplained/complete-guide-to-elasticsearch](https://github.com/codingexplained/complete-guide-to-elasticsearch).

My respository is structured as follows:

- [`ml_search/`](./ml_search/)
- [`notebooks/`](./notebooks/)

Mikel Sagardia, 2024.  
No guarantees.

Table of contents:

- [Elastic Search Guide](#elastic-search-guide)
  - [Introduction](#introduction)
    - [Elastic Stack](#elastic-stack)
    - [Common Application Architecture](#common-application-architecture)
  - [Getting Started: Setting Up Elastic Search and Kibana](#getting-started-setting-up-elastic-search-and-kibana)
    - [Setup](#setup)
      - [Elastic Cloud](#elastic-cloud)
      - [Windows](#windows)
      - [Unix: Mac OSX, Linux](#unix-mac-osx-linux)
      - [Docker](#docker)
      - [Summary: How to Start Elastic Search](#summary-how-to-start-elastic-search)
    - [Basic Architecture: Cluster, Nodes, Documents, Indices](#basic-architecture-cluster-nodes-documents-indices)
    - [Inspecting a Cluster with the Console](#inspecting-a-cluster-with-the-console)
    - [Interacting with the Cluster via cURL and Python](#interacting-with-the-cluster-via-curl-and-python)
    - [Building an Index](#building-an-index)
      - [Inverted Index](#inverted-index)
      - [B-Tree](#b-tree)
      - [Doc Values](#doc-values)
    - [Sharding and Scalability](#sharding-and-scalability)
    - [Replication and Snapshots](#replication-and-snapshots)
      - [Creating Indices](#creating-indices)
    - [Adding Nodes to the Cluster](#adding-nodes-to-the-cluster)
    - [Node Roles](#node-roles)
  - [Managing Documents](#managing-documents)
    - [Creating and Deleting Indices](#creating-and-deleting-indices)
    - [Indexing Documents = Adding Documents](#indexing-documents--adding-documents)
    - [Retrieving Documents by ID](#retrieving-documents-by-id)
  - [Mapping \& Analysis](#mapping--analysis)
  - [Searching for Data](#searching-for-data)
  - [Joining Queries](#joining-queries)
  - [Controlling Query Results](#controlling-query-results)
  - [Aggregations](#aggregations)
  - [Improving Search Results](#improving-search-results)
  - [Kibana](#kibana)
  - [Logstash](#logstash)
  - [License](#license)


## Introduction

[Elastic Search](https://www.elastic.co/) is an open source analytics and full-text search engine.

We can add complex search functionalities to our applications (e.g., Wiki, e-commerce, etc.), similar to Google, with:

- autocompletion
- highlighting
- typo correction
- handling synonyms
- adjusting relevance
- etc.

We can also query structured data and we can use the platform as an analytics tool (with visualization).

Common example: APM (Application Performance Management) = store logs, be able to search in them, visualize in dashboard, etc.

We can apply ML techniques to get insights from data: forecasts, anomaly detection, etc.

In ES data is stored in **Documents**, represented as `JSON` objects:

- A Document is analog to a **row** in a relational DB.
- A Document contains **Fields**, analog to columns in a relational DB.

We perform queries via a REST API, also with `JSONs`.

Some other properties:

- ES is written in Java, on top of Apache Lucene.
- Easy to use, but has many features.
- Distributed.
- Highly scalable.
- Used by many large companies: FB, Netflix, etc.
- Queries can be done using Query DSL

### Elastic Stack

Elastic has built several products which can interact with each other:

- Elastic Search (ES): search.
- **Kibana**: analytics and visualization; dashboard for Elastic Search, ML. [Kibana Demo](https://demo.elastic.co/app/dashboards#/view/welcome_dashboard).
- Logstash: processing of logs and any data; events from different sources (there are many input/output plugins) are processed (e.g., clean, structure, etc.) and sent to Elastic Search or other destinations.
- X-Pack: additional features for ES and Kibana, such as 
  - security and access management, 
  - monitoring of resources (CPU, memory, etc.), 
  - **machine learning** (anomaly detection, forecasting, etc.)
  - graph structures
  - SQL capabilities, besides Query DSL
- Beats - Filebeats, Metricbeats, etc.: lightweight agents which collect and send data to ES. Filebeats collects logs. They are for data ingestion.

![Elastic Stack](./assets/elastic_stack.png)

### Common Application Architecture

Let's consider an e-commerce site, where users can buy things via a web store/page. We have these components:

- The web frontend.
- The application backend.
- A relational DB where all the products are stored.

If we want to add realtime search capabilities to the web page so that users can find things easily, we need to add **Elastic Search**.

The best approach is to **replicate** all the data entries in the DB into ES; we can do that 

- initially with a script
- and then, we let the backend update ES when the DB is updated, too.

Then, we might connect **Kibana** to ES in order to visualize data in a dashboard: number of orders per week, revenue, etc.

![Architecture: Basic](./assets/architecture_2.png)

As our web grows, we might have increeased traffic and the servers might be suffering; that's when we add **Metricbeat**, which monitors performance and resources on the backend machine and sends them to ES. Basically, ES opens an ingest node where Metricbeat sends data. We can visualize all that in Kibana, too.

Similarly, we can monitor access and error logs. We do that with **Filebeat**, similar to Metricbeat.

As the web grows we might need some data processing before ingestion to ES. One option would be to implement that processing in the backend, but it has several drawbacks: 

- the backend should run the web, not loose resources processing logs, 
- if we have a decentrilized architecture (i.e., microservices), the logging is also decentralized, and we might want to have a centralized and homogeneous processing.

Thus, it makes sense to add **Logstash** to the architecture, which enables log/data processing before ingestion to ES. Although the data from Metricbeat often times doesn't need to be processed, the data from Filebeat could need to be processed, so it is first sent to Logstash, which transforms it and sends it to ES. However, we always have the felxibility to send the data to ES first, without any processing.

![Architecture: Full Elastic Stack](./assets/architecture_3.png)

<!--
Our database and ES should be synchronized. However, ideally, our application should have read permissions in ES. How is that possible?
-->

## Getting Started: Setting Up Elastic Search and Kibana

### Setup

There are many options to install Elastic Search and Kibana:

- Install on local machine: Windows, Mac OS, Linux
- Install as Docker image
- Use Elastic Cloud
- ...

Usually, we want to have an ES cluster on the cloud for scalability; it can be a self-managed ES cluster, not necessarily the Elastic Cloud solution.

#### Elastic Cloud

Elastic Cloud has everything set up and running for us: [https://cloud.elastic.co](https://cloud.elastic.co). There's a free trial for 14 days since the moment we create the cluster; after the 14 days the cluster is shut down.

This is a hosted and managed solution, the easiest one to learn how to use ES.

We can leave everything with the default options:

- If we are asked, we choose a pre-configured Elastic Stack solution.
- Additionally, we can select where to store our data! (platform &mdash; AWS, GCP, Azure &mdash;, and location &mdash; Europe, US, etc. &mdash;).
- If shown, we copy the credentials: `user:elastic`, `pw:...`
- In the new version some other variables are shown: `Elasticsearch endpoint`, `Cloud ID`.
- I copied everything to the non-committed/pushed file `elastic_cloud.txt`.

#### Windows

Full, official guide: [Install Elasticsearch with .zip on Windows](https://www.elastic.co/guide/en/elasticsearch/reference/current/zip-windows.html).

ES and Kibana can be downloaded from 

- [https://www.elastic.co/downloads/elasticsearch](https://www.elastic.co/downloads/elasticsearch)
- [https://www.elastic.co/downloads/kibana](https://www.elastic.co/downloads/kibana)

ES is programmed in Java and Kibana uses npm. However, we don't need to install those dependencies, since they are provided in the archive folder which is contained in the downloaded application. In other words, they are run directly from the uncompressed archive which is downloaded.

To set them up:

- Download the archives of Elastic Search and Kibana.
- Unzip both archives (maybe we need WinRAR/7zip or a similar, because Kibana has a very large archive...).
- Go to the extracted directory and start `elasticsearch`.

```powershell
# Elastic Search: go to extracted directory and run binary
cd C:\Users\msagardia\packages\elasticsearch-8.14.3
bin\elasticsearch.bat
# Unix: bin/elasticsearch
```

:warning: **IMPORTANT**: save the ZIP archive, since we need to extract it every time we want a new node!

When we run `bin\elasticsearch.bat` for the first time a **cluster is created** and its configuration is performed in some minutes. Some values are created are delivered:

- A **superuser** and its password, we copy and save them, e.g., in `.env`
  - User: `elastic`, PW: `...`
  - These are our credentials; we can create other users with restricted pws later one
  - To reset the superuser pw: `bin/elasticsearch-reset-password -u elastic`
- A fingerprint certificate; data is transmitted encrypted.
  - We copy it to `.env`: `ELASTIC_FINGERPRINT`.
- An enrollment token: necessary for secure communications with ES from other nodes, e.g., Kibana.
  - We copy the token to `.env`: `ELASTIC_ENROLLMENT_TOKEN`.
  - We use it to enroll Kibana.
  - The token is valid for 30 mins, but we can create new ones: `bin/elasticsearch-create-enrollment-token.bat -s kibana`
  - We can use this enrollment token to add other nodes, too

When the ES cluster is up and running, we launch Kibana:

```powershell
# Kibana: go to extracted directory and run binary
cd C:\Users\msagardia\packages\kibana-8.14.3
bin\kibana.bat
# Unix: bin/kibana
```

After a short moment, Kibaba is ready at port `5601`:

- We open the URL in the Terminal, which has a security code: `http://localhost:5601/?code=xxxxxx`.
- We are requested for the enrollment token, `ELASTIC_ENROLLMENT_TOKEN`, which we paste.

Now everything is setup. We can log in into Kibana, which is like the GUI for ES. We can user the superuser credentials.

To shut down, we need to `Ctrl+C` both Terminals. To start again, we need to run `bin\elasticsearch.bat` and `bin\kibana.bat` in separate terminals again.

**IMPORTANT: However, we might need to add the following line to the file `.../kibana-8.14.3/config/kibana.yaml`**:

```
elasticsearch.hosts: ["https://localhost:9200"]
```

![Elastic Search Kibana Web UI](./assets/elastic_web_ui_kibana.png)

#### Unix: Mac OSX, Linux

Full, official guide: [Install Elasticsearch from archive on Linux or MacOS](https://www.elastic.co/guide/en/elasticsearch/reference/current/targz.html).

Very similar setup as with Windows: download, uncompress, run scripts, copy credentials, etc.

In the case of MacOS, we need to deactivate the Gatekeeper to be able to run Kibana. So before running `bin/kibana`:

```bash
cd .../path/where/kibana/directory/is
xattr -d -r com.apple.quarantine kibana-8.14.3

# Then, we start Kibana
cd kibana-8.14.3
bin/kibana
```

#### Docker

Full, official guide: [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/current/docker.html).

#### Summary: How to Start Elastic Search

```powershell
## -- Terminal 1
# Elastic Search: go to extracted directory and run binary
cd C:\Users\msagardia\packages\elasticsearch-8.14.3
bin\elasticsearch.bat
# Unix: bin/elasticsearch
# Wait until cluster up & running: "... current.health="GREEN"..."
# If provided, copy to .env
# - ELASTIC_USER
# - ELASIC_PASSWORD
# - ELASTIC_FINGERPRINT
# - ELASTIC_ENROLLMENT_TOKEN
# To get new ELASTIC_ENROLLMENT_TOKEN
# bin/elasticsearch-create-enrollment-token.bat -s kibana
# Also, after first time, add to .../kibana-8.14.3/config/kibana.yaml:
# elasticsearch.hosts: ["https://localhost:9200"]

## -- Terminal 2
# Kibana: go to extracted directory and run binary
cd C:\Users\msagardia\packages\kibana-8.14.3
bin\kibana.bat
# Unix: bin/kibana
# Use ELASTIC_USER & ELASIC_PASSWORD as credentials

## -- Browser
# Kibana Web UI: http://localhost:5601
# Elastic Search API: https://localhost:9200
```

Troublesooting: If we get an error, make sure that `.../kibana-8.14.3/config/kibana.yaml` contains the line

```
elasticsearch.hosts: ["https://localhost:9200"]
```

And, additionally, check:

```powershell
# Set variables in Powershell for easier and more secure use.
# To use them: $Env:ELASTIC_USER
$Env:ELASTIC_USER = "elastic"
$Env:ELASTIC_PASSWORD = "..."
# Bash. To use them: $ELASTIC_USER
export ELASTIC_USER="elastic"
export ELASTIC_PASSWORD="..."

# Example: Basic query to get general cluster info
cd C:\Users\msagardia\packages\elasticsearch-8.14.3
curl.exe --cacert config\certs\http_ca.crt -u "$($Env:ELASTIC_USER):$($Env:ELASTIC_PASSWORD)" --insecure -X GET https://localhost:9200

```

### Basic Architecture: Cluster, Nodes, Documents, Indices

Elastic Search is distirbuted and it consists of **nodes** within a **cluster**:

- When we start an ES instance, it is really a **node** which can contain some TBs of data.
- If our data grows, we can start another instance = another node.
- In developement, we can start several nodes on our device without the need of dealing with VMs or containers; in production, different nodes are usually assigned to different VMs or containers.
- Each/all node/s belong to a **cluster**. Usually we have one cluster for application. Clusters are completely independent from each other.
- When a node starts up:
  - it joins an existing cluster
  - or it starts a cluster and joins to it.
  - Therefore, even a setup with a single node is nested in a cluster.

![Cluster](./assets/cluster.png)

Each data in our nodes is a **Document**, which:

- can be represented as a `JSON`,
- is equivalent to a row in a relational DB,
- consists of the **Fields** we want, which are equivalent to columns in a relational DB.

In addition to our fields, we also store some other metadata in the Document `JSONs`.

![Document](./assets/document.png)

Documents are stored in **Indices**:

- Indices are logical groups of Documents, e.g., *People Index*, *Departments Index*.
- There are no limits in terms of how many Documents go into an Index.
- We run our search queries against Indices.

![Indices](./assets/indices.png)

### Inspecting a Cluster with the Console

![Elastic Search Kibana Web UI](./assets/elastic_web_ui_kibana.png)

When we have started both `elasticsearch` and `kibana`, we open the web UI under the URL:

[`http://localhost:5601/`](http://localhost:5601/)

Then, we can start the **Console** in the web UI:

    Hamburger Menu > Management > Dev Tools

We can use the console to communicate with ES:

- The console uses the REST API under the hood.
- The console is the easiest way to communicate with ES: it has autocompletion.
- However, we often will use the REST API via cURL or related services.

The console takes HTTP methods:

1. **GET:** Retrieve data from Elasticsearch, such as documents or index information.
2. **POST:** Send data to Elasticsearch, typically used for creating or updating documents.
3. **PUT:** Create or replace documents or indices.
4. **DELETE:** Remove documents or indices.
5. **HEAD:** Retrieve metadata without the body.
6. **PATCH:** Apply partial updates to documents.

The query structure is

    HTTP Method + API & Command & Query
    JSON (optional parameters, if required)

For instance:

```bash
# Get cluster health: _cluster API, health Command
GET /_cluster/health
```

When we run that command (play button), we get back a `JSON`:

```json
{
  "cluster_name": "elasticsearch",
  "status": "green",
  "timed_out": false,
  "number_of_nodes": 1,
  "number_of_data_nodes": 1,
  "active_primary_shards": 31,
  "active_shards": 31,
  "relocating_shards": 0,
  "initializing_shards": 0,
  "unassigned_shards": 0,
  "delayed_unassigned_shards": 0,
  "number_of_pending_tasks": 0,
  "number_of_in_flight_fetch": 0,
  "task_max_waiting_in_queue_millis": 0,
  "active_shards_percent_as_number": 100
}
```

Further examples:

```bash
# List all nodes: _cat API (Compact Aligned Text), nodes Command, v Query (verbose)
# In our case we see a single node and its IP + properties
GET /_cat/nodes?v

# List indices
# We only have system indices, leading with a .
GET /_cat/indices?v
```

![Console](./assets/console.png)

### Interacting with the Cluster via cURL and Python

We can interact with ES using its REST API, e.g., via `cURL`. We need to:

- use a certificate located in the elastic package (although that can be bypassed with the flag `--insecure`),
- use HTTPS, not HTTP,
- use our user and PW credentials.

```powershell
# General structure
cd /path/to/elasticsearch
curl --cacert config/certs/http_ca.crt -u elastic:<YOUR_PASSWORD_HERE> [--insecure] -X <HTTP_METHOD> [-H ...] [-d ...] <URL+API+Command+Query>
# NOTE: Sometimes the certificate doesn't work,
# so we can add the flag --insecure.
# This bypasses the SSL certificate verification.
# That should not be done in production!
# The option -H is a header, often used with -d
# and -d is the data we send.
# Example (Windows needs to escape "):
# ... -H "Content-Type: application/json" -d '{"name":"John", "age":30}'
# ... -H "Content-Type: application/json" -d '{\"name\":\"John\", \"age\":30}'

# Set variables in Powershell for easier and more secure use.
# To use them: $Env:ELASTIC_USER
$Env:ELASTIC_USER = "elastic"
$Env:ELASTIC_PASSWORD = "..."
# Bash. To use them: $ELASTIC_USER
export ELASTIC_USER="elastic"
export ELASTIC_PASSWORD="..."

# Example: Basic query to get general cluster info
cd C:\Users\msagardia\packages\elasticsearch-8.14.3
curl.exe --cacert config\certs\http_ca.crt -u "$($Env:ELASTIC_USER):$($Env:ELASTIC_PASSWORD)" --insecure -X GET https://localhost:9200

# Example: Get the index products (not created yet)
curl.exe --cacert config\certs\http_ca.crt -u "$($Env:ELASTIC_USER):$($Env:ELASTIC_PASSWORD)" --insecure -X GET -H "Content-Type: application/json" -d '{ \"query\": { \"match_all\": {} } }' https://localhost:9200/products/_search
# Bash: -d '{ "query": { "match_all": {} } }'

# Example: GET /_cat/nodes?v -> get all nodes
curl.exe --cacert config\certs\http_ca.crt -u "$($Env:ELASTIC_USER):$($Env:ELASTIC_PASSWORD)" --insecure -X GET "https://localhost:9200/_cat/nodes?v"

# Example: GET /_cat/indices?v -> list all indices
curl.exe --cacert config\certs\http_ca.crt -u "$($Env:ELASTIC_USER):$($Env:ELASTIC_PASSWORD)" --insecure -X GET "https://localhost:9200/_cat/indices?v"
```

Besides using `cURL`, we can also connect to Elastic Search using Python. The notebook [`notebooks/elastic_intro.ipynb`](./notebooks/elastic_intro.ipynb) shows how to do that via `requests` and the package `elasticsearch`:

```python
import os
import requests
from requests.auth import HTTPBasicAuth
from dotenv import load_dotenv
load_dotenv()

elastic_user = os.getenv("ELASTIC_USER")
elastic_password = os.getenv("ELASIC_PASSWORD")

### -- requests

# Define the URL
url = "https://localhost:9200/_cat/nodes?v"

# Make the GET request
response = requests.get(
    url,
    auth=HTTPBasicAuth(elastic_user, elastic_password),
    #verify=False,  # Disable SSL verification
    verify="C:\\Users\\msagardia\\packages\\elasticsearch-8.14.3\\config\\certs\\http_ca.crt",
    proxies={"http": None, "https": None}  # Bypass proxy
)

# Print the response
print(response.text)

# Define the URL
url = "https://localhost:9200/_cat/indices?v"

# Make the GET request
response = requests.get(
    url,
    auth=HTTPBasicAuth(elastic_user, elastic_password),
    #verify=False,  # Disable SSL verification
    verify="C:\\Users\\msagardia\\packages\\elasticsearch-8.14.3\\config\\certs\\http_ca.crt",
    proxies={"http": None, "https": None}  # Bypass proxy
)

# Print the response
print(response.text)

### -- elasticsearch

import warnings
from urllib3.exceptions import InsecureRequestWarning
from elasticsearch import Elasticsearch

# Suppress only the specific InsecureRequestWarning
warnings.simplefilter('ignore', InsecureRequestWarning)

# Create an instance of the Elasticsearch client
es = Elasticsearch(
    ['https://localhost:9200'],
    basic_auth=(elastic_user, elastic_password),
    verify_certs=False  # This disables SSL verification, similar to --insecure
)

# Make a GET request to /_cat/nodes?v
response = es.cat.nodes(format="json")

# Print the response
print(response)

# Make a GET request to /_cat/indices?v
response = es.cat.indices(format="json")

# Print the response
print(response)
```

### Building an Index

This is my understanding of how an index works. The key idea is that we'd like to be able to search very quickly our documents.

Let's imagine we want to build an index similar to the ones in Elastic Search. Our goal is fast search in our database.

On one side, we store our Documents, probably as files.

On the other side we build the following structures:

- An **inverted index** which maps terms with documents and frequencies.
- A **B-tree**, which allows fast search and filtering of given (often data/numeric) fields.
- **Doc Values** or fields in **Column Format**, which allow faster sorting and aggregation operations.

In the following, I explain my intuitions of each of them.

#### Inverted Index

An inverted index is a table which maps terms with document ids, positions and frequencies. A possible data structure for that would by a hash table (like a Python dictionary).

An inverted index can be built as follows:

- Each new document is processed by tokenizing (& stemming) is text.
- For each term/token an entry is created and maintained, which contains:
  - List of document ids where the term/token appears.
  - Positions in the document where the term/token appears: field, character position, etc.
  - TF-IDF frequencies to understand how relevant the term is.

TF-IDF, Term Frequency Inverse Document Frequency:

- TF: Measures how frequently a term (word) occurs in a document.

      TF(t,d) = num times term t appears in document d / total num terms in d

- IDF: Measures the importance of a term in the corpus.

      IDF(t,D) = log(total num documents in corpus D / num documents containing term t)

- TF-IDF: **Importance of a term in a document.**

      TFIDF(t,d,D) = TF(t,d) * IDF(t,D)

Probably, for each token = term t, a table of `TF(t,d)` is maintained, as well as the value `IDF(t,D)`. Then, the `TFIDF(t,d,D)` is updated, which is unique for each `(t,d)` pair.

When we start a text search, the query will be tokenized into the indexed terms and those terms are searched in the inverted index (hash table).

With that, we have a list of all the candidate documents. We can:

- get a set of documents common to all terms
- rank the set according to their importance thanks to the `TFIDF(t,d,D)`; the TFIDF is associated to each term-document, but we could compute an aggregate value for each query-document pair.

See [ml_search/search_examples.ipynb](./ml_search/search_examples.ipynb) for a simple implementation.

#### B-Tree

A B-tree is a self-balancing tree data structure that maintains sorted data and allows for efficient insertion, deletion, and search operations. It is commonly used in databases and file systems. Key features:

- Balanced Tree Structure: Ensures that the tree remains balanced by maintaining a certain number of keys in each node. The balancing is automatic.
- Multiple Children: Each node can have multiple children (more than two), which makes the tree shallower and operations faster.
- Efficient Search: By dividing keys among nodes, B-trees allow for `log(n)` search time.
- Disk Storage Friendly: Minimizes disk reads by maximizing the number of keys stored in each node, suitable for systems that read and write large blocks of data.
- Scalability: Handles large amounts of data efficiently.

Operations:

- Search: Similar to binary search but generalized to multiple children.
- Insertion: Adds elements in sorted order, splitting nodes as necessary.
- Deletion: Removes elements while maintaining tree balance, merging nodes if needed.

B-trees are often applied to numerical fields; for instance, let's consider these documents:

```python
[
  { "id": 1, "name": "Product A", "price": 10, "date": "2023-01-01" },
  { "id": 2, "name": "Product B", "price": 20, "date": "2023-01-05" },
  { "id": 3, "name": "Product C", "price": 15, "date": "2023-02-01" },
  { "id": 4, "name": "Product D", "price": 25, "date": "2023-02-03" },
  { "id": 5, "name": "Product E", "price": 30, "date": "2023-02-05" }
]
```

When documents are indexed, the price and date fields are stored using B-trees. This ensures that these fields are efficiently organized for range queries and sorting.

The creation process could be the following:

- Initial Insertion: Start with the root node. Insert the first document's price as the root.
- Subsequent Insertions: Add each document's price in sorted order. If the current node has space (according to B-tree properties, which vary by B-tree degree), insert the new value. Otherwise, split the node and promote the middle value.
- Balancing: Ensure the tree remains balanced. Splitting nodes and promoting middle values help maintain the B-tree properties, ensuring that no node has too many or too few children.

In the previous list of Documents:

    Insert Document 1:
        Root: 10

    Insert Document 2:
        Root: 10
        Child: 20

    Insert Document 3:
        Root: 10
        Children: 15, 20

    Insert Document 4:
        Root: 15
        Children: 10, 20, 25

    Insert Document 5:
        Root: 15
        Children: 10, 20, 25, 30

```
       15
      / | \
    10 20  25
            \
            30
```

See [ml_search/search_examples.ipynb](./ml_search/search_examples.ipynb) for a simple implementation.

#### Doc Values

Doc Values are columnar data of numerical fields. Columnar data is stored as an array, i.e., the values are contiguous in memory. That enables much faster operations in the entire field/column, such as sorting or aggregation operations, like average computation.

I would implement the Doc Value structures with:

- An array which contains the document ids, i.e., an index.
- An array which contains the values in a column/fields, ordered according to the id index.
- A bitstring (an array of bits) used to mask whether to consider the array values or not.

Then, if we want to compute the min/max, mean or similar values of a field/column:

- We take the associated column/field array
- Update the bitstring according to the previous search results (e.g., B-trees & inverted indices).
- Run the column/field-wise operation with the masked array.

See [ml_search/search_examples.ipynb](./ml_search/search_examples.ipynb) for a simple implementation.

### Sharding and Scalability

If our node is full, we can create a new one which will increase the storage capacity of the cluster. In the process, **Sharding** is used:

- Shrading is a way to divide indices into smaller pieces.
- Each piece is referred to as shard.
- Sharding is done at the index level.
- Main purpose: horizontally scale data volume.
- Each shard is independent and a almost a fully functional index on its own.
- Each shard is an Apache Lucene index.
- Each shard can be as big as possible in terms of storage, but it can contain 2 billion documents.
- When we apply sharding, each query is parallelized, i.e., each node and shard run the query.
- We can have primary and replica shard:
  - Primary Shards: These are the main shards that contain the original data.
  - Replica Shards: These are copies of the primary shards and provide failover capabilities, ensuring high availability.

For instance, if we have an index of 600 GB and 2 nodes of 500 GB each, we can divide the index in 2 shards, A and B. Then, each shard goes to a node.

![Sharding](./assets/sharding.png)

The column `pri` when we run `GET request to /_cat/indices?v` is the number of primary shards.

An index contains a single shard by default. If we need to create a new shard, we can use the split API. Similarly, we can recue the number of shards with a shrink API. The number of shards we should use depends on

- the number of indices
- the number of nodes and their capacity
- the number of queries
- ...

Good rule of thumb: if we're going to have millions of documents, use a couple of shards. That number 5 was the default in the older versions.

### Replication and Snapshots

Sometimes node fail; a way to enable fault-tolerance is to replicate shards. Elastic Search replicates shards by default: shards are copied, creating replica shards:

-  When creating an index we can choose how many replicas we'd like (default is 1 replica, i.e, we have 2 shards: primary + replica).
- A replica shard should be always in a different node as the primary shard, i.e., the shard it was copied from, because otherwise we are not fault-tolerant. Thus, replication makes sense in terms of robustness when we have a cluster with at least 2 nodes.
  - In a cluster with a single node, the default replica will be unassigned and the index will be in yellow status.
- A replica shard can be used instead of the primary shard for all the operations.
- For critical operations, we replicate at least 2x, i.e., we'l nned at least 3 nodes.

Replication not only ensure fault-tolerance, but it can also ease the use of parallel queries: we run the same query in two index (shard) copies.; thus, the throughput increases. We can apply that also by having replica shards in the same node, i.e., the same copies in the same node. That is called a **replication group**. In those cases, multi-threding is used.

![Replicas](./assets/replicas.png)

In addition to replication, Elastic Search also allows to take snapshots and backups:

- Snapshots can be used to restore to a given point.
- Snapshots can be taken at index level or for the entire cluster.
- Snapshots are state dumps to a file; those files can be used to restore the cluster, if something goes wrong.

#### Creating Indices

- When we create an index, a replica is automaticaly generated
- However, if we have a single node, that replica is unassigned, so the index is in "yellow" state, not "green" state.
- The system indices (those with leading . in their name) are auto-replicated when we create new nodes; i.e., with one node, they have no replicas.

Here's some commands to create an index and inspect it:

```bash
# Create Index pages
PUT /pages

# Get list of all indices
# pages should appear, but in "yellow" status, 
# because its shard replica is not assigned to another node
GET /_cat/indices?v

# Get list of all shards
# pages shoudl appear 2x, but the replica shard should be UNASSIGNED
GET /_cat/shards?v
```

![Replica shards](./assets/replica_shards.png)

### Adding Nodes to the Cluster

Sharding for fault-tolerance and replication for increasing throughput can be done if we have at least 2 nodes. In managed cloud solutions, nodes are added automatically. However, in local deployments, we need to create nodes manually.

If we are running Elastic Search in a development environment, we can set new nodes as follows:

- Download and extract again the elasticsearch archive in the `.../packages` directory. Name that new folder `elastic-second-node`. It should be at the same level as `elasticsearch-8.14.3/`, which is the first node. Do not copy the folder `elasticsearch-8.14.3/`, but extract a new folder frmo the original archive!
- Open `.../packages/elastic-second-node/config/elasticsearch.yaml` and uncomment/modify the line `node.name = elastic-second-node`
- Create an enrollment token for the second node.
  ```powershell
  cd .../packages/elasticsearch-8.14.3
  bin\elasticsearch-create-enrollment-token.bat --scope node
  # we take/copy the ENROLLMENT_TOKEN
  ```
- Go to `.../packages/elastic-second-node` and start a second `elasticsearch` with the enrollment token:
  ```powershell
  cd .../packages/elastic-second-node
  bin\elasticsearch --enrollment-token <ENROLLMENT_TOKEN>
  ```
- Go to Kibana/Web UI and check the cluster: now, we should have 2 nodes.
- We can kill a node simple with `Ctrl+C` in the Terminal where it is running. When that occurs, the clusters performs some house-keeping: [Delaying allocation when a node leaves](https://www.elastic.co/guide/en/elasticsearch/reference/current/delayed-allocation.html).

:warning: **IMPORTANT**: 

- This approach is only for development environments; in production, we need to perform further configurations.
- **I have not achieved to perform all these steps on a Windows machine:** I was getting a certificate error when running `bin\elasticsearch-create-enrollment-token.bat --scope node`.

I we don't add further nodes, we can continue, but the cluster and the shards/indices will be in "yellow" state, because they are not distributed across several nodes.

### Node Roles

Depending what they're used for, nodes can have different roles:

- Master: `node.master: true | false`. Resposible for performing cluster-wide operations, e.g., deleting indices, etc. For large projects, we should have dedicated master nodes.
- Data: `node.data: true | false`. Stores data and perform search queries.
- Ingest: `node.ingest: true | false`. Runs ingest pipelines, i.e., processing and adding a document to an index. Such a pipeline is like a simplified Logstash pipeline, where simple transformations are performed.
- Machine Learning:
  - `node.ingest: true | false`: to run ML jobs.
  - `xpack.ml.enabled: true | false`: enable/disable ML API for the node
- Coordination: Distribution of queries and aggregation of results; if a node has none of the previous nodes, it is a coordination node.
- Voting-only: `node.voting_only: true | false`. Rarely used; it votes which is the master node. It appears in very large projects.

We can see the roles when we run the command `GET /_cat/nodes?v`.

The field `node.role` in the response table contains the initial letters of each role assigned to each node.

![Nodes](./assets/nodes.png)

## Managing Documents

### Creating and Deleting Indices

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
```

### Indexing Documents = Adding Documents

```
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
```

### Retrieving Documents by ID

```
# Retrieve Document by ID
GET /products/_doc/100
```

We get:

```json
{
  "_index": "products",
  "_id": "100",
  "_version": 1,
  "_seq_no": 1,
  "_primary_term": 1,
  "found": true,
  "_source": {
    "name": "Toaster",
    "price": 49,
    "in_stock": 4
  }
}
```

If the `id=100` would not exist, we'd get `"found":false`.



## Mapping & Analysis

TBD.

:construction:

## Searching for Data

TBD.

:construction:

## Joining Queries

TBD.

:construction:

## Controlling Query Results

TBD.

:construction:

## Aggregations

TBD.

:construction:

## Improving Search Results

TBD.

:construction:

## Kibana

TBD.

:construction:

## Logstash

TBD.

:construction:

## License

This project is licensed under the Apache License 2.0. See the [LICENSE](./LICENSE) file for details.

