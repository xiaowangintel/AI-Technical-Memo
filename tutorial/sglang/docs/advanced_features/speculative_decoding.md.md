# speculative_decoding.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/advanced_features/speculative_decoding.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: SGLang provides several speculative decoding options, including EAGLE-2/EAGLE-3, MTP, classic draft-model decoding, and an NGRAM-based variant. / 该文档围绕 投机解码 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Overview
**EN:** SGLang provides several speculative decoding options, including EAGLE-2/EAGLE-3, MTP, classic draft-model decoding, and an NGRAM-based variant.
**CN:** 本节围绕 Overview 展开，概述了 MTP, LLM, decoding, NGRAM-based 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Jump to sections
**EN:** This content focuses on Jump to sections and highlights Decoding, Speculative, EAGLE-2, EAGLE-2 Decoding.
**CN:** 本节围绕 Jump to sections 展开，概述了 Decoding, Speculative, EAGLE-2, EAGLE-2 Decoding 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick guidance
**EN:** **Best speed/quality (recommended)**: Use **EAGLE-3** with --speculative-algorithm EAGLE3. **Strong default / broad compatibility**: Use **EAGLE-2** with --speculative-algorithm EAGLE.
**CN:** 本节围绕 Quick guidance 展开，概述了 EAGLE, NGRAM, Enable, EAGLE-2 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Method comparison (mini table)
**EN:** This section provides a comparison table for Method comparison (mini table), covering columns such as Method, Draft source, Separate draft model? and examples such as EAGLE-2, EAGLE-2 + torch.compile, EAGLE-2 + FR-Spec, EAGLE-3.
**CN:** 本节围绕 Method comparison (mini table) 展开，概述了 EAGLE-2, EAGLE, EAGLE3, STANDALONE 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Performance Highlights
**EN:** This section provides a comparison table for Performance Highlights, covering columns such as Method, Throughput (tokens/s) and examples such as SGLang (w/o speculative, 1x H100), SGLang + EAGLE-2 (1x H100), SGLang + EAGLE-3 (1x H100).
**CN:** 本节围绕 性能 Highlights 展开，概述了 H100, EAGLE3, tokens/s, see 等要点，并说明相关配置、流程、示例或限制条件。

### Section: EAGLE Decoding
**EN:** This section provides a comparison table for EAGLE Decoding, covering columns such as Parameter, Description, Default and examples such as --speculative-draft-model-path, --speculative-num-steps, --speculative-eagle-topk, --speculative-num-draft-tokens.
**CN:** 本节围绕 EAGLE Decoding 展开，概述了 model, Auto, EAGLE, draft 等要点，并说明相关配置、流程、示例或限制条件。

### Section: EAGLE-2 Decoding
**EN:** You can enable EAGLE-2 Decoding by setting --speculative-algorithm EAGLE and choosing an appropriate model.
**CN:** 本节围绕 EAGLE-2 Decoding 展开，概述了 EAGLE, model, Send, List 等要点，并说明相关配置、流程、示例或限制条件。

### Section: EAGLE-2 Decoding with torch.compile
**EN:** You can optionally enable torch.compile to apply kernel-level optimizations (operator fusion, autotune) to the draft model. The actual speedup depends on your hardware, model architecture, and batch size.
**CN:** 本节围绕 EAGLE-2 Decoding with torch.compile 展开，概述了 model, --enable-torch-compile, --torch-compile-max-bs, H100 等要点，并说明相关配置、流程、示例或限制条件。

### Section: EAGLE-2 Decoding via Frequency-Ranked Speculative Sampling
**EN:** By employing a truncated high-frequency token vocabulary in the draft model, EAGLE speculative decoding reduces lm_head computational overhead while accelerating the pipeline without quality degradation. For more details, check out the paper.
**CN:** 本节围绕 EAGLE-2 Decoding via Frequency-Ranked Speculative Sampling 展开，概述了 EAGLE, model, tokens, FR-Spec 等要点，并说明相关配置、流程、示例或限制条件。

