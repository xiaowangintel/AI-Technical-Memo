# rocm_multiarch_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torch/_inductor/rocm_multiarch_utils.py`
- **Repository / 仓库**: `pytorch/pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: This module implements TorchInductor support logic. It exposes functions such as `get_rocm_compiler`, `get_rocm_bundler`, `get_rocm_target_archs`, `_sanitize_llvm_ir_for_rocm`, `compile_llvm_ir_to_code_object`, `create_multiarch_bundle`, and `...+1`. Module note: ROCm Multi-Architecture Support Utilities
- **用途（中文）**: 该模块实现 TorchInductor 的支撑逻辑。同时提供 `get_rocm_compiler`、`get_rocm_bundler`、`get_rocm_target_archs`、`_sanitize_llvm_ir_for_rocm`、`compile_llvm_ir_to_code_object`、`create_multiarch_bundle`、`另有1项` 等函数。模块文档字符串给出了额外背景说明。

## Line-by-Line Analysis / 逐行分析
### Lines 1-20 / 第 1-20 行
````python
"""
ROCm Multi-Architecture Support Utilities
Compile LLVM IR to multi-arch bundles that HIP can load automatically.
"""

import logging
import os
import re
import subprocess

import torch
from torch.utils.cpp_extension import _join_rocm_home, ROCM_HOME


log = logging.getLogger(__name__)


def get_rocm_compiler() -> str:
    """
    Get path to ROCm's clang compiler.
````
- **EN**: Imports dependencies such as `logging`, `os`, `re`, `subprocess`, `torch`, and `torch.utils.cpp_extension` for the logic in this range. Introduces function `get_rocm_compiler`. Initializes or updates values such as `log`.
- **CN**: 这里导入了 `logging`、`os`、`re`、`subprocess`、`torch`、`torch.utils.cpp_extension` 等依赖，为后续逻辑提供基础能力。这里定义了函数`get_rocm_compiler`。初始化或更新了 `log` 等值。

### Lines 21-40 / 第 21-40 行
````python
    Uses PyTorch's ROCM_HOME detection.

    Returns:
        Path to clang compiler

    Raises:
        RuntimeError: If ROCm is not found
    """
    if ROCM_HOME is None:
        raise RuntimeError(
            "ROCm installation not found. "
            "PyTorch was not built with ROCm support or ROCM_HOME is not set."
        )

    # ROCm's clang is at <ROCM_HOME>/llvm/bin/clang
    clang_path = _join_rocm_home("llvm", "bin", "clang")

    if not os.path.exists(clang_path):
        raise RuntimeError(
            f"ROCm clang not found at {clang_path}. ROCM_HOME is set to {ROCM_HOME}"
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Returns`, `Raises`, `RuntimeError`, and `clang_path`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Returns`、`Raises`、`RuntimeError`、`clang_path` 等值。

### Lines 41-60 / 第 41-60 行
````python
        )

    return clang_path


def get_rocm_bundler() -> str:
    """
    Get path to clang-offload-bundler.
    Uses PyTorch's ROCM_HOME detection.

    Returns:
        Path to bundler

    Raises:
        RuntimeError: If bundler is not found
    """
    if ROCM_HOME is None:
        raise RuntimeError(
            "ROCm installation not found. "
            "PyTorch was not built with ROCm support or ROCM_HOME is not set."
````
- **EN**: Introduces function `get_rocm_bundler`. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `Returns`, `Raises`, and `RuntimeError`.
- **CN**: 这里定义了函数`get_rocm_bundler`。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `Returns`、`Raises`、`RuntimeError` 等值。

### Lines 61-80 / 第 61-80 行
````python
        )

    # Bundler is at <ROCM_HOME>/llvm/bin/clang-offload-bundler
    bundler_path = _join_rocm_home("llvm", "bin", "clang-offload-bundler")

    if not os.path.exists(bundler_path):
        raise RuntimeError(
            f"clang-offload-bundler not found at {bundler_path}. "
            f"ROCM_HOME is set to {ROCM_HOME}"
        )

    return bundler_path


def get_rocm_target_archs() -> list[str]:
    env_archs = os.environ.get("PYTORCH_ROCM_ARCH", "").strip()
    if env_archs:
        archs = [arch.strip() for arch in env_archs.replace(";", ",").split(",")]
        archs = [arch for arch in archs if arch]
        if archs:
````
- **EN**: Introduces function `get_rocm_target_archs`. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`get_rocm_target_archs`。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 81-100 / 第 81-100 行
````python
            # Ensure current device arch is included
            if torch.cuda.is_available():
                for dev_idx in range(torch.cuda.device_count()):
                    current_arch = torch.cuda.get_device_properties(
                        dev_idx
                    ).gcnArchName.split(":")[0]
                    if current_arch not in archs:
                        archs.append(current_arch)
            return archs

    try:
        from torch._inductor import config

        if hasattr(config, "rocm") and hasattr(config.rocm, "target_archs"):
            archs = config.rocm.target_archs
            if archs:
                return archs

    except Exception:
        pass
````
- **EN**: Imports dependencies such as `torch._inductor` for the logic in this range. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里导入了 `torch._inductor` 等依赖，为后续逻辑提供基础能力。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 101-120 / 第 101-120 行
````python

    return torch.cuda.get_arch_list()


def _sanitize_llvm_ir_for_rocm(llvm_ir_path: str) -> str:
    """
    Sanitize LLVM IR to be compatible with ROCm's clang.

    Triton's LLVM (upstream) may emit attributes and metadata that ROCm's
    older clang does not yet support. Only strips attributes confirmed to
    cause parse errors — preserves all others to maintain correct codegen.

    Currently strips:
        - nocreateundeforpoison: function attribute (upstream LLVM, not in ROCm)
        - dwarfAddressSpace: debug metadata field (upstream LLVM, not in ROCm)

    Returns:
        Path to sanitized .ll file, or original path if no changes needed.
    """
    with open(llvm_ir_path) as f:
````
- **EN**: Introduces function `_sanitize_llvm_ir_for_rocm`. Touches the filesystem to load, validate, or store compiler artifacts. References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Handles device-specific behavior, capabilities, or dispatch decisions.
- **CN**: 这里定义了函数`_sanitize_llvm_ir_for_rocm`。这一段会访问文件系统，用于加载、校验或保存编译产物。这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。处理与设备相关的行为、能力信息或派发决策。

### Lines 121-140 / 第 121-140 行
````python
        content = f.read()

    sanitized = content
    sanitized = re.sub(r"\bnocreateundeforpoison\b\s*", "", sanitized)
    sanitized = re.sub(r",\s*dwarfAddressSpace:\s*\d+", "", sanitized)

    if sanitized == content:
        return llvm_ir_path

    sanitized_path = llvm_ir_path + ".sanitized.ll"
    with open(sanitized_path, "w") as f:
        f.write(sanitized)

    log.debug(
        "Sanitized LLVM IR for ROCm clang compatibility: %s -> %s",
        llvm_ir_path,
        sanitized_path,
    )
    return sanitized_path

````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `content`, `sanitized`, and `sanitized_path`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `content`、`sanitized`、`sanitized_path` 等值。

### Lines 141-160 / 第 141-160 行
````python

def compile_llvm_ir_to_code_object(
    llvm_ir_path: str, output_path: str, target_arch: str
) -> bool:
    """
    Compile unbundled LLVM IR to a single-arch code object.

    Args:
        llvm_ir_path: Path to .ll file
        output_path: Where to write .hsaco file
        target_arch: Target architecture (e.g., 'gfx90a')

    Returns:
        True if successful
    """
    if not os.path.exists(llvm_ir_path):
        return False

    os.makedirs(os.path.dirname(output_path), exist_ok=True)

````
- **EN**: Introduces function `compile_llvm_ir_to_code_object`. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`compile_llvm_ir_to_code_object`。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 161-180 / 第 161-180 行
````python
    try:
        clang = get_rocm_compiler()
    except RuntimeError:
        return False

    # Sanitize LLVM IR to remove attributes unsupported by ROCm's clang
    llvm_ir_path = _sanitize_llvm_ir_for_rocm(llvm_ir_path)

    # Using clang and not hipcc since we are not compiling source code
    # Instead we use the LLVM IR (.ll) provided by triton
    cmd = [
        clang,
        "-target",
        "amdgcn-amd-amdhsa",
        f"-mcpu={target_arch}",
        llvm_ir_path,
        "-o",
        output_path,
    ]

````
- **EN**: References Triton-specific behavior, indicating that this logic participates in GPU kernel generation or execution. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`, `clang`, `llvm_ir_path`, and `cmd`.
- **CN**: 这一段涉及 Triton 相关行为，说明它参与 GPU 内核生成或执行。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `try`、`clang`、`llvm_ir_path`、`cmd` 等值。

