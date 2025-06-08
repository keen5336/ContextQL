

# ContextQL Type System Specification

This document defines the structure, behavior, and versioning model for types in ContextQL. Types are immutable, semantically rich definitions used to structure nodes within the graph. All types are identity-scoped and permission-aware.

---

## 📦 Type Structure

Each type in ContextQL is defined by a schema object that includes:

```json
{
  "type": "FacebookLead",
  "version": "1.0",
  "inherits": ["Lead@1.0", "Contact", "Person"],
  "status": "active",
  "createdBy": "user_abc",
  "createdForOrg": "org_xyz",
  "currentOwner": "org_xyz",
  "permissions": {
    "read": ["org_member"],
    "extend": ["schema_editor"],
    "useAsBase": ["org_xyz"]
  }
}
```

### Required Fields

- `type`: Unique name of the type (case-sensitive).
- `version`: Semantic version identifier, immutable.
- `inherits`: Ordered list of supertypes (each with optional version tag).
- `status`: One of `active`, `deprecated`, `locked`, or `archived`.

### Optional Fields

- `deprecatedBy`: Reference to the type that supersedes this one.
- `createdBy`, `createdForOrg`, `currentOwner`: Identity and ownership tracking.
- `permissions`: Schema-level access control.

---

## ⛓️ Inheritance and Type Chains

Each type builds a flattened `typeChain`, resolving all ancestors in order. For example:

```json
"FacebookLead": {
  "typeChain": ["FacebookLead", "Lead@1.0", "Contact", "Person"]
}
```

Types do not retroactively inherit from new versions of their parents. If `Lead@1.1` is introduced, it does not alter types that inherited from `Lead@1.0`.

---

## 🧬 Versioning and Deprecation

Types are **immutable**. Once a type is created, it cannot be modified or deleted.

- New behavior or structure requires a new version.
- Versions are linked via `deprecatedBy`.
- Deprecation removes a type from use, but not from history.
- Archived types may not be referenced or extended.

Example of type evolution:

```plaintext
Lead@1.0
 ├─ FacebookLead
 └─ EmailLead

Lead@1.1
 └─ InstagramLead
```

`FacebookLead` and `EmailLead` continue to inherit from `Lead@1.0`. `InstagramLead` inherits from `Lead@1.1`.

---

## 🔍 Querying by Type

Queries and live queries can match:

- A specific version: `"find": "Lead@1.1"`
- A general type across versions: `"find": "Lead"` matches all inheritors of any version
- Any node whose `typeChain` contains the desired type

This enables agents to subscribe broadly or narrowly based on schema lineage.

---

## 🧠 Design Intent

The type system in ContextQL is designed to:

- Prevent schema ambiguity over time
- Enable stable agent behavior and memory replay
- Support audit-critical workflows
- Avoid broken inheritance chains and orphan types
- Maintain extensibility without allowing retroactive mutation

Types are not simply labels—they are durable contracts.
