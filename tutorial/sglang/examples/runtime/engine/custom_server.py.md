# custom_server.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/engine/custom_server.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This runtime example demonstrates custom server with the SGLang engine APIs. / 该运行时示例展示了如何使用 SGLang 引擎 API 完成 custom server。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Import dependencies and runtime symbols
````python
from sanic import Sanic, text
from sanic.response import json

import sglang as sgl
````
**EN:** Import web framework primitives together with SGLang runtime objects so the module can serve inference over HTTP.
**CN:** 该部分同时导入 Web 框架原语和 SGLang 运行时对象，以便模块通过 HTTP 提供推理服务。

### Lines 6-9: Create shared application state
````python
engine = None

# Create an instance of the Sanic app
app = Sanic("sanic-server")
````
**EN:** These statements allocate the web application object and any module-level state shared by request handlers.
**CN:** 这些语句创建 Web 应用对象以及请求处理函数共享的模块级状态。

### Lines 12-22: Handle one generation request
````python
# Define an asynchronous route handler
@app.route("/generate", methods=["POST"])
async def generate(request):
    prompt = request.json.get("prompt")
    if not prompt:
        return json({"error": "Prompt is required"}, status=400)

    # async_generate returns a dict
    result = await engine.async_generate(prompt)

    return text(result["text"])
````
**EN:** This function validates the inbound request, invokes the shared engine once, and returns the generated text to the client.
**CN:** 该函数会校验传入请求，调用一次共享引擎，并把生成文本返回给客户端。

### Lines 25-43: Stream a web response
````python
@app.route("/generate_stream", methods=["POST"])
async def generate_stream(request):
    prompt = request.json.get("prompt")

    if not prompt:
        return json({"error": "Prompt is required"}, status=400)

    # async_generate returns a dict
    result = await engine.async_generate(prompt, stream=True)

    # https://sanic.dev/en/guide/advanced/streaming.md#streaming
    # init the response
    response = await request.respond()

    # result is an async generator
    async for chunk in result:
        await response.send(chunk["text"])

    await response.eof()
````
**EN:** This function validates the inbound request, calls the shared engine in streaming mode, and forwards each chunk to the HTTP client.
**CN:** 该函数会校验传入请求，以流式模式调用共享引擎，并把每个输出分块转发给 HTTP 客户端。

### Lines 46-49: Drive the engine runtime
````python
def run_server():
    global engine
    engine = sgl.Engine(model_path="meta-llama/Meta-Llama-3.1-8B-Instruct")
    app.run(host="0.0.0.0", port=8000, single_process=True)
````
**EN:** This function constructs or uses an SGLang engine instance and wraps a concrete runtime interaction around it.
**CN:** 该函数会构建或使用一个 SGLang 引擎实例，并围绕它封装具体的运行时交互。

### Lines 52-53: Program entry point
````python
if __name__ == "__main__":
    run_server()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Streaming output / 流式输出**: Results can be consumed incrementally as tokens arrive. / 结果可以在 token 到达时被逐步消费。
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。
- **HTTP serving / HTTP 服务**: The code exposes generation through web endpoints. / 代码通过 Web 端点暴露生成功能。

## Dependencies / 依赖关系
- **Third-party / 第三方**: sanic.Sanic, sanic.response.json, sanic.text
- **Project-specific / 项目相关**: sglang
