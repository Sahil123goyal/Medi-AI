# Medi-AI — Biochemistry Study Assistant 🧬

A Retrieval-Augmented Generation (RAG) chatbot that answers biochemistry questions for medical students. It retrieves relevant passages from a biochemistry textbook using a Pinecone vector store and generates grounded, accurate answers with a Groq-hosted LLaMA 3.3 model — served through a Flask web app.

## Features

- 📚 Answers grounded in a real biochemistry textbook, not the model's raw memory
- 🔍 Semantic search over textbook content using sentence-transformer embeddings
- ⚡ Fast inference via Groq's LLaMA 3.3 70B model
- 💬 Simple, themed chat UI (Flask + Bootstrap)
- 🧠 If the answer isn't in the textbook, the assistant says so instead of guessing

## Tech stack

| Layer | Technology |
|---|---|
| LLM | Groq — `llama-3.3-70b-versatile` |
| Orchestration | LangChain (`create_retrieval_chain`, `create_stuff_documents_chain`) |
| Embeddings | HuggingFace `sentence-transformers/all-MiniLM-L6-v2` |
| Vector store | Pinecone (serverless, cosine similarity, 384 dims) |
| Backend | Flask |
| Frontend | HTML, CSS, Bootstrap, jQuery |

## How it works

1. PDFs in `data/` are loaded and split into overlapping chunks.
2. Each chunk is embedded with `all-MiniLM-L6-v2` and upserted into a Pinecone index (`store_index.py`, run once).
3. At query time, the top-k most similar chunks are retrieved from Pinecone.
4. The chunks are passed as context to LLaMA 3.3 via a RAG prompt, which answers using only that context.
5. The answer is returned to the chat UI (`app.py`).

## Project structure

```
Medi-AI/
├── data/                      # Source PDFs (textbook content)
├── src/                       # Core package: loaders, splitting, embeddings, prompt
├── research/                  # Exploratory notebooks (trials.ipynb)
├── static/                    # CSS for the chat UI
├── templates/                 # chat.html
├── genai/                     # (project-specific module)
├── medibot/                   # (project-specific module)
├── Demo_photos/               # Screenshots / demo images
├── app.py                     # Flask app — serves the chatbot
├── store_index.py             # One-time script: build/populate the Pinecone index
├── setup.py                   # Package setup
├── pyproject.toml
├── requirements.txt
├── template.sh                # Project scaffolding script
├── .env                       # API keys (not committed)
├── .gitignore
└── LICENSE
```

## Setup

### 1. Clone and create an environment

```bash
git clone <your-repo-url>
cd Medi-AI
python -m venv myenv
source myenv/bin/activate      # Windows: myenv\Scripts\activate
pip install -r requirements.txt
```

### 2. Configure environment variables

Create a `.env` file in the project root:

```
PINECONE_API_KEY=your_pinecone_api_key
GROQ_API_KEY=your_groq_api_key
```

### 3. Add your source documents

Place the textbook PDF(s) inside the `data/` folder.

### 4. Build the vector index (run once)

```bash
python store_index.py
```

This loads the PDFs, chunks them, generates embeddings, and upserts them into a Pinecone index named `medical-chatbot`.

> ⚠️ Only run this when you want to (re)build the index — it re-embeds and re-upserts every time it runs. Re-running it repeatedly without clearing the index first will create duplicate vectors.

### 5. Run the app

```bash
python app.py
```

Visit `http://localhost:8080` in your browser.

## Notes

- `app.py` connects to the **existing** Pinecone index — it does not modify it.
- If you update the source PDFs, clear the Pinecone index and re-run `store_index.py` to rebuild it from scratch.

## License

See [LICENSE](./LICENSE) for details.
