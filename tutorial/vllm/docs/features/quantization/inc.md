# inc.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/features/quantization/inc.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析

**EN:** This document covers Intel Neural Compressor's **AutoRound** quantization algorithm and its integration with vLLM. AutoRound supports INT2–INT8, MXFP8, MXFP4, NVFP4, and GGUF formats, with both CLI and Python API workflows for quantization and deployment.

**CN:** 本文档介绍 Intel Neural Compressor 的 **AutoRound** 量化算法及其与 vLLM 的集成，AutoRound 支持 INT2–INT8、MXFP8、MXFP4、NVFP4 和 GGUF 格式，提供量化和部署的 CLI 及 Python API 工作流。

### Key Features / 主要特性

**EN:** Superior accuracy at 2–3 bits; fast mixed bits/dtypes scheme generation; supports AutoRound/AutoAWQ/AutoGPTQ/GGUF export formats; per-layer mixed-bit quantization; RTN mode for quick quantization; 10+ VLM support; 10+ backends.

**CN:** 2–3 位时精度优异；快速混合位/类型方案生成；支持导出 AutoRound/AutoAWQ/AutoGPTQ/GGUF 格式；逐层混合位量化；RTN 模式快速量化；支持 10+ 视觉语言模型；10+ 后端。

### Supported Recipes on Intel Platforms / Intel 平台支持的方案

**EN:** Currently vLLM supports W4A16 and W8A16 (weight-only). Additional recipes planned for future releases.

**CN:** 目前 vLLM 支持 W4A16 和 W8A16（仅权重量化），更多方案计划在未来版本支持。

### Quantization Workflow / 量化工作流

**EN:** Install `auto-round`. Use CLI (`auto-round --model ... --scheme W4A16 --format auto_round`) or Python API (`AutoRound(model_name, scheme="W4A16").quantize_and_save(output_dir)`). Multiple accuracy/speed trade-offs available via `nsamples` and `iters`.

**CN:** 安装 `auto-round`。使用 CLI（`auto-round --model ... --scheme W4A16 --format auto_round`）或 Python API（`AutoRound(model_name, scheme="W4A16").quantize_and_save(output_dir)`），可通过 `nsamples` 和 `iters` 调节精度/速度权衡。

### Deployment in vLLM / 在 vLLM 中部署

**EN:** Use `vllm serve Intel/DeepSeek-R1-0528-Qwen3-8B-int4-AutoRound`. Note: Intel GPU/CPU deployments require `--enforce-eager` for `wNa16` models.

**CN:** 使用 `vllm serve Intel/DeepSeek-R1-0528-Qwen3-8B-int4-AutoRound`。注意：Intel GPU/CPU 部署 `wNa16` 模型需要 `--enforce-eager`。

### Evaluation / 评估

**EN:** Use `lm_eval` with vLLM backend; set appropriate `max_model_len` and `max_num_batched_tokens`.

**CN:** 使用 `lm_eval` 配合 vLLM 后端；设置合适的 `max_model_len` 和 `max_num_batched_tokens`。

## Key Concepts / 关键概念

- **EN:** AutoRound — Intel's advanced LLM quantization algorithm with best/base/light accuracy recipes. **CN:** AutoRound——Intel 的高级 LLM 量化算法，提供 best/base/light 精度方案。
- **EN:** Per-layer mixed-bit — different layers can have different bit widths for fine-grained accuracy-efficiency control. **CN:** 逐层混合位——不同层可使用不同位宽，实现细粒度精度-效率控制。
- **EN:** W4A16 / W8A16 — weight-only quantization modes where activations remain in 16-bit. **CN:** W4A16 / W8A16——仅权重量化模式，激活保持 16 位。
- **EN:** RTN (Round-To-Nearest) — fastest quantization recipe with slight accuracy loss; no calibration dataset required. **CN:** RTN（舍入到最近）——最快的量化方案，精度损失轻微，无需校准数据集。
- **EN:** `low_gpu_mem_usage` — option to save ~20 GB GPU memory during quantization at ~30% speed cost. **CN:** `low_gpu_mem_usage`——量化期间节省约 20 GB GPU 内存的选项，代价是约 30% 的速度下降。
