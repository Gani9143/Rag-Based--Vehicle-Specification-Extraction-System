# 🚗 RAG-Based Vehicle Specification Extraction System
_ AI-Powered Mechanic Assistant for Automatic Spec Retrieval_

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?style=for-the-badge&logo=python)
![FAISS](https://img.shields.io/badge/FAISS-Vector%20DB-orange?style=for-the-badge)
![LangChain](https://img.shields.io/badge/LangChain-RAG-green?style=for-the-badge)
![Groq](https://img.shields.io/badge/Groq-Llama3.3-ff9900?style=for-the-badge)

---

## 📌 Overview
Modern automotive service manuals are hundreds of pages long, making it time-consuming for mechanics to find torque values, fluid quantities, bolt specs, alignment angles, or part numbers.

**RAG-Based Vehicle Specification Extraction System** automates this task. It uses a Retrieval-Augmented Generation pipeline (FAISS + LLM) to retrieve the most relevant manual chunks and extract technical specifications accurately.

**Example user query:**  
> “What is the torque for the front lower control arm bolt?”

**Example JSON output:**
```json
{
  "component": "Front Lower Control Arm",
  "spec_type": "Torque",
  "value": 125,
  "unit": "Nm",
  "source": {"pdf": "sample-service-manual.pdf", "page": 142}
}
```

Key Features
🔍 1. Full RAG Architecture

Uses FAISS vector search + Groq Llama 3.3 LLM to retrieve the most relevant manual chunks and extract technical specifications accurately.

🧩 2. Strict JSON Schema Output

All outputs follow a strict structure:

Component | Spec Type | Value | Unit | Source (PDF + Page)

📐 3. Smart Unit Parsing

Automatically detects and splits compound units such as:

17 Nm (12.6 lb-ft) / 150 lb-in

Supports SI and Imperial units.

🧠 4. Context-Aware Table Extraction

✔ Extracts tables using Camelot/Tabula  
✔ Converts table data into JSON  
✔ Injects headers into every row so chunking never loses meaning

Header Injection Example

Raw:

| Bolt | 17 |

Injected:

Component: Bolt | Spec: Torque | Value: 17 Nm

🧱 5. Chunking Strategy Optimized for Manuals

Chunk size: 2000 characters  
Overlap: 300 characters  
Separator priority:

["\n\n", "\n", " ", ""]

Keeps paragraphs and tables intact for better retrieval accuracy.

🛡️ 6. Crash Protection

If LLM returns unstructured/free text:

System detects invalid JSON  
Attempts to extract numbers and units  
Returns fallback readable results

☁️ 7. Streamlit Deployment Ready

Designed for Streamlit Cloud:

Low memory usage

Fast FAISS retrieval

Clean UI output

🧠 Technical Deep Dive
📄 1. PDF Ingestion Pipeline

Performs:

PDF text extraction

Table extraction

Header injection

Page-level metadata tagging

🧩 2. Chunking Strategy Table
Property	Value
Chunk size	2000 chars
Overlap	300 chars
Algorithm	RecursiveCharacterTextSplitter
Goal	Preserve table + paragraph context

🧬 3. Embeddings

Model: all-MiniLM-L6-v2

Dimensionality: 384

Metadata example:

{
  "page": 142,
  "chunk_id": "c142_3",
  "source": "sample-service-manual.pdf"
}

🧱 Vector Database – FAISS
Default: Flat L2

Max accuracy

Suitable for small/medium datasets

Advanced Options
Index	Purpose
HNSW	Large dataset, 10x faster search
IVF	Clustering for scalable vector search
PQ	High compression for large manuals

🏗️ System Architecture (RAG Flow)
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

🛠️ Tech Stack
Component	Technology
Frontend	Streamlit
LLM	Groq Llama 3.3
Embeddings	Sentence Transformers
Vector DB	FAISS
PDF Parsing	Camelot / PyPDF2
RAG Pipeline	LangChain
Validation	Pydantic / JSON Schema

## Groq Llama 3.3 — Model details & recommended usage
This project uses Groq's distribution of Llama 3.3 (instruction-tuned). Below are practical details and best practices to get predictable, schema-conformant outputs when running the RAG + extraction pipeline.

- Model
  - Name: Groq Llama 3.3 (instruction-tuned)
  - Typical release size referenced in this repo: 70B parameters (check the provider for exact artifact names)
- Access & Licensing
  - Groq/Meta models may have specific access and license terms. Make sure you have the appropriate rights to download or run the model in your environment. If using a hosted offering (Groq Cloud or other providers), follow their usage terms.
- Deployment & Hardware
  - 70B-class models typically require high-memory accelerators for full-precision inference. Options:
    - Hosted inference (recommended for most users): Groq Cloud or other inference providers.
    - Local inference: Use quantized weights (4-bit/8-bit) and a large-memory GPU/accelerator. Hardware requirements vary by quantization and runtime — verify with your chosen runtime.
- Inference Settings (recommended defaults for structured extraction)
  - temperature: 0.0 — 0.2 (lower is better for deterministic, numeric outputs)
  - top_p: 0.9 — 0.95
  - max_tokens: 256 — 1024 (depending on the expected JSON payload)
  - repetition_penalty: 1.0
  - batch size: keep small for stable outputs
- Prompting & Safety
  - Use a strict system prompt that instructs the model to output only valid JSON conforming to the schema (no extra commentary).
  - Include the JSON Schema in the prompt or as part of the system instruction to improve compliance.
  - If the model returns invalid JSON, use the project's built-in post-processing / crash protection to extract numbers/units and build fallback responses.
- Example System Prompt (short)
```text
You are a technical extraction assistant. Given the retrieved manual text fragments and page metadata, output exactly one valid JSON object that matches this schema: { "component": string, "spec_type": string, "value": number, "unit": string, "source": { "pdf": string, "page": integer } }. Do NOT output any explanation, markdown, or extra fields. If unsure, output null for missing fields.
```

🧪 Prompt engineering tips
- Provide the model with top-k relevant chunks + explicit examples of desired JSON outputs.
- Use few-shot examples (2–3) in the prompt showing exact JSON the system should return.
- Force deterministic settings (temperature near 0) for numeric extraction tasks.

📂 Project Structure
Rag-Based--Vehicle-Specification-Extraction-System/
│
├── faiss_db_index/             
├── sample-service-manual.pdf    
├── vehicle_specs.json           
├── final.ipynb                  
├── README.md
└── .gitignore

⚙️ Installation
Clone the Repository
git clone https://github.com/<user>/Rag-Based--Vehicle-Specification-Extraction-System.git
cd Rag-Based--Vehicle-Specification-Extraction-System

Install Dependencies
pip install -r requirements.txt

Run Notebook

Open and run:

final.ipynb

📘 Sample Query & Output
Input

“What is the torque for the rear suspension arm bolt?”

Output
{
  "component": "Rear Suspension Arm Bolt",
  "spec_type": "Torque",
  "value": 155,
  "unit": "Nm",
  "source": {
    "pdf": "sample-service-manual.pdf",
    "page": 118
  }
}

🔮 Future Roadmap
Feature	Priority	Status
Table-Aware Chunking	⭐⭐⭐⭐⭐	Complete
Hybrid Search (BM25 + FAISS)	⭐⭐⭐⭐	Planned
Vision-Based Spec Extraction	⭐⭐⭐	Backlog
Full Streamlit UI	⭐⭐⭐⭐	Planned
FastAPI Backend	⭐⭐⭐	Planned
Camelot-First Pipeline	⭐⭐⭐⭐⭐	In Progress

🤝 Contributing

Pull requests are welcome!

📄 License

MIT License

❤️ Credits

HuggingFace Transformers

FAISS (Meta AI)

Sentence Transformers

LangChain

Groq Llama 3.3
