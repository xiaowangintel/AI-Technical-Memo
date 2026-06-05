# context_extension_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/context_extension/context_extension_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This script demonstrates how to extend the context length of a Qwen model using the YARN method (rope_parameters) and run a simple chat example / 演示 vLLM 示例目录中与 context extension offline 相关的工作流。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This script demonstrates how to extend the context length
of a Qwen model using the YARN method (rope_parameters)
and run a simple chat example.

Usage:
    python examples/features/context_extension/context_extension_offline.py
"""
```
**EN:** This script demonstrates how to extend the context length of a Qwen model using the YARN method (rope_parameters) and run a simple chat example.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from vllm import LLM, RequestOutput, SamplingParams
```
**EN:** This block pulls in vLLM APIs like vllm.
**CN:** 这一部分引入 vllm 等 vLLM API。

### Function: create_llm
```python
def create_llm():
    rope_theta = 1000000
    original_max_position_embeddings = 32768
    factor = 4.0

    # Use yarn to extend context
    hf_overrides = {
        "rope_parameters": {
            "rope_theta": rope_theta,
            "rope_type": "yarn",
            "factor": factor,
            "original_max_position_embeddings": original_max_position_embeddings,
        },
        "max_model_len": int(original_max_position_embeddings * factor),
    }

    llm = LLM(model="Qwen/Qwen3-0.6B", hf_overrides=hf_overrides)
    return llm
```
**EN:** This function constructs a core runtime component. Key operations include LLM and int. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。关键操作包括 LLM 和 int。其返回值会继续传给示例管线的下一阶段。

### Function: run_llm_chat
```python
def run_llm_chat(llm):
    sampling_params = SamplingParams(
        temperature=0.8,
        top_p=0.95,
        max_tokens=128,
    )

    conversation = [
        {"role": "system", "content": "You are a helpful assistant"},
        {"role": "user", "content": "Hello"},
        {"role": "assistant", "content": "Hello! How can I assist you today?"},
    ]
    outputs = llm.chat(conversation, sampling_params, use_tqdm=False)
    return outputs, [
        conversation,
    ]
```
**EN:** This function orchestrates the end-to-end workflow. It works with parameters such as llm. Key operations include SamplingParams and llm.chat. The return value feeds the next stage of the example pipeline.
**CN:** 该函数编排端到端工作流。它会处理 llm 等参数。关键操作包括 SamplingParams 和 llm.chat。其返回值会继续传给示例管线的下一阶段。

### Function: print_outputs
```python
def print_outputs(outputs: list[RequestOutput], conversations: list):
    print("\nGenerated Outputs:\n" + "-" * 80)
    for i, output in enumerate(outputs):
        prompt = conversations[i]
        generated_text = output.outputs[0].text
        print(f"Prompt: {prompt!r}\n")
        print(f"Generated text: {generated_text!r}")
        print("-" * 80)
```
**EN:** This function encapsulates a reusable step in the example. It works with parameters such as outputs and conversations. Key operations include print and enumerate.
**CN:** 该函数封装示例中的可复用步骤。它会处理 outputs 和 conversations 等参数。关键操作包括 print 和 enumerate。

### Function: main
```python
def main():
    llm = create_llm()
    outputs, conversations = run_llm_chat(llm)
    print_outputs(outputs, conversations)
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include create_llm, run_llm_chat, and print_outputs.
**CN:** 该函数编排端到端工作流。关键操作包括 create_llm、run_llm_chat，以及 print_outputs。

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
- **Internal structure / 内部结构**: top-level blocks such as `create_llm`, `run_llm_chat`, `print_outputs`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `LLM`, `int`, `SamplingParams`, `llm.chat`, `print`, `enumerate`, `create_llm`, `run_llm_chat` reveal the main execution path / 这些调用体现了主要执行链路。
