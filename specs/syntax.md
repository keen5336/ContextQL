# ContextQL Syntax Specification

This document defines the structural grammar of ContextQL queries using a JSON-based format. The goal is to keep the syntax expressive, composable, and familiar—drawing inspiration from MongoDB, GraphQL, and Cypher where appropriate.

## 🎯 Overview

ContextQL queries are JSON objects that describe what to find, how to filter it, and what to return. Queries support deeply nested relationships and contextual resolution.

## 🧱 Basic Query Structure

```json
{
  "find": "NodeType",
  "where": {
    "field": { "$op": "value" }
  },
  "return": ["fieldA", "fieldB"]
}
```

### Top-level fields:
- `find` (string): The type of node you are querying.
- `where` (object): Filters using operators like `$eq`, `$gt`, `$in`, etc.
- `return` (array): Fields or relationships to return.

## 🔍 Filter Operators

| Operator | Description             |
|----------|-------------------------|
| `$eq`    | Equals                  |
| `$ne`    | Not equal               |
| `$gt`    | Greater than            |
| `$lt`    | Less than               |
| `$in`    | In array                |
| `$and`   | Logical AND (array)     |
| `$or`    | Logical OR (array)      |

## 🔗 Relationship Expansion

```json
{
  "find": "Person",
  "where": { "name": { "$eq": "Gema" } },
  "expand": {
    "partner": {
      "return": ["name", "age"]
    }
  }
}
```

- `expand`: Defines related nodes to traverse and what fields to return within them.

## 🧠 Future Additions (Strategically Constrained)

ContextQL follows a minimal-instruction-set philosophy (RISC-like). Features are only added if they directly improve clarity, composability, or performance for identity-aware graph queries.

### Planned

- **Pagination Controls**
  - `limit`: Cap the number of results returned
  - `offset`: Skip N results (for paged queries)

- **Field-Based Sorting**
  - `sort`: Allow ordering by scalar fields like `createdAt`, `score`, etc.
  - No computed expressions, dynamic prioritization, or logic-based sorting

- **Mutations**
  - Declarative structure for creating/updating/deleting nodes and edges
  - Fully permissioned like reads

### Will Not Implement (By Design)

- **Field Projection or Aliasing**
  - No support for renaming, reshaping, or aliasing fields within queries
  - Returned data is raw—clients or agents handle formatting

- **Variable Binding**
  - No symbolic references or reusable fragments inside a query
  - Encourages direct expression and avoids abstract naming indirection

- **Computed Sort Logic**
  - No sorting by derived values or runtime expressions
  - Only fixed field-based sorting is supported

- **Aggregation Pipelines**
  - No group-by, aggregation, or reduce operations
  - Result shaping is expected to happen post-query via agent logic

### Design Rationale

ContextQL is deliberately transparent and composable. Its syntax favors direct expression over abstraction.  
Rather than mimic developer convenience languages, it provides a consistent execution substrate—leaving naming, shaping, and transformation to agents, clients, or macros.

---
ContextQL offers durable primitives. Everything else is optional layering.
