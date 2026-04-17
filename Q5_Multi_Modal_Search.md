# Q5. Multi-Modal Content Search System

## 1. Problem Statement
Users upload documents (text, audio, video) and want to search across all content using natural language.

## 2. Requirements
1. Ingest multiple content types (PDF, audio, video).
2. Extract text from each source.
3. Generate embeddings and store them.
4. Allow semantic search queries.
5. Return relevant results with context.
6. Support hybrid search (keyword + semantic).

## 3. Follow-up Questions
* How will you design schema for different content types?
* How will you store and query embeddings?
* How do you handle large documents?
* How do you rank results?

---

## 4. Schema Design (Fields)

* **`KnowledgeSources`**: `id`, `user_id`, `media_category` (audio, pdf, video), `external_storage_url`
* **`UniversalTextBlocks`**: `id`, `source_id`, `chunk_index_ordering`, `raw_text`, `display_time_or_page`
* **`VectorIndices`** (If PgVector): `id`, `text_block_id`, `tenant_id`, `embedding_data` (VECTOR(1536))

---

## 5. High-Level Design (HLD) & Explanatory Walkthrough

```mermaid
graph TD
    Client -->|Search Qty: 'Revenue Plan'| API
    
    API -->|Vectorize Prompt| EmbedModel[Embedding AI]
    EmbedModel -->|Float Vector '[0.1, 0.4]'| Postgres[(PostgreSQL DB)]
    
    Postgres -->|1. Keyword Match BM25| KeywordEngine[Native Postgres FTS]
    Postgres -->|2. Semantic Match| VectorEngine[Pinecone / Native pgvector]
    
    KeywordEngine -->|List A| Ranker[SQL View - RRF Fusion Alg]
    VectorEngine -->|List B| Ranker
    
    Ranker -->|Top 10 Resolved Results| Client
```

### Explanatory Walkthrough (Teaching Notes)
When dealing with multi-modal content, the foundational trick is normalizing every single file type down into one unified "textual block" sequence. The search engine doesn't care if it's reading an mp3 or a pdf—it only queries text chunks.

1. **Normalizing the Chaos (Ingestion)**: A `Header Detection lambda` determines if an upload is a video or a PDF. Videos get mapped to Whisper STT. PDFs go straight into PyMuPDF parsers. Both distinct engines map their outputs to one identical table: `UniversalTextBlocks.`

2. **Generating Embeddings for Large Documents**: A 200-page PDF string destroys AI API limits. The background worker chops it into **Chunks** (e.g., 512 tokens with a 50-token overlap). Each chunk generates a floating-point embedding payload. 

3. **The Hybrid Search Magic**: Vector Search knows that "Canine" equals "Dog". Keyword Search knows that finding exactly "Invoice 894AZB" is critical. We execute ONE unified query that runs keyword search and nearest neighbor math, using Reciprocal Rank Fusion to blend them into the ultimate answer.

---

## 6. LLD, Thought Process & Failure Handling

* **PgVector vs Dedicated Vector DBs (Pinecone/Milvus)**: 
  While `pgvector` inside PostgreSQL allows executing elegant hybrid SQL queries natively, at massive scale (billions of embeddings), Postgres RAM limitations will bottleneck heavily. An enterprise system often maintains PostgreSQL explicitly for relational state, while pumping vectors into a dedicated external Pinecone cluster. Pinecone builds highly-optimized ANN (Approximate Nearest Neighbor) indices independently, preventing heavy vector math from blocking your transactional Postgres engine.

* **Ranking Results (Reciprocal Rank Fusion)**:
  RRF is a mathematical formula. If Document A is ranked #1 by Keyword and #10 by Semantic, RRF scores them mathematically to find the best consensus match without requiring external complicated Machine Learning rerankers. Doing this via SQL Common-Table-Expressions (CTEs) is incredibly efficient.

* **Corrupted or Unreadable Data Blocks**:
  Scanning complex messy PDFs might cause Python parsers to output random garbage unicode blocks. If a chunk has 80% non-ascii symbols, we silently drop the chunk and label it a bad read, preventing it from polluting the Vector DB table.

---

## 7. Follow-up SQL Queries

**1. Heavy LLD Database Logic: The Hybrid Rank Fusion Query:**  
*This single query executes classical Keyword Search against Semantic Vector Math, returning the 5 most perfectly fused text blocks.*
```sql
WITH semantic_search AS (
    SELECT v.text_block_id, RANK() OVER (ORDER BY v.embedding_data <=> '[0.12, 0.44...]') AS semantic_rank
    FROM vector_indices v
    WHERE v.tenant_id = 'tenant-xyz'
    LIMIT 50
),
keyword_search AS (
    SELECT t.id AS text_block_id, RANK() OVER (ORDER BY ts_rank_cd(to_tsvector('english', t.raw_text), plainto_tsquery('english', 'revenue plan'))) AS keyword_rank
    FROM universal_text_blocks t
    JOIN knowledge_sources k ON t.source_id = k.id
    WHERE k.user_id = 'tenant-xyz'
    LIMIT 50
)
SELECT 
    COALESCE(s.text_block_id, k.text_block_id) AS text_block_id,
    COALESCE(1.0 / (s.semantic_rank + 60), 0.0) + COALESCE(1.0 / (k.keyword_rank + 60), 0.0) AS rrf_score
FROM semantic_search s
FULL OUTER JOIN keyword_search k ON s.text_block_id = k.text_block_id
ORDER BY rrf_score DESC
LIMIT 5;
```

**2. Hydrating the Presentation UI:**  
*Once the query above returns the `text_block_id` array, grab the context (e.g., PDF page number or Audio timestamp) for the ui UI player.*
```sql
SELECT ks.media_category, ks.external_storage_url, utb.raw_text, utb.display_time_or_page
FROM universal_text_blocks utb
JOIN knowledge_sources ks ON utb.source_id = ks.id
WHERE utb.id IN ('mapped-chunk-1', 'mapped-chunk-2')
ORDER BY utb.chunk_index_ordering ASC;
```

**3. Audit Ingestion Failures:**  
*Identify uploaded knowledge sources that successfully passed the API but whose Parsers completely failed to generate universal chunks.*
```sql
SELECT k.id, k.external_storage_url, k.media_category
FROM knowledge_sources k
LEFT JOIN universal_text_blocks utb ON k.id = utb.source_id
WHERE utb.id IS NULL AND k.ingestion_status = 'completed';
```

**4. Vector DB Multi-tenant Billing Tracker:**  
*Vectors cost money natively. Isolate individual user accounts generating more than 10,000 blocks of textual load.*
```sql
SELECT ks.user_id, COUNT(v.id) AS total_vectors_stored
FROM knowledge_sources ks
JOIN universal_text_blocks utb ON ks.id = utb.source_id
JOIN vector_indices v ON utb.id = v.text_block_id
GROUP BY ks.user_id
HAVING COUNT(v.id) > 10000;
```

**5. Clean Data Pipeline Wiping (GDPR):**  
*Safely initiating cascade deletion for an entire tenant pipeline.*
```sql
DELETE FROM knowledge_sources WHERE user_id = 'tenant-id' RETURNING id;
-- (Foreign Keys utilizing ON DELETE CASCADE instantly wipe universal_text_blocks and vector_indices automatically).
```



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
