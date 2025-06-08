

# Story 1: Evaluate a Simple Query with Identity Context

**Story ID**: `runtime-tests/query/eval-basic`  
**Title**: Implement a Runtime Test to Evaluate a Basic `find` Query Against a Mock Graph Store

---

## 📘 Description

Establish the first functional test to validate the ContextQL runtime's ability to evaluate a simple `find` query. This story will act as a proof of life for the engine pipeline—from receiving a request, resolving permission context, querying in-memory graph data, and returning filtered results.

This test is foundational. All query logic (filtering, permissions, field visibility) builds from this structure.

---

## 📂 File Structure Setup

Create the following directory and test file:

```
tests/
└── runtime/
    └── query-basic.test.ts
```

In this test file, simulate a minimal runtime that includes:

- A basic `executeQuery()` function (temporary or imported)
- An in-memory graph of mock nodes
- Permission context injected directly

---

## 📁 Example In-Memory Node Store (mock)

Define this inside the test file or import from a `mockGraphStore.ts` file:

```ts
const mockNodes = [
  {
    id: "n1",
    type: "Note",
    title: "Hello",
    content: "Top secret",
    typeChain: ["Note"],
    permissions: {
      read: ["reader"]
    }
  },
  {
    id: "n2",
    type: "Note",
    title: "World",
    content: "Private stuff",
    typeChain: ["Note"],
    permissions: {
      read: ["admin"]
    }
  }
];
```

---

## 🧪 Test Case: `returns a filtered node result`

```ts
test("returns only allowed node with matching title", async () => {
  const result = await executeQuery({
    context: {
      userId: "u1",
      roles: ["reader"]
    },
    query: {
      find: "Note",
      where: { title: { $eq: "Hello" } },
      return: ["title"]
    }
  });

  expect(result).toEqual([
    {
      title: "Hello"
    }
  ]);
});
```

---

## 🔧 Implementation Guidance (for Codex)

If `executeQuery()` does not exist yet, it should be implemented as a minimal function within the test file. For now, the goal is to:

- Iterate over `mockNodes`
- Match `typeChain` against `query.find`
- Apply `$eq` filter to `title`
- Check `context.roles` against `permissions.read`
- Return only requested fields

---

## ✅ Acceptance Criteria

- [ ] `tests/runtime/query-basic.test.ts` exists and runs
- [ ] Query with valid `find`, `where`, and `return` structure is parsed
- [ ] Nodes outside permission scope are excluded
- [ ] Fields not in `return` are not included in response
- [ ] Result is shaped as defined in `runtime-contracts.md`


---

# Story 2: Create Runtime Test Harness for Query Evaluation

**Story ID**: `runtime-tests/harness/init`  
**Title**: Build a Minimal Test Harness for Executing ContextQL Queries in Isolation

---

## 📘 Description

Establish a foundational test harness that enables isolated, testable execution of ContextQL queries against an in-memory dataset. This harness is necessary for validating runtime behavior without introducing external dependencies or system integration concerns.

This is not the engine itself—it is the controlled execution environment used by tests like those described in Story 1.

---

## 📂 File Structure Setup

Create the following directories and files:

```
src/
└── test-runtime/
    ├── executeQuery.ts        # test-only query evaluator stub
    └── mockGraphStore.ts      # in-memory mock node store

tests/
└── runtime/
    └── query-basic.test.ts    # existing or initial test file
```

The test harness lives under `src/test-runtime/` and is NOT intended for production. It enables simulated identity context, static node data, and permission-checked evaluation for query development.

---

## 🔧 Harness Implementation Requirements

Implement the following test-facing utility function:

```ts
// src/test-runtime/executeQuery.ts
export async function executeQuery(input: ContextQLQuery): Promise<any> {
  // Placeholder logic (expand in future tasks)
}
```

This function should:

- Accept a `ContextQLQuery` object
- Read from an in-memory node list (via `mockGraphStore.ts`)
- Filter based on `find` and `where` clauses
- Apply permission checks using the provided `context`
- Return field-filtered results matching the `return` clause

---

## 📁 Example Mock Store Format

```ts
// src/test-runtime/mockGraphStore.ts
export const mockNodes = [
  {
    id: "n1",
    type: "Note",
    title: "Hello",
    content: "Top secret",
    typeChain: ["Note"],
    permissions: {
      read: ["reader"]
    }
  },
  {
    id: "n2",
    type: "Note",
    title: "World",
    content: "Private stuff",
    typeChain: ["Note"],
    permissions: {
      read: ["admin"]
    }
  }
];
```

---

## ✅ Acceptance Criteria

- [ ] A file named `executeQuery.ts` exists under `src/test-runtime/`
- [ ] The function can process a minimal `ContextQLQuery` object and return valid results
- [ ] A mock node store is accessible via `mockGraphStore.ts`
- [ ] Test files like `query-basic.test.ts` can import `executeQuery()` and run in isolation
- [ ] No external systems (e.g., database, socket, server) are required for tests to pass
- [ ] Implementation is minimal and expected to evolve in future stories
# Story 3: Implement Query Execution Logic for Mock Store

**Story ID**: `runtime-engine/query/basic-resolver`  
**Title**: Implement Minimal Query Resolution Against In-Memory Graph Data

---

## 📘 Description

This story introduces the first working logic to resolve and execute queries passed into the `executeQuery()` function created in Story 2. It provides basic support for field filtering, type matching, and permission validation, using the mock store format defined in the test harness.

This logic forms the foundation of the ContextQL engine. It is intended to be simple, predictable, and purely functional.

---

## 📂 Target File

```
src/
└── test-runtime/
    └── executeQuery.ts
```

---

## 🔧 Implementation Requirements

Update `executeQuery(input: ContextQLQuery)` to perform the following steps in order:

1. **Extract query fields**:
   - `find`, `where`, `return`
2. **Filter nodes from `mockGraphStore.ts`**:
   - Must include `query.find` in the node’s `typeChain`
3. **Apply `where` filter**:
   - Implement `$eq` for scalar fields only
   - Other operators may be stubbed
4. **Run permission check**:
   - Include only nodes where `context.roles` intersect `node.permissions.read`
5. **Shape result**:
   - Only include fields listed in `return`

---

## 📁 Example Implementation Snippet

```ts
import { mockNodes } from "./mockGraphStore";

export async function executeQuery(input: ContextQLQuery) {
  const { context, query } = input;
  const { find, where, return: returnFields } = query;

  return mockNodes
    .filter(node => node.typeChain.includes(find))
    .filter(node => {
      if (!where) return true;
      return Object.entries(where).every(([field, condition]) => {
        if ("$eq" in condition) return node[field] === condition["$eq"];
        return false; // stub for unsupported ops
      });
    })
    .filter(node => {
      const allowed = node.permissions?.read ?? [];
      return context.roles?.some(role => allowed.includes(role));
    })
    .map(node => {
      const result: Record<string, any> = {};
      for (const field of returnFields || []) {
        result[field] = node[field];
      }
      return result;
    });
}
```

---

## ✅ Acceptance Criteria

- [ ] `executeQuery.ts` resolves queries using mock data
- [ ] Results are permission-filtered based on `context.roles`
- [ ] `$eq` filter is implemented for scalars
- [ ] Fields returned match `return` clause exactly
- [ ] Function passes all cases in `query-basic.test.ts`
- [ ] No side effects, network calls, or mutations occur