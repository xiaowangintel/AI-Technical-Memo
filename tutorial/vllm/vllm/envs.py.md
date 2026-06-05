# envs.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `vllm/envs.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Central registry for all supported vLLM environment variables, including parsing, validation, lazy access, caching, and compile-cache factor hashing. / vLLM 全部环境变量的中心注册表，负责解析、校验、延迟访问、缓存以及编译缓存因子哈希。

## Line-by-Line Analysis / 逐行分析
### Typed public surface for env attributes (lines 14-45)
```python
if TYPE_CHECKING:
    VLLM_HOST_IP: str = ""
    VLLM_PORT: int | None = None
    VLLM_RPC_BASE_PATH: str = tempfile.gettempdir()
    VLLM_USE_MODELSCOPE: bool = False
    VLLM_RINGBUFFER_WARNING_INTERVAL: int = 60
    VLLM_NCCL_SO_PATH: str | None = None
    LD_LIBRARY_PATH: str | None = None
    VLLM_ROCM_SLEEP_MEM_CHUNK_SIZE: int = 256
    LOCAL_RANK: int = 0
    CUDA_VISIBLE_DEVICES: str | None = None
    VLLM_ENGINE_ITERATION_TIMEOUT_S: int = 60
    VLLM_ENGINE_READY_TIMEOUT_S: int = 600
    VLLM_API_KEY: str | None = None
    VLLM_DEBUG_LOG_API_SERVER_RESPONSE: bool = False
    S3_ACCESS_KEY_ID: str | None = None
    S3_SECRET_ACCESS_KEY: str | None = None
    S3_ENDPOINT_URL: str | None = None
    VLLM_MODEL_REDIRECT_PATH: str | None = None
    VLLM_CACHE_ROOT: str = os.path.expanduser("~/.cache/vllm")
    VLLM_CONFIG_ROOT: str = os.path.expanduser("~/.config/vllm")
    VLLM_USAGE_STATS_SERVER: str = "https://stats.vllm.ai"
    VLLM_NO_USAGE_STATS: bool = False
    VLLM_DO_NOT_TRACK: bool = False
    VLLM_USAGE_SOURCE: str = "production"
    VLLM_CONFIGURE_LOGGING: bool = True
    VLLM_LOGGING_LEVEL: str = "INFO"
    VLLM_LOGGING_PREFIX: str = ""
    VLLM_LOGGING_STREAM: str = "ext://sys.stdout"
    VLLM_LOGGING_CONFIG_PATH: str | None = None
    VLLM_LOGGING_COLOR: str = "auto"
    NO_COLOR: bool = False
```
**EN:** The `TYPE_CHECKING` block is not runtime logic; it is a contract for type checkers and IDEs. It documents the module's pseudo-attributes (`envs.VLLM_PORT`, `envs.VLLM_LOGGING_LEVEL`, etc.) even though those values are actually resolved lazily via `__getattr__` later in the file.
**CN:** `TYPE_CHECKING` 代码块并不是运行时逻辑，而是给类型检查器和 IDE 的契约。它声明了模块的“伪属性”（如 `envs.VLLM_PORT`、`envs.VLLM_LOGGING_LEVEL` 等），尽管这些值在文件后面实际上是通过 `__getattr__` 延迟解析的。

### Conversion helpers and validated single-value envs (lines 278-381)
```python
def get_default_cache_root():
    return os.getenv(
        "XDG_CACHE_HOME",
        os.path.join(os.path.expanduser("~"), ".cache"),
    )


def get_default_config_root():
    return os.getenv(
        "XDG_CONFIG_HOME",
        os.path.join(os.path.expanduser("~"), ".config"),
    )


def maybe_convert_int(value: str | None) -> int | None:
    if value is None:
        return None
    return int(value)


def maybe_convert_bool(value: str | None) -> bool | None:
    if value is None:
        return None
    return bool(int(value))


def maybe_convert_json_str_or_file(value: str | None) -> dict[str, Any] | None:
    if value is None:
        return None
    if os.path.exists(value):
        with open(value) as f:
            return json.load(f)
    return json.loads(value)


def disable_compile_cache() -> bool:
    return bool(int(os.getenv("VLLM_DISABLE_COMPILE_CACHE", "0")))


def use_aot_compile() -> bool:
    from vllm.utils.torch_utils import is_torch_equal_or_newer

    default_value = (
        "1"
        if is_torch_equal_or_newer("2.10.0") and not disable_compile_cache()
        else "0"
    )

    return os.environ.get("VLLM_USE_AOT_COMPILE", default_value) == "1"


def use_mega_aot_artifact():
    from vllm.utils.torch_utils import is_torch_equal_or_newer

    default_value = (
        "1" if is_torch_equal_or_newer("2.12.0.dev") and use_aot_compile() else "0"
    )

    return os.environ.get("VLLM_USE_MEGA_AOT_ARTIFACT", default_value) == "1"


def env_with_choices(
    env_name: str,
    default: str | None,
    choices: list[str] | Callable[[], list[str]],
    case_sensitive: bool = True,
) -> Callable[[], str | None]:
    """
    Create a lambda that validates environment variable against allowed choices

    Args:
        env_name: Name of the environment variable
        default: Default value if not set (can be None)
        choices: List of valid string options or callable that returns list
        case_sensitive: Whether validation should be case sensitive

    Returns:
        Lambda function for environment_variables dict
    """

    def _get_validated_env() -> str | None:
        value = os.getenv(env_name)
        if value is None:
            return default

        # Resolve choices if it's a callable (for lazy loading)
        actual_choices = choices() if callable(choices) else choices

        if not case_sensitive:
            check_value = value.lower()
            check_choices = [choice.lower() for choice in actual_choices]
        else:
            check_value = value
            check_choices = actual_choices

        if check_value not in check_choices:
            raise ValueError(
                f"Invalid value '{value}' for {env_name}. "
                f"Valid options: {actual_choices}."
            )

        return value

    return _get_validated_env
```
**EN:** These helpers normalize common patterns: XDG-aware cache/config roots, optional int/bool/JSON parsing, compile-cache feature defaults, and a generic `env_with_choices()` validator. The validator is important because it pushes correctness to the boundary—invalid values fail fast the moment code reads the variable.
**CN:** 这些辅助函数统一了常见模式：兼容 XDG 的缓存/配置根目录、可选的 int/bool/JSON 解析、编译缓存相关默认值，以及通用的 `env_with_choices()` 校验器。这个校验器很重要，因为它把正确性前移到了边界层：一旦代码读取变量，非法值就会立刻失败。

