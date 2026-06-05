# utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This module collects shared utility helpers that are reused across the multimodal generation stack. Key symbols include `expand_path_fields`, `find_nccl_library`, `_patched_set_stream`. / 该模块汇总了多模态生成栈中可复用的通用工具函数。 关键符号包括 `expand_path_fields`, `find_nccl_library`, `_patched_set_stream`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-35: Imports and module setup / 导入与模块初始化
```python
# Copied and adapted from: https://github.com/hao-ai-lab/FastVideo

# SPDX-License-Identifier: Apache-2.0
# Adapted from vllm: https://github.com/vllm-project/vllm/blob/v0.7.3/vllm/utils.py

import argparse
import ctypes
import importlib
import importlib.util
import inspect
import math
import os
import signal
import sys
# ...

logger = init_logger(__name__)

T = TypeVar("T")
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 38-52: Function `expand_path_fields` / 函数 `expand_path_fields`
```python
def expand_path_fields(obj) -> None:
    """In-place expanduser on all dataclass fields whose name ends with '_path' or '_paths'."""
    eu = os.path.expanduser
    for f in fields(obj):
        v = getattr(obj, f.name)
        if f.name.endswith("_path") and isinstance(v, str):
            setattr(obj, f.name, eu(v))
        elif f.name.endswith("_path") and isinstance(v, list):
            setattr(obj, f.name, [eu(x) if isinstance(x, str) else x for x in v])
        elif f.name.endswith("_paths") and isinstance(v, dict):
            setattr(
                obj,
                f.name,
                {k: eu(p) if isinstance(p, str) else p for k, p in v.items()},
            )
```
**EN:** This function drives `expand_path_fields` with inputs such as `obj`. In-place expanduser on all dataclass fields whose name ends with '_path' or '_paths'.
**CN:** 这个函数负责 `expand_path_fields`，主要处理 `obj` 等输入。 文档字符串说明：In-place expanduser on all dataclass fields whose name ends with '_path' or '_paths'.

### Lines 53-64: Top-level configuration / 顶层配置
```python


# TODO(will): used to convert server_args.precision to torch.dtype. Find a
# cleaner way to do this.
PRECISION_TO_TYPE = {
    "fp32": torch.float32,
    "fp16": torch.float16,
    "bf16": torch.bfloat16,
}

STR_BACKEND_ENV_VAR: str = "SGLANG_DIFFUSION_ATTENTION_BACKEND"
STR_ATTN_CONFIG_ENV_VAR: str = "SGLANG_DIFFUSION_ATTENTION_CONFIG"
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 67-92: Function `find_nccl_library` / 函数 `find_nccl_library`
```python
def find_nccl_library() -> str:
    """
    We either use the library file specified by the `VLLM_NCCL_SO_PATH`
    environment variable, or we find the library file brought by PyTorch.
    After importing `torch`, `libnccl.so.2`, `librccl.so.1` or `libmccl.so.2`
    can be found by `ctypes` automatically.
    """
    so_file = envs.SGLANG_DIFFUSION_NCCL_SO_PATH

    # manually load the nccl library
    if so_file:
        logger.info(
            "Found nccl from environment variable SGLANG_DIFFUSION_NCCL_SO_PATH=%s",
            so_file,
# ...
        else:
            raise ValueError("NCCL only supports CUDA, ROCm and MUSA backends.")
        logger.info("Found nccl from library %s", so_file)
    return str(so_file)
```
**EN:** This function drives `find_nccl_library`. We either use the library file specified by the `VLLM_NCCL_SO_PATH`
**CN:** 这个函数负责 `find_nccl_library`。 文档字符串说明：We either use the library file specified by the `VLLM_NCCL_SO_PATH`

### Lines 93-97: Top-level configuration / 顶层配置
```python


prev_set_stream = torch.cuda.set_stream

_current_stream = None
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 100-104: Function `_patched_set_stream` / 函数 `_patched_set_stream`
```python
def _patched_set_stream(stream: torch.cuda.Stream | None) -> None:
    global _current_stream
    _current_stream = stream
    if stream is not None:
        prev_set_stream(stream)
