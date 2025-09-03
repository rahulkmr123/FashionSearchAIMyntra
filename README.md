## Project Report: Fashion Search AI

- **Overview**: Fashion semantic search over Myntra dataset using local embeddings, ChromaDB persistence, cache, reranking, and generation.

### 1. Objectives
- Build a fast, scalable fashion search that returns relevant items for natural-language queries.
- Persist embeddings locally to enable reproducible, cost-efficient runs.
- Improve result quality via reranking and provide concise explanations.

### 2. Dataset
- **Source**: `Assignment/FashionData/FashionLargeDataset.csv` (14,214 items), images under `Assignment/FashionData/images/`.
- **Key fields**: `p_id`, `name`, `brand`, `price`, `avg_rating`, `description`, `p_attributes`.
- **Cleaning**: Filled missing ratings; constructed a consolidated `metadata` dict for each item.

### 3. Methods
- **Embeddings**: `all-MiniLM-L6-v2` (SentenceTransformers) for local, fast embeddings.
- **Vector store**: `chromadb.PersistentClient(path=Assignment/FashionData/ChromaDB_Data)`, collection `Fashion_Products_Local`.
- **Caching**: Disk-backed collection `Fashion_Cache_1`; nested results are saved as a JSON string in metadata to meet Chroma’s primitive-only rule.
- **Retrieval**: Cosine distance from Chroma; smaller distance is more similar.
- **Reranking**: CrossEncoder `ms-marco-MiniLM-L-6-v2` with two strategies:
  - semantic_first: rerank the top-k semantically similar items only.
  - global_rerank: rerank all candidates and pick top-k.
- **Generation**: OpenAI Chat model produces concise, user-facing summaries grounded in retrieved items.

### 4. System Architecture
- Ingestion/Prep → Embeddings → ChromaDB Persistence → Search → Cache Check → Rerank → Response Generation.
- **Batching**: Size 100; stable IDs as strings; deterministic on re-runs.
- **Cache**: After a main search, store compacted results as JSON (e.g., `search_results_json`); on cache read, decode JSON and return fields.

### 5. Implementation Highlights
- Notebook: `Assignment/MrHelpMateFashionSearch.ipynb`
  - Section 3: embeddings and persistence
  - Section 4: semantic search + cache
  - Section 5: reranking
  - Section 6: generation
  - Section 7: conclusion
- Clean logging: emojis removed; production-suitable prints.
- Verified persistence: `fashion_collection.count()` equals 14214.

### 6. Results
- Index built for all 14,214 items.
- Retrieval quality subjectively improved with reranking (semantic_first/global_rerank).
- Cache reduces latency on repeated queries; JSON metadata avoids Chroma metadata type errors.

### 7. Limitations
- No quantitative evaluation (Recall@k, MRR, nDCG) yet.
- Product descriptions may include HTML artifacts.
- Basic image presence checks; no fallback thumbnails.
- OpenAI generation requires API key and has cost/latency tradeoffs.

### 8. Future Work
- **Quality**: Add offline evaluation (Recall@k, nDCG), golden set tests, and regression checks.
- **Cache**: Add TTL/versioning and normalized keys; consider deduping similar queries.
- **Filtering**: Add structured filters (brand, price range, color) and hybrid search (text + metadata).
- **Metadata hygiene**: Strip HTML, standardize attributes, and enrich with faceted fields.
- **Serving**: Wrap search/rerank/generation into API endpoints (e.g., FastAPI); externalize config via environment variables.
- **Scale**: Consider stronger embeddings (e.g., e5-large-v2) and index optimizations (IVF/PQ) for larger corpora.

### 9. Reproducibility
- Paths (relative to repo root):
  - Data: `Assignment/FashionData/FashionLargeDataset.csv`
  - Images: `Assignment/FashionData/images/`
  - Chroma path: `Assignment/FashionData/ChromaDB_Data`
  - Notebook: `Assignment/MrHelpMateFashionSearch.ipynb`
- Run Sections 3–6 in the notebook; ensure Python env has SentenceTransformers, ChromaDB, and optional OpenAI key.

### 10. References
- Internal artifacts:
  - Notebook: `Assignment/MrHelpMateFashionSearch.ipynb`
  - Chroma files: `Assignment/FashionData/ChromaDB_Data/` (includes `chroma.sqlite3`)
  - Images: `Assignment/FashionData/images/`
- Attached PDFs and materials:
  - Building Effective Search Systems HelpMateAI (`helpmate-ai-building-effective-search-systems/Building Effective Search Systems HelpMateAI.pdf`)
  - Query screenshots (`FashionSearchAI-MyntraDataset/ScreenShots/`)
- Libraries:
  - SentenceTransformers, ChromaDB, OpenAI Python SDK


### 11. How to Run

1) Create and activate a virtual environment (macOS/Linux)

```
cd "<ProjectDir>"
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip setuptools wheel
```

2) Install dependencies

```
pip install sentence-transformers==5.1.0 chromadb==1.0.20 pandas scikit-learn matplotlib pillow openai tiktoken torch
```

3) Ensure dataset and images are present <Kaggle>

- CSV: `Assignment/FashionData/FashionLargeDataset.csv`
- Images: `Assignment/FashionData/images/`

4) (Optional) Set OpenAI key for the generation step

```
export OPENAI_API_KEY="<your_api_key>"
```

5) Launch Jupyter and open your notebook

```
pip install jupyterlab
jupyter lab
```

- Open an fashion search notebook implementing the pipeline. MrHelpMateFashionSearch.ipynb
- Run cells to:
  - Load the CSV and images
  - Build/verify the ChromaDB collection at `Assignment/FashionData/ChromaDB_Data`
  - Perform search → cache → rerank → (optional) generation

7) Troubleshooting

- If the collection is empty, (re)run the embedding/batching step to populate `Fashion_Products_Local`.
- If you see metadata type errors, ensure cache metadata is JSON-serialized (primitive-only fields).
- For performance, keep batch size near 100 and use the same collection name across runs.


