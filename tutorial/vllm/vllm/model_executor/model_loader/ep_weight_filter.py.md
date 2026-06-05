# ep_weight_filter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/model_executor/model_loader/ep_weight_filter.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Filter out non-local expert weights during loading to avoid redundant I/O. / 该文件的核心目的为：Filter out non-local expert weights during loading to avoid redundant I/O.

## Line-by-Line Analysis / 逐行分析
### Module docstring (lines 3-9)
```python
"""Filter out non-local expert weights during loading to avoid redundant I/O.

In DP+EP deployments each rank only needs its own expert shard.  Skipping
non-local expert tensors *before* they are read from disk eliminates the
majority of storage I/O for MoE models (experts typically account for
~85-90 % of total weight bytes).
"""
```
**EN:** This opening docstring states the module's intent, expected usage, and any important assumptions before the implementation details begin.
**CN:** 开头的文档字符串先说明模块意图、预期用法以及实现展开前需要了解的重要假设。

### Imports (lines 11-11)
```python
import regex as re
```
**EN:** This import block pulls in standard-library helpers, third-party packages, and vLLM internals used later in the file. It establishes the building blocks for the remaining logic.
**CN:** 这一组导入语句引入了标准库辅助工具、第三方依赖以及后续要使用的 vLLM 内部模块，为剩余逻辑建立基础。

### Constants / assignments (lines 17-17)
```python
_EXPERT_ID_RE = re.compile(r"\.experts\.(\d+)\.")
```
**EN:** This constant/configuration block defines `_EXPERT_ID_RE`, which are later reused to control behavior, document types, or centralize shared values.
**CN:** 该常量/配置代码块定义了 `_EXPERT_ID_RE`，这些名称会在后续逻辑中复用，用于控制行为、说明类型或集中管理共享值。

### Function `parse_expert_id` (lines 20-28)
```python
def parse_expert_id(weight_name: str) -> int | None:
    """Return the expert id embedded in *weight_name*, or ``None`` if it is
    not an per-expert weight.

    Returns ``None`` for dense weights (attention, layernorm, embedding),
    shared experts, and 3D fused-expert tensors where all experts are stored
    in a single tensor without a numeric expert id in the name."""
    m = _EXPERT_ID_RE.search(weight_name)
    return int(m.group(1)) if m else None
```
**EN:** Function `parse_expert_id` parses configuration, arguments, or structured metadata. The docstring highlights: Return the expert id embedded in *weight_name*, or ``None`` if it is not an per-expert weight. Key calls such as `_EXPERT_ID_RE.search`, `int`, `m.group` show the concrete execution path.
**CN:** Function `parse_expert_id` 负责解析配置、参数或结构化元数据。 文档字符串强调：Return the expert id embedded in *weight_name*, or ``None`` if it is not an per-expert weight. 像 `_EXPERT_ID_RE.search`, `int`, `m.group` 这样的关键调用展示了该代码块的具体执行路径。

### Function `compute_local_expert_ids` (lines 31-61)
```python
def compute_local_expert_ids(
    num_experts: int,
    ep_size: int,
    ep_rank: int,
    placement: str = "linear",
) -> set[int] | None:
    """Compute the set of global expert ids owned by *ep_rank*.

    Returns ``None`` when EP is not active (``ep_size <= 1``), meaning all
    experts are local and no filtering should be performed.

    The distribution logic mirrors
    :func:`vllm.model_executor.layers.fused_moe.layer.determine_expert_map`.

    Args:
        placement: ``"linear"`` for contiguous assignment,
            ``"round_robin"`` for interleaved assignment.
    """
    if ep_size <= 1:
        return None

    if placement == "linear":
        base = num_experts // ep_size
        remainder = num_experts % ep_size
        start = ep_rank * base + min(ep_rank, remainder)
    # ... omitted for brevity ...
    else:
        raise ValueError(f"Unknown expert placement strategy: {placement}")
```
**EN:** Function `compute_local_expert_ids` provides a reusable helper around the module's main workflow. The docstring highlights: Compute the set of global expert ids owned by *ep_rank*. Key calls such as `min`, `set`, `range`, `ValueError` show the concrete execution path.
**CN:** Function `compute_local_expert_ids` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Compute the set of global expert ids owned by *ep_rank*. 像 `min`, `set`, `range`, `ValueError` 这样的关键调用展示了该代码块的具体执行路径。

### Function `should_skip_weight` (lines 64-81)
```python
def should_skip_weight(
    weight_name: str,
    local_expert_ids: set[int] | None,
) -> bool:
    """Return ``True`` if *weight_name* is an expert weight that does not
    belong to the local rank and should be skipped during loading."""
    if local_expert_ids is None:
        return False
    eid = parse_expert_id(weight_name)
    if eid is None:
        # Not an expert weight (dense / shared-expert / embedding) → keep.
        return False
    # Only skip heavy weight tensors, never scale/metadata tensors.
    # Scale tensors are tiny and some backends need them from ALL experts
    # (e.g. FlashInfer NVFP4 computes a global max of activation scales).
    if not weight_name.endswith(".weight"):
        return False
    return eid not in local_expert_ids
```
**EN:** Function `should_skip_weight` provides a reusable helper around the module's main workflow. The docstring highlights: Return ``True`` if *weight_name* is an expert weight that does not belong to the local rank and should be skipped during loading. Key calls such as `parse_expert_id`, `weight_name.endswith` show the concrete execution path.
**CN:** Function `should_skip_weight` 为模块主流程提供可复用的辅助逻辑。 文档字符串强调：Return ``True`` if *weight_name* is an expert weight that does not belong to the local rank and should be skipped during loading. 像 `parse_expert_id`, `weight_name.endswith` 这样的关键调用展示了该代码块的具体执行路径。

## Key Concepts / 关键概念
- **Tensor processing / 张量处理**
  - **EN:** The module moves arrays/tensors through typed helper functions or model-facing transformations.
  - **CN:** 该模块会通过带类型的辅助函数或面向模型的变换来处理数组/张量。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `import regex as re`
