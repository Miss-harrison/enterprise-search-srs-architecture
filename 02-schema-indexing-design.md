# 02. Search Schema & Indexing Strategy

## 1. Field Schema Specification
To prevent index bloat and keep memory utilization predictable, fields are strictly separated by whether they require full-text tokenization, exact-match filtering, or payload retrieval.

| Field Name | Solr Field Type | Indexed | Stored | DocValues | Purpose / Query Usage |
| :--- | :--- | :---: | :---: | :---: | :--- |
| `record_id` | `string` | Yes | Yes | Yes | Unique primary key for atomic updates & deduplication |
| `title` | `text_en_splitting` | Yes | Yes | No | Primary full-text search field (high boost weight `^3.0`) |
| `title_autocomplete` | `text_edge_ngram` | Yes | No | No | Dedicated index-only field for low-latency typeahead |
| `description` | `text_general` | Yes | No | No | Secondary keyword matching (`^1.0`); omitted from payload |
| `category_slug` | `string` | Yes | Yes | Yes | Exact-match filtering (`fq`) and hierarchical faceting |
| `price` | `pfloat` | Yes | Yes | Yes | Range queries (`fq=price:[10 TO 100]`) and price sorting |
| `in_stock` | `boolean` | Yes | Yes | Yes | High-reuse filter query (`fq=in_stock:true`) |
| `updated_at` | `pdate` | Yes | Yes | Yes | Delta-indexing watermark and recency boosting |

## 2. Indexing & Performance Optimization Decisions
* **Filter Queries (`fq`) vs. Main Query (`q`):** Non-scoring constraints (`category_slug`, `in_stock`, `price`) are routed exclusively through `fq` parameters so Solr caches bitsets independently of keyword relevance scoring.
* **DocValues Enforcement:** Enabled `docValues="true"` on all sorting and faceting fields to offload heap memory usage to OS disk cache and prevent JVM garbage collection spikes.
* **Payload Slimming (`fl`):** API consumers must explicitly pass the `fl` (field list) parameter so large unindexed metadata blobs are never serialized unnecessarily over the network.
* **Deep Paging Prevention:** Replaced high-offset `start` pagination with `cursorMark` tokens for batch analytics exports to prevent distributed shard sorting bottlenecks.
