# update_deepgemm_whl_index.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/update_deepgemm_whl_index.py`
- **Repository**: sgl-project/sglang
- **Purpose**: Generates a PEP 503 simple index for sgl-deep-gemm wheels under sgl-whl/cu<version>/sgl-deep-gemm/index.html. This python module is part of SGLang's `scripts` automation flow. / 该Python 模块用于支撑 SGLang 中的 `update_deepgemm_whl_index` 流程，主要负责依赖安装、发布自动化。它属于 `scripts` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: File header and usage notes / 文件头与使用说明
```python
# Generates a PEP 503 simple index for sgl-deep-gemm wheels under
# sgl-whl/cu<version>/sgl-deep-gemm/index.html. Mirrors the layout used by
# update_kernel_whl_index.py so consumers can `pip install
# sgl-deep-gemm --extra-index-url https://...whl/cu129`.
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 6-9: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import hashlib
import pathlib
import re
```
**EN:** This block loads argparse, hashlib, pathlib, re. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, hashlib, pathlib, re。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 11-11: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
SUPPORTED_CUDA_VERSIONS = ["129", "130"]
```
**EN:** This section defines SUPPORTED_CUDA_VERSIONS, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 SUPPORTED_CUDA_VERSIONS，用于把可复用的默认值集中在模块顶部。

### Lines 14-31: Defines the `update_wheel_index` routine / 定义 `update_wheel_index` 例程
```python
def update_wheel_index(cuda_version, wheel_dir):
    index_dir = pathlib.Path(f"sgl-whl/cu{cuda_version}/sgl-deep-gemm")
    index_dir.mkdir(exist_ok=True, parents=True)
    base_url = "https://github.com/sgl-project/whl/releases/download"

    suffix = f"+cu{cuda_version}"
    for path in sorted(pathlib.Path(wheel_dir).glob("*.whl")):
        if suffix not in path.name:
            continue
        with open(path, "rb") as f:
            sha256 = hashlib.sha256(f.read()).hexdigest()
        match = re.match(r"sgl_deep_gemm-([0-9][^-+]*)(?:\+cu[0-9]+)?-", path.name)
        if not match:
            continue
        ver = match.group(1)
        full_url = f"{base_url}/v{ver}/{path.name}#sha256={sha256}"
        with (index_dir / "index.html").open("a") as f:
            f.write(f'<a href="{full_url}">{path.name}</a><br>\n')
```
**EN:** This block defines `update_wheel_index`. It accepts 2 parameter(s): cuda_version, wheel_dir. Internally it creates directories, reads or writes files, touches filesystem paths.
**CN:** 该代码块定义了 `update_wheel_index`。它接收 2 个参数：cuda_version, wheel_dir。内部会创建目录、读写文件、处理文件系统路径。

### Lines 34-41: Defines the `main` routine / 定义 `main` 例程
```python
def main():
    parser = argparse.ArgumentParser()
    parser.add_argument(
        "--cuda", type=str, required=True, choices=SUPPORTED_CUDA_VERSIONS
    )
    parser.add_argument("--wheel-dir", type=str, default="dist")
    args = parser.parse_args()
    update_wheel_index(args.cuda, args.wheel_dir)
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数。

### Lines 44-45: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Filesystem coordination** / 文件系统协同
- **Git state inspection** / Git 状态检查

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `hashlib`, `pathlib`, `re`
