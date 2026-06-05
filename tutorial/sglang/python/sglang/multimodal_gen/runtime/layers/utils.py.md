# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `CustomOpWrapper`, `get_group_size`, and `get_group_rank`, organizing the main control flow, data structures, or helper routines for this module. The module docstring highlights: Utility methods for model layers. / 该文件属于运行时算子层。它围绕 `CustomOpWrapper`、`get_group_size` 和 `get_group_rank` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。 文档字符串进一步说明了该模块的职责范围。

## Line-by-Line Analysis / 逐行分析
### Lines 5-14: module setup and imports / 模块初始化与导入
```python
"""Utility methods for model layers."""

import inspect
from typing import Any, Callable, List, Optional

import torch
from torch.library import Library

from sglang.kernel_api_logging import debug_torch_op
from sglang.multimodal_gen.runtime.platforms import current_platform
```
**EN:** This block establishes the module context and imports `inspect`, `typing`, `torch`, `torch.library`, `sglang.kernel_api_logging`, and `sglang.multimodal_gen.runtime.platforms`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `inspect`、`typing`、`torch`、`torch.library`、`sglang.kernel_api_logging` 和 `sglang.multimodal_gen.runtime.platforms`。这些依赖为后续实现提供所需符号。

### Lines 17-23: `get_group_size` implementation / `get_group_size` 实现
```python
def get_group_size(group) -> int:
    if hasattr(group, "world_size"):
        return group.world_size  # GroupCoordinator
    elif hasattr(group, "size") and callable(getattr(group, "size", None)):
        return group.size()  # ProcessGroup
    else:
        raise ValueError(f"Unsupported group type: {type(group)}")
```
**EN:** This block defines function `get_group_size`. It retrieves group size. Key calls include `hasattr`, `callable`, `group.size`, `ValueError`, and `getattr`. The implementation branches on conditions. Parameters such as `group` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_group_size`。 它用于获取group size。 关键调用包括 `hasattr`、`callable`、`group.size`、`ValueError` 和 `getattr`。 实现中包含条件分支。 本段逻辑主要由 `group` 等参数驱动。

### Lines 26-32: `get_group_rank` implementation / `get_group_rank` 实现
```python
def get_group_rank(group) -> int:
    if hasattr(group, "rank_in_group"):
        return group.rank_in_group  # GroupCoordinator
    elif hasattr(group, "rank") and callable(getattr(group, "rank", None)):
        return group.rank()  # ProcessGroup
    else:
        raise ValueError(f"Unsupported group type: {type(group)}")
```
**EN:** This block defines function `get_group_rank`. It retrieves group rank. Key calls include `hasattr`, `callable`, `group.rank`, `ValueError`, and `getattr`. The implementation branches on conditions. Parameters such as `group` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_group_rank`。 它用于获取group rank。 关键调用包括 `hasattr`、`callable`、`group.rank`、`ValueError` 和 `getattr`。 实现中包含条件分支。 本段逻辑主要由 `group` 等参数驱动。

### Lines 35-49: `get_token_bin_counts_and_mask` implementation / `get_token_bin_counts_and_mask` 实现
```python
def get_token_bin_counts_and_mask(
    tokens: torch.Tensor,
    vocab_size: int,
    num_seqs: int,
) -> tuple[torch.Tensor, torch.Tensor]:
    # Compute the bin counts for the tokens.
    # vocab_size + 1 for padding.
    bin_counts = torch.zeros(
        (num_seqs, vocab_size + 1), dtype=torch.long, device=tokens.device
    )
    bin_counts.scatter_add_(1, tokens, torch.ones_like(tokens))
    bin_counts = bin_counts[:, :vocab_size]
    mask = bin_counts > 0

    return bin_counts, mask
```
**EN:** This block defines function `get_token_bin_counts_and_mask`. It retrieves token bin counts and mask. Key calls include `torch.zeros`, `bin_counts.scatter_add_`, and `torch.ones_like`. Parameters such as `tokens`, `vocab_size`, and `num_seqs` drive the behavior in this section.
**CN:** 该代码块定义了函数 `get_token_bin_counts_and_mask`。 它用于获取token bin counts and mask。 关键调用包括 `torch.zeros`、`bin_counts.scatter_add_` 和 `torch.ones_like`。 本段逻辑主要由 `tokens`、`vocab_size` 和 `num_seqs` 等参数驱动。

