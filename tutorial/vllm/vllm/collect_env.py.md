# collect_env.py — Code Analysis / 代码分析
## Source / 来源
- **File**: `vllm/collect_env.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Collects a support-friendly snapshot of runtime, package, GPU/XPU, and vLLM build information. / 收集运行时、包、GPU/XPU 与 vLLM 构建信息，便于排障与问题报告。

## Line-by-Line Analysis / 逐行分析
### System environment schema and package filters (lines 29-110)
```python
# System Environment Information
SystemEnv = namedtuple(
    "SystemEnv",
    [
        "torch_version",
        "is_debug_build",
        "cuda_compiled_version",
        "gcc_version",
        "clang_version",
        "cmake_version",
        "os",
        "libc_version",
        "python_version",
        "python_platform",
        "is_cuda_available",
        "cuda_runtime_version",
        "cuda_module_loading",
        "nvidia_driver_version",
        "nvidia_gpu_models",
        "cudnn_version",
        "xpu_available",
        "xpu_runtime_version",
        "intel_graphics_compiler_version",
        "intel_gpu_models",
        "oneapi_compiler_version",
        "level_zero_loader_version",
        "level_zero_driver_version",
        "oneccl_version",
        "libigdgmm_version",
        "vllm_xpu_kernels_version",
        "sycl_version",
        "pip_version",  # 'pip' or 'pip3'
        "pip_packages",
        "conda_packages",
        "hip_compiled_version",
        "hip_runtime_version",
        "miopen_runtime_version",
        "caching_allocator_config",
        "is_xnnpack_available",
        "cpu_info",
        "rocm_version",  # vllm specific field
        "vllm_version",  # vllm specific field
        "vllm_build_flags",  # vllm specific field
        "gpu_topo",  # vllm specific field
        "env_vars",
    ],
)

DEFAULT_CONDA_PATTERNS = {
    "torch",
    "numpy",
    "cudatoolkit",
    "soumith",
    "mkl",
    "magma",
    "triton",
    "optree",
    "nccl",
    "transformers",
    "zmq",
    "nvidia",
    "pynvml",
    "flashinfer-python",
    "helion",
}

DEFAULT_PIP_PATTERNS = {
    "torch",
    "numpy",
    "mypy",
    "flake8",
    "triton",
    "optree",
    "onnx",
    "nccl",
    "transformers",
    "zmq",
    "nvidia",
    "pynvml",
    "flashinfer-python",
    "helion",
}
```
**EN:** The `SystemEnv` namedtuple defines one stable record shape for every probe result, including CUDA, ROCm, XPU, CPU, package listings, and vLLM-specific fields such as build flags and GPU topology. The two pattern sets limit `conda list` and `pip list` output to packages that matter for ML runtime debugging instead of dumping the full environment.
**CN:** `SystemEnv` 命名元组为所有探测结果定义了稳定的数据结构，覆盖 CUDA、ROCm、XPU、CPU、包列表，以及 vLLM 特有的构建标志和 GPU 拓扑。两个模式集合用于过滤 `conda list` 与 `pip list` 输出，只保留与机器学习运行时排障相关的包，而不是输出整个环境。

### Subprocess helpers (lines 113-157)
```python
def run(command):
    """Return (return-code, stdout, stderr)."""
    shell = True if type(command) is str else False
    try:
        p = subprocess.Popen(
            command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=shell
        )
        raw_output, raw_err = p.communicate()
        rc = p.returncode
        if get_platform() == "win32":
            enc = "oem"
        else:
            enc = locale.getpreferredencoding()
        output = raw_output.decode(enc)
        if command == "nvidia-smi topo -m":
            # don't remove the leading whitespace of `nvidia-smi topo -m`
            #   because they are meaningful
            output = output.rstrip()
        else:
            output = output.strip()
        err = raw_err.decode(enc)
        return rc, output, err.strip()

    except FileNotFoundError:
        cmd_str = command if isinstance(command, str) else command[0]
        return 127, "", f"Command not found: {cmd_str}"