### List/set validators and dynamic defaults (lines 384-504)
```python
def env_list_with_choices(
    env_name: str,
    default: list[str],
    choices: list[str] | Callable[[], list[str]],
    case_sensitive: bool = True,
) -> Callable[[], list[str]]:
    """
    Create a lambda that validates environment variable
    containing comma-separated values against allowed choices

    Args:
        env_name: Name of the environment variable
        default: Default list of values if not set
        choices: List of valid string options or callable that returns list
        case_sensitive: Whether validation should be case sensitive

    Returns:
        Lambda function for environment_variables
        dict that returns list of strings
    """

    def _get_validated_env_list() -> list[str]:
        value = os.getenv(env_name)
        if value is None:
            return default

        # Split comma-separated values and strip whitespace
        values = [v.strip() for v in value.split(",") if v.strip()]

        if not values:
            return default

        # Resolve choices if it's a callable (for lazy loading)
        actual_choices = choices() if callable(choices) else choices

        # Validate each value
        for val in values:
            if not case_sensitive:
                check_value = val.lower()
                check_choices = [choice.lower() for choice in actual_choices]
            else:
                check_value = val
                check_choices = actual_choices

            if check_value not in check_choices:
                raise ValueError(
                    f"Invalid value '{val}' in {env_name}. "
                    f"Valid options: {actual_choices}."
                )

        return values

    return _get_validated_env_list


def env_set_with_choices(
    env_name: str,
    default: list[str],
    choices: list[str] | Callable[[], list[str]],
    case_sensitive: bool = True,
) -> Callable[[], set[str]]:
    """
    Creates a lambda which that validates environment variable
    containing comma-separated values against allowed choices which
    returns choices as a set.
    """

    def _get_validated_env_set() -> set[str]:
        return set(env_list_with_choices(env_name, default, choices, case_sensitive)())

    return _get_validated_env_set


def get_vllm_port() -> int | None:
    """Get the port from VLLM_PORT environment variable.

    Returns:
        The port number as an integer if VLLM_PORT is set, None otherwise.

    Raises:
        ValueError: If VLLM_PORT is a URI, suggest k8s service discovery issue.
    """
    if "VLLM_PORT" not in os.environ:
        return None

    port = os.getenv("VLLM_PORT", "0")

    try:
        return int(port)
    except ValueError as err:
        from urllib3.util import parse_url

        parsed = parse_url(port)
        if parsed.scheme:
            raise ValueError(
                f"VLLM_PORT '{port}' appears to be a URI. "
                "This may be caused by a Kubernetes service discovery issue,"
                "check the warning in: https://docs.vllm.ai/en/stable/serving/env_vars.html"
            ) from None
        raise ValueError(f"VLLM_PORT '{port}' must be a valid integer") from err


def get_env_or_set_default(
    env_name: str,
    default_factory: Callable[[], str],
) -> Callable[[], str]:
    """
    Create a lambda that returns an environment variable value if set,
    or generates and sets a default value using the provided factory function.
    """

    def _get_or_set_default() -> str:
        value = os.getenv(env_name)
        if value is not None:
            return value

        default_value = default_factory()
        os.environ[env_name] = default_value
        return default_value

    return _get_or_set_default
```
**EN:** `env_list_with_choices()` and `env_set_with_choices()` extend the same validation model to comma-separated lists. `get_vllm_port()` is more opinionated: it detects when Kubernetes-style service discovery accidentally injects a URI into `VLLM_PORT` and emits a tailored error. `get_env_or_set_default()` goes one step further by synthesizing and persisting a default value back into `os.environ`.
**CN:** `env_list_with_choices()` 和 `env_set_with_choices()` 把同样的校验模型扩展到了逗号分隔列表。`get_vllm_port()` 则更“有主见”：它能识别 Kubernetes 风格服务发现把 URI 误写进 `VLLM_PORT` 的情况，并给出定制化报错。`get_env_or_set_default()` 更进一步，会生成默认值并回写到 `os.environ`。

