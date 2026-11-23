# NSU Assistant Chatbot

A Retrieval-Augmented Generation (RAG) chatbot for answering questions using information stored in a local text knowledge base. The project includes a FastAPI backend and a browser-extension-style chat popup.

## How It Works

1. The backend loads the knowledge base from `Server/vault.txt`.
2. It loads the saved embeddings from `Server/vault_embeddings.pkl`. If the embeddings file is missing, it generates embeddings for each line in the vault using Ollama's `mxbai-embed-large` model.
3. When a user sends a question, the backend creates an embedding for it and compares it with the stored embeddings using cosine similarity.
4. The three most relevant text entries are selected and added to the prompt as context.
5. The prompt is sent to the Hugging Face Inference API, using `mistralai/Mistral-7B-Instruct-v0.3`.
6. The generated answer is returned to the chat interface.

## Project Structure

```text
NSU_Assistant_Chatbot/
├── Client/
│   ├── logo.png
│   ├── manifest.json
│   ├── popup.html
│   ├── script.js
│   └── style.css
└── Server/
    ├── main.py
    ├── rag.py
    ├── vault.txt
    └── vault_embeddings.pkl
```

- `Client/` contains the chat popup interface.
- `Server/main.py` defines the FastAPI application and `/message/` endpoint.
- `Server/rag.py` implements embedding generation, retrieval, and response generation.
- `Server/vault.txt` contains the knowledge-base text (one entry per line).
- `Server/vault_embeddings.pkl` stores the precomputed embeddings.

## Requirements

- Python
- [Ollama](https://ollama.com/) with the `mxbai-embed-large` embedding model
- A Hugging Face access token with access to the Inference API model used by the project

## Setup

### 1. Get the project

```bash
git clone https://github.com/fidaweb/NSU_Assistant_Chatbot_.git
cd NSU_Assistant_Chatbot_
```

### 2. Install Python dependencies

```bash
cd Server
python -m venv .venv
```

Activate the virtual environment:

**Linux/macOS**
```bash
source .venv/bin/activate
```

**Windows**
```powershell
.venv\Scripts\activate
```

Install the packages used by the backend:

```bash
pip install fastapi uvicorn torch ollama openai python-dotenv requests
```

### 3. Set up Ollama

Install and start [Ollama](https://ollama.com/), then download the embedding model:

```bash
ollama pull mxbai-embed-large
```

Make sure the Ollama service is running before starting the backend.

### 4. Configure the Hugging Face token

Create a file named `.env` inside the `Server/` directory:

```env
HUGGING_FACE_KEY=your_hugging_face_token
```

Replace the placeholder with your own token. Keep `.env` private and do not commit it to Git.

## Run the Backend

Run these commands from the `Server/` directory so the application can find `vault.txt` and `vault_embeddings.pkl`:

```bash
uvicorn main:app --reload
```

The API will be available at:

```text
http://127.0.0.1:8000
```

The chat endpoint accepts a `POST` request at `/message/` with JSON in this format:

```json
{
  "msg": "Your question here"
}
```

It returns:

```json
{
  "message": "The chatbot response"
}
```

## Load the Client

The `Client/` folder contains a browser extension popup.

1. Open your browser's extension management page.
2. Enable developer mode.
3. Choose the option to load an unpacked extension.
4. Select the `Client/` folder.

Keep the backend running while using the popup. The client currently sends requests to `http://127.0.0.1:8000/message/`, so it expects the backend to run on the same computer.

## Notes

- The first run may take longer because the backend generates and saves embeddings if `vault_embeddings.pkl` is not available.
- The knowledge base is read from `vault.txt`; add or update its entries there.
- If you change the knowledge-base content, regenerate or remove the old `vault_embeddings.pkl` so the embeddings match the updated text.
- The current backend allows all CORS origins. Restrict this before deploying the project publicly.
- The client manifest uses Manifest V2; browser support for this version may vary.

## License

No license file was included in the project archive. Add a `LICENSE` file if you intend to specify reuse or distribution terms.
