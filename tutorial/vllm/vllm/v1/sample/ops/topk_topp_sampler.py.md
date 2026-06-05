# topk_topp_sampler.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/sample/ops/topk_topp_sampler.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `TopKTopPSampler`, `compiled_random_sample`, `apply_top_k_top_p` for the V1 `sample/ops` subsystem. / 为 V1 的 `sample/ops` 子系统实现 `TopKTopPSampler`, `compiled_random_sample`, `apply_top_k_top_p`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import torch
import torch.nn as nn
from packaging import version

from vllm import envs
from vllm._aiter_ops import rocm_aiter_ops
from vllm.config.model import LogprobsMode
from vllm.logger import init_logger
from vllm.platforms import CpuArchEnum, current_platform
from vllm.triton_utils import HAS_TRITON

if HAS_TRITON:
    from vllm.v1.sample.ops.topk_topp_triton import apply_top_k_top_p_triton

logger = init_logger(__name__)
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`。

### `TopKTopPSampler` class / `TopKTopPSampler` 类
```python
class TopKTopPSampler(nn.Module):
    """
    Module that performs optional top-k and top-p filtering followed by
    weighted random sampling of logits.

    Implementations may update the logits tensor in-place.
    """
```
**EN:** Introduces the `TopKTopPSampler` class on top of `nn.Module`. Core methods include `__init__`, `forward_native`, `forward_cuda`, `forward_cpu`, `forward_hip`, `aiter_sample`. Docstring signal: Module that performs optional top-k and top-p filtering followed by weighted random sampling of logits.
**CN:** 这里定义 `TopKTopPSampler` 类，其基类包括 `nn.Module`。核心方法包括 `__init__`, `forward_native`, `forward_cuda`, `forward_cpu`, `forward_hip`, `aiter_sample`。

### `TopKTopPSampler.__init__` method / `TopKTopPSampler.__init__` 方法
```python
    def __init__(self, logprobs_mode: LogprobsMode = "raw_logprobs") -> None:
        super().__init__()
        self.logprobs_mode = logprobs_mode
        # flashinfer optimization does not apply if intermediate
        # logprobs/logits after top_k/top_p need to be returned
        if (
            logprobs_mode not in ("processed_logits", "processed_logprobs")
            and current_platform.is_cuda()
        ):
            if envs.VLLM_USE_FLASHINFER_SAMPLER:
                from vllm.v1.attention.backends.flashinfer import FlashInferBackend

                capability = current_platform.get_device_capability()
                assert capability is not None
                if FlashInferBackend.supports_compute_capability(capability):
                    logger.info_once(
                        "Using FlashInfer for top-p & top-k sampling.",
                        scope="global",
                    )
                    self.forward = self.forward_cuda
                elif envs.is_set("VLLM_USE_FLASHINFER_SAMPLER"):
                    # User explicitly opted in but the GPU can't run FlashInfer.
                    capability_str = capability.as_version_str()
                    raise RuntimeError(
                        "FlashInfer does not support compute capability "
                        f"{capability_str}, unset VLLM_USE_FLASHINFER_SAMPLER=1."
                    )
                else:
                    # Default-on path; hardware can't run FlashInfer →
                    # quietly fall back to the PyTorch-native sampler
                    # instead of failing server startup.
                    logger.warning_once(
                        "FlashInfer top-p/top-k sampling not supported on "
                        "compute capability %s; falling back to PyTorch-native "
                        "sampler. Set VLLM_USE_FLASHINFER_SAMPLER=0 to silence.",
                        capability.as_version_str(),
                    )
                    self.forward = self.forward_native
            else:
                # User explicitly set VLLM_USE_FLASHINFER_SAMPLER=0.
                logger.info_once(
                    "FlashInfer top-p/top-k sampling disabled via "
                    "VLLM_USE_FLASHINFER_SAMPLER=0; using PyTorch-native sampler."
                )
                self.forward = self.forward_native

        elif current_platform.is_cpu():
            arch = current_platform.get_cpu_architecture()
            # Fall back to native implementation for POWERPC and RISCV.
            # On PowerPC argmax produces incorrect output with torch.compile.
            # PR: https://github.com/vllm-project/vllm/pull/26987
            if arch in (CpuArchEnum.RISCV, CpuArchEnum.POWERPC):
                self.forward = self.forward_native
            else:
                self.forward = self.forward_cpu
        elif current_platform.is_xpu():
            if envs.VLLM_XPU_USE_SAMPLER_KERNEL:
                self.forward = self.forward_xpu
            else:
                self.forward = self.forward_native
        elif (
            logprobs_mode not in ("processed_logits", "processed_logprobs")
            and rocm_aiter_ops.is_enabled()
        ):
            try:
                import aiter.ops.sampling  # noqa: F401

                self.aiter_ops = torch.ops.aiter
                logger.info_once(
                    "Using aiter sampler on ROCm (lazy import, sampling-only)."
                )
                self.forward = self.forward_hip
            except ImportError:
                logger.warning_once(
                    "aiter.ops.sampling is not available on ROCm. "
                    "Falling back to forward_native implementation."
                )
                self.forward = self.forward_native
        else:
            self.forward = self.forward_native
