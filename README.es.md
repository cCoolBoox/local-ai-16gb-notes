# local-ai-16gb-notes

Qué funciona, qué no y qué conviene evitar al montar una **IA totalmente local con una sola GPU de consumo de 16 GB** (chat, agente de código, visión, RAG, imagen y voz).
Son nuestras notas de trabajo, limpias de cualquier dato privado: cada veredicto lleva fecha y, si es un «no», la condición que nos haría volver a probarlo.

Equipo: Ryzen 7 5800X · 32 GB de RAM · **RTX 5060 Ti 16 GB (Blackwell, sm_120)** · Windows 11 · Docker Desktop. Motores: Ollama (nativo), TabbyAPI (EXL3) y ComfyUI (Docker).
Pruebas entre el 03-09-2026 y el 09-10-2026. Última revisión: 10-10-2026.

> English: [README.md](README.md). Las notas detalladas (`notes/`) están en inglés.

## Cómo leerlo

- **Una máquina, las pruebas de una persona.** No son benchmarks de los modelos: es lo que pasó en este equipo con nuestras tareas.
- Un «no» significa «no para 16 GB, en Windows, para lo que necesitábamos, en esa fecha». Los modelos cambian cada mes: lo que tenga más de ~3 meses, vuélvelo a comprobar.
- No se incluyen pesos de modelos; mantienen sus licencias (algunas son no comerciales: se indica donde importa).

## Resumen en un párrafo

Con 16 GB de VRAM **solo cabe un motor pesado a la vez** (Ollama *o* TabbyAPI *o* ComfyUI). Diseña el conjunto alrededor de eso, libera la VRAM antes de cambiar y deja la voz y el análisis de documentos en CPU.
Elecciones sólidas hoy: `qwen3:14b` para chat general, `gemma4:12b-it-qat` si necesitas imagen/audio/vídeo de entrada, `gpt-oss:20b` para razonar, un MoE 30B-A3B de código en EXL3 (35,8 tok/s con decodificación especulativa n-gram),
un 27B de 3 bits en GGUF por Ollama como agente de terminal, Krea 2 Turbo y Z-Image-Turbo para imágenes, y `qwen3-embedding:0.6b` + `bge-reranker-v2-m3` para RAG.

## Contenido

- [notes/what-runs.md](notes/what-runs.md): lo que funciona hoy, con tamaños, velocidad y uso.
- [notes/pitfalls.md](notes/pitfalls.md): las trampas que nos costaron tiempo (Blackwell, ComfyUI, TabbyAPI/exllamav3, Ollama, topes de salida, texto en imágenes) y su arreglo.
- [notes/discarded.md](notes/discarded.md): lo que probamos y descartamos, por qué y cuándo reabrirlo.

Si lo ejecutas en otra GPU y algún veredicto cambia, abre un issue con la GPU, el controlador, el sistema y las versiones. Las correcciones son bienvenidas.

Licencia: CC0 1.0. Sin garantía; mira la fecha de cada nota.
