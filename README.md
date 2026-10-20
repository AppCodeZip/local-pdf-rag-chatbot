# Local PDF RAG Chatbot (free local LLM)

This project converts the original IBM watsonx.ai RAG demo into a local app using **Ollama + Llama 3.2 + Chroma + Gradio**. It does not call a paid hosted LLM API. The models run on your computer, so performance depends on your hardware and available RAM.

## 1. Install prerequisites

1. Install Python 3.11.
2. Install Visual Studio Code and its Python extension.
3. Install Ollama from https://ollama.com/download and start the Ollama app.

Verify in a terminal:

```bash
python3.11 --version
ollama --version
```

## 2. Download free local models

Run these commands once. The model downloads can be several GB:

```bash
ollama pull llama3.2:3b
ollama pull nomic-embed-text
ollama list
```

- `llama3.2:3b`: local chat/generation model.
- `nomic-embed-text`: local embedding model used to search PDF chunks.
- Ollama serves the models locally; no IBM project ID or API key is needed.

## 3. Create and activate a virtual environment

Open this project folder in VS Code, then open **Terminal → New Terminal**.

### macOS / Linux

```bash
python3.11 -m venv my_env
source my_env/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

### Windows PowerShell

```powershell
py -3.11 -m venv my_env
.\my_env\Scripts\Activate.ps1
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Always activate `my_env` before running the app so packages install into the project's isolated environment.

## 4. Run the app

Make sure Ollama is running, then in the activated environment:

```bash
python app.py
```

Open the local URL shown in the terminal, usually http://127.0.0.1:7860. Upload a text-based PDF and ask a question.

## 5. Debugging in VS Code

1. Open `app.py`.
2. Select the interpreter from `my_env` (`Cmd+Shift+P` on macOS or `Ctrl+Shift+P` on Windows → **Python: Select Interpreter**).
3. Choose **Run → Start Debugging** or press `F5`. If prompted, choose **Python Debugger**.
4. Add breakpoints inside `answer_question`, `load_and_split_pdf`, or `build_vector_store`.
5. Inspect `chunks`, `relevant_docs`, `context`, and `response.content`.

## 6. What happens in RAG?

1. **Load:** `PyPDFLoader` extracts text and page metadata.
2. **Split:** the splitter creates manageable, overlapping chunks.
3. **Embed:** `OllamaEmbeddings` converts chunks into numeric vectors.
4. **Store:** Chroma keeps vectors and their text/metadata.
5. **Retrieve:** similarity search selects the most relevant chunks for the question.
6. **Generate:** the LLM answers using those chunks as context.

The model does not automatically learn or permanently train on the PDF. The PDF is retrieved at question time.

## 7. Important settings in `app.py`

- `LLM_MODEL`: generation model downloaded with Ollama.
- `EMBEDDING_MODEL`: embedding model downloaded with Ollama.
- `LLM_TEMPERATURE`: lower values usually mean more consistent answers.
- `LLM_NUM_PREDICT`: maximum generated tokens; affects answer length and speed.
- `CHUNK_SIZE`: approximate maximum chunk length in characters.
- `CHUNK_OVERLAP`: repeated characters between adjacent chunks to avoid cutting context too abruptly.
- `RETRIEVER_K`: how many relevant chunks are supplied to the LLM.

These values are learning-friendly starting points, not universal optimal settings.

## Troubleshooting

### `ollama` command not found
Install Ollama and restart the terminal. Verify `ollama --version`.

### Connection refused / cannot connect to Ollama
Open/start the Ollama app, then try:

```bash
ollama list
```

### Model not found
Run:

```bash
ollama pull llama3.2:3b
ollama pull nomic-embed-text
```

### `ModuleNotFoundError`
Activate `my_env`, then run `python -m pip install -r requirements.txt`.

### PDF gives no text
Some PDFs are scans containing images rather than selectable text. This starter version does not include OCR; test with a text-based PDF first.

### Slow answers or memory errors
Try a smaller Ollama chat model, such as `qwen2.5:1.5b`, update `LLM_MODEL` in `app.py`, and pull it with `ollama pull qwen2.5:1.5b`. Model quality may differ.

### Security note
The app binds to `127.0.0.1` and disables Gradio public sharing, so it is intended for local learning/debugging. Do not enable public sharing for private PDFs.
