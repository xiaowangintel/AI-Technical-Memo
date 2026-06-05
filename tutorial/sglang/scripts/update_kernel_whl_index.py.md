# update_kernel_whl_index.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/update_kernel_whl_index.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Reference: https://github.com/flashinfer-ai/flashinfer/blob/v0.2.0/scripts/update_whl_index.py This python module is part of SGLang's `scripts` automation flow. / 该Python 模块用于支撑 SGLang 中的 `update_kernel_whl_index` 流程，主要负责策略校验、发布自动化。它属于 `scripts` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: File header and usage notes / 文件头与使用说明
```python
# Reference: https://github.com/flashinfer-ai/flashinfer/blob/v0.2.0/scripts/update_whl_index.py
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 3-6: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import hashlib
import pathlib
import re
```
**EN:** This block loads argparse, hashlib, pathlib, re. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, hashlib, pathlib, re。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 9-10: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
SUPPORTED_CUDA_VERSIONS = ["129", "130"]
DEFAULT_CUDA_VERSION = "130"
```
**EN:** This section defines SUPPORTED_CUDA_VERSIONS, DEFAULT_CUDA_VERSION, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 SUPPORTED_CUDA_VERSIONS, DEFAULT_CUDA_VERSION，用于把可复用的默认值集中在模块顶部。

### Lines 13-29: Defines the `check_wheel_cuda_version` routine / 定义 `check_wheel_cuda_version` 例程
```python
def check_wheel_cuda_version(path_name, target_cuda_version):
    # Skip non-CUDA backend wheels (rocm, musa, ...). Their +<backend><ver>
    # local-version tags don't match the CUDA wheel regex below, and they are
    # published by the dedicated release-rocm*/release-musa* jobs.
    if re.search(r"\+(rocm|musa)", path_name):
        return False

    # For other CUDA versions, the wheel path name will contain the cuda version suffix, e.g. sglang_kernel-0.4.0+cu130-cp310-abi3-manylinux2014_x86_64.whl
    if target_cuda_version != DEFAULT_CUDA_VERSION:
        return target_cuda_version in path_name

    # For the default CUDA version, the wheel path name will not contain any cuda version suffix, e.g. sglang_kernel-0.4.0-cp310-abi3-manylinux2014_x86_64.whl
    # So we need to check if the wheel path name contains any other cuda version suffix
    for cuda_version in SUPPORTED_CUDA_VERSIONS:
        if cuda_version != DEFAULT_CUDA_VERSION and cuda_version in path_name:
            return False
    return True
```
**EN:** This block defines `check_wheel_cuda_version`. It accepts 2 parameter(s): path_name, target_cuda_version. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `check_wheel_cuda_version`。它接收 2 个参数：path_name, target_cuda_version。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 32-48: Defines the `update_wheel_index` routine / 定义 `update_wheel_index` 例程
```python
def update_wheel_index(cuda_version=DEFAULT_CUDA_VERSION, rocm_version=None):
    index_dir = pathlib.Path(f"sgl-whl/cu{cuda_version}/sglang-kernel")
    index_dir.mkdir(exist_ok=True, parents=True)
    base_url = "https://github.com/sgl-project/whl/releases/download"

    for path in sorted(pathlib.Path("sgl-kernel/dist").glob("*.whl")):
        # Skip the wheel if mismatches the passed in cuda_version
        if not check_wheel_cuda_version(path.name, cuda_version):
            continue
        with open(path, "rb") as f:
            sha256 = hashlib.sha256(f.read()).hexdigest()
        ver = re.findall(
            r"sglang_kernel-([0-9.]+(?:\.post[0-9]+)?)(?:\+cu[0-9]+)?-", path.name
        )[0]
        full_url = f"{base_url}/v{ver}/{path.name}#sha256={sha256}"
        with (index_dir / "index.html").open("a") as f:
            f.write(f'<a href="{full_url}">{path.name}</a><br>\n')
```
**EN:** This block defines `update_wheel_index`. It accepts 2 parameter(s): cuda_version, rocm_version. Internally it creates directories, reads or writes files, touches filesystem paths.
**CN:** 该代码块定义了 `update_wheel_index`。它接收 2 个参数：cuda_version, rocm_version。内部会创建目录、读写文件、处理文件系统路径。

### Lines 51-68: Defines the `_update_non_cuda_wheel_index` routine / 定义 `_update_non_cuda_wheel_index` 例程
```python
def _update_non_cuda_wheel_index(backend, version):
    index_dir = pathlib.Path(f"sgl-whl/{backend}{version}/sglang-kernel")
    index_dir.mkdir(exist_ok=True, parents=True)
    base_url = "https://github.com/sgl-project/whl/releases/download"

    for path in sorted(pathlib.Path("sgl-kernel/dist").glob("*.whl")):
        # Skip the wheel if not for this backend
        if re.search(f"{backend}", path.name) is None:
            continue
        with open(path, "rb") as f:
            sha256 = hashlib.sha256(f.read()).hexdigest()
        ver = re.findall(
            rf"sglang_kernel-([0-9.]+(?:\.post[0-9]+)?)(?:\+{backend}[0-9]+)?-",
            path.name,
        )[0]
        full_url = f"{base_url}/v{ver}/{path.name}#sha256={sha256}"
        with (index_dir / "index.html").open("a") as f:
            f.write(f'<a href="{full_url}">{path.name}</a><br>\n')
```
**EN:** This block defines `_update_non_cuda_wheel_index`. It accepts 2 parameter(s): backend, version. Internally it creates directories, reads or writes files, touches filesystem paths.
**CN:** 该代码块定义了 `_update_non_cuda_wheel_index`。它接收 2 个参数：backend, version。内部会创建目录、读写文件、处理文件系统路径。

### Lines 71-72: Defines the `update_wheel_index_rocm` routine / 定义 `update_wheel_index_rocm` 例程
```python
def update_wheel_index_rocm(rocm_version):
    _update_non_cuda_wheel_index("rocm", rocm_version)
```
**EN:** This block defines `update_wheel_index_rocm`. It accepts 1 parameter(s): rocm_version. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `update_wheel_index_rocm`。它接收 1 个参数：rocm_version。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 75-76: Defines the `update_wheel_index_musa` routine / 定义 `update_wheel_index_musa` 例程
```python
def update_wheel_index_musa(musa_version):
    _update_non_cuda_wheel_index("musa", musa_version)
```
**EN:** This block defines `update_wheel_index_musa`. It accepts 1 parameter(s): musa_version. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `update_wheel_index_musa`。它接收 1 个参数：musa_version。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 79-90: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--cuda", type=str, default=DEFAULT_CUDA_VERSION)
    parser.add_argument("--rocm", type=str, default=None)
    parser.add_argument("--musa", type=str, default=None)
    args = parser.parse_args()
    if args.musa is not None:
        update_wheel_index_musa(args.musa)
    elif args.rocm is not None:
        update_wheel_index_rocm(args.rocm)
    else:
        update_wheel_index(args.cuda)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数。

### Lines 93-94: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `hashlib`, `pathlib`, `re`
