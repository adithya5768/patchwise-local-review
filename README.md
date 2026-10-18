# Running PatchWise on a local model with 16 GB VRAM

[Browse the full interactive page](https://adithya5768.github.io/patchwise-local-review/)

I've been playing with [PatchWise](https://github.com/qualcomm/PatchWise) and wanted to see if it could actually find bugs in real kernel code, especially with a local model on my PC. Borrowing [Sashiko](https://github.com/sashiko-dev/sashiko)'s idea of reviewing a known buggy commit, I ran it on a commit I knew had a real bug and saw what happened. This repo contains the full output. I ran Claude Code v2.1.296's `/code-review` on the same commit for comparison, same model and setup — but this is really a story about PatchWise.

## TL;DR

PatchWise found 6 real issues including the actual OOB write bug (later fixed upstream). Claude Code found 3 — and missed the bug entirely. Setup was 3 lines of YAML + one llama.cpp command. Tooling architecture matters more than model size. This wasn't Fable or Astra — it was a Q4 quantized model running on an RTX 4070 Ti Super.

## The Commit

`f7a22071dbf3` — SMC: Fix missing bounds check on memcpy offset + size vs buffer length. This commit fixed an OOB access bug introduced in an earlier SMC loopback-ism patch. It was fixed over two years later in `78237e3c0720` ("dibs: loopback: validate offset and size in move_data()").

## The Model

- **Model**: [unsloth/Qwen3.6-35B-A3B-MTP-GGUF](https://huggingface.co/unsloth/Qwen3.6-35B-A3B-MTP-GGUF) (UD-Q4_K_XL)
- **Size**: 22.9 GB after Q4 quantization, 3B active parameters
- **Served via**: llama.cpp (OpenAI API compatible)
- **Hardware**: AMD Ryzen 5 7600X, NVIDIA RTX 4070 Ti Super, 16 GB VRAM, 32 GB RAM
- **Usage**: ~14 GB dedicated GPU VRAM, ~29 GB system RAM

## Results

PatchWise independently arrived at the bounds check bug. It did not cheat (zero git tool calls — no `git log`, no `git show`, no search for the fix commit).
