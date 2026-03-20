---
name: semantic-memory
description: "Semantic vector memory — search past sessions by meaning, build vector indexes over project knowledge, find related context even when wording differs. Use for 'what did we do about X', 'find related notes', 'semantic search'. Do NOT use for simple key-value lookups or file reads."
---

# Semantic Memory — Vector-Indexed Knowledge Retrieval

## When to Activate
- User asks "what did we discuss about...", "find anything related to..."
- Need to find relevant context across many memory files
- Pattern matching across sessions where exact keywords differ
- Building project knowledge bases that grow over time

## Architecture

### Two-Tier Memory (Enhanced)
1. **Hot Cache** — CLAUDE.md + recent session state (keyword match)
2. **Vector Index** — .loop/memory-index/ with embeddings (semantic match)

### How It Works
```
User query: "that auth issue from last week"
  ↓
1. Keyword search CLAUDE.md, memory/*.md (fast, exact)
2. Embed query → search vector index (semantic, fuzzy)
3. Merge results, rank by relevance + recency
4. Return top-K matches with source links
```

## Index Management Protocol

### Building the Index
```bash
# Index all memory and session files
python3 .loop/scripts/index-memory.py --source memory/ --output .loop/memory-index/
```

Read `references/vector-index-setup.md` for:
- Lightweight embedding options (sentence-transformers, OpenAI, local models)
- FAISS / ChromaDB / SQLite-VSS index setup
- Incremental indexing (only re-embed changed files)
- Index storage format (.jsonl with embeddings)

### Querying the Index
1. Embed the user's query using same model as index
2. Cosine similarity search, top-K results (default K=5)
3. Filter by recency (boost last 30 days)
4. Filter by category (project, feedback, reference)
5. Return matches with relevance scores and source file paths

### What Gets Indexed
- memory/*.md files (all types)
- PROGRESS.md checkpoints
- .loop/patterns.jsonl (self-improvement patterns)
- Session summaries (.loop/logs/summary-*.md)
- CLAUDE.md sections

### What Does NOT Get Indexed
- Raw code files (use codebase-understanding for that)
- Full log files (too noisy — only summaries)
- Credentials or secrets

## Similarity Threshold
- **> 0.85**: High confidence match — present directly
- **0.70 - 0.85**: Likely relevant — present with context
- **0.50 - 0.70**: Possibly related — mention as "you might also want..."
- **< 0.50**: Ignore — too noisy

## Integration with Other Skills
- **memory**: Semantic-memory enhances the existing memory skill
- **self-improvement**: Search past patterns by meaning
- **logging**: Index session summaries for later retrieval
- **codebase-understanding**: Complement code search with knowledge search