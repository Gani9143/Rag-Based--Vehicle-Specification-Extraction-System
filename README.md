# 🚗 RAG-Based Vehicle Specification Extraction System

AI-Powered Mechanic Assistant for automatic technical-spec retrieval from vehicle service manuals.

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue?style=for-the-badge&logo=python)](https://www.python.org/)
[![FAISS](https://img.shields.io/badge/FAISS-Vector%20DB-orange?style=for-the-badge)](https://github.com/facebookresearch/faiss)
[![LangChain](https://img.shields.io/badge/LangChain-RAG-green?style=for-the-badge)](https://github.com/langchain-ai/langchain)
[![Groq](https://img.shields.io/badge/Groq-Llama3.3-ff9900?style=for-the-badge)](https://www.groq.com/)

---

Table of contents
- Overview
- Quick demo
- Features
- How it works (architecture)
- Installation & usage
- Configuration & best practices
- Project structure
- Roadmap
- Contributing
- License & credits

---

## 📌 Overview

Mechanics and technicians often need a single number from very long service manuals — torque, clearances, fluid capacities, or part numbers. This project automates the retrieval and extraction of those technical specifications using a Retrieval-Augmented Generation (RAG) pipeline:

- PDF ingestion (text + tables)
- Chunking with header injection to retain table context
- Embeddings (SentenceTransformers)
- FAISS vector retrieval
- LLM (Groq Llama 3.3) for strict JSON extraction
- Post-processing & unit normalization

Example user query:
> "What is the torque for the front lower control arm bolt?"

Example JSON output:
```json
{
  "component": "Front Lower Control Arm",
  "spec_type": "Torque",
  "value": 125,
  "unit": "Nm",
  "source": {"pdf": "sample-service-manual.pdf", "page": 142}
}
```

---

## ✨ Key Features

- Full RAG stack: FAISS vector DB + instruction-tuned LLM for deterministic extraction
- Strict JSON schema for all outputs (component, spec_type, value, unit, source)
- Table-aware PDF parsing (Camelot / Tabula) + header injection so rows remain meaningful
- Smart unit parsing and normalization (SI ↔ Imperial, multi-unit recognition)
- Crash protection: invalid model outputs are detected and fallback extraction attempts are made
- Streamlit-ready frontend and low-memory FAISS setup for quick demos

---

## 🧩 Design choices & extraction rules

- Chunk size: 2000 characters, overlap: 300 (RecursiveCharacterTextSplitter)
- Separator priority: ["\n\n", "\n", " ", ""]
- Embeddings: all-MiniLM-L6-v2 (dim = 384)
- Default FAISS index: Flat (L2) for maximum accuracy on small/medium datasets
- LLM inference defaults for structured extraction: temperature 0.0–0.2, top_p 0.9–0.95

Metadata example for vectors:
```json
{
  "page": 142,
  "chunk_id": "c142_3",
  "source": "sample-service-manual.pdf"
}
```

---

## 🛠 How it works (RAG flow)

                  ┌───────────────────────────┐
                  │     PDF Ingestion         │
                  │  (text + tables extract)  │
                  └──────────────┬────────────┘
                                 ▼
                      ┌───────────────────┐
                      │ Chunker + Header │
                      │    Injection     │
                      └──────────┬────────┘
                                 ▼
                      ┌──────────────────────┐
                      │ Embeddings Generator │
                      │  (MiniLM-L6-v2)      │
                      └──────────┬───────────┘
                                 ▼
                      ┌──────────────────────┐
                      │     FAISS Index      │
                      │   (Vector Storage)   │
                      └──────────┬───────────┘
                                 ▼
   ┌──────────────────────┐   User Query   ┌─────────────────────┐
   │  Streamlit Frontend  │───────────────►│     Retriever       │
   └──────────────────────┘                └─────────────────────┘
                                            │ Top-k chunks
                                            ▼
                                ┌─────────────────────────┐
                                │ Groq Llama 3.3 (70B)    │
                                │ RAG + JSON Extraction   │
                                └──────────┬──────────────┘
                                           ▼
                                 ┌──────────────────┐
                                 │ Post Processing  │
                                 │ Units + Schema   │
                                 └──────────────────┘
                                           ▼
                               ┌─────────────────────────┐
                               │ Final JSON Specification│
                               └─────────────────────────┘

---

## 💾 Project structure (simplified)

Rag-Based--Vehicle-Specification-Extraction-System/
- faiss_db_index/             
- sample-service-manual.pdf    
- vehicle_specs.json           
- final.ipynb                  
- README.md
- requirements.txt (recommended)
- .gitignore

---

## ⚙️ Installation

1. Clone
```bash
git clone https://github.com/Gani9143/Rag-Based--Vehicle-Specification-Extraction-System.git
cd Rag-Based--Vehicle-Specification-Extraction-System
```

2. Create venv (recommended)
```bash
python -m venv .venv
source .venv/bin/activate    # macOS / Linux
.venv\Scripts\activate       # Windows
```

3. Install dependencies
```bash
pip install -r requirements.txt
```

4. Run the demo notebook
- Open final.ipynb in Jupyter / VS Code and run cells in order.

Optional: Run Streamlit demo (if implemented)
```bash
streamlit run app.py
```

---

## 🔧 Configuration & best practices

- Model & inference:
  - Use deterministic settings (temperature 0–0.2) for numeric outputs.
  - Include the JSON Schema in the prompt and provide 2–3 few-shot examples of expected JSON.
- Retrieval:
  - For small sets, Flat L2 FAISS is fine. For large corpora consider HNSW / IVF + PQ.
- Chunking:
  - Keep chunk size large enough to include table context. Inject headers into extracted table rows so each chunk is self-describing.
- Post-processing:
  - If the LLM returns invalid JSON, attempt regex-based numeric + unit extraction as a fallback.

Example minimal system prompt (illustrative):
```
You are an extraction assistant. Given retrieved manual fragments and metadata, output exactly one valid JSON object matching:
{ "component": string, "spec_type": string, "value": number, "unit": string, "source": {"pdf": string, "page": number} }
```

---

## 🧪 Examples & expected outputs

Input:
> "What is the torque for the rear suspension arm bolt?"

Output:
```json
{
  "component": "Rear Suspension Arm Bolt",
  "spec_type": "Torque",
  "value": 155,
  "unit": "Nm",
  "source": { "pdf": "sample-service-manual.pdf", "page": 118 }
}
```

Unit parsing examples handled:
- "17 Nm (12.6 lb-ft) / 150 lb-in" → capture both primary and converted units when present and normalize.

---

## 🔮 Roadmap

- Table-aware chunking — Complete
- Camelot-first pipeline — In progress
- Hybrid search (BM25 + FAISS) — Planned
- Full Streamlit UI — Planned
- Vision-based spec extraction (images/diagrams) — Backlog
- FastAPI backend for batch extraction — Planned

---

## 🤝 Contributing

Contributions welcome! Please:
- Open issues for bugs/feature requests
- Send PRs with clear descriptions and tests where possible
- Follow the project's code style and add documentation for new features

---

## 📄 License

MIT License — see LICENSE file.

---

## ❤️ Credits

- HuggingFace Transformers
- FAISS (Meta AI)
- Sentence Transformers
- LangChain
- Groq Llama 3.3 (model provider)
