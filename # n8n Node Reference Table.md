# n8n Node Reference Table

Workflow 2: PDF-based RAG system | Workflow 3: AI agent chat — 12 nodes total

**Legend:**
- 📸 Confirmed from screenshot
- 🧠 n8n knowledge (assumption)
- ⚠️ Error state

---

## 🟢 Workflow 2 — Build a PDF-based RAG system (OpenAI · Pinecone · Cohere reranking)

| Node | Type | Parameters | Settings | What It Does | JSON Input | JSON Output | Key Transformation |
|------|------|------------|----------|--------------|------------|-------------|-------------------|
| **On Form Submission** `Trigger` | Entry point | Form fields, Form path 🧠 | Response mode 🧠 | Starts the workflow when a user submits a form with a PDF file | HTTP multipart form data 🧠 | `mimetype: "application/pdf"` 📸 · `size: 2180872` 📸 · `submittedAt: "2026-05-13T11:10:59..."` 📸 · `formMode: "test"` 📸 | HTTP form → n8n binary data item |
| **Default Data Loader** `Data` ⚠️ Error | Sub-node | Data type (binary), MIME type 🧠 | Loader type: PDF 🧠 | Loads a binary file (PDF) and converts it into a processable document. Currently broken due to pdf-parse v2 incompatibility | Binary PDF item from trigger 📸 | ⚠️ `"Failed to load pdf-parse. Please install pdf-parse@^1"` 📸 | Binary file → document text (blocked by error) |
| **Recursive Character Text Splitter** `Data` | Sub-node | Chunk size, Chunk overlap, Separators 🧠 | Language / mode 🧠 | Splits document text into overlapping chunks recursively: paragraph → sentence → word → character | Document text from Data Loader 🧠 | Array of text chunks with metadata (chunk index, source) 🧠 | 1 document → N chunks |
| **Embeddings OpenAI** `AI` | Sub-node | Model (text-embedding-3-small), Dimensions 🧠 | OpenAI credential 🧠 | Converts text chunks into vector embeddings via the OpenAI Embeddings API | Text strings (chunks) 🧠 | Float array (e.g. 1536 dimensions) per chunk 🧠 | Text → numerical vector representation |
| **Pinecone Vector Store** `Vector` | Main | Index name, Namespace, Operation (insert / query) 🧠 | Pinecone credential, Metric: cosine 🧠 | Stores embedding vectors in Pinecone (insert mode) or retrieves the most similar chunks for a query (query mode) | Vectors + source metadata 🧠 | Insert: confirmation / Query: top-K chunks with similarity scores 🧠 | Vectors → persistent index / index → ranked results |
| **Reranker Cohere** `AI` | Sub-node | Model (rerank-english-v3.0), Top N, Query 🧠 | Cohere credential 🧠 | Re-scores the top-K retrieved chunks by relevance to improve precision before passing context to the LLM | Candidate chunks array + query string 🧠 | Re-ordered top-N chunks with relevance scores 🧠 | K retrieved chunks → N reranked chunks (K > N) |

---

## 🟣 Workflow 3 — AI agent chat (gpt-4o-mini · Simple Memory · SerpAPI)

| Node | Type | Parameters | Settings | What It Does | JSON Input | JSON Output | Key Transformation |
|------|------|------------|----------|--------------|------------|-------------|-------------------|
| **When Chat Message Received** `Trigger` | Entry point | Session ID, Message 🧠 | Chat mode (built-in) 🧠 | Starts the workflow when a user sends a message via the built-in chat interface | UI chat event 🧠 | `action: "sendMessage"` 📸 · `sessionId: "b532e2a5ebac..."` 📸 · `chatInput: "hi how to use this workflow?"` 📸 | UI event → n8n trigger item |
| **AI Agent** `AI` | Main | Chat Model*, Memory*, Tool* (sub-nodes) 📸 | System prompt, Max iterations 🧠 | Orchestrates the conversation loop: receives message, decides when to call tools (SerpAPI), maintains memory, generates response via LLM | Chat message + conversation history 🧠 | Agent response (text) + tool call logs 🧠 | Query → (retrieve → reason → respond) loop |
| **OpenAI Chat Model** `AI` | Sub-node | Model: `gpt-4o-mini` 📸, Temperature, Max tokens 🧠 | OpenAI account 📸 | LLM backbone of the AI Agent. Handles reasoning and response generation | Messages array (system + user + assistant turns) 🧠 | Assistant message + token usage metadata 🧠 | Prompt → completion |
| **Simple Memory** `AI` | Sub-node | Context window length, Session key 🧠 | Memory type: buffer 🧠 | Stores conversation history in memory for the current session, giving the agent context across turns. 2 items active 📸 | New message to append 🧠 | Full conversation history array 🧠 | Stateless call → stateful conversation window |
| **SerpAPI** `Tool` | Sub-node (Tool) | Query (dynamically set by agent), Engine: google 🧠 | SerpAPI credential 📸 | Gives the agent access to live Google search results. The agent autonomously decides when to trigger a search based on the user query | Search query string from agent 🧠 | Array of results: title, link, snippet per item 🧠 | Question → live web context → informed answer |