# awesome-llamafiles

A curated distro of ready-made `.llamafile` executables: the most useful open
models, wrapped with the best launch presets for notable scenarios. Host-
and client-agnostic — anything that speaks to a llamafile server (including
the [pi-embedded-ai](https://github.com/andrea-tomassi/pi-embedded-ai)
extension) can use these.

**Distro, not a factory**: these files ship as-is; building llamafiles
from GGUF (quantization, pinning, CI) is explicitly future work.

Install: copy the files into any directory your client discovers
(`~/.pi/agent/models/` is the convention for pi-embedded-ai), or point
`PI_LLAMAFILE_DIR` at this checkout.

## Models

| File | Release | Source | SHA256 | Best for |
|------|---------|--------|--------|----------|
| `LFM3-vulkan.llamafile` (1.06 GB) | `models-v2` | LFM3-1.7B QAD Q4_0 + embedded **Vulkan** backend, llamafile v0.10.7 | `6c95cc8bbfc663bfabc7d18f9bfb5f2339557a35d801d41ef6e0b51720b909ea` | **default pick**: uses a GPU when the host has one, CPU otherwise |
| `LFM3-1.7B-QAD-Q4_0.llamafile` (1.05 GB) | `models-v1` | LFM3-1.7B QAD Q4_0, CPU-only build | `6b01eabc3f1b7f17a68b46b780020f444cc5ee0951f3a7e036f2344cc6697be6` | smallest, maximum portability (any ABI) |

## GPU support

`LFM3-vulkan.llamafile` carries the Vulkan backend (`ggml-vulkan.so`) **inside the
single file**: llamafile extracts it on first run and loads it. The launcher must
leave `--gpu` at its default `auto`, which is what `pi-embedded-ai` does.

| Host | Behaviour |
|------|-----------|
| Vulkan device and driver present | model runs on the GPU (`-ngl`/`--gpu-layers` default offloads every layer) |
| No GPU, no driver, driver crashes on probe | **CPU**, no error: the backend is skipped |
| Non-x86-64 / non-glibc ABI (musl, macOS, ARM, old glibc) | the DSO does not load: **CPU**, same file still works |
| `--gpu vulkan` forced on a host without Vulkan | fatal error — never pass an explicit backend |

Caveat: this build used the distro `glslc` (shaderc 2023.8), so the `coopmat2`,
`integer_dot`, `bfloat16` and `fp8` shader paths are absent. Mainstream NVIDIA/AMD
paths work; rebuild with the LunarG Vulkan SDK (`VULKAN_SDK=...`) for full coverage.

## Download and verify

The weights are release assets, not files in this repository (GitHub rejects
blobs over 100 MB). Download, then confirm the checksum before using them:

```sh
# the GPU-capable default
gh release download models-v2 --repo andrea-tomassi/awesome-llamafiles --pattern '*.llamafile'
sha256sum LFM3-vulkan.llamafile
# 6c95cc8bbfc663bfabc7d18f9bfb5f2339557a35d801d41ef6e0b51720b909ea

# the CPU-only, maximum-portability variant
gh release download models-v1 --repo andrea-tomassi/awesome-llamafiles --pattern '*.llamafile'
sha256sum LFM3-1.7B-QAD-Q4_0.llamafile
# 6b01eabc3f1b7f17a68b46b780020f444cc5ee0951f3a7e036f2344cc6697be6
```

Then place it where your client discovers models: `~/.pi/agent/models/` for
[pi-embedded-ai](https://github.com/andrea-tomassi/pi-embedded-ai), or point
`PI_LLAMAFILE_DIR` at the directory holding it.

A mismatch means a truncated or tampered download: delete it and fetch again.

## Suggested presets

Copy-paste launch settings per scenario (also readable as `PI_LLAMAFILE_*`
env for pi-embedded-ai):

| Scenario | Model | `--ctx-size` | Notes |
|----------|-------|--------------|-------|
| coding agent (default) | LFM3-1.7B-QAD-Q4_0 | 131072 | reasoning on; ~2.7 GB RAM; the OOTB preset |
| long-context review | LFM3-1.7B-QAD-Q4_0 | 262144 | full trained context; ~4.5 GB RAM |
| tiny host | LFM3-1.7B-QAD-Q4_0 | 32768 | minimal KV cache; faster load, shorter memory |

## License of the weights

The model weights (`*.llamafile`) are **not** covered by this repository's
MIT license ([LICENSE](./LICENSE) covers docs and repo files only). They
are released by LiquidAI under the **LFM Open License v1.0**, shipped in full
in this repository as [`LICENSE.LFM3`](LICENSE.LFM3).

Compliance notes for redistribution:

- **Attribution**: retain all copyright, patent, trademark, and attribution
  notices (Section 4(c)); the model table above credits LiquidAI as the
  source of the weights.
- **Pass the license on**: any recipient of the `.llamafile` (or a derivative
  work) must get a copy of LICENSE.LFM3 (Section 4(a)).
- **Mark modifications**: modified files must carry prominent notices stating
  that you changed them (Section 4(b)).
- **Commercial threshold**: Commercial Use is licensed only while you or your
  Legal Entity stays below the Threshold — annual revenue of $10,000,000 or
  more is *not* licensed under this Agreement (Section 5). Qualified
  Non-Profit Organizations using the Work for non-commercial or research
  purposes are exempt from the Threshold.
- **Automatic termination**: the license terminates automatically and
  immediately on non-compliance; all copies in your possession must then be
  deleted (Section 11).

Read the model card at
https://huggingface.co/LiquidAI/LFM3-1.7B-GGUF before redistributing.
