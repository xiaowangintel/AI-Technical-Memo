# lora.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/runtime/lora.py`
- **Repository**: sgl-project/sglang
- **Purpose**: OpenAI-compatible LoRA adapter usage with SGLang. Server Setup: python -m sglang.launch_server \ --model meta-llama/Llama-3.1-8B-Instruct \ --enable-lora \ --lora-paths sql=/path/to/sql python=/path/to/python / 该文件的顶部说明概述了此示例的目标与使用场景。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: Module overview and usage
````python
"""
OpenAI-compatible LoRA adapter usage with SGLang.

Server Setup:
    python -m sglang.launch_server \\
        --model meta-llama/Llama-3.1-8B-Instruct \\
        --enable-lora \\
        --lora-paths sql=/path/to/sql python=/path/to/python
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 11-11: Import dependencies and runtime symbols
````python
import openai
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 13-13: Set top-level configuration
````python
client = openai.Client(base_url="http://127.0.0.1:30000/v1", api_key="EMPTY")
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 16-56: Parse arguments and run the workflow
````python
def main():
    print("SGLang OpenAI-Compatible LoRA Examples\n")

    # Example 1: NEW - Adapter in model parameter (OpenAI-compatible)
    print("1. Chat with LoRA adapter in model parameter:")
    response = client.chat.completions.create(
        model="meta-llama/Llama-3.1-8B-Instruct:sql",  # ← adapter:name syntax
        messages=[{"role": "user", "content": "Convert to SQL: show all users"}],
        max_tokens=50,
    )
    print(f"   Response: {response.choices[0].message.content}\n")

    # Example 2: Completions API with adapter
    print("2. Completion with LoRA adapter:")
    response = client.completions.create(
        model="meta-llama/Llama-3.1-8B-Instruct:python",
        prompt="def fibonacci(n):",
        max_tokens=50,
    )
    print(f"   Response: {response.choices[0].text}\n")

    # Example 3: OLD - Backward compatible with explicit lora_path
    print("3. Backward compatible (explicit lora_path):")
    response = client.chat.completions.create(
        model="meta-llama/Llama-3.1-8B-Instruct",
        messages=[{"role": "user", "content": "Convert to SQL: show all users"}],
        extra_body={"lora_path": "sql"},
        max_tokens=50,
    )
    print(f"   Response: {response.choices[0].message.content}\n")

    # Example 4: Base model (no adapter)
    print("4. Base model without adapter:")
    response = client.chat.completions.create(
        model="meta-llama/Llama-3.1-8B-Instruct",
        messages=[{"role": "user", "content": "Hello!"}],
        max_tokens=30,
    )
    print(f"   Response: {response.choices[0].message.content}\n")

    print("All examples completed!")
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 59-67: Program entry point
````python
if __name__ == "__main__":
    try:
        main()
    except Exception as e:
        print(f"Error: {e}")
        print(
            "\nEnsure server is running:\n"
            "  python -m sglang.launch_server --model ... --enable-lora --lora-paths ..."
        )
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Structured generation / 结构化生成**: Schemas or constrained decoding keep outputs machine-readable. / 模式约束或受限解码让输出保持机器可读。
- **Adapter loading / 适配器加载**: The runtime attaches LoRA or related adapters on top of a base model. / 运行时会在基础模型之上挂载 LoRA 或相关适配器。

## Dependencies / 依赖关系
- **Third-party / 第三方**: openai
