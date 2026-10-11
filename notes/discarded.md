# Tried and dropped

Format: what, verdict and why, date (2026), **reopen only if**. Everything is from one 16 GB Blackwell machine. Check the date.

## Language and code models

| What | Verdict and why | Date | Reopen only if |
|---|---|---|---|
| 27B at Q4 (17 GB) in Ollama | Does not fit 16 GB; CPU offload, slow | 09-16 | Ollama supports a quant of 14.5 GB or less |
| `llama.cpp` direct with a 27B IQ4_XS (+MTP) | Worked (7/7 in substance). 9-14 tok/s out of the box; **37-39 tok/s tuned** (Flash Attention, Q4_0 KV also for the draft, 1 slot, MTP-1, no offload, `--fit off`, **context 32768**; at 65536 it fell to 7 tok/s). Dropped as the active engine: a third engine to maintain and unreliable exit codes in our agent harness | 09-14/17 | You need ~40 % more speed than TabbyAPI |
| `llama.cpp` with `-fa on -ctk q8_0 -ctv q8_0` | 5/7, two real failures; saves only ~150 MB | 09-16 | Much longer context where the KV cache weighs more |
| `-nkvo` (KV cache in RAM) | 60 tok/s at 73K without it vs 35 tok/s at 150K with it: only if you need huge context | 09-17 | Context above 73K is essential |
| Swift-Qwen3.8-27B (a fine-tune) | Did not fix the timeout / over-verification on our Snake task | 09-17 | A version trained on its own data appears |
| 1-bit 27B (Bonsai) | 2/4 as an agent, fidelity errors | 09-16 | - |
| `devstral:24b` as a terminal agent | Stalls (0/7), inconsistent tool calls. Fine for Aider/chat | 09-14 | A new Devstral |
| `qwen2.5-coder:14b` as an agent | Narrates the tool call as text (0/7). FIM only | 09-14 | - |
| `gpt-oss:20b` as a terminal agent | Gets tangled editing files / stalls | 09-14 | - |
| `qwen3:14b` as a terminal agent | 4/7, 40 min to 2 h per task from thinking. Fine as a general model | 09-14 | - |
| `gemma4:26b` in Ollama | Does not fit; once failed to load (`Gemma4Assistant requires ctx_other`) | 09-14 | - |
| `--n-cpu-moe` for gemma4:26b | Ollama closed two PRs for marginal gain | 09-16 | - |
| Qwen3.8 at maximum reasoning (`xhigh`) | Over-thinks: 7-8x tokens, no better result | 09-16 | - |
| TabbyAPI `ngram` draft with the 27B at 32K | `Insufficient VRAM in split` | 09-16 | Context lowered a lot |
| Hot-loading vision in TabbyAPI | Neither the API nor `tabby_config.yml` enables it; needs top-level `config.yml` at startup | 09-16 | - |
| Updating TabbyAPI + exllamav3 to 1.5.1 / 1.6.0 | **Reverted.** `gate and up must share a bit width` on the mixed-bit Qwen3-Coder-30B quant (see pitfalls #6). Production stayed on TabbyAPI `fcc1a107` (2026-08-26) + exllamav3 1.4.4. Auto-detection of `tool_format` did work | 09-23, 10-09 | Never for that quant: the maintainer removed mixed gate/up support on purpose (exllamav3#502). Use uniform-bit quants, or stay on 1.4.4 |
| Second GPU / tensor-parallel (NInfer TP2) | Needs a second identical GPU | 09-16 | You buy a second 5060 Ti |
| MTP for gemma4 in Ollama | No official recipe for Windows/CUDA | 09-11 | Ollama supports it |
| Whisper large-v3-turbo on GPU | Not worth the VRAM | 09-11 | VRAM to spare |
| `bge-m3` as the RAG embedder | Replaced by `qwen3-embedding:0.6b` (Open WebUI does not use its sparse/ColBERT parts) | 09-11 | - |
| `qwen2.5vl:7b` | Replaced by `gemma4:12b-it-qat` | 09 | - |
| A third-party "install all models" Ollama script | Do not run: clashes with Ollama Desktop, never-applied configs, `num_gpu -1` gives OOM | 09 | - |
| `LiquidAI/d1-3B` as a local classifier/verifier | **Not installed.** On 154 synthetic messages (triage, moderation, claim-vs-source checking, prompt injection) 83 % right vs 90-93 % for `qwen3:14b` / `gemma4:12b`; ~60 ms vs 0.4-0.7 s; it followed an injected instruction; a d1-first cascade did not beat `qwen3:14b` alone. Windows torch has no flash attention (explicit-attention path), 6 GB VRAM (10.6 with long sources), LFM license. Open test: [d1-decision-bench](https://github.com/cCoolBoox/d1-decision-bench) | 10-09 | Thousands of messages a day where 0.5 s each does not fit, or a version that resists injected instructions |
| `qwen3:14b` or `gpt-oss:20b` at low reasoning as a course-text reviewer, instead of `gpt-oss:20b` medium | With known errors injected into two courses (with and without reference material, 52 errors, with repeats): medium at a 16,000-token cap found as many as `qwen3:14b` with ~4x fewer extra flags with material (5 vs 21) and won without material (22/24 vs 19/24); low produced 51 extra flags. At an 8,000 cap the reviewer reviewed nothing with material | 10-09 | A local model that matches `gpt-oss` in less time (11-15 min per course with material today) |

## Image

| What | Verdict and why | Date | Reopen only if |
|---|---|---|---|
| Ideogram 4.0 | Non-commercial license, restricted model | 09 | The license changes |
| Qwen-Image-2.1 | **Not adopted: blocked by license, not quality.** "Qwen Research" license is non-commercial only (a company's marketing counts as commercial). A blind A/B of 15 pairs against Krea 2 Turbo had no clear winner (Qwen-2.1 8 / Krea 7). Qwen-2.1 tended to respect the empty third asked for a headline, rendered skin and hair with more detail and wrote text with accents perfectly; it also has native RGBA transparency and multi-reference editing, which Krea lacks. Krea matched or won on compositional details. On a harder round (portrait + sci-fi dome) 2-2. One rater, small samples: a hint, not a conclusion. Weights deleted from our disk on 2026-10-09 | 09-22/23 | A commercial license is granted |
| Course illustrations, blind, 6 scenes | Krea 2 won 5/6, Z-Image 1, Qwen-Image-2.1 0. One rater | 09-24 | - |
| Two-stage latent upscale (Z-Image) | Made images worse | - | - |
| `res_multistep` sampler in our pipeline | Replaced by `dpmpp_2m` + `beta` (same quality, 2x faster) | - | - |
| Lowercase headline with Z-Image | 11/12 failures; capitals 6/6 | - | - |
| Exact text from Qwen-Image / Qwen-Image-Edit | Garbage; use an overlay | - | - |
| Hex colours, brands and abstract words in image prompts | Drawn as unreadable text | - | - |
| Removing "No text, no letters, no logos, no watermark" for Krea 2 Turbo | 6 blind pairs: no visible difference | - | The engine changes, or stray logos/text appear |
| Chatterbox (voice cloning) | No CUDA build for Blackwell; Kokoro stays | 09-03 | An image with cu128/sm_120 support |
| Google image API on the free tier | `429 limit 0`: needs billing; an app subscription does not count | - | Billing is enabled |
