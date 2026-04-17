# 🧠 AI System Design Interview Prep

Welcome to the definitive **AI System Design** repository. This collection of notes is specifically curated for tackling senior architecture interviews with a focus on modern AI pipelines, massive media processing, asynchronous workflows, and generative LLM orchestration.

Unlike standard system design (which often focuses heavily on traditional CRUD apps), these problems force you to tackle the realities of AI: handling GPU timeouts, embedding vectors, Context Window starvation, and non-deterministic logic flow.

---

## 📚 Problem Set Directory

Each guide is structured as a **Teaching Note**, walking you through the High-Level Design (HLD) logic, abstract schema structures, and the raw thought-processes required to survive edge-case probing.

| Module | Core Focus | Concepts Mastered |
| :--- | :--- | :--- |
| **[Q1. AI Video Generation Platform](./Q1_AI_Video_Gen.md)** | Orchestrating multi-scene generative AI | DAGs, Scatter-Gather fan-outs, Semantic Caching |
| **[Q2. AI Audio Chat System](./Q2_AI_Audio_Chat.md)** | Long-form audio intelligence + RAG | Chunking strategies, Vector DB execution, Isolation |
| **[Q3. WhatsApp AI Chat System](./Q3_WhatsApp_AI_Chat.md)** | High-throughput bi-directional routing | Cassandra partitioning, Denormalization, AI Ghost-users |
| **[Q4. AI Video Editing Pipeline](./Q4_AI_Video_Editing.md)** | Processing brutal 20GB+ media architectures | FFmpeg chunk rendering, Metadata/Pixel isolation |
| **[Q5. Multi-Modal Search System](./Q5_Multi_Modal_Search.md)** | Fusing Text, Audio, and PDF intelligence | Reciprocal Rank Fusion, pgvector vs Pinecone |
| **[Q6. Async AI Orchestration](./Q6_Async_Job_Orchestration.md)** | The backbone job queue system | Idempotent row-locking, Dead-Letter analytics |
| **[Q7. AI Content Versioning](./Q7_AI_Content_Versioning.md)** | Git for Generative AI (Tree revisions) | S3 Garbage Collection, Delta branches, Rollbacks |

---

## 🛠️ What's Inside Each Note?

Every scenario follows a rigorous, interview-grade blueprint:
1. **The Problem & Requirements:** The exact prompt you are handed.
2. **Abstract Schema Design:** Key relational and NoSQL entity fields required to hold the architecture together.
3. **High-Level Design (HLD) & Mermaid Diagrams:** Visual mappings of component routing. 
4. **Teaching Walkthroughs:** Step-by-step narrative explanations of how data flows from A to B.
5. **LLD, Failures & Trade-offs:** The toughest part of the interview. How do we handle `Out of Memory` errors, `At-Least-Once` queue duplicates, or `Rate Limited` third-party providers?
6. **5 SQL Follow-Up Queries:** Advanced SQL queries (including `pgvector` math, CTE graph traversals, and Pessimistic Locking) designed to show extreme database competence.

---

## 💡 Key Architectural Themes Covered

As you read through these notes, you will see a recurrence of several battle-tested AI design patterns:

* **Retrieval-Augmented Generation (RAG)** vs Token Window limits.
* **Idempotency** guarantees when scaling spot-instances horizontally.
* **Vector Mathematics** (Cosine Similarity native inside SQL via `pgvector`).
* **Storage Tiering** (Moving dead generational assets to Glacier).
* **Asynchronous WebSockets & Pub/Sub** to achieve true typing-like UI without thrashing databases.

---
> *"Compute is expensive. Metadata is practically free."* — Principle of AI Media Design



<script type="module">
  import mermaid from 'https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.esm.min.mjs';
  mermaid.initialize({ startOnLoad: false });
  document.addEventListener("DOMContentLoaded", function() {
    const blocks = document.querySelectorAll('pre code.language-mermaid');
    blocks.forEach(function(block) {
      const div = document.createElement('div');
      div.className = 'mermaid';
      div.textContent = block.textContent;
      const parent = block.closest('.highlighter-rouge') || block.closest('pre');
      if (parent) {
        parent.replaceWith(div);
      }
    });
    mermaid.run();
  });
</script>
