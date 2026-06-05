# automatic_prefix_caching_offline.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/features/automatic_prefix_caching/automatic_prefix_caching_offline.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Demonstration script for Automatic Prefix Caching (APC) in vLLM / 演示自动前缀缓存机制。

## Line-by-Line Analysis / 逐行分析
### Module overview
```python
"""
Demonstration script for Automatic Prefix Caching (APC) in vLLM.

Automatic Prefix Caching (APC) allows the vLLM engine to reuse cached
KV (key-value) pairs from previous prompts if a new query shares the same
prefix. This reduces redundant computation and improves inference speed.

To enable APC, set `enable_prefix_caching=True` when initializing the
vLLM engine.

This script uses a long Markdown table as the shared prompt prefix and
compares the generation time for two queries that share the same prefix
but ask different questions.

Run:
python examples/features/automatic_prefix_caching/automatic_prefix_caching_offline.py
"""
```
**EN:** Demonstration script for Automatic Prefix Caching (APC) in vLLM.
**CN:** 这段模块级字符串用于说明示例目标，并为后续代码提供上下文。

### Imports
```python
import time

from vllm import LLM, SamplingParams
```
**EN:** This block loads helper libraries such as time and pulls in vLLM APIs like vllm.
**CN:** 这一部分加载 time 等辅助库，并引入 vllm 等 vLLM API。

### Top-level setup
```python
LONG_PROMPT = (
    "You are a helpful assistant in recognizes the content of tables in markdown format. Here is a table as follows.\n# Table\n"
    """
| ID  | Name          | Age | Occupation    | Country       | Email                  | Phone Number   | Address                       |
|-----|---------------|-----|---------------|---------------|------------------------|----------------|------------------------------|
| 1   | John Doe      | 29  | Engineer      | USA           | john.doe@example.com   | 555-1234       | 123 Elm St, Springfield, IL  |
| 2   | Jane Smith    | 34  | Doctor        | Canada        | jane.smith@example.com | 555-5678       | 456 Oak St, Toronto, ON      |
| 3   | Alice Johnson | 27  | Teacher       | UK            | alice.j@example.com    | 555-8765       | 789 Pine St, London, UK      |
| 4   | Bob Brown     | 45  | Artist        | Australia     | bob.b@example.com      | 555-4321       | 321 Maple St, Sydney, NSW    |
| 5   | Carol White   | 31  | Scientist     | New Zealand   | carol.w@example.com    | 555-6789       | 654 Birch St, Wellington, NZ |
| 6   | Dave Green    | 28  | Lawyer        | Ireland       | dave.g@example.com     | 555-3456       | 987 Cedar St, Dublin, IE     |
| 7   | Emma Black    | 40  | Musician      | USA           | emma.b@example.com     | 555-1111       | 246 Ash St, New York, NY     |
    # ... key logic omitted for brevity ...
| 25  | Wendy Orange  | 27  | Artist        | USA           | wendy.o@example.com    | 555-7879       | 135 Elm St, Denver, CO       |
| 26  | Xavier Green  | 34  | Scientist     | Canada        | xavier.g@example.com   | 555-9091       | 357 Oak St, Montreal, QC     |
| 27  | Yara Red      | 41  | Teacher       | UK            | yara.r@example.com     | 555-1214       | 975 Pine St, Leeds, UK       |
| 28  | Zack Blue     | 30  | Lawyer        | Australia     | zack.b@example.com     | 555-3436       | 135 Birch St, Adelaide, SA   |
| 29  | Amy White     | 33  | Musician      | New Zealand   | amy.w@example.com      | 555-5658       | 159 Maple St, Wellington, NZ |
| 30  | Ben Black     | 38  | Chef          | Ireland       | ben.b@example.com      | 555-7870       | 246 Fir St, Waterford, IE    |
"""
)
```
**EN:** This setup block defines the module-level state used by later functions and runtime calls. It introduces values such as LONG_PROMPT.
**CN:** 该初始化代码块定义了后续函数和运行流程会复用的模块级状态。其中包含 LONG_PROMPT 等变量。

### Function: get_generation_time
```python
def get_generation_time(llm, sampling_params, prompts):
    # time the generation
    start_time = time.time()
    output = llm.generate(prompts, sampling_params=sampling_params)
    end_time = time.time()
    # print the output and generation time
    print("-" * 30)
    print(f"Output: {output[0].outputs[0].text}")
    print(f"Generation time: {end_time - start_time} seconds.")
    print("-" * 30)
```
**EN:** This function encapsulates a reusable step in the example. It works with parameters such as llm, sampling_params, and prompts. Key operations include print, time.time, and llm.generate.
**CN:** 该函数封装示例中的可复用步骤。它会处理 llm、sampling_params，以及 prompts 等参数。关键操作包括 print、time.time，以及 llm.generate。

### Function: main
```python
def main():
    # set enable_prefix_caching=True to enable APC
    llm = LLM(model="lmsys/longchat-13b-16k", enable_prefix_caching=True)

    sampling_params = SamplingParams(temperature=0, max_tokens=100)

    # Querying the age of John Doe
    get_generation_time(
        llm,
        sampling_params,
        LONG_PROMPT
        + "Question: what is the age of John Doe? Your answer: The age of John Doe is ",
    )

    # Querying the age of Zack Blue
    # This query will be faster since vllm avoids computing the KV cache of LONG_PROMPT again.
    get_generation_time(
        llm,
        sampling_params,
        LONG_PROMPT
        + "Question: what is the age of Zack Blue? Your answer: The age of Zack Blue is ",
    )
```
**EN:** This function orchestrates the end-to-end workflow. Key operations include get_generation_time, LLM, and SamplingParams.
**CN:** 该函数编排端到端工作流。关键操作包括 get_generation_time、LLM，以及 SamplingParams。

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
- **vLLM APIs / vLLM API**: `vllm` drive the model-side behavior / 这些模块负责模型侧行为。
- **External libraries / 外部库**: `time` provide transport, UI, or data utilities / 这些库提供传输、界面或数据处理能力。
- **Internal structure / 内部结构**: top-level blocks such as `get_generation_time`, `main` divide the example into reusable steps / 顶层代码块把示例拆分为可复用步骤。
- **Runtime flow / 运行流程**: calls like `print`, `time.time`, `llm.generate`, `get_generation_time`, `LLM`, `SamplingParams`, `main` reveal the main execution path / 这些调用体现了主要执行链路。
