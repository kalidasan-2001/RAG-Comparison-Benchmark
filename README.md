# LangChain vs LlamaIndex: RAG Comparison Project

## 📖 Project Overview
This project is a hands-on comparison between two of the most popular frameworks for building **Retrieval-Augmented Generation (RAG)** applications: **LangChain** and **LlamaIndex**.

We build the exact same RAG pipeline in both frameworks using the same data, the same LLM (Groq/Llama-3), and the same embedding model to see which one performs better in terms of speed and quality.

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- A [Groq API Key](https://console.groq.com/) (Free)

### Installation
Create a virtual environment and install the required libraries:

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install dependencies
pip install langchain langchain-community langchain-groq llama-index llama-index-llms-groq llama-index-embeddings-huggingface faiss-cpu sentence-transformers numpy pandas matplotlib pypdf
```

---

## 🧠 Code Walkthrough (Cell by Cell)

Here is a detailed explanation of the logic inside the Jupyter Notebook.

### 1. Setup & Configuration
**Goal**: Initialize the environment and load API keys.
- **Logic**: We check if the `GROQ_API_KEY` is set. We also define global constants like `CHUNK_SIZE` (500 chars) to ensure both frameworks split text the same way.
- **Key Tech**: `os.environ` for secrets management.

### 2. Data Loading
**Goal**: Read text from PDF files.
- **Logic**: We use `pypdf` to extract text from all PDF files in the `data/` folder.
- **Result**: A list of raw text strings (`raw_docs`), where each string represents the content of one PDF.

### 3. Shared Models (The "Fairness" Layer)
**Goal**: Ensure a fair fight.
- **Logic**: We initialize the **same** Embedding Model (`HuggingFaceEmbeddings`) and the **same** LLM (`ChatGroq` / `LlamaGroq`) for both pipelines.
- **Why?**: If we used different models, we wouldn't know if the performance difference came from the framework or the model.

### 4. ⛓️ The LangChain Pipeline (The "Manual" Approach)
**Goal**: Build a RAG system using LangChain's modular blocks.
- **Step 1: Splitting**: We use `RecursiveCharacterTextSplitter` to chop text into 500-character chunks.
- **Step 2: Vector Store**: We use **FAISS** (Facebook AI Similarity Search). This is a C++ optimized library that is extremely fast at finding similar text.
- **Step 3: Chain**: We create a `retrieval_chain` that:
    1.  Takes the user query.
    2.  Searches FAISS for the top 3 chunks.
    3.  Pastes them into a simple prompt: `"Answer based on context: {context} Question: {input}"`.
    4.  Sends it to the LLM.

### 5. 🦙 The LlamaIndex Pipeline (The "Automated" Approach)
**Goal**: Build a RAG system using LlamaIndex's high-level abstractions.
- **Step 1: Ingestion**: It converts text into "Nodes" (smart chunks).
- **Step 2: Indexing**: We use `VectorStoreIndex`. By default, this creates a simple in-memory dictionary (Python list) to store vectors.
- **Step 3: Query Engine**: We create a query engine that:
    1.  Searches the index.
    2.  Uses a **Default Prompt** (which is long and detailed, containing instructions like "Context information is below...").
    3.  Synthesizes an answer.

### 6. Evaluation & Metrics
**Goal**: Measure performance scientifically.
We define a list of "Gold" questions and answers (what the AI *should* say) and run them through both pipelines.

#### The Metrics:
1.  **⏱️ Latency (Time)**:
    -   **Definition**: How many seconds it takes to get an answer.
    -   **Goal**: Lower is better.
2.  **🎯 Answer Quality (Cosine Similarity)**:
    -   **Definition**: We convert the AI's answer and the "Gold" answer into numbers (vectors) and calculate the angle between them.
    -   **Scale**: 0 to 1. (1.0 means the answers are semantically identical).
    -   **Goal**: Higher is better.
3.  **📝 Output Length**:
    -   **Definition**: Number of characters in the answer.
    -   **Goal**: Neutral (depends on preference).

---

## 📊 Results & Findings

In this specific demo, you will likely see **LangChain performing faster (Lower Latency)**. Here is why:

### Why was LangChain Faster?
1.  **Vector Store**: LangChain used **FAISS** (C++ optimized engine), while LlamaIndex defaulted to a **Simple Vector Store** (Python List), which is slower.
2.  **Prompt Size**: LangChain used a tiny, custom prompt (~10 words). LlamaIndex used its robust default prompt (~100+ words), which takes the LLM longer to process.
3.  **Overhead**: LangChain simply passed the text through. LlamaIndex performed extra "response synthesis" steps to ensure safety and formatting.

### Summary of Trade-offs

| Feature | LangChain 🦜🔗 | LlamaIndex 🦙 |
| :--- | :--- | :--- |
| **Philosophy** | "Lego Blocks" (Build it yourself) | "Smart Manager" (Handles it for you) |
| **Setup** | Verbose (Explicitly define splitters, stores, chains) | Concise (Just `VectorStoreIndex.from_documents`) |
| **Speed (Default)** | Fast (if using FAISS/Minimal chains) | Slower (due to robust default prompts) |
| **Best For** | Custom applications requiring fine-grained control | Rapid prototyping and connecting to complex data sources |

## 🏁 Conclusion
Both frameworks are powerful.
- Choose **LangChain** if you want to build a custom pipeline from scratch and control every single prompt and step.
- Choose **LlamaIndex** if you want to quickly ingest data and have a robust system working out-of-the-box without worrying about the low-level details.
