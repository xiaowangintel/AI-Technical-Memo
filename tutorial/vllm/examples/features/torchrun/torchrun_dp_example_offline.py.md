# torchrun_dp_example_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/torchrun/torchrun_dp_example_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: experimental support for data-parallel inference with torchrun Note the data load balancing and distribution is done out of the vllm engine, no internal lb supported in external_launcher mode / 演示 vLLM 示例目录中与 torchrun dp example offline 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
experimental support for data-parallel inference with torchrun
Note the data load balancing and distribution is done out of the vllm engine,
no internal lb supported in external_launcher mode.

To run this example:
```bash
$ torchrun --nproc-per-node=2 examples/features/torchrun/torchrun_dp_example_offline.py
```

With custom parallelism settings:
```bash
$ torchrun --nproc-per-node=8 examples/features/torchrun/torchrun_dp_example_offline.py \
    --tp-size=2 --pp-size=1 --dp-size=4 --enable-ep
```
"""  # noqa: E501
```
**EN:** experimental support for data-parallel inference with torchrun Note the data load balancing and distribution is done out of the vllm engine, no internal lb supported in external_launcher mode.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse

from vllm import LLM, SamplingParams
```
**EN:** This block loads helper libraries such as argparse and pulls in vLLM APIs like vllm.
**CN:** 这一部分加载 argparse 等辅助库，并引入 vllm 等 vLLM API。

### Function: parse_args
```python
def parse_args():
    parser = argparse.ArgumentParser(
        description="Data-parallel inference with torchrun"
    )
    parser.add_argument(
        "--tp-size",
        type=int,
        default=1,
        help="Tensor parallel size (default: 1)",
    )
    parser.add_argument(
        "--pp-size",
        type=int,
        default=1,
        help="Pipeline parallel size (default: 1)",
    )
    parser.add_argument(
        "--dp-size",
        type=int,
        default=2,
    # ... key logic omitted for brevity ...
    )
    parser.add_argument(
        "--gpu-memory-utilization",
        type=float,
        default=0.6,
        help="GPU memory utilization (default: 0.6)",
    )
    parser.add_argument(
        "--seed",
        type=int,
        default=1,
        help="Random seed (default: 1)",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, argparse.ArgumentParser, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、argparse.ArgumentParser，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Top-level setup
```python
args = parse_args()


# Create prompts, the same across all ranks
prompts = [
    "Hello, my name is",
    "The president of the United States is",
    "The capital of France is",
    "The future of AI is",
]

# Create sampling parameters, the same across all ranks
# ... key logic omitted for brevity ...
device_group = get_world_group().device_group
```

3. to access the model directly in every rank, use the following code:
```python
llm.llm_engine.model_executor.driver_worker.worker.model_runner.model
```
"""
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as args, prompts, sampling_params, llm, dp_rank, and dp_size. It also performs early helper calls such as parse_args, SamplingParams, LLM, enumerate, and llm.generate.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 args、prompts、sampling_params、llm、dp_rank，以及 dp_size 等变量。它还会提前执行 parse_args、SamplingParams、LLM、enumerate，以及 llm.generate 等辅助调用。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `parse_args` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `parser.add_argument`, `argparse.ArgumentParser`, `parser.parse_args`, `parse_args`, `SamplingParams`, `LLM`, `enumerate`, `llm.generate` reveal the main execution path / 这些调用体现了主要执行链路。
