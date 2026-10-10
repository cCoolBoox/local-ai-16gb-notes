# local-ai-16gb-notes

What runs, what does not, and what to avoid when you build a **fully local AI stack on one 16 GB consumer GPU**
(chat, code agent, vision, RAG, image generation, voice). These are our working notes, cleaned of anything private:
every verdict has a date and, when it is a "no", the condition that would make us try again.

Machine: Ryzen 7 5800X · 32 GB RAM · **RTX 5060 Ti 16 GB (Blackwell, sm_120)** · Windows 11 · Docker Desktop. Engines: Ollama (native), TabbyAPI (EXL3), ComfyUI (Docker).
Tests ran between 2026-09-03 and 2026-10-09. Last review: 2026-10-10.

> Español: [README.es.md](README.es.md)

## How to read this

- **One machine, one person's tests.** These are not benchmarks of the models; they are what happened on this box with our prompts and tasks.
- Nothing here is a recommendation to buy anything. A "no" means "not for 16 GB, on Windows, for what we needed, on that date".
- Models move every month. Anything older than ~3 months should be re-checked before you rely on it.
- Model weights are not included and keep their own licenses (some are non-commercial or restricted: noted where it matters).

## The one-paragraph version

With 16 GB of VRAM **only one heavy engine fits at a time** (Ollama *or* TabbyAPI *or* ComfyUI). Plan the stack around that, free the VRAM before switching, and keep voice and document parsing on CPU.
Good all-round choices today: `qwen3:14b` for general chat, `gemma4:12b-it-qat` when you need image/audio/video input, `gpt-oss:20b` for reasoning, a 30B-A3B MoE coder in EXL3 for code (35.8 tok/s with n-gram speculative decoding),
a 27B 3-bit GGUF through Ollama for a terminal agent, Krea 2 Turbo and Z-Image-Turbo for images, `qwen3-embedding:0.6b` + `bge-reranker-v2-m3` for RAG.

## Contents

| File | What it has |
|---|---|
| [notes/what-runs.md](notes/what-runs.md) | The models and services that work today, with size, speed and what each is for |
| [notes/pitfalls.md](notes/pitfalls.md) | The traps that cost us time (Blackwell, ComfyUI, TabbyAPI/exllamav3, Ollama, output caps, text in images) and the fix |
| [notes/discarded.md](notes/discarded.md) | What we tried and dropped, why, and when to reopen it |

## Related open tests

- [d1-decision-bench](https://github.com/cCoolBoox/d1-decision-bench): is a tiny open "decision model" (LiquidAI d1-3B) worth installing next to Ollama? (answer for us: not at our volume).

## Contributing

If you run the same thing on another GPU (a 5070 Ti, a 4060 Ti 16 GB, a 3090...) and a verdict changes, open an issue with the GPU, driver, OS and versions. Corrections are welcome.

## License

CC0 1.0: use it however you like. No warranty; check the date of each note.
