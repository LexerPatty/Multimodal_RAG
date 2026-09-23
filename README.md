# Multimodal_RAG
Build a production-style pipeline that can understand and retrieve information from text, tables, images, charts, graphs, and diagrams inside PDF documents



![Python](https://img.shields.io/badge/Python-3.10%2B-blue?style=for-the-badge&logo=python&logoColor=white)
![LangChain](https://img.shields.io/badge/LangChain-Framework-green?style=for-the-badge)
![Pinecone](https://img.shields.io/badge/Pinecone-VectorDB-teal?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-orange?style=for-the-badge)

A comprehensive, production-ready Multimodal Retrieval-Augmented Generation (RAG) framework designed to extract, index, retrieve, and reason across complex enterprise documents containing **Text**, **Structured Tables**, **Images**, and **Analytical Graphs**.

---

## 📌 Executive Summary & Motivation

Traditional RAG pipelines focus primarily on raw text chunks. However, real-world enterprise documents (financial reports, technical manuals, market research) mix multiple content types on the same page. Standard plain-text extraction often fails by:
- Flattening structured tables and losing key column/row relationships.
- Dropping essential data encoded inside charts, trends, and pie graphs.
- Ignoring visual flowcharts, architectural diagrams, and process workflows.

**Multimodal RAG** addresses this by converting all content types into retrievable embeddings while retaining the original visual evidence for final synthesis.

---

## 📐 End-to-End System Architecture

```
                               ┌─────────────────────────────────────────┐
                               │             PDF Ingestion               │
                               └────────────────────┬────────────────────┘
                                                    │
                 ┌──────────────────────────────────┼──────────────────────────────────┐
                 ▼                                  ▼                                  ▼
        ┌─────────────────┐                ┌─────────────────┐                ┌─────────────────┐
        │   Text Extraction│                │ Table Extraction│                │ Visual Extraction│
        │   (PyMuPDF)     │                │ (Markdown Format)│                │ (Images/Charts) │
        └────────┬────────┘                └────────┬────────┘                └────────┬────────┘
                 │                                  │                                  │
                 │                                  │                                  ▼
                 │                                  │                         ┌─────────────────┐
                 │                                  │                         │  Vision Model   │
                 │                                  │                         │(Visual Summary) │
                 │                                  │                         └────────┬────────┘
                 │                                  │                                  │
                 └──────────────────┬───────────────┴──────────────────────────────────┘
                                    │
                                    ▼
                         ┌────────────────────┐
                         │ Text Embedding     │
                         │ Model              │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │  Pinecone Vector   │
                         │     Database       │
                         └─────────┬──────────┘
                                   │
                                   ▼
                         ┌────────────────────┐
                         │ Similarity Search  │
                         │   (User Query)     │
                         └─────────┬──────────┘
                                   │
                  ┌────────────────┴────────────────┐
                  ▼                                 ▼
       [Visual Artifact Found?]          [Text/Table Only]
                  │                                 │
         YES      ▼                        NO       ▼
        ┌──────────────────┐               ┌──────────────────┐
        │ Vision Language  │               │     Text LLM     │
        │      Model       │               │                  │
        └────────┬─────────┘               └────────┬─────────┘
                 │                                  │
                 └──────────────────┬───────────────┘
                                    │
                                    ▼
                         ┌────────────────────┐
                         │ Final Answer +     │
                         │ Source Citation    │
                         └────────────────────┘
```

---

## ⚙️ How It Works: Step-by-Step Pipeline

### Step 1: Modality Extraction & Normalization
When a document is uploaded, it is parsed into three isolated channels:
1. **Paragraphs & Text:** Extracted directly using page-aware metadata.
2. **Tables:** Extracted and converted into Markdown tables to preserve row-column alignments.
3. **Visuals (Charts & Diagrams):** Extracted as raw image files and routed to a Vision Model to generate a descriptive, factual summary.

### Step 2: Vector Store Ingestion
All three modalities (Text, Table Markdown, and Visual Summaries) are processed through a single **Text Embedding Model**. 
- Vectors are indexed into a unified **Pinecone Namespace**.
- Metadata attached to each vector records the original page number, modality tag (`text`, `table`, or `visual`), and local image reference path.

### Step 3: Adaptive Query Routing & Answer Generation
1. **Retrieval:** The user query is embedded and matched against Pinecone to fetch the `Top-K` relevant matches.
2. **Routing Logic:**
   - **Text & Table Context:** If no visual components are retrieved, the context is sent to a Standard Text LLM.
   - **Visual Context:** If a visual element is retrieved, its image file path is pulled from the metadata, and both the original raw image and text context are passed to a Vision Language Model (VLM).

---

## 📸 Multimodal Data Processing Examples

Below are representative processing examples across different modalities found within enterprise reports.

### Example 1: Text Retrieval
* **Input Query:** *"What was NovaCore's FY2026 revenue?"*
* **Retrieved Content:** Page 2 paragraph narrative.
* **Result:** `$132.0 Million` (Source: Page 2).

---

### Example 2: Structured Table Processing

| Region | 2026 Revenue | YoY Growth | Customers | CSAT |
| :--- | :--- | :--- | :--- | :--- |
| **Europe** | $41.2M | 31% | 118 | 4.7/5 |
| **North America** | $38.6M | 27% | 104 | 4.6/5 |
| **Asia Pacific** | $34.9M | 24% | 96 | 4.8/5 |
| **Middle East** | $17.3M | 18% | 51 | 4.5/5 |

* **Input Query:** *"Which region grew the fastest and which had the highest customer satisfaction (CSAT)?"*
* **System Processing:** Preserves table structure during indexing, allowing precise cross-column alignment.
* **Result:** **Europe** had the highest growth rate (**31% YoY**), while **Asia Pacific** achieved the highest CSAT rating (**4.8/5**).

---

### Example 3: Graph & Trend Analysis

```
  Revenue (USD Millions)
   38 │                                                  ● (Q4: 37.8M)
   34 │                                       ●
   30 │                            ●
   26 │               ●
   22 │  ●
      └────────────────────────────────────────────────────────
        Q1 2025    Q3 2025      Q1 2026    Q3 2026      Q4 2026
```

* **Input Query:** *"Which quarter had the highest revenue, and what was the trend?"*
* **Indexing Strategy:** The chart image is summarized visually into text for semantic search in Pinecone. The raw image is fetched at inference time for exact measurement reading by the VLM.
* **Result:** **Q4 2026** reached the peak revenue at approximately **$37.8M**, demonstrating continuous quarter-over-quarter growth throughout 2025 and 2026.

---

### Example 4: Process & Diagram Reasoning

```
┌───────────────────┐     ┌───────────────────┐     ┌───────────────────┐     ┌───────────────────┐
│ Component         │────>│ Assembly          │────>│ Quality Lab       │────>│ Regional Hubs     │
│ Suppliers         │     │ (Penang, Malaysia)│     │ (Singapore)       │     │ (Rotterdam/Dubai) │
└───────────────────┘     └───────────────────┘     └─────────┬─────────┘     └───────────────────┘
                                                              │
                                                    [CRITICAL CONTROL POINT]
                                                   (Thermal & Firmware Tests)
```

* **Input Query:** *"Where is the critical quality-control point in the supply chain?"*
* **System Processing:** Graph nodes and connection arrows are interpreted by the visual model.
* **Result:** The **Quality Lab in Singapore**, where every batch must complete thermal, network, and firmware testing prior to regional hub export.

---

### Example 5: Pie Chart Proportion Reasoning

```
                Penang Facility Electricity Mix (2026)
                
                     ┌───────────────────────┐
                     │  ■ Grid (51%)         │
                     │  ■ Solar (34%)        │
                     │  ■ Renewables (15%)   │
                     └───────────────────────┘
```

* **Input Query:** *"What percentage of Penang facility electricity was supplied by solar power?"*
* **Result:** **34%** of electricity was generated from solar sources, representing the second largest energy contributor behind grid electricity (51%).

---

## 🌟 Key Advantages

1. **Unified Vector Namespace:** Simplifies vector storage by managing all document formats within one vector index.
2. **Zero Loss of Visual Evidence:** Avoids relying solely on lossy text descriptions by passing raw images directly to Vision Models when needed.
3. **Adaptive Routing:** Saves computational costs by triggering multimodal models only when retrieved items contain visual artifacts.
4. **Metadata Grounding:** Every generated answer includes references back to specific page numbers, tables, or visual figures.

---

## 📄 License

Distributed under the MIT License. See `LICENSE` for more information.