# What runs today (16 GB, Windows 11)

Sizes are on disk. "Fits" means fully on the GPU with the context we use. Reviewed 2026-10-10.

## Text, reasoning and vision: Ollama (native Windows)

| Model | Size | Use |
|---|---|---|
| `qwen3:14b` | 9.3 GB | General default: chat, structured extraction, automation agents |
| `gemma4:12b-it-qat` | 7.2 GB | Multimodal (image + audio + video input), tools; also our prompt writer/reviewer |
| `gpt-oss:20b` | 13 GB | Reasoning + tools. Fits just barely. Reasoning level matters (see pitfalls) |
| `devstral:24b` | 14 GB | Chat/edit and Aider. **Not** as a terminal agent (0/7 on our task battery) |
| `qwen2.5-coder:14b` | 9.0 GB | The only one of ours with fill-in-the-middle (editor autocomplete). Not an agent |
| `qwen3-embedding:0.6b` | 0.6 GB | Embeddings for RAG (32K context, 1024 dims) |

Anti-OOM settings that we apply to Ollama: `OLLAMA_KV_CACHE_TYPE=q4_0`, `OLLAMA_MAX_LOADED_MODELS=1`, `OLLAMA_NUM_PARALLEL=1`.

## Code: TabbyAPI (EXL3) and a terminal agent through Ollama

| Model | Size | Engine | Notes |
|---|---|---|---|
| Qwen3-Coder-30B-A3B, EXL3 3.5 bpw | 13.1 GB | TabbyAPI | MoE: best quality per GB for us. `draft_mode: ngram` -> 35.8 tok/s. Needs an explicit `tool_format` (see pitfalls) |
| Qwen3.8-27B, GSQ+RCO IQ3_S GGUF (ISTA-DASLab) | 11.8 GB | Ollama | **Terminal agent default.** 7/7 on our 7-task battery in two independent passes (6/7 before, with the EXL3 quant), `num_ctx` 65536, survived a real ~49,000-token task |
| Qwen3.8-27B, EXL3 3.0 bpw | 13.8 GB | TabbyAPI | Our only working vision path (see pitfalls); 27 tok/s |

Reasoning effort for the Qwen3.8 family: `low`/`medium`, or off for file and tool tasks. The maximum level spent 7-8x the thinking tokens with no better result.

## Image: ComfyUI (Docker, GPU)

| Model | Size | Time | Notes |
|---|---|---|---|
| **Krea 2 Turbo** (fp8 + qwen3-vl 4B encoder + Qwen-Image VAE) | 13.1 + 5.2 GB | 34-61 s | Default. Best composition and text rendering; 8 steps, CFG 1, euler/simple. Community license: free commercial use under US$1M revenue and 50 seats, with attribution |
| **Z-Image-Turbo** (fp8) | 5.7 + 7.5 GB | 10-20 s | Photography. Apache 2.0. 8 steps, CFG 1; `dpmpp_2m` + `beta` was as good as `res_multistep` and 2x faster |
| FLUX.2 Klein 4B (Q8_0) | 4.3 GB | 15-20 s | Fast alternative; follows the prompt very literally |

No image model wrote exact text reliably: put prices, names and slogans on afterwards with Pillow.

## Voice (CPU, Docker: it does not compete for VRAM)

| Component | Notes |
|---|---|
| `faster-whisper-small` (Speaches) | ~5 s per sentence on CPU; fine for Spanish |
| Kokoro TTS (`ef_dora`) | Fast on CPU, Apache 2.0, does not clone voices |

## RAG and documents (Open WebUI + Docker)

| Component | Notes |
|---|---|
| Open WebUI | UI, RAG, tools |
| Docling | PDF/table/OCR parsing for RAG, CPU |
| `qwen3-embedding:0.6b` + `BAAI/bge-reranker-v2-m3` | Embeddings + reranker. If you change the embedder you must re-upload your documents |
| SearXNG + Valkey | Private metasearch for web-enabled chat |
| mcpo | Exposes MCP tools to Open WebUI as OpenAPI |

On a synthetic 52-fragment manual with paraphrased questions, retrieving the right passage worked for: keywords only 1/24, embeddings 16/24, hybrid (keywords + embeddings) **20/24** at k=10 and 4,500 characters. LLM re-ranking did not help and was 8x slower, so we left it off.

## Local checking of generated text

`gpt-oss:20b` with medium reasoning and a 16,000-token output cap was the best local reviewer of AI-written course material in our test (43 of 52 injected errors found, almost no extra flags; 11-15 min for 4 lessons). Data and scripts: [reviewer-injection-bench](https://github.com/cCoolBoox/reviewer-injection-bench).
