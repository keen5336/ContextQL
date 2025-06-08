# ContextQL Vision

ContextQL is more than a query language—it's a foundational engine for managing deeply entangled, semantically rich data with identity-aware access. It’s designed to replace traditional document stores (like MongoDB) and key-value caches (like Redis) with a unified, graph-native context engine.

## 🌱 Why ContextQL?

Modern systems—especially those involving agents, automation, or human workflows—require:

- Flexible relationships, not rigid schemas
- Identity-native permissioning, not bolt-on ACLs
- Temporal and contextual awareness
- Structured introspection and programmatic adaptability

ContextQL is built to meet those needs, while maintaining human-readable and agent-composable syntax.

## 🔍 Core Design Pillars

### 1. **Graph-first, not table-first**

- Native support for nodes, edges, and semantic links
- Recursive queries and pattern-based traversal

### 2. **Identity as a first-class primitive**

- Every query executes within a permission context
- Fine-grained control down to node, edge, or field

### 3. **Contextual by design**

- Supports evolving states, roles, and intent-aware access
- Queries shaped by who, when, where, and why

### 4. **Composable and modular**

- Queries as JSON makes them easy to compose, inspect, and manipulate
- Fits naturally into pipelines, AI agents, and automation

### 5. **Self-hosted and scalable**

- Lightweight reference engine in Node.js (Mongo/Redis-backed)
- Planned native engine in Rust with local-first + distributed support

## 🎯 Use Cases

- AI agent memory graphs
- CRM histories and task threading
- Dynamic user-generated systems (wikis, workflows, notes)
- Federated knowledge graphs
- Automation backends with permission-checked logic

## 🔭 Long-Term Goals

- Rust-native core engine with pluggable storage
- Streaming and live query support
- Built-in reasoning and inference for graph traversal
- Open ecosystem with plugin/query extensions
- Lightweight CLI + local daemon support

## 📦 JSON, Compression, and Transport Efficiency

ContextQL uses JSON not only for readability and agent composability, but also because—when compressed—it performs competitively with binary formats.

JSON is:

- Highly compressible (70–90% size reduction with Gzip/Brotli)
- UTF-8 friendly and widely supported
- Human-readable and AI-operable
- Structurally repetitive, making it ideal for wire compression

This makes JSON the optimal transport and query format for both developer experience and performance.

Binary formats offer marginal gains and are not necessary at the protocol layer.

## 🧬 Binary Encoding Policy

ContextQL does not support binary encoding in external queries or data payloads.

If binary data is required in node content, it must be base64-encoded and treated as opaque by the engine.

Internal runtimes (e.g. future Rust implementations) may use binary formats for storage or transport—but these optimizations:

- Will not affect external behavior
- Will never surface in the API
- Will not influence query semantics or permission logic

The external protocol remains JSON by design—durable, transparent, and portable.