### Section: EAGLE-3 Decoding
**EN:** You can enable EAGLE-3 decoding by setting --speculative-algorithm EAGLE3 and choosing an appropriate model.
**CN:** 本节围绕 EAGLE-3 Decoding 展开，概述了 EAGLE3, model, Send, List 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Multi Token Prediction
**EN:** We support MTP (Multi-Token Prediction) in SGLang by using speculative decoding. We use XiaomiMiMo/MiMo-7B-RL as an example here (for DeepSeek MTP usage, refer to deepseek_v32 doc).
**CN:** 本节围绕 Multi 令牌 Prediction 展开，概述了 XiaomiMiMo/MiMo-7B-RL, MTP, XiaomiMiMo, MiMo-7B-RL 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Standalone Speculative Decoding (Small Draft Model)
**EN:** This section provides a comparison table for Standalone Speculative Decoding (Small Draft Model), covering columns such as Parameter, Description, Default and examples such as --speculative-draft-model-path, --speculative-num-steps, --speculative-eagle-topk, --speculative-num-draft-tokens.
**CN:** 本节围绕 Standalone 投机解码 (Small Draft 模型) 展开，概述了 STANDALONE, model, draft, --speculative-draft-model-path 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Speculative Decoding V2 (Overlap Scheduler)
**EN:** SGLang provides an **experimental Speculative Decoding V2** implementation that enables an overlap scheduler and uses V2 speculative workers (e.g. StandaloneWorkerV2, EAGLEWorkerV2).
**CN:** 本节围绕 投机解码 V2 (Overlap Scheduler) 展开，概述了 --speculative-eagle-topk, speculative-eagle-topk, SpecV2, STANDALONE 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Ngram Speculative Decoding
**EN:** SGLang also supports **ngram-based speculative decoding** (no separate draft model). It retrieves draft tokens from an ngram cache built from previously generated tokens, and then verifies them with the target model.
**CN:** 本节围绕 Ngram 投机解码 展开，概述了 NGRAM, draft, model, Enable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Ngram-specific parameters
**EN:** This section provides a comparison table for Ngram-specific parameters, covering columns such as Parameter, Description, Default and examples such as --speculative-num-draft-tokens, --speculative-ngram-min-bfs-breadth, --speculative-ngram-max-bfs-breadth, --speculative-ngram-match-type.
**CN:** 本节围绕 Ngram-specific parameters 展开，概述了 BFS, Ngram,  |
| , Maximum 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Full Parameter Reference
**EN:** Below is a comprehensive list of all speculative decoding parameters available in SGLang:
**CN:** 本节围绕 Full Parameter Reference 展开，概述了 Below, list, decoding, available 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Core parameters
**EN:** This section provides a comparison table for Core parameters, covering columns such as Parameter, Type, Default and examples such as --speculative-algorithm, --speculative-draft-model-path, --speculative-draft-model-revision, --speculative-draft-load-format.
**CN:** 本节围绕 Core parameters 展开，概述了 str, draft, model, int 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Ngram-specific parameters
**EN:** This section provides a comparison table for Ngram-specific parameters, covering columns such as Parameter, Type, Default and examples such as --speculative-ngram-min-bfs-breadth, --speculative-ngram-max-bfs-breadth, --speculative-ngram-match-type, --speculative-ngram-max-trie-depth.
**CN:** 本节围绕 Ngram-specific parameters 展开，概述了 int,  | , BFS, Maximum 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Environment variables
**EN:** This section provides a comparison table for Environment variables, covering columns such as Variable, Default, Description and examples such as SGLANG_ENABLE_SPEC_V2, SGLANG_NGRAM_FORCE_GREEDY_VERIFY.
**CN:** 本节围绕 Environment variables 展开，概述了 SGLANG_ENABLE_SPEC_V2, SGLANG_NGRAM_FORCE_GREEDY_VERIFY, Force, Variable 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Other related flags
**EN:** This section provides a comparison table for Other related flags, covering columns such as Parameter, Description and examples such as --enable-multi-layer-eagle, --enable-torch-compile, --torch-compile-max-bs.
**CN:** 本节围绕 Other related flags 展开，概述了 Enable, torch.compile, EAGLE, MiMoV2 等要点，并说明相关配置、流程、示例或限制条件。

