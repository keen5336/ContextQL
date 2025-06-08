# ContextQL Permissions Specification

This document defines how permissions are applied to ContextQL queries. Every query is evaluated in the context of identity and access control—ensuring data visibility and operations are dynamically scoped to the caller's role, origin, and purpose.

## 🔐 Core Concepts

### Identity Context

Each query includes an implicit or explicit identity context:

```json
"context": {
  "userId": "abc123",
  "roles": ["admin", "editor"],
  "orgId": "org456"
}
```

- `userId`: The identity of the requester
- `roles`: One or more role tags used for access checks
- `orgId`: Optional organizational scoping

### Permission Application Phases

1. **Before resolution**: Filters out non-accessible nodes early
2. **During traversal**: Edge visibility is evaluated recursively
3. **Field-level masking**: Specific fields may be hidden based on role/context

## 🧠 Node-Level Access

Each node type can specify:

- Who can `read`, `write`, or `delete`
- Custom logic for derived visibility (`isOwner`, `createdBy`, etc.)

## 🔗 Edge Access Control

Edge relationships are permission-filtered like nodes:

- `canTraverse`: Who can see or follow this edge?
- `canMutate`: Who can create/remove this edge?

## 🧾 Field-Level Security

Fields may be:

- Fully visible
- Masked (e.g., null or redacted)
- Role-conditional

Example:

```json
"fields": {
  "email": {
    "visibleTo": ["admin", "self"]
  }
}
```

## 🎯 Policy Resolution Strategy

ContextQL uses a **deny-by-default** policy:

- If no explicit permission matches, access is denied
- Policies may be layered (global, org, user, node-type)


## ✂️ Excluded Behaviors (By Design)

The permissions system does not support:

- Field-level overrides inside queries (e.g. requesting masked fields)
- Permission overrides or impersonation through context payloads
- Dynamic policy redefinition within queries
- Role escalation or inference

All permission decisions are resolved strictly at query evaluation time based on static policy and provided identity context.

## 🧭 Design Intent

ContextQL treats permission enforcement as part of the execution substrate—not an external module. All queries are evaluated with explicit, immutable identity context and follow a deny-by-default resolution model.

Permissioning is not a plugin—it is fundamental to how the graph is read, traversed, and manipulated. This ensures predictable data boundaries, agent safety, and long-term structural integrity.

## 🧠 Future Ideas

- Fine-grained query tracing
- Field masking reasons (for UI)
- Consent/audit tagging
- Conditional edge traversal

---

This file is part of the core enforcement mechanism of the ContextQL engine and works in tandem with `semantics.md` and `syntax.md`.