### Lines 52-52: supporting statements / 辅助语句
```python
sglang_lib = Library("sglang", "FRAGMENT")  # noqa
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `sglang_lib`. The code collaborates with `Library`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `sglang_lib` 等名称。 代码会与 `Library` 协同工作。

### Lines 55-126: `direct_register_custom_op` implementation / `direct_register_custom_op` 实现
```python
def direct_register_custom_op(
    op_name: str,
    op_func: Callable,
    mutates_args: List[str],
    fake_impl: Optional[Callable] = None,
    target_lib: Optional[Library] = None,
):
    """
    `torch.library.custom_op` can have significant overhead because it
    needs to consider complicated dispatching logic. This function
    directly registers a custom op and dispatches it to the CUDA backend.
    See https://gist.github.com/youkaichao/ecbea9ec9fc79a45d2adce1784d7a9a5
    for more details.

    By default, the custom op is registered to the vLLM library. If you
    want to register it to a different library, you can pass the library
    object to the `target_lib` argument.

    IMPORTANT: the lifetime of the operator is tied to the lifetime of the
    library object. If you want to bind the operator to a different library,
    make sure the library object is alive when the operator is used.

    Note: This function will silently skip registration if the operator
    with the same name is already registered to avoid RuntimeError in
    multi-engine scenarios (e.g., VERL framework).
    """
    import torch.library

    my_lib = target_lib or sglang_lib

    # Check if operator is already registered to avoid duplicate registration
    # This is important for scenarios where multiple SGLang engines run in the same process
    try:
        # Try to access the operator to see if it's already registered
        lib_name = my_lib.m.name if hasattr(my_lib.m, "name") else "sglang"
        if hasattr(torch.ops, lib_name) and hasattr(
            getattr(torch.ops, lib_name), op_name
        ):
            # Operator already exists, skip registration
            return
    except (AttributeError, RuntimeError):
        # Operator doesn't exist, proceed with registration
        pass

    if hasattr(torch.library, "infer_schema"):
        schema_str = torch.library.infer_schema(op_func, mutates_args=mutates_args)
    else:
        # for pytorch 2.4
        import torch._custom_op.impl

        schema_str = torch._custom_op.impl.infer_schema(op_func, mutates_args)

    try:
        my_lib.define(op_name + schema_str)
        my_lib.impl(
            op_name, op_func, "CUDA" if not current_platform.is_npu() else "PrivateUse1"
        )
        if fake_impl is not None:
            my_lib._register_fake(op_name, fake_impl)
    except RuntimeError as error:
        if "Tried to register an operator" in str(error) and "multiple times" in str(
            error
        ):
            # Silently ignore duplicate registration errors
            # This can happen in multi-engine scenarios
            pass
        else:
            # Re-raise other RuntimeErrors
            raise error
    except AttributeError as error:
        # Always re-raise AttributeError as it indicates missing dependencies
        raise error
```
**EN:** This block defines function `direct_register_custom_op`. `torch.library.custom_op` can have significant overhead because it needs to consider complicated dispatching logic. This function directly registers a custom op and dispatches it to the CUDA backend. Key calls include `hasattr`, `torch.library.infer_schema`, `torch._custom_op.impl.infer_schema`, `my_lib.define`, and `my_lib.impl`. The implementation branches on conditions, handles exceptional paths. Parameters such as `op_name`, `op_func`, `mutates_args`, `fake_impl`, and `target_lib` drive the behavior in this section.
**CN:** 该代码块定义了函数 `direct_register_custom_op`。 它用于处理 direct register custom op 相关逻辑。 关键调用包括 `hasattr`、`torch.library.infer_schema`、`torch._custom_op.impl.infer_schema`、`my_lib.define` 和 `my_lib.impl`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `op_name`、`op_func`、`mutates_args`、`fake_impl` 和 `target_lib` 等参数驱动。

### Lines 129-129: `CustomOpWrapper` class overview / `CustomOpWrapper` 类概览
```python
class CustomOpWrapper:
```
**EN:** This block defines class `CustomOpWrapper`. It encapsulates custom op wrapper behavior.
**CN:** 该代码块定义了类 `CustomOpWrapper`。 它用于封装 custom op wrapper 相关行为。

### Lines 130-141: `__init__` implementation / `__init__` 实现
```python
    def __init__(
        self,
        op_name: str,
        op_func: Callable,
        mutates_args: List[str],
        **extra_kwargs,
    ):
        self.op_name = op_name
        self.op_func = op_func
        self.mutates_args = mutates_args
        self.extra_kwargs = extra_kwargs
        self._impl: Optional[Callable] = None