```
**EN:** This function drives `_patched_set_stream` with inputs such as `stream`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_patched_set_stream`，主要处理 `stream` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 105-107: Top-level configuration / 顶层配置
```python


torch.cuda.set_stream = _patched_set_stream
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 110-139: Function `current_stream` / 函数 `current_stream`
```python
def current_stream() -> torch.cuda.Stream | None:
    """
    replace `torch.cuda.current_stream()` with `sglang.multimodal_gen.utils.current_stream()`.
    it turns out that `torch.cuda.current_stream()` is quite expensive,
    as it will construct a new stream object at each call.
    here we patch `torch.cuda.set_stream` to keep track of the current stream
    directly, so that we can avoid calling `torch.cuda.current_stream()`.

    the underlying hypothesis is that we do not call `torch._C._cuda_setStream`
    from C/C++ code.
    """
    from sglang.multimodal_gen.runtime.platforms import current_platform

    # For non-CUDA platforms, return None
# ...
            if current_platform.is_rocm()
            else torch.cuda.current_stream()
        )
    return _current_stream
```
**EN:** This function drives `current_stream`. replace `torch.cuda.current_stream()` with `sglang.multimodal_gen.utils.current_stream()`.
**CN:** 这个函数负责 `current_stream`。 文档字符串说明：replace `torch.cuda.current_stream()` with `sglang.multimodal_gen.utils.current_stream()`.

### Lines 142-168: Class `StoreBoolean` / 类 `StoreBoolean`
```python
class StoreBoolean(argparse.Action):

    def __init__(self, option_strings, dest, default=False, required=False, help=None):
        super().__init__(
            option_strings=option_strings,
            dest=dest,
            nargs="?",
            const=True,
            default=default,
            required=required,
            help=help,
        )

    def __call__(self, parser, namespace, values, option_string=None):
# ...
                    f"Invalid boolean value: {values}. " "Expected 'true' or 'false'."
                )
        else:
            setattr(namespace, self.dest, bool(values))
```
**EN:** This class models `StoreBoolean` as a specialization of `argparse.Action`. Important methods include `__init__`, `__call__`.
**CN:** 该类实现 `StoreBoolean`，并继承/扩展 `argparse.Action`。 其中较重要的方法包括 `__init__`, `__call__`。

### Lines 171-389: Class `FlexibleArgumentParser` / 类 `FlexibleArgumentParser`
```python
class FlexibleArgumentParser(argparse.ArgumentParser):
    """ArgumentParser that allows both underscore and dash in names."""

    def __init__(self, *args, **kwargs) -> None:
        # Set the default 'formatter_class' to SortedHelpFormatter
        if "formatter_class" not in kwargs:
            kwargs["formatter_class"] = SortedHelpFormatter
        super().__init__(*args, **kwargs)

    def parse_args(  # type: ignore[override]
        self, args=None, namespace=None
    ) -> argparse.Namespace:
        if args is None:
            args = sys.argv[1:]
# ...

        process_dict("", config)

        return processed_args
```
**EN:** This class models `FlexibleArgumentParser` as a specialization of `argparse.ArgumentParser`. ArgumentParser that allows both underscore and dash in names. Important methods include `__init__`, `parse_args`, `_pull_args_from_config`, `_load_config_file`.
**CN:** 该类实现 `FlexibleArgumentParser`，并继承/扩展 `argparse.ArgumentParser`。 文档字符串指出：ArgumentParser that allows both underscore and dash in names. 其中较重要的方法包括 `__init__`, `parse_args`, `_pull_args_from_config`, `_load_config_file`。

### Lines 392-432: Function `warn_for_unimplemented_methods` / 函数 `warn_for_unimplemented_methods`
```python
def warn_for_unimplemented_methods(cls: type[T]) -> type[T]:
    """
    A replacement for `abc.ABC`.
    When we use `abc.ABC`, subclasses will fail to instantiate
    if they do not implement all abstract methods.
    Here, we only require `raise NotImplementedError` in the
    base class, and log a warning if the method is not implemented
    in the subclass.
    """

    original_init = cls.__init__

    def find_unimplemented_methods(self: object):
        unimplemented_methods = []
