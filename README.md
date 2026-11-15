# awesome-llamafiles

Curated, ready-made `.llamafile` executables with the best presets for notable
scenarios. See [MODELS.md](./MODELS.md) for the catalog, provenance, hashes,
presets, and the license of the weights.

> **📦 The weights are not in this repository.**
> A `.llamafile` is over 1 GB, past GitHub's per-file limit for git, so each one
> ships as a **release asset**:
>
> **[⬇ Release `models-v1`](https://github.com/andrea-tomassi/awesome-llamafiles/releases/tag/models-v1)**
> — `LFM3-1.7B-QAD-Q4_0.llamafile` (1.05 GB)
>
> ```sh
> gh release download models-v1 --repo andrea-tomassi/awesome-llamafiles --pattern '*.llamafile'
> # or, without gh:
> curl -fLO https://github.com/andrea-tomassi/awesome-llamafiles/releases/download/models-v1/LFM3-1.7B-QAD-Q4_0.llamafile
> ```
>
> Verify it, then drop it where your client looks for models
> (`~/.pi/agent/models/` for pi-embedded-ai, or point `PI_LLAMAFILE_DIR` here).

This repository holds the catalog, the presets, the license of the weights, and
its checksums — never the binaries.

Host- and client-agnostic: anything that can talk to a llamafile server works,
including [pi-embedded-ai](https://github.com/andrea-tomassi/pi-embedded-ai).
