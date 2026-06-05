# multi_instance_data_parallel.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/data_parallel/multi_instance_data_parallel.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstrates data-parallel execution patterns. / 演示数据并行执行模式。

## Line-by-Line Analysis / 逐行分析
### Imports
```python
import asyncio
import threading

from vllm.engine.arg_utils import AsyncEngineArgs
from vllm.engine.async_llm_engine import AsyncLLMEngine
from vllm.outputs import RequestOutput
from vllm.sampling_params import SamplingParams
from vllm.v1.metrics.loggers import AggregatedLoggingStatLogger
```
**EN:** This block loads helper libraries such as asyncio and threading and pulls in vLLM APIs like vllm.engine.arg_utils, vllm.engine.async_llm_engine, vllm.outputs, vllm.sampling_params, and vllm.v1.metrics.loggers.
**CN:** 这一部分加载 asyncio 和 threading 等辅助库，并引入 vllm.engine.arg_utils、vllm.engine.async_llm_engine、vllm.outputs、vllm.sampling_params，以及 vllm.v1.metrics.loggers 等 vLLM API。

### Top-level setup
```python
"""
To run this example, run the following commands simultaneously with
different CUDA_VISIBLE_DEVICES:
    python examples/features/data_parallel/multi_instance_data_parallel.py

    vllm serve ibm-research/PowerMoE-3b -dp 2 -dpr 1 \
        --data-parallel-address 127.0.0.1 --data-parallel-rpc-port 62300 \
        --data-parallel-size-local 1 --enforce-eager --headless

Once both instances have completed the handshake, this example will
send a request to the instance with DP rank 1.
"""
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。

### Function: _do_background_logging
```python
def _do_background_logging(engine, interval, stop_event):
    try:
        while not stop_event.is_set():
            asyncio.run(engine.do_log_stats())
            stop_event.wait(interval)
    except Exception as e:
        print(f"vLLM background logging shutdown: {e}")
        pass
```
**EN:** This function encapsulates a reusable step in the example. It works with parameters such as engine, interval, and stop_event. Key operations include stop_event.is_set, asyncio.run, stop_event.wait, print, and engine.do_log_stats.
**CN:** 该函数封装示例中的可复用步骤。它会处理 engine、interval，以及 stop_event 等参数。关键操作包括 stop_event.is_set、asyncio.run、stop_event.wait、print，以及 engine.do_log_stats。

### Async function: main
```python
async def main():
    engine_args = AsyncEngineArgs(
        model="ibm-research/PowerMoE-3b",
        data_parallel_size=2,
        tensor_parallel_size=1,
        dtype="auto",
        max_model_len=2048,
        data_parallel_address="127.0.0.1",
        data_parallel_rpc_port=62300,
        data_parallel_size_local=1,
        enforce_eager=True,
        enable_log_requests=True,
        disable_custom_all_reduce=True,
    )

    engine_client = AsyncLLMEngine.from_engine_args(
        engine_args,
        # Example: Using aggregated logger
        stat_loggers=[AggregatedLoggingStatLogger],
    )
    # ... key logic omitted for brevity ...
        prompt = "Who won the 2004 World Series?"
        final_output: RequestOutput | None = None
        async for output in engine_client.generate(
            prompt=prompt,
            sampling_params=sampling_params,
            request_id=f"abcdef-{i}",
            data_parallel_rank=1,
        ):
            final_output = output
        if final_output:
            print(final_output.outputs[0].text)

    stop_logging_event.set()
    logging_thread.join()
```
**EN:** This function asynchronously orchestrates the end-to-end workflow. Key operations include AsyncEngineArgs, AsyncLLMEngine.from_engine_args, threading.Event, threading.Thread, and logging_thread.start.
**CN:** 该函数以异步方式编排端到端工作流。关键操作包括 AsyncEngineArgs、AsyncLLMEngine.from_engine_args、threading.Event、threading.Thread，以及 logging_thread.start。

### Entry point
```python
if __name__ == "__main__":
    asyncio.run(main())
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to asyncio.run and main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 asyncio.run 和 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm.engine.arg_utils`, `vllm.engine.async_llm_engine`, `vllm.outputs`, `vllm.sampling_params`, `vllm.v1.metrics.loggers` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `asyncio`, `threading` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `_do_background_logging`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `stop_event.is_set`, `asyncio.run`, `stop_event.wait`, `print`, `engine.do_log_stats`, `AsyncEngineArgs`, `AsyncLLMEngine.from_engine_args`, `threading.Event` reveal the main execution path / 这些调用体现了主要执行链路。
