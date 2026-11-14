# pi-llamafile-models

Ready-made `.llamafile` weights for the
[pi-llamafile](https://github.com/andrea-tomassi/pi-llamafile) extension.
**Distro, not a factory**: these files are copied as-is; building llamafiles
from GGUF (quantization, pinning, CI) is explicitly future work.

Install: copy the files into any directory the extension discovers
(`~/.pi/agent/models/` is the conventional one), or point `PI_LLAMAFILE_DIR`
at this checkout.

## Models

| File | Source | SHA256 |
|------|--------|--------|
| `LFM3-1.7B-QAD-Q4_0.llamafile` (1.05 GB) | LiquidAI LFM3-1.7B QAD Q4_0, wrapped as llamafile v0.10.7 (engine: llama.cpp @ `e853ec9a8`) | `6b01eabc3f1b7f17a68b46b780020f444cc5ee0951f3a7e036f2344cc6697be6` |

## License of the weights

The model weights are **not** covered by this repository's MIT license. They
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
