# sequence_reward_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/reward/sequence_reward_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example offline usage of sequence reward models / 演示奖励模型推理流程。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Example offline usage of sequence reward models.

The key distinction between sequence classification and token classification
lies in their output granularity: sequence classification produces a single
result for an entire input sequence, whereas token classification yields a
result for each individual token within the sequence.
"""
```
**EN:** Example offline usage of sequence reward models.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from argparse import Namespace

from vllm import LLM, EngineArgs
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.print_utils import print_embeddings
```
**EN:** This block loads helper libraries such as argparse and pulls in vLLM APIs like vllm, vllm.utils.argparse_utils, and vllm.utils.print_utils.
**CN:** 这一部分加载 argparse 等辅助库，并引入 vllm、vllm.utils.argparse_utils，以及 vllm.utils.print_utils 等 vLLM API。

### Function: parse_args
```python
def parse_args():
    parser = FlexibleArgumentParser()
    parser = EngineArgs.add_cli_args(parser)
    # Set example specific arguments
    parser.set_defaults(
        model="Skywork/Skywork-Reward-V2-Qwen3-0.6B",
        runner="pooling",
        enforce_eager=True,
        max_model_len=1024,
        trust_remote_code=True,
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include FlexibleArgumentParser, EngineArgs.add_cli_args, parser.set_defaults, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 FlexibleArgumentParser、EngineArgs.add_cli_args、parser.set_defaults，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args: Namespace):
    # Sample prompts.
    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ]

    # Create an LLM.
    # You should pass runner="pooling" for reward models
    llm = LLM(**vars(args))

    # Generate rewards. The output is a list of PoolingRequestOutput.
    # Use pooling_task="classify" for sequence reward models.
    outputs = llm.encode(prompts, pooling_task="classify")

    # Print the outputs.
    print("\nGenerated Outputs:\n" + "-" * 60)
    for prompt, output in zip(prompts, outputs):
        rewards = output.outputs.data
        print(f"Prompt: {prompt!r}")
        print_embeddings(rewards.tolist(), prefix="Reward")
        print("-" * 60)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include print, LLM, llm.encode, zip, and print_embeddings.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 print、LLM、llm.encode、zip，以及 print_embeddings。

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
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。
- **Task-specific scoring / 任务特定打分**: Model outputs are converted into scores, labels, or reward values. / 模型输出会被转换为分数、标签或奖励值。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.utils.argparse_utils`, `vllm.utils.print_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `FlexibleArgumentParser`, `EngineArgs.add_cli_args`, `parser.set_defaults`, `parser.parse_args`, `print`, `LLM`, `llm.encode`, `zip` reveal the main execution path / 这些调用体现了主要执行链路。
