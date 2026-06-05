# cpu_offload_lmcache.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/disaggregated/lmcache/cpu_offload_lmcache.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This file demonstrates the example usage of cpu offloading with LMCache in vLLM v1 or v0 / 演示基于 LMCache 的解耦服务与缓存复用。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This file demonstrates the example usage of cpu offloading
with LMCache in vLLM v1 or v0.

Usage:

    Specify vLLM version

    -v v0 : Use LMCacheConnector
            model = mistralai/Mistral-7B-Instruct-v0.2
            (Includes enable_chunked_prefill = True)

    -v v1 : Use LMCacheConnectorV1 (default)
            model = meta-llama/Meta-Llama-3.1-8B-Instruct
            (Without enable_chunked_prefill)

Note that `lmcache` is needed to run this example.
Requirements:
https://docs.lmcache.ai/getting_started/installation.html#prerequisites
Learn more about LMCache environment setup, please refer to:
https://docs.lmcache.ai/getting_started/installation.html
"""
```
**EN:** This file demonstrates the example usage of cpu offloading with LMCache in vLLM v1 or v0.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
import contextlib
import os
import time
from dataclasses import asdict

from lmcache.integration.vllm.utils import ENGINE_NAME
from lmcache.v1.cache_engine import LMCacheEngineBuilder

from vllm import LLM, SamplingParams
from vllm.config import KVTransferConfig
from vllm.engine.arg_utils import EngineArgs
```
**EN:** This block loads helper libraries such as argparse, contextlib, os, time, and dataclasses and pulls in vLLM APIs like vllm, vllm.config, and vllm.engine.arg_utils.
**CN:** 这一部分加载 argparse、contextlib、os、time，以及 dataclasses 等辅助库，并引入 vllm、vllm.config，以及 vllm.engine.arg_utils 等 vLLM API。

### Function: setup_environment_variables
```python
def setup_environment_variables():
    # LMCache-related environment variables
    # Use experimental features in LMCache
    os.environ["LMCACHE_USE_EXPERIMENTAL"] = "True"
    # LMCache is set to use 256 tokens per chunk
    os.environ["LMCACHE_CHUNK_SIZE"] = "256"
    # Enable local CPU backend in LMCache
    os.environ["LMCACHE_LOCAL_CPU"] = "True"
    # Set local CPU memory limit to 5.0 GB
    os.environ["LMCACHE_MAX_LOCAL_CPU_SIZE"] = "5.0"
```
**EN:** This function constructs a core runtime component.
**CN:** 该函数构建核心运行时组件。

### Function: build_llm_with_lmcache
```python
def build_llm_with_lmcache(lmcache_connector: str, model: str):
    ktc = KVTransferConfig(
        kv_connector=lmcache_connector,
        kv_role="kv_both",
    )
    # Set GPU memory utilization to 0.8 for an A40 GPU with 40GB
    # memory. Reduce the value if your GPU has less memory.
    # Note: LMCache supports chunked prefill (see vLLM#14505, LMCache#392).
    llm_args = EngineArgs(
        model=model,
        kv_transfer_config=ktc,
        max_model_len=8000,
        gpu_memory_utilization=0.8,
    )

    llm = LLM(**asdict(llm_args))
    try:
        yield llm
    finally:
        # Clean up lmcache backend
        LMCacheEngineBuilder.destroy(ENGINE_NAME)
```
**EN:** This function streams incremental results while performing its core task. It works with parameters such as lmcache_connector and model. Key operations include KVTransferConfig, EngineArgs, LLM, LMCacheEngineBuilder.destroy, and asdict. It yields data progressively instead of waiting for a single final result.
**CN:** 该函数在执行核心任务时持续产出增量结果。它会处理 lmcache_connector 和 model 等参数。关键操作包括 KVTransferConfig、EngineArgs、LLM、LMCacheEngineBuilder.destroy，以及 asdict。它会逐步产出数据，而不是只等待单次最终结果。

### Function: print_output
```python
def print_output(
    llm: LLM,
    prompt: list[str],
    sampling_params: SamplingParams,
    req_str: str,
):
    # Should be able to see logs like the following:
    # `LMCache INFO: Storing KV cache for 6006 out of 6006 tokens for request 0`
    # This indicates that the KV cache has been stored in LMCache.
    start = time.time()
    outputs = llm.generate(prompt, sampling_params)
    print("-" * 50)
    for output in outputs:
        generated_text = output.outputs[0].text
        print(f"Generated text: {generated_text!r}")
    print(f"Generation took {time.time() - start:.2f} seconds, {req_str} request done.")
    print("-" * 50)
```
**EN:** This function encapsulates a reusable step in the example. It works with parameters such as llm, prompt, sampling_params, and req_str. Key operations include print, time.time, and llm.generate.
**CN:** 该函数封装示例中的可复用步骤。它会处理 llm、prompt、sampling_params，以及 req_str 等参数。关键操作包括 print、time.time，以及 llm.generate。

### Function: parse_args
```python
def parse_args():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "-v",
        "--version",
        choices=["v0", "v1"],
        default="v1",
        help="Specify vLLM version (default: v1)",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include argparse.ArgumentParser, parser.add_argument, and parser.parse_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 argparse.ArgumentParser、parser.add_argument，以及 parser.parse_args。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main():
    lmcache_connector = "LMCacheConnectorV1"
    model = "meta-llama/Meta-Llama-3.1-8B-Instruct"
    setup_environment_variables()
    with build_llm_with_lmcache(lmcache_connector, model) as llm:
        # This example script runs two requests with a shared prefix.
        # Define the shared prompt and specific prompts
        shared_prompt = "Hello, how are you?" * 1000
        first_prompt = [
            shared_prompt + "Hello, my name is",
        ]
        second_prompt = [
            shared_prompt + "Tell me a very long story",
        ]

        sampling_params = SamplingParams(temperature=0, top_p=0.95, max_tokens=10)

        # Print the first output
        print_output(llm, first_prompt, sampling_params, "first")

        time.sleep(1)

        # print the second output
        print_output(llm, second_prompt, sampling_params, "second")
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include print_output, setup_environment_variables, build_llm_with_lmcache, SamplingParams, and time.sleep.
**CN:** 该函数编排端到端工作流。关键操作包括 print_output、setup_environment_variables、build_llm_with_lmcache、SamplingParams，以及 time.sleep。

### Entry point
```python
if __name__ == "__main__":
    main()
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.config`, `vllm.engine.arg_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse`, `contextlib`, `os`, `time`, `dataclasses`, `lmcache.integration.vllm.utils`, `lmcache.v1.cache_engine` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `setup_environment_variables`, `build_llm_with_lmcache`, `print_output`, `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `KVTransferConfig`, `EngineArgs`, `LLM`, `LMCacheEngineBuilder.destroy`, `asdict`, `print`, `time.time`, `llm.generate` reveal the main execution path / 这些调用体现了主要执行链路。
