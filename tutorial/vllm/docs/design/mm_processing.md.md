# mm_processing.md — Documentation Analysis / 文档分析

## Source / 来源
- **File**: `docs/design/mm_processing.md`
- **Repository**: vllm-project/vllm

## Content Analysis / 内容分析
- **EN:** The document explains how `BaseMultiModalProcessor` maintains the mapping between placeholder feature tokens (such as repeated `<image>` tokens) and the original multimodal inputs returned by a Hugging Face processor.  
  **CN:** 本文说明 `BaseMultiModalProcessor` 如何维护特征占位符 token（例如重复的 `<image>`）与 Hugging Face processor 产出的原始多模态输入之间的对应关系。
- **EN:** A central responsibility is prompt update detection: vLLM models prompt edits performed by HF processors with `PromptUpdate`, then checks whether updated tokens exist so the system can recover token-to-input correspondence.  
  **CN:** 文档的核心之一是提示词更新检测：vLLM 用 `PromptUpdate` 描述 HF processor 对 prompt 的修改，并通过检测更新后 token 是否存在来恢复 token 与输入的对应关系。
- **EN:** To support tokenized prompts in a separate process, the design introduces dummy text generation plus model-agnostic automatic prompt updating, so multimodal data can still go through HF processing without rewriting the processor itself.  
  **CN:** 为了支持在独立进程中处理已分词提示词，设计引入了 dummy text 生成与模型无关的自动 prompt 更新，从而无需重写 HF processor 也能处理多模态数据。
- **EN:** The document also addresses processor output caching for slow HF processors: cache hits are reused, missing items are processed in one batch, and prompt updates are re-applied afterward to keep tokens and multimodal data aligned.  
  **CN:** 文档还处理了慢速 HF processor 的输出缓存：命中项直接复用，缺失项批量处理，随后再次应用 prompt 更新，以保持 token 与多模态数据的一致性。

## Key Concepts / 关键概念
- **EN:** **`BaseMultiModalProcessor`** — the abstraction that connects HF processor outputs to vLLM prompt/token handling.  
  **CN:** **`BaseMultiModalProcessor`** —— 将 HF processor 输出与 vLLM 的 prompt/token 处理连接起来的核心抽象。
- **EN:** **`PromptUpdate`** — a structured description of how placeholder tokens are inserted or replaced in the prompt.  
  **CN:** **`PromptUpdate`** —— 对 prompt 中占位符 token 如何插入或替换的结构化描述。
- **EN:** **Dummy text** — synthetic text generated from multimodal input counts so HF processors can run even when the real text has already been tokenized.  
  **CN:** **Dummy text** —— 根据多模态输入数量生成的伪文本，使 HF processor 在真实文本已被分词后仍可正常运行。
- **EN:** **Automatic prompt updating** — `_apply_prompt_updates` replays prompt edits outside HF so output tokens remain consistent with multimodal features.  
  **CN:** **自动 prompt 更新** —— `_apply_prompt_updates` 在 HF 之外重放 prompt 修改，使输出 token 与多模态特征保持一致。
- **EN:** **Processor output caching** — only uncached multimodal items are reprocessed, which cuts repeated HF preprocessing cost for expensive models such as Qwen2-VL.  
  **CN:** **Processor 输出缓存** —— 仅重新处理未命中的多模态项，从而降低如 Qwen2-VL 这类高开销模型的重复预处理成本。
