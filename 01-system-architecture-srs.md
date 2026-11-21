# 01. System Architecture & Software Requirements Specification (SRS)

## 1. Project Objective
Design a scalable, fault-tolerant search and analytics layer capable of indexing **2M+ product and content records** while maintaining sub-200ms query response times across web and mobile clients.

## 2. High-Level Architecture Flow
```text
[Primary DB / CMS] 
       │
       ▼ (Delta & Full Ingestion Pipelines)
[ETL / Data Normalization Layer]
       │
       ▼ (JSON / HTTP Indexing API)
[Apache Solr Cluster (SolrCloud + ZooKeeper Ensemble)]
  ├── Collection: catalog_prod (Sharded + Replicated)
  ├── Filter Cache (fq) & Query Result Cache
  └── Custom Synonym & Stopword Dictionaries
       │
       ▼ (REST / JSON Search API)
[Frontend Search UI: Autocomplete, Facets, Relevance Ranking]
```

## 3. Functional Requirements
* **FR-01 Full-Text & Fuzzy Search:** Support multi-field keyword matching with configurable edit-distance tolerance (`~1` / `~2`) for user typos.
* **FR-02 Dynamic Faceted Navigation:** Return real-time facet counts for category, brand, availability, and price ranges without degrading main query throughput.
* **FR-03 Autocomplete / Typeahead:** Deliver prefix-matched suggestions within **< 80ms** after the 2nd keystroke using edge n-gram tokenization.
* **FR-04 Business Relevance Boosting:** Apply query-time boost weights (`qf` / `bq`) prioritizing in-stock items, exact title matches, and high-conversion tags.

## 4. Non-Functional Requirements (SLAs)
* **NFR-01 Query Latency:** 95th percentile (P95) search API response time must remain **≤ 180ms** under 500 concurrent requests/sec.
* **NFR-02 Indexing Freshness:** Incremental catalog updates (price/stock changes) must reflect in live search results within **≤ 60 seconds** via soft-commit tuning.
* **NFR-03 High Availability:** Zero-downtime deployments and automatic failover managed via multi-node SolrCloud replication and ZooKeeper coordination.
