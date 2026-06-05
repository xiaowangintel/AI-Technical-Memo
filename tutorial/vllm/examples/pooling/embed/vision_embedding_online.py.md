# vision_embedding_online.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/pooling/embed/vision_embedding_online.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Example Python client for multimodal embedding API using vLLM API server / 演示使用池化模型生成嵌入。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""Example Python client for multimodal embedding API using vLLM API server.

Refer to each `run_*` function for the command to run the server for that model.
"""
```
**EN:** Example Python client for multimodal embedding API using vLLM API server.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import argparse
import io
from typing import Literal

import pybase64 as base64
from openai import OpenAI
from openai._types import NOT_GIVEN, NotGiven
from openai.types.chat import ChatCompletionMessageParam
from openai.types.create_embedding_response import CreateEmbeddingResponse
from PIL import Image

from vllm.utils.print_utils import print_embeddings
```
**EN:** This block loads helper libraries such as argparse, io, typing, pybase64, and openai and pulls in vLLM APIs like vllm.utils.print_utils.
**CN:** 这一部分加载 argparse、io、typing、pybase64，以及 openai 等辅助库，并引入 vllm.utils.print_utils 等 vLLM API。

### Top-level setup
```python
openai_api_key = "EMPTY"
openai_api_base = "http://localhost:8000/v1"

image_url = "https://vllm-public-assets.s3.us-west-2.amazonaws.com/multimodal_asset/cat_snow.jpg"
text = "A cat standing in the snow."
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as openai_api_key, openai_api_base, image_url, and text.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 openai_api_key、openai_api_base、image_url，以及 text 等变量。

### Function: create_chat_embeddings
```python
def create_chat_embeddings(
    client: OpenAI,
    *,
    messages: list[ChatCompletionMessageParam],
    model: str,
    encoding_format: Literal["base64", "float"] | NotGiven = NOT_GIVEN,
    continue_final_message: bool = False,
    add_special_tokens: bool = False,
) -> CreateEmbeddingResponse:
    """
    Convenience function for accessing vLLM's Chat Embeddings API,
    which is an extension of OpenAI's existing Embeddings API.
    """
    return client.post(
        "/embeddings",
        cast_to=CreateEmbeddingResponse,
        body={
            "messages": messages,
            "model": model,
            "encoding_format": encoding_format,
            "continue_final_message": continue_final_message,
            "add_special_tokens": add_special_tokens,
        },
    )
```
**EN:** Convenience function for accessing vLLM's Chat Embeddings API, which is an extension of OpenAI's existing Embeddings API.. It works with parameters such as client. Key operations include client.post. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。它会处理 client 等参数。关键操作包括 client.post。其返回值会继续传给示例管线的下一阶段。

### Function: run_clip
```python
def run_clip(client: OpenAI, model: str):
    """
    Start the server using:

    vllm serve openai/clip-vit-base-patch32 \
        --runner pooling
    """

    response = create_chat_embeddings(
        client,
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
                ],
            }
        ],
        model=model,
        encoding_format="float",
    )

    print("Image embedding output:", response.data[0].embedding)

    response = create_chat_embeddings(
        client,
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "a photo of a cat"},
                ],
            }
        ],
        model=model,
        encoding_format="float",
    )

    print("Text embedding output:", response.data[0].embedding)
```
**EN:** Start the server using: vllm serve openai/clip-vit-base-patch32 --runner pooling. It works with parameters such as client and model. Key operations include create_chat_embeddings and print.
**CN:** 该函数编排端到端工作流。它会处理 client 和 model 等参数。关键操作包括 create_chat_embeddings 和 print。

### Function: run_dse_qwen2_vl
```python
def run_dse_qwen2_vl(client: OpenAI, model: str):
    """
    Start the server using:

    vllm serve MrLight/dse-qwen2-2b-mrl-v1 \
        --runner pooling \
        --trust-remote-code \
        --max-model-len 8192 \
        --chat-template examples/pooling/embed/template/dse_qwen2_vl.jinja
    """
    response = create_chat_embeddings(
        client,
        messages=[
            {
                "role": "user",
                "content": [
                    {
                        "type": "image_url",
                        "image_url": {
                            "url": image_url,
    # ... key logic omitted for brevity ...
                        "type": "image_url",
                        "image_url": {
                            "url": f"data:image/jpeg;base64,{image_placeholder}",
                        },
                    },
                    {"type": "text", "text": "Query: What is the weather like today?"},
                ],
            }
        ],
        model=model,
        encoding_format="float",
    )

    print("Text embedding output:", response.data[0].embedding)
```
**EN:** Start the server using: vllm serve MrLight/dse-qwen2-2b-mrl-v1 --runner pooling --trust-remote-code --max-model-len 8192 --chat-template examples/pooling/embed/template/dse_qwen2_vl.jinja. It works with parameters such as client and model. Key operations include create_chat_embeddings, print, io.BytesIO, Image.new, and image_placeholder.save.
**CN:** 该函数编排端到端工作流。它会处理 client 和 model 等参数。关键操作包括 create_chat_embeddings、print、io.BytesIO、Image.new，以及 image_placeholder.save。

### Function: run_qwen3_vl
```python
def run_qwen3_vl(client: OpenAI, model: str):
    """
    Start the server using:

    vllm serve Qwen/Qwen3-VL-Embedding-2B \
        --runner pooling \
        --max-model-len 8192
    """

    default_instruction = "Represent the user's input."

    print("Text embedding output:")
    response = create_chat_embeddings(
        client,
        messages=[
            {
                "role": "system",
                "content": [
                    {"type": "text", "text": default_instruction},
                ],
    # ... key logic omitted for brevity ...
            },
            {
                "role": "assistant",
                "content": [
                    {"type": "text", "text": ""},
                ],
            },
        ],
        model=model,
        encoding_format="float",
        continue_final_message=True,
        add_special_tokens=True,
    )
    print_embeddings(response.data[0].embedding)
```
**EN:** Start the server using: vllm serve Qwen/Qwen3-VL-Embedding-2B --runner pooling --max-model-len 8192. It works with parameters such as client and model. Key operations include print, create_chat_embeddings, and print_embeddings.
**CN:** 该函数编排端到端工作流。它会处理 client 和 model 等参数。关键操作包括 print、create_chat_embeddings，以及 print_embeddings。

### Function: run_siglip
```python
def run_siglip(client: OpenAI, model: str):
    """
    Start the server using:

    vllm serve google/siglip-base-patch16-224 \
        --runner pooling \
        --chat-template template_basic.jinja
    """

    response = create_chat_embeddings(
        client,
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
                ],
            }
        ],
        model=model,
        encoding_format="float",
    )

    print("Image embedding output:", response.data[0].embedding)

    response = create_chat_embeddings(
        client,
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "a photo of a cat"},
                ],
            }
        ],
        model=model,
        encoding_format="float",
    )

    print("Text embedding output:", response.data[0].embedding)
```
**EN:** Start the server using: vllm serve google/siglip-base-patch16-224 --runner pooling --chat-template template_basic.jinja. It works with parameters such as client and model. Key operations include create_chat_embeddings and print.
**CN:** 该函数编排端到端工作流。它会处理 client 和 model 等参数。关键操作包括 create_chat_embeddings 和 print。

### Function: run_vlm2vec
```python
def run_vlm2vec(client: OpenAI, model: str):
    """
    Start the server using:

    vllm serve TIGER-Lab/VLM2Vec-Full \
        --runner pooling \
        --trust-remote-code \
        --max-model-len 4096 \
        --chat-template examples/pooling/embed/template/vlm2vec_phi3v.jinja
    """

    response = create_chat_embeddings(
        client,
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "image_url", "image_url": {"url": image_url}},
                    {"type": "text", "text": "Represent the given image."},
                ],
    # ... key logic omitted for brevity ...
        messages=[
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "A cat and a dog"},
                ],
            }
        ],
        model=model,
        encoding_format="float",
    )

    print("Text embedding output:")
    print_embeddings(response.data[0].embedding)
```
**EN:** Start the server using: vllm serve TIGER-Lab/VLM2Vec-Full --runner pooling --trust-remote-code --max-model-len 4096 --chat-template examples/pooling/embed/template/vlm2vec_phi3v.jinja. It works with parameters such as client and model. Key operations include create_chat_embeddings, print, and print_embeddings.
**CN:** 该函数编排端到端工作流。它会处理 client 和 model 等参数。关键操作包括 create_chat_embeddings、print，以及 print_embeddings。

### Top-level setup
```python
model_example_map = {
    "clip": run_clip,
    "qwen3_vl": run_qwen3_vl,
    "dse_qwen2_vl": run_dse_qwen2_vl,
    "siglip": run_siglip,
    "vlm2vec": run_vlm2vec,
}
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as model_example_map.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 model_example_map 等变量。

### Function: parse_args
```python
def parse_args():
    parser = argparse.ArgumentParser(
        "Script to call a specified VLM through the API. Make sure to serve "
        "the model with `--runner pooling` before running this."
    )
    parser.add_argument(
        "--model",
        type=str,
        choices=model_example_map.keys(),
        required=True,
        help="The name of the embedding model.",
    )
    return parser.parse_args()
```
**EN:** This function builds and validates command-line arguments. Key operations include argparse.ArgumentParser, parser.add_argument, parser.parse_args, and model_example_map.keys. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建并校验命令行参数。关键操作包括 argparse.ArgumentParser、parser.add_argument、parser.parse_args，以及 model_example_map.keys。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main(args):
    client = OpenAI(
        # defaults to os.environ.get("OPENAI_API_KEY")
        api_key=openai_api_key,
        base_url=openai_api_base,
    )

    models = client.models.list()
    model_id = models.data[0].id

    model_example_map[args.model](client, model_id)
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as args. Key operations include OpenAI and client.models.list.
**CN:** 该函数编排端到端工作流。它会处理 args 等参数。关键操作包括 OpenAI 和 client.models.list。

### Entry point
```python
if __name__ == "__main__":
    args = parse_args()
    main(args)
```
**EN:** This entry-point guard prevents the demo code from running on import and triggers the example only when the file is executed directly. It typically hands control to parse_args and main.
**CN:** 这个入口保护条件可避免模块在被导入时立即执行，并只在脚本直接运行时触发示例流程。 它通常会把控制流交给 parse_args 和 main。

## Key Concepts / 关键概念
- **Example orchestration / 示例编排**: The script is structured as a runnable example rather than a reusable library module. / 该脚本更强调可直接运行的示例结构，而不是可复用库模块。
- **vLLM inference flow / vLLM 推理流程**: The file drives a vLLM engine or request path to produce model outputs. / 该文件驱动 vLLM 引擎或请求链路来生成模型输出。
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Multimodal preprocessing / 多模态预处理**: The script prepares non-text inputs before sending them to the model. / 脚本会在调用模型前准备非文本输入。
- **Disaggregated execution / 解耦执行**: Prefill, decode, or cache components are split across services or processes. / 预填充、解码或缓存组件被拆分到不同服务或进程中。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm.utils.print_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `argparse`, `io`, `typing`, `pybase64`, `openai`, `openai._types`, `openai.types.chat`, `openai.types.create_embedding_response` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `create_chat_embeddings`, `run_clip`, `run_dse_qwen2_vl`, `run_qwen3_vl`, `run_siglip`, `run_vlm2vec`, `parse_args`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `client.post`, `create_chat_embeddings`, `print`, `io.BytesIO`, `Image.new`, `image_placeholder.save`, `buffer.seek`, `decode` reveal the main execution path / 这些调用体现了主要执行链路。
