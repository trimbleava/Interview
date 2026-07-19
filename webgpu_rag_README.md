[← Back to interview prep](interview_prep_full.md#any-experience-with-ai-integrated-applications)

# WebGPU RAG knowledge graph — prototype

![WebGPU RAG pipeline architecture](webgpu_rag_pipeline.svg)

## What it does
A retrieval-augmented generation (RAG) tool that runs entirely in the browser: drop in PDFs, code,
or text files, and it builds a searchable semantic index, a visual knowledge graph of terms, and
answers questions about the documents using a language model — all without a backend server or
API key, since both the retrieval and the generation happen on-device via WebGPU.

## Architecture, stage by stage
1. **File intake** — accepts PDFs (parsed client-side via `pdfjs-dist`), plain text/code files, and
   zip archives (`jszip`), with a text-cleanup pass to normalize extracted content.
2. **Chunking** — splits documents into overlapping chunks, snapping breaks to sentence or word
   boundaries where possible rather than cutting mid-word.
3. **Embedding index (Web Worker)** — a TF-IDF hashed tokenizer builds the semantic index inside a
   dedicated Web Worker, off the main thread, so indexing a large file doesn't freeze the UI.
4. **RAG retrieval engine** — embeds the query the same way, ranks all chunks by vector similarity,
   and returns the top-K most relevant.
5. **Two consumers of the same retrieved chunks**:
   - **Knowledge graph builder** — extracts term co-occurrence relationships across chunks and
     renders them as an interactive force-directed graph.
   - **Answer generation** — builds a grounded prompt from the retrieved chunks and runs it
     through an in-browser LLM (`@mlc-ai/web-llm`, WebGPU-accelerated, tested against several
     small instruction models for the best size/quality tradeoff), with an extractive,
     non-LLM synthesis path as a fallback.

## How this maps to the job description

| JD requirement | What this project demonstrates |
|---|---|
| AI-integrated web applications (preferred, nice-to-have) | The most direct match in this whole prep doc — a full RAG pipeline including retrieval, grounding, and generation, running against a real LLM |
| Async / concurrency thinking | Web Worker offloading for the embedding index keeps the UI thread responsive during heavy computation — same motivation as async I/O in a backend, applied client-side |
| Service-oriented / modular design | Each concern (file intake, chunking, embeddings, retrieval, graph building, generation) lives in its own module with a narrow interface — the same separation-of-concerns instinct that backend service boundaries require |
| Python proficiency | **Not demonstrated here** — this is a React/JavaScript prototype, not Python. Don't lead with this project for Python-specific questions. |

## Honest framing for the interview
This is a prototype, and it's JavaScript/React, not Python — be upfront about that if it comes up.
Its value in this interview is narrow but real: it's your best answer to the "AI-integrated web
applications" preferred skill, and it's a good example of architectural instinct (worker-thread
offloading, modular pipeline design) that happens to be language-agnostic. Don't stretch it to
cover the core Python/Django/FastAPI/task-queue requirements — use the other projects for those,
and bring this one out specifically when AI integration or system design comes up.
