# test_lora_hf_sgl_logprob_diff.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/lora/test_lora_hf_sgl_logprob_diff.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates lora hf sgl logprob diff behavior in SGLang's lora area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 lora 领域中与 lora hf sgl logprob diff 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: supporting source context / 辅助源码上下文
```python
# Copyright 2023-2024 SGLang Team
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 15-28: supporting statements / 辅助语句
```python
"""
Test to compare log probabilities between HuggingFace+LoRA and SGLang+LoRA.

This test:
1. Runs SGLang with LoRA and collects log probabilities
2. Runs HuggingFace with LoRA and collects log probabilities
3. Compares the differences (max and mean) between the two implementations
4. Uses unittest framework for easy integration with test suites

Usage:
    python test_lora_hf_sgl_logprob_diff.py
    or
    python -m unittest test_lora_hf_sgl_logprob_diff
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 30-40: module imports and dependencies / 模块导入与依赖
```python
import multiprocessing as mp
import os
import unittest
from typing import Any, Dict, List, Optional, Tuple

import numpy as np
import torch

from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.runners import HFRunner, SRTRunner
from sglang.test.test_utils import DEFAULT_PORT_FOR_SRT_TEST_RUNNER, CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `multiprocessing`, `os`, `unittest`, `typing`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `multiprocessing`, `os`, `unittest`, `typing`。

### Lines 42-72: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(
    est_time=150,
    stage="base-b",
    runner_config="1-gpu-small",
)
register_amd_ci(
    est_time=250,
    suite="stage-b-test-1-gpu-small-amd",
)
# Test configuration constants
BASE_MODEL = "meta-llama/Llama-2-7b-hf"
LORA_PATHS = ["yushengsu/sglang_lora_logprob_diff_without_tuning"]
LORA_BACKEND = "csgmv"
DISABLE_CUDA_GRAPH = False
LORA_TARGET_MODULES = None
LOGPROB_THRESHOLD = 1e-01
MAX_NEW_TOKENS = 32

# Default test prompts
DEFAULT_TEST_PROMPTS = [
    "SGL is a",
    "AI is a field of computer science focused on",
    "Write a long story.",
    "Write a short story.",
    "What are the main components of a computer?",
]

# Formatting constants
DIVIDER_WIDTH = 80
SECTION_CHAR = "="
SUBSECTION_CHAR = "-"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 75-79: function print section header / 函数 print section header
```python
def print_section_header(title: str):
    """Print a major section header."""
    print("\n" + SECTION_CHAR * DIVIDER_WIDTH)
    print(title)
    print(SECTION_CHAR * DIVIDER_WIDTH)
```
**EN:** Print a major section header. This block implements `print_section_header` and captures one focused piece of the module's behavior.
**CN:** Print a major section header. 该代码块实现 `print_section_header`，承担模块行为中的一个聚焦逻辑片段。

### Lines 82-86: function print subsection header / 函数 print subsection header
```python
def print_subsection_header(title: str):
    """Print a subsection header."""
    print(f"\n{SUBSECTION_CHAR * 40}")
    print(f"{title}")
    print(SUBSECTION_CHAR * 40)
```
**EN:** Print a subsection header. This block implements `print_subsection_header` and captures one focused piece of the module's behavior.
**CN:** Print a subsection header. 该代码块实现 `print_subsection_header`，承担模块行为中的一个聚焦逻辑片段。

### Lines 89-93: function print config info / 函数 print config info
```python
def print_config_info(title: str, config: Dict[str, Any]):
    """Print configuration information in a consistent format."""
    print_section_header(title)
    for key, value in config.items():
        print(f"  {key}: {value}")
```
**EN:** Print configuration information in a consistent format. This block implements `print_config_info` and captures one focused piece of the module's behavior.
**CN:** Print configuration information in a consistent format. 该代码块实现 `print_config_info`，承担模块行为中的一个聚焦逻辑片段。

### Lines 96-122: function compare logprobs for type / 函数 compare logprobs for type
```python
def compare_logprobs_for_type(
    sglang_logprobs: torch.Tensor, hf_logprobs: torch.Tensor, logprob_type: str
) -> Dict[str, Any]:
    """
    Compare logprobs for a specific type (prefill or decode).

    Args:
        sglang_logprobs: SGLang log probabilities
        hf_logprobs: HuggingFace log probabilities
        logprob_type: Type of logprobs ("prefill" or "decode")

    Returns:
        Dictionary containing comparison statistics
    """
    diff = torch.abs(sglang_logprobs - hf_logprobs)
    max_diff = torch.max(diff).item()
    mean_diff = torch.mean(diff).item()
    shape = list(sglang_logprobs.shape)
    matches_threshold = max_diff < LOGPROB_THRESHOLD

    return {
        "max_diff": max_diff,
        "mean_diff": mean_diff,
        "shape": shape,
        "matches_threshold": matches_threshold,
        "type": logprob_type,
    }
```
**EN:** Compare logprobs for a specific type (prefill or decode). This block implements `compare_logprobs_for_type` and captures one focused piece of the module's behavior.
**CN:** Compare logprobs for a specific type (prefill or decode). 该代码块实现 `compare_logprobs_for_type`，承担模块行为中的一个聚焦逻辑片段。

### Lines 125-134: function print logprob comparison / 函数 print logprob comparison
```python
def print_logprob_comparison(comparison: Dict[str, Any]):
    """Print logprob comparison results in a consistent format."""
    logprob_type = comparison["type"].capitalize()
    print(f"\n{logprob_type} logprobs:")
    print(f"  Shape:           {comparison['shape']}")
    print(f"  Max difference:  {comparison['max_diff']:.6e}")
    print(f"  Mean difference: {comparison['mean_diff']:.6e}")

    status = "PASS" if comparison["matches_threshold"] else "FAIL"
    print(f"  Status:          {status} (threshold: {LOGPROB_THRESHOLD:.0e})")
```
**EN:** Print logprob comparison results in a consistent format. This block implements `print_logprob_comparison` and captures one focused piece of the module's behavior.
**CN:** Print logprob comparison results in a consistent format. 该代码块实现 `print_logprob_comparison`，承担模块行为中的一个聚焦逻辑片段。

### Lines 137-169: function compare output strings / 函数 compare output strings
```python
def compare_output_strings(
    sglang_output: str, hf_output: str, max_display_len: int = 200
) -> Dict[str, Any]:
    """
    Compare output strings between SGLang and HuggingFace.

    Args:
        sglang_output: SGLang generated text
        hf_output: HuggingFace generated text
        max_display_len: Maximum length for display

    Returns:
        Dictionary containing comparison results
    """
    outputs_match = sglang_output.strip() == hf_output.strip()

    # Truncate for display if needed
    sglang_display = (
        sglang_output[:max_display_len]
        if len(sglang_output) > max_display_len
        else sglang_output
    )
    hf_display = (
        hf_output[:max_display_len] if len(hf_output) > max_display_len else hf_output
    )

    return {
        "match": outputs_match,
        "sglang_output": sglang_output,
        "hf_output": hf_output,
        "sglang_display": sglang_display,
        "hf_display": hf_display,
    }
```
**EN:** Compare output strings between SGLang and HuggingFace. This block implements `compare_output_strings` and captures one focused piece of the module's behavior.
**CN:** Compare output strings between SGLang and HuggingFace. 该代码块实现 `compare_output_strings`，承担模块行为中的一个聚焦逻辑片段。

### Lines 172-178: function print output comparison / 函数 print output comparison
```python
def print_output_comparison(comparison: Dict[str, Any]):
    """Print output string comparison in a consistent format."""
    print(f"\nOutput strings:")
    status = "MATCH" if comparison["match"] else "DIFFER"
    print(f"  Status:      {status}")
    print(f"  SGLang:      {comparison['sglang_display']}")
    print(f"  HuggingFace: {comparison['hf_display']}")
```
**EN:** Print output string comparison in a consistent format. This block implements `print_output_comparison` and captures one focused piece of the module's behavior.
**CN:** Print output string comparison in a consistent format. 该代码块实现 `print_output_comparison`，承担模块行为中的一个聚焦逻辑片段。

### Lines 181-197: function prepare lora paths per prompt / 函数 prepare lora paths per prompt
```python
def prepare_lora_paths_per_prompt(
    lora_paths: List[str], num_prompts: int
) -> List[Optional[str]]:
    """
    Prepare LoRA paths for each prompt by cycling through available LoRAs.

    Args:
        lora_paths: List of available LoRA adapter paths
        num_prompts: Number of prompts to generate LoRA paths for

    Returns:
        List of LoRA paths (one per prompt), or None values if no LoRAs
    """
    if not lora_paths:
        return [None] * num_prompts

    return [lora_paths[i % len(lora_paths)] for i in range(num_prompts)]
```
**EN:** Prepare LoRA paths for each prompt by cycling through available LoRAs. This block implements `prepare_lora_paths_per_prompt` and captures one focused piece of the module's behavior.
**CN:** Prepare LoRA paths for each prompt by cycling through available LoRAs. 该代码块实现 `prepare_lora_paths_per_prompt`，承担模块行为中的一个聚焦逻辑片段。

### Lines 200-251: helper routine run sglang with lora / 辅助流程 run sglang with lora
```python
def run_sglang_with_lora(
    model_path: str,
    lora_paths: List[str],
    prompts: List[str],
    max_new_tokens: int,
    torch_dtype: torch.dtype,
    lora_backend: str,
    port: int,
    disable_cuda_graph: bool,
    lora_target_modules: Optional[List[str]],
    tp_size: int,
) -> Dict[str, Any]:
    """Run SGLang with LoRA and return log probabilities."""
    config = {
        "Model": model_path,
        "LoRA paths": lora_paths,
        "LoRA backend": lora_backend,
        "Disable CUDA graph": disable_cuda_graph,
        "Port": port,
        "Number of prompts": len(prompts),
        "Tensor parallel size": tp_size,
    }
    print_config_info("Running SGLang with LoRA", config)

    lora_paths_per_prompt = prepare_lora_paths_per_prompt(lora_paths, len(prompts))

    with SRTRunner(
        model_path,
        torch_dtype=torch_dtype,
        model_type="generation",
        tp_size=tp_size,
        lora_paths=lora_paths,
        max_loras_per_batch=len(lora_paths) if lora_paths else 1,
        lora_backend=lora_backend,
        disable_cuda_graph=disable_cuda_graph,
        disable_radix_cache=True,
        port=port,
        mem_fraction_static=0.88,
        lora_target_modules=lora_target_modules,
    ) as srt_runner:
        srt_outputs = srt_runner.forward(
            prompts,
            max_new_tokens=max_new_tokens,
            lora_paths=lora_paths_per_prompt,
        )

    return {
        "top_input_logprobs": srt_outputs.top_input_logprobs,
        "top_output_logprobs": srt_outputs.top_output_logprobs,
        "output_strs": srt_outputs.output_strs,
        "lora_paths": lora_paths_per_prompt,
    }
```
**EN:** Run SGLang with LoRA and return log probabilities. This helper encapsulates `run_sglang_with_lora` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Run SGLang with LoRA and return log probabilities. 该辅助函数封装了 `run_sglang_with_lora`，以便周围测试复用准备、执行或校验逻辑。

### Lines 254-288: helper routine run hf with lora / 辅助流程 run hf with lora
```python
def run_hf_with_lora(
    model_path: str,
    lora_paths: List[str],
    prompts: List[str],
    max_new_tokens: int,
    torch_dtype: torch.dtype,
) -> Dict[str, Any]:
    """Run HuggingFace with LoRA and return log probabilities."""
    config = {
        "Model": model_path,
        "LoRA paths": lora_paths,
        "Number of prompts": len(prompts),
    }
    print_config_info("Running HuggingFace with LoRA", config)

    lora_paths_per_prompt = prepare_lora_paths_per_prompt(lora_paths, len(prompts))

    with HFRunner(
        model_path,
        torch_dtype=torch_dtype,
        model_type="generation",
        patch_model_do_sample_false=True,
    ) as hf_runner:
        hf_outputs = hf_runner.forward(
            prompts,
            max_new_tokens=max_new_tokens,
            lora_paths=lora_paths_per_prompt,
        )

    return {
        "top_input_logprobs": hf_outputs.top_input_logprobs,
        "top_output_logprobs": hf_outputs.top_output_logprobs,
        "output_strs": hf_outputs.output_strs,
        "lora_paths": lora_paths_per_prompt,
    }
```
**EN:** Run HuggingFace with LoRA and return log probabilities. This helper encapsulates `run_hf_with_lora` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Run HuggingFace with LoRA and return log probabilities. 该辅助函数封装了 `run_hf_with_lora`，以便周围测试复用准备、执行或校验逻辑。

### Lines 291-358: function compare single prompt / 函数 compare single prompt
```python
def compare_single_prompt(
    prompt_idx: int,
    sglang_data: Dict[str, Any],
    hf_data: Dict[str, Any],
) -> Dict[str, Any]:
    """
    Compare logprobs and outputs for a single prompt.

    Args:
        prompt_idx: Index of the prompt being compared
        sglang_data: SGLang results data
        hf_data: HuggingFace results data

    Returns:
        Dictionary containing all comparison results
    """
    print_subsection_header(f"Prompt {prompt_idx + 1}")
    print(f"LoRA adapter: {sglang_data['lora_paths'][prompt_idx]}")

    result = {
        "prompt_idx": prompt_idx,
        "lora_path": sglang_data["lora_paths"][prompt_idx],
    }

    # Compare prefill (input) logprobs
    sglang_prefill = torch.tensor(sglang_data["top_input_logprobs"][prompt_idx])
    hf_prefill = torch.tensor(hf_data["top_input_logprobs"][prompt_idx])
    prefill_comparison = compare_logprobs_for_type(
        sglang_prefill, hf_prefill, "prefill"
    )
    print_logprob_comparison(prefill_comparison)

    # Store prefill results
    result["prefill_max_diff"] = prefill_comparison["max_diff"]
    result["prefill_mean_diff"] = prefill_comparison["mean_diff"]
    result["prefill_shape"] = prefill_comparison["shape"]
    result["prefill_logprob_match"] = prefill_comparison["matches_threshold"]

    # Compare decode (output) logprobs
    sglang_decode = torch.tensor(sglang_data["top_output_logprobs"][prompt_idx])
    hf_decode = torch.tensor(hf_data["top_output_logprobs"][prompt_idx])
    decode_comparison = compare_logprobs_for_type(sglang_decode, hf_decode, "decode")
    print_logprob_comparison(decode_comparison)

    # Store decode results
    result["decode_max_diff"] = decode_comparison["max_diff"]
    result["decode_mean_diff"] = decode_comparison["mean_diff"]
    result["decode_shape"] = decode_comparison["shape"]
    result["decode_logprob_match"] = decode_comparison["matches_threshold"]

    # Overall logprob match
    result["overall_logprob_match"] = (
        prefill_comparison["matches_threshold"]
        and decode_comparison["matches_threshold"]
    )

    # Compare output strings
    sglang_output = sglang_data["output_strs"][prompt_idx]
    hf_output = hf_data["output_strs"][prompt_idx]
    output_comparison = compare_output_strings(sglang_output, hf_output)
    print_output_comparison(output_comparison)

    # Store output results
    result["outputs_match"] = output_comparison["match"]
    result["sglang_output"] = output_comparison["sglang_output"]
    result["hf_output"] = output_comparison["hf_output"]

    return result
```
**EN:** Compare logprobs and outputs for a single prompt. This block implements `compare_single_prompt` and captures one focused piece of the module's behavior.
**CN:** Compare logprobs and outputs for a single prompt. 该代码块实现 `compare_single_prompt`，承担模块行为中的一个聚焦逻辑片段。

### Lines 361-419: function print overall statistics / 函数 print overall statistics
```python
def print_overall_statistics(results: List[Dict[str, Any]]):
    """Print overall statistics across all prompts."""
    print_section_header("Overall Statistics")

    # Gather statistics
    prefill_max_diffs = [r["prefill_max_diff"] for r in results]
    prefill_mean_diffs = [r["prefill_mean_diff"] for r in results]
    decode_max_diffs = [r["decode_max_diff"] for r in results]
    decode_mean_diffs = [r["decode_mean_diff"] for r in results]

    # Print logprob statistics
    print("\nLogprob Differences:")
    print(f"  Prefill:")
    print(f"    Max of max:   {max(prefill_max_diffs):.6e}")
    print(f"    Mean of max:  {np.mean(prefill_max_diffs):.6e}")
    print(f"    Mean of mean: {np.mean(prefill_mean_diffs):.6e}")

    print(f"  Decode:")
    print(f"    Max of max:   {max(decode_max_diffs):.6e}")
    print(f"    Mean of max:  {np.mean(decode_max_diffs):.6e}")
    print(f"    Mean of mean: {np.mean(decode_mean_diffs):.6e}")

    # Print match statistics
    num_prompts = len(results)
    logprob_match_count = sum(r["overall_logprob_match"] for r in results)
    prefill_match_count = sum(r["prefill_logprob_match"] for r in results)
    decode_match_count = sum(r["decode_logprob_match"] for r in results)
    outputs_match_count = sum(r["outputs_match"] for r in results)

    print(f"\nLogprob Statistics (threshold: {LOGPROB_THRESHOLD:.0e}):")
    overall_status = "PASSED" if logprob_match_count == num_prompts else "FAILED"
    print(f"  Overall logprob: {logprob_match_count}/{num_prompts} {overall_status}")
    print(f"  Prefill logprob: {prefill_match_count}/{num_prompts}")
    print(f"  Decode logprob:  {decode_match_count}/{num_prompts}")

    print(f"\nString Statistics:")
    print(f"  Output strings:  {outputs_match_count}/{num_prompts}")

    # Return overall stats for saving
    return {
        "logprob_differences": {
            "prefill": {
                "max_of_max_diffs": max(prefill_max_diffs),
                "mean_of_max_diffs": float(np.mean(prefill_max_diffs)),
                "mean_of_mean_diffs": float(np.mean(prefill_mean_diffs)),
            },
            "decode": {
                "max_of_max_diffs": max(decode_max_diffs),
                "mean_of_max_diffs": float(np.mean(decode_max_diffs)),
                "mean_of_mean_diffs": float(np.mean(decode_mean_diffs)),
            },
        },
        "match_statistics": {
            "overall_logprob_match_rate": logprob_match_count / num_prompts,
            "prefill_logprob_match_rate": prefill_match_count / num_prompts,
            "decode_logprob_match_rate": decode_match_count / num_prompts,
            "outputs_match_rate": outputs_match_count / num_prompts,
        },
    }
```
**EN:** Print overall statistics across all prompts. This block implements `print_overall_statistics` and captures one focused piece of the module's behavior.
**CN:** Print overall statistics across all prompts. 该代码块实现 `print_overall_statistics`，承担模块行为中的一个聚焦逻辑片段。

### Lines 422-437: function compare logprobs / 函数 compare logprobs
```python
def compare_logprobs(
    sglang_logprobs: Dict[str, Any], hf_logprobs: Dict[str, Any]
) -> Tuple[List[Dict[str, Any]], Dict[str, Any]]:
    """Compare log probabilities and compute statistics."""
    print_section_header("Comparing Log Probabilities")

    results = []
    num_prompts = len(sglang_logprobs["top_input_logprobs"])

    for i in range(num_prompts):
        result = compare_single_prompt(i, sglang_logprobs, hf_logprobs)
        results.append(result)

    overall_stats = print_overall_statistics(results)

    return results, overall_stats
```
**EN:** Compare log probabilities and compute statistics. This block implements `compare_logprobs` and captures one focused piece of the module's behavior.
**CN:** Compare log probabilities and compute statistics. 该代码块实现 `compare_logprobs`，承担模块行为中的一个聚焦逻辑片段。

### Lines 440-440: class TestLoRAHFSGLLogprobDifference declaration / 类 TestLoRAHFSGLLogprobDifference 声明
```python
class TestLoRAHFSGLLogprobDifference(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 441-443: supporting statements / 辅助语句
```python
    """
    Test case to compare log probabilities between HuggingFace+LoRA and SGLang+LoRA.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 445-508: method run comparison test / 方法 run comparison test
