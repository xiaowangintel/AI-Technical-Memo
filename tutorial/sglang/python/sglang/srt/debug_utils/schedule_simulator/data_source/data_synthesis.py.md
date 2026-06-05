# data_synthesis.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/srt/debug_utils/schedule_simulator/data_source/data_synthesis.py`
- **Repository**: `sgl-project/sglang`
- **Purpose**: This module belongs to SGLang's SRT debug utilities and focuses on simulation input loading. It mainly loads or normalizes simulator inputs before the core simulation runs. / 该模块属于 SGLang 的 SRT 调试工具，聚焦于模拟输入加载。它主要用于在核心模拟运行前加载或规范化模拟输入。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Import dependencies and shared types / 导入依赖与共享类型
```python
import random
from typing import List, Optional

from sglang.srt.debug_utils.schedule_simulator.request import SimRequest
```
**EN:** This import block pulls in the standard library, third-party packages, and neighboring SGLang helpers that the rest of the file builds on.
**CN:** 这一段导入了标准库、第三方包以及相邻的 SGLang 辅助模块，后续实现都会建立在这些依赖之上。

### Lines 7-33: Implement function `generate_random_requests` / 实现函数 `generate_random_requests`
```python
def generate_random_requests(
    num_requests: int,
    input_len: int,
    output_len: int,
    range_ratio: float = 1.0,
    seed: Optional[int] = None,
) -> List[SimRequest]:
    if seed is not None:
        random.seed(seed)

    requests = []
    for i in range(num_requests):
        isl = _random_len(input_len, range_ratio)
        osl = _random_len(output_len, range_ratio)
        requests.append(
            SimRequest(
                request_id=f"syn{i}",
                input_len=isl,
                output_len=osl,
            )
        )

    print(
        f"Generated {len(requests)} random requests "
        f"(input_len={input_len}, output_len={output_len}, range_ratio={range_ratio})"
    )
    return requests
```
**EN:** Function `generate_random_requests` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `generate_random_requests` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 36-74: Implement function `generate_gsp_requests` / 实现函数 `generate_gsp_requests`
```python
def generate_gsp_requests(
    num_groups: int,
    prompts_per_group: int,
    system_prompt_len: int,
    question_len: int,
    output_len: int,
    range_ratio: float = 1.0,
    seed: Optional[int] = None,
) -> List[SimRequest]:
    if seed is not None:
        random.seed(seed)

    requests = []
    idx = 0
    for group_idx in range(num_groups):
        group_id = f"g{group_idx}"
        prefix_len = _random_len(system_prompt_len, range_ratio)
        for _ in range(prompts_per_group):
            q_len = _random_len(question_len, range_ratio)
            osl = _random_len(output_len, range_ratio)
            requests.append(
                SimRequest(
                    request_id=f"gsp{idx}",
                    input_len=prefix_len + q_len,
                    output_len=osl,
                    group_id=group_id,
                    prefix_len=prefix_len,
                )
            )
            idx += 1

    random.shuffle(requests)
    print(
        f"Generated {len(requests)} GSP requests "
        f"({num_groups} groups x {prompts_per_group} prompts, "
        f"system_prompt_len={system_prompt_len}, question_len={question_len}, "
        f"output_len={output_len})"
    )
    return requests
```
**EN:** Function `generate_gsp_requests` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `generate_gsp_requests` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

### Lines 77-79: Implement helper `_random_len` / 实现辅助函数 `_random_len`
```python
def _random_len(full_len: int, range_ratio: float) -> int:
    min_len = max(int(full_len * range_ratio), 1)
    return random.randint(min_len, full_len)
```
**EN:** Function `_random_len` performs one concrete unit of work in this module. It gathers inputs, applies the local algorithm, and produces outputs consumed by later debug steps.
**CN:** 函数 `_random_len` 在本模块中承担一个明确的工作单元：收集输入、执行局部算法，并产出供后续调试步骤消费的结果。

## Key Concepts / 关键概念
- **Core symbols / 核心符号**: `generate_random_requests`, `generate_gsp_requests`, `_random_len`
- **Module role / 模块角色**: Simulation input loading / 模拟输入加载
- **Implementation focus / 实现重点**: Loads or normalizes simulator inputs before the core simulation runs / 在核心模拟运行前加载或规范化模拟输入

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `typing`
- **Third-party / 第三方**: None / 无
- **Internal / 内部**: `sglang.srt.debug_utils.schedule_simulator.request`