### Registry examples: installation and runtime basics (lines 514-602)
```python
environment_variables: dict[str, Callable[[], Any]] = {
    # ================== Installation Time Env Vars ==================
    # Target device of vLLM, supporting [cuda (by default),
    # rocm, cpu]
    "VLLM_TARGET_DEVICE": lambda: os.getenv("VLLM_TARGET_DEVICE", "cuda").lower(),
    # Main CUDA version of vLLM. This follows PyTorch but can be overridden.
    "VLLM_MAIN_CUDA_VERSION": lambda: (
        os.getenv("VLLM_MAIN_CUDA_VERSION", "").lower() or "13.0"
    ),
    # Controls PyTorch float32 matmul precision mode within vLLM workers.
    # Valid options mirror torch.set_float32_matmul_precision
    "VLLM_FLOAT32_MATMUL_PRECISION": env_with_choices(
        "VLLM_FLOAT32_MATMUL_PRECISION",
        "highest",
        ["highest", "high", "medium"],
        case_sensitive=False,
    ),
    # Enable batch-invariant mode: deterministic results regardless of
    # batch composition. Requires NVIDIA GPU with compute capability >= 9.0.
    "VLLM_BATCH_INVARIANT": lambda: bool(int(os.getenv("VLLM_BATCH_INVARIANT", "0"))),
    # Use tensor descriptors for Q/K/V loads and output stores in the
    # Triton unified-attention kernel.  Enables HW 2D block reads on
    # Intel Xe2/Xe3; the non-TD branch is dead-code-eliminated at Triton
    # compile time so other platforms see no overhead.  Tri-state override:
    # unset (default) lets the `triton_attn` backend auto-select per
    # platform (currently auto-enabled on XPU only); ``1`` forces TD on;
    # ``0`` forces TD off.  Useful for A/B benchmarking the TD path.
    "VLLM_TRITON_ATTN_USE_TD": lambda: {"1": True, "0": False}.get(
        os.getenv("VLLM_TRITON_ATTN_USE_TD", "").strip()
    ),
    # Maximum number of compilation jobs to run in parallel.
    # By default this is the number of CPUs
    "MAX_JOBS": lambda: os.getenv("MAX_JOBS", None),
    # Number of threads to use for nvcc
    # By default this is 1.
    # If set, `MAX_JOBS` will be reduced to avoid oversubscribing the CPU.
    "NVCC_THREADS": lambda: os.getenv("NVCC_THREADS", None),
    # If set, vllm will use precompiled binaries (*.so)
    "VLLM_USE_PRECOMPILED": lambda: (
        os.environ.get("VLLM_USE_PRECOMPILED", "").strip().lower() in ("1", "true")
        or bool(os.environ.get("VLLM_PRECOMPILED_WHEEL_LOCATION"))
    ),
    # If set, skip adding +precompiled suffix to version string
    "VLLM_SKIP_PRECOMPILED_VERSION_SUFFIX": lambda: bool(
        int(os.environ.get("VLLM_SKIP_PRECOMPILED_VERSION_SUFFIX", "0"))
    ),
    # Used to mark that setup.py is running in a Docker build context,
    # in order to force the use of precompiled binaries.
    "VLLM_DOCKER_BUILD_CONTEXT": lambda: (
        os.environ.get("VLLM_DOCKER_BUILD_CONTEXT", "").strip().lower() in ("1", "true")
    ),
    # CMake build type
    # If not set, defaults to "Debug" or "RelWithDebInfo"
    # Available options: "Debug", "Release", "RelWithDebInfo"
    "CMAKE_BUILD_TYPE": env_with_choices(
        "CMAKE_BUILD_TYPE", None, ["Debug", "Release", "RelWithDebInfo"]
    ),
    # If set, vllm will print verbose logs during installation
    "VERBOSE": lambda: bool(int(os.getenv("VERBOSE", "0"))),
    # Root directory for vLLM configuration files
    # Defaults to `~/.config/vllm` unless `XDG_CONFIG_HOME` is set
    # Note that this not only affects how vllm finds its configuration files
    # during runtime, but also affects how vllm installs its configuration
    # files during **installation**.
    "VLLM_CONFIG_ROOT": lambda: os.path.expanduser(
        os.getenv(
            "VLLM_CONFIG_ROOT",
            os.path.join(get_default_config_root(), "vllm"),
        )
    ),
    # ================== Runtime Env Vars ==================
    # Root directory for vLLM cache files
    # Defaults to `~/.cache/vllm` unless `XDG_CACHE_HOME` is set
    "VLLM_CACHE_ROOT": lambda: os.path.expanduser(
        os.getenv(
            "VLLM_CACHE_ROOT",
            os.path.join(get_default_cache_root(), "vllm"),
        )
    ),
    # used in distributed environment to determine the ip address
    # of the current node, when the node has multiple network interfaces.
    # If you are using multi-node inference, you should set this differently
    # on each node.
    "VLLM_HOST_IP": lambda: os.getenv("VLLM_HOST_IP", ""),
    # used in distributed environment to manually set the communication port
    # Note: if VLLM_PORT is set, and some code asks for multiple ports, the
    # VLLM_PORT will be used as the first port, and the rest will be generated
    # by incrementing the VLLM_PORT value.
    "VLLM_PORT": get_vllm_port,
```
**EN:** The huge `environment_variables` dict is the real heart of the module. Each key maps to a callable, not a value, so evaluation is deferred until first access. This excerpt shows the pattern: normalize target-device strings, validate matmul precision, derive cache/config paths, and use dedicated helper functions such as `get_vllm_port()` for tricky variables.
**CN:** 庞大的 `environment_variables` 字典才是这个模块真正的核心。每个键映射到的是“可调用对象”而不是值本身，因此求值会延迟到首次访问时进行。这个片段展示了典型模式：规范化目标设备字符串、校验 matmul 精度、推导缓存/配置目录，并对复杂变量（如 `VLLM_PORT`）复用专门的辅助函数。