```python
    def _run_comparison_test(
        self,
        model_path: str,
        lora_paths: List[str],
        prompts: List[str],
        max_new_tokens: int = MAX_NEW_TOKENS,
        torch_dtype: torch.dtype = torch.float16,
        lora_backend: str = LORA_BACKEND,
        port: int = DEFAULT_PORT_FOR_SRT_TEST_RUNNER,
        disable_cuda_graph: bool = DISABLE_CUDA_GRAPH,
        lora_target_modules: Optional[List[str]] = LORA_TARGET_MODULES,
        tp_size: int = 1,
    ):
        """
        Run comparison test between SGLang and HuggingFace with LoRA.
        """
        print_section_header(f"Testing {model_path} with LoRA adapters")

        # Step 1: Run SGLang with LoRA
        sglang_logprobs = run_sglang_with_lora(
            model_path=model_path,
            lora_paths=lora_paths,
            prompts=prompts,
            max_new_tokens=max_new_tokens,
            torch_dtype=torch_dtype,
            lora_backend=lora_backend,
            port=port,
            disable_cuda_graph=disable_cuda_graph,
            lora_target_modules=lora_target_modules,
            tp_size=tp_size,
        )

        # Clear GPU memory
        print("\nClearing GPU memory...")
        torch.cuda.empty_cache()

        # Step 2: Run HuggingFace with LoRA
        hf_logprobs = run_hf_with_lora(
            model_path=model_path,
            lora_paths=lora_paths,
            prompts=prompts,
            max_new_tokens=max_new_tokens,
            torch_dtype=torch_dtype,
        )

        # Step 3: Compare log probabilities
        results, overall_stats = compare_logprobs(sglang_logprobs, hf_logprobs)

        # Assert that all prompts pass the threshold
        for result in results:
            self.assertTrue(
                result["prefill_logprob_match"],
                f"Prefill logprob mismatch for prompt {result['prompt_idx']} "
                f"(max_diff={result['prefill_max_diff']:.6e}, threshold={LOGPROB_THRESHOLD:.0e})",
            )
            self.assertTrue(
                result["decode_logprob_match"],
                f"Decode logprob mismatch for prompt {result['prompt_idx']} "
                f"(max_diff={result['decode_max_diff']:.6e}, threshold={LOGPROB_THRESHOLD:.0e})",
            )

        print_section_header("Test completed successfully!")

        return results, overall_stats
```
**EN:** Run comparison test between SGLang and HuggingFace with LoRA. This block implements `_run_comparison_test` and captures one focused piece of the module's behavior.
**CN:** Run comparison test between SGLang and HuggingFace with LoRA. 该代码块实现 `_run_comparison_test`，承担模块行为中的一个聚焦逻辑片段。

