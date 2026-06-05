# disaggregated_prefill.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/disaggregated/disaggregated_prefill.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This file demonstrates the example usage of disaggregated prefilling We will launch 2 vllm instances (GPU 0 for prefill and GPU 1 for decode), and then transfer the KV cache between them / 演示解耦服务相关的基础组件。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This file demonstrates the example usage of disaggregated prefilling
We will launch 2 vllm instances (GPU 0 for prefill and GPU 1 for decode),
and then transfer the KV cache between them.
"""
```
**EN:** This file demonstrates the example usage of disaggregated prefilling We will launch 2 vllm instances (GPU 0 for prefill and GPU 1 for decode), and then transfer the KV cache between them.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import os
import time
from multiprocessing import Event, Process

from vllm import LLM, SamplingParams
from vllm.config import KVTransferConfig
```
**EN:** This block loads helper libraries such as os, time, and multiprocessing and pulls in vLLM APIs like vllm and vllm.config.
**CN:** 这一部分加载 os、time，以及 multiprocessing 等辅助库，并引入 vllm 和 vllm.config 等 vLLM API。

### Function: run_prefill
```python
def run_prefill(prefill_done):
    # We use GPU 0 for prefill node.
    os.environ["CUDA_VISIBLE_DEVICES"] = "0"

    # The prefill node receives two requests, while the decode node receives
    # three requests. So the decode node will only receive the KV Cache for
    # requests 1 and 3. The decode node will use the KV Cache of requests 1
    # and 3 and do prefilling on request 2.
    prompts = [
        "Hello, my name is",
        "Hi, your name is",
        # The decode node will actually "prefill" this request.
        "Tell me a very long story",
    ]
    sampling_params = SamplingParams(temperature=0, top_p=0.95, max_tokens=1)

    # Using P2pNcclConnector to transmit KV caches between vLLM instances.
    # This instance is the prefill node (kv_producer, rank 0).
    # The number of parallel instances for KV cache transfer is set to 2,
    # as required for P2pNcclConnector.
    # ... key logic omitted for brevity ...
        gpu_memory_utilization=0.8,
    )

    llm.generate(prompts, sampling_params)
    print("Prefill node is finished.")
    prefill_done.set()

    # To keep the prefill node running in case the decode node is not done;
    # otherwise, the script might exit prematurely, causing incomplete decoding.
    try:
        while True:
            time.sleep(1)
    except KeyboardInterrupt:
        print("Script stopped by user.")
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as prefill_done. Key operations include print, SamplingParams, KVTransferConfig, LLM, and llm.generate.
**CN:** 该函数编排端到端工作流。它会处理 prefill_done 等参数。关键操作包括 print、SamplingParams、KVTransferConfig、LLM，以及 llm.generate。

### Function: run_decode
```python
def run_decode(prefill_done):
    # We use GPU 1 for decode node.
    os.environ["CUDA_VISIBLE_DEVICES"] = "1"

    prompts = [
        "Hello, my name is",
        "Hi, your name is",
        "Tell me a very long story",
    ]
    sampling_params = SamplingParams(temperature=0, top_p=0.95)

    # Using P2pNcclConnector to transmit KV caches between vLLM instances.
    # This instance is the decode node (kv_consumer, rank 1).
    # The number of parallel instances for KV cache transfer is set to 2,
    # as required for P2pNcclConnector.
    ktc = KVTransferConfig(
        kv_connector="P2pNcclConnector",
        kv_role="kv_consumer",
        kv_rank=1,
        kv_parallel_size=2,
    )

    # Set GPU memory utilization to 0.8 for an A6000 GPU with 40GB
    # memory. You may need to adjust the value to fit your GPU.
    llm = LLM(
        model="meta-llama/Meta-Llama-3.1-8B-Instruct",
        kv_transfer_config=ktc,
        max_model_len=2000,
        gpu_memory_utilization=0.8,
    )

    # Wait for the producer to start the pipe
    print("Waiting for prefill node to finish...")
    prefill_done.wait()

    # At this point when the prefill_done is set, the kv-cache should have been
    # transferred to this decode node, so we can start decoding.
    outputs = llm.generate(prompts, sampling_params)
    for output in outputs:
        prompt = output.prompt
        generated_text = output.outputs[0].text
        print(f"Prompt: {prompt!r}, Generated text: {generated_text!r}")
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as prefill_done. Key operations include print, SamplingParams, KVTransferConfig, LLM, and prefill_done.wait.
**CN:** 该函数编排端到端工作流。它会处理 prefill_done 等参数。关键操作包括 print、SamplingParams、KVTransferConfig、LLM，以及 prefill_done.wait。

### Function: main
```python
def main():
    prefill_done = Event()
    prefill_process = Process(target=run_prefill, args=(prefill_done,))
    decode_process = Process(target=run_decode, args=(prefill_done,))

    # Start prefill node
    prefill_process.start()

    # Start decode node
    decode_process.start()

    # Terminate the prefill node when decode is finished
    decode_process.join()
    prefill_process.terminate()
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include Process, Event, prefill_process.start, decode_process.start, and decode_process.join.
**CN:** 该函数编排端到端工作流。关键操作包括 Process、Event、prefill_process.start、decode_process.start，以及 decode_process.join。

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
- **External libraries / 外部库**: `os`, `time`, `multiprocessing` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `run_prefill`, `run_decode`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `SamplingParams`, `KVTransferConfig`, `LLM`, `llm.generate`, `prefill_done.set`, `time.sleep`, `prefill_done.wait` reveal the main execution path / 这些调用体现了主要执行链路。
