# run_suite.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/srt/run_suite.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates run suite behavior in SGLang's srt area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 SRT 领域中与 run suite 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: module imports and dependencies / 模块导入与依赖
```python
import argparse
import glob
from pathlib import Path

import tabulate

from sglang.test.ci.ci_utils import TestFile, run_unittest_files
```
**EN:** This block imports the modules needed by the rest of the file, including `argparse`, `glob`, `pathlib`, `tabulate`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `argparse`, `glob`, `pathlib`, `tabulate`。

### Lines 8-12: supporting source context / 辅助源码上下文
```python

# NOTE: please sort the test cases alphabetically by the test file name
# NOTE: per-commit-4-gpu, per-commit-8-gpu-h200, per-commit-8-gpu-h20, per-commit-4-gpu-b200,
# per-commit-4-gpu-gb200, per-commit-4-gpu-deepep, and per-commit-8-gpu-h200-deepep suites
# have been migrated to base-c suites in test/registered/ using the CI registry system.
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 13-92: module-level constants and configuration (part 1/2) / 模块级常量与配置（第 1/2 部分）
```python
suites = {
    # quantization_test suite migrated to test/registered/quant/
    # All CUDA tests migrated to test/registered/
    "__not_in_ci__": [],
}

# Add AMD tests
# NOTE: please sort the test cases alphabetically by the test file name
suite_amd = {
    "per-commit-amd": [
        # TestFile("hicache/test_hicache.py", 116), # Disabled temporarily, see https://github.com/sgl-project/sglang/issues/12575
        # TestFile("hicache/test_hicache_mla.py", 127), # Disabled temporarily,  # Temporarily disabled, see https://github.com/sgl-project/sglang/issues/12574
        # TestFile("hicache/test_hicache_storage.py", 127), # Disabled temporarily, see https://github.com/sgl-project/sglang/issues/12575
        # LoRA tests moved to test/registered/lora/ - AMD entries need to be re-added there
        # TestFile("lora/test_lora_backend.py", 99), # Disabled temporarily, see https://github.com/sgl-project/sglang/issues/13107
        # TestFile("lora/test_lora_cuda_graph.py", 250), # Disabled temporarily, see https://github.com/sgl-project/sglang/issues/13107
        # TestFile("lora/test_lora_qwen3.py", 97), # Disabled temporarily, see https://github.com/sgl-project/sglang/issues/13107
        # TestFile("test_torch_compile_moe.py", 210), # Disabled temporarily, see https://github.com/sgl-project/sglang/issues/13107
        # Disabled temporarily
        # TestFile("test_vlm_input_format.py", 300),
        # TestFile("openai_server/features/test_openai_server_hidden_states.py", 240),
        # TestFile("rl/test_update_weights_from_tensor.py", 48),
        # TestFile("test_no_overlap_scheduler.py", 234), # Disabled temporarily and track in #7703
        # TestFile("test_vision_chunked_prefill.py", 175), # Disabled temporarily and track in #7701
        # TestFile("test_wave_attention_backend.py", 150), # Disabled temporarily, see https://github.com/sgl-project/sglang/issues/11127
        # The time estimation for `test_int4fp8_moe.py` assumes `mistralai/Mixtral-8x7B-Instruct-v0.1` is already cached (running on 1xMI300X).
    ],
    # per-commit-4-gpu-amd migrated to test/registered/distributed/ using the CI registry system
    "per-commit-4-gpu-amd": [],
    # NOTE: AMD nightly suites (nightly-amd, nightly-amd-vlm, nightly-amd-8-gpu)
    # have been migrated to test/registered/amd/nightly/ and are now managed
    # by test/run_suite.py using the registry system.
}

