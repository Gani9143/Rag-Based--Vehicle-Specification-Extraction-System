# 🚗 AI Mechanic: RAG-Based Vehicle Specification Extraction System

An AI-Powered Mechanic Assistant for automatic technical-spec retrieval from vehicle service manuals using Retrieval-Augmented Generation (RAG).

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue?style=for-the-badge&logo=python)](https://www.python.org/)
[![FAISS](https://img.shields.io/badge/FAISS-Vector%20DB-orange?style=for-the-badge)](https://github.com/facebookresearch/faiss)
[![LangChain](https://img.shields.io/badge/LangChain-RAG-green?style=for-the-badge)](https://github.com/langchain-ai/langchain)
[![Groq](https://img.shields.io/badge/Groq-Llama3.3-ff9900?style=for-the-badge)](https://www.groq.com/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=for-the-badge)](https://opensource.org/licenses/MIT)

---

## 📚 Table of Contents
* [📌 Overview](#-overview)
* [✨ Key Features](#-key-features)
* [🛠 How it Works (RAG Flow)](#-how-it-works-rag-flow)
* [🧩 Design Choices & Extraction Rules](#-design-choices--extraction-rules)
* [🧪 Examples & Expected Outputs](#-examples--expected-outputs)
* [⚙️ Installation](#%EF%B8%8F-installation)
* [🔧 Configuration & Best Practices](#-configuration--best-practices)
* [🔮 Roadmap](#-roadmap)
* [🤝 Contributing](#-contributing)
* [📄 License & Credits](#-license--credits)

---

## 📌 Overview

Mechanics and technicians frequently spend valuable time searching dense, lengthy service manuals for a single, critical piece of data—be it a **torque specification**, a **fluid capacity**, or a **clearance dimension**.

This project automates the retrieval and strict JSON extraction of these technical specifications using a **Retrieval-Augmented Generation (RAG) pipeline**. By leveraging an instruction-tuned LLM (Groq's Llama 3.3), the system delivers deterministic, actionable data directly from complex PDF documents.

### **Quick Demo**

| User Query | Extracted JSON Output |
| :--- | :--- |
| **"What is the torque for the front lower control arm bolt?"** | ```json{ "component": "Front Lower Control Arm", "spec_type": "Torque", "value": 125, "unit": "Nm", "source": {"pdf": "sample-service-manual.pdf", "page": 142}}``` |

---

## ✨ Key Features

* **Full RAG Stack:** Utilizes **FAISS** for rapid vector retrieval combined with **Groq's Llama 3.3** for ultra-fast, accurate extraction.
* **Strict JSON Output:** Enforces a rigid output schema (`component`, `spec_type`, `value`, `unit`, `source`) for seamless integration with downstream systems.
* **Table-Aware Ingestion:** Uses specialized PDF parsers (Camelot / Tabula) combined with **Header Injection** during chunking to ensure data from tables remains contextually meaningful.
* **Smart Unit Normalization:** Automatically recognizes and normalizes multi-unit entries (e.g., SI ↔ Imperial, `Nm` to `lb-ft`).
* **Crash Protection:** Includes fallback logic to attempt regex-based extraction if the LLM produces invalid JSON, maximizing data recovery.
* **Demo Ready:** Low-memory FAISS setup and an integrated Streamlit frontend for quick, interactive demonstrations.

---

## 🛠 How it Works (RAG Flow)

The system follows a standard RAG pattern, optimized for parsing and retrieving technical data. The key innovation lies in the **Table-Aware Chunking** step.

### **High-Level Architecture Flow**

| Stage | Process | Key Components |
| :--- | :--- | :--- |
| **1. Document Ingestion** | Extracts text and structured data (tables) from PDFs. | PDF Miner, Camelot/Tabula |
| **2. Preprocessing** | Splits documents into chunks, **injecting headers** into table chunks for retained context. | RecursiveCharacterTextSplitter, Custom Table Processor |
| **3. Embedding** | Converts text chunks into vector representations. | `all-MiniLM-L6-v2` |
| **4. Storage** | Indexes the vectors and metadata (page number, source) for fast lookup. | **FAISS Index** (Flat L2) |
| **5. Retrieval** | The User Query is embedded and used to retrieve the most relevant `Top-k` chunks. | Retriever (FAISS Search) |
| **6. Generation** | The query and the retrieved chunks are sent to the LLM for structured analysis. | **Groq Llama 3.3** |
| **7. Post-Processing** | Validates the JSON schema, normalizes units, and prepares the final output. | Custom Unit Parser, JSON Validator |

---

## 🧩 Design Choices & Extraction Rules

We've selected specific parameters to maximize accuracy and determinism for technical specification extraction:

* **Chunking:**
    * **Chunk Size:** 2000 characters
    * **Overlap:** 300 characters
    * **Splitter:** `RecursiveCharacterTextSplitter` with high-priority separators `["\n\n", "\n"]`.
* **Embeddings:** `all-MiniLM-L6-v2` (Dimension: 384)
* **Vector DB:** Default FAISS index is **Flat (L2)** for maximum accuracy on small/medium datasets, prioritizing precision over index build time.
* **LLM Inference:**
    * **Temperature:** $0.0 - 0.2$ (low to ensure deterministic, numeric output)
    * **`top_p`:** $0.9 - 0.95$

**Metadata Example:** Vectors are tagged with essential source data.
```json
{
  "page": 142,
  "chunk_id": "c142_3",
  "source": "sample-service-manual.pdf"
}
