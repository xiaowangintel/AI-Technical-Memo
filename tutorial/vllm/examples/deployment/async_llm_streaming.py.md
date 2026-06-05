# async_llm_streaming.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/deployment/async_llm_streaming.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Simple example demonstrating streaming offline inference with AsyncLLM (V1 engine) / 演示 vLLM 示例目录中与 async llm streaming 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Simple example demonstrating streaming offline inference with AsyncLLM (V1 engine).

This script shows the core functionality of vLLM's AsyncLLM engine for streaming
token-by-token output in offline inference scenarios. It demonstrates DELTA mode
streaming where you receive new tokens as they are generated.

Usage:
    python examples/deployment/async_llm_streaming.py
"""
```
**EN:** Simple example demonstrating streaming offline inference with AsyncLLM (V1 engine).
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import asyncio

from vllm import SamplingParams
from vllm.engine.arg_utils import AsyncEngineArgs
from vllm.sampling_params import RequestOutputKind
from vllm.v1.engine.async_llm import AsyncLLM
```
**EN:** This block loads helper libraries such as asyncio and pulls in vLLM APIs like vllm, vllm.engine.arg_utils, vllm.sampling_params, and vllm.v1.engine.async_llm.
**CN:** 这一部分加载 asyncio 等辅助库，并引入 vllm、vllm.engine.arg_utils、vllm.sampling_params，以及 vllm.v1.engine.async_llm 等 vLLM API。

### Async function: stream_response
```python
async def stream_response(engine: AsyncLLM, prompt: str, request_id: str) -> None:
    """
    Stream response from AsyncLLM and display tokens as they arrive.

    This function demonstrates the core streaming pattern:
    1. Create SamplingParams with DELTA output kind
    2. Call engine.generate() and iterate over the async generator
    3. Print new tokens as they arrive
    4. Handle the finished flag to know when generation is complete
    """
    print(f"\n🚀 Prompt: {prompt!r}")
    print("💬 Response: ", end="", flush=True)

    # Configure sampling parameters for streaming
    sampling_params = SamplingParams(
        max_tokens=100,
        temperature=0.8,
        top_p=0.95,
        seed=42,  # For reproducible results
        output_kind=RequestOutputKind.DELTA,  # Get only new tokens each iteration
    )

    try:
        # Stream tokens from AsyncLLM
        async for output in engine.generate(
            request_id=request_id, prompt=prompt, sampling_params=sampling_params
        ):
            # Process each completion in the output
            for completion in output.outputs:
                # In DELTA mode, we get only new tokens generated since last iteration
                new_text = completion.text
                if new_text:
                    print(new_text, end="", flush=True)

            # Check if generation is finished
            if output.finished:
                print("\n✅ Generation complete!")
                break

    except Exception as e:
        print(f"\n❌ Error during streaming: {e}")
        raise
```
**EN:** Stream response from AsyncLLM and display tokens as they arrive.. It works with parameters such as engine, prompt, and request_id. Key operations include print, SamplingParams, and engine.generate.
**CN:** 该函数以异步方式向调用方持续输出增量更新。它会处理 engine、prompt，以及 request_id 等参数。关键操作包括 print、SamplingParams，以及 engine.generate。

### Async function: main
```python
async def main():
    print("🔧 Initializing AsyncLLM...")

    # Create AsyncLLM engine with simple configuration
    engine_args = AsyncEngineArgs(
        model="meta-llama/Llama-3.2-1B-Instruct",
        enforce_eager=True,  # Faster startup for examples
    )
    engine = AsyncLLM.from_engine_args(engine_args)

    try:
        # Example prompts to demonstrate streaming
        prompts = [
            "The future of artificial intelligence is",
            "In a galaxy far, far away",
            "The key to happiness is",
        ]

        print(f"🎯 Running {len(prompts)} streaming examples...")

        # Process each prompt
        for i, prompt in enumerate(prompts, 1):
            print(f"\n{'=' * 60}")
            print(f"Example {i}/{len(prompts)}")
            print(f"{'=' * 60}")

            request_id = f"stream-example-{i}"
            await stream_response(engine, prompt, request_id)

            # Brief pause between examples
            if i < len(prompts):
                await asyncio.sleep(0.5)

        print("\n🎉 All streaming examples completed!")

    finally:
        # Always clean up the engine
        print("🔧 Shutting down engine...")
        engine.shutdown()
```
**EN:** This function asynchronously orchestrates the end-to-end workflow. Key operations include print, len, AsyncEngineArgs, AsyncLLM.from_engine_args, and enumerate.
**CN:** 该函数以异步方式编排端到端工作流。关键操作包括 print、len、AsyncEngineArgs、AsyncLLM.from_engine_args，以及 enumerate。

### Entry point
```python
if __name__ == "__main__":
    try:
        asyncio.run(main())
    except KeyboardInterrupt:
        print("\n🛑 Interrupted by user")
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to asyncio.run, main, and print.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 asyncio.run、main，以及 print。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.engine.arg_utils`, `vllm.sampling_params`, `vllm.v1.engine.async_llm` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `asyncio` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `stream_response`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `SamplingParams`, `engine.generate`, `len`, `AsyncEngineArgs`, `AsyncLLM.from_engine_args`, `enumerate`, `engine.shutdown` reveal the main execution path / 这些调用体现了主要执行链路。