# Keep the Arm64 bootstrap suite limited to hosted-runner-safe unit kernels.
# `test_extend.py`, `test_mamba.py`, and `test_mla.py` still hit the
# x86-specific BF16 BRGEMM/VNNI path on Arm and need dedicated fallbacks.
suite_arm64 = {
    "per-commit-cpu-arm64": [
        TestFile("cpu/test_activation.py"),
        TestFile("cpu/test_decode.py"),
        TestFile("cpu/test_norm.py"),
        TestFile("cpu/test_qwen3.py"),
        TestFile("cpu/test_rope.py"),
        TestFile("cpu/test_server_args_backend.py"),
        TestFile("cpu/test_topk.py"),
    ],
}

# Add Intel Xeon tests
suite_xeon = {
    "per-commit-cpu": [
        TestFile("cpu/test_activation.py"),
        TestFile("cpu/test_binding.py"),
        TestFile("cpu/test_bmm.py"),
        TestFile("cpu/test_causal_conv1d.py"),
        TestFile("cpu/test_cpu_graph.py"),
        TestFile("cpu/test_decode.py"),
        TestFile("cpu/test_extend.py"),
        TestFile("cpu/test_flash_attn.py"),
        TestFile("cpu/test_gemm.py"),
        TestFile("cpu/test_intel_amx_attention_backend_a.py"),
        TestFile("cpu/test_intel_amx_attention_backend_b.py"),
        TestFile("cpu/test_intel_amx_attention_backend_c.py"),
        TestFile("cpu/test_mamba.py"),
        TestFile("cpu/test_mla.py"),
        TestFile("cpu/test_moe.py"),
        TestFile("cpu/test_norm.py"),
        TestFile("cpu/test_qkv_proj_with_rope.py"),
        TestFile("cpu/test_qwen3.py"),
        TestFile("cpu/test_rope.py"),
        TestFile("cpu/test_server_args_backend.py"),
        TestFile("cpu/test_shared_expert.py"),
        TestFile("cpu/test_topk.py"),
    ],
}

# Add Intel XPU tests
# NOTE: please sort the test cases alphabetically by the test file name
suite_xpu = {
```
**EN:** This block defines shared names such as `suites`, `suite_amd`, `suite_arm64`, `suite_xeon`, `suite_xpu`. These values centralize configuration that later tests or helpers reuse. This chunk continues part 1 of the same logical block.
**CN:** 该代码块定义了 `suites`, `suite_amd`, `suite_arm64`, `suite_xeon`, `suite_xpu` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。 这一段对应同一逻辑块的第 1 部分。

### Lines 93-104: module-level constants and configuration (part 2/2) / 模块级常量与配置（第 2/2 部分）
```python
    "per-commit-xpu": [
        TestFile("xpu/test_deepseek_ocr.py", 360),
        TestFile("xpu/test_deepseek_ocr_triton.py", 360),
        # TestFile("xpu/test_internvl.py"),
        TestFile("xpu/test_intel_xpu_backend.py"),
    ],
}

suites.update(suite_amd)
suites.update(suite_arm64)
suites.update(suite_xeon)
suites.update(suite_xpu)
```
**EN:** This block defines shared names such as `suites`, `suite_amd`, `suite_arm64`, `suite_xeon`, `suite_xpu`. These values centralize configuration that later tests or helpers reuse. This chunk continues part 2 of the same logical block.
**CN:** 该代码块定义了 `suites`, `suite_amd`, `suite_arm64`, `suite_xeon`, `suite_xpu` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。 这一段对应同一逻辑块的第 2 部分。

### Lines 107-148: function auto partition / 函数 auto partition
```python
def auto_partition(files, rank, size):
    """
    Partition files into size sublists with approximately equal sums of estimated times
    using stable sorting, and return the partition for the specified rank.

    Args:
        files (list): List of file objects with estimated_time attribute
        rank (int): Index of the partition to return (0 to size-1)
        size (int): Number of partitions

    Returns:
        list: List of file objects in the specified rank's partition
    """
    weights = [f.estimated_time for f in files]

    if not weights or size <= 0 or size > len(weights):
        return []

    # Create list of (weight, original_index) tuples
    # Using negative index as secondary key to maintain original order for equal weights
    indexed_weights = [(w, -i) for i, w in enumerate(weights)]
    # Stable sort in descending order by weight
    # If weights are equal, larger (negative) index comes first (i.e., earlier original position)
    indexed_weights = sorted(indexed_weights, reverse=True)

    # Extract original indices (negate back to positive)
    indexed_weights = [(w, -i) for w, i in indexed_weights]

    # Initialize partitions and their sums
    partitions = [[] for _ in range(size)]
    sums = [0.0] * size

    # Greedy approach: assign each weight to partition with smallest current sum
    for weight, idx in indexed_weights:
        # Find partition with minimum sum
        min_sum_idx = sums.index(min(sums))
        partitions[min_sum_idx].append(idx)
        sums[min_sum_idx] += weight

    # Return the files corresponding to the indices in the specified rank's partition
    indices = partitions[rank]
    return [files[i] for i in indices]