### Section: OOM Troubleshooting
**EN:** > [!WARNING] > **Out of Memory (OOM)?** Speculative decoding may increase GPU memory usage because the draft tree, CUDA graphs, and verification-related buffers consume additional VRAM. If you encounter OOM errors, try the following adjustments.
**CN:** 本节围绕 OOM Troubleshooting 展开，概述了 OOM, Out, GPU, CUDA 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 1: Lower static memory fraction (most effective)
**EN:** ``bash --mem-fraction-static 0.5 # when omitted, this value is auto-computed ` - --mem-fraction-static` controls the memory budget for model weights + KV cache pool. Lowering it directly increases dynamic headroom for activations and CUDA graph buffers.
**CN:** 本节围绕 Step 1: Lower static memory fraction (most effective) 展开，概述了 CUDA, value, omitted, Lowering 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 2: Reduce CUDA graph batch size
**EN:** ``bash # Fewer CUDA graph captures = less memory reserved --cuda-graph-max-bs 4 # or even 2 for tight memory situations ` - If omitted, --cuda-graph-max-bs` is auto-selected based on GPU memory and TP size, and can be much larger on high-memory GPUs.
**CN:** 本节围绕 Step 2: Reduce CUDA graph batch size 展开，概述了 memory, GPU, GPUs, cuda-graph-max-bs 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 3: Reduce draft tree size
**EN:** This content focuses on Step 3: Reduce draft tree size and highlights memory, These, After, Before.
**CN:** 本节围绕 Step 3: Reduce draft tree size 展开，概述了 memory, These, After, Before 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Step 4: Limit concurrent requests
**EN:** ``bash # Fewer concurrent requests lowers in-flight load and can reduce OOM risk --max-running-requests 4 ``
**CN:** 本节围绕 Step 4: Limit concurrent requests 展开，概述了 OOM, Fewer, load, risk 等要点，并说明相关配置、流程、示例或限制条件。

### Section: Quick OOM recovery recipe
**EN:** Increase --mem-fraction-static` last, only after the run is stable.
**CN:** 本节围绕 Quick OOM recovery recipe 展开，概述了 OOM, Then, EAGLE, Increase 等要点，并说明相关配置、流程、示例或限制条件。

### Section: References
**EN:** EAGLE process is as follows: - Within EAGLE the draft model predicts the next feature vector, i.e. the last hidden state of the original LLM, using the feature sequence $(f_1, ..., f_k)$ and the token sequence $(t_2, ..., t_ )$.
**CN:** 本节围绕 References 展开，概述了 EAGLE, EAGLE-3, SpecForge, EAGLE-2 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** model / **CN:** 模型
- **EN:** EAGLE / **CN:** EAGLE
- **EN:** draft / **CN:** draft
- **EN:** EAGLE-2 / **CN:** EAGLE-2
- **EN:** STANDALONE / **CN:** STANDALONE
- **EN:** --speculative-eagle-topk / **CN:** --speculative-eagle-topk
- **EN:** speculative-eagle-topk / **CN:** speculative-eagle-topk
- **EN:** EAGLE3 / **CN:** EAGLE3

## Dependencies / 依赖关系
- `adaptive_speculative_decoding.md`
- `../basic_usage/deepseek_v32.md`
- `bench_speculative.py`
- `//github.com/sgl-project/sglang/blob/main/scripts/playground/bench_speculative.py`
