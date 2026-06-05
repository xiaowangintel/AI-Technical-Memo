# envs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/envs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the multimodal generation module. It centers on `get_default_cache_root`, `get_default_config_root`, and `maybe_convert_int`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于多模态生成模块。它围绕 `get_default_cache_root`、`get_default_config_root` 和 `maybe_convert_int` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 6-10: module setup and imports / 模块初始化与导入
```python
import logging
import os
from typing import TYPE_CHECKING, Any, Callable

from sglang.multimodal_gen.runtime.utils.common import get_bool_env_var
```
**EN:** This block establishes the module context and imports `logging`, `os`, `typing`, and `sglang.multimodal_gen.runtime.utils.common`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `logging`、`os`、`typing` 和 `sglang.multimodal_gen.runtime.utils.common`。这些依赖为后续实现提供所需符号。

### Lines 12-64: supporting statements / 辅助语句
```python
logger = logging.getLogger(__name__)

if TYPE_CHECKING:
    SGLANG_DIFFUSION_RINGBUFFER_WARNING_INTERVAL: int = 60
    SGLANG_DIFFUSION_NCCL_SO_PATH: str | None = None
    LD_LIBRARY_PATH: str | None = None
    LOCAL_RANK: int = 0
    CUDA_VISIBLE_DEVICES: str | None = None
    SGLANG_DIFFUSION_CACHE_ROOT: str = os.path.expanduser("~/.cache/sgl_diffusion")
    SGLANG_DIFFUSION_CONFIG_ROOT: str = os.path.expanduser("~/.config/sgl_diffusion")
    SGLANG_DIFFUSION_CONFIGURE_LOGGING: int = 1
    SGLANG_DIFFUSION_LOGGING_LEVEL: str = "INFO"
    SGLANG_DIFFUSION_LOGGING_PREFIX: str = ""
    SGLANG_DIFFUSION_LOGGING_CONFIG_PATH: str | None = None
    SGLANG_DIFFUSION_TRACE_FUNCTION: int = 0
    SGLANG_DIFFUSION_WORKER_MULTIPROC_METHOD: str = "fork"
    SGLANG_DIFFUSION_TARGET_DEVICE: str = "cuda"
    MAX_JOBS: str | None = None
    NVCC_THREADS: str | None = None
    CMAKE_BUILD_TYPE: str | None = None
    VERBOSE: bool = False
    SGLANG_DIFFUSION_SERVER_DEV_MODE: bool = False
    SGLANG_DIFFUSION_STAGE_LOGGING: bool = False
    # cache-dit env vars (primary transformer)
    SGLANG_CACHE_DIT_ENABLED: bool = False
    SGLANG_CACHE_DIT_FN: int = 1
    SGLANG_CACHE_DIT_BN: int = 0
    SGLANG_CACHE_DIT_WARMUP: int = 4
    SGLANG_CACHE_DIT_RDT: float = 0.24
    SGLANG_CACHE_DIT_MC: int = 3
    SGLANG_CACHE_DIT_TAYLORSEER: bool = False
    SGLANG_CACHE_DIT_TS_ORDER: int = 1
    SGLANG_CACHE_DIT_SCM_PRESET: str = "none"
    SGLANG_CACHE_DIT_SCM_COMPUTE_BINS: str | None = None
    SGLANG_CACHE_DIT_SCM_CACHE_BINS: str | None = None
    SGLANG_CACHE_DIT_SCM_POLICY: str = "dynamic"
    # cache-dit env vars (secondary transformer, e.g., Wan2.2 low-noise expert)
    SGLANG_CACHE_DIT_SECONDARY_FN: int = 1
    SGLANG_CACHE_DIT_SECONDARY_BN: int = 0
    SGLANG_CACHE_DIT_SECONDARY_WARMUP: int = 4
    SGLANG_CACHE_DIT_SECONDARY_RDT: float = 0.24
    SGLANG_CACHE_DIT_SECONDARY_MC: int = 3
    SGLANG_CACHE_DIT_SECONDARY_TAYLORSEER: bool = False
    SGLANG_CACHE_DIT_SECONDARY_TS_ORDER: int = 1
    # model loading
    SGLANG_USE_RUNAI_MODEL_STREAMER: bool = True
    SGLANG_DIFFUSION_FLASHINFER_FP4_GEMM_BACKEND: str | None = None
    SGLANG_DIFFUSION_VAE_CHANNELS_LAST_3D: bool = True
    SGLANG_USE_CUDA_HUNYUANVIDEO_GROUP_NORM_SILU: bool = False
    SGLANG_USE_ROCM_VAE: bool = False
    SGLANG_USE_ROCM_CUDNN_BENCHMARK: bool = False
    SGLANG_USE_ROCM_VAE_CONV2D: bool = False
    SGLANG_USE_ROCM_VAE_CONV2D_BF16: bool = False
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `logging.getLogger`, and `os.path.expanduser`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `logging.getLogger` 和 `os.path.expanduser` 协同工作。

### Lines 67-71: `get_default_cache_root` implementation / `get_default_cache_root` 实现
```python
def get_default_cache_root() -> str:
    return os.getenv(
        "XDG_CACHE_HOME",
        os.path.join(os.path.expanduser("~"), ".cache"),
    )