```
**EN:** Partition files into size sublists with approximately equal sums of estimated times using stable sorting, and return the partition for the specified rank. This block implements `auto_partition` and captures one focused piece of the module's behavior.
**CN:** Partition files into size sublists with approximately equal sums of estimated times using stable sorting, and return the partition for the specified rank. 该代码块实现 `auto_partition`，承担模块行为中的一个聚焦逻辑片段。

### Lines 151-195: function sanity check suites / 函数 sanity check suites
```python
def _sanity_check_suites(suites):
    dir_base = Path(__file__).parent
    disk_files = set(
        [
            str(x.relative_to(dir_base))
            for x in dir_base.glob("**/*.py")
            if x.name.startswith("test_")
        ]
    )

    suite_files = set(
        [test_file.name for _, suite in suites.items() for test_file in suite]
    )

    missing_files = sorted(list(disk_files - suite_files))
    missing_text = "\n".join(f'TestFile("{x}"),' for x in missing_files)
    assert len(missing_files) == 0, (
        f"Some test files are not in test suite. "
        f"If this is intentional, please add the following to `not_in_ci` section:\n"
        f"{missing_text}"
    )

    nonexistent_files = sorted(list(suite_files - disk_files))
    nonexistent_text = "\n".join(f'TestFile("{x}"),' for x in nonexistent_files)
    assert (
        len(nonexistent_files) == 0
    ), f"Some test files in test suite do not exist on disk:\n{nonexistent_text}"

    not_in_ci_files = set(
        [test_file.name for test_file in suites.get("__not_in_ci__", [])]
    )
    in_ci_files = set(
        [
            test_file.name
            for suite_name, suite in suites.items()
            if suite_name != "__not_in_ci__"
            for test_file in suite
        ]
    )
    intersection = not_in_ci_files & in_ci_files
    intersection_text = "\n".join(f'TestFile("{x}"),' for x in intersection)
    assert len(intersection) == 0, (
        f"Some test files are in both `not_in_ci` section and other suites:\n"
        f"{intersection_text}"
    )
```
**EN:** This block implements `_sanity_check_suites` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_sanity_check_suites`，承担模块行为中的一个聚焦逻辑片段。

