# 🚗 RAG-Based Vehicle Specification Extraction System
_ AI-Powered Mechanic Assistant for Automatic Spec Retrieval_

![Python](https://img.shields.io/badge/Python-3.9%2B-blue?style=for-the-badge&logo=python)
![FAISS](https://img.shields.io/badge/FAISS-Vector%20DB-orange?style=for-the-badge)
![LangChain](https://img.shields.io/badge/LangChain-RAG-green?style=for-the-badge)
![Groq](https://img.shields.io/badge/Groq-Llama3.3-ff9900?style=for-the-badge)

---

## 📌 Overview
Modern automotive service manuals are hundreds of pages long, making it time-consuming for mechanics to find torque values, fluid quantities, bolt specs, alignment angles, or part numbers.

**RAG-Based Vehicle Specification Extraction System** automates this task. It uses a Retrieval-Augmented Generation pipeline (FAISS + LLM) to extract precise technical specifications from service manuals and returns them in a **strict JSON schema**.

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
