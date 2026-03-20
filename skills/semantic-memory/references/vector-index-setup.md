# Vector Index Setup Guide

## Option 1: SQLite-VSS (Recommended — Zero Dependencies)

SQLite-VSS adds vector search to SQLite. No external services needed.

```python
# .loop/scripts/index-memory.py
import sqlite3
import json
import os
from pathlib import Path
from datetime import datetime

DB_PATH = '.loop/memory-index/memory.db'

def setup_db():
    os.makedirs(os.path.dirname(DB_PATH), exist_ok=True)
    conn = sqlite3.connect(DB_PATH)
    conn.execute('''CREATE TABLE IF NOT EXISTS memories (
        id INTEGER PRIMARY KEY,
        source_file TEXT,
        content TEXT,
        category TEXT,
        created_at TEXT,
        updated_at TEXT,
        embedding BLOB
    )''')
    conn.execute('''CREATE INDEX IF NOT EXISTS idx_category ON memories(category)''')
    conn.execute('''CREATE INDEX IF NOT EXISTS idx_updated ON memories(updated_at)''')
    conn.commit()
    return conn

## Embedding Options

### Local (No API Key Required)
```python
# sentence-transformers — best quality/speed tradeoff
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')  # 80MB, fast
# model = SentenceTransformer('all-mpnet-base-v2')  # 420MB, better quality

def embed(text: str) -> list[float]:
    return model.encode(text, normalize_embeddings=True).tolist()
```

### API-Based
```python
# OpenAI embeddings
import openai

def embed(text: str) -> list[float]:
    response = openai.embeddings.create(
        model="text-embedding-3-small",
        input=text
    )
    return response.data[0].embedding
```

## Option 2: ChromaDB (Feature-Rich)

```python
import chromadb

client = chromadb.PersistentClient(path=".loop/memory-index/chroma")
collection = client.get_or_create_collection(
    name="project_memory",
    metadata={"hnsw:space": "cosine"}
)

def index_memory(file_path: str, content: str, metadata: dict):
    collection.upsert(
        documents=[content],
        metadatas=[{**metadata, "source": file_path}],
        ids=[file_path]
    )

def search(query: str, n_results: int = 5, category: str = None):
    where = {"category": category} if category else None
    results = collection.query(
        query_texts=[query],
        n_results=n_results,
        where=where
    )
    return results
```

## Option 3: FAISS (Highest Performance)

```python
import faiss
import numpy as np
import json

class FAISSIndex:
    def __init__(self, dimension=384):
        self.index = faiss.IndexFlatIP(dimension)  # Inner product (cosine with normalized vectors)
        self.metadata = []
    
    def add(self, embedding: list[float], meta: dict):
        vector = np.array([embedding], dtype='float32')
        faiss.normalize_L2(vector)
        self.index.add(vector)
        self.metadata.append(meta)
    
    def search(self, query_embedding: list[float], k: int = 5):
        vector = np.array([query_embedding], dtype='float32')
        faiss.normalize_L2(vector)
        scores, indices = self.index.search(vector, k)
        results = []
        for score, idx in zip(scores[0], indices[0]):
            if idx >= 0:
                results.append({**self.metadata[idx], 'score': float(score)})
        return results
    
    def save(self, path: str):
        faiss.write_index(self.index, f"{path}/index.faiss")
        with open(f"{path}/metadata.json", 'w') as f:
            json.dump(self.metadata, f)
    
    def load(self, path: str):
        self.index = faiss.read_index(f"{path}/index.faiss")
        with open(f"{path}/metadata.json") as f:
            self.metadata = json.load(f)
```

## Incremental Indexing

Only re-embed files that changed since last index:

```python
import hashlib

HASH_FILE = '.loop/memory-index/file_hashes.json'

def get_file_hash(path: str) -> str:
    with open(path, 'rb') as f:
        return hashlib.md5(f.read()).hexdigest()

def get_changed_files(source_dir: str) -> list[str]:
    current_hashes = {}
    for path in Path(source_dir).rglob('*.md'):
        current_hashes[str(path)] = get_file_hash(str(path))
    
    try:
        with open(HASH_FILE) as f:
            old_hashes = json.load(f)
    except FileNotFoundError:
        old_hashes = {}
    
    changed = [p for p, h in current_hashes.items() if old_hashes.get(p) != h]
    
    with open(HASH_FILE, 'w') as f:
        json.dump(current_hashes, f)
    
    return changed
```

## Chunking Strategy

Large files should be split into meaningful chunks before embedding:

```python
def chunk_markdown(content: str, max_chars: int = 500) -> list[str]:
    """Split markdown by headers, keeping chunks under max_chars."""
    sections = []
    current = []
    current_len = 0
    
    for line in content.split('\n'):
        if line.startswith('#') and current_len > 100:
            sections.append('\n'.join(current))
            current = [line]
            current_len = len(line)
        else:
            current.append(line)
            current_len += len(line)
            if current_len > max_chars:
                sections.append('\n'.join(current))
                current = []
                current_len = 0
    
    if current:
        sections.append('\n'.join(current))
    
    return [s for s in sections if len(s.strip()) > 20]
```