# Pitfalls (and the fix)

Each item: what happens, the fix, and how sure we are. Reviewed 2026-10-10.

## VRAM and engines

1. **Only one heavy engine fits.** Ollama, TabbyAPI and ComfyUI each want most of the 16 GB. Before switching: `ollama stop <model>` for every loaded model, `POST /v1/model/unload` on TabbyAPI, and stop the GPU containers you do not need. We wrapped it in a script (stop -> work -> restore). Voice and Docling run on CPU, so they never conflict. *Certain.*
2. **A 27B at Q4 (17 GB) does not fit.** Ollama offloads to CPU and it becomes slow. The 3-bit GSQ+RCO GGUF (11.8 GB) does fit and kept quality in our agent battery. *Certain on this machine.*
3. **Ollama OOM with 27B-class models**: set `OLLAMA_KV_CACHE_TYPE=q4_0`, `OLLAMA_MAX_LOADED_MODELS=1`, `OLLAMA_NUM_PARALLEL=1`. *Certain.*
4. **Set `num_ctx` yourself.** We ran the terminal agent at 65536 with the 11.8 GB quant (tested with a real ~49,000-token task); a bigger context will not fit a bigger model. Match the context to the VRAM margin you actually have before copying someone else's recipe. *Certain.*

## TabbyAPI / exllamav3 (EXL3)

5. **Set `tool_format` explicitly** (`qwen3_coder` for the Coder, `qwen3_5` for Qwen3.8) and `reasoning: true`, in a per-model `tabby_config.yml`. Without it the model's tool call comes back as plain text and the agent never sees a call. The maintainer confirmed this was a bug ([issue #479](https://github.com/theroyallab/tabbyAPI/issues/479), which we opened): newer versions default to `auto` and warn. We confirmed auto-detection works on a newer version. *Certain.*
6. **`exllamav3` >= 1.5.1 refuses some MoE quants**: `RuntimeError: BC_BlockSparseMLP: gate and up must share a bit width`. It hit the community `Qwen3-Coder-30B-A3B` 3.5 bpw quant whose `gate` and `up` projections have different bit widths. The author of exllamav3 said that quant was made in a short window before the optimizer required `K_gate = K_up`, that it should still be supported and he would look at it, but that `K_gate != K_up` is undesirable anyway because it costs performance. Re-tested on 1.6.0 and on 1.6.1 (2026-10-11, loading the model directly with exllamav3, separate from the production install): same error. In that quant `gate_proj` is 3-bit in all 6144 experts while `up_proj` is 3-bit in 3456 and 4-bit in 2688 (read from the safetensors headers). The maintainer asked us to track it in the exllamav3 repo. *Certain; still failing on 1.6.1.*
   - **Check a quant before you download 13 GB**: read only the `.safetensors` headers (the first bytes of each file, e.g. with HTTP range requests) and compare the shapes of the `gate` and `up` trellis tensors for the experts. Equal = uniform.
   - A **uniform** quant (branch `3.34bpw-h5-opt` of `amanwalksdownthestreet/Qwen3-Coder-30B-A3B-Instruct-exl3`, 12.4 GiB) loads and works on 1.4.4: ~84 tok/s in a short test (same as the 3.5 bpw one), well-formed tool calls, 5/6 code tasks with hidden tests (the 3.5 bpw one also 5/6; tiny sample). On 1.6.0 it loads then runs out of VRAM ("Insufficient VRAM in split"), undiagnosed.
7. **Do not `uv pip install` over exllamav3 casually**: it can pull a CPU-only `torch==2.11.0` from PyPI. Reinstall the CUDA (`cu130`) torch wheel by hand afterwards, every time. *Certain.*
8. **`draft_mode: ngram` only with the Coder-30B.** With the 27B at 32K context it fails with "Insufficient VRAM in split". *Certain.*
9. **Vision in TabbyAPI** is only applied from the top-level `config.yml` **at startup**, needs `max_seq_len: 16384` and `draft_mode: disabled` (14.7 of 16.3 GB used). Loading it "hot" through the API does not enable it. *Certain.*
10. **Reasoning effort**: for Qwen3.8, `low`/`medium`/off. The maximum level spends 7-8x the thinking tokens without a better answer. Turning reasoning off shortens the answer; it does not raise tokens per second. *Certain.*

## ComfyUI on Blackwell

11. **ComfyUI 0.34.x: add `--disable-pinned-memory`** to the launch args. Without it you hit `HostBuffer.read_file_slice failed` from the dynamic-VRAM streaming ([ComfyUI #15255](https://github.com/Comfy-Org/ComfyUI/issues/15255)). *Certain on 0.34.2.*
12. Use a CUDA 13-era image for sm_120 (we use `yanwk/comfyui-boot:cu132`). Chatterbox (voice cloning) had no Blackwell build and its Blackwell Dockerfile did not compile for us, so we stayed with Kokoro. *As of 2026-09-03.*

## Image prompts

13. **No model wrote exact text reliably.** Qwen-Image produced garbage ("MIE ABILEPFRARE"). Put exact text on afterwards with Pillow. Keep any headline inside the image to 4 words or fewer.
14. **Z-Image-Turbo: write headlines in CAPITALS.** In lowercase it dropped letters in 11 of 12 tries; in capitals 6 of 6 were right. Krea 2 handles lowercase.
15. **Hex colours, brand names and abstract words in a prompt get drawn as unreadable text.** Strip them; describe only what the camera sees.
16. Appending "No text, no letters, no logos, no watermark" to a Krea 2 Turbo prompt: in 6 blind pairs (same seed) with and without it, we saw no difference. Keep or drop it.

## RAG

17. **Changing the embedder invalidates your index**: re-upload the documents.
18. `qwen3-embedding` works better when the query carries an instruction (`Instruct: ...\nQuery: ...`) and the documents do not. In Open WebUI this is an environment variable (`RAG_EMBEDDING_QUERY_PREFIX`) that only takes effect at startup.
19. When you feed reference material to a text generator, pass it whole, or keep each title attached to its paragraphs: if titles end up separated from their text, the model fills the gap with invented details. Check specifics against the material afterwards.

## Output caps and silent failures

20. **`gpt-oss` counts its reasoning tokens inside `num_predict`.** With a long input and `num_predict` 8000 the answer was cut (`done_reason: length`) in every lesson and the run found 0 problems with no error message. At 16000 it worked. If a reasoning model "finds nothing", check `done_reason` first.
21. **Low reasoning is not just faster, it is noisier**: `gpt-oss:20b` at `low` flagged every unit of a whole lesson as "not in the reference material" (41 extra flags in one test).
22. **A verifier made of the same model that wrote the text finds nothing**; use a different one. A local `qwen3:14b` with JSON-schema output did work as a fallback verifier in our tests (64/64 on synthetic pairs).

## Windows specifics

23. PyTorch wheels for Windows have no flash attention. Models that assume it (e.g. d1-3B) need their explicit-attention path; see [d1-decision-bench](https://github.com/cCoolBoox/d1-decision-bench) for the patch we used.
24. Ollama Desktop and third-party "install all models" scripts can fight over the same service/config; install models explicitly and apply the env vars in one place. One such script we reviewed set `num_gpu -1`, which ran out of memory, and carried configs that were never applied.