```
**EN:** This block defines method `__init__` on `CustomOpWrapper`. It initializes the instance state. Parameters such as `op_name`, `op_func`, and `mutates_args` drive the behavior in this section.
**CN:** 该代码块定义了 `CustomOpWrapper` 的方法 `__init__`。 它用于初始化实例状态。 本段逻辑主要由 `op_name`、`op_func` 和 `mutates_args` 等参数驱动。

### Lines 143-144: `__call__` implementation / `__call__` 实现
```python
    def __call__(self, *args, **kwargs):
        return self.real_impl(*args, **kwargs)
```
**EN:** This block defines method `__call__` on `CustomOpWrapper`. It makes the object callable. Key calls include `self.real_impl`.
**CN:** 该代码块定义了 `CustomOpWrapper` 的方法 `__call__`。 它用于让对象可以像函数一样被调用。 关键调用包括 `self.real_impl`。

### Lines 146-161: `real_impl` implementation / `real_impl` 实现
```python
    @property
    def real_impl(self) -> Callable:
        if self._impl is None:
            if not hasattr(torch.ops.sglang, self.op_name):

                # NOTE(dark): if torch compile fail here, mark the decorator as eager
                # lazy registration does not work with torch compile
                direct_register_custom_op(
                    op_name=self.op_name,
                    op_func=self.op_func,
                    mutates_args=self.mutates_args,
                    fake_impl=self.fake_impl,
                )
            self._impl = debug_torch_op(self.op_func, self.op_name)
            assert self._impl is not None
        return self._impl
```
**EN:** This block defines method `real_impl` on `CustomOpWrapper`. It handles real impl logic. Key calls include `debug_torch_op`, `hasattr`, and `direct_register_custom_op`. The implementation branches on conditions.
**CN:** 该代码块定义了 `CustomOpWrapper` 的方法 `real_impl`。 它用于处理 real impl 相关逻辑。 关键调用包括 `debug_torch_op`、`hasattr` 和 `direct_register_custom_op`。 实现中包含条件分支。

### Lines 163-190: `fake_impl` implementation / `fake_impl` 实现
```python
    @property
    def fake_impl(self) -> Callable:
        if "fake_impl" in self.extra_kwargs:
            return self.extra_kwargs["fake_impl"]
        assert "out_shape" in self.extra_kwargs
        signature = inspect.signature(self.op_func)
        out_shape = self.extra_kwargs["out_shape"]

        # check out_shape in signature

        def fake_impl(*args, **kwargs):
            if out_shape is None:
                return None
            bound = signature.bind(*args, **kwargs)
            bound.apply_defaults()
            try:
                return torch.empty_like(
                    bound.args[out_shape]
                    if isinstance(out_shape, int)
                    else bound.arguments[out_shape]
                )
            except (IndexError, KeyError):
                raise RuntimeError(
                    f"Cannot find output argument at position `{out_shape}` for "
                    f"custom operator `{self.op_name}` with signature `{signature}`."
                )

        return fake_impl