### Registry examples: logging controls (lines 712-749)
```python
    # Logging configuration
    # If set to 0, vllm will not configure logging
    # If set to 1, vllm will configure logging using the default configuration
    #    or the configuration file specified by VLLM_LOGGING_CONFIG_PATH
    "VLLM_CONFIGURE_LOGGING": lambda: bool(
        int(os.getenv("VLLM_CONFIGURE_LOGGING", "1"))
    ),
    "VLLM_LOGGING_CONFIG_PATH": lambda: os.getenv("VLLM_LOGGING_CONFIG_PATH"),
    # this is used for configuring the default logging level
    "VLLM_LOGGING_LEVEL": lambda: os.getenv("VLLM_LOGGING_LEVEL", "INFO").upper(),
    # this is used for configuring the default logging stream
    "VLLM_LOGGING_STREAM": lambda: os.getenv("VLLM_LOGGING_STREAM", "ext://sys.stdout"),
    # if set, VLLM_LOGGING_PREFIX will be prepended to all log messages
    "VLLM_LOGGING_PREFIX": lambda: os.getenv("VLLM_LOGGING_PREFIX", ""),
    # Controls colored logging output. Options: "auto" (default, colors when terminal),
    # "1" (always use colors), "0" (never use colors)
    "VLLM_LOGGING_COLOR": lambda: os.getenv("VLLM_LOGGING_COLOR", "auto"),
    # Standard unix flag for disabling ANSI color codes
    "NO_COLOR": lambda: os.getenv("NO_COLOR", "0") != "0",
    # If set, vllm will log stats at this interval in seconds
    # If not set, vllm will log stats every 10 seconds.
    "VLLM_LOG_STATS_INTERVAL": lambda: (
        val
        if (val := float(os.getenv("VLLM_LOG_STATS_INTERVAL", "10."))) > 0.0
        else 10.0
    ),
    # Trace function calls
    # If set to 1, vllm will trace function calls
    # Useful for debugging
    "VLLM_TRACE_FUNCTION": lambda: int(os.getenv("VLLM_TRACE_FUNCTION", "0")),
    # Whether to use the FlashInfer top-k / top-p sampler on CUDA. Enabled
    # by default when the hardware supports it — set to 0 to opt out
    # explicitly, which forces the PyTorch-native (Triton for bs>=8) path.
    "VLLM_USE_FLASHINFER_SAMPLER": lambda: (
        bool(int(os.environ["VLLM_USE_FLASHINFER_SAMPLER"]))
        if "VLLM_USE_FLASHINFER_SAMPLER" in os.environ
        else True
    ),
```
**EN:** Logging-related env vars are registered the same way as compute knobs, which means log level, stream, prefix, color policy, and tracing flags are all centrally typed and discoverable. That centralization is what lets `vllm.logger` rebuild config from `envs` without duplicating parsing rules.
**CN:** 日志相关环境变量与计算开关使用同样的注册机制，因此日志级别、输出流、前缀、颜色策略和 tracing 开关都具备统一的类型与可发现性。正因为有这一层中心化注册，`vllm.logger` 才能直接从 `envs` 重建配置，而无需重复实现解析规则。

### Registry examples: media-fetch controls (lines 825-860)
```python
    # Timeout for fetching images when serving multimodal models
    # Default is 5 seconds
    "VLLM_IMAGE_FETCH_TIMEOUT": lambda: int(os.getenv("VLLM_IMAGE_FETCH_TIMEOUT", "5")),
    # Timeout for fetching videos when serving multimodal models
    # Default is 30 seconds
    "VLLM_VIDEO_FETCH_TIMEOUT": lambda: int(
        os.getenv("VLLM_VIDEO_FETCH_TIMEOUT", "30")
    ),
    # Timeout for fetching audio when serving multimodal models
    # Default is 10 seconds
    "VLLM_AUDIO_FETCH_TIMEOUT": lambda: int(
        os.getenv("VLLM_AUDIO_FETCH_TIMEOUT", "10")
    ),
    # Directory for caching media downloads (images, video, audio fetched
    # from URLs during inference). Empty string disables caching.
    "VLLM_MEDIA_CACHE": lambda: os.getenv("VLLM_MEDIA_CACHE", ""),
    # Maximum cache size in MB. When exceeded, least-recently-used entries
    # are evicted. Default is 5120 (5 GB).
    "VLLM_MEDIA_CACHE_MAX_SIZE_MB": lambda: int(
        os.getenv("VLLM_MEDIA_CACHE_MAX_SIZE_MB", "5120")
    ),
    # Time-to-live in hours for cached media files. Entries older than this
    # are evicted regardless of cache size. Default is 24 hours.
    "VLLM_MEDIA_CACHE_TTL_HOURS": lambda: float(
        os.getenv("VLLM_MEDIA_CACHE_TTL_HOURS", "24")
    ),
    # Maximum number of retries for fetching media (images, audio, video)
    # from URLs. Each retry quadruples the timeout. Default is 3.
    "VLLM_MEDIA_FETCH_MAX_RETRIES": lambda: int(
        os.getenv("VLLM_MEDIA_FETCH_MAX_RETRIES", "3")
    ),
    # Whether to allow HTTP redirects when fetching from media URLs.
    # Default to True
    "VLLM_MEDIA_URL_ALLOW_REDIRECTS": lambda: bool(
        int(os.getenv("VLLM_MEDIA_URL_ALLOW_REDIRECTS", "1"))
    ),
```
**EN:** These entries drive the behavior in `vllm.connections` and multimodal loaders: per-media timeout, cache path, cache size, TTL, retry count, redirect policy, and worker-thread count. Because these are parsed centrally, downstream code can assume strongly typed ints/bools/floats instead of re-parsing strings.
**CN:** 这些条目直接驱动 `vllm.connections` 与多模态加载器的行为：包括不同媒体类型的超时、缓存路径、缓存大小、TTL、重试次数、重定向策略和工作线程数。由于它们在这里被统一解析，下游代码可以直接拿到强类型的 int/bool/float，而不用再次解析字符串。