### Lines 181-200 / 第 181-200 行
````python
    try:
        subprocess.run(cmd, capture_output=True, text=True, check=True)

        if not os.path.exists(output_path):
            return False

        return True

    except subprocess.CalledProcessError:
        return False


def create_multiarch_bundle(code_objects: dict, output_bundle_path: str) -> bool:
    """
    Bundle multiple architecture code objects into a single multi-arch bundle.

    Uses clang-offload-bundler to create a fat binary that HIP runtime can load.
    The runtime automatically selects the correct architecture at load time.

    Args:
````
- **EN**: Introduces function `create_multiarch_bundle`. Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这里定义了函数`create_multiarch_bundle`。这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 201-220 / 第 201-220 行
````python
        code_objects: Dict mapping architecture to code object path
        output_bundle_path: Path for output bundle

    Returns:
        True if successful
    """
    if not code_objects:
        return False

    os.makedirs(os.path.dirname(output_bundle_path), exist_ok=True)

    try:
        bundler = get_rocm_bundler()
    except RuntimeError:
        return False

    # Build targets and inputs lists for clang-offload-bundler
    targets = ["host-x86_64-unknown-linux-gnu"]

    # We include a dummy host entry to satisfy the bundler format
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `code_objects`, `output_bundle_path`, `Returns`, `try`, `bundler`, and `targets`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `code_objects`、`output_bundle_path`、`Returns`、`try`、`bundler`、`targets` 等值。