```
**EN:** This block defines method `fake_impl` on `CustomOpWrapper`. It handles fake impl logic. Key calls include `inspect.signature`, `signature.bind`, `bound.apply_defaults`, `torch.empty_like`, and `RuntimeError`. The implementation branches on conditions, handles exceptional paths.
**CN:** 该代码块定义了 `CustomOpWrapper` 的方法 `fake_impl`。 它用于处理 fake impl 相关逻辑。 关键调用包括 `inspect.signature`、`signature.bind`、`bound.apply_defaults`、`torch.empty_like` 和 `RuntimeError`。 实现中包含条件分支，处理异常路径。

### Lines 194-267: `register_custom_op` implementation / `register_custom_op` 实现
```python
def register_custom_op(
    fn: Optional[Callable] = None,
    *,
    op_name: Optional[str] = None,
    mutates_args: Optional[List[str]] = None,
    eager: bool = True,
    **extra_kwargs,
) -> Any:
    """
    A decorator to register a custom operator.

    Example usage:
    ```python
    # inplace operator, out_shape is None by default
    @register_custom_op(mutates_args=["x"])
    def add_1_(x: torch.Tensor) -> None:
        x.add_(1)

    # operator with output, out_shape indicates the position of output
    @register_custom_op(mutates_args=["x"], out_shape=0)
    def add(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor:
        return x.add_(y)
    ```

    :param fn: The function to be registered as a custom operator.
               If None, return a decorator.
    :type fn: Callable
    :param op_name: The name of the operator. If None, use the function name
    :type op_name: Optional[str]
    :param mutates_args: A list of argument names that are mutated in-place.
    :type mutates_args: List[str]
    :param out_shape: The position (int for positional, str for keyword) of the output-shape tensor.
                      It is used to generate a fake implementation for torch.compile compatibility.
                      If the operator is inplace and has no output, set to None.
    :type out_shape: Optional[List[Union[int, str]]]
    :param fake_impl: A fake implementation for the operator.
                      Only one of `out_shape` or `fake_impl` should be provided.
    :type fake_impl: Optional[Callable]
    :param eager: Whether to register the operator eagerly.
                  If False, the registration will be deferred until the first call.
                  If you met any issue with torch.compile, try to set eager=True.
                  Currently, to avoid misuse, we set eager=True by default.
    :type eager: bool
    :return: The registered JIT custom operator, or a decorator.
             NOTE: the real register will occur at the first call of the function.
    :rtype: Callable
    """
    extra_kwarg_keys = set(extra_kwargs.keys())
    expected_kwarg_keys = set({"out_shape", "fake_impl"})
    assert (
        expected_kwarg_keys >= extra_kwarg_keys
    ), f"Unexpected extra kwargs: {extra_kwarg_keys - expected_kwarg_keys}"

    has_out_shape = "out_shape" in extra_kwargs
    has_fake_impl = "fake_impl" in extra_kwargs
    assert not (
        has_out_shape and has_fake_impl
    ), "Only one of `out_shape` or `fake_impl` should be provided."
    # Assume inplace if neither out_shape nor fake_impl is provided
    if not (has_out_shape or has_fake_impl):
        extra_kwargs["out_shape"] = None

    def decorator(op_func: Callable) -> Callable:
        wrapper = CustomOpWrapper(
            op_name=op_name or op_func.__name__,
            op_func=op_func,
            mutates_args=mutates_args or [],
            **extra_kwargs,
        )
        return wrapper.real_impl if eager else wrapper

    if fn is not None:
        return decorator(fn)
    return decorator
```
**EN:** This block defines function `register_custom_op`. A decorator to register a custom operator. Example usage: ```python # inplace operator, out_shape is None by default @register_custom_op(mutates_args=["x"]) def add_1_(x: torch.Tensor) -> None: x.add_(1) # operator with output, out_shape indicates the position of output @register_custom_op(mutates_args=["x"], out_shape=0) def add(x: torch.Tensor, y: torch.Tensor) -> torch.Tensor: return x.add_(y) ``` :param fn: The function to be registered as a custom operator. Key calls include `set`, `extra_kwargs.keys`, `CustomOpWrapper`, and `decorator`. The implementation branches on conditions. Parameters such as `fn` drive the behavior in this section.
**CN:** 该代码块定义了函数 `register_custom_op`。 它用于注册custom op。 关键调用包括 `set`、`extra_kwargs.keys`、`CustomOpWrapper` 和 `decorator`。 实现中包含条件分支。 本段逻辑主要由 `fn` 等参数驱动。

## Key Concepts / 关键概念
- `get_group_size`: Top-level function that retrieves group size. / 顶层函数，用于获取group size。
- `get_group_rank`: Top-level function that retrieves group rank. / 顶层函数，用于获取group rank。
- `get_token_bin_counts_and_mask`: Top-level function that retrieves token bin counts and mask. / 顶层函数，用于获取token bin counts and mask。
- `direct_register_custom_op`: `torch.library.custom_op` can have significant overhead because it needs to consider complicated dispatching logic. / 顶层函数，用于处理 direct register custom op 相关逻辑。
- `CustomOpWrapper`: Primary class that encapsulates custom op wrapper behavior. / 核心类，用于封装 custom op wrapper 相关行为。
- `register_custom_op`: A decorator to register a custom operator. / 顶层函数，用于注册custom op。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `inspect`, `typing`
- **Third-party / 第三方依赖**: `torch`, `torch.library`, `torch._custom_op.impl`
- **Internal modules / 内部模块**: `sglang.kernel_api_logging`, `sglang.multimodal_gen.runtime.platforms`

- **Total lines / 总行数**: 267