# ...
        find_unimplemented_methods(self)

    type.__setattr__(cls, "__init__", wrapped_init)
    return cls
```
**EN:** This function drives `warn_for_unimplemented_methods`. A replacement for `abc.ABC`.
**CN:** 这个函数负责 `warn_for_unimplemented_methods`。 文档字符串说明：A replacement for `abc.ABC`.

### Lines 435-445: Function `align_to` / 函数 `align_to`
```python
def align_to(value: int, alignment: int) -> int:
    """align height, width according to alignment

    Args:
        value (int): height or width
        alignment (int): target alignment factor

    Returns:
        int: the aligned value
    """
    return int(math.ceil(value / alignment) * alignment)
```
**EN:** This function drives `align_to` with inputs such as `value`, `alignment`. align height, width according to alignment
**CN:** 这个函数负责 `align_to`，主要处理 `value`, `alignment` 等输入。 文档字符串说明：align height, width according to alignment

### Lines 448-454: Function `resolve_obj_by_qualname` / 函数 `resolve_obj_by_qualname`
```python
def resolve_obj_by_qualname(qualname: str) -> Any:
    """
    Resolve an object by its fully qualified name.
    """
    module_name, obj_name = qualname.rsplit(".", 1)
    module = importlib.import_module(module_name)
    return getattr(module, obj_name)
```
**EN:** This function drives `resolve_obj_by_qualname` with inputs such as `qualname`. Resolve an object by its fully qualified name.
**CN:** 这个函数负责 `resolve_obj_by_qualname`，主要处理 `qualname` 等输入。 文档字符串说明：Resolve an object by its fully qualified name.

### Lines 458-485: Function `import_pynvml` / 函数 `import_pynvml`
```python
def import_pynvml():
    """
    Historical comments:

    libnvml.so is the library behind nvidia-smi, and
    pynvml is a Python wrapper around it. We use it to get GPU
    status without initializing CUDA context in the current process.
    Historically, there are two packages that provide pynvml:
    - `nvidia-ml-py` (https://pypi.org/project/nvidia-ml-py/): The official
        wrapper. It is a dependency of sglang-diffusion, and is installed when users
        install sglang-diffusion. It provides a Python module named `pynvml`.
    - `pynvml` (https://pypi.org/project/pynvml/): An unofficial wrapper.
        Prior to version 12.0, it also provides a Python module `pynvml`,
        and therefore conflicts with the official one which is a standalone Python file.
# ...
    """
    import sglang.multimodal_gen.third_party.pynvml as pynvml

    return pynvml
```
**EN:** This function drives `import_pynvml`. Historical comments:
**CN:** 这个函数负责 `import_pynvml`。 文档字符串说明：Historical comments:

### Lines 488-497: Function `update_environment_variables` / 函数 `update_environment_variables`
```python
def update_environment_variables(envs: dict[str, str]):
    for k, v in envs.items():
        if k in os.environ and os.environ[k] != v:
            logger.warning(
                "Overwriting environment variable %s " "from '%s' to '%s'",
                k,
                os.environ[k],
                v,
            )
        os.environ[k] = v
