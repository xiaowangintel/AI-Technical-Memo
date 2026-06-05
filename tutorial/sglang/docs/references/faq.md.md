# faq.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/references/faq.md`
- **Repository**: `sgl-project/sglang`
- **Purpose**: Troubleshooting This page lists common errors and tips for resolving them. CUDA Out of Memory If you encounter out-of-memory (OOM) errors, you can adjust the following parameters:. / 该文档围绕 Troubleshooting and Frequently Asked Questions 展开，帮助读者理解相关背景、配置方式、使用流程或限制条件。

## Content Analysis / 内容分析

### Section: Troubleshooting
**EN:** This page lists common errors and tips for resolving them.
**CN:** 本节围绕 Troubleshooting 展开，概述了 tips, lists, common, errors 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CUDA Out of Memory
**EN:** If you encounter out-of-memory (OOM) errors, you can adjust the following parameters: - If OOM occurs during prefill, try reducing --chunked-prefill-size to 4096 or 2048. This saves memory but slows down the prefill speed for long prompts.
**CN:** 本节围绕 CUDA Out of Memory 展开，概述了 OOM, memory, --mem-fraction-static, try 等要点，并说明相关配置、流程、示例或限制条件。

### Section: CUDA Error: Illegal Memory Access Encountered
**EN:** This error may result from kernel errors or out-of-memory issues: - If it is a kernel error, resolving it may be challenging. Please file an issue on GitHub.
**CN:** 本节围绕 CUDA Error: Illegal Memory Access Encountered 展开，概述了 may, error, Out, OOM 等要点，并说明相关配置、流程、示例或限制条件。

### Section: The server hangs
**EN:** If the server hangs during initialization or running, it can be memory issues (out of memory), network issues (nccl errors), or other bugs in sglang.
**CN:** 本节围绕 The 服务端 hangs 展开，概述了 memory, decrease, initialization, out 等要点，并说明相关配置、流程、示例或限制条件。

### Section: The results are not deterministic, even with a temperature of 0
**EN:** You may notice that when you send the same request twice, the results from the engine will be slightly different, even when the temperature is set to 0.
**CN:** 本节围绕 The results are not deterministic, even with a temperature of 0 展开，概述了 different, prefix, dynamic, caching 等要点，并说明相关配置、流程、示例或限制条件。

## Key Concepts / 关键概念
- **EN:** OOM / **CN:** OOM
- **EN:** memory / **CN:** memory
- **EN:** --mem-fraction-static / **CN:** --mem-fraction-static
- **EN:** errors / **CN:** errors
- **EN:** during / **CN:** during
- **EN:** try / **CN:** try
- **EN:** may / **CN:** may
- **EN:** CUDA / **CN:** CUDA

## Dependencies / 依赖关系
- No explicit repository-local references detected / 未检测到显式的仓库内引用
