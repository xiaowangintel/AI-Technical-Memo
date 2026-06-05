# fastapi_engine_inference.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/engine/fastapi_engine_inference.py`
- **Repository**: sgl-project/sglang
- **Purpose**: FastAPI server example for text generation using SGLang Engine and demonstrating client usage. Starts the server, sends requests to it, and prints responses. / 该文件的顶部说明概述了此示例的目标与使用场景。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: Module overview and usage
````python
"""
FastAPI server example for text generation using SGLang Engine and demonstrating client usage.

Starts the server, sends requests to it, and prints responses.

Usage:
python fastapi_engine_inference.py --model-path Qwen/Qwen2.5-0.5B-Instruct --tp_size 1 --host 127.0.0.1 --port 8000 [--startup-timeout 60]
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 10-19: Import dependencies and runtime symbols
````python
import os
import subprocess
import time
from contextlib import asynccontextmanager

import requests
from fastapi import FastAPI, Request

import sglang as sgl
from sglang.utils import terminate_process
````
**EN:** Import web framework primitives together with SGLang runtime objects so the module can serve inference over HTTP.
**CN:** 该部分同时导入 Web 框架原语和 SGLang 运行时对象，以便模块通过 HTTP 提供推理服务。

### Lines 21-21: Set top-level configuration
````python
engine = None
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 24-40: Drive the engine runtime
````python
# Use FastAPI's lifespan manager to initialize/shutdown the engine
@asynccontextmanager
async def lifespan(app: FastAPI):
    """Manages SGLang engine initialization during server startup."""
    global engine
    # Initialize the SGLang engine when the server starts
    # Adjust model_path and other engine arguments as needed
    print("Loading SGLang engine...")
    engine = sgl.Engine(
        model_path=os.getenv("MODEL_PATH"), tp_size=int(os.getenv("TP_SIZE"))
    )
    print("SGLang engine loaded.")
    yield
    # Clean up engine resources when the server stops (optional, depends on engine needs)
    print("Shutting down SGLang engine...")
    # engine.shutdown() # Or other cleanup if available/necessary
    print("SGLang engine shutdown.")
````
**EN:** This function constructs or uses an SGLang engine instance and wraps a concrete runtime interaction around it.
**CN:** 该函数会构建或使用一个 SGLang 引擎实例，并围绕它封装具体的运行时交互。

### Lines 43-43: Create shared application state
````python
app = FastAPI(lifespan=lifespan)
````
**EN:** These statements allocate the web application object and any module-level state shared by request handlers.
**CN:** 这些语句创建 Web 应用对象以及请求处理函数共享的模块级状态。

### Lines 46-74: Handle one generation request
````python
@app.post("/generate")
async def generate_text(request: Request):
    """FastAPI endpoint to handle text generation requests."""
    global engine
    if not engine:
        return {"error": "Engine not initialized"}, 503

    try:
        data = await request.json()
        prompt = data.get("prompt")
        max_new_tokens = data.get("max_new_tokens", 128)
        temperature = data.get("temperature", 0.7)

        if not prompt:
            return {"error": "Prompt is required"}, 400

        # Use async_generate for non-blocking generation
        state = await engine.async_generate(
            prompt,
            sampling_params={
                "max_new_tokens": max_new_tokens,
                "temperature": temperature,
            },
            # Add other parameters like stop, top_p etc. as needed
        )

        return {"generated_text": state["text"]}
    except Exception as e:
        return {"error": str(e)}, 500
````
**EN:** This function validates the inbound request, invokes the shared engine once, and returns the generated text to the client.
**CN:** 该函数会校验传入请求，调用一次共享引擎，并把生成文本返回给客户端。

### Lines 77-126: Handle server
````python
# Helper function to start the server
def start_server(args, timeout=60):
    """Starts the Uvicorn server as a subprocess and waits for it to be ready."""
    base_url = f"http://{args.host}:{args.port}"
    command = [
        "python",
        "-m",
        "uvicorn",
        "fastapi_engine_inference:app",
        f"--host={args.host}",
        f"--port={args.port}",
    ]

    process = subprocess.Popen(command, stdout=None, stderr=None)

    start_time = time.perf_counter()
    with requests.Session() as session:
        while time.perf_counter() - start_time < timeout:
            try:
                # Check the /docs endpoint which FastAPI provides by default
                response = session.get(
                    f"{base_url}/docs", timeout=5
                )  # Add a request timeout
                if response.status_code == 200:
                    print(f"Server {base_url} is ready (responded on /docs)")
                    return process
            except requests.ConnectionError:
                # Specific exception for connection refused/DNS error etc.
                pass
            except requests.Timeout:
                # Specific exception for request timeout
                print(f"Health check to {base_url}/docs timed out, retrying...")
                pass
            except requests.RequestException as e:
                # Catch other request exceptions
                print(f"Health check request error: {e}, retrying...")
                pass
            # Use a shorter sleep interval for faster startup detection
            time.sleep(1)

    # If loop finishes, raise the timeout error
    # Attempt to terminate the failed process before raising
    if process:
        print(
            "Server failed to start within timeout, attempting to terminate process..."
        )
        terminate_process(process)  # Use the imported terminate_process
    raise TimeoutError(
        f"Server failed to start at {base_url} within the timeout period."
    )
````
**EN:** This function encapsulates the “Handle server” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle server”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 129-150: Handle requests
````python
def send_requests(server_url, prompts, max_new_tokens, temperature):
    """Sends generation requests to the running server for a list of prompts."""
    # Iterate through prompts and send requests
    for i, prompt in enumerate(prompts):
        print(f"\n[{i+1}/{len(prompts)}] Sending prompt: '{prompt}'")
        payload = {
            "prompt": prompt,
            "max_new_tokens": max_new_tokens,
            "temperature": temperature,
        }

        try:
            response = requests.post(f"{server_url}/generate", json=payload, timeout=60)

            result = response.json()

            print(f"Prompt: {prompt}\nResponse: {result['generated_text']}")

        except requests.exceptions.Timeout:
            print(f"  Error: Request timed out for prompt '{prompt}'")
        except requests.exceptions.RequestException as e:
            print(f"  Error sending request for prompt '{prompt}': {e}")
````
**EN:** This function encapsulates the “Handle requests” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle requests”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 153-195: Program entry point
````python
if __name__ == "__main__":
    """Main entry point for the script."""

    import argparse

    parser = argparse.ArgumentParser()
    parser.add_argument("--host", type=str, default="127.0.0.1")
    parser.add_argument("--port", type=int, default=8000)
    parser.add_argument("--model-path", type=str, default="Qwen/Qwen2.5-0.5B-Instruct")
    parser.add_argument("--tp_size", type=int, default=1)
    parser.add_argument(
        "--startup-timeout",
        type=int,
        default=60,
        help="Time in seconds to wait for the server to be ready (default: %(default)s)",
    )
    args = parser.parse_args()

    # Pass the model to the child uvicorn process via an env var
    os.environ["MODEL_PATH"] = args.model_path
    os.environ["TP_SIZE"] = str(args.tp_size)

    # Start the server
    process = start_server(args, timeout=args.startup_timeout)

    # Define the prompts and sampling parameters
    prompts = [
        "Hello, my name is",
        "The president of the United States is",
        "The capital of France is",
        "The future of AI is",
    ]
    max_new_tokens = 64
    temperature = 0.1

    # Define server url
    server_url = f"http://{args.host}:{args.port}"

    # Send requests to the server
    send_requests(server_url, prompts, max_new_tokens, temperature)

    # Terminate the server process
    terminate_process(process)
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。
- **HTTP serving / HTTP 服务**: The code exposes generation through web endpoints. / 代码通过 Web 端点暴露生成功能。

## Dependencies / 依赖关系
- **Standard library / 标准库**: argparse, contextlib.asynccontextmanager, os, subprocess, time
- **Third-party / 第三方**: fastapi.FastAPI, fastapi.Request, requests
- **Project-specific / 项目相关**: sglang, sglang.utils.terminate_process
