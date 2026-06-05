# gen_diffusion_ci_outputs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/scripts/gen_diffusion_ci_outputs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates gen diffusion ci outputs with focused assertions and fixtures. Key symbols include `main`. / 该测试模块通过有针对性的断言与夹具，验证 gen diffusion ci outputs 的实现。 关键符号包括 `main`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Imports and module setup / 导入与模块初始化
```python
#!/usr/bin/env python3
"""
Generate diffusion CI outputs for consistency testing.

This script reuses the CI test code by calling run_suite.py with SGLANG_GEN_GT=1,
ensuring that GT generation uses exactly the same code path as CI tests.

Usage:
    python gen_diffusion_ci_outputs.py --suite 1-gpu --partition-id 0 --total-partitions 2 --out-dir ./output
    python gen_diffusion_ci_outputs.py --suite 1-gpu --case-ids qwen_image_t2i flux_image_t2i --out-dir ./output
    python gen_diffusion_ci_outputs.py --suite 1-gpu-b200 --out-dir ./output
"""

import argparse
# ...
    run_pytest,
)

logger = init_logger(__name__)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 35-220: Function `main` / 函数 `main`
```python
def main():
    """Main entry point."""
    parser = argparse.ArgumentParser(description="Generate diffusion CI outputs")
    parser.add_argument(
        "--suite",
        type=str,
        choices=list(SUITES.keys()),
        required=True,
        help="Test suite to run (choices: " + ", ".join(list(SUITES.keys())) + ")",
    )
    parser.add_argument(
        "--partition-id",
        type=int,
        required=False,
# ...
        if args.continue_on_error:
            logger.warning(f"pytest exited with code {exit_code}")
        else:
            sys.exit(exit_code)
```
**EN:** This function drives `main`. Main entry point.
**CN:** 这个函数负责 `main`。 文档字符串说明：Main entry point.

### Lines 221-224: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- Automated verification / 自动化验证
- Pytest-based assertions / 基于 Pytest 的断言
- Image generation flow / 图像生成流程
- Symbol `main` anchors the module API / 符号 `main` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.test.run_suite`
- **Stdlib / 标准库**: `argparse`, `os`, `sys`, `pathlib`
