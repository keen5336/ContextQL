# ContextQL

**ContextQL** is a semantic query language for navigating contextual, relational, and pattern-indexed data. It's inspired by the shape of GraphQL, but built for systems where _meaning matters_—not just structure.

This repository contains:

- The **ContextQL** language specification and parser
- The **ContextDB** reference engine (Node.js) for resolving queries
- Examples, adapters, and the foundation for future modules

---

## 🔍 What Makes ContextQL Different?

ContextQL is built to power systems that:

- Model **relationships over time**
- Enforce **identity- and permission-aware access**
- Resolve meaning dynamically through **semantic patterns**
- Support AI agents, memory graphs, and dynamic inference

This isn’t just a query language—it’s a **contextual invocation layer** for intelligent systems.

---

## 📦 What’s Included

```
/packages
  /contextql       # Language parser and spec
  /contextdb       # Reference runtime (Node.js + Mongo + Redis)
  /adapters        # Backend connectors (e.g. memory, Mongo)
  /cli             # CLI tooling (optional)

/specs             # Grammar, semantics, and permission docs
/examples          # Real-world query examples
```

---

## 🧠 Philosophy

We believe querying should reflect **intent**, not just schema.
ContextQL aims to support:

- Declarative access to relational memory
- Composable, pattern-driven queries
- Seamless use across local, distributed, and agent-based systems

---

## 📖 License

This project is licensed under the [Elastic License 2.0](https://www.elastic.co/licensing/elastic-license).  
You are free to use, modify, and self-host the code, but **you may not offer it as a hosted service**.

For licensing questions, reach out to: `contact@yourdomain.dev`
