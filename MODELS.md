# awesome-llamafiles

A curated distro of ready-made `.llamafile` executables: the most useful open
models, wrapped with the best launch presets for notable scenarios. Host-
and client-agnostic — anything that speaks to a llamafile server (including
the [pi-embedded-ai](https://github.com/andrea-tomassi/pi-embedded-ai)
extension) can use these.

**Distro, not a factory**: these files ship as-is; building llamafiles
from GGUF (quantization, pinning, CI) is not automated in this repository.
The one documented exception is the `models-v4` pi build, whose exact
provenance is recorded below so it can be reproduced by hand.

Install: copy the files into any directory your client discovers
(`~/.pi/agent/models/` is the convention for pi-embedded-ai), or point
`PI_LLAMAFILE_DIR` at this checkout.

## Models

| File | Release | Source | SHA256 | Best for |
|------|---------|--------|--------|----------|
| `LFM3-1.7B-pi-Q6_K.llamafile` (1.47 GB) | `models-v4` | LFM3-1.7B F16 + [harness-hub pi LoRA](https://huggingface.co/harness-hub/LFM3-1.7B-pi) merged, Q6_K, llamafile v0.10.7 | `961125f363ed9746b5e6bf4c30e7711f4dac49a324ee15024811fd9820a8ddf7` | **recommended**: agentic tool calling, tuned for `pi`; CPU-only |
| `LFM3-vulkan.llamafile` (1.06 GB) | `models-v2` | LFM3-1.7B QAD Q4_0 + embedded Vulkan DSO, llamafile v0.10.7 | `6c95cc8bbfc663bfabc7d18f9bfb5f2339557a35d801d41ef6e0b51720b909ea` | experimental, runs on CPU — see [VULKAN-ATTEMPTS.md](./VULKAN-ATTEMPTS.md) |
| `LFM3-1.7B-QAD-Q4_0.llamafile` (1.05 GB) | `models-v1` | LFM3-1.7B QAD Q4_0, CPU-only build | `6b01eabc3f1b7f17a68b46b780020f444cc5ee0951f3a7e036f2344cc6697be6` | smallest footprint: base model without the adapter, maximum portability (any ABI) |

## Variants at a glance

| Need | Asset |
|------|-------|
| CPU, single file, agentic tool calling (**recommended**) | `models-v4` → `LFM3-1.7B-pi-Q6_K.llamafile` |
| CPU, single file, smallest footprint, no adapter | `models-v1` → `LFM3-1.7B-QAD-Q4_0.llamafile` |
| GPU on NVIDIA, fastest | `models-v3-cuda` → `LFM3-QAD-Q4_0.gguf` + a CUDA `llama.cpp` |
| AMD / Intel / Apple GPU | the GGUF with a ROCm, Vulkan or Metal build of `llama.cpp` |
| experimental, unusable GPU path | `models-v2` → `LFM3-vulkan.llamafile` |

### NVIDIA support for `models-v3-cuda`

Verified on an RTX 4060 Ti 16 GB (Ada, CC 8.9, driver 580.178.04) with
`ghcr.io/ggml-org/llama.cpp:server-cuda` and `-ngl 99`: 9206 tok/s prefill,
172 tok/s generation, 1.65 GB VRAM at 32k context.

Supported: NVIDIA compute capability 5.0 and newer — Maxwell, Pascal, Volta,
Turing, Ampere, Ada, Hopper (whatever the CUDA image ships kernels for).
Not supported: AMD (needs ROCm or Vulkan), Intel (Vulkan/SYCL), Apple (Metal),
or any host without an NVIDIA driver.

```sh
docker run --rm --gpus all -p 8080:8080 -v $PWD:/models \
  ghcr.io/ggml-org/llama.cpp:server-cuda \
  -m /models/LFM3-QAD-Q4_0.gguf -ngl 99 -c 32768 --host 0.0.0.0 --port 8080
```

## GPU support — status

**The `models-v2` artifact does not enable the GPU yet.** It was built by compiling
llamafile's `vulkan.sh` against the llamafile repository's pinned llama.cpp, but that
backend does not match the ggml ABI of the v0.10.7 binary: on a machine with a
Vulkan GPU it loads, enumerates the device, assigns every layer to it, and then
aborts while creating tensors (`GGML_ASSERT(buft)`, or a segfault). llamafile patches
its Vulkan backend, and those patches do not apply to the llama.cpp commit the
binary embeds, so a compatible DSO cannot be produced from this tree. Building the
llamafile itself with `vulkan.sh` would match by construction; that is the open path.

**What works today on a GPU host**: serve the GGUF with a CUDA build of
llama.cpp. Measured on the RTX 4060 Ti (16 GB) of `u24-server`, LFM3-1.7B QAD Q4_0,
32k context:

| Metric | GPU (CUDA, `-ngl 99`) | CPU (this project's baseline) |
|--------|----------------------|-------------------------------|
| prefill | **9206 tok/s** (441 tok) | ~89 tok/s |
| generation | **172 tok/s** | ~9 tok/s |
| VRAM | 1650 MiB | — |

```sh
# on a GPU host, with the GGUF extracted from any .llamafile (it is a zip entry)
docker run --rm --gpus all -p 8080:8080 -v $PWD:/models \
  ghcr.io/ggml-org/llama.cpp:server-cuda \
  -m /models/LFM3-QAD-Q4_0.gguf -ngl 99 -c 32768 --host 0.0.0.0 --port 8080
```

The extension is client-agnostic: it talks to whatever serves `/v1` and `/props` on
the port, so a CUDA `llama-server` works exactly like the llamafile.

### Fallback contract (verified)

| Host | Behaviour with `models-v2` |
|------|---------------------------|
| Vulkan device and a matching DSO | not achievable yet, see above |
| No GPU, no driver, driver crashes on probe | **CPU**, no error |
| Non-x86-64 / non-glibc ABI | the DSO does not load: **CPU** |
| `--gpu vulkan` forced without Vulkan | fatal error — never pass an explicit backend |

## Download and verify

The weights are release assets, not files in this repository (GitHub rejects
blobs over 100 MB). Download, then confirm the checksum before using them:

```sh
# the recommended build: LFM3-1.7B with the pi LoRA merged into the weights
gh release download models-v4 --repo andrea-tomassi/awesome-llamafiles --pattern '*.llamafile'
sha256sum LFM3-1.7B-pi-Q6_K.llamafile
# 961125f363ed9746b5e6bf4c30e7711f4dac49a324ee15024811fd9820a8ddf7

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
| coding agent (default) | LFM3-1.7B-pi-Q6_K | 131072 | adapter-tuned for tool use; ~3.1 GB RAM; the OOTB preset |
| long-context review | LFM3-1.7B-pi-Q6_K | 262144 | full trained context; ~4.8 GB RAM |
| tiny host | LFM3-1.7B-pi-Q6_K | 32768 | minimal KV cache; faster load, shorter memory; ~1.9 GB RAM |
| smallest footprint, no adapter | LFM3-1.7B-QAD-Q4_0 | 131072 | ~2.7 GB RAM; the `models-v1` build |
| tiny host, no adapter | LFM3-1.7B-QAD-Q4_0 | 32768 | ~1.9 GB RAM |

Thinking is on by default in the LFM3 chat template. Either leave room in
`max_tokens` for the reasoning pass, or pass
`chat_template_kwargs: {"enable_thinking": false}` to skip it; a tight budget
otherwise spends everything on reasoning and returns empty content.

## Provenance of the `models-v4` pi build

The only artifact here with a recorded build recipe. Everything below was run
by hand; there is no CI for it.

| Stage | Input / command |
|-------|-----------------|
| Base | `LiquidAI/LFM3-1.7B-GGUF` → `LFM3-1.7B-F16.gguf` (3.47 GB) |
| Adapter | `harness-hub/LFM3-1.7B-pi-GGUF` → `LFM3-1.7B-harbor-pi-lora-F16.gguf` (66 MB, rank 32) |
| Merge | `llama-export-lora -m LFM3-1.7B-F16.gguf --lora <adapter> -o merged-f16.gguf` (llama.cpp `a07831a`) |
| Quantize | `llama-quantize merged-f16.gguf merged-Q6_K.gguf Q6_K` |
| Package | `zipalign -j0 LFM3-1.7B-pi-Q6_K.llamafile LFM3-1.7B-pi-Q6_K.gguf .args` on a v0.10.7 runtime |

The F16 base is used on purpose: the merge dequantizes to F32 and applies the
delta, so merging into an already-quantized base would lock that error in.
`.args` holds three lines (`-m`, `/zip/LFM3-1.7B-pi-Q6_K.gguf`, `...`), the same
shape as `models-v1`. The adapter is baked into the weights, so there is no
second file and no `--lora` flag at runtime.

Checks run on the published file:

- `unzip -l`: exactly the GGUF and `.args`, nothing inherited from the runtime
  artifact it was carved from.
- `/props`: `model_path = /zip/LFM3-1.7B-pi-Q6_K.gguf`, `n_ctx_slot = 131072`.
- `/v1/chat/completions` with a tool schema returns `finish_reason: tool_calls`
  and well-formed JSON arguments.
- Against an unmerged control built through the same pipeline on the same
  runtime, the merged model differs at temperature 0 on the same prompts, which
  is what proves the adapter is live rather than silently dropped.

What this build does **not** establish:

- **Quality.** The upstream card reports +10.7 points and 37% fewer tool calls
  for the adapter, measured with vLLM at BF16. That has never been reproduced
  through llama.cpp, and a Q6_K quantization of the merge will not match it
  exactly.
- **Speed.** CPU generation measured ~4.3 tok/s on a 12-core shared VM. For
  agent loops, serve a GGUF on a GPU host instead (see above).

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
  that you changed them (Section 4(b)). The `models-v4` build is such a
  modification: [harness-hub/LFM3-1.7B-pi](https://huggingface.co/harness-hub/LFM3-1.7B-pi),
  a LoRA adapter for LFM3-1.7B distributed under the same LFM terms, merged
  into the base weights and re-quantized to Q6_K, then packaged as a llamafile.
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
