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
are released by LiquidAI under their open model license; redistribution here
is with attribution to LiquidAI. Read the model card at
https://huggingface.co/LiquidAI/LFM3-1.7B-GGUF before redistributing.
