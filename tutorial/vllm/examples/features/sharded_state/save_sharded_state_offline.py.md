# save_sharded_state_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/sharded_state/save_sharded_state_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Saves each worker's model state dict directly to a checkpoint, which enables a fast load path for large tensor-parallel models where each worker only needs to read its own shard rather than the entire checkpoint / 演示 vLLM 示例目录中与 save sharded state offline 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Saves each worker's model state dict directly to a checkpoint, which enables a
fast load path for large tensor-parallel models where each worker only needs to
read its own shard rather than the entire checkpoint.

Example usage:

python save_sharded_state_offline.py \
    --model /path/to/load \
    --tensor-parallel-size 8 \
    --output /path/to/save

Then, the model can be loaded with

llm = LLM(
    model="/path/to/save",
    load_format="sharded_state",
    tensor_parallel_size=8,
)
"""
```
**EN:** Saves each worker's model state dict directly to a checkpoint, which enables a fast load path for large tensor-parallel models where each worker only needs to read its own shard rather than the entire checkpoint.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import os
import shutil
from pathlib import Path

from vllm import LLM, EngineArgs
from vllm.model_executor.model_loader import ShardedStateLoader
from vllm.utils.argparse_utils import FlexibleArgumentParser
```
**EN:** This block loads helper libraries such as os, shutil, and pathlib and pulls in vLLM APIs like vllm, vllm.model_executor.model_loader, and vllm.utils.argparse_utils.
**CN:** 这一部分加载 os、shutil，以及 pathlib 等辅助库，并引入 vllm、vllm.model_executor.model_loader，以及 vllm.utils.argparse_utils 等 vLLM API。

### Function: parse_args
```python
def parse_args():
    parser = FlexibleArgumentParser()
    EngineArgs.add_cli_args(parser)
    parser.add_argument(
        "--output", "-o", required=True, type=str, help="path to output checkpoint"
    )
    parser.add_argument(
        "--file-pattern",
        type=str,
        default=ShardedStateLoader.DEFAULT_PATTERN,
        help="string pattern of saved filenames",
    )
    parser.add_argument(
        "--max-file-size",
        type=int,
        default=5 * 1024**3,
        help="max size (in bytes) of each safetensors file",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, FlexibleArgumentParser, EngineArgs.add_cli_args, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、FlexibleArgumentParser、EngineArgs.add_cli_args，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    engine_args = EngineArgs.from_cli_args(args)
    if engine_args.enable_lora:
        raise ValueError("Saving with enable_lora=True is not supported!")
    model_path = engine_args.model
    if not Path(model_path).is_dir():
        raise ValueError("model path must be a local directory")
    # Create LLM instance from arguments
    llm = LLM.from_engine_args(engine_args)
    # Prepare output directory
    Path(args.output).mkdir(exist_ok=True)
    # Dump worker states to output directory

    llm.llm_engine.engine_core.save_sharded_state(
        path=args.output, pattern=args.file_pattern, max_size=args.max_file_size
    )

    # Copy metadata files to output directory
    for file in os.listdir(model_path):
        if os.path.splitext(file)[1] not in (".bin", ".pt", ".safetensors"):
            if os.path.isdir(os.path.join(model_path, file)):
                shutil.copytree(
                    os.path.join(model_path, file), os.path.join(args.output, file)
                )
            else:
                shutil.copy(os.path.join(model_path, file), args.output)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include os.path.join, ValueError, Path, EngineArgs.from_cli_args, and LLM.from_engine_args.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 os.path.join、ValueError、Path、EngineArgs.from_cli_args，以及 LLM.from_engine_args。

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

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.model_executor.model_loader`, `vllm.utils.argparse_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `os`, `shutil`, `pathlib` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `parser.add_argument`, `FlexibleArgumentParser`, `EngineArgs.add_cli_args`, `parser.parse_args`, `os.path.join`, `ValueError`, `Path`, `EngineArgs.from_cli_args` reveal the main execution path / 这些调用体现了主要执行链路。