### Lines 510-520: test case lora logprob comparison basic / 测试用例 lora logprob comparison basic
```python
    def test_lora_logprob_comparison_basic(self):
        """
        Basic test comparing HF and SGLang LoRA logprobs with small model.
        """
        prompts = DEFAULT_TEST_PROMPTS[:2]  # Use fewer prompts for faster testing

        self._run_comparison_test(
            model_path=BASE_MODEL,
            lora_paths=LORA_PATHS,
            prompts=prompts,
        )
```
**EN:** Basic test comparing HF and SGLang LoRA logprobs with small model. This test exercises `test_lora_logprob_comparison_basic` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Basic test comparing HF and SGLang LoRA logprobs with small model. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_logprob_comparison_basic`。

### Lines 522-530: test case lora logprob comparison full / 测试用例 lora logprob comparison full
```python
    def test_lora_logprob_comparison_full(self):
        """
        Full test comparing HF and SGLang LoRA logprobs with all prompts.
        """
        self._run_comparison_test(
            model_path=BASE_MODEL,
            lora_paths=LORA_PATHS,
            prompts=DEFAULT_TEST_PROMPTS,
        )
```
**EN:** Full test comparing HF and SGLang LoRA logprobs with all prompts. This test exercises `test_lora_logprob_comparison_full` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Full test comparing HF and SGLang LoRA logprobs with all prompts. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_logprob_comparison_full`。

