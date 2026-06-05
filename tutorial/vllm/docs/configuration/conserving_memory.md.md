# conserving_memory.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/configuration/conserving_memory.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
**EN:** This guide collects practical ways to reduce memory usage and avoid OOMs in vLLM. It covers tensor parallelism, quantization, context and batch-size limits, CUDA graph reduction, cache sizing, multimodal item limits, and multimodal processor knobs.

**EN:** A notable strength is that it distinguishes between several memory domains: model weights, KV cache, multimodal activation profiling, processor caches, and actual runtime media inputs. That helps users choose the right knob instead of blindly lowering everything.

**CN:** 这份指南汇总了在 vLLM 中降低内存占用、避免 OOM 的实用方法，包括张量并行、量化、上下文与批大小限制、减少 CUDA Graph、调整缓存大小、多模态输入上限，以及多模态处理器参数。

**CN:** 它的一个亮点是清楚区分了多种内存来源：模型权重、KV cache、多模态激活预留、处理器缓存，以及运行时真实媒体输入。这能帮助用户更有针对性地调参，而不是盲目把所有值都调低。

## Key Concepts / 关键概念
- **EN:** OOM mitigation — Provides a menu of knobs for reducing memory pressure under different bottlenecks.
- **CN:** OOM 缓解 — 针对不同瓶颈提供多种降低内存压力的调节手段。
- **EN:** Multimodal limits — Shows how per-modality caps and size hints influence reserved memory.
- **CN:** 多模态限制 — 说明按模态设置数量上限和尺寸提示如何影响预留内存。
- **EN:** Cache tuning — Separates CPU KV cache and multimodal processor cache adjustments.
- **CN:** 缓存调优 — 区分 CPU KV cache 与多模态处理器缓存的调节方式。
- **EN:** CUDA graph tradeoff — Explains the speed-versus-memory exchange of graph capture settings.
- **CN:** CUDA Graph 权衡 — 解释图捕获设置在速度与显存之间的权衡。