```
**EN:** This block defines function `get_default_cache_root`. It retrieves default cache root. Key calls include `os.getenv`, `os.path.join`, and `os.path.expanduser`.
**CN:** 该代码块定义了函数 `get_default_cache_root`。 它用于获取default cache root。 关键调用包括 `os.getenv`、`os.path.join` 和 `os.path.expanduser`。

### Lines 74-78: `get_default_config_root` implementation / `get_default_config_root` 实现
```python
def get_default_config_root() -> str:
    return os.getenv(
        "XDG_CONFIG_HOME",
        os.path.join(os.path.expanduser("~"), ".config"),
    )
```
**EN:** This block defines function `get_default_config_root`. It retrieves default config root. Key calls include `os.getenv`, `os.path.join`, and `os.path.expanduser`.
**CN:** 该代码块定义了函数 `get_default_config_root`。 它用于获取default config root。 关键调用包括 `os.getenv`、`os.path.join` 和 `os.path.expanduser`。

### Lines 81-82: `maybe_convert_int` implementation / `maybe_convert_int` 实现
```python
def maybe_convert_int(value: str | None) -> int | None:
    return int(value) if value is not None else None
```
**EN:** This block defines function `maybe_convert_int`. It handles maybe convert int logic. Key calls include `int`. Parameters such as `value` drive the behavior in this section.
**CN:** 该代码块定义了函数 `maybe_convert_int`。 它用于处理 maybe convert int 相关逻辑。 关键调用包括 `int`。 本段逻辑主要由 `value` 等参数驱动。

### Lines 86-87: `_lazy_str` implementation / `_lazy_str` 实现
```python
def _lazy_str(key: str, default: str | None = None) -> Callable[[], str | None]:
    return lambda: os.getenv(key, default)
```
**EN:** This block defines function `_lazy_str`. It handles lazy str logic. Key calls include `os.getenv`. Parameters such as `key`, and `default` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_lazy_str`。 它用于处理 lazy str 相关逻辑。 关键调用包括 `os.getenv`。 本段逻辑主要由 `key` 和 `default` 等参数驱动。

### Lines 90-97: `_lazy_int` implementation / `_lazy_int` 实现
```python
def _lazy_int(key: str, default: str | int | None = None) -> Callable[[], int | None]:
    def _getter():
        val = os.getenv(key)
        if val is None:
            return int(default) if default is not None else None
        return int(val)

    return _getter
```
**EN:** This block defines function `_lazy_int`. It handles lazy int logic. Key calls include `os.getenv`, and `int`. The implementation branches on conditions. Parameters such as `key`, and `default` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_lazy_int`。 它用于处理 lazy int 相关逻辑。 关键调用包括 `os.getenv` 和 `int`。 实现中包含条件分支。 本段逻辑主要由 `key` 和 `default` 等参数驱动。

### Lines 100-101: `_lazy_float` implementation / `_lazy_float` 实现
```python
def _lazy_float(key: str, default: str | float) -> Callable[[], float]:
    return lambda: float(os.getenv(key, str(default)))
