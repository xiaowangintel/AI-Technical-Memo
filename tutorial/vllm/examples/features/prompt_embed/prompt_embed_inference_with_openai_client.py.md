# prompt_embed_inference_with_openai_client.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/prompt_embed/prompt_embed_inference_with_openai_client.py`
- **Repository**: vllm-project/vllm
- **Purpose**: vLLM OpenAI-Compatible Client with Prompt Embeddings / 演示 vLLM 示例目录中与 prompt embed inference with openai client 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""vLLM OpenAI-Compatible Client with Prompt Embeddings.

This script demonstrates how to:
1. Generate prompt embeddings using Hugging Face Transformers.
2. Encode them in base64 format.
3. Send them to a vLLM server for inference via both:
    - OpenAI-compatible Chat Completions API
    - OpenAI-compatible Completions API

Important distinction between the two APIs:

- Chat Completions API: `prompt_embeds` content parts should encode ONLY
# ... key logic omitted for brevity ...
      You must request access to use it:
      https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct

Dependencies:
- transformers
- torch
- openai
"""
```
**EN:** vLLM OpenAI-Compatible Client with Prompt Embeddings.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import transformers
from openai import OpenAI

from vllm.utils.serial_utils import tensor2base64
```
**EN:** This block loads helper libraries such as transformers and openai and pulls in vLLM APIs like vllm.utils.serial_utils.
**CN:** 这一部分加载 transformers 和 openai 等辅助库，并引入 vllm.utils.serial_utils 等 vLLM API。

### Function: run_chat_completion_prompt_embeds
```python
def run_chat_completion_prompt_embeds(
    client: OpenAI,
    model_name: str,
    tokenizer: transformers.PreTrainedTokenizerBase,
    embedding_layer,
    messages: list[dict],
) -> None:
    """Run a Chat Completions API request using prompt_embeds content parts.

    This example embeds ONLY the user-provided content of the final user turn, the
    vLLM server applies the chat template around it at request time.
    """
    user_content = messages[-1]["content"]
    content_token_ids = tokenizer(
        user_content, return_tensors="pt", add_special_tokens=False
    ).input_ids
    content_prompt_embeds = embedding_layer(content_token_ids).squeeze(0)
    encoded_embeds = tensor2base64(content_prompt_embeds)

    api_messages = [
        *messages[:-1],
        {
            "role": messages[-1]["role"],
            "content": [{"type": "prompt_embeds", "data": encoded_embeds}],
        },
    ]

    chat_completion = client.chat.completions.create(
        model=model_name,
        max_tokens=6,
        temperature=0.0,
        messages=api_messages,
    )

    print("-" * 30)
    print("Chat Completions API")
    print(chat_completion.choices[0].message.content)
    print("-" * 30)
```
**EN:** Run a Chat Completions API request using prompt_embeds content parts.. It works with parameters such as client, model_name, tokenizer, embedding_layer, and messages. Key operations include print, squeeze, tensor2base64, client.chat.completions.create, and tokenizer.
**CN:** 该函数编排端到端工作流。它会处理 client、model_name、tokenizer、embedding_layer，以及 messages 等参数。关键操作包括 print、squeeze、tensor2base64、client.chat.completions.create，以及 tokenizer。

### Function: run_completion_prompt_embeds
```python
def run_completion_prompt_embeds(
    client: OpenAI,
    model_name: str,
    tokenizer: transformers.PreTrainedTokenizerBase,
    embedding_layer,
    messages: list[dict],
) -> None:
    """Run a Completions API request using prompt embeddings.

    The Completions endpoint does not apply a chat template,
    so the caller must apply it and embed the full templated prompt.
    """
    templated_token_ids = tokenizer.apply_chat_template(
        messages, add_generation_prompt=True, return_tensors="pt", return_dict=True
    ).input_ids
    templated_prompt_embeds = embedding_layer(templated_token_ids).squeeze(0)
    encoded_embeds = tensor2base64(templated_prompt_embeds)

    completion = client.completions.create(
        model=model_name,
        prompt=None,
        max_tokens=6,
        temperature=0.0,
        # NOTE: The OpenAI client allows passing in extra JSON body via the
        # `extra_body` argument.
        extra_body={"prompt_embeds": encoded_embeds},
    )

    print("-" * 30)
    print("Completions API")
    print(completion.choices[0].text)
    print("-" * 30)
```
**EN:** Run a Completions API request using prompt embeddings.. It works with parameters such as client, model_name, tokenizer, embedding_layer, and messages. Key operations include print, squeeze, tensor2base64, client.completions.create, and tokenizer.apply_chat_template.
**CN:** 该函数编排端到端工作流。它会处理 client、model_name、tokenizer、embedding_layer，以及 messages 等参数。关键操作包括 print、squeeze、tensor2base64、client.completions.create，以及 tokenizer.apply_chat_template。

### Function: main
```python
def main() -> None:
    client = OpenAI(
        api_key="EMPTY",
        base_url="http://localhost:8000/v1",
    )

    model_name = "meta-llama/Llama-3.2-1B-Instruct"

    tokenizer = transformers.AutoTokenizer.from_pretrained(model_name)
    transformers_model = transformers.AutoModelForCausalLM.from_pretrained(model_name)
    embedding_layer = transformers_model.get_input_embeddings()

    messages = [
        {"role": "user", "content": "Please tell me about the capital of France."}
    ]

    # Chat Completions API: embed ONLY the user content. The server wraps
    # the embedding in the chat template when it renders the messages.
    run_chat_completion_prompt_embeds(
        client, model_name, tokenizer, embedding_layer, messages
    )

    # Completions API: embed the FULL templated prompt. The caller must
    # apply the chat template up-front.
    run_completion_prompt_embeds(
        client, model_name, tokenizer, embedding_layer, messages
    )
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include OpenAI, transformers.AutoTokenizer.from_pretrained, transformers.AutoModelForCausalLM.from_pretrained, transformers_model.get_input_embeddings, and run_chat_completion_prompt_embeds.
**CN:** 该函数编排端到端工作流。关键操作包括 OpenAI、transformers.AutoTokenizer.from_pretrained、transformers.AutoModelForCausalLM.from_pretrained、transformers_model.get_input_embeddings，以及 run_chat_completion_prompt_embeds。

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
- **OpenAI-compatible API usage / OpenAI 兼容 API 使用**: The example talks to an HTTP endpoint that follows the OpenAI schema. / 该示例通过遵循 OpenAI 协议的 HTTP 端点完成交互。
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm.utils.serial_utils` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `transformers`, `openai` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `run_chat_completion_prompt_embeds`, `run_completion_prompt_embeds`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `squeeze`, `tensor2base64`, `client.chat.completions.create`, `tokenizer`, `embedding_layer`, `client.completions.create`, `tokenizer.apply_chat_template` reveal the main execution path / 这些调用体现了主要执行链路。
