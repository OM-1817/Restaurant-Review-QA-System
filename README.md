# Restaurant Review QA System

A retrieval-augmented generation (RAG) chatbot that answers questions about restaurants by intelligently retrieving and synthesizing relevant reviews using LangChain and Ollama.

## Features

- **Semantic Search**: Uses BGE-M3 embeddings to find contextually relevant reviews from a large corpus
- **Persistent Vector Database**: ChromaDB stores and retrieves embeddings efficiently
- **LLM-Powered Synthesis**: Leverages Llama 3.2 to generate human-readable answers grounded in actual review data
- **Hallucination Reduction**: RAG architecture grounds responses in real data, reducing model-generated inaccuracies
- **Interactive CLI**: User-friendly question-answer interface with streaming responses

## Tech Stack

| Component | Technology |
|-----------|-----------|
| **LLM Framework** | LangChain |
| **Language Model** | Ollama (Llama 3.2) |
| **Embeddings** | Ollama (BGE-M3) |
| **Vector DB** | ChromaDB |
| **Data Processing** | Pandas |
| **Language** | Python 3.8+ |

## Project Structure

```
.
├── vector.py                          # Vector store setup & retriever
├── main.py                            # Interactive QA loop
├── realistic_restaurant_reviews.csv   # Review dataset
├── chrome_langchain_db/               # Persistent ChromaDB storage
└── README.md
```

## Installation

### Prerequisites
- Python 3.8+
- Ollama installed and running locally ([download](https://ollama.ai))

### Setup

1. **Clone and navigate to project**
   ```bash
   cd restaurant-qa-system
   ```

2. **Create virtual environment**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. **Install dependencies**
   ```bash
   pip install langchain langchain-ollama langchain-chroma pandas
   ```

4. **Pull required Ollama models**
   ```bash
   ollama pull llama3.2
   ollama pull bge-m3
   ```

5. **Ensure Ollama is running**
   ```bash
   ollama serve
   ```
   (Run in a separate terminal)

## Usage

### Running the QA System

```bash
python main.py
```

Then ask questions interactively:
```
-------------------------------
Ask your question (q to quit): What do people think about the pizza here?

[Retrieves 5 relevant reviews and generates answer]

-------------------------------
Ask your question (q to quit): Are there any complaints about the service?

[LLM synthesizes answer from retrieved reviews]

-------------------------------
Ask your question (q to quit): q
```

## How It Works

### Architecture Flow

```
User Question
    ↓
Semantic Retrieval (BGE-M3 Embeddings)
    ↓
ChromaDB Vector Store (k=5 nearest reviews)
    ↓
LLM Prompt Chain (Llama 3.2)
    ↓
Context-Grounded Answer
```

### Hallucination Reduction Strategy

**Problem**: LLMs can generate plausible-sounding but incorrect information when answering from memory alone.

**Solution - Retrieval-Augmented Generation (RAG)**:
1. **Retrieve**: Fetch actual reviews from the vector database matching the question semantically
2. **Ground**: Pass retrieved reviews explicitly in the prompt context
3. **Generate**: LLM synthesizes an answer using only the provided review context
4. **Result**: Responses are anchored to real data, drastically reducing hallucinations

**Example**:
```python
# Without RAG: "Customers loved the ambiance" (could be made up)
# With RAG: "Based on reviews, customers praised the warm lighting and cozy atmosphere"
```

## File Descriptions

### `vector.py`
- Initializes Ollama embeddings with BGE-M3 model
- Creates/loads ChromaDB vector store
- Loads restaurant reviews from CSV and stores as embeddings
- Exports retriever with k=5 (retrieves top 5 reviews)

### `main.py`
- Interactive command-line interface
- Accepts user questions in a loop
- Retrieves relevant reviews using the retriever
- Invokes LLM chain with context to generate answers
- Exit with 'q'

## Configuration

### Tunable Parameters

**In `vector.py`:**
- Embedding model: `OllamaEmbeddings(model="bge-m3")`
- Vector DB location: `./chrome_langchain_db`
- Collection name: `"restaurant_reviews"`

**In `main.py`:**
- Language model: `OllamaLLM(model="llama3.2")`
- Retrieved documents: `search_kwargs={"k": 5}` (increase for more context)
- System prompt: Customize the `template` variable

## Performance Considerations

- **First run**: Embeddings generation takes ~2-5 minutes for 1000 reviews
- **Subsequent runs**: Instant (loads from persistent ChromaDB)
- **Memory**: ~2GB RAM required (Ollama models run locally)
- **Latency**: ~2-5 seconds per query (LLM inference on CPU)

## Future Enhancements

- Add filtering by rating/date in retrieval
- Implement multi-turn conversation with memory
- Add source citations showing which reviews informed the answer
- Deploy as web API with FastAPI
- Fine-tune embeddings on restaurant domain


## Author

Om Vaghani
