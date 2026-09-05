# RAG Q&A Chatbot with Message History

A conversational Retrieval-Augmented Generation (RAG) chatbot that lets you upload PDFs and ask questions about them — with full multi-turn chat memory, powered by LangChain and Groq's blazing-fast inference.

## Highlights

- **History-aware retrieval** — Uses LangChain's `create_history_aware_retriever` to rewrite follow-up questions (e.g. "what about the second one?") into standalone, context-complete queries before hitting the vector store. This is what makes multi-turn Q&A actually work, instead of just stuffing raw chat history into the prompt.
- **Session-based conversation memory** — Powered by `RunnableWithMessageHistory` with a per-`session_id` store, so you can run multiple independent conversations in the same app instance.
- **Multi-PDF ingestion** — Upload one or several PDFs at once; they're parsed, chunked, and embedded together into a single searchable knowledge base.
- **Fast open-weight inference** — Answers are generated using Groq's `openai/gpt-oss-120b` model via `ChatGroq`, giving low-latency responses even on longer contexts.
- **Local, on-the-fly embeddings** — Uses `sentence-transformers/all-MiniLM-L6-v2` via `langchain-huggingface`, so no external embedding API calls are needed.
- **Ephemeral vector store** — Chroma is initialized fresh per session directly from the uploaded documents (no persistent DB required to get started).

## How it works

1. Upload one or more PDFs through the Streamlit UI.
2. Documents are split into chunks (`chunk_size=5000`, `chunk_overlap=500`) and embedded with `all-MiniLM-L6-v2`.
3. Chunks are indexed in a Chroma vector store, exposed as a retriever.
4. On each user query:
   - The **history-aware retriever** rewrites the question using prior chat turns (if needed) into a standalone query.
   - Relevant chunks are retrieved and passed, along with the chat history, into a **stuff-documents** QA chain.
   - Groq's `gpt-oss-120b` generates a concise (≤3 sentence) answer grounded in the retrieved context.
5. The full exchange is stored in a session-scoped chat history (`ChatMessageHistory`), so follow-up questions retain context.

## Tech Stack

| Component | Tool |
|---|---|
| LLM | Groq (`openai/gpt-oss-120b`) via `langchain-groq` |
| Embeddings | HuggingFace `all-MiniLM-L6-v2` |
| Vector Store | Chroma |
| Orchestration | LangChain (`langchain-classic`, history-aware retriever + retrieval chain) |
| Document Loading | `PyPDFLoader` + `RecursiveCharacterTextSplitter` |
| UI | Streamlit |
| Env Management | `python-dotenv` |

