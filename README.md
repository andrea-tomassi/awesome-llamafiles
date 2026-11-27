# awesome-llamafiles

Curated, ready-made `.llamafile` executables with the best presets for notable
scenarios. See [MODELS.md](./MODELS.md) for the catalog, provenance, hashes,
presets, the GPU status, and the license of the weights.

> **📦 The weights are not in this repository.**
> A `.llamafile` is over 1 GB, past GitHub's per-file limit for git, so each one
> ships as a **release asset**:
>
> **[⬇ Release `models-v4`](https://github.com/andrea-tomassi/awesome-llamafiles/releases/tag/models-v4)**:
> `LFM3-1.7B-pi-Q6_K.llamafile` (1.47 GB), the recommended build. LFM3-1.7B with
> the [harness-hub pi LoRA](https://huggingface.co/harness-hub/LFM3-1.7B-pi)
> merged into the weights, tuned for tool-calling agent work in `pi`. CPU-only,
> single file, runs anywhere.
>
> ```sh
> gh release download models-v4 --repo andrea-tomassi/awesome-llamafiles --pattern '*.llamafile'
> # or, without gh:
> curl -fLO https://github.com/andrea-tomassi/awesome-llamafiles/releases/download/models-v4/LFM3-1.7B-pi-Q6_K.llamafile
> ```
>
> Smallest-footprint alternative, base model without the adapter:
> [`models-v1`](https://github.com/andrea-tomassi/awesome-llamafiles/releases/tag/models-v1) →
> `LFM3-1.7B-QAD-Q4_0.llamafile` (1.05 GB).
>
> Verify either one, then drop it where your client looks for models
> (`~/.pi/agent/models/` for pi-embedded-ai, or point `PI_LLAMAFILE_DIR` here).

## GPU inference

The `.llamafile` format cannot ship a working GPU backend: llamafile patches its
llama.cpp Vulkan backend, and those patches do not apply outside the llamafile
build tree, so a separately compiled DSO crashes on tensor creation (verified on
an RTX 4060 Ti). For GPU inference, serve the bare GGUF with a CUDA build of
llama.cpp instead — measured **9206 tok/s prefill, 172 tok/s generation** on an
RTX 4060 Ti. The full recipe and the variant catalog are in
[MODELS.md](./MODELS.md).

Host- and client-agnostic: anything that can talk to a llamafile server works,
including [pi-embedded-ai](https://github.com/andrea-tomassi/pi-embedded-ai).