### Lines 198-277: function main (part 1/2) / 函数 main（第 1/2 部分）
```python
def main():
    arg_parser = argparse.ArgumentParser()
    arg_parser.add_argument(
        "--timeout-per-file",
        type=int,
        default=1200,
        help="The time limit for running one file in seconds.",
    )
    arg_parser.add_argument(
        "--suite",
        type=str,
        default=list(suites.keys())[0],
        choices=list(suites.keys()) + ["all"],
        help="The suite to run",
    )
    arg_parser.add_argument(
        "--auto-partition-id",
        type=int,
        help="Use auto load balancing. The part id.",
    )
    arg_parser.add_argument(
        "--auto-partition-size",
        type=int,
        help="Use auto load balancing. The number of parts.",
    )
    arg_parser.add_argument(
        "--continue-on-error",
        action="store_true",
        default=False,
        help="Continue running remaining tests even if one fails (useful for nightly tests)",
    )
    arg_parser.add_argument(
        "--enable-retry",
        action="store_true",
        default=False,
        help="Enable smart retry for accuracy/performance assertion failures (not code errors)",
    )
    arg_parser.add_argument(
        "--max-attempts",
        type=int,
        default=2,
        help="Maximum number of attempts per file including initial run (default: 2)",
    )
    arg_parser.add_argument(
        "--retry-wait-seconds",
        type=int,
        default=60,
        help="Seconds to wait between retries (default: 60)",
    )
    arg_parser.add_argument(
        "--retry-timeout-increase",
        type=int,
        default=600,
        help="Additional timeout in seconds when retry is enabled (default: 600)",
    )
    args = arg_parser.parse_args()
    print(f"{args=}")

    _sanity_check_suites(suites)

    if args.suite == "all":
        files = glob.glob("**/test_*.py", recursive=True)
    else:
        files = suites[args.suite]

    if args.auto_partition_size:
        files = auto_partition(files, args.auto_partition_id, args.auto_partition_size)

    # Print test info at beginning (similar to test/run_suite.py pretty_print_tests)
    if args.auto_partition_size:
        partition_info = (
            f"{args.auto_partition_id + 1}/{args.auto_partition_size} "
            f"(0-based id={args.auto_partition_id})"
        )
    else:
        partition_info = "full"

    headers = ["Suite", "Partition"]
    rows = [[args.suite, partition_info]]
    msg = tabulate.tabulate(rows, headers=headers, tablefmt="psql") + "\n"
```
**EN:** This block implements `main` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `main`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 278-307: function main (part 2/2) / 函数 main（第 2/2 部分）
```python

    total_est_time = sum(f.estimated_time for f in files)
    msg += f"✅ Enabled {len(files)} test(s) (est total {total_est_time:.1f}s):\n"
    for f in files:
        msg += f"  - {f.name} (est_time={f.estimated_time})\n"

    print(msg, flush=True)

    # Add extra timeout when retry is enabled
    timeout = args.timeout_per_file
    if args.enable_retry:
        timeout += args.retry_timeout_increase

    exit_code = run_unittest_files(
        files,
        timeout,
        args.continue_on_error,
        args.enable_retry,
        args.max_attempts,
        args.retry_wait_seconds,
    )

    # Print tests again at the end for visibility
    msg = "\n" + tabulate.tabulate(rows, headers=headers, tablefmt="psql") + "\n"
    msg += f"✅ Executed {len(files)} test(s) (est total {total_est_time:.1f}s):\n"
    for f in files:
        msg += f"  - {f.name} (est_time={f.estimated_time})\n"
    print(msg, flush=True)

    exit(exit_code)
```
**EN:** This block implements `main` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `main`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 310-316: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    print(
        "DEPRECATION NOTICE: The folder `test/srt` should be deprecated as soon as possible. "
        "Migrate tests to the new CI registry system described in `test/README.md`.",
        flush=True,
    )
    main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `auto_partition`: Partition files into size sublists with approximately equal sums of estimated times using stable sorting, and return the partition for the specified rank. / 该代码块实现 `auto_partition`，承担模块行为中的一个聚焦逻辑片段。
- `_sanity_check_suites`: This block implements `_sanity_check_suites` and captures one focused piece of the module's behavior. / 该代码块实现 `_sanity_check_suites`，承担模块行为中的一个聚焦逻辑片段。
- `main`: This block implements `main` and captures one focused piece of the module's behavior. / 该代码块实现 `main`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `glob`, `pathlib`
- **Third-party modules / 第三方模块**: `tabulate`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_utils`

- **Total lines / 总行数**: 316