```
**EN:** This function drives `update_environment_variables` with inputs such as `envs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `update_environment_variables`，主要处理 `envs` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 500-521: Function `run_method` / 函数 `run_method`
```python
def run_method(
    obj: Any, method: str | bytes | Callable, args: tuple[Any], kwargs: dict[str, Any]
) -> Any:
    """
    Run a method of an object with the given arguments and keyword arguments.
    If the method is string, it will be converted to a method using getattr.
    If the method is serialized bytes and will be deserialized using
    cloudpickle.
    If the method is a callable, it will be called directly.
    """
    if isinstance(method, bytes):
        func = partial(cloudpickle.loads(method), obj)
    elif isinstance(method, str):
        try:
# ...
            ) from None
    else:
        func = partial(method, obj)  # type: ignore
    return func(*args, **kwargs)
```
**EN:** This function drives `run_method` with inputs such as `obj`, `method`, `args`, `kwargs`. Run a method of an object with the given arguments and keyword arguments.
**CN:** 这个函数负责 `run_method`，主要处理 `obj`, `method`, `args`, `kwargs` 等输入。 文档字符串说明：Run a method of an object with the given arguments and keyword arguments.

### Lines 524-527: Function `shallow_asdict` / 函数 `shallow_asdict`
```python
def shallow_asdict(obj) -> dict[str, Any]:
    if not is_dataclass(obj):
        raise TypeError("Expected dataclass instance")
    return {f.name: getattr(obj, f.name) for f in fields(obj)}
```
**EN:** This function drives `shallow_asdict` with inputs such as `obj`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `shallow_asdict`，主要处理 `obj` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 531-544: Function `kill_itself_when_parent_died` / 函数 `kill_itself_when_parent_died`
```python
def kill_itself_when_parent_died() -> None:
    # if sys.platform == "linux":
    # sigkill this process when parent worker manager dies
    PR_SET_PDEATHSIG = 1
    import platform

    if platform.system() == "Linux":
        libc = ctypes.CDLL("libc.so.6")
        libc.prctl(PR_SET_PDEATHSIG, signal.SIGKILL)
    # elif platform.system() == "Darwin":
    #     libc = ctypes.CDLL("libc.dylib")
    #     logger.warning("kill_itself_when_parent_died is only supported in linux.")
    else:
        logger.warning("kill_itself_when_parent_died is only supported in linux.")
```
**EN:** This function drives `kill_itself_when_parent_died`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `kill_itself_when_parent_died`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 547-550: Function `get_exception_traceback` / 函数 `get_exception_traceback`
```python
def get_exception_traceback() -> str:
    etype, value, tb = sys.exc_info()
    err_str = "".join(traceback.format_exception(etype, value, tb))
    return err_str
```
**EN:** This function drives `get_exception_traceback`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `get_exception_traceback`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 553-562: Class `TypeBasedDispatcher` / 类 `TypeBasedDispatcher`
```python
class TypeBasedDispatcher:

    def __init__(self, mapping: list[tuple[type, Callable]]):
        self._mapping = mapping

    def __call__(self, obj: Any):
        for ty, fn in self._mapping:
            if isinstance(obj, ty):
                return fn(obj)
        raise ValueError(f"Invalid object: {obj}")
```
**EN:** This class models `TypeBasedDispatcher`. Important methods include `__init__`, `__call__`.
**CN:** 该类实现 `TypeBasedDispatcher`。 其中较重要的方法包括 `__init__`, `__call__`。

### Lines 565-571: Class `MixedPrecisionState` / 类 `MixedPrecisionState`
```python
@dataclass
class MixedPrecisionState:
    param_dtype: torch.dtype | None = None
    reduce_dtype: torch.dtype | None = None
    output_dtype: torch.dtype | None = None
    compute_dtype: torch.dtype | None = None
    mp_policy: MixedPrecisionPolicy | None = None
```
**EN:** This class models `MixedPrecisionState`.
**CN:** 该类实现 `MixedPrecisionState`。

### Lines 572-575: Top-level configuration / 顶层配置
```python


# Thread-local storage for mixed precision state
_mixed_precision_state = threading.local()
```
**EN:** This block establishes imports, constants, registration hooks, or helper state used by later definitions.
**CN:** 该代码块建立后续定义所需的导入、常量、注册钩子或辅助状态。

