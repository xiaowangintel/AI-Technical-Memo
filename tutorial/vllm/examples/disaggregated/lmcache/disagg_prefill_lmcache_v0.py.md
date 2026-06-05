# disagg_prefill_lmcache_v0.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/disaggregated/lmcache/disagg_prefill_lmcache_v0.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This file demonstrates the example usage of disaggregated prefilling with LMCache / 演示基于 LMCache 的解耦服务与缓存复用。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This file demonstrates the example usage of disaggregated prefilling
with LMCache.
We will launch 2 vllm instances (GPU 0 for prefill and GPU 1 for decode),
and launch an additional LMCache server.
KV cache is transferred in the following manner:
vLLM prefill node -> LMCache server -> vLLM decode node.

Note that `pip install lmcache` is needed to run this example.
Learn more about LMCache in https://github.com/LMCache/LMCache.
"""
```
**EN:** This file demonstrates the example usage of disaggregated prefilling with LMCache.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import os
import subprocess
import time
from multiprocessing import Event, Process

from lmcache.experimental.cache_engine import LMCacheEngineBuilder
from lmcache.integration.vllm.utils import ENGINE_NAME

from vllm import LLM, SamplingParams
from vllm.config import KVTransferConfig
```
**EN:** This block loads helper libraries such as os, subprocess, time, multiprocessing, and lmcache.experimental.cache_engine and pulls in vLLM APIs like vllm and vllm.config.
**CN:** 这一部分加载 os、subprocess、time、multiprocessing，以及 lmcache.experimental.cache_engine 等辅助库，并引入 vllm 和 vllm.config 等 vLLM API。

### Top-level setup
```python
port = 8100
# Use experimental features in LMCache
os.environ["LMCACHE_USE_EXPERIMENTAL"] = "True"
# LMCache is set to use 256 tokens per chunk
os.environ["LMCACHE_CHUNK_SIZE"] = "256"
# Disable local CPU backend in LMCache
os.environ["LMCACHE_LOCAL_CPU"] = "False"
# Set local CPU memory buffer limit to 5.0 GB
os.environ["LMCACHE_MAX_LOCAL_CPU_SIZE"] = "5.0"
# Set the remote URL for LMCache server
os.environ["LMCACHE_REMOTE_URL"] = f"lm://localhost:{port}"
# Set the serializer/deserializer between vllm and LMCache server
# `naive` indicates using raw bytes of the tensor without any compression
os.environ["LMCACHE_REMOTE_SERDE"] = "naive"

prompts = [
    "Hello, how are you?" * 1000,
]
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as port and prompts.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 port 和 prompts 等变量。

### Function: run_prefill
```python
def run_prefill(prefill_done, prompts):
    # We use GPU 0 for prefill node.
    os.environ["CUDA_VISIBLE_DEVICES"] = "0"

    sampling_params = SamplingParams(temperature=0, top_p=0.95, max_tokens=1)

    ktc = KVTransferConfig(
        kv_connector="LMCacheConnector",
        kv_role="kv_producer",
        kv_rank=0,
        kv_parallel_size=2,
    )
    # Set GPU memory utilization to 0.8 for an A40 GPU with 40GB
    # memory. Reduce the value if your GPU has less memory.
    llm = LLM(
        model="mistralai/Mistral-7B-Instruct-v0.2",
        kv_transfer_config=ktc,
        max_model_len=8000,
        gpu_memory_utilization=0.8,
        enforce_eager=True,
    )

    # llm.generate(prompts, sampling_params)
    outputs = llm.generate(prompts, sampling_params)
    for output in outputs:
        generated_text = output.outputs[0].text
        print(f"Generated text: {generated_text!r}")
    print("Prefill node is finished.")
    prefill_done.set()

    # Clean up lmcache backend
    LMCacheEngineBuilder.destroy(ENGINE_NAME)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as prefill_done and prompts. Key operations include print, SamplingParams, KVTransferConfig, LLM, and llm.generate.
**CN:** 该函数编排端到端工作流。它会处理 prefill_done 和 prompts 等参数。关键操作包括 print、SamplingParams、KVTransferConfig、LLM，以及 llm.generate。

### Function: run_decode
```python
def run_decode(prefill_done, prompts, timeout=1):
    # We use GPU 1 for decode node.
    os.environ["CUDA_VISIBLE_DEVICES"] = "1"

    sampling_params = SamplingParams(temperature=0, top_p=0.95, max_tokens=10)

    ktc = KVTransferConfig(
        kv_connector="LMCacheConnector",
        kv_role="kv_consumer",
        kv_rank=1,
        kv_parallel_size=2,
    )
    # Set GPU memory utilization to 0.8 for an A40 GPU with 40GB
    # of memory. Reduce the value if your GPU has less memory.
    llm = LLM(
        model="mistralai/Mistral-7B-Instruct-v0.2",
        kv_transfer_config=ktc,
        max_model_len=8000,
        gpu_memory_utilization=0.8,
        enforce_eager=True,
    )

    print("Waiting for prefill node to finish...")
    prefill_done.wait()
    time.sleep(timeout)

    outputs = llm.generate(prompts, sampling_params)
    for output in outputs:
        generated_text = output.outputs[0].text
        print(f"Generated text: {generated_text!r}")

    # Clean up lmcache backend
    LMCacheEngineBuilder.destroy(ENGINE_NAME)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as prefill_done, prompts, and timeout. Key operations include print, SamplingParams, KVTransferConfig, LLM, and prefill_done.wait.
**CN:** 该函数编排端到端工作流。它会处理 prefill_done、prompts，以及 timeout 等参数。关键操作包括 print、SamplingParams、KVTransferConfig、LLM，以及 prefill_done.wait。

### Function: run_lmcache_server
```python
def run_lmcache_server(port):
    server_proc = subprocess.Popen(
        ["python", "-m", "lmcache.experimental.server", "localhost", str(port)]
    )
    return server_proc
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as port. Key operations include subprocess.Popen and str. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 port 等参数。关键操作包括 subprocess.Popen 和 str。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main():
    prefill_done = Event()
    prefill_process = Process(target=run_prefill, args=(prefill_done, prompts))
    decode_process = Process(target=run_decode, args=(prefill_done, prompts))
    lmcache_server_process = run_lmcache_server(port)

    # Start prefill node
    prefill_process.start()

    # Start decode node
    decode_process.start()

    # Clean up the processes
    decode_process.join()
    prefill_process.terminate()
    lmcache_server_process.terminate()
    lmcache_server_process.wait()
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include Process, Event, run_lmcache_server, prefill_process.start, and decode_process.start.
**CN:** 该函数编排端到端工作流。关键操作包括 Process、Event、run_lmcache_server、prefill_process.start，以及 decode_process.start。

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
- **vLLM APIs / vLLM API**: `vllm`, `vllm.config` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `os`, `subprocess`, `time`, `multiprocessing`, `lmcache.experimental.cache_engine`, `lmcache.integration.vllm.utils` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `run_prefill`, `run_decode`, `run_lmcache_server`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `SamplingParams`, `KVTransferConfig`, `LLM`, `llm.generate`, `prefill_done.set`, `LMCacheEngineBuilder.destroy`, `prefill_done.wait` reveal the main execution path / 这些调用体现了主要执行链路。
