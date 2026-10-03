# CodeGenie

An automated AI engine written in **Python** that connects multiple Large Language Model (LLM) providers with a specialized context retrieval system (using **LlamaIndex** and **Milvus**) to process user requests, retrieve document context, and send intelligent responses.

<img width="967" height="683" alt="image" src="https://github.com/user-attachments/assets/12828505-bf8e-4568-8239-07ed9279d209" />

---

### 1. External Data Ingestion (Bottom Layer)

Before answering user requests, the system builds its knowledge base (Retrieval-Augmented Generation / RAG):

* **Data Connectors:** CodeGenie uses **LlamaHub** (part of LlamaIndex) to pull raw data and code from sources like **GitHub** or local **Directory Paths**.


* **Index & Embeddings:** LlamaHub creates data indexes and converts text into vector embeddings.


* **Vector Database (Milvus):** These computed vector embeddings are stored in **Milvus** (running inside a Docker container) for fast similarity searches.



---

### 2. User Request & Execution Flow (Steps 1 – 10)

1. **User Inputs (1 & 2):** An **Actor** (developer or user) sends a command via a terminal/CLI interface. This triggers an internal **Workflow** inside LlamaIndex.


2. **Context Query (3, 5, 6):** The workflow queries the **Milvus Vector Database** to look up relevant files, code snippets, or architectural context related to the user's input.


3. **Context Response (7 & 8):** Milvus returns the matching context chunks back to the **LlamaIndex Workflows** core, which passes it up to the **Prompt Engine**.


4. **Prompt Building & LLM Routing (8):**
* The **Prompt Engine** uses **LangChain** abstractions to combine the user's initial prompt with the retrieved context from Milvus.


* It routes the combined prompt to the chosen LLM backend along with platform-specific parameters (like model name).




5. **LLM Provider Options (Top Layer):**
* **EPAM DIAL:** Enterprise AI API gateway routing to Vertex AI, Azure OpenAI, or AWS Bedrock.


* **Direct Cloud AI Services:** Direct connections to **Google Cloud Vertex AI**, **Microsoft Azure OpenAI**, or **Amazon Bedrock**.




6. **Final Output (9 & 10):** The LLM model returns the generated answer (e.g., code, architecture explanation) back through LlamaIndex Workflows directly to the **Actor**.



---

### 3. Log Analytics (Bottom Left)

* **Log Analytics Job:** Runs alongside the primary application flow. It parses **User Application Logs** and uses **Matplotlib** to generate visualizations, metrics, or diagnostic reports.



---

### Summary Table

| Component | Technology | Role in Architecture |
| --- | --- | --- |
| **Orchestration Core** | **LlamaIndex** & **LangChain** | Manages multi-step RAG workflows and prompt assembly.

 |
| **Data Ingestion** | **LlamaHub Connectors** | Pulls code and documents from GitHub and local paths.

 |
| **Vector Database** | **Milvus** (Docker) | Stores and retrieves vector embeddings for context queries.

 |
| **LLM Gateway** | **EPAM DIAL / Cloud APIs** | Connects to Google Vertex AI, Azure OpenAI, and AWS Bedrock.

 |
| **Analytics** | **Matplotlib** | Visualizes log analytics data from application runs.

 |
