# 🚗 Intelligent Vehicle Specification Extraction System (Mechanic AI)

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?style=for-the-badge&logo=python)
![Streamlit](https://img.shields.io/badge/Streamlit-FF4B4B?style=for-the-badge&logo=Streamlit)
![LangChain](https://img.shields.io/badge/LangChain-1C3C3C?style=for-the-badge&logo=LangChain)
![Groq](https://img.shields.io/badge/Groq-Llama3-orange?style=for-the-badge)

## 📌 Project Overview
**Mechanic AI** is a specialized Retrieval-Augmented Generation (RAG) system developed to automate the extraction of technical specifications from automotive service manuals. 

Service manuals are often hundreds of pages long, making it time-consuming for mechanics to locate specific values like torque settings, fluid capacities, or alignment angles. This tool allows users to ask natural language questions (e.g., *"What is the torque for the front suspension?"*) and receive precise, structured data in a standardized JSON format.

---

## 🚀 Key Features

* **RAG Architecture:** Uses semantic vector search to retrieve the exact page from the manual before generating an answer.
* **Structured Data Extraction:** Enforces a strict schema (`Component` | `Spec Type` | `Value` | `Unit`) for all outputs.
* **Smart Unit Parsing:** Automatically handles complex multi-unit patterns found in manuals (e.g., parsing `Nm -> lb-ft -> lb-in` correctly).
* **Crash Protection:** Includes robust error handling to display readable text if the AI returns unstructured conversation instead of data.
* **Cloud Ready:** Fully optimized for deployment on Streamlit Community Cloud with secure API key management.

---

## 🏗️ System Architecture

The system follows a standard RAG pipeline:

1.  **Ingestion:** The PDF manual is split into chunks and converted into vector embeddings using **HuggingFace** models (`all-MiniLM-L6-v2`).
2.  **Storage:** These vectors are stored in a local **FAISS** index (the "Brain").
3.  **Retrieval:** When a user queries the app, the system searches FAISS for the top 3 most relevant context chunks.
4.  **Generation:** The context and query are sent to **Groq Llama 3**, which extracts the specifications into JSON format.

---
## 🧠 Technical Deep Dive:
### 🧩 Chunking Strategy
To optimize retrieval accuracy for technical tables and long specifications, the system employs a custom **Recursive Character Text Splitting** strategy.

* **Method:** `RecursiveCharacterTextSplitter`
* **Chunk Size:** 2000 characters (Optimized to keep entire small tables within a single context window).
* **Overlap:** 300 characters (Ensures critical context isn't lost at the boundaries of a split).
* **Separators:** `["\n\n", "\n", " ", ""]`
    * *Logic:* Prioritizes keeping paragraphs and table rows together by splitting on double newlines first, then single newlines.
###  Vector Search Strategy
While the current implementation uses a **Flat Index (Exact Search)** for maximum accuracy on standard service manuals, the system architecture is designed to scale using advanced FAISS indexing strategies:

* **Distance Metric:** Uses **Euclidean Distance (L2)** to calculate the precise semantic similarity between the user's query and the manual's text chunks.
* **Scalability Options:**
    * **HNSW (Hierarchical Navigable Small World):** For larger datasets (e.g., thousands of manuals), the system can switch to HNSW graph-based indexing to maintain millisecond retrieval speeds.
    * **IVF (Inverted File Index):** Capable of clustering vector spaces (Voronoi cells) to reduce search scope and memory usage for production-grade deployments.
    * 

## 🛠️ Tech Stack

| Component | Tool | Purpose |
| :--- | :--- | :--- |
| **Frontend** | **Streamlit** | Web interface and interaction logic. |
| **LLM Engine** | **Groq API** | Ultra-fast inference using Llama 3.3 (70B). |
| **Vector DB** | **FAISS** | Similarity search for document retrieval. |
| **Orchestration** | **LangChain** | Managing the retrieval and prompt chains. |

---

## 🚀 Future Roadmap & Architectural Improvements

To enhance the robustness and accuracy of Mechanic AI, the following high-impact features are prioritized for future releases.

| Feature | Impact | Status |
| :--- | :--- | :--- |
| **Context-Aware Chunking** | ⭐⭐⭐⭐⭐ (High) | ![Planned](https://img.shields.io/badge/Status-Planned-blue?style=flat-square) |
| **Hybrid Search (BM25)** | ⭐⭐⭐⭐ (Medium) | ![Research](https://img.shields.io/badge/Status-Researching-yellow?style=flat-square) |
| **Vision (Multi-Modal)** | ⭐⭐⭐ (Medium) | ![Backlog](https://img.shields.io/badge/Status-Backlog-lightgrey?style=flat-square) |

---

### 📄 1. Context-Aware Table Chunking (Header Injection)
> **The Problem:** Standard RAG splitters cut large tables into pieces, separating data rows from their headers. The AI sees "17" but forgets if it's "Torque" or "Quantity."

* **Proposed Solution:** Implement a pre-processing pipeline to **inject headers into every row**.
* **Transformation Logic:**
    * *Raw:* `| Bolt | 17 |`
    * *Injected:* `Component: Bolt | Spec: Torque | Value: 17 Nm`
* **Result:** Guarantees **100% context retention** for every data point, regardless of document splitting.

---

### 🔍 2. Hybrid Search Architecture
> **The Problem:** Vector search is great for concepts but struggles with exact "keyword" matches like specific Part Numbers (e.g., `#99-X-200`).

* **Proposed Solution:** Combine **BM25 (Keyword Search)** with **FAISS (Semantic Search)** using Reciprocal Rank Fusion (RRF).
* **Result:** The system becomes "bilingual"—able to understand broad questions *and* find exact part numbers instantly.

---

### 👁️ 3. Multi-Modal Vision Capabilities
* **Proposed Solution:** Integrate **Llama 3.2 Vision**.
* **User Story:** A mechanic uploads a photo of a rusted bolt. The AI identifies the component visually and automatically retrieves its torque spec without the user typing a word.

---

### 📑 4. Advanced PDF Parsing (Camelot/Tabula)
* **Proposed Solution:** Move beyond simple text extraction. Use **Camelot** to extract tables directly into Pandas DataFrames before embedding.