def run_and_read_all(run_lambda, command):
    """Run command using run_lambda; reads and returns entire output if rc is 0."""
    rc, out, _ = run_lambda(command)
    if rc != 0:
        return None
    return out


def run_and_parse_first_match(run_lambda, command, regex):
    """Run command using run_lambda, returns the first regex match if it exists."""
    rc, out, _ = run_lambda(command)
    if rc != 0:
        return None
    match = re.search(regex, out)
    if match is None:
        return None
    return match.group(1)
```
**EN:** `run()` centralizes subprocess execution, decoding, whitespace cleanup, and the special `nvidia-smi topo -m` formatting rule. `run_and_read_all()` and `run_and_parse_first_match()` build a small probing DSL on top of that, so later functions only express a command plus a parsing regex.
**CN:** `run()` 统一处理子进程执行、输出解码、空白清理，以及 `nvidia-smi topo -m` 的特殊格式保留规则。`run_and_read_all()` 与 `run_and_parse_first_match()` 在其上构建了一个轻量探测 DSL，使后续函数只需描述命令和解析正则。

### GPU and accelerator probing (lines 199-244)
```python
def get_gpu_info(run_lambda):
    if get_platform() == "darwin" or (
        TORCH_AVAILABLE
        and hasattr(torch.version, "hip")
        and torch.version.hip is not None
    ):
        if TORCH_AVAILABLE and torch.cuda.is_available():
            if torch.version.hip is not None:
                prop = torch.cuda.get_device_properties(0)
                if hasattr(prop, "gcnArchName"):
                    gcnArch = " ({})".format(prop.gcnArchName)
                else:
                    gcnArch = "NoGCNArchNameOnOldPyTorch"
            else:
                gcnArch = ""
            return torch.cuda.get_device_name(None) + gcnArch
        return None
    smi = get_nvidia_smi()
    uuid_regex = re.compile(r" \(UUID: .+?\)")
    rc, out, _ = run_lambda(smi + " -L")
    if rc != 0:
        return None
    # Anonymize GPUs by removing their UUID
    return re.sub(uuid_regex, "", out)


def get_running_cuda_version(run_lambda):
    return run_and_parse_first_match(run_lambda, "nvcc --version", r"release .+ V(.*)")


def get_cudnn_version(run_lambda):
    """Return a list of libcudnn.so; it's hard to tell which one is being used."""
    if get_platform() == "win32":
        system_root = os.environ.get("SYSTEMROOT", "C:\\Windows")
        cuda_path = os.environ.get("CUDA_PATH", "%CUDA_PATH%")
        where_cmd = os.path.join(system_root, "System32", "where")
        cudnn_cmd = '{} /R "{}\\bin" cudnn*.dll'.format(where_cmd, cuda_path)
    elif get_platform() == "darwin":
        # CUDA libraries and drivers can be found in /usr/local/cuda/. See
        # https://docs.nvidia.com/cuda/cuda-installation-guide-mac-os-x/index.html#install
        # https://docs.nvidia.com/deeplearning/sdk/cudnn-install/index.html#installmac
        # Use CUDNN_LIBRARY when cudnn library is installed elsewhere.
        cudnn_cmd = "ls /usr/local/cuda/lib/libcudnn*"
    else:
        cudnn_cmd = 'ldconfig -p | grep libcudnn | rev | cut -d" " -f1 | rev'
    rc, out, _ = run_lambda(cudnn_cmd)