```
**EN:** This block defines function `_lazy_float`. It handles lazy float logic. Key calls include `float`, `os.getenv`, and `str`. Parameters such as `key`, and `default` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_lazy_float`。 它用于处理 lazy float 相关逻辑。 关键调用包括 `float`、`os.getenv` 和 `str`。 本段逻辑主要由 `key` 和 `default` 等参数驱动。

### Lines 104-105: `_lazy_bool` implementation / `_lazy_bool` 实现
```python
def _lazy_bool(key: str, default: str = "false") -> Callable[[], bool]:
    return lambda: get_bool_env_var(key, default)
```
**EN:** This block defines function `_lazy_bool`. It handles lazy bool logic. Key calls include `get_bool_env_var`. Parameters such as `key`, and `default` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_lazy_bool`。 它用于处理 lazy bool 相关逻辑。 关键调用包括 `get_bool_env_var`。 本段逻辑主要由 `key` 和 `default` 等参数驱动。

### Lines 108-119: `_lazy_bool_any` implementation / `_lazy_bool_any` 实现
```python
def _lazy_bool_any(keys: list[str], default: str = "false") -> Callable[[], bool]:
    def _getter():
        for key in keys:
            if get_bool_env_var(key, "false"):
                return True
        return (
            get_bool_env_var("", default)
            if not keys
            else get_bool_env_var(keys[0], default)
        )

    return _getter
```
**EN:** This block defines function `_lazy_bool_any`. It handles lazy bool any logic. Key calls include `get_bool_env_var`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `keys`, and `default` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_lazy_bool_any`。 它用于处理 lazy bool any 相关逻辑。 关键调用包括 `get_bool_env_var`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `keys` 和 `default` 等参数驱动。

