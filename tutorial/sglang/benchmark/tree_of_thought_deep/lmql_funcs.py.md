# lmql_funcs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/tree_of_thought_deep/lmql_funcs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on tree of thought deep lmql. It primarily coordinates asynchronous or parallel execution, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. / 该 Python 模块聚焦于 tree of thought deep lmql 相关流程。它主要用于协调异步或并行执行、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and setup / 导入与初始化
```python
from bench_other import (
    ASSISTANT_PREFIX,
    ASSISTANT_SUFFIX,
    USER_PREFIX,
    USER_SUFFIX,
    temp,
)
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。

### Lines 10-22: Async function `propose_plan_async` / 异步函数 `propose_plan_async`
```python
async def propose_plan_async(s, question, num_branches, call_generate):
    s += (
        USER_PREFIX
        + """Please generate a high-level plan for solving the following question. As the first step, just say what method and idea you will use to solve the question. You can reorganize the information in the question. Do not do the actual calculation. Keep your response concise and within 80 words. Question: """
        + question
        + USER_SUFFIX
    )

    s += ASSISTANT_PREFIX
    comps = await call_generate(
        s, max_tokens=256, temperature=temp, stop=None, n=num_branches
    )
    return [s + comp + ASSISTANT_SUFFIX for comp in comps]
```
**EN:** `propose_plan_async` is a async function that coordinates asynchronous or parallel execution and processes tokenized prompts or decoded outputs. It returns `[s + comp + ASSISTANT_SUFFIX for comp in comps]` to the caller. Notable calls include `call_generate`.
**CN:** `propose_plan_async` 是一个异步函数，用于协调异步或并行执行、处理分词后的提示词或解码后的输出。它会向调用方返回 `[s + comp + ASSISTANT_SUFFIX for comp in comps]`。其中较关键的调用包括 `call_generate`。

### Lines 25-35: Async function `execute_plan_async` / 异步函数 `execute_plan_async`
```python
async def execute_plan_async(s, num_branches, call_generate):
    s += (
        USER_PREFIX
        + """The plan looks good! Now, use real numbers and do the calculation. Please solve the question step-by-step according to the high-level plan. Give me the final answer. Make your response short."""
        + USER_SUFFIX
    )
    s += ASSISTANT_PREFIX
    comps = await call_generate(
        s, max_tokens=256, temperature=temp, stop=None, n=num_branches
    )
    return [s + comp + ASSISTANT_SUFFIX for comp in comps]
```
**EN:** `execute_plan_async` is a async function that coordinates asynchronous or parallel execution, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. It returns `[s + comp + ASSISTANT_SUFFIX for comp in comps]` to the caller. Notable calls include `call_generate`.
**CN:** `execute_plan_async` 是一个异步函数，用于协调异步或并行执行、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `[s + comp + ASSISTANT_SUFFIX for comp in comps]`。其中较关键的调用包括 `call_generate`。

### Lines 38-48: Async function `reflect_solution_async` / 异步函数 `reflect_solution_async`
```python
async def reflect_solution_async(s, num_branches, call_generate):
    s += (
        USER_PREFIX
        + """Okay. Now, evaluate your own solution and give it a score on a scale of 1 to 5. Please do rigorous check of the correctness."""
        + USER_SUFFIX
    )
    s += ASSISTANT_PREFIX
    comps = await call_generate(
        s, max_tokens=256, temperature=temp, stop=None, n=num_branches
    )
    return [s + comp + ASSISTANT_SUFFIX for comp in comps]
```
**EN:** `reflect_solution_async` is a async function that coordinates asynchronous or parallel execution, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. It returns `[s + comp + ASSISTANT_SUFFIX for comp in comps]` to the caller. Notable calls include `call_generate`.
**CN:** `reflect_solution_async` 是一个异步函数，用于协调异步或并行执行、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `[s + comp + ASSISTANT_SUFFIX for comp in comps]`。其中较关键的调用包括 `call_generate`。

### Lines 51-61: Async function `get_final_answer_async` / 异步函数 `get_final_answer_async`
```python
async def get_final_answer_async(s, num_branches, call_generate):
    s += (
        USER_PREFIX
        + """Based on your reflection, do you change your mind? Now, give me the final answer after careful consideration."""
        + USER_SUFFIX
    )
    s += ASSISTANT_PREFIX
    comps = await call_generate(
        s, max_tokens=256, temperature=temp, stop=None, n=num_branches
    )
    return [s + comp + ASSISTANT_SUFFIX for comp in comps]
```
**EN:** `get_final_answer_async` is a async function that coordinates asynchronous or parallel execution, processes tokenized prompts or decoded outputs, and computes evaluation scores and aggregate statistics. It returns `[s + comp + ASSISTANT_SUFFIX for comp in comps]` to the caller. Notable calls include `call_generate`.
**CN:** `get_final_answer_async` 是一个异步函数，用于协调异步或并行执行、处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `[s + comp + ASSISTANT_SUFFIX for comp in comps]`。其中较关键的调用包括 `call_generate`。

### Lines 64-82: Async function `tree_search_async` / 异步函数 `tree_search_async`
```python
async def tree_search_async(question, num_branches, call_generate):
    plan_forks = await propose_plan_async("", question, num_branches, call_generate)

    sol_states = []
    for plan in plan_forks:
        forks = await execute_plan_async(plan, num_branches, call_generate)
        sol_states.extend(forks)

    ref_states = []
    for sol in sol_states:
        forks = await reflect_solution_async(sol, num_branches, call_generate)
        ref_states.extend(forks)

    solutions = []
    for sol in ref_states:
        ans = await get_final_answer_async(sol, num_branches, call_generate)
        solutions.append(ans)

    return solutions
```
**EN:** `tree_search_async` is a async function that coordinates asynchronous or parallel execution and computes evaluation scores and aggregate statistics. It returns `solutions` to the caller. Notable calls include `propose_plan_async`, `sol_states.extend`, `ref_states.extend`.
**CN:** `tree_search_async` 是一个异步函数，用于协调异步或并行执行、计算评测分数与聚合统计结果。它会向调用方返回 `solutions`。其中较关键的调用包括 `propose_plan_async`, `sol_states.extend`, `ref_states.extend`。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。

## Dependencies / 依赖关系
- **Third-party / 第三方依赖**: `bench_other`