### Registry examples: generated names and advanced runtime flags (lines 1642-1779)
```python
    # Name of the shared memory buffer used for object storage.
    # Only effective when mm_config.mm_processor_cache_type == "shm".
    # Automatically generates a unique UUID-based name per process tree
    # if not explicitly set.
    "VLLM_OBJECT_STORAGE_SHM_BUFFER_NAME": get_env_or_set_default(
        "VLLM_OBJECT_STORAGE_SHM_BUFFER_NAME",
        lambda: f"VLLM_OBJECT_STORAGE_SHM_BUFFER_{uuid.uuid4().hex}",
    ),
    # The size in MB of the buffers (NVL and RDMA) used by DeepEP
    "VLLM_DEEPEP_BUFFER_SIZE_MB": lambda: int(
        os.getenv("VLLM_DEEPEP_BUFFER_SIZE_MB", "1024")
    ),
    # Force DeepEP to use intranode kernel for inter-node communication in
    # high throughput mode. This is useful archive higher prefill throughput
    # on system supports multi-node nvlink (e.g GB200).
    "VLLM_DEEPEP_HIGH_THROUGHPUT_FORCE_INTRA_NODE": lambda: bool(
        int(os.getenv("VLLM_DEEPEP_HIGH_THROUGHPUT_FORCE_INTRA_NODE", "0"))
    ),
    # Allow DeepEP to use MNNVL (multi-node nvlink) for internode_ll kernel,
    # turn this for better latency on GB200 like system
    "VLLM_DEEPEP_LOW_LATENCY_USE_MNNVL": lambda: bool(
        int(os.getenv("VLLM_DEEPEP_LOW_LATENCY_USE_MNNVL", "0"))
    ),
    # The number of SMs/CUs to allocate for communication kernels when
    # running DBO; the rest will be allocated to compute.
    # Default: 20 on CUDA (SMs), 64 on ROCm (CUs).
    "VLLM_DBO_COMM_SMS": lambda: int(
        os.getenv(
            "VLLM_DBO_COMM_SMS",
            "64"
            if hasattr(__import__("torch").version, "hip")
            and __import__("torch").version.hip is not None
            else "20",
        )
    ),
    # Enable max_autotune & coordinate_descent_tuning in inductor_config
    # to compile static shapes passed from compile_sizes in compilation_config
    # If set to 1, enable max_autotune; By default, this is enabled (1)
    "VLLM_ENABLE_INDUCTOR_MAX_AUTOTUNE": lambda: bool(
        int(os.getenv("VLLM_ENABLE_INDUCTOR_MAX_AUTOTUNE", "1"))
    ),
    # If set to 1, enable coordinate_descent_tuning;
    # By default, this is enabled (1)
    "VLLM_ENABLE_INDUCTOR_COORDINATE_DESCENT_TUNING": lambda: bool(
        int(os.getenv("VLLM_ENABLE_INDUCTOR_COORDINATE_DESCENT_TUNING", "1"))
    ),
    # Flag to enable NCCL symmetric memory allocation and registration
    "VLLM_USE_NCCL_SYMM_MEM": lambda: bool(
        int(os.getenv("VLLM_USE_NCCL_SYMM_MEM", "0"))
    ),
    # NCCL header path
    "VLLM_NCCL_INCLUDE_PATH": lambda: os.environ.get("VLLM_NCCL_INCLUDE_PATH", None),
    # Flag to enable FBGemm kernels on model execution
    "VLLM_USE_FBGEMM": lambda: bool(int(os.getenv("VLLM_USE_FBGEMM", "0"))),
    # GC debug config
    # - VLLM_GC_DEBUG=0: disable GC debugger
    # - VLLM_GC_DEBUG=1: enable GC debugger with gc.collect elpased times
    # - VLLM_GC_DEBUG='{"top_objects":5}': enable GC debugger with
    #                                      top 5 collected objects
    "VLLM_GC_DEBUG": lambda: os.getenv("VLLM_GC_DEBUG", ""),
    # Debug workspace allocations.
    # logging of workspace resize operations.
    "VLLM_DEBUG_WORKSPACE": lambda: bool(int(os.getenv("VLLM_DEBUG_WORKSPACE", "0"))),
    # Disables parallel execution of shared_experts via separate cuda stream
    "VLLM_DISABLE_SHARED_EXPERTS_STREAM": lambda: bool(
        int(os.getenv("VLLM_DISABLE_SHARED_EXPERTS_STREAM", "0"))
    ),
    # Limits when we run shared_experts in a separate stream.
    # We found out that for large batch sizes, the separate stream
    # execution is not beneficial (most likely because of the input clone)
    # TODO(alexm-redhat): Tune to be more dynamic based on GPU type
    "VLLM_SHARED_EXPERTS_STREAM_TOKEN_THRESHOLD": lambda: int(
        int(os.getenv("VLLM_SHARED_EXPERTS_STREAM_TOKEN_THRESHOLD", 256))
    ),
    # Token-count cutoff for multi-stream overlap of the attention input
    # GEMM with auxiliary GEMMs (e.g. fused_wqa_wkv overlapped with indexer
    # weights / kv-score projections in DeepSeek-V4). At or below this many
    # tokens the FP8 main GEMM has idle SMs to share with the bf16 aux GEMMs
    # and overlap is a 5-45% win; above it the FP8 GEMM saturates the device
    # and the cross-stream sync becomes pure overhead. Set to 0 to disable
    # the multi-stream path entirely. See #PR 41526 for the empirical result
    # for the default value of 1024 tokens.
    "VLLM_MULTI_STREAM_GEMM_TOKEN_THRESHOLD": lambda: int(
        os.getenv("VLLM_MULTI_STREAM_GEMM_TOKEN_THRESHOLD", "1024")
    ),
    # Format for saving torch.compile cache artifacts
    # - "binary": saves as binary file
    #     Safe for multiple vllm serve processes accessing the same torch compile cache.
    # - "unpacked": saves as directory structure (for inspection/debugging)
    #     NOT multiprocess safe - race conditions may occur with multiple processes.
    #     Allows viewing and setting breakpoints in Inductor's code output files.
    "VLLM_COMPILE_CACHE_SAVE_FORMAT": env_with_choices(
        "VLLM_COMPILE_CACHE_SAVE_FORMAT", "binary", ["binary", "unpacked"]
    ),
    # Flag to control the v2 model runner. If unset, use config defaults.
    "VLLM_USE_V2_MODEL_RUNNER": lambda: maybe_convert_bool(
        os.getenv("VLLM_USE_V2_MODEL_RUNNER", None)
    ),
    # Log model inspection after loading.
    # If enabled, logs a transformers-style hierarchical view of the model
    # with quantization methods and attention backends.
    "VLLM_LOG_MODEL_INSPECTION": lambda: bool(
        int(os.getenv("VLLM_LOG_MODEL_INSPECTION", "0"))
    ),
    # Debug logging for --enable-mfu-metrics
    "VLLM_DEBUG_MFU_METRICS": lambda: bool(
        int(os.getenv("VLLM_DEBUG_MFU_METRICS", "0"))
    ),
    # Disable using pytorch's pin memory for CPU offloading.
    "VLLM_WEIGHT_OFFLOADING_DISABLE_PIN_MEMORY": lambda: bool(
        int(os.getenv("VLLM_WEIGHT_OFFLOADING_DISABLE_PIN_MEMORY", "0"))
    ),
    # Disable using UVA (Unified Virtual Addressing) for CPU offloading.
    "VLLM_WEIGHT_OFFLOADING_DISABLE_UVA": lambda: bool(
        int(os.getenv("VLLM_WEIGHT_OFFLOADING_DISABLE_UVA", "0"))
    ),
    # Disable logging of vLLM logo at server startup time.
    "VLLM_DISABLE_LOG_LOGO": lambda: bool(int(os.getenv("VLLM_DISABLE_LOG_LOGO", "0"))),
    # Disable PDL for LoRA, as enabling PDL with LoRA on SM100 causes
    # Triton compilation to fail.
    "VLLM_LORA_DISABLE_PDL": lambda: bool(int(os.getenv("VLLM_LORA_DISABLE_PDL", "0"))),
    # Enable CUDA compatibility mode for datacenter GPUs with older
    # driver versions than the CUDA toolkit major version of vLLM.
    "VLLM_ENABLE_CUDA_COMPATIBILITY": lambda: (
        os.environ.get("VLLM_ENABLE_CUDA_COMPATIBILITY", "0").strip().lower()
        in ("1", "true")
    ),
    # Path to the CUDA compatibility libraries when CUDA compatibility is enabled.
    "VLLM_CUDA_COMPATIBILITY_PATH": lambda: os.environ.get(
        "VLLM_CUDA_COMPATIBILITY_PATH", None
    ),
    # Skip model name validation in OpenAI API requests.
    # When set to 1, any model name will be accepted in the 'model' field
    # of API requests. This is useful for proxy/gateway scenarios where
    # the actual model is served but different names may be used in requests.
    "VLLM_SKIP_MODEL_NAME_VALIDATION": lambda: (
        os.getenv("VLLM_SKIP_MODEL_NAME_VALIDATION", "0").strip().lower()
        in ("1", "true")
```
**EN:** This later section shows that the registry is not just static defaults. `VLLM_OBJECT_STORAGE_SHM_BUFFER_NAME` can auto-generate a per-process-tree shared-memory name, and later entries cover compile tuning, debug behavior, CUDA compatibility, and API compatibility flags. In other words, the file is both a schema and a policy engine.
**CN:** 后半段说明这个注册表不仅仅是静态默认值。`VLLM_OBJECT_STORAGE_SHM_BUFFER_NAME` 可以自动生成进程树级别的共享内存名，后续条目还覆盖编译调优、调试行为、CUDA 兼容性和 API 兼容性开关。换句话说，这个文件既是配置模式定义，也是策略引擎。

