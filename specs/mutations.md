

# ContextQL Mutation Specification

This document defines how data is created, updated, or deleted within ContextQL. Mutations operate under strict permission rules and are always executed within an identity context. While `syntax.md` covers formatting, this file specifies semantic behavior and intent.

---

## 📜 1. Purpose of Mutations

Mutations are privileged operations that alter the structure or content of the graph. Each mutation is identity-bound and permission-scoped. There is no implicit inference, transformation, or fallback behavior. All mutations are explicit.

ContextQL mutations are designed for use by AI agents, automation layers, and permissioned systems—not traditional developer convenience.

---

## 🔧 2. Mutation Structure

Each mutation is a declarative object with one of the following top-level keys:

- `create`: Add a new node
- `update`: Modify fields on an existing node
- `link`: Create an edge between two nodes
- `unlink`: Remove an edge
- `delete`: Remove a node (if permitted)

Examples:

```json
{ "create": { "type": "Note", "data": { "title": "Hello" } } }

{ "update": { "id": "abc123", "data": { "title": "Updated Title" } } }

{ "link": { "from": "user123", "edge": "owns", "to": "note456" } }
```

Each operation must be explicit. No chaining or fallback is permitted.

---

## ⚙️ 3. Execution Semantics

- Each mutation is atomic: partial failures do not yield partial success.
- Mutations are identity-scoped and permission-checked.
- No mutation can affect nodes or edges beyond its explicit declaration.
- There is no support for inferred cascading, computed logic, or procedural chaining.

---

## 🧾 4. Permission Enforcement

All mutations are subject to permission enforcement as defined in `permissions.md`.

- Nodes require `create`, `update`, or `delete` permission
- Edges require `canTraverse` and `canMutate` permission
- Fields may be write-restricted on a per-role basis

If a mutation violates any permission rule, it fails silently or with a structured error depending on runtime policy.

---

## 📤 5. Mutation Return Behavior

By default, all mutations return the resulting node(s), filtered by permission context, as if queried immediately after mutation.

This enables AI agents to operate efficiently without redundant follow-up queries.

Clients may optionally suppress this behavior:

```json
"return": "id"
```

This returns only the node ID(s) involved.

Additional return modes (e.g. `false`, field-specific) may be considered in future versions.

---

## 🚫 6. Excluded Behaviors (By Design)

ContextQL mutations do not support:

- Upserts or implicit creation-on-update
- Cascading deletes or inferred edge cleanups
- Computed fields or transformation logic
- Mutation hooks, macros, or reactivity
- Field aliasing or shape transformation on return

All shaping and behavior chaining must occur at the agent or application layer.

---

## 🧭 7. Design Intent

Mutations in ContextQL are low-level graph operations—not developer-friendly abstractions.

They are precise, permissioned, and explicitly scoped. The goal is to maximize transparency and structural integrity, especially for agent-based systems and audit-critical workflows.

No behavior is implicit. Every change must be fully declared, validated, and authorized.