### Lines 578-582: Function `get_mixed_precision_state` / 函数 `get_mixed_precision_state`
```python
def get_mixed_precision_state() -> MixedPrecisionState:
    """Get the current mixed precision state."""
    if not hasattr(_mixed_precision_state, "state"):
        raise ValueError("Mixed precision state not set")
    return cast(MixedPrecisionState, _mixed_precision_state.state)
```
**EN:** This function drives `get_mixed_precision_state`. Get the current mixed precision state.
**CN:** 这个函数负责 `get_mixed_precision_state`。 文档字符串说明：Get the current mixed precision state.

### Lines 585-604: Function `set_mixed_precision_policy` / 函数 `set_mixed_precision_policy`
```python
def set_mixed_precision_policy(
    param_dtype: torch.dtype,
    reduce_dtype: torch.dtype,
    output_dtype: torch.dtype | None = None,
    mp_policy: MixedPrecisionPolicy | None = None,
):
    """Set mixed precision policy globally.

    Args:
        param_dtype: Parameter dtype used for training
        reduce_dtype: Reduction dtype used for gradients
        output_dtype: Optional output dtype
    """
    state = MixedPrecisionState(
# ...
        output_dtype=output_dtype,
        mp_policy=mp_policy,
    )
    _mixed_precision_state.state = state
```
**EN:** This function drives `set_mixed_precision_policy` with inputs such as `param_dtype`, `reduce_dtype`, `output_dtype`, `mp_policy`. Set mixed precision policy globally.
**CN:** 这个函数负责 `set_mixed_precision_policy`，主要处理 `param_dtype`, `reduce_dtype`, `output_dtype`, `mp_policy` 等输入。 文档字符串说明：Set mixed precision policy globally.

### Lines 607-613: Function `get_compute_dtype` / 函数 `get_compute_dtype`
```python
def get_compute_dtype() -> torch.dtype:
    """Get the current compute dtype from mixed precision policy."""
    if not hasattr(_mixed_precision_state, "state"):
        return torch.get_default_dtype()
    else:
        state = get_mixed_precision_state()
        return state.param_dtype
```
**EN:** This function drives `get_compute_dtype`. Get the current compute dtype from mixed precision policy.
**CN:** 这个函数负责 `get_compute_dtype`。 文档字符串说明：Get the current compute dtype from mixed precision policy.

### Lines 616-674: Function `dict_to_3d_list` / 函数 `dict_to_3d_list`
```python
def dict_to_3d_list(
    mask_strategy: dict[str, Any] | None = None,
    t_max: int | None = None,
    l_max: int | None = None,
    h_max: int | None = None,
) -> list[list[list[torch.Tensor | None]]]:
    """
    Convert a dictionary of mask indices to a 3D list of tensors.
    Args:
        mask_strategy: keys are "t_l_h", values are torch.Tensor masks.
        t_max, l_max, h_max: if provided (all three), force the output shape to (t_max, l_max, h_max).
                            If all three are None, infer shape from the data.
    """
    # Case 1: no data, but fixed shape requested
# ...
            result[t][l][h] = value
        # else: silently ignore any key that doesn't fit

    return result
```
**EN:** This function drives `dict_to_3d_list` with inputs such as `mask_strategy`, `t_max`, `l_max`, `h_max`. Convert a dictionary of mask indices to a 3D list of tensors.
**CN:** 这个函数负责 `dict_to_3d_list`，主要处理 `mask_strategy`, `t_max`, `l_max`, `h_max` 等输入。 文档字符串说明：Convert a dictionary of mask indices to a 3D list of tensors.

