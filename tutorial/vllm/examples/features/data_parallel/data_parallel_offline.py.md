# data_parallel_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/data_parallel/data_parallel_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Usage: Single node: python examples/features/data_parallel/data_parallel_offline.py --model="ibm-research/PowerMoE-3b" -dp=2 -tp=2 Multi-node: Node 0 (assume the node has ip of 10.99.48.128): python / 演示数据并行执行模式。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Usage:
Single node:
    python examples/features/data_parallel/data_parallel_offline.py \
            --model="ibm-research/PowerMoE-3b" \
            -dp=2 \
            -tp=2

Multi-node:
    Node 0 (assume the node has ip of 10.99.48.128):
            python examples/features/data_parallel/data_parallel_offline.py \
                    --model="ibm-research/PowerMoE-3b" \
                    -dp=2 \
                    -tp=2 \
                    --dp-num-nodes=2 \
                    --dp-node-rank=0 \
                    --dp-master-addr=10.99.48.128 \
                    --dp-master-port=13345
    Node 1:
            python examples/features/data_parallel/data_parallel_offline.py \
                    --model="ibm-research/PowerMoE-3b" \
                    -dp=2 \
                    -tp=2 \
                    --dp-num-nodes=2 \
                    --dp-node-rank=1 \
                    --dp-master-addr=10.99.48.128 \
                    --dp-master-port=13345
"""
```
**EN:** Usage: Single node: python examples/features/data_parallel/data_parallel_offline.py --model="ibm-research/PowerMoE-3b" -dp=2 -tp=2 Multi-node: Node 0 (assume the node has ip of 10.99.48.128): python...
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import os
from time import sleep

from vllm import LLM, EngineArgs, SamplingParams
from vllm.platforms import current_platform
from vllm.utils.argparse_utils import FlexibleArgumentParser
from vllm.utils.network_utils import get_open_port
```
**EN:** This block loads helper libraries such as os and time and pulls in vLLM APIs like vllm, vllm.platforms, vllm.utils.argparse_utils, and vllm.utils.network_utils.
**CN:** 这一部分加载 os 和 time 等辅助库，并引入 vllm、vllm.platforms、vllm.utils.argparse_utils，以及 vllm.utils.network_utils 等 vLLM API。

### Function: create_parser
```python
def create_parser():
    parser = FlexibleArgumentParser(description="Data Parallel Inference")

    # Add all engine args
    EngineArgs.add_cli_args(parser)
    parser.set_defaults(
        model="ibm-research/PowerMoE-3b",
        enable_expert_parallel=True,
    )

    # Add DP-specific args (separate from engine args to avoid conflicts)
    parser.add_argument(
        "--dp-num-nodes",
        type=int,
        default=1,
        help="Total number of nodes for data parallel.",
    )
    parser.add_argument(
        "--dp-node-rank",
        type=int,
        default=0,
        help="Rank of the current node for data parallel.",
    )
    parser.add_argument(
        "--dp-master-addr",
        type=str,
        default="",
        help="Master node IP address for DP coordination.",
    )
    parser.add_argument(
        "--dp-master-port",
        type=int,
        default=0,
        help="Master node port for DP coordination.",
    )
    parser.add_argument(
        "--timeout",
        type=int,
        default=300,
        help="Number of seconds before unresponsive process is killed.",
    )

    return parser
```
**EN:** This function builds and validates command-line arguments. Key operations include parser.add_argument, FlexibleArgumentParser, EngineArgs.add_cli_args, and parser.set_defaults. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 parser.add_argument、FlexibleArgumentParser、EngineArgs.add_cli_args，以及 parser.set_defaults。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(
    dp_size,
    local_dp_rank,
    global_dp_rank,
    dp_master_ip,
    dp_master_port,
    engine_args,
):
    os.environ["VLLM_DP_RANK"] = str(global_dp_rank)
    os.environ["VLLM_DP_RANK_LOCAL"] = str(local_dp_rank)
    os.environ["VLLM_DP_SIZE"] = str(dp_size)
    os.environ["VLLM_DP_MASTER_IP"] = dp_master_ip
    os.environ["VLLM_DP_MASTER_PORT"] = str(dp_master_port)

    # CUDA_VISIBLE_DEVICES for each DP rank is set automatically inside the
    # engine processes.

    # Sample prompts.
    prompts = [
        "Hello, my name is",
    # ... key logic omitted for brevity ...
    # Print the outputs.
    for i, output in enumerate(outputs):
        if i >= 5:
            # print only 5 outputs
            break
        prompt = output.prompt
        generated_text = output.outputs[0].text
        print(
            f"DP rank {global_dp_rank}, Prompt: {prompt!r}, "
            f"Generated text: {generated_text!r}"
        )

    # Give engines time to pause their processing loops before exiting.
    sleep(1)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as dp_size, local_dp_rank, global_dp_rank, dp_master_ip, dp_master_port, and engine_args. Key operations include str, len, print, start, and SamplingParams. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 dp_size、local_dp_rank、global_dp_rank、dp_master_ip、dp_master_port，以及 engine_args 等参数。关键操作包括 str、len、print、start，以及 SamplingParams。其返回值会继续传给示例管线的下一阶段。

### Entry point
```python
if __name__ == "__main__":
    parser = create_parser()
    args = vars(parser.parse_args())

    # Extract DP-specific args (pop to remove from engine_args)
    dp_size = args.pop("data_parallel_size")
    dp_num_nodes = args.pop("dp_num_nodes")
    dp_node_rank = args.pop("dp_node_rank")
    dp_master_addr = args.pop("dp_master_addr")
    dp_master_port = args.pop("dp_master_port")
    timeout = args.pop("timeout")

    # ... key logic omitted for brevity ...
        if proc.exitcode is None:
            print(f"Killing process {proc.pid} that didn't stop within 5 minutes.")
            proc.kill()
            exit_code = 1
        elif proc.exitcode:
            exit_code = proc.exitcode

    exit(exit_code)
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to args.pop, create_parser, vars, and current_platform.is_rocm.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 args.pop、create_parser、vars，以及 current_platform.is_rocm。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.platforms`, `vllm.utils.argparse_utils`, `vllm.utils.network_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `os`, `time` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `create_parser`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `parser.add_argument`, `FlexibleArgumentParser`, `EngineArgs.add_cli_args`, `parser.set_defaults`, `str`, `len`, `print`, `start` reveal the main execution path / 这些调用体现了主要执行链路。