```
**EN:** This method initializes the object state within `TopKTopPSampler`. Key calls include `__init__`, `is_cuda`, `is_cpu`, `super`, `get_device_capability`, `supports_compute_capability`. It touches state such as `logprobs_mode`, `forward`, `aiter_ops`. The control flow contains 10 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`TopKTopPSampler`。 关键调用包括 `__init__`, `is_cuda`, `is_cpu`, `super`, `get_device_capability`, `supports_compute_capability`。 它会读写 `logprobs_mode`, `forward`, `aiter_ops` 等状态。 控制流包含 10 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `TopKTopPSampler.forward_cpu` method / `TopKTopPSampler.forward_cpu` 方法
```python
    def forward_cpu(
        self,
        logits: torch.Tensor,
        generators: dict[int, torch.Generator],
        k: torch.Tensor | None,
        p: torch.Tensor | None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        """
        PyTorch-native implementation of top-k and top-p sampling for CPU.

        The logits tensor may be updated in-place.
        """
        logits = apply_top_k_top_p_pytorch(logits, k, p, allow_cpu_sync=True)
        logits_to_return = None
        if self.logprobs_mode == "processed_logits":
            logits_to_return = logits
        elif self.logprobs_mode == "processed_logprobs":
            logits_to_return = logits.log_softmax(dim=-1, dtype=torch.float32)

        if len(generators) != logits.shape[0]:
            return compiled_random_sample(logits), logits_to_return

        probs = logits.softmax(dim=-1, dtype=torch.float32)
        q = torch.empty_like(probs)
        q.exponential_()
        for i, generator in generators.items():
            q[i].exponential_(generator=generator)

        return probs.div_(q).argmax(dim=-1).view(-1), logits_to_return
```
**EN:** This method drives the forward-pass computation within `TopKTopPSampler`. The docstring frames it as: PyTorch-native implementation of top-k and top-p sampling for CPU. Key calls include `apply_top_k_top_p_pytorch`, `softmax`, `empty_like`, `exponential_`, `items`, `len`. The control flow contains 3 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会驱动前向计算流程，其作用域位于`TopKTopPSampler`。 关键调用包括 `apply_top_k_top_p_pytorch`, `softmax`, `empty_like`, `exponential_`, `items`, `len`。 控制流包含 3 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `TopKTopPSampler.forward_xpu` method / `TopKTopPSampler.forward_xpu` 方法
```python
    def forward_xpu(
        self,
        logits: torch.Tensor,
        generators: dict[int, torch.Generator],
        k: torch.Tensor | None,
        p: torch.Tensor | None,
    ) -> tuple[torch.Tensor, torch.Tensor | None]:
        if generators:
            logger.warning_once(
                "xpu kernel topk_topp_sampler does not support "
                "per-request generators. Falling back to "
                "PyTorch-native implementation."
            )
            return self.forward_native(logits, generators, k, p)
        random_sampled = torch.empty(
            logits.shape[0], dtype=torch.int64, device=logits.device
        )
        logits_to_return = None
        if (
            self.logprobs_mode == "processed_logits"
            or self.logprobs_mode == "processed_logprobs"
        ):
            logits_to_return = torch.empty_like(logits)

        assert len(generators) != logits.shape[0], (
            "xpu kernel topk_topp_sampler does not support batch-wise generators."
        )
        generator = torch.xpu.default_generators[logits.device.index]

        state = generator.get_state()
        seed, offset = state.view(torch.int64)
        seeds = torch.tensor(
            [seed, offset], dtype=torch.int64, device=torch.device("cpu")
        )
        # The XPU kernel expects k as int64 (Long), but the input batch
        # stores top_k as int32. Cast here to avoid dtype mismatch.
        if k is not None:
            k = k.to(torch.int64)
        torch.ops.vllm.xpu_topk_topp_sampler(
            random_sampled, logits_to_return, logits, k, p, self.logprobs_mode, seeds
        )
        # The custom XPU sampler kernel consumes RNG values internally, so advance
        # the default generator's offset to keep future draws deterministic.
        offset += logits.numel()
        state.view(torch.int64)[1] = offset
        generator.set_state(state)
        return random_sampled, logits_to_return
```
**EN:** This method drives the forward-pass computation within `TopKTopPSampler`. Key calls include `empty`, `get_state`, `view`, `tensor`, `xpu_topk_topp_sampler`, `numel`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会驱动前向计算流程，其作用域位于`TopKTopPSampler`。 关键调用包括 `empty`, `get_state`, `view`, `tensor`, `xpu_topk_topp_sampler`, `numel`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `compiled_random_sample` function / `compiled_random_sample` 函数
```python
@torch.compile(dynamic=True)
def compiled_random_sample(logits: torch.Tensor) -> torch.Tensor:
    probs = logits.softmax(dim=-1, dtype=torch.float32)
    q = torch.empty_like(probs)
    q.exponential_()
    return probs.div(q).argmax(dim=-1).view(-1)
```
**EN:** This function implements `compiled_random_sample` within the module. Key calls include `compile`, `softmax`, `empty_like`, `exponential_`, `view`, `argmax`.
**CN:** 该函数会实现 `compiled_random_sample`，其作用域位于the module。 关键调用包括 `compile`, `softmax`, `empty_like`, `exponential_`, `view`, `argmax`。

### `apply_top_k_top_p` function / `apply_top_k_top_p` 函数
```python
def apply_top_k_top_p(
    logits: torch.Tensor, k: torch.Tensor | None, p: torch.Tensor | None
) -> torch.Tensor:
    if p is None and k is None:
        return logits

    if HAS_TRITON and logits.shape[0] >= 8:
        return apply_top_k_top_p_triton(logits, k, p)

    # Use pytorch sort implementation for small batch sizes.
    return apply_top_k_top_p_pytorch(logits, k, p)
```
**EN:** This function implements `apply_top_k_top_p` within the module. Key calls include `apply_top_k_top_p_pytorch`, `apply_top_k_top_p_triton`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `apply_top_k_top_p`，其作用域位于the module。 关键调用包括 `apply_top_k_top_p_pytorch`, `apply_top_k_top_p_triton`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `apply_top_k_top_p_pytorch` function / `apply_top_k_top_p_pytorch` 函数
```python
def apply_top_k_top_p_pytorch(
    logits: torch.Tensor,
    k: torch.Tensor | None,
    p: torch.Tensor | None,
    allow_cpu_sync: bool = False,
) -> torch.Tensor:
    """Apply top-k and top-p masks to the logits.

    If a top-p is used, this function will sort the logits tensor,
    which can be slow for large batches.

    The logits tensor may be updated in-place.
    """
    if p is None:
        if k is None:
            return logits

        if allow_cpu_sync:
            # Avoid sorting vocab for top-k only case.
            return apply_top_k_only(logits, k)

    logits_sort, logits_idx = logits.sort(dim=-1, descending=False)

    if k is not None:
        # Apply top-k.
        top_k_mask = logits_sort.size(1) - k.to(torch.long)  # shape: B
        # Get all the top_k values.
        top_k_mask = logits_sort.gather(1, top_k_mask.unsqueeze(dim=1))
        top_k_mask = logits_sort < top_k_mask
        logits_sort.masked_fill_(top_k_mask, -float("inf"))

    if p is not None:
        # Apply top-p.
        probs_sort = logits_sort.softmax(dim=-1)
        probs_sum = torch.cumsum(probs_sort, dim=-1, out=probs_sort)
        top_p_mask = probs_sum <= 1 - p.unsqueeze(dim=1)
        # at least one
        top_p_mask[:, -1] = False
        logits_sort.masked_fill_(top_p_mask, -float("inf"))

    # Re-sort the probabilities.
    return logits.scatter_(dim=-1, index=logits_idx, src=logits_sort)
```
**EN:** This function implements `apply_top_k_top_p_pytorch` within the module. The docstring frames it as: Apply top-k and top-p masks to the logits. Key calls include `sort`, `scatter_`, `gather`, `masked_fill_`, `softmax`, `cumsum`. The control flow contains 5 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `apply_top_k_top_p_pytorch`，其作用域位于the module。 关键调用包括 `sort`, `scatter_`, `gather`, `masked_fill_`, `softmax`, `cumsum`。 控制流包含 5 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `apply_top_k_only` function / `apply_top_k_only` 函数
```python
def apply_top_k_only(logits: torch.Tensor, k: torch.Tensor) -> torch.Tensor:
    """
    Apply top-k mask to the logits.

    This implementation doesn't involve sorting the entire vocab.
    Note however that it involves a GPU->CPU sync which can be detrimental for
    async scheduling performance.

    The logits tensor may be updated in-place.
    """
    no_top_k_mask = k == logits.shape[1]
    # Set non-top-k rows to 1 so that we can gather.
    k = k.masked_fill(no_top_k_mask, 1)
    max_top_k = k.max()
    # topk.values tensor has shape [batch_size, max_top_k].
    # Convert top k to 0-based index in range [0, max_top_k).
    k_index = k.sub_(1).unsqueeze(1)
    top_k_mask = logits.topk(max_top_k, dim=1).values.gather(1, k_index.long())
    # Handle non-topk rows.
    top_k_mask.masked_fill_(no_top_k_mask.unsqueeze(1), -float("inf"))
    return logits.masked_fill_(logits < top_k_mask, -float("inf"))
```
**EN:** This function implements `apply_top_k_only` within the module. The docstring frames it as: Apply top-k mask to the logits. Key calls include `masked_fill`, `max`, `unsqueeze`, `gather`, `masked_fill_`, `long`.
**CN:** 该函数会实现 `apply_top_k_only`，其作用域位于the module。 关键调用包括 `masked_fill`, `max`, `unsqueeze`, `gather`, `masked_fill_`, `long`。

### `random_sample` function / `random_sample` 函数
```python
def random_sample(
    probs: torch.Tensor,
    generators: dict[int, torch.Generator],
) -> torch.Tensor:
    """Randomly sample from the probabilities.

    We use this function instead of torch.multinomial because torch.multinomial
    causes CPU-GPU synchronization.
    """
    q = torch.empty_like(probs)
    # NOTE(woosuk): To batch-process the requests without their own seeds,
    # which is the common case, we first assume that every request does
    # not have its own seed. Then, we overwrite the values for the requests
    # that have their own seeds.
    if len(generators) != probs.shape[0]:
        q.exponential_()
    if generators:
        # TODO(woosuk): This can be slow because we handle each request
        # one by one. Optimize this.
        for i, generator in generators.items():
            q[i].exponential_(generator=generator)
    return probs.div_(q).argmax(dim=-1).view(-1)
```
**EN:** This function implements `random_sample` within the module. The docstring frames it as: Randomly sample from the probabilities. Key calls include `empty_like`, `view`, `len`, `exponential_`, `items`, `argmax`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `random_sample`，其作用域位于the module。 关键调用包括 `empty_like`, `view`, `len`, `exponential_`, `items`, `argmax`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `flashinfer_sample` function / `flashinfer_sample` 函数
```python
def flashinfer_sample(
    logits: torch.Tensor,
    k: torch.Tensor | None,
    p: torch.Tensor | None,
    generators: dict[int, torch.Generator],
) -> torch.Tensor:
    """Sample from the logits using FlashInfer.

    Statistically, this function is equivalent to the `random_sample` function.
    However, this function is faster because it avoids sorting the logits tensor
    via rejection sampling.

    NOTE: The outputs of this function do not necessarily match the outputs of
    the `random_sample` function. It only guarantees that the outputs are
    statistically equivalent.
    """
    import flashinfer

    if version.parse(flashinfer.__version__) < version.parse("0.2.3"):
        raise ImportError(
            "FlashInfer version >= 0.2.3 required for top-k and top-p sampling. "
        )

    assert not (k is None and p is None)
    if k is None:
        # Top-p only.
        probs = logits.softmax(dim=-1, dtype=torch.float32)
        next_token_ids = flashinfer.sampling.top_p_sampling_from_probs(
            probs, p, deterministic=True
        )
    elif p is None:
        # Top-k only.
        probs = logits.softmax(dim=-1, dtype=torch.float32)
        next_token_ids = flashinfer.sampling.top_k_sampling_from_probs(
            probs, k, deterministic=True
        )
    else:
        # Both top-k and top-p.
        next_token_ids = flashinfer.sampling.top_k_top_p_sampling_from_logits(
            logits, k, p, deterministic=True
        )

    return next_token_ids.view(-1)
```
**EN:** This function implements `flashinfer_sample` within the module. The docstring frames it as: Sample from the logits using FlashInfer. Key calls include `view`, `parse`, `ImportError`, `softmax`, `top_p_sampling_from_probs`, `top_k_sampling_from_probs`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `flashinfer_sample`，其作用域位于the module。 关键调用包括 `view`, `parse`, `ImportError`, `softmax`, `top_p_sampling_from_probs`, `top_k_sampling_from_probs`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_to_tensor_scalar_tuple` function / `_to_tensor_scalar_tuple` 函数
```python
def _to_tensor_scalar_tuple(x):
    if isinstance(x, torch.Tensor):
        return (x, 0)
    else:
        return (None, x)
```
**EN:** This function implements `_to_tensor_scalar_tuple` within the module. Key calls include `isinstance`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_to_tensor_scalar_tuple`，其作用域位于the module。 关键调用包括 `isinstance`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `TopKTopPSampler`: central class or interface in this module. / `TopKTopPSampler`：本模块中的核心类或接口。
- `compiled_random_sample`: top-level helper or orchestration entry point. / `compiled_random_sample`：顶层辅助函数或编排入口。
- `apply_top_k_top_p`: top-level helper or orchestration entry point. / `apply_top_k_top_p`：顶层辅助函数或编排入口。
- `apply_top_k_top_p_pytorch`: top-level helper or orchestration entry point. / `apply_top_k_top_p_pytorch`：顶层辅助函数或编排入口。
- `apply_top_k_only`: top-level helper or orchestration entry point. / `apply_top_k_only`：顶层辅助函数或编排入口。
- `random_sample`: top-level helper or orchestration entry point. / `random_sample`：顶层辅助函数或编排入口。
- `flashinfer_sample`: top-level helper or orchestration entry point. / `flashinfer_sample`：顶层辅助函数或编排入口。
- `_to_tensor_scalar_tuple`: top-level helper or orchestration entry point. / `_to_tensor_scalar_tuple`：顶层辅助函数或编排入口。

## Dependencies / 依赖关系
- External / 外部依赖: `torch`, `packaging`, `aiter`, `flashinfer`
- Internal vLLM / 内部依赖: `vllm`, `vllm._aiter_ops`, `vllm.config.model`, `vllm.logger`, `vllm.platforms`, `vllm.triton_utils`, `vllm.v1.sample.ops.topk_topp_triton`, `vllm.v1.attention.backends.flashinfer`