### Lines 122-133: `_lazy_path` implementation / `_lazy_path` 实现
```python
def _lazy_path(
    key: str, default_func: Callable[[], str] | None = None
) -> Callable[[], str | None]:
    def _getter():
        val = os.getenv(key)
        if val is None:
            if default_func is None:
                return None
            val = default_func()
        return os.path.expanduser(val)

    return _getter
```
**EN:** This block defines function `_lazy_path`. It handles lazy path logic. Key calls include `os.getenv`, `os.path.expanduser`, and `default_func`. The implementation branches on conditions. Parameters such as `key`, and `default_func` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_lazy_path`。 它用于处理 lazy path 相关逻辑。 关键调用包括 `os.getenv`、`os.path.expanduser` 和 `default_func`。 实现中包含条件分支。 本段逻辑主要由 `key` 和 `default_func` 等参数驱动。

### Lines 141-313: supporting statements / 辅助语句
```python
environment_variables: dict[str, Callable[[], Any]] = {
    # ================== Installation Time Env Vars ==================
    # Target device of sglang-diffusion, supporting [cuda (by default),
    # rocm, neuron, cpu, openvino]
    "SGLANG_DIFFUSION_TARGET_DEVICE": _lazy_str(
        "SGLANG_DIFFUSION_TARGET_DEVICE", "cuda"
    ),
    # Maximum number of compilation jobs to run in parallel.
    # By default this is the number of CPUs
    "MAX_JOBS": _lazy_str("MAX_JOBS"),
    # Number of threads to use for nvcc
    # By default this is 1.
    # If set, `MAX_JOBS` will be reduced to avoid oversubscribing the CPU.
    "NVCC_THREADS": _lazy_str("NVCC_THREADS"),
    # If set, sgl_diffusion will use precompiled binaries (*.so)
    "SGLANG_DIFFUSION_USE_PRECOMPILED": _lazy_bool_any(
        [
            "SGLANG_DIFFUSION_USE_PRECOMPILED",
            "SGLANG_DIFFUSION_PRECOMPILED_WHEEL_LOCATION",
        ]
    ),
    # CMake build type
    # If not set, defaults to "Debug" or "RelWithDebInfo"
    # Available options: "Debug", "Release", "RelWithDebInfo"
    "CMAKE_BUILD_TYPE": _lazy_str("CMAKE_BUILD_TYPE"),
    # If set, sgl_diffusion will print verbose logs during installation
    "VERBOSE": _lazy_bool("VERBOSE"),
    # Root directory for SGL-diffusion configuration files
    # Defaults to `~/.config/sgl_diffusion` unless `XDG_CONFIG_HOME` is set
    # Note that this not only affects how sgl_diffusion finds its configuration files
    # during runtime, but also affects how sgl_diffusion installs its configuration
    # files during **installation**.
    "SGLANG_DIFFUSION_CONFIG_ROOT": _lazy_path(
        "SGLANG_DIFFUSION_CONFIG_ROOT",
        lambda: os.path.join(get_default_config_root(), "sgl_diffusion"),
    ),
    # ================== Runtime Env Vars ==================
    # Root directory for SGL-diffusion cache files
    # Defaults to `~/.cache/sgl_diffusion` unless `XDG_CACHE_HOME` is set
    "SGLANG_DIFFUSION_CACHE_ROOT": _lazy_path(
        "SGLANG_DIFFUSION_CACHE_ROOT",
        lambda: os.path.join(get_default_cache_root(), "sgl_diffusion"),
    ),
    # Interval in seconds to log a warning message when the ring buffer is full
    "SGLANG_DIFFUSION_RINGBUFFER_WARNING_INTERVAL": _lazy_int(
        "SGLANG_DIFFUSION_RINGBUFFER_WARNING_INTERVAL", 60
    ),
    # Path to the NCCL library file. It is needed because nccl>=2.19 brought
    # by PyTorch contains a bug: https://github.com/NVIDIA/nccl/issues/1234
    "SGLANG_DIFFUSION_NCCL_SO_PATH": _lazy_str("SGLANG_DIFFUSION_NCCL_SO_PATH"),
    # when `SGLANG_DIFFUSION_NCCL_SO_PATH` is not set, sgl_diffusion will try to find the nccl
    # library file in the locations specified by `LD_LIBRARY_PATH`
    "LD_LIBRARY_PATH": _lazy_str("LD_LIBRARY_PATH"),
    # Internal flag to enable Dynamo fullgraph capture
    "SGLANG_DIFFUSION_TEST_DYNAMO_FULLGRAPH_CAPTURE": _lazy_bool(
        "SGLANG_DIFFUSION_TEST_DYNAMO_FULLGRAPH_CAPTURE", "1"
    ),
    # local rank of the process in the distributed setting, used to determine
    # the GPU device id
    "LOCAL_RANK": _lazy_int("LOCAL_RANK", 0),
    # used to control the visible devices in the distributed setting
    "CUDA_VISIBLE_DEVICES": _lazy_str("CUDA_VISIBLE_DEVICES"),
    # timeout for each iteration in the engine
    "SGLANG_DIFFUSION_ENGINE_ITERATION_TIMEOUT_S": _lazy_int(
        "SGLANG_DIFFUSION_ENGINE_ITERATION_TIMEOUT_S", 60
    ),
    # Logging configuration
    # If set to 0, sgl_diffusion will not configure logging
    # If set to 1, sgl_diffusion will configure logging using the default configuration
    #    or the configuration file specified by SGLANG_DIFFUSION_LOGGING_CONFIG_PATH
    "SGLANG_DIFFUSION_CONFIGURE_LOGGING": _lazy_int(
        "SGLANG_DIFFUSION_CONFIGURE_LOGGING", 1
    ),
    "SGLANG_DIFFUSION_LOGGING_CONFIG_PATH": _lazy_str(
        "SGLANG_DIFFUSION_LOGGING_CONFIG_PATH"
    ),
    # this is used for configuring the default logging level
    "SGLANG_DIFFUSION_LOGGING_LEVEL": _lazy_str(
        "SGLANG_DIFFUSION_LOGGING_LEVEL", "INFO"
    ),
    # if set, SGLANG_DIFFUSION_LOGGING_PREFIX will be prepended to all log messages
    "SGLANG_DIFFUSION_LOGGING_PREFIX": _lazy_str("SGLANG_DIFFUSION_LOGGING_PREFIX", ""),
    # Trace function calls
    # If set to 1, sgl_diffusion will trace function calls
    # Useful for debugging
    "SGLANG_DIFFUSION_TRACE_FUNCTION": _lazy_int("SGLANG_DIFFUSION_TRACE_FUNCTION", 0),
    # Path to the attention configuration file. Only used for sliding tile
    # attention for now.
    "SGLANG_DIFFUSION_ATTENTION_CONFIG": _lazy_path(
        "SGLANG_DIFFUSION_ATTENTION_CONFIG"
    ),
    # Optional override to force a specific attention backend (e.g. "aiter")
    "SGLANG_DIFFUSION_ATTENTION_BACKEND": _lazy_str(
        "SGLANG_DIFFUSION_ATTENTION_BACKEND"
    ),
    # Use dedicated multiprocess context for workers.
    # Both spawn and fork work
    "SGLANG_DIFFUSION_WORKER_MULTIPROC_METHOD": _lazy_str(
        "SGLANG_DIFFUSION_WORKER_MULTIPROC_METHOD", "fork"
    ),
    # Enables torch profiler if set. Path to the directory where torch profiler
    # traces are saved. Note that it must be an absolute path.
    "SGLANG_DIFFUSION_TORCH_PROFILER_DIR": _lazy_path(
        "SGLANG_DIFFUSION_TORCH_PROFILER_DIR"
    ),
    # If set, sgl_diffusion will run in development mode, which will enable
    # some additional endpoints for developing and debugging,
    # e.g. `/reset_prefix_cache`
    "SGLANG_DIFFUSION_SERVER_DEV_MODE": _lazy_bool("SGLANG_DIFFUSION_SERVER_DEV_MODE"),
    # If set, sgl_diffusion will enable stage logging, which will print the time
    # taken for each stage
    "SGLANG_DIFFUSION_STAGE_LOGGING": _lazy_bool("SGLANG_DIFFUSION_STAGE_LOGGING"),
    "SGLANG_DIFFUSION_VAE_CHANNELS_LAST_3D": _lazy_bool(
        "SGLANG_DIFFUSION_VAE_CHANNELS_LAST_3D", "true"
    ),
    # ================== cache-dit Env Vars ==================
    # Enable cache-dit acceleration for DiT inference
    "SGLANG_CACHE_DIT_ENABLED": _lazy_bool("SGLANG_CACHE_DIT_ENABLED"),
    # Number of first blocks to always compute (DBCache F parameter)
    "SGLANG_CACHE_DIT_FN": _lazy_int("SGLANG_CACHE_DIT_FN", 1),
    # Number of last blocks to always compute (DBCache B parameter)
    "SGLANG_CACHE_DIT_BN": _lazy_int("SGLANG_CACHE_DIT_BN", 0),
    # Warmup steps before caching (DBCache W parameter)
    "SGLANG_CACHE_DIT_WARMUP": _lazy_int("SGLANG_CACHE_DIT_WARMUP", 4),
    # Residual difference threshold (DBCache R parameter)
    "SGLANG_CACHE_DIT_RDT": _lazy_float("SGLANG_CACHE_DIT_RDT", 0.24),
    # Maximum continuous cached steps (DBCache MC parameter)
    "SGLANG_CACHE_DIT_MC": _lazy_int("SGLANG_CACHE_DIT_MC", 3),
    # Enable TaylorSeer calibrator
    "SGLANG_CACHE_DIT_TAYLORSEER": _lazy_bool("SGLANG_CACHE_DIT_TAYLORSEER", "false"),
    # TaylorSeer order (1 or 2)
    "SGLANG_CACHE_DIT_TS_ORDER": _lazy_int("SGLANG_CACHE_DIT_TS_ORDER", 1),
    # SCM preset: none, slow, medium, fast, ultra
    "SGLANG_CACHE_DIT_SCM_PRESET": _lazy_str("SGLANG_CACHE_DIT_SCM_PRESET", "none"),
    # SCM custom compute bins (e.g., "8,3,3,2,2")
    "SGLANG_CACHE_DIT_SCM_COMPUTE_BINS": _lazy_str("SGLANG_CACHE_DIT_SCM_COMPUTE_BINS"),
    # SCM custom cache bins (e.g., "1,2,2,2,3")
    "SGLANG_CACHE_DIT_SCM_CACHE_BINS": _lazy_str("SGLANG_CACHE_DIT_SCM_CACHE_BINS"),
    # SCM policy: dynamic or static
    "SGLANG_CACHE_DIT_SCM_POLICY": _lazy_str("SGLANG_CACHE_DIT_SCM_POLICY", "dynamic"),
    # model loading
    "SGLANG_USE_RUNAI_MODEL_STREAMER": _lazy_bool(
        "SGLANG_USE_RUNAI_MODEL_STREAMER", "true"
    ),
    # FlashInfer FP4 GEMM backend override for diffusion NVFP4.
    # Supported values:
    # - auto
    # - flashinfer_cudnn
    # - flashinfer_cutlass
    # - flashinfer_trtllm
    # Legacy aliases `cudnn` and `trtllm` are also accepted.
    "SGLANG_DIFFUSION_FLASHINFER_FP4_GEMM_BACKEND": _lazy_str(
        "SGLANG_DIFFUSION_FLASHINFER_FP4_GEMM_BACKEND"
    ),
    # ROCm: use AITer GroupNorm in VAE for improved performance
    "SGLANG_USE_ROCM_VAE": _lazy_bool("SGLANG_USE_ROCM_VAE"),
    # ROCm: enable cudnn.benchmark (MIOpen auto-tuning) for VAE conv layers
    "SGLANG_USE_ROCM_CUDNN_BENCHMARK": _lazy_bool("SGLANG_USE_ROCM_CUDNN_BENCHMARK"),
    # ROCm: replace CausalConv3d with temporal-unfolded batched Conv2D in VAE
    "SGLANG_USE_ROCM_VAE_CONV2D": _lazy_bool("SGLANG_USE_ROCM_VAE_CONV2D"),
    # ROCm: use BF16 compute for the Conv2D replacement (implies CONV2D=true)
    "SGLANG_USE_ROCM_VAE_CONV2D_BF16": _lazy_bool("SGLANG_USE_ROCM_VAE_CONV2D_BF16"),
}

