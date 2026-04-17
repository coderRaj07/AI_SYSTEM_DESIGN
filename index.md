# AI System Design Interview Problems

Welcome to the System Design preparation index. This repository focuses on modern structural problems involving AI pipelines, asynchronous generation orchestration, and advanced conversational/RAG architectures.

## Interview Problem Sets
* [Q1: AI Video Generation Platform](./Q1_AI_Video_Gen.md)
* [Q2: AI Audio Intelligence + Chat System](./Q2_AI_Audio_Chat.md)
* [Q3: WhatsApp-like AI Chat System](./Q3_WhatsApp_AI_Chat.md)
* [Q4: AI Video Editing Pipeline](./Q4_AI_Video_Editing.md)
* [Q5: Multi-Modal Content Search System](./Q5_Multi_Modal_Search.md)
* [Q6: Async AI Job Orchestration System](./Q6_Async_Job_Orchestration.md)
* [Q7: AI Content Creation with Versioning](./Q7_AI_Content_Versioning.md)

---
*Created for deep-dive technical engineering notes.*



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
