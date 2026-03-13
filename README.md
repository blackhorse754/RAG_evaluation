# RAG_evaluation
**Medical Chatbot for ASHA Workers**
 
---
 
## 1. What Is This Framework?
 
SmartHealthGPT is a medical chatbot designed to assist ASHA (Accredited Social Health Activist) workers in India. It uses a **Retrieval-Augmented Generation (RAG)** architecture — meaning the system first retrieves relevant clinical guidelines, then generates an answer grounded in those guidelines.
 
Because ASHA workers use this tool to make real healthcare decisions in the field, standard LLM benchmarks are insufficient. This framework defines a purpose-built set of **13 metrics** that together assess whether the system is safe, accurate, complete, private, and fast enough for frontline primary care use.
 
---
 
## 2. Why a Specialised Evaluation Framework?
 
A RAG system has two components that can each fail independently:
 
- **Retrieval failure** — the wrong guideline is fetched → the answer is grounded in irrelevant text
- **Generation failure** — the right context is retrieved but the model ignores, contradicts, or misrepresents it
 
Standard NLP metrics (BLEU, ROUGE, perplexity) cannot distinguish between these failure modes. This framework uses metrics that specifically probe the retrieval–generation interface, with added emphasis on clinical safety and real-world deployment constraints.
 
---
 
## 3. The 13 Metrics at a Glance
 
| Metric | What It Measures | Why It Matters for ASHA Workers | Group | Needs Ground Truth? | Production Ready? |
|---|---|---|---|---|---|
| **Faithfulness** | Fraction of generated claims supported by retrieved guidelines | Primary clinical safety guardrail — unfaithful = hallucinated | Answer Quality | ✗ No | ✓ Yes |
| **Hallucination Rate** | Fraction of claims that contradict or are absent from retrieved context | Contradictions are patient-safety events; tracked separately from omissions | Answer Quality | ✗ No | ✓ Yes |
| **Completeness** | How many of the 3–5 key clinical steps from the ideal answer are covered | Missing a step in ASHA guidance can be a clinical mistake | Answer Quality | ✓ Yes | Dev only |
| **Factual Correctness** | Statement-level F1 score (TP/FP/FN) vs. ground truth | Best matches expert clinical judgement of a "correct" response | Answer Quality | ✓ Yes | Dev only |
| **Contextual Coherence** | 0–100 LLM judge: does the answer follow logically from retrieved context? | ASHA workers need clearly ordered, logical steps — not just correct facts | Answer Quality | ✗ No | ✓ Yes |
| **Information Density** | LLM judge: informative but concise; penalises verbose answers | Long answers slow ASHA workers; action-focused responses are safer | Answer Quality | ✗ No | ✓ Yes |
| **Recall@k / nDCG@k** | Are the right guideline passages retrieved and ranked highest? | Foundational retrieval health check — wrong retrieval breaks everything downstream | Retrieval Quality | ✓ Yes | Dev only |
| **Context Utilisation** | How much retrieved material is actually used in the answer | Too little = ignoring guidelines; too much verbatim = unusable passages | Retrieval Quality | ✗ No | ✓ Yes |
| **Turn Depth Performance (RB_alg)** | Harmonic mean of completeness, faithfulness & appropriateness per turn | Tracks quality degradation in multi-turn chats — critical for medical safety | Multi-turn | ✓ Yes | Dev only |
| **PII Leakage Rate** | % of responses containing names, Aadhaar numbers, health IDs | DPDP Act compliance; ASHA handles sensitive beneficiary records | Safety & Privacy | ✗ No | ✓ Yes |
| **TTFT** | Time from query to first token displayed | Users notice this delay first; target <2s on slow rural networks | Performance | ✗ No | ✓ Yes |
| **Total Latency (p50/p95/p99)** | Full end-to-end response time including tail latency | p99 surfaces worst-case rural network failures | Performance | ✗ No | ✓ Yes |
| **Single Query Latency (retrieval vs generation)** | Breakdown of which stage is slow | Directs optimisation: slow retrieval → tune vector DB; slow generation → smaller model | Performance | ✗ No | ✓ Yes |
 
---
 
## 4. Evaluation Groups Explained
 
### Group 1 — Answer Quality (6 metrics)
 
These metrics assess the generated answer itself. Together they check that the answer is:
- **True** → Faithfulness
- **Not invented** → Hallucination Rate
- **Not missing steps** → Completeness
- **Factually correct** → Factual Correctness
- **Logically ordered** → Contextual Coherence
- **Concise** → Information Density
 
Most can run without labelled ground truth, making them suitable for live production monitoring.
 
### Group 2 — Retrieval Quality (2 metrics)
 
**Recall@k** and **nDCG@k** measure whether the vector database fetches the right ASHA guideline passages and ranks them correctly. These are the foundation of the pipeline — if retrieval fails, no amount of generation tuning can compensate. **Context Utilisation** then checks that retrieved text is actually reflected in the answer.
 
### Group 3 — Multi-turn Safety (1 metric)
 
**Turn Depth Performance (RB_alg)** from the MTRAG framework tracks how answer quality changes across conversation turns. The harmonic mean of completeness, faithfulness, and appropriateness penalises any single weak dimension — a chatbot that is fluent but unfaithful will score poorly regardless.
 
> **Why harmonic mean?** It deliberately penalises imbalance. A bot scoring perfectly on two dimensions but failing on one still receives a low overall score — preventing a single strong sub-score from masking a critical weakness.
 
### Group 4 — Safety & Privacy (1 metric)
 
**PII Leakage Rate** monitors whether the chatbot inadvertently exposes beneficiary names, Aadhaar numbers, or health IDs. Compliance with India's **Digital Personal Data Protection (DPDP) Act 2023** is non-negotiable.
 
### Group 5 — Performance & Latency (3 metrics)
 
Three latency metrics cover different failure modes:
- **TTFT** → perceived responsiveness (what the user feels)
- **Total Latency p99** → worst-case rural network delays
- **Retrieval vs. Generation split** → pinpoints which component to optimise
 
---
 
## 5. Implementation Approach
 
The framework is designed to be pragmatic and deployable immediately:
 
- **8 of 13 metrics require no ground-truth labels** — they run on live production traffic using NLI models (e.g., DeBERTa) or zero-shot LLM judges
- **5 of 13 metrics require labelled data** — reserved for development evaluation using curated QA pairs from ASHA clinical guidelines
- **Latency metrics** require lightweight timing instrumentation only — no additional ML models needed
- **PII detection** uses regex + a lightweight NER model (e.g., Microsoft Presidio) as a post-generation filter in both dev and production
 
---
 
## 6. Key Design Decisions
 
| Decision | Rationale |
|---|---|
| Faithfulness and Hallucination Rate are **separate** metrics | Faithfulness measures what is supported; Hallucination Rate tracks active contradictions — the highest-risk failure mode |
| Completeness **complements** Faithfulness | A response can be fully supported by retrieved text yet still omit critical clinical steps |
| RB_alg uses **harmonic mean** | Prevents a single strong sub-score from masking a critical weakness |
| Explicit **dev / production split** | 8 metrics run continuously in production; 5 require ground-truth labels for development cycles only |
 
---
 
## Sources
 
RAGChecker · RAGAS · RAGEval · MTRAG · CCRS · RAGBench · MedRAG · Gan et al. (2025) arXiv:2504.14891