### Lazy access, caching, and validation (lines 1826-1895)
```python
def __getattr__(name: str):
    """
    Gets environment variables lazily.

    NOTE: After enable_envs_cache() invocation (which triggered after service
    initialization), all environment variables will be cached.
    """
    if name in environment_variables:
        return environment_variables[name]()
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")


def _is_envs_cache_enabled() -> bool:
    """Checked if __getattr__ is wrapped with functools.cache"""
    global __getattr__
    return hasattr(__getattr__, "cache_clear")


def enable_envs_cache() -> None:
    """
    Enables caching of environment variables. This is useful for performance
    reasons, as it avoids the need to re-evaluate environment variables on
    every call.

    NOTE: Currently, it's invoked after service initialization to reduce
    runtime overhead. This also means that environment variables should NOT
    be updated after the service is initialized.
    """
    if _is_envs_cache_enabled():
        # Avoid wrapping functools.cache multiple times
        return
    # Tag __getattr__ with functools.cache
    global __getattr__
    __getattr__ = functools.cache(__getattr__)

    # Cache all environment variables
    for key in environment_variables:
        __getattr__(key)


def disable_envs_cache() -> None:
    """
    Resets the environment variables cache. It could be used to isolate environments
    between unit tests.
    """
    global __getattr__
    # If __getattr__ is wrapped by functions.cache, unwrap the caching layer.
    if _is_envs_cache_enabled():
        assert hasattr(__getattr__, "__wrapped__")
        __getattr__ = __getattr__.__wrapped__


def __dir__():
    return list(environment_variables.keys())


def is_set(name: str):
    """Check if an environment variable is explicitly set."""
    if name in environment_variables:
        return name in os.environ
    raise AttributeError(f"module {__name__!r} has no attribute {name!r}")


def validate_environ(hard_fail: bool) -> None:
    for env in os.environ:
        if env.startswith("VLLM_") and env not in environment_variables:
            if hard_fail:
                raise ValueError(f"Unknown vLLM environment variable detected: {env}")
            else:
                logger.warning("Unknown vLLM environment variable detected: %s", env)
```
**EN:** The module fakes constant-like attributes through `__getattr__`: reading `envs.VLLM_LOGGING_LEVEL` just executes the callable stored in `environment_variables`. `enable_envs_cache()` then wraps `__getattr__` with `functools.cache` and eagerly populates all values after initialization, while `validate_environ()` warns or fails on unknown `VLLM_` variables to catch typos early.
**CN:** 模块通过 `__getattr__` 模拟出“常量式”属性：读取 `envs.VLLM_LOGGING_LEVEL` 本质上是在执行 `environment_variables` 中保存的可调用对象。初始化完成后，`enable_envs_cache()` 会用 `functools.cache` 包装 `__getattr__` 并预热全部值；`validate_environ()` 则会对未知的 `VLLM_` 变量发出警告或直接失败，以便尽早发现拼写错误。

