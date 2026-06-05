# launch_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/lora/launch_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on lora launch server. It primarily builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and prepares tensors and invokes GPU kernels. / 该 Python 模块聚焦于 lora launch server 相关流程。它主要用于构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、准备张量并调用 GPU 内核。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Imports and setup / 导入与初始化
```python
import argparse
import os

NUM_LORAS = 4
LORA_PATH = {
    "base": "meta-llama/Llama-2-7b-hf",
    "lora": "winddude/wizardLM-LlaMA-LoRA-7B",
}
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file. In practice it builds command-line arguments and runtime configuration.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。 从实现上看，它会构建命令行参数与运行时配置。

### Lines 11-34: Function `launch_server` / 函数 `launch_server`
```python
def launch_server(args):
    base_path = LORA_PATH["base"]
    lora_path = LORA_PATH["lora"]

    if args.base_only:
        cmd = f"python3 -m sglang.launch_server --model {base_path} "
    else:
        cmd = f"python3 -m sglang.launch_server --model {base_path} --lora-paths "
        for i in range(NUM_LORAS):
            lora_name = f"lora{i}"
            cmd += f"{lora_name}={lora_path} "
    cmd += f"--disable-radix "
    cmd += f"--max-loras-per-batch {args.max_loras_per_batch} "
    cmd += f"--max-running-requests {args.max_running_requests} "
    cmd += f"--lora-backend {args.lora_backend} "
    cmd += f"--tp-size {args.tp_size} "
    if args.disable_custom_all_reduce:
        cmd += "--disable-custom-all-reduce"
    if args.enable_mscclpp:
        cmd += "--enable-mscclpp"
    if args.enable_torch_symm_mem:
        cmd += "--enable-torch-symm-mem"
    print(cmd)
    os.system(cmd)
```
**EN:** `launch_server` is a function that sends requests to serving or OpenAI-compatible APIs, prepares tensors and invokes GPU kernels, and orchestrates external commands and log collection. Notable calls include `print`, `os.system`, `range`.
**CN:** `launch_server` 是一个函数，用于向服务端或 OpenAI 兼容 API 发送请求、准备张量并调用 GPU 内核、编排外部命令并收集日志。其中较关键的调用包括 `print`, `os.system`, `range`。

### Lines 37-82: CLI entry point / 命令行入口
```python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--base-only",
        action="store_true",
    )
    parser.add_argument(
        "--max-loras-per-batch",
        type=int,
        default=8,
    )
    parser.add_argument(
        "--max-running-requests",
        type=int,
        default=8,
    )
    parser.add_argument(
        "--lora-backend",
        type=str,
        default="csgmv",
    )
    parser.add_argument(
        "--tp-size",
        type=int,
        default=1,
        help="Tensor parallel size for distributed inference",
    )
    # disable_custom_all_reduce
    parser.add_argument(
        "--disable-custom-all-reduce",
        action="store_true",
        help="Disable custom all reduce when device does not support p2p communication",
    )
    parser.add_argument(
        "--enable-mscclpp",
        action="store_true",
        help="Enable using mscclpp for small messages for all-reduce kernel and fall back to NCCL.",
    )
    parser.add_argument(
        "--enable-torch-symm-mem",
        action="store_true",
        help="Enable using torch symm mem for all-reduce kernel and fall back to NCCL.",
    )
    args = parser.parse_args()

    launch_server(args)
```
**EN:** This block serves as the executable entry point, wiring parsed arguments or setup code into the main benchmark flow. In practice it builds command-line arguments and runtime configuration, sends requests to serving or OpenAI-compatible APIs, and prepares tensors and invokes GPU kernels.
**CN:** 该代码块充当可执行入口，把解析后的参数或初始化逻辑接入主基准测试流程。 从实现上看，它会构建命令行参数与运行时配置、向服务端或 OpenAI 兼容 API 发送请求、准备张量并调用 GPU 内核。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Serving APIs / 服务接口**: Talks to HTTP or OpenAI-compatible model-serving endpoints. / 与 HTTP 或 OpenAI 兼容模型服务端点交互。
- **GPU kernels / GPU 内核**: Focuses on tensor preparation and low-level accelerator kernels. / 关注张量准备与底层加速器内核。
- **Orchestration / 编排**: Launches tools, captures logs, or stitches multi-step experiments together. / 启动工具、抓取日志或拼接多步实验流程。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `os`
