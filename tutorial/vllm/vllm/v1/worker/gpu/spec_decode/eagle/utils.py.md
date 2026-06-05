# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/worker/gpu/spec_decode/eagle/utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `_should_share`, `load_eagle_model` for the V1 `worker/gpu/spec_decode/eagle` subsystem. / 为 V1 的 `worker/gpu/spec_decode/eagle` 子系统实现 `_should_share`, `load_eagle_model`。

## Line-by-Line Analysis / 逐行分析
### Imports and typing / 导入与类型定义
```python
import torch
import torch.nn as nn

from vllm.config import VllmConfig
from vllm.distributed.parallel_state import get_pp_group
from vllm.model_executor.model_loader import get_model
```
**EN:** Sets up the module namespace with standard-library helpers, external packages such as `torch`, and internal vLLM modules such as `vllm.config`, `vllm.distributed.parallel_state`, `vllm.model_executor.model_loader`, `vllm.compilation.backends`. TYPE_CHECKING-only imports keep runtime dependencies light while preserving static typing.
**CN:** 该代码块为模块准备命名空间，引入标准库工具、`torch` 等外部依赖，以及 `vllm.config`, `vllm.distributed.parallel_state`, `vllm.model_executor.model_loader`, `vllm.compilation.backends` 等 vLLM 内部模块。其中仅用于 TYPE_CHECKING 的导入可在保留静态类型信息的同时避免运行时依赖膨胀。

### `_should_share` function / `_should_share` 函数
```python
def _should_share(eagle: nn.Module, flag: str, draft, target) -> bool:
    """Share when the draft has no own copy, or its copy matches the target."""

    if not getattr(eagle, flag, False) or draft is None:
        return True
    if target is None:
        return False
    # torch.equal on GPU allocates a bool mask the size of the input.
    # Use the faster GPU path when there is plenty of headroom;
    # otherwise compare on CPU.
    w = draft.weight
    if w.is_cuda and torch.cuda.mem_get_info(w.device)[0] < w.numel() * 2:
        return torch.equal(w.cpu(), target.weight.cpu())
    return torch.equal(w, target.weight)
```
**EN:** This function implements `_should_share` within the module. The docstring frames it as: Share when the draft has no own copy, or its copy matches the target. Key calls include `equal`, `getattr`, `cpu`, `mem_get_info`, `numel`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_should_share`，其作用域位于the module。 关键调用包括 `equal`, `getattr`, `cpu`, `mem_get_info`, `numel`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `load_eagle_model` function / `load_eagle_model` 函数
```python
def load_eagle_model(target_model: nn.Module, vllm_config: VllmConfig) -> nn.Module:
    from vllm.compilation.backends import set_model_tag

    speculative_config = vllm_config.speculative_config
    assert speculative_config is not None
    draft_model_config = speculative_config.draft_model_config
    with set_model_tag("eagle_head"):
        eagle_model = get_model(
            vllm_config=vllm_config, model_config=draft_model_config
        )

    target_language_model = (
        target_model.get_language_model()
        if hasattr(target_model, "get_language_model")
        else target_model
    )
    target_inner = target_language_model.model
    draft_inner = eagle_model.model

    # Skip embedding sharing under PP — each rank owns its own embedding.
    if get_pp_group().world_size == 1:
        target_embed = getattr(target_inner, "embed_tokens", None) or getattr(
            target_inner, "embedding", None
        )
        draft_embed = getattr(draft_inner, "embed_tokens", None)
        if target_embed is not None and _should_share(
            eagle_model, "has_own_embed_tokens", draft_embed, target_embed
        ):
            if draft_embed is not None:
                del draft_inner.embed_tokens
            draft_inner.embed_tokens = target_embed

    target_lm_head = getattr(target_model, "lm_head", None)
    draft_lm_head = getattr(eagle_model, "lm_head", None)
    if target_lm_head is not None and _should_share(
        eagle_model, "has_own_lm_head", draft_lm_head, target_lm_head
    ):
        if draft_lm_head is not None:
            del eagle_model.lm_head
        eagle_model.lm_head = target_lm_head

        # MTP layers route logits through layer.shared_head.head, not
        # eagle_model.lm_head, so the per-layer copies need fixing up too.
        layers = getattr(draft_inner, "layers", None)
        if layers is not None:
            items = layers.values() if isinstance(layers, nn.ModuleDict) else layers
            for layer in items:
                sh = getattr(layer, "shared_head", None)
                if sh is not None and hasattr(sh, "head"):
                    del sh.head
                    sh.head = target_lm_head

    # MTP also shares a topk_indices_buffer between target and draft.
    if hasattr(target_inner, "topk_indices_buffer"):
        if hasattr(draft_inner, "topk_indices_buffer"):
            del draft_inner.topk_indices_buffer
        draft_inner.topk_indices_buffer = target_inner.topk_indices_buffer

    return eagle_model
```
**EN:** This function loads external or cached state within the module. Key calls include `getattr`, `hasattr`, `set_model_tag`, `get_model`, `get_language_model`, `_should_share`. The control flow contains 11 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会加载外部或缓存状态，其作用域位于the module。 关键调用包括 `getattr`, `hasattr`, `set_model_tag`, `get_model`, `get_language_model`, `_should_share`。 控制流包含 11 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `_should_share`: top-level helper or orchestration entry point. / `_should_share`：顶层辅助函数或编排入口。
- `load_eagle_model`: top-level helper or orchestration entry point. / `load_eagle_model`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`
- Internal vLLM / 内部依赖: `vllm.config`, `vllm.distributed.parallel_state`, `vllm.model_executor.model_loader`, `vllm.compilation.backends`
