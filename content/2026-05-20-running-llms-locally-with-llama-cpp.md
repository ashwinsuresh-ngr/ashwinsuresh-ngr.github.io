Title: Running LLMs Locally with llama.cpp
Date: 2026-05-20
Category: GenAI
Tags: GenAI, llama.cpp, local-LLM, quantization, tutorial
Slug: running-llms-locally-with-llama-cpp

Understanding what llama.cpp is, as covered in the previous post, is one thing — actually getting a model running on your own machine is the practical next step. The process has gotten dramatically simpler over the past couple of years, to the point where running a genuinely capable open-weight model locally is now realistically achievable on a laptop in well under an hour. Here's how it actually works, end to end.

## Step 1: Build or Install llama.cpp

llama.cpp can be built directly from source using CMake, or installed through a package manager on some platforms. Building from source gives the most control over which hardware backend (CUDA, Metal, ROCm, Vulkan) gets compiled in:

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
cmake -B build -DGGML_CUDA=ON   # enable CUDA if you have an NVIDIA GPU
cmake --build build --config Release
```

For Apple Silicon Macs, Metal acceleration is typically enabled by default; for CPU-only setups, no special flags are needed beyond a standard build.

## Step 2: Choose and Download a Quantized Model

As covered in the previous post, models are distributed in GGUF format, and most popular open-weight models — Llama, Qwen, Gemma, Mistral, and others — are available pre-quantized on Hugging Face. Picking a quantization level (Q4_K_M is a common, reasonable default) depends on the balance between available memory and desired quality covered in the previous post:

```bash
huggingface-cli download bartowski/Llama-3.1-8B-Instruct-GGUF \
  --include "Llama-3.1-8B-Instruct-Q4_K_M.gguf" --local-dir ./models
```

## Step 3: Run Interactive Inference

The llama-cli tool provides a straightforward way to interact with a loaded model directly from the terminal:

```bash
./build/bin/llama-cli -m ./models/Llama-3.1-8B-Instruct-Q4_K_M.gguf \
  -p "Explain vector search in simple terms" -n 256
```

This loads the model and generates a response to the given prompt, with `-n` controlling the maximum number of tokens generated — directly connecting to the max-token concepts covered in the controlling LLM responses post earlier in this series.

## Step 4: Tune Performance with GPU Offloading

As covered in the previous post, the `-ngl` flag controls how many of a model's layers get offloaded to GPU, letting even a modest GPU meaningfully speed up inference on a model too large to fit entirely in VRAM:

```bash
./build/bin/llama-cli -m ./models/model.gguf -ngl 32 -p "..."
```

Finding the right value for `-ngl` is typically an empirical process — starting near the maximum your GPU's VRAM can support and adjusting downward if you hit out-of-memory errors, balancing speed against what actually fits.

## Step 5: Serve an OpenAI-Compatible API

For building an actual application on top of a local model — rather than just interacting via terminal — llama-server exposes an HTTP API that's compatible with the OpenAI API format, letting existing tooling and SDKs built around that format work with a local model with minimal changes:

```bash
./build/bin/llama-server -m ./models/model.gguf -ngl 32 --port 8080
```

```python
import requests

response = requests.post("http://localhost:8080/v1/chat/completions", json={
    "model": "local-model",
    "messages": [{"role": "user", "content": "Summarize this text: ..."}]
})
```

This connects directly to the Python fundamentals and FastAPI posts covered elsewhere in this series — a locally served model can slot into the same kind of application architecture as a hosted API, just pointed at localhost instead of a commercial provider's endpoint.

## Why Run Locally in the First Place

- **Privacy.** Prompts and documents never leave your own hardware — genuinely important for sensitive data in domains like legal, medical, or internal business information where sending content to a third-party API isn't acceptable.
- **Cost.** Once a model is downloaded, inference is free beyond your own electricity and hardware — no per-token API costs, which matters a lot for high-volume or experimental use.
- **Availability and control.** No dependency on an external API's uptime, rate limits, or pricing changes — the model runs exactly as configured, indefinitely, on hardware you control.
- **Offline capability.** A locally running model works without an internet connection at all, relevant for edge deployments or genuinely air-gapped environments.

## The Realistic Trade-Offs

As covered in the parameters post earlier in this series, smaller, locally runnable models generally can't match the raw reasoning capability of the largest frontier models available only through hosted APIs — a real trade-off worth being honest about. A practical, increasingly common architecture reflects this directly: route the majority of routine queries to a local llama.cpp-served model, and reserve calls to a larger hosted model for the harder subset of tasks that genuinely need it — capturing much of the cost and privacy benefit of local inference without giving up capability where it matters most.

## The Bottom Line

Running LLMs locally with llama.cpp has become a genuinely practical, well-documented process: build or install the engine, download a quantized GGUF model sized to your hardware, and either interact with it directly or serve it through an OpenAI-compatible API for use in a real application. The privacy, cost, and control benefits are real and growing more compelling as open-weight models continue closing the capability gap with hosted frontier systems — making local inference an increasingly serious option, not just a hobbyist curiosity, for a meaningful share of real-world AI workloads.