```
**EN:** `get_gpu_info()` prefers PyTorch runtime APIs on macOS or HIP builds, but falls back to `nvidia-smi -L` on standard CUDA systems and strips GPU UUIDs for privacy. `get_cudnn_version()` uses platform-specific discovery commands because cuDNN may live in different library locations on Windows, macOS, and Linux.
**CN:** `get_gpu_info()` 在 macOS 或 HIP 构建上优先使用 PyTorch 运行时 API，在常规 CUDA 系统上回退到 `nvidia-smi -L`，并移除 GPU UUID 以保护隐私。`get_cudnn_version()` 则按平台选择不同命令，因为 cuDNN 在 Windows、macOS、Linux 上的库路径并不一致。

### Host, Python, and package inventory (lines 535-678)
```python
def get_cpu_info(run_lambda):
    rc, out, err = 0, "", ""
    if get_platform() == "linux":
        rc, out, err = run_lambda("lscpu")
    elif get_platform() == "win32":
        rc, out, err = run_lambda(
            "wmic cpu get Name,Manufacturer,Family,Architecture,ProcessorType,DeviceID, \
        CurrentClockSpeed,MaxClockSpeed,L2CacheSize,L2CacheSpeed,Revision /VALUE"
        )
    elif get_platform() == "darwin":
        rc, out, err = run_lambda("sysctl -n machdep.cpu.brand_string")
    cpu_info = "None"
    if rc == 0:
        cpu_info = out
    else:
        cpu_info = err
    return cpu_info


def get_platform():
    if sys.platform.startswith("linux"):
        return "linux"
    elif sys.platform.startswith("win32"):
        return "win32"
    elif sys.platform.startswith("cygwin"):
        return "cygwin"
    elif sys.platform.startswith("darwin"):
        return "darwin"
    else:
        return sys.platform


def get_mac_version(run_lambda):
    return run_and_parse_first_match(run_lambda, "sw_vers -productVersion", r"(.*)")


def get_windows_version(run_lambda):
    system_root = os.environ.get("SYSTEMROOT", "C:\\Windows")
    wmic_cmd = os.path.join(system_root, "System32", "Wbem", "wmic")
    findstr_cmd = os.path.join(system_root, "System32", "findstr")
    return run_and_read_all(
        run_lambda, "{} os get Caption | {} /v Caption".format(wmic_cmd, findstr_cmd)
    )


def get_lsb_version(run_lambda):
    return run_and_parse_first_match(
        run_lambda, "lsb_release -a", r"Description:\t(.*)"
    )


def check_release_file(run_lambda):
    return run_and_parse_first_match(
        run_lambda, "cat /etc/*-release", r'PRETTY_NAME="(.*)"'
    )


def get_os(run_lambda):
    from platform import machine

    platform = get_platform()

    if platform == "win32" or platform == "cygwin":
        return get_windows_version(run_lambda)

    if platform == "darwin":
        version = get_mac_version(run_lambda)
        if version is None:
            return None
        return "macOS {} ({})".format(version, machine())

    if platform == "linux":
        # Ubuntu/Debian based
        desc = get_lsb_version(run_lambda)
        if desc is not None:
            return "{} ({})".format(desc, machine())

        # Try reading /etc/*-release
        desc = check_release_file(run_lambda)
        if desc is not None:
            return "{} ({})".format(desc, machine())

        return "{} ({})".format(platform, machine())

    # Unknown platform
    return platform


def get_python_platform():
    import platform

    return platform.platform()


def get_libc_version():
    import platform

    if get_platform() != "linux":
        return "N/A"
    return "-".join(platform.libc_ver())


def is_uv_venv():
    if os.environ.get("UV"):
        return True
    pyvenv_cfg_path = os.path.join(sys.prefix, "pyvenv.cfg")
    if os.path.exists(pyvenv_cfg_path):
        with open(pyvenv_cfg_path, "r") as f:
            return any(line.startswith("uv = ") for line in f)
    return False