# Add cache-dit Secondary Transformer Env Vars via programmatic generation to reduce duplication
_CACHE_DIT_SECONDARY_CONFIGS = [
    ("FN", int, "1"),
    ("BN", int, "0"),
    ("WARMUP", int, "4"),
    ("RDT", float, "0.24"),
    ("MC", int, "3"),
    ("TS_ORDER", int, "1"),
]
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `environment_variables`, and `_CACHE_DIT_SECONDARY_CONFIGS`. The code collaborates with `_lazy_str`, `_lazy_bool_any`, `_lazy_bool`, and `_lazy_path`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `environment_variables` 和 `_CACHE_DIT_SECONDARY_CONFIGS` 等名称。 代码会与 `_lazy_str`、`_lazy_bool_any`、`_lazy_bool` 和 `_lazy_path` 协同工作。

### Lines 316-326: `_create_secondary_getter` implementation / `_create_secondary_getter` 实现
```python
def _create_secondary_getter(suffix, type_func, default_val):
    primary_key = f"SGLANG_CACHE_DIT_{suffix}"
    secondary_key = f"SGLANG_CACHE_DIT_SECONDARY_{suffix}"

    def _getter():
        val = os.getenv(secondary_key)
        if val is not None:
            return type_func(val)
        return type_func(os.getenv(primary_key, str(default_val)))

    return secondary_key, _getter
```
**EN:** This block defines function `_create_secondary_getter`. It creates secondary getter. Key calls include `os.getenv`, `type_func`, and `str`. The implementation branches on conditions. Parameters such as `suffix`, `type_func`, and `default_val` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_create_secondary_getter`。 它用于创建secondary getter。 关键调用包括 `os.getenv`、`type_func` 和 `str`。 实现中包含条件分支。 本段逻辑主要由 `suffix`、`type_func` 和 `default_val` 等参数驱动。

### Lines 329-331: supporting statements / 辅助语句
```python
for suffix, type_func, default_val in _CACHE_DIT_SECONDARY_CONFIGS:
    key, getter = _create_secondary_getter(suffix, type_func, default_val)
    environment_variables[key] = getter