### Lines 677-680: Function `set_random_seed` / 函数 `set_random_seed`
```python
def set_random_seed(seed: int) -> None:
    from sglang.multimodal_gen.runtime.platforms import current_platform

    current_platform.seed_everything(seed)
```
**EN:** This function drives `set_random_seed` with inputs such as `seed`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `set_random_seed`，主要处理 `seed` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 683-685: Function `is_vsa_available` / 函数 `is_vsa_available`
```python
@lru_cache(maxsize=1)
def is_vsa_available() -> bool:
    return importlib.util.find_spec("vsa") is not None
```
**EN:** This function drives `is_vsa_available`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_vsa_available`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 688-697: Function `is_vmoba_available` / 函数 `is_vmoba_available`
```python
@lru_cache(maxsize=1)
def is_vmoba_available() -> bool:
    if importlib.util.find_spec("kernel.csrc.attn.vmoba_attn.vmoba") is None:
        return False
    try:
        import flash_attn

        return flash_attn.__version__ >= "2.7.4"
    except Exception:
        return False
```
**EN:** This function drives `is_vmoba_available`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `is_vmoba_available`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 701-763: Function `masks_like` / 函数 `masks_like`
```python
def masks_like(
    tensors, zero=False, generator=None, p=0.2
) -> tuple[list[torch.Tensor], list[torch.Tensor]]:
    """
    Generate binary masks for Text-to-Image-to-Video (TI2V) tasks.

    Creates masks to control which frames should be preserved vs replaced.
    Primarily used to fix the first frame to the input image while generating other frames.

    Args:
        tensors: List of tensors with shape [C, T, H, W]
        zero: If True, set first frame (dim 1, index 0) to zero. Default: False
        generator: Optional random generator for stochastic masking
        p: Probability of applying special noise when generator is provided. Default: 0.2
# ...
                u[:, 0] = torch.zeros_like(u[:, 0])
                v[:, 0] = torch.zeros_like(v[:, 0])

    return out1, out2
```
**EN:** This function drives `masks_like` with inputs such as `tensors`, `zero`, `generator`, `p`. Generate binary masks for Text-to-Image-to-Video (TI2V) tasks.
**CN:** 这个函数负责 `masks_like`，主要处理 `tensors`, `zero`, `generator`, `p` 等输入。 文档字符串说明：Generate binary masks for Text-to-Image-to-Video (TI2V) tasks.

### Lines 767-789: Function `best_output_size` / 函数 `best_output_size`
```python
def best_output_size(w, h, dw, dh, expected_area):
    # float output size
    ratio = w / h
    ow = (expected_area * ratio) ** 0.5
    oh = expected_area / ow

    # process width first
    ow1 = int(ow // dw * dw)
    oh1 = int(expected_area / ow1 // dh * dh)
    assert ow1 % dw == 0 and oh1 % dh == 0 and ow1 * oh1 <= expected_area
    ratio1 = ow1 / oh1

    # process height first
    oh2 = int(oh // dh * dh)
# ...
    if max(ratio / ratio1, ratio1 / ratio) < max(ratio / ratio2, ratio2 / ratio):
        return ow1, oh1
    else:
        return ow2, oh2
```
**EN:** This function drives `best_output_size` with inputs such as `w`, `h`, `dw`, `dh`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `best_output_size`，主要处理 `w`, `h`, `dw`, `dh` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 792-799: Function `calculate_dimensions` / 函数 `calculate_dimensions`
```python
def calculate_dimensions(target_area, ratio):
    width = math.sqrt(target_area * ratio)
    height = width / ratio

    width = round(width / 32) * 32
    height = round(height / 32) * 32

    return width, height, None
```
**EN:** This function drives `calculate_dimensions` with inputs such as `target_area`, `ratio`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `calculate_dimensions`，主要处理 `target_area`, `ratio` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Attention mechanism / 注意力机制
- Artifact storage management / 产物存储管理
- Caching strategy / 缓存策略
- Distributed execution / 分布式执行

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.envs`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.runtime.platforms`, `sglang.multimodal_gen.third_party.pynvml`
- **External / 外部**: `ctypes`, `cloudpickle`, `torch`, `yaml`, `torch.distributed.fsdp`, `platform`, `flash_attn`
- **Stdlib / 标准库**: `argparse`, `importlib`, `importlib.util`, `inspect`, `math`, `os`, `signal`, `sys`