def get_pip_packages(run_lambda, patterns=None):
    """Return `pip list` output. Note: will also find conda-installed pytorch and numpy packages."""
    if patterns is None:
        patterns = DEFAULT_PIP_PATTERNS

    def run_with_pip():
        try:
            import importlib.util

            pip_spec = importlib.util.find_spec("pip")
            pip_available = pip_spec is not None
        except ImportError:
            pip_available = False

        if pip_available:
            cmd = [sys.executable, "-mpip", "list", "--format=freeze"]
        elif is_uv_venv():
            print("uv is set")
            cmd = ["uv", "pip", "list", "--format=freeze"]
        else:
            raise RuntimeError(
                "Could not collect pip list output (pip or uv module not available)"
            )

        out = run_and_read_all(run_lambda, cmd)
        return "\n".join(
            line for line in out.splitlines() if any(name in line for name in patterns)
        )

    pip_version = "pip3" if sys.version[0] == "3" else "pip"
    out = run_with_pip()
    return pip_version, out
```
**EN:** This block normalizes OS detection (`lsb_release`, `/etc/*-release`, `sw_vers`, `wmic`), CPU reporting (`lscpu` or platform equivalents), Python platform strings, and package collection. The `is_uv_venv()` + `run_with_pip()` logic is noteworthy because it supports both standard `pip` and `uv pip`, which matters in modern Python environments.
**CN:** 这部分统一了操作系统识别（`lsb_release`、`/etc/*-release`、`sw_vers`、`wmic`）、CPU 信息采集（`lscpu` 或平台等价命令）、Python 平台字符串以及包清单收集。`is_uv_venv()` 与 `run_with_pip()` 的组合很关键，因为它同时支持标准 `pip` 和 `uv pip`，适配了新的 Python 环境管理方式。

### Environment filtering and structured aggregation (lines 704-817)
```python
def get_env_vars():
    env_vars = ""
    secret_terms = ("secret", "token", "api", "access", "password")
    report_prefix = (
        "TORCH",
        "NCCL",
        "PYTORCH",
        "CUDA",
        "CUBLAS",
        "CUDNN",
        "OMP_",
        "MKL_",
        "NVIDIA",
        "ZE_",
        "ONEAPI_",
        "SYCL_",
        "NEOReadDebugKeys",
        "IGC_",
        "CCL_",
        "I_MPI_",
    )
    for k, v in os.environ.items():
        if any(term in k.lower() for term in secret_terms):
            continue
        if k in environment_variables:
            env_vars = env_vars + "{}={}".format(k, v) + "\n"
        if k.startswith(report_prefix):
            env_vars = env_vars + "{}={}".format(k, v) + "\n"

    return env_vars


def get_env_info():
    run_lambda = run
    pip_version, pip_list_output = get_pip_packages(run_lambda)

    if TORCH_AVAILABLE:
        version_str = torch.__version__
        debug_mode_str = str(torch.version.debug)
        cuda_available_str = str(torch.cuda.is_available())
        cuda_version_str = torch.version.cuda
        if (
            not hasattr(torch.version, "hip") or torch.version.hip is None
        ):  # cuda version
            hip_compiled_version = hip_runtime_version = miopen_runtime_version = "N/A"
        else:  # HIP version

            def get_version_or_na(cfg, prefix):
                _lst = [s.rsplit(None, 1)[-1] for s in cfg if prefix in s]
                return _lst[0] if _lst else "N/A"

            cfg = torch._C._show_config().split("\n")
            hip_runtime_version = get_version_or_na(cfg, "HIP Runtime")
            miopen_runtime_version = get_version_or_na(cfg, "MIOpen")
            cuda_version_str = "N/A"
            hip_compiled_version = torch.version.hip
    else:
        version_str = debug_mode_str = cuda_available_str = cuda_version_str = "N/A"
        hip_compiled_version = hip_runtime_version = miopen_runtime_version = "N/A"

    sys_version = sys.version.replace("\n", " ")

    conda_packages = get_conda_packages(run_lambda)

    rocm_version = get_rocm_version(run_lambda)
    vllm_version = get_vllm_version()
    vllm_build_flags = summarize_vllm_build_flags()
    gpu_topo = get_gpu_topo(run_lambda)

    return SystemEnv(
        torch_version=version_str,
        is_debug_build=debug_mode_str,
        python_version="{} ({}-bit runtime)".format(
            sys_version, sys.maxsize.bit_length() + 1
        ),
        python_platform=get_python_platform(),
        is_cuda_available=cuda_available_str,
        cuda_compiled_version=cuda_version_str,
        cuda_runtime_version=get_running_cuda_version(run_lambda),
        cuda_module_loading=get_cuda_module_loading_config(),
        nvidia_gpu_models=get_gpu_info(run_lambda),
        nvidia_driver_version=get_nvidia_driver_version(run_lambda),
        cudnn_version=get_cudnn_version(run_lambda),
        xpu_available=str(get_xpu_available()),
        xpu_runtime_version=get_xpu_runtime_version(),
        intel_graphics_compiler_version=get_intel_graphics_compiler_version(run_lambda),
        intel_gpu_models=get_intel_gpu_models(),
        oneapi_compiler_version=get_oneapi_compiler_version(run_lambda),
        level_zero_loader_version=get_level_zero_loader_version(run_lambda),
        level_zero_driver_version=get_level_zero_driver_version(run_lambda),
        oneccl_version=get_oneapi_ccl_version(run_lambda),
        libigdgmm_version=get_libigdgmm_version(run_lambda),
        vllm_xpu_kernels_version=get_vllm_xpu_kernels_version(run_lambda),
        sycl_version=get_sycl_version(run_lambda),
        hip_compiled_version=hip_compiled_version,
        hip_runtime_version=hip_runtime_version,
        miopen_runtime_version=miopen_runtime_version,
        pip_version=pip_version,
        pip_packages=pip_list_output,
        conda_packages=conda_packages,
        os=get_os(run_lambda),
        libc_version=get_libc_version(),
        gcc_version=get_gcc_version(run_lambda),
        clang_version=get_clang_version(run_lambda),
        cmake_version=get_cmake_version(run_lambda),
        caching_allocator_config=get_cachingallocator_config(),
        is_xnnpack_available=is_xnnpack_available(),
        cpu_info=get_cpu_info(run_lambda),
        rocm_version=rocm_version,
        vllm_version=vllm_version,
        vllm_build_flags=vllm_build_flags,
        gpu_topo=gpu_topo,
        env_vars=get_env_vars(),
    )
```
**EN:** `get_env_vars()` reports vLLM and accelerator-related environment variables but skips keys whose names look secret (`token`, `password`, `access`, etc.). `get_env_info()` is the orchestration layer: it reads Torch build/runtime state, package versions, CPU/GPU topology, and vLLM-specific metadata, then returns one populated `SystemEnv` object.
**CN:** `get_env_vars()` 会报告 vLLM 与加速器相关环境变量，但会跳过名称疑似敏感信息的键（如 `token`、`password`、`access` 等）。`get_env_info()` 是总控层：它汇总 Torch 的构建/运行时状态、包版本、CPU/GPU 拓扑和 vLLM 专属元数据，最终返回一个完整的 `SystemEnv` 对象。

### Pretty formatting and CLI entrypoint (lines 881-1056)
```python
def pretty_str(envinfo):
    def replace_nones(dct, replacement="Could not collect"):
        for key in dct.keys():
            if dct[key] is not None:
                continue
            dct[key] = replacement
        return dct

    def replace_bools(dct, true="Yes", false="No"):
        for key in dct.keys():
            if dct[key] is True:
                dct[key] = true
            elif dct[key] is False:
                dct[key] = false
        return dct

    def prepend(text, tag="[prepend]"):
        lines = text.split("\n")
        updated_lines = [tag + line for line in lines]
        return "\n".join(updated_lines)

    def replace_if_empty(text, replacement="No relevant packages"):
        if text is not None and len(text) == 0:
            return replacement
        return text

    def maybe_start_on_next_line(string):
        # If `string` is multiline, prepend a \n to it.
        if string is not None and len(string.split("\n")) > 1:
            return "\n{}\n".format(string)
        return string

    mutable_dict = envinfo._asdict()

    # If nvidia_gpu_models is multiline, start on the next line
    mutable_dict["nvidia_gpu_models"] = maybe_start_on_next_line(
        envinfo.nvidia_gpu_models
    )

    # If the machine doesn't have CUDA, report some fields as 'No CUDA'
    dynamic_cuda_fields = [
        "cuda_runtime_version",
        "nvidia_gpu_models",
        "nvidia_driver_version",
    ]
    all_cuda_fields = dynamic_cuda_fields + ["cudnn_version"]
    all_dynamic_cuda_fields_missing = all(
        mutable_dict[field] is None for field in dynamic_cuda_fields
    )
    if (
        TORCH_AVAILABLE
        and not torch.cuda.is_available()
        and all_dynamic_cuda_fields_missing
    ):
        for field in all_cuda_fields:
            mutable_dict[field] = "No CUDA"
        if envinfo.cuda_compiled_version is None:
            mutable_dict["cuda_compiled_version"] = "None"

    # If the machine doesn't have XPU, report XPU fields as 'No XPU'
    dynamic_xpu_fields = [
        "intel_graphics_compiler_version",
        "intel_gpu_models",
        "level_zero_loader_version",
        "level_zero_driver_version",
        "oneccl_version",
        "libigdgmm_version",
        "vllm_xpu_kernels_version",
    ]
    all_xpu_fields = dynamic_xpu_fields + [
        "oneapi_compiler_version",
        "sycl_version",
    ]
    all_dynamic_xpu_fields_missing = all(
        mutable_dict[field] is None for field in dynamic_xpu_fields
    )
    xpu_available = mutable_dict.get("xpu_available") == "True"
    if not xpu_available and all_dynamic_xpu_fields_missing:
        for field in all_xpu_fields:
            mutable_dict[field] = "No XPU"
    if envinfo.xpu_runtime_version is None or envinfo.xpu_runtime_version == "N/A":
        mutable_dict["xpu_runtime_version"] = "N/A"

    # If intel_gpu_models is multiline, start on the next line
    mutable_dict["intel_gpu_models"] = maybe_start_on_next_line(
        mutable_dict.get("intel_gpu_models")
    )

    # Replace True with Yes, False with No
    mutable_dict = replace_bools(mutable_dict)

    # Replace all None objects with 'Could not collect'
    mutable_dict = replace_nones(mutable_dict)

    # If either of these are '', replace with 'No relevant packages'
    mutable_dict["pip_packages"] = replace_if_empty(mutable_dict["pip_packages"])
    mutable_dict["conda_packages"] = replace_if_empty(mutable_dict["conda_packages"])

    # Tag conda and pip packages with a prefix
    # If they were previously None, they'll show up as ie '[conda] Could not collect'
    if mutable_dict["pip_packages"]:
        mutable_dict["pip_packages"] = prepend(
            mutable_dict["pip_packages"], "[{}] ".format(envinfo.pip_version)
        )
    if mutable_dict["conda_packages"]:
        mutable_dict["conda_packages"] = prepend(
            mutable_dict["conda_packages"], "[conda] "
        )
    mutable_dict["cpu_info"] = envinfo.cpu_info

    CUDA_FMT = """
==============================
       CUDA / GPU Info
==============================
Is CUDA available            : {is_cuda_available}
CUDA runtime version         : {cuda_runtime_version}
CUDA_MODULE_LOADING set to   : {cuda_module_loading}
GPU models and configuration : {nvidia_gpu_models}
Nvidia driver version        : {nvidia_driver_version}
cuDNN version                : {cudnn_version}
HIP runtime version          : {hip_runtime_version}
MIOpen runtime version       : {miopen_runtime_version}
Is XNNPACK available         : {is_xnnpack_available}
""".strip()

    XPU_FMT = """
==============================
      Intel XPU / GPU Info
==============================
Is XPU available             : {xpu_available}
XPU runtime version          : {xpu_runtime_version}
Intel GPU models             : {intel_gpu_models}

--Compile time--
oneAPI compiler version      : {oneapi_compiler_version}
SYCL compiler build          : {sycl_version}
oneCCL version               : {oneccl_version}

--Runtime--
Intel Graphics Compiler (IGC): {intel_graphics_compiler_version}
Intel GMM (libigdgmm)        : {libigdgmm_version}
Level Zero loader version    : {level_zero_loader_version}
Level Zero driver version    : {level_zero_driver_version}
vLLM XPU kernels version     : {vllm_xpu_kernels_version}
""".strip()

    invalid_vers = {"N/A", "Could not collect", "None"}
    sections = []

    if (
        mutable_dict.get("is_cuda_available") in ("True", "Yes")
        or mutable_dict.get("cuda_compiled_version") not in invalid_vers
    ):
        sections.append(CUDA_FMT)

    if (
        mutable_dict.get("xpu_available") in ("True", "Yes")
        or mutable_dict.get("xpu_runtime_version") not in invalid_vers
    ):
        sections.append(XPU_FMT)

    mutable_dict["gpu_info"] = (
        ("\n\n".join(sections) + "\n").format(**mutable_dict) if sections else ""
    )

    return env_info_fmt.format(**mutable_dict)


def get_pretty_env_info():
    return pretty_str(get_env_info())


def main():
    print("Collecting environment information...")
    output = get_pretty_env_info()
    print(output)
```
**EN:** `pretty_str()` converts missing values into readable placeholders, expands multiline GPU/package fields, conditionally inserts CUDA/XPU sections, and prefixes pip/conda lines so readers know where each package came from. `main()` then prints the final report and, on Linux, also surfaces the newest Torch crash minidump path if one exists.
**CN:** `pretty_str()` 会把缺失值转成可读占位符、处理多行 GPU/包字段、按条件插入 CUDA/XPU 小节，并给 pip/conda 条目加前缀，便于读者判断来源。`main()` 随后打印最终报告；在 Linux 上，如果存在 Torch 崩溃转储，还会额外提示最新的 minidump 路径。

## Key Concepts / 关键概念
- EN: Structured probing: most functions return plain strings or `None`, and `get_env_info()` only materializes them into one record at the end.
- CN: 结构化探测：多数函数只返回普通字符串或 `None`，最后由 `get_env_info()` 统一组装成一条记录。
- EN: Cross-platform fallback chains are deliberate, because environment collection must still work when a preferred tool like `nvidia-smi` or `lsb_release` is missing.
- CN: 跨平台回退链是有意设计的，因为即使 `nvidia-smi` 或 `lsb_release` 等首选工具缺失，环境采集也应继续工作。
- EN: Secret-aware reporting is built into environment export rather than left to callers.
- CN: 环境变量导出时内置了敏感信息过滤，而不是把这件事留给调用方。

## Dependencies / 依赖关系
- EN: Optional runtime dependency on PyTorch (`torch`) and regex parsing via `regex`.
- CN: 可选依赖 PyTorch（`torch`），并使用 `regex` 做文本解析。
- EN: Heavy use of system tools such as `nvidia-smi`, `nvcc`, `hipcc`, `icpx`, `conda`, `pip`, `lscpu`, and `ldconfig`.
- CN: 大量依赖系统工具，如 `nvidia-smi`、`nvcc`、`hipcc`、`icpx`、`conda`、`pip`、`lscpu`、`ldconfig`。
- EN: Imports `environment_variables` from `vllm.envs` so the report can include only recognized vLLM environment knobs.
- CN: 从 `vllm.envs` 导入 `environment_variables`，以便报告只输出被 vLLM 识别的环境开关。