### Lines 221-240 / 第 221-240 行
````python
    inputs = ["/dev/null"]

    for arch, path in sorted(code_objects.items()):
        if not os.path.exists(path):
            continue
        # hipv4 = HIP version 4 code object format
        # amdgcn-amd-amdhsa = target triple for ROCm/HSA runtime
        # arch = specific GPU (gfx90a, gfx942, etc.)
        targets.append(f"hipv4-amdgcn-amd-amdhsa--{arch}")
        inputs.append(path)

    if len(inputs) == 1:  # Only host, no device code
        return False

    cmd = [
        bundler,
        "--type=o",
        # CRITICAL: HIP runtime expects 4096-byte alignment for loading bundles
        # Without this, hipModuleLoadData gives segmentation fault
        "-bundle-align=4096",  # CRITICAL: Required by HIP runtime!
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 241-260 / 第 241-260 行
````python
        f"--targets={','.join(targets)}",
    ]

    for input_file in inputs:
        cmd.append(f"--input={input_file}")

    cmd.append(f"--output={output_bundle_path}")

    try:
        subprocess.run(cmd, capture_output=True, text=True, check=True)

        if not os.path.exists(output_bundle_path):
            return False

        return True

    except subprocess.CalledProcessError:
        return False


````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `try` 等值。

