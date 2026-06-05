# tests.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `./docs/contributing/model/tests.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** Defines the minimum testing bar for new model contributions: the model must be loadable in CI through the test registry using dummy weights.
- **CN:** 本文定义了新模型贡献的最低测试门槛：模型必须通过测试注册表和 dummy 权重在 CI 中成功加载。
- **EN:** It then outlines optional but valuable correctness checks against Hugging Face outputs, plus multimodal processing tests that compare text and token entry paths.
- **CN:** 随后，文档介绍了更有价值但可选的正确性测试，包括与 Hugging Face 输出对比，以及比较文本输入与 token 输入路径的多模态处理测试。

## Key Concepts / 关键概念
- **EN:** Model loading test — Adding an example repository to the test registry verifies that initialization succeeds.
- **CN:** 模型加载测试 — 把示例仓库加入测试注册表，可以验证模型初始化流程是否成功。
- **EN:** Exact vs logprob checks — Generative models can be compared by exact text match or top-k logprob similarity.
- **CN:** 精确输出与 logprob 检查 — 生成式模型既可比对精确文本，也可比对 top-k logprob 相似性。
- **EN:** Pooling similarity — Pooling models are validated with cosine similarity rather than token generation.
- **CN:** Pooling 相似度 — Pooling 模型通常通过余弦相似度而不是文本生成结果来验证。
- **EN:** Multimodal processing tests — Common multimodal tests ensure equivalent behavior across text, token, and cached multimodal inputs.
- **CN:** 多模态处理测试 — 通用多模态测试会验证文本、token 与缓存多模态输入之间的一致性。