### Lines 532-557: test case lora logprob comparison chunked / 测试用例 lora logprob comparison chunked
```python
    def test_lora_logprob_comparison_chunked(self):
        """
        Test with logprobs chunking enabled and a small chunk size so that
        even short prompts trigger the multi-pass lm_head LoRA path.
        """
        saved = {}
        env_overrides = {
            "SGLANG_ENABLE_LOGITS_PROCESSER_CHUNK": "true",
            "SGLANG_LOGITS_PROCESSER_CHUNK_SIZE": "4",
        }
        for key, val in env_overrides.items():
            saved[key] = os.environ.get(key)
            os.environ[key] = val

        try:
            self._run_comparison_test(
                model_path=BASE_MODEL,
                lora_paths=LORA_PATHS,
                prompts=DEFAULT_TEST_PROMPTS,
            )
        finally:
            for key, orig in saved.items():
                if orig is None:
                    os.environ.pop(key, None)
                else:
                    os.environ[key] = orig
```
**EN:** Test with logprobs chunking enabled and a small chunk size so that even short prompts trigger the multi-pass lm_head LoRA path. This test exercises `test_lora_logprob_comparison_chunked` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test with logprobs chunking enabled and a small chunk size so that even short prompts trigger the multi-pass lm_head LoRA path. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_logprob_comparison_chunked`。

### Lines 560-572: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    try:
        mp.set_start_method("spawn")
    except RuntimeError:
        pass

    try:
        unittest.main(warnings="ignore", verbosity=2)
    finally:
        # Final cleanup
        if torch.cuda.is_available():
            torch.cuda.empty_cache()
            torch.cuda.synchronize()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `print_section_header`: Print a major section header. / 该代码块实现 `print_section_header`，承担模块行为中的一个聚焦逻辑片段。
- `print_subsection_header`: Print a subsection header. / 该代码块实现 `print_subsection_header`，承担模块行为中的一个聚焦逻辑片段。
- `print_config_info`: Print configuration information in a consistent format. / 该代码块实现 `print_config_info`，承担模块行为中的一个聚焦逻辑片段。
- `compare_logprobs_for_type`: Compare logprobs for a specific type (prefill or decode). / 该代码块实现 `compare_logprobs_for_type`，承担模块行为中的一个聚焦逻辑片段。
- `print_logprob_comparison`: Print logprob comparison results in a consistent format. / 该代码块实现 `print_logprob_comparison`，承担模块行为中的一个聚焦逻辑片段。
- `compare_output_strings`: Compare output strings between SGLang and HuggingFace. / 该代码块实现 `compare_output_strings`，承担模块行为中的一个聚焦逻辑片段。
- `print_output_comparison`: Print output string comparison in a consistent format. / 该代码块实现 `print_output_comparison`，承担模块行为中的一个聚焦逻辑片段。
- `prepare_lora_paths_per_prompt`: Prepare LoRA paths for each prompt by cycling through available LoRAs. / 该代码块实现 `prepare_lora_paths_per_prompt`，承担模块行为中的一个聚焦逻辑片段。
- `TestLoRAHFSGLLogprobDifference._run_comparison_test`: Run comparison test between SGLang and HuggingFace with LoRA. / 该代码块实现 `_run_comparison_test`，承担模块行为中的一个聚焦逻辑片段。
- `TestLoRAHFSGLLogprobDifference.test_lora_logprob_comparison_basic`: Basic test comparing HF and SGLang LoRA logprobs with small model. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_logprob_comparison_basic`。
- `TestLoRAHFSGLLogprobDifference.test_lora_logprob_comparison_full`: Full test comparing HF and SGLang LoRA logprobs with all prompts. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_logprob_comparison_full`。
- `TestLoRAHFSGLLogprobDifference.test_lora_logprob_comparison_chunked`: Test with logprobs chunking enabled and a small chunk size so that even short prompts trigger the multi-pass lm_head LoRA path. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_logprob_comparison_chunked`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `multiprocessing`, `os`, `unittest`, `typing`
- **Third-party modules / 第三方模块**: `numpy`, `torch`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.test.runners`, `sglang.test.test_utils`

- **Total lines / 总行数**: 572
