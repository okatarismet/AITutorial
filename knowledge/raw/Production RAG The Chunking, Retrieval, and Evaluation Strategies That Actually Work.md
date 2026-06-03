---
title: "Production RAG: The Chunking, Retrieval, and Evaluation Strategies That Actually Work"
source: "https://towardsai.net/p/machine-learning/production-rag-the-chunking-retrieval-and-evaluation-strategies-that-actually-work"
author:
  - "[[Ayoub Nainia]]"
published: 2025-12-29
created: 2026-06-03
description: "Author(s): Ayoub Nainia Originally published on Towards AI. RAG isn’t a retrieval problem, it’s a system design problem. The sooner you start treating it li ..."
tags:
  - "clippings"
---
#### Author(s): Ayoub Nainia

Originally published on [Towards AI](https://towardsai.net/).

> RAG isn’t a retrieval problem, it’s a system design problem. The sooner you start treating it like one, the sooner it will stop breaking.

If you’ve built your first RAG (Retrieval-Augmented Generation) system, you’ve probably experienced the harsh reality: the basic tutorial approach works for demos, but falls apart with real documents and user queries. Your users complain about irrelevant answers, missed information, or responses that confidently cite the wrong context.

I’ve spent this year building and refining RAG systems in production, and I’ve learned that the gap between “RAG demo” and “RAG that works” is massive.

In this post, we cover the practical strategies that actually move the needle.

![Production RAG: The Chunking, Retrieval, and Evaluation Strategies That Actually Work](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ws3XB0vMDmx9dLSw9MTSwg.png)

Production RAG pipeline: Hybrid retrieval (vector + BM25), cross-encoder reranking, context expansion, and quality checks. Notice the fallback loop for low-confidence answers.

## The Problem with Basic RAG

Most RAG tutorials follow the same pattern:

1. Split documents into fixed-size chunks (512 tokens)
2. Embed chunks with an off-the-shelf model
3. Store in a vector database
4. Retrieve top-k similar chunks
5. Stuff into [LLM](https://academy.towardsai.net/courses/beginner-to-advanced-llm-dev "LLM Dev") context

This works for simple cases, but breaks down when:

- Users ask questions spanning multiple sections
- Important context is split across chunk boundaries
- Semantically similar text isn’t actually relevant
- Documents have complex structure (tables, lists, code)
- You need to retrieve from thousands of documents

The symptoms are familiar: incomplete answers, hallucinations when the right context isn’t retrieved, or correct information that’s simply missed because it was split incorrectly.

Let’s fix these issues systematically.

**In this article:**

- **Part 1**: Chunking Strategies (Semantic, hierarchical, and hybrid approaches that preserve context)
- **Part 2**: Retrieval Optimization (Hybrid search, reranking, and context expansion)
- **Part 3**: Evaluation Framework (Metrics that actually predict user satisfaction)
- **Part 4**: Production Best Practices (Caching, monitoring, and graceful degradation)

## Part 1: Chunking Strategies That Preserve Context

### 1\. The Fixed-Size Trap

Fixed-size chunking (splitting every N tokens) is convenient but destructive. It ignores document structure and splits sentences mid-thought. Here’s what happens:

```
def naive_chunk(text, chunk_size=512):
 tokens = tokenize(text)
 return [tokens[i:i+chunk_size] for i in range(0, len(tokens), chunk_size)]
```

**Problem**: A paragraph explaining a concept gets split, with the second half losing context.

### 2\. Strategy 1: Semantic Chunking

Split on semantic boundaries (paragraphs, sections) while respecting size limits:

```
def semantic_chunk(50
):
 chunks = []
 current_chunk = []
 current_size = 0
 
 # Split on double newlines (paragraphs)
 paragraphs = text.split('\n\n')
 
 for para in paragraphs:
 para_tokens = tokenize(para)
 para_size = len(para_tokens)
 
 # If paragraph is too large, split it
 if para_size > max_tokens:
 # Split to sentences
 sentences = split_sentences(para)
 for sent in sentences:
 sent_size = len(tokenize(sent))
 if current_size + sent_size > max_tokens:
 # Save current chunk with overlap
 chunks.append(' '.join(current_chunk))
 # Keep last few sentences for context
 current_chunk = current_chunk[-overlap:]
 current_size = sum(len(tokenize(s)) for s in current_chunk)
 current_chunk.append(sent)
 current_size += sent_size
 else:
 # Add paragraph to current chunk
 if current_size + para_size > max_tokens:
 chunks.append(' '.join(current_chunk))
 current_chunk = [para]
 current_size = para_size
 else:
 current_chunk.append(para)
 current_size += para_size
 
 if current_chunk:
 chunks.append(' '.join(current_chunk))
 
 return chunks
```

**Key improvements**:

- Preserves paragraph structure
- Adds overlap between chunks (critical for boundary cases)
- Respects sentence boundaries

**Why this works**: When you preserve paragraph boundaries, you maintain the logical flow of ideas. The overlap between chunks is critical, since it ensures that concepts spanning boundaries are captured in both chunks. In my testing, this alone improved retrieval accuracy by about 30% compared to fixed-size chunking.

**The key insight:** text isn’t just a stream of tokens. It has structure, and that structure carries meaning.

### 3\. Strategy 2: Document Structure-Aware Chunking

For structured documents, preserve hierarchy:

```
def hierarchical_chunk(document):
 chunks = []
 for section in document.sections:
 section_header = f"# {section.title}\n"
 
 # Include parent context
 parent_context = ""
 if section.parent:
 parent_context = f"(From {section.parent.title})\n"
 
 for para in section.paragraphs:
 chunk = {
 'text': para.text,
 'metadata': {
 'section': section.title,
 'parent': section.parent.title if section.parent else None,
 'header_context': section_header,
 'full_path': section.get_path() # e.g., "Chapter 3 > Section 3.2"
 }
 }
 chunks.append(chunk)
 
 return chunks
```

**Why this matters**: When retrieving, you can include section headers and hierarchical context, helping the [LLM](https://academy.towardsai.net/courses/beginner-to-advanced-llm-dev "LLM Dev") understand where information fits in the document structure.

In practice, this metadata becomes part of what you feed to the LLM, which will allow it to have spatial awareness within the document.

### 4\. Strategy 3: Hybrid Chunking for Complex Documents

Real-world documents are messy. They contain tables, code snippets, diagrams, and mixed content. Each content type needs different handling.

```
def hybrid_chunk(document):
 chunks = []
 
 for element in document.elements:
 if element.type == 'table':
 # Keep tables intact, add descriptive text
 chunk = {
 'text': f"Table: {element.caption}\n{element.to_markdown()}",
 'type': 'table',
 'metadata': element.metadata
 }
 chunks.append(chunk)
 
 elif element.type == 'code':
 # Code blocks with context
 chunk = {
 'text': f"\`\`\`{element.language}\n{element.code}\n\`\`\`\n{element.description}",
 'type': 'code',
 'metadata': element.metadata
 }
 chunks.append(chunk)
 
 elif element.type == 'text':
 # Use semantic chunking for regular text
 text_chunks = semantic_chunk(element.text)
 chunks.extend([{'text': t, 'type': 'text'} for t in text_chunks])
 
 return chunks
```

**Why different strategies matter:** Tables lose all meaning when split. Code without context is useless. But regular paragraphs benefit from semantic splitting. By treating each content type appropriately, you preserve the information density.

I learned this the hard way when users kept complaining that “the pricing table is incomplete”. Turned out we were splitting tables across chunks, making them incomprehensible.

## Part 2: Retrieval Optimization

### 1\. The Top-K Problem

Simply retrieving the top-k most similar chunks often fails because:

- Semantic similarity doesn’t equal relevance
- You miss important context from surrounding chunks
- Similar but irrelevant chunks rank high

### 2\. Strategy 1: Hybrid Search

Combining vector similarity with keyword search dramatically improves retrieval quality. Vector search captures semantic meaning, while BM25 catches exact term matches.

```
def hybrid_search(query, vector_db, bm25_index, alpha=0.5):
 """Combine semantic and keyword search"""
 
 # Vector search
 vector_results = vector_db.search(query, top_k=20)
 vector_scores = {doc.id: doc.score for doc in vector_results}
 
 # BM25 keyword search
 bm25_results = bm25_index.search(query, top_k=20)
 bm25_scores = {doc.id: doc.score for doc in bm25_results}
 
 # Combine scores
 all_doc_ids = set(vector_scores.keys()) | set(bm25_scores.keys())
 combined_scores = {}
 
 for doc_id in all_doc_ids:
 # Normalize scores to 0-1 range
 v_score = vector_scores.get(doc_id, 0)
 k_score = bm25_scores.get(doc_id, 0)
 
 # Weighted combination
 combined_scores[doc_id] = (alpha * v_score + (1 - alpha) * k_score)
 
 # Sort and return top k
 ranked = sorted(combined_scores.items(), key=lambda x: x[1], reverse=True)
 return [doc_id for doc_id, score in ranked[:10]]
```

**When to use**: Queries with specific terms (product names, technical jargon) benefit from keyword matching.

 [![](https://miro.medium.com/v2/da:true/resize:fit:0/60026f4340686a391639ac58864da18070aa773cea45de6e55fa47fd56bfdb74) ![](https://miro.medium.com/v2/da:true/resize:fit:0/c061bd6cb52734164bf0c66f2543a6bc2acbe24ae3985dc15c898b3ddb2e1940)](https://medium.com/plans?source=upgrade_membership---post_li_non_moc_upsell--11ae2d26cd6c---------------------------------------)

The alpha parameter lets you tune the balance. I’ve found 0.5 works well for general content, but lean more toward BM25 (alpha=0.3) for technical docs with lots of specific terminology.

### 3\. Strategy 2: Query Expansion

Reformulate the user’s query to improve retrieval:

```
def expand_query(query, llm):
 """Generate alternative phrasings"""
 prompt = f"""Given this question: "{query}"
 
Generate 3 alternative ways to phrase this question that might help find relevant information:
1. A more specific version
2. A more general version
3. Using different terminology
Return as JSON list."""
 alternatives = llm.generate(prompt)
 
 # Search with all query variants
 all_results = []
 for q in [query] + alternatives:
 results = vector_db.search(q, top_k=5)
 all_results.extend(results)
 
 # Deduplicate and rerank
 return deduplicate_and_rerank(all_results)
```

**The trick:** Don’t just search with alternatives. Use them to cast a wider net, then deduplicate and rerank. This catches relevant documents that use different terminology than the user. The downside is latency (multiple searches), so use this selectively for complex queries.

### 4\. Strategy 3: Reranking

Use a cross-encoder to rerank retrieved chunks:

```
from sentence_transformers import CrossEncoder
def rerank_results(query, chunks, top_k=5):
 """Rerank with cross-encoder for better relevance"""
 
 reranker = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')
 
 # Create query-chunk pairs
 pairs = [[query, chunk.text] for chunk in chunks]
 
 # Score all pairs
 scores = reranker.predict(pairs)
 
 # Sort by score
 ranked_indices = scores.argsort()[::-1][:top_k]
 
 return [chunks[i] for i in ranked_indices]
```

**Why this works**: Cross-encoders see the query and chunk together, capturing relevance that bi-encoders miss.

### 5\. Strategy 4: Context Expansion

Retrieve neighboring chunks around top results:

```
def retrieve_with_context(query, vector_db, window=1):
 """Retrieve chunks plus surrounding context"""
 
 # Initial retrieval
 top_chunks = vector_db.search(query, top_k=5)
 
 # Expand to include neighbors
 expanded_chunks = []
 for chunk in top_chunks:
 # Get chunk position in document
 doc_id = chunk.metadata['document_id']
 chunk_idx = chunk.metadata['chunk_index']
 
 # Retrieve surrounding chunks
 for i in range(chunk_idx - window, chunk_idx + window + 1):
 neighbor = get_chunk(doc_id, i)
 if neighbor:
 expanded_chunks.append(neighbor)
 
 # Deduplicate and maintain order
 return deduplicate_by_position(expanded_chunks)
```

This prevents information loss from split context.

## Part 3: Evaluation That Actually Matters

### 1\. Beyond Cosine Similarity

Most teams stop at “does the right chunk rank high?” But what users care about is: “Did I get the right answer?”

### 2\. Evaluation Framework

```
class RAGEvaluator:
 def __init__(self, test_cases):
 self.test_cases = test_cases # List of (query, expected_answer, relevant_docs)
 
 def evaluate_retrieval(self, retrieval_fn):
 """Measure retrieval quality"""
 metrics = {
 'recall@k': [],
 'precision@k': [],
 'mrr': [] # Mean Reciprocal Rank
 }
 
 for query, _, relevant_docs in self.test_cases:
 retrieved = retrieval_fn(query, k=10)
 retrieved_ids = [doc.id for doc in retrieved]
 
 # Recall: what % of relevant docs were retrieved?
 relevant_retrieved = set(retrieved_ids) & set(relevant_docs)
 recall = len(relevant_retrieved) / len(relevant_docs)
 metrics['recall@k'].append(recall)
 
 # Precision: what % of retrieved docs were relevant?
 precision = len(relevant_retrieved) / len(retrieved_ids)
 metrics['precision@k'].append(precision)
 
 # MRR: reciprocal rank of first relevant doc
 for rank, doc_id in enumerate(retrieved_ids, 1):
 if doc_id in relevant_docs:
 metrics['mrr'].append(1.0 / rank)
 break
 else:
 metrics['mrr'].append(0.0)
 
 return {k: sum(v) / len(v) for k, v in metrics.items()}
 
 def evaluate_end_to_end(self, rag_system, llm_judge):
 """Measure answer quality"""
 scores = {
 'correctness': [],
 'completeness': [],
 'faithfulness': [] # Does answer stick to retrieved context?
 }
 
 for query, expected_answer, _ in self.test_cases:
 # Generate answer
 answer = rag_system.answer(query)
 
 # LLM-as-judge evaluation
 eval_prompt = f"""Query: {query}
Expected Answer: {expected_answer}
Generated Answer: {answer}
Retrieved Context: {answer.context}
Rate the generated answer on:
1. Correctness (0-1): Does it match the expected answer?
2. Completeness (0-1): Does it cover all aspects?
3. Faithfulness (0-1): Is it supported by the retrieved context?
Return JSON."""
 ratings = llm_judge.evaluate(eval_prompt)
 for metric, score in ratings.items():
 scores[metric].append(score)
 
 return {k: sum(v) / len(v) for k, v in scores.items()}
```

### 3\. Key Metrics to Track

- Retrieval Recall@k: Are relevant documents being found?
- Answer Correctness: Does the final answer match ground truth?
- Faithfulness: Does the answer cite retrieved context (no hallucination)?
- Latency: p50, p95, p99 response times

Cost: Tokens used per query

### 4\. A/B Testing Different Strategies

```
def compare_strategies(test_queries):
 strategies = {
 'baseline': baseline_rag,
 'hybrid_search': hybrid_rag,
 'reranked': reranked_rag,
 'full_pipeline': optimized_rag
 }
 
 results = {}
 for name, strategy in strategies.items():
 metrics = evaluate(strategy, test_queries)
 results[name] = metrics
 
 # Compare improvements
 baseline_score = results['baseline']['correctness']
 for name, metrics in results.items():
 improvement = (metrics['correctness'] - baseline_score) / baseline_score * 100
 print(f"{name}: {metrics['correctness']:.3f} (+{improvement:.1f}%)")
```

## Part 4: Production Best Practices

### 1\. Metadata Filtering

Don’t search the entire corpus for every query:

```
def filtered_search(query, user_context):
 """Filter by metadata before vector search"""
 
 # Build filter from context
 filters = {
 'user_id': user_context.user_id,
 'access_level': user_context.permissions,
 'date_range': user_context.relevant_timeframe
 }
 
 # Search only relevant subset
 results = vector_db.search(
 query, 
 filters=filters,
 top_k=10
 )
 
 return results
```

This dramatically improves speed and relevance.

### 2\. Caching

Cache at multiple levels:

```
lass CachedRAG:
 def __init__(self, rag_system):
 self.rag = rag_system
 self.query_cache = {} # Query -> Answer
 self.retrieval_cache = {} # Query -> Chunks
 
 def answer(self, query):
 # Exact match cache
 if query in self.query_cache:
 return self.query_cache[query]
 
 # Semantic similarity cache
 similar_queries = self.find_similar_cached_queries(query, threshold=0.95)
 if similar_queries:
 return self.query_cache[similar_queries[0]]
 
 # Generate new answer
 answer = self.rag.answer(query)
 self.query_cache[query] = answer
 
 return answer
```

### 3\. Monitoring

Track what matters in production:

```
def log_rag_interaction(query, answer, retrieved_chunks, user_feedback=None):
 """Log for debugging and improvement"""
 
 metrics = {
 'timestamp': datetime.now(),
 'query': query,
 'num_chunks_retrieved': len(retrieved_chunks),
 'retrieval_scores': [c.score for c in retrieved_chunks],
 'answer_length': len(answer),
 'generation_latency': answer.latency_ms,
 'user_feedback': user_feedback, # thumbs up/down
 }
 
 # Store for analysis
 analytics_db.insert(metrics)
 
 # Alert on anomalies
 if metrics['retrieval_scores'][0] < 0.5:
 alert("Low confidence retrieval", metrics)
```

### 4\. Handling Failure Cases

```
def robust_rag(query):
 
 try:
 # Try optimal strategy
 chunks = hybrid_search_with_reranking(query)
 
 if not chunks or chunks[0].score < 0.3:
 # Fall back to broader search
 chunks = fallback_search(query)
 
 if not chunks:
 # Admit when you don't know
 return {
 'answer': "I couldn't find relevant information to answer this question.",
 'confidence': 0.0,
 'suggestion': "Try rephrasing or asking about a different aspect."
 }
 
 answer = generate_answer(query, chunks)
 
 return {
 'answer': answer.text,
 'confidence': answer.confidence,
 'sources': [c.metadata for c in chunks]
 }
 
 except Exception as e:
 log_error(e)
 return fallback_response()
```

## Putting It All Together

Here’s the full production-ready pipeline:

```
class ProductionRAG:
 def __init__(self):
 self.chunker = HybridChunker()
 self.vector_db = VectorDB()
 self.bm25_index = BM25Index()
 self.reranker = CrossEncoder()
 self.cache = QueryCache()
 self.evaluator = RAGEvaluator()
 
 def ingest_document(self, document):
 """Process and index a document"""
 # Smart chunking
 chunks = self.chunker.chunk(document)
 
 # Embed and index
 embeddings = self.embed(chunks)
 self.vector_db.add(chunks, embeddings)
 self.bm25_index.add(chunks)
 
 return len(chunks)
 
 def answer(self, query, user_context=None):
 """Full RAG pipeline"""
 # Check cache
 if cached := self.cache.get(query):
 return cached
 
 # Hybrid retrieval
 vector_results = self.vector_db.search(query, top_k=20, filters=user_context)
 bm25_results = self.bm25_index.search(query, top_k=20)
 combined = self.merge_results(vector_results, bm25_results)
 
 # Rerank
 top_chunks = self.reranker.rerank(query, combined, top_k=5)
 
 # Expand context
 chunks_with_context = self.expand_context(top_chunks)
 
 # Generate answer
 answer = self.generate(query, chunks_with_context)
 
 # Cache and return
 self.cache.set(query, answer)
 return answer
```

## The Reality Check

After implementing these strategies, here’s what I learned:

**What worked best:**

- Semantic chunking with overlap (improvement over fixed-size)
- Hybrid search (improvement in retrieval recall)
- Reranking (improvement in answer quality)
- Context expansion (eliminated most “incomplete answer” complaints)

**What didn’t matter as much:**

- Exotic embedding models
- Extremely large k values
- Complex query expansion (simple rephrasing worked as well)

**The biggest wins came from:**

- Good evaluation framework (can’t improve what you don’t measure)
- Proper document preprocessing (garbage in, garbage out)
- Metadata filtering (speed + relevance)

## Conclusion

Building production RAG isn’t about implementing every fancy technique. It’s about:

1. Understanding your failure modes
2. Measuring the right things
3. Iterating on what actually improves user experience
4. Building robust systems that degrade gracefully

Start with semantic chunking and hybrid search. Add reranking if retrieval quality is your bottleneck. Implement proper evaluation before anything else.

The difference between a demo and production RAG is treating it as a system that needs constant measurement and iteration, not a one-time implementation.

*What challenges are you facing with your RAG system?*

Published via [Towards AI](https://towardsai.net/)