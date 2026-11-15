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

| File | Source | SHA256 | Best for |
|------|--------|--------|----------|
| `LFM3-1.7B-QAD-Q4_0.llamafile` (1.05 GB) | LiquidAI LFM3-1.7B QAD Q4_0, llamafile v0.10.7 (engine: llama.cpp @ `e853ec9a8`) | `6b01eabc3f1b7f17a68b46b780020f444cc5ee0951f3a7e036f2344cc6697be6` | coding agent, reasoning, 128k context on ~3 GB RAM |

## Download and verify

The weights are release assets, not files in this repository (GitHub rejects
blobs over 100 MB). Download, then confirm the checksum before using them:

```sh
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