### Compile cache factor hashing (lines 1898-2014)
```python
def compile_factors() -> dict[str, object]:
    """Return env vars used for torch.compile cache keys.

    Start with every known vLLM env var; drop entries in `ignored_factors`;
    hash everything else. This keeps the cache key aligned across workers."""

    ignored_factors: set[str] = {
        "MAX_JOBS",
        "VLLM_RPC_BASE_PATH",
        "VLLM_USE_MODELSCOPE",
        "VLLM_RINGBUFFER_WARNING_INTERVAL",
        "VLLM_DEBUG_DUMP_PATH",
        "VLLM_PORT",
        "VLLM_CACHE_ROOT",
        "LD_LIBRARY_PATH",
        "VLLM_SERVER_DEV_MODE",
        "VLLM_DP_MASTER_IP",
        "VLLM_DP_MASTER_PORT",
        "VLLM_NIXL_SIDE_CHANNEL_HOST",
        "VLLM_RANDOMIZE_DP_DUMMY_INPUTS",
        "VLLM_CI_USE_S3",
        "VLLM_MODEL_REDIRECT_PATH",
        "VLLM_HOST_IP",
        "VLLM_FORCE_AOT_LOAD",
        "S3_ACCESS_KEY_ID",
        "S3_SECRET_ACCESS_KEY",
        "S3_ENDPOINT_URL",
        "VLLM_USAGE_STATS_SERVER",
        "VLLM_NO_USAGE_STATS",
        "VLLM_DO_NOT_TRACK",
        "VLLM_LOGGING_LEVEL",
        "VLLM_LOGGING_PREFIX",
        "VLLM_LOGGING_STREAM",
        "VLLM_LOGGING_CONFIG_PATH",
        "VLLM_LOGGING_COLOR",
        "VLLM_LOG_STATS_INTERVAL",
        "VLLM_DEBUG_LOG_API_SERVER_RESPONSE",
        "VLLM_TUNED_CONFIG_FOLDER",
        "VLLM_ENGINE_ITERATION_TIMEOUT_S",
        "VLLM_HTTP_TIMEOUT_KEEP_ALIVE",
        "VLLM_EXECUTE_MODEL_TIMEOUT_SECONDS",
        "VLLM_KEEP_ALIVE_ON_ENGINE_DEATH",
        "VLLM_IMAGE_FETCH_TIMEOUT",
        "VLLM_VIDEO_FETCH_TIMEOUT",
        "VLLM_AUDIO_FETCH_TIMEOUT",
        "VLLM_MEDIA_CACHE",
        "VLLM_MEDIA_CACHE_MAX_SIZE_MB",
        "VLLM_MEDIA_CACHE_TTL_HOURS",
        "VLLM_MEDIA_FETCH_MAX_RETRIES",
        "VLLM_MEDIA_URL_ALLOW_REDIRECTS",
        "VLLM_MEDIA_LOADING_THREAD_COUNT",
        "VLLM_MAX_AUDIO_CLIP_FILESIZE_MB",
        "VLLM_VIDEO_LOADER_BACKEND",
        "VLLM_MEDIA_CONNECTOR",
        "VLLM_OBJECT_STORAGE_SHM_BUFFER_NAME",
        "VLLM_ASSETS_CACHE",
        "VLLM_ASSETS_CACHE_MODEL_CLEAN",
        "VLLM_WORKER_MULTIPROC_METHOD",
        "VLLM_ENABLE_V1_MULTIPROCESSING",
        "VLLM_V1_OUTPUT_PROC_CHUNK_SIZE",
        "VLLM_CPU_KVCACHE_SPACE",
        "VLLM_CPU_MOE_PREPACK",
        "VLLM_ZENTORCH_WEIGHT_PREPACK",
        "VLLM_TEST_FORCE_LOAD_FORMAT",
        "VLLM_ENABLE_CUDA_COMPATIBILITY",
        "VLLM_CUDA_COMPATIBILITY_PATH",
        "VLLM_SKIP_MODEL_NAME_VALIDATION",
        "LOCAL_RANK",
        "CUDA_VISIBLE_DEVICES",
        "NO_COLOR",
    }

    from vllm.config.utils import normalize_value

    factors: dict[str, object] = {}
    for factor, getter in environment_variables.items():
        if factor in ignored_factors:
            continue

        try:
            raw = getter()
        except Exception as exc:  # pragma: no cover - defensive logging
            logger.warning(
                "Skipping environment variable %s while hashing compile factors: %s",
                factor,
                exc,
            )
            continue

        factors[factor] = normalize_value(raw)

    ray_noset_env_vars = [
        # Refer to
        # https://github.com/ray-project/ray/blob/c584b1ea97b00793d1def71eaf81537d70efba42/python/ray/_private/accelerators/nvidia_gpu.py#L11
        # https://github.com/ray-project/ray/blob/c584b1ea97b00793d1def71eaf81537d70efba42/python/ray/_private/accelerators/amd_gpu.py#L11
        # https://github.com/ray-project/ray/blob/b97d21dab233c2bd8ed7db749a82a1e594222b5c/python/ray/_private/accelerators/amd_gpu.py#L10
        # https://github.com/ray-project/ray/blob/c584b1ea97b00793d1def71eaf81537d70efba42/python/ray/_private/accelerators/npu.py#L12
        # https://github.com/ray-project/ray/blob/c584b1ea97b00793d1def71eaf81537d70efba42/python/ray/_private/accelerators/hpu.py#L12
        # https://github.com/ray-project/ray/blob/c584b1ea97b00793d1def71eaf81537d70efba42/python/ray/_private/accelerators/neuron.py#L14
        # https://github.com/ray-project/ray/blob/c584b1ea97b00793d1def71eaf81537d70efba42/python/ray/_private/accelerators/tpu.py#L38
        # https://github.com/ray-project/ray/blob/c584b1ea97b00793d1def71eaf81537d70efba42/python/ray/_private/accelerators/intel_gpu.py#L10
        # https://github.com/ray-project/ray/blob/c584b1ea97b00793d1def71eaf81537d70efba42/python/ray/_private/accelerators/rbln.py#L10
        "RAY_EXPERIMENTAL_NOSET_CUDA_VISIBLE_DEVICES",
        "RAY_EXPERIMENTAL_NOSET_ROCR_VISIBLE_DEVICES",
        "RAY_EXPERIMENTAL_NOSET_HIP_VISIBLE_DEVICES",
        "RAY_EXPERIMENTAL_NOSET_ASCEND_RT_VISIBLE_DEVICES",
        "RAY_EXPERIMENTAL_NOSET_HABANA_VISIBLE_MODULES",
        "RAY_EXPERIMENTAL_NOSET_NEURON_RT_VISIBLE_CORES",
        "RAY_EXPERIMENTAL_NOSET_TPU_VISIBLE_CHIPS",
        "RAY_EXPERIMENTAL_NOSET_ONEAPI_DEVICE_SELECTOR",
        "RAY_EXPERIMENTAL_NOSET_RBLN_RT_VISIBLE_DEVICES",
    ]

    for var in ray_noset_env_vars:
        factors[var] = normalize_value(os.getenv(var))

    return factors
```
**EN:** `compile_factors()` computes the subset of environment state that should influence `torch.compile` cache keys. It explicitly ignores operational settings like ports or log formatting, normalizes retained values, and also incorporates Ray's `NOSET_*` accelerator variables so distributed workers agree on the same compilation identity.
**CN:** `compile_factors()` 负责计算哪些环境状态应该进入 `torch.compile` 的缓存键。它会显式忽略端口、日志格式等运维层配置，对保留值做标准化处理，并把 Ray 的 `NOSET_*` 加速器变量也纳入其中，从而保证分布式 worker 对编译身份达成一致。

