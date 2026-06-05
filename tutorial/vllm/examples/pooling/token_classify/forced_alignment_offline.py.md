# forced_alignment_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/token_classify/forced_alignment_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Offline forced alignment example using Qwen3-ForcedAligner-0.6B / 演示利用池化输出进行词元级分类。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Offline forced alignment example using Qwen3-ForcedAligner-0.6B.

Forced alignment takes audio and reference text as input and produces
word-level timestamps. The model predicts a time bin at each <timestamp>
token position; multiplying by ``timestamp_segment_time`` gives milliseconds.

Usage::

    python forced_alignment_offline.py \
        --model Qwen/Qwen3-ForcedAligner-0.6B
"""
```
**EN:** Offline forced alignment example using Qwen3-ForcedAligner-0.6B.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from argparse import Namespace

import numpy as np

from vllm import LLM, EngineArgs
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as argparse and numpy and pulls in vLLM APIs like vllm and vllm.utils.argparse_utils.
**CN:** 这一部分加载 argparse 和 numpy 等辅助库，并引入 vllm 和 vllm.utils.argparse_utils 等 vLLM API。

### Function: parse_args
```python
def parse_args():
    parser = FlexibleArgumentParser()
    parser = EngineArgs.add_cli_args(parser)
    parser.set_defaults(
        model="Qwen/Qwen3-ForcedAligner-0.6B",
        runner="pooling",
        enforce_eager=True,
        hf_overrides={"architectures": ["Qwen3ASRForcedAlignerForTokenClassification"]},
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include FlexibleArgumentParser, EngineArgs.add_cli_args, parser.set_defaults, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 FlexibleArgumentParser、EngineArgs.add_cli_args、parser.set_defaults，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: build_prompt
```python
def build_prompt(words: list[str]) -> str:
    """Build the forced alignment prompt from a word list.

    Format: <|audio_start|><|audio_pad|><|audio_end|>
            word1<timestamp><timestamp>word2<timestamp><timestamp>...
    """
    body = "<timestamp><timestamp>".join(words) + "<timestamp><timestamp>"
    return f"<|audio_start|><|audio_pad|><|audio_end|>{body}"
```
**EN:** Build the forced alignment prompt from a word list.. It works with parameters such as words. Key operations include join. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。它会处理 words 等参数。关键操作包括 join。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args: Namespace):
    llm = LLM(**vars(args))

    config = llm.llm_engine.vllm_config.model_config.hf_config
    timestamp_token_id = config.timestamp_token_id
    timestamp_segment_time = config.timestamp_segment_time

    # Example: align these words against a 5-second audio clip
    words = ["Hello", "world"]
    prompt = build_prompt(words)

    # Use a 5-second silent audio as placeholder (replace with real audio)
    sample_rate = 16000
    audio = np.zeros(sample_rate * 5, dtype=np.float32)

    outputs = llm.encode(
        [{"prompt": prompt, "multi_modal_data": {"audio": audio}}],
        pooling_task="token_classify",
    )

    for output in outputs:
        logits = output.outputs.data  # [num_tokens, classify_num]
        predictions = logits.argmax(dim=-1)
        token_ids = output.prompt_token_ids

        # Extract timestamps at <timestamp> positions
        ts_predictions = [
            pred.item() * timestamp_segment_time
            for tid, pred in zip(token_ids, predictions)
            if tid == timestamp_token_id
        ]

        # Pair up start/end times per word
        for i, word in enumerate(words):
            start_ms = ts_predictions[i * 2]
            end_ms = ts_predictions[i * 2 + 1]
            print(f"{word:15s} {start_ms / 1000:.3f}s - {end_ms / 1000:.3f}s")
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include LLM, build_prompt, np.zeros, llm.encode, and logits.argmax.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 LLM、build_prompt、np.zeros、llm.encode，以及 logits.argmax。

### Entry point
```python
if __name__ == "__main__":
    args = parse_args()
    main(args)
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parse_args and main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parse_args 和 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。
- **Task-specific scoring / 任务特定打分**: Model outputs are converted into scores, labels, or reward values. / 模型输出会被转换为分数、标签或奖励值。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse`, `numpy` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `parse_args`, `build_prompt`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `FlexibleArgumentParser`, `EngineArgs.add_cli_args`, `parser.set_defaults`, `parser.parse_args`, `join`, `LLM`, `build_prompt`, `np.zeros` reveal the main execution path / 这些调用体现了主要执行链路。
