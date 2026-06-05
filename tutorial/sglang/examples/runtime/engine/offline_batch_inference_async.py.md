# offline_batch_inference_async.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/engine/offline_batch_inference_async.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This runtime example demonstrates offline batch inference async with the SGLang engine APIs. / 该运行时示例展示了如何使用 SGLang 引擎 API 完成 offline batch inference async。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module overview and usage
````python
"""
Usage:
python offline_batch_inference_async.py --model-path Qwen/Qwen2-VL-7B-Instruct

Note:
This demo shows the usage of async generation,
which is useful to implement an online-like generation with batched inference.
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 10-16: Import dependencies and runtime symbols
````python
import argparse
import asyncio
import dataclasses
import time

import sglang as sgl
from sglang.srt.server_args import ServerArgs
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 19-19: Define InferenceEngine class
````python
class InferenceEngine:
````
**EN:** This class collects the state and helper methods required by this example.
**CN:** 该类汇集了此示例所需的状态以及辅助方法。

### Lines 20-21: Method: Initialize class state
````python
    def __init__(self, **kwargs):
        self.engine = sgl.Engine(**kwargs)
````
**EN:** This method initializes lightweight dependencies and stores the object state needed by later methods.
**CN:** 该方法会初始化轻量级依赖，并保存后续方法需要使用的对象状态。

### Lines 23-25: Method: Handle one generation request
````python
    async def generate(self, prompt, sampling_params):
        result = await self.engine.async_generate(prompt, sampling_params)
        return result
````
**EN:** This method validates the inbound request, invokes the shared engine once, and returns the generated text to the client.
**CN:** 该方法会校验传入请求，调用一次共享引擎，并把生成文本返回给客户端。

### Lines 28-57: Drive the engine runtime
````python
async def run_server(server_args):
    inference = InferenceEngine(**dataclasses.asdict(server_args))

    # Sample prompts.
    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ] * 100

    # Create a sampling params object.
    sampling_params = {"temperature": 0.8, "top_p": 0.95}

    # Run the generation tasks concurrently in async mode.
    tasks = []
    for prompt in prompts:
        task = asyncio.create_task(inference.generate(prompt, sampling_params))
        tasks.append(task)

    # Get and print the result
    for task in tasks:
        await task
        while True:
            if not task.done():
                time.sleep(1)
            else:
                result = task.result()
                print(f"Generated text: {result['text']}")
                break
````
**EN:** This function constructs or uses an SGLang engine instance and wraps a concrete runtime interaction around it.
**CN:** 该函数会构建或使用一个 SGLang 引擎实例，并围绕它封装具体的运行时交互。

### Lines 60-65: Program entry point
````python
if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    ServerArgs.add_cli_args(parser)
    args = parser.parse_args()
    server_args = ServerArgs.from_cli_args(args)
    asyncio.run(run_server(server_args))
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Batch inference / 批量推理**: The workflow processes multiple requests in one pass. / 该流程会在一次执行中处理多个请求。
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。

## Dependencies / 依赖关系
- **Standard library / 标准库**: argparse, asyncio, dataclasses, time
- **Project-specific / 项目相关**: sglang, sglang.srt.server_args.ServerArgs