### Lines 261-280 / 第 261-280 行
````python
def compile_multiarch_bundle_from_llvm_ir(
    llvm_ir_path: str, output_bundle_path: str, target_archs: list[str] | None = None
) -> bool:
    """
    Complete workflow: LLVM IR → multiple code objects → bundle.

    This is the main entry point for multi-arch compilation.

    Args:
        llvm_ir_path: Path to .ll file
        output_bundle_path: Where to write bundle
        target_archs: Optional list of architectures

    Returns:
        True if successful
    """
    if target_archs is None:
        # Get architectures from environment variable or config
        target_archs = get_rocm_target_archs()

````
- **EN**: Introduces function `compile_multiarch_bundle_from_llvm_ir`. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `llvm_ir_path`, `Args`, `output_bundle_path`, `target_archs`, and `Returns`.
- **CN**: 这里定义了函数`compile_multiarch_bundle_from_llvm_ir`。包含分支、循环或上下文管理等控制流。初始化或更新了 `llvm_ir_path`、`Args`、`output_bundle_path`、`target_archs`、`Returns` 等值。

### Lines 281-300 / 第 281-300 行
````python
    # Step 1: Compile LLVM IR to code object for each architecture
    code_objects = {}
    temp_dir = os.path.dirname(output_bundle_path)
    kernel_name = os.path.splitext(os.path.basename(llvm_ir_path))[0]

    for arch in target_archs:
        # Create temporary single-architecture code object
        # Format: kernel_name_gfx90a.co, kernel_name_gfx942.co, etc.
        co_path = os.path.join(temp_dir, f"{kernel_name}_{arch}.co")

        # Compile with clang backend: LLVM IR → GPU machine code
        if compile_llvm_ir_to_code_object(llvm_ir_path, co_path, arch):
            code_objects[arch] = co_path

    if not code_objects:
        return False

    # Step 2: Bundle all code objects together
    # Uses clang-offload-bundler to create fat binary
    success = create_multiarch_bundle(code_objects, output_bundle_path)
````
- **EN**: Touches the filesystem to load, validate, or store compiler artifacts. Handles device-specific behavior, capabilities, or dispatch decisions. Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management.
- **CN**: 这一段会访问文件系统，用于加载、校验或保存编译产物。处理与设备相关的行为、能力信息或派发决策。包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。

### Lines 301-310 / 第 301-310 行
````python

    # Step 3: Clean up temporary single-arch code objects
    # The bundle contains all the code, so intermediates are no longer needed
    for co_path in code_objects.values():
        try:
            os.remove(co_path)
        except Exception:
            pass

    return success
````
- **EN**: Includes returns or checks that define the contract of this code path. Contains control-flow that branches, iterates, or scopes resource management. Initializes or updates values such as `try`. This range continues the implementation of function `compile_multiarch_bundle_from_llvm_ir`.
- **CN**: 包含返回语句或断言/异常检查，用来限定这条代码路径的契约。包含分支、循环或上下文管理等控制流。初始化或更新了 `try` 等值。这一段延续了函数`compile_multiarch_bundle_from_llvm_ir` 的具体实现。

## Key Concepts / 关键概念
- **EN**: Triton kernel integration  
  **CN**: Triton 内核集成
- **EN**: Device capability modeling  
  **CN**: 设备能力建模
- **EN**: Kernel construction or dispatch  
  **CN**: 内核构建或派发
- **EN**: Primary functions: `get_rocm_compiler`, `get_rocm_bundler`, `get_rocm_target_archs`, `_sanitize_llvm_ir_for_rocm`, `compile_llvm_ir_to_code_object`, `create_multiarch_bundle`, and `...+1`  
  **CN**: 主要函数：`get_rocm_compiler`、`get_rocm_bundler`、`get_rocm_target_archs`、`_sanitize_llvm_ir_for_rocm`、`compile_llvm_ir_to_code_object`、`create_multiarch_bundle`、`另有1项`

## Dependencies / 依赖关系
- **Standard library / 标准库**: `logging`, `os`, `re`, `subprocess`
- **Third-party / 第三方**: None / 无
- **PyTorch/Internal / PyTorch 内部**: `torch`, `torch.utils.cpp_extension`, `torch._inductor`