## Key Concepts / 关键概念
- EN: The registry stores callables instead of values, enabling lazy evaluation and environment-dependent defaults.
- CN: 注册表存储的是可调用对象而不是具体值，因此可以实现延迟求值和依赖环境的默认值。
- EN: Validation is embedded into getters, so invalid configuration is rejected where the value first becomes semantic.
- CN: 校验逻辑嵌入在 getter 中，因此配置一旦“变得有语义”就会被检查并拒绝非法值。
- EN: `compile_factors()` separates behavioral knobs from operational noise to keep compile-cache keys stable but meaningful.
- CN: `compile_factors()` 把真正影响行为的开关与运维噪声分离开，保证编译缓存键既稳定又有意义。

## Dependencies / 依赖关系
- EN: Depends on stdlib modules such as `os`, `json`, `tempfile`, `uuid`, and `logging` for parsing and defaults.
- CN: 依赖 `os`、`json`、`tempfile`、`uuid`、`logging` 等标准库模块完成解析和默认值构造。
- EN: Consults `vllm.utils.torch_utils` and `vllm.config.utils.normalize_value` for Torch-version-aware defaults and cache-key normalization.
- CN: 通过 `vllm.utils.torch_utils` 与 `vllm.config.utils.normalize_value` 获取 Torch 版本感知默认值和缓存键标准化能力。
- EN: Downstream modules such as `vllm.logger`, `vllm.connections`, and many runtime subsystems consume these exported pseudo-attributes directly.
- CN: 下游模块如 `vllm.logger`、`vllm.connections` 以及大量运行时子系统都会直接消费这里导出的伪属性。