```
**EN:** This block gathers supporting statements at module scope. The code collaborates with `_create_secondary_getter`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 代码会与 `_create_secondary_getter` 协同工作。

### Lines 335-339: `_secondary_taylorseer_getter` implementation / `_secondary_taylorseer_getter` 实现
```python
def _secondary_taylorseer_getter():
    return get_bool_env_var(
        "SGLANG_CACHE_DIT_SECONDARY_TAYLORSEER",
        default=os.getenv("SGLANG_CACHE_DIT_TAYLORSEER", "false"),
    )
```
**EN:** This block defines function `_secondary_taylorseer_getter`. It handles secondary taylorseer getter logic. Key calls include `get_bool_env_var`, and `os.getenv`.
**CN:** 该代码块定义了函数 `_secondary_taylorseer_getter`。 它用于处理 secondary taylorseer getter 相关逻辑。 关键调用包括 `get_bool_env_var` 和 `os.getenv`。

### Lines 342-344: supporting statements / 辅助语句
```python
environment_variables["SGLANG_CACHE_DIT_SECONDARY_TAYLORSEER"] = (
    _secondary_taylorseer_getter
)
```
**EN:** This block gathers supporting statements at module scope.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。

### Lines 348-352: `__getattr__` implementation / `__getattr__` 实现
```python
def __getattr__(name: str):
    # lazy evaluation of environment variables
    if name in environment_variables:
        return environment_variables[name]()
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")
```
**EN:** This block defines function `__getattr__`. It handles getattr logic. Key calls include `AttributeError`, and `environment_variables`. The implementation branches on conditions. Parameters such as `name` drive the behavior in this section.
**CN:** 该代码块定义了函数 `__getattr__`。 它用于处理 getattr 相关逻辑。 关键调用包括 `AttributeError` 和 `environment_variables`。 实现中包含条件分支。 本段逻辑主要由 `name` 等参数驱动。

### Lines 355-356: `__dir__` implementation / `__dir__` 实现
```python
def __dir__():
    return list(environment_variables.keys())
