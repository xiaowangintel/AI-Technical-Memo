# multilora_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/lora/multilora_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: This example shows how to use the multi-LoRA functionality for offline inference / 演示 LoRA 适配器加载与推理。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
This example shows how to use the multi-LoRA functionality
for offline inference.

Requires HuggingFace credentials for access to Llama2.
"""
```
**EN:** This example shows how to use the multi-LoRA functionality for offline inference.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
from huggingface_hub import snapshot_download

from vllm import EngineArgs, LLMEngine, RequestOutput, SamplingParams
from vllm.lora.request import LoRARequest
```
**EN:** This block loads helper libraries such as huggingface_hub and pulls in vLLM APIs like vllm and vllm.lora.request.
**CN:** 这一部分加载 huggingface_hub 等辅助库，并引入 vllm 和 vllm.lora.request 等 vLLM API。

### Function: create_test_prompts
```python
def create_test_prompts(
    lora_path: str,
) -> list[tuple[str, SamplingParams, LoRARequest | None]]:
    """Create a list of test prompts with their sampling parameters.

    2 requests for base model, 4 requests for the LoRA. We define 2
    different LoRA adapters (using the same model for demo purposes).
    Since we also set `max_loras=1`, the expectation is that the requests
    with the second LoRA adapter will be run after all requests with the
    first adapter have finished.
    """
    return [
        (
            "A robot may not injure a human being",
            SamplingParams(temperature=0.0, logprobs=1, max_tokens=128),
            None,
        ),
        (
            "To be or not to be,",
            SamplingParams(
                temperature=0.8, top_k=5, presence_penalty=0.2, max_tokens=128
            ),
            None,
        ),
        (
            "[user] Write a SQL query to answer the question based on the table schema.\n\n context: CREATE TABLE table_name_74 (icao VARCHAR, airport VARCHAR)\n\n question: Name the ICAO for lilongwe international airport [/user] [assistant]",  # noqa: E501
            SamplingParams(temperature=0.0, logprobs=1, max_tokens=128),
            LoRARequest("sql-lora", 1, lora_path),
        ),
        (
            "[user] Write a SQL query to answer the question based on the table schema.\n\n context: CREATE TABLE table_name_74 (icao VARCHAR, airport VARCHAR)\n\n question: Name the ICAO for lilongwe international airport [/user] [assistant]",  # noqa: E501
            SamplingParams(temperature=0.0, logprobs=1, max_tokens=128),
            LoRARequest("sql-lora2", 2, lora_path),
        ),
    ]
```
**EN:** Create a list of test prompts with their sampling parameters.. It works with parameters such as lora_path. Key operations include SamplingParams and LoRARequest. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。它会处理 lora_path 等参数。关键操作包括 SamplingParams 和 LoRARequest。其返回值会继续传给示例管线的下一阶段。

### Function: process_requests
```python
def process_requests(
    engine: LLMEngine,
    test_prompts: list[tuple[str, SamplingParams, LoRARequest | None]],
):
    """Continuously process a list of prompts and handle the outputs."""
    request_id = 0

    print("-" * 50)
    while test_prompts or engine.has_unfinished_requests():
        if test_prompts:
            prompt, sampling_params, lora_request = test_prompts.pop(0)
            engine.add_request(
                str(request_id), prompt, sampling_params, lora_request=lora_request
            )
            request_id += 1

        request_outputs: list[RequestOutput] = engine.step()

        for request_output in request_outputs:
            if request_output.finished:
                print(request_output)
                print("-" * 50)
```
**EN:** Continuously process a list of prompts and handle the outputs.. It works with parameters such as engine and test_prompts. Key operations include print, engine.has_unfinished_requests, engine.step, test_prompts.pop, and engine.add_request.
**CN:** 该函数发送请求并处理返回结果。它会处理 engine 和 test_prompts 等参数。关键操作包括 print、engine.has_unfinished_requests、engine.step、test_prompts.pop，以及 engine.add_request。

### Function: initialize_engine
```python
def initialize_engine() -> LLMEngine:
    """Initialize the LLMEngine."""
    # max_loras: controls the number of LoRAs that can be used in the same
    #   batch. Larger numbers will cause higher memory usage, as each LoRA
    #   slot requires its own preallocated tensor.
    # max_lora_rank: controls the maximum supported rank of all LoRAs. Larger
    #   numbers will cause higher memory usage. If you know that all LoRAs will
    #   use the same rank, it is recommended to set this as low as possible.
    # max_cpu_loras: controls the size of the CPU LoRA cache.
    engine_args = EngineArgs(
        model="meta-llama/Llama-3.2-3B-Instruct",
        enable_lora=True,
        max_loras=1,
        max_lora_rank=8,
        max_cpu_loras=2,
        max_num_seqs=256,
    )
    return LLMEngine.from_engine_args(engine_args)
```
**EN:** Initialize the LLMEngine.. Key operations include EngineArgs and LLMEngine.from_engine_args. The return value feeds the next stage of the example pipeline.
**CN:** 该函数构建核心运行时组件。关键操作包括 EngineArgs 和 LLMEngine.from_engine_args。其返回值会继续传给示例管线的下一阶段。

### Function: main
```python
def main():
    """Main function that sets up and runs the prompt processing."""
    engine = initialize_engine()
    lora_path = snapshot_download(repo_id="jeeejeee/llama32-3b-text2sql-spider")
    test_prompts = create_test_prompts(lora_path)
    process_requests(engine, test_prompts)
```
**EN:** Main function that sets up and runs the prompt processing.. Key operations include initialize_engine, snapshot_download, create_test_prompts, and process_requests.
**CN:** 该函数编排端到端工作流。关键操作包括 initialize_engine、snapshot_download、create_test_prompts，以及 process_requests。

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

## Dependencies / 依赖关系
- **vLLM APIs / vLLM API**: `vllm`, `vllm.lora.request` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `huggingface_hub` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `create_test_prompts`, `process_requests`, `initialize_engine`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `SamplingParams`, `LoRARequest`, `print`, `engine.has_unfinished_requests`, `engine.step`, `test_prompts.pop`, `engine.add_request`, `str` reveal the main execution path / 这些调用体现了主要执行链路。
