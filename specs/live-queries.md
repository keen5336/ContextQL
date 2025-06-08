

# ContextQL Live Queries Specification

This document defines the structure and behavior of live queries in ContextQL. Live queries allow agents or systems to subscribe to changes in the graph and receive real-time or buffered updates when data matching certain criteria is created, updated, or deleted.

---

## 📡 What is a Live Query?

A **live query** is a persistent subscription that watches for changes on nodes of a given type (or type lineage), filtered by conditions and scoped by identity.

It is used to:

- Push updates to users or agents
- Trigger internal workflows
- Monitor system state in real-time

---

## 🧱 Structure of a Live Query

```json
{
  "live": {
    "find": "Lead",
    "where": { "source": { "$eq": "facebook" } },
    "on": ["create", "update"],
    "return": ["name", "score"],
    "since": "2025-06-07T16:17:44Z"
  }
}
```

### Fields:

- `find`: Base type to watch (can be abstract or version-specific, e.g., `"Lead"` or `"Lead@1.1"`)
- `where`: Optional filter to apply to changes before notifying
- `on`: Array of triggers (`"create"`, `"update"`, `"delete"`)
- `return`: Fields to emit in the event payload
- `since`: Optional timestamp to backfill missed events on reconnect

---

## 🔁 Event Emission Model

Each time a matching mutation occurs, ContextQL:

1. Resolves the node’s `typeChain`
2. Checks for live queries subscribed to any of those types
3. Applies the `where` filter (if any)
4. Runs permission checks based on the identity context of the subscriber
5. Emits an event with the specified return fields

Example emitted payload:

```json
{
  "event": "update",
  "nodeType": "FacebookLead",
  "nodeId": "abc123",
  "data": {
    "name": "Alice",
    "score": 92
  },
  "triggeredBy": "mutation",
  "timestamp": "2025-06-07T16:22:45Z"
}
```

---

## ⛓️ Type Inheritance Resolution

Live queries respect the type hierarchy defined in `types.md`.

- Subscribing to `"Lead"` also captures changes to `"FacebookLead"`, `"InstagramLead"`, etc.
- The system uses `typeChain` to match derived types to parent subscriptions.
- Deprecated versions are still matched unless specifically excluded.

---

## 🧠 Reconnection and Backfill (`since`)

Live queries may include a `since` timestamp:

- On registration, the engine first runs a query for all changes since that time.
- This allows agents like Larry to catch up on missed updates.
- Only events the agent had permission to see at the time are returned.

The `since` field is optional and defaults to "now."

---

## 🪪 Identity and Permission Scope

All live queries are identity-bound:

- Listeners only receive events for data they can see
- Return fields are filtered based on current permission rules
- If the user or agent’s context changes, live query results may shift accordingly

---

## 🔒 Durability and Disconnect Behavior

Two modes:

- **Ephemeral**: tied to a socket or session, dropped on disconnect
- **Durable**: persists across sessions (for agents); buffered events are delivered on reconnect

The system may track `lastDelivered` timestamps for durable listeners and replay missed events automatically using `since`.

---

## ⚠️ Exclusions (By Design)

- No streaming diffs or field-level patches
- No aggregation or computed logic
- No wildcard type subscriptions (must specify a root type)
- No retroactive creation of live queries on past mutations

---

## 🧭 Design Intent

Live queries in ContextQL are designed for:

- Reliable agent orchestration
- Permissioned push updates
- Event-based automation
- Clean separation of type resolution, filtering, and delivery

They are not general-purpose websockets—they are declarative subscriptions tied to a graph-native semantic model.

---
This spec works in tandem with `types.md`, `permissions.md`, and `semantics.md` to provide a consistent model for streaming awareness in the ContextQL runtime.