# prompt_embed_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/prompt_embed/prompt_embed_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstrates how to generate prompt embeddings using Hugging Face Transformers and use them as input to vLLM for both single and batch inference / 演示 vLLM 示例目录中与 prompt embed offline 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Demonstrates how to generate prompt embeddings using
Hugging Face Transformers  and use them as input to vLLM
for both single and batch inference.

Model: meta-llama/Llama-3.2-1B-Instruct
Note: This model is gated on Hugging Face Hub.
      You must request access to use it:
      https://huggingface.co/meta-llama/Llama-3.2-1B-Instruct

Requirements:
- vLLM
- transformers

Run:
    python examples/features/prompt_embed/prompt_embed_offline.py
"""
```
**EN:** Demonstrates how to generate prompt embeddings using Hugging Face Transformers and use them as input to vLLM for both single and batch inference.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import torch
from transformers import AutoModelForCausalLM, AutoTokenizer, PreTrainedTokenizer

from vllm import LLM
```
**EN:** This block loads helper libraries such as torch and transformers and pulls in vLLM APIs like vllm.
**CN:** 这一部分加载 torch 和 transformers 等辅助库，并引入 vllm 等 vLLM API。

### Function: init_tokenizer_and_llm
```python
def init_tokenizer_and_llm(model_name: str):
    tokenizer = AutoTokenizer.from_pretrained(model_name)
    transformers_model = AutoModelForCausalLM.from_pretrained(model_name)
    embedding_layer = transformers_model.get_input_embeddings()
    llm = LLM(model=model_name, enable_prompt_embeds=True)
    return tokenizer, embedding_layer, llm
```
**EN:** This function constructs a core runtime component. It works with parameters such as model_name. Key operations include AutoTokenizer.from_pretrained, AutoModelForCausalLM.from_pretrained, transformers_model.get_input_embeddings, and LLM. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。它会处理 model_name 等参数。关键操作包括 AutoTokenizer.from_pretrained、AutoModelForCausalLM.from_pretrained、transformers_model.get_input_embeddings，以及 LLM。其返回值会继续传给示例管线的下一阶段。

### Function: get_prompt_embeds
```python
def get_prompt_embeds(
    chat: list[dict[str, str]],
    tokenizer: PreTrainedTokenizer,
    embedding_layer: torch.nn.Module,
):
    token_ids = tokenizer.apply_chat_template(
        chat, add_generation_prompt=True, return_tensors="pt", return_dict=True
    ).input_ids
    prompt_embeds = embedding_layer(token_ids).squeeze(0)
    return prompt_embeds
```
**EN:** This function computes pooled representations from model outputs. It works with parameters such as chat, tokenizer, and embedding_layer. Key operations include squeeze, tokenizer.apply_chat_template, and embedding_layer. The return value feeds the next stage of the example pipeline.
**CN:** 该函数从模型输出中计算池化表示。它会处理 chat、tokenizer，以及 embedding_layer 等参数。关键操作包括 squeeze、tokenizer.apply_chat_template，以及 embedding_layer。其返回值会继续传给示例管线的下一阶段。

### Function: single_prompt_inference
```python
def single_prompt_inference(
    llm: LLM, tokenizer: PreTrainedTokenizer, embedding_layer: torch.nn.Module
):
    chat = [{"role": "user", "content": "Please tell me about the capital of France."}]
    prompt_embeds = get_prompt_embeds(chat, tokenizer, embedding_layer)

    outputs = llm.generate(
        {
            "prompt_embeds": prompt_embeds,
        }
    )

    print("\n[Single Inference Output]")
    print("-" * 30)
    for o in outputs:
        print(o.outputs[0].text)
    print("-" * 30)
```
**EN:** This function invokes model inference and collects outputs. It works with parameters such as llm, tokenizer, and embedding_layer. Key operations include print, get_prompt_embeds, and llm.generate.
**CN:** 该函数触发模型推理并收集输出。它会处理 llm、tokenizer，以及 embedding_layer 等参数。关键操作包括 print、get_prompt_embeds，以及 llm.generate。

### Function: batch_prompt_inference
```python
def batch_prompt_inference(
    llm: LLM, tokenizer: PreTrainedTokenizer, embedding_layer: torch.nn.Module
):
    chats = [
        [{"role": "user", "content": "Please tell me about the capital of France."}],
        [{"role": "user", "content": "When is the day longest during the year?"}],
        [{"role": "user", "content": "Where is bigger, the moon or the sun?"}],
    ]

    prompt_embeds_list = [
        get_prompt_embeds(chat, tokenizer, embedding_layer) for chat in chats
    ]

    outputs = llm.generate([{"prompt_embeds": embeds} for embeds in prompt_embeds_list])

    print("\n[Batch Inference Outputs]")
    print("-" * 30)
    for i, o in enumerate(outputs):
        print(f"Q{i + 1}: {chats[i][0]['content']}")
        print(f"A{i + 1}: {o.outputs[0].text}\n")
    print("-" * 30)
```
**EN:** This function invokes model inference and collects outputs. It works with parameters such as llm, tokenizer, and embedding_layer. Key operations include print, llm.generate, enumerate, and get_prompt_embeds.
**CN:** 该函数触发模型推理并收集输出。它会处理 llm、tokenizer，以及 embedding_layer 等参数。关键操作包括 print、llm.generate、enumerate，以及 get_prompt_embeds。

### Function: main
```python
def main():
    model_name = "meta-llama/Llama-3.2-1B-Instruct"
    tokenizer, embedding_layer, llm = init_tokenizer_and_llm(model_name)
    single_prompt_inference(llm, tokenizer, embedding_layer)
    batch_prompt_inference(llm, tokenizer, embedding_layer)
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include init_tokenizer_and_llm, single_prompt_inference, and batch_prompt_inference.
**CN:** 该函数编排端到端工作流。关键操作包括 init_tokenizer_and_llm、single_prompt_inference，以及 batch_prompt_inference。

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
- **Representation extraction / 表示提取**: The output is a dense or token-level representation rather than free-form text. / 输出是稠密向量或词元级表示，而不是自由文本。

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `torch`, `transformers` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `init_tokenizer_and_llm`, `get_prompt_embeds`, `single_prompt_inference`, `batch_prompt_inference`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `AutoTokenizer.from_pretrained`, `AutoModelForCausalLM.from_pretrained`, `transformers_model.get_input_embeddings`, `LLM`, `squeeze`, `tokenizer.apply_chat_template`, `embedding_layer`, `print` reveal the main execution path / 这些调用体现了主要执行链路。
