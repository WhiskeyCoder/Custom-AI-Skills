# OpenSearch Data Skill

## Overview

This skill provides read access to an OpenSearch cluster for querying structured data, analytical outputs, signal graphs, and historical archives.

It enables AI agents to retrieve, filter, and process large-scale datasets for analysis, reporting, and automation workflows.

---

## OpenSearch Instance

* **URL:** `http://<HOST>:9200`
* **Kibana UI:** `http://<HOST>:5601`
* **Version:** `OpenSearch 2.x`
* **Authentication:** Configurable (may be disabled for local environments)

---

## Example Index Types

| Index                | Type       | Description                          |
| -------------------- | ---------- | ------------------------------------ |
| `events`             | Real-time  | Time-sensitive event data            |
| `collections`        | Aggregated | Collected data from external sources |
| `security`          | logs | securitylogs               |
| `manaul_reports`   | Reports    | manually submitted data            |
| `alerts`             | Monitoring | Alert-based outputs                  |

> Note: Actual index names and schemas may vary depending on implementation.

---

## Usage

### Basic Search

```bash id="q1r9xk"
curl -s -X GET "http://<HOST>:9200/{index}/_search" \
  -H "Content-Type: application/json" \
  -d '{
    "query": { "match_all": {} },
    "size": 10
  }'
```

---

### Full-Text Search

```bash id="u1v6pw"
curl -s -X GET "http://<HOST>:9200/{index}/_search" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "multi_match": {
        "query": "example query",
        "fields": ["title", "content", "description"]
      }
    },
    "size": 10
  }'
```

---

### Date Range Query

```bash id="h4mz6b"
curl -s -X GET "http://<HOST>:9200/{index}/_search" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "range": {
        "timestamp": {
          "gte": "2026-01-01",
          "lte": "2026-01-31"
        }
      }
    }
  }'
```

---

### Filtered Query

```bash id="t0y6e2"
curl -s -X GET "http://<HOST>:9200/{index}/_search" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "bool": {
        "must": [
          { "multi_match": { "query": "example", "fields": ["title", "content"] } }
        ],
        "filter": [
          { "term": { "source": "api_feed" } }
        ]
      }
    }
  }'
```

---

### High-Confidence Filtering

```bash id="z8k2dl"
curl -s -X GET "http://<HOST>:9200/{index}/_search" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "bool": {
        "filter": [
          { "range": { "confidence_score": { "gte": 70 } } }
        ]
      }
    }
  }'
```

---

### Multi-Index Search

```bash id="n7xkq1"
curl -s -X GET "http://<HOST>:9200/index1,index2/_search" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "multi_match": {
        "query": "example",
        "fields": ["title", "content"]
      }
    }
  }'
```

---

## Result Processing

### Example Parsing

```bash id="a9r4y2"
curl -s -X GET "http://<HOST>:9200/{index}/_search" \
  -H "Content-Type: application/json" \
  -d '{ "query": { "match_all": {} }, "size": 5 }' \
| python3 -c "
import sys, json
data = json.load(sys.stdin)
for hit in data['hits']['hits']:
    src = hit['_source']
    print(src.get('title', 'N/A'))
"
```

---

## Data Handling Guidelines

### Confidence Levels

* **High confidence:** ≥ 70 → Suitable for direct use
* **Medium confidence:** ≥ 40 → Requires validation
* **Low confidence:** < 40 → Treat as unverified

---

### Prioritisation

* Time-sensitive or flagged items should be prioritised
* Recent data typically holds higher operational relevance
* Historical data should be filtered by date range to improve performance

---

## Error Handling

* **Index not found:** Verify index name
* **Empty results:** Check field names and query structure
* **Timeouts:** Reduce query size or add filters
* **Large datasets:** Use pagination (`search_after` or `scroll`)
* **Unavailable indices:** Some clusters may have partial availability depending on configuration

---

## Best Practices

* Always limit result size (`size`)
* Use filters instead of broad queries where possible
* Avoid full-index scans on large datasets
* Preserve metadata for traceability
* Combine multiple queries for deeper analysis
* Validate data before downstream use

---

## Notes for AI / Agent Usage

* Prefer targeted queries over broad searches
* Always handle missing fields gracefully
* Maintain source attribution where available
* Use confidence scoring to guide decision-making
* Structure outputs for downstream processing or reporting

---

## Philosophy

Efficient data retrieval is the foundation of reliable usage.

Query smart. Filter aggressively. Extract only what matters.

---
