

# ContextQL Runtime Contracts

This document defines the boundary interface between the ContextQL engine and external callers—whether human, agent, or system. It specifies the request/response format, error handling, and lifecycle responsibilities of a compliant runtime.

These contracts are the low-level system interfaces for executing ContextQL operations. All query and mutation structures must conform to the identity-bound semantics defined in `permissions.md`.

---

## 🔁 Query Contract

Defines how a standard graph query is submitted to the runtime.

```ts
type ContextQLQuery = {
  context: {
    userId: string;
    roles?: string[];
    orgId?: string;
  };
  query: {
    find: string;                  // Node type to query (e.g. "Note")
    where?: Record<string, any>;   // Optional filter expression
    return?: string[];             // Fields to return (required for visibility)
    expand?: Record<string, any>;  // Optional edge traversal
  };
};
```

### Returns:

A list of nodes, each permission-filtered according to `context`.

```ts
type QueryResult = Array<Record<string, any>>;
```

---

## ✏️ Mutation Contract

Mutations allow explicit state changes to the graph.

```ts
type ContextQLMutation = {
  context: {
    userId: string;
    roles?: string[];
    orgId?: string;
  };
  mutation: {
    create?: { type: string; data: Record<string, any> };
    update?: { id: string; data: Record<string, any> };
    link?: { from: string; edge: string; to: string };
    unlink?: { from: string; edge: string; to: string };
    delete?: { id: string };
    return?: "id" | string[];
  };
};
```

### Returns:

```ts
type MutationResult = {
  success: boolean;
  node?: Record<string, any>; // Permission-filtered node if returned
  id?: string;                // Returned if "id" is requested
  error?: string;             // Optional structured error message
};
```

Mutation returns the post-operation view of the node, shaped and masked by permission context.

---

## 📡 Live Query Contract

Live queries are persistent subscriptions to graph state changes.

```ts
type LiveQueryRegistration = {
  context: {
    userId: string;
    roles?: string[];
    orgId?: string;
  };
  live: {
    find: string;                         // Root type (e.g. "Lead")
    where?: Record<string, any>;         // Optional filter
    on: ("create" | "update" | "delete")[];  // Trigger events
    return: string[];                    // Fields to emit
    since?: string;                      // ISO timestamp for backfill
  };
};
```

### Emits:

```ts
type LiveQueryEvent = {
  event: "create" | "update" | "delete";
  nodeType: string;
  nodeId: string;
  data: Record<string, any>;
  triggeredBy: "mutation" | "system";
  timestamp: string;
};
```

Live queries are permission-scoped and emit only what the subscriber is authorized to see.

---

## 🧭 Contract Principles

- All inputs must include `context` for permission evaluation.
- No implicit behavior is permitted (e.g. no upserts, no inferred fields).
- Contracts are transport-agnostic (HTTP, socket, embedded runtime).
- Permission logic is enforced strictly and must align with `permissions.md`.

This file defines the core developer-facing and agent-facing interface for ContextQL runtime execution.