```
**EN:** This block defines function `__dir__`. It handles dir logic. Key calls include `list`, and `environment_variables.keys`.
**CN:** 该代码块定义了函数 `__dir__`。 它用于处理 dir 相关逻辑。 关键调用包括 `list` 和 `environment_variables.keys`。

## Key Concepts / 关键概念
- `get_default_cache_root`: Top-level function that retrieves default cache root. / 顶层函数，用于获取default cache root。
- `get_default_config_root`: Top-level function that retrieves default config root. / 顶层函数，用于获取default config root。
- `maybe_convert_int`: Top-level function that handles maybe convert int logic. / 顶层函数，用于处理 maybe convert int 相关逻辑。
- `_lazy_str`: Top-level function that handles lazy str logic. / 顶层函数，用于处理 lazy str 相关逻辑。
- `_lazy_int`: Top-level function that handles lazy int logic. / 顶层函数，用于处理 lazy int 相关逻辑。
- `_lazy_float`: Top-level function that handles lazy float logic. / 顶层函数，用于处理 lazy float 相关逻辑。
- `_lazy_bool`: Top-level function that handles lazy bool logic. / 顶层函数，用于处理 lazy bool 相关逻辑。
- `_lazy_bool_any`: Top-level function that handles lazy bool any logic. / 顶层函数，用于处理 lazy bool any 相关逻辑。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `os`, `typing`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.runtime.utils.common`

- **Total lines / 总行数**: 356
