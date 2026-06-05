# test_wan22_fp8_mla.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/amd/test_wan22_fp8_mla.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on amd wan22 fp8 mla in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 amd wan22 fp8 mla 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Document the module
```python
"""AMD test for Wan2.2-T2V-A14B with FP8 MLA attention (1-GPU and 8-GPU)."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding module.
**CN:** 该代码块保存说明性文本，用于解释周围module的设计意图。

### Lines 3-7: Import dependencies
```python
from __future__ import annotations

import logging

import pytest
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 9-9: Implement expr logic
```python
pytest.importorskip("sgl_kernel", reason="sgl_kernel is required for FP8 MLA tests")
```
**EN:** This block contains supporting statements that connect the surrounding setup and assertions.
**CN:** 该代码块包含连接周边初始化与断言的辅助语句。

### Lines 11-24: Import dependencies
```python
from sglang.multimodal_gen.test.server.test_server_common import (  # noqa: F401
    DiffusionServerBase,
    diffusion_server,
)
from sglang.multimodal_gen.test.server.test_server_utils import (
    ServerContext,
    get_generate_fn,
)
from sglang.multimodal_gen.test.server.testcase_configs import (
    DiffusionServerArgs,
    DiffusionTestCase,
    T2V_sampling_params,
)
from sglang.test.ci.ci_register import register_amd_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also registers the case for AMD CI coverage.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 26-26: Define module constants
```python
logger = logging.getLogger(__name__)
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 28-28: Register CI metadata
```python
register_amd_ci(est_time=3600, suite="nightly-amd-fp8-mla-diffusion", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 AMD CI 覆盖范围。

### Lines 30-31: Define module constants (part 1)
```python
MODEL = "Wan-AI/Wan2.2-T2V-A14B-Diffusers"
FP8_ENV = {"SGLANG_DIFFUSION_AITER_FP8_ATTN": "1"}
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 33-92: Define module constants (part 2)
```python
FP8_MLA_CASES = [
    DiffusionTestCase(
        "wan2_2_t2v_a14b_fp8_mla_1gpu",
        DiffusionServerArgs(
            model_path=MODEL,
            modality="video",
            num_gpus=1,
            extras=["--enable-torch-compile false"],
            env_vars=FP8_ENV,
        ),
        T2V_sampling_params,
        run_perf_check=False,
        run_consistency_check=False,
        run_models_api_check=False,
        run_t2v_input_reference_check=False,
    ),
    DiffusionTestCase(
        "wan2_2_t2v_a14b_fp8_mla_1gpu_compile",
        DiffusionServerArgs(
            model_path=MODEL,
            modality="video",
            num_gpus=1,
            extras=["--enable-torch-compile true"],
            env_vars=FP8_ENV,
        ),
        T2V_sampling_params,
        run_perf_check=False,
        run_consistency_check=False,
        run_models_api_check=False,
        run_t2v_input_reference_check=False,
    ),
    DiffusionTestCase(
        "wan2_2_t2v_a14b_fp8_mla_8gpu",
        DiffusionServerArgs(
            model_path=MODEL,
            modality="video",
            num_gpus=8,
            ulysses_degree=4,
            cfg_parallel=True,
            extras=["--enable-torch-compile false"],
            env_vars=FP8_ENV,
        ),
        T2V_sampling_params,
        run_perf_check=False,
        run_consistency_check=False,
        run_models_api_check=False,
        run_t2v_input_reference_check=False,
    ),
    DiffusionTestCase(
        "wan2_2_t2v_a14b_fp8_mla_8gpu_compile",
        DiffusionServerArgs(
            model_path=MODEL,
            modality="video",
            num_gpus=8,
            ulysses_degree=4,
            cfg_parallel=True,
            extras=["--enable-torch-compile true"],
            env_vars=FP8_ENV,
        ),
        T2V_sampling_params,
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 93-98: Define module constants (part 3)
```python
        run_perf_check=False,
        run_consistency_check=False,
        run_models_api_check=False,
        run_t2v_input_reference_check=False,
    ),
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 101-101: Define class TestWan22FP8MLA
```python
class TestWan22FP8MLA(DiffusionServerBase):
```
**EN:** This declaration introduces the `TestWan22FP8MLA` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestWan22FP8MLA` 测试类，并说明它通过继承承担的职责。

### Lines 102-102: Document the class `TestWan22FP8MLA`
```python
    """AMD test for FP8 MLA attention on Wan2.2-T2V-A14B."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestWan22FP8MLA`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestWan22FP8MLA`的设计意图。

### Lines 104-109: Define helper: teardown class
```python
    @classmethod
    def teardown_class(cls):
        try:
            super().teardown_class()
        except AttributeError:
            pass
```
**EN:** This helper function encapsulates reusable logic inside `TestWan22FP8MLA` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWan22FP8MLA` 内部调用，从而让场景结构更清晰。

### Lines 111-113: Define helper: case
```python
    @pytest.fixture(params=FP8_MLA_CASES, ids=lambda c: c.id)
    def case(self, request) -> DiffusionTestCase:
        return request.param
```
**EN:** This helper function encapsulates reusable logic inside `TestWan22FP8MLA` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestWan22FP8MLA` 内部调用，从而让场景结构更清晰。

### Lines 115-130: Run test: diffusion generation
```python
    def test_diffusion_generation(
        self,
        case: DiffusionTestCase,
        diffusion_server: ServerContext,
    ):
        generate_fn = get_generate_fn(
            model_path=case.server_args.model_path,
            modality=case.server_args.modality,
            sampling_params=case.sampling_params,
        )

        perf_record, content = self.run_and_collect(
            diffusion_server, case.id, generate_fn
        )

        assert len(content) > 0, "FP8 MLA generation produced empty output"
```
**EN:** This test method exercises diffusion generation and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 diffusion generation 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 133-136: Expose unittest entrypoint
```python
if __name__ == "__main__":
    import sys

    sys.exit(pytest.main([__file__, "-v"]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.multimodal_gen.test.server.test_server_common`, `sglang.multimodal_gen.test.server.test_server_utils`, `sglang.multimodal_gen.test.server.testcase_configs`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `__future__`, `logging`, `pytest`, `sys`
- Notable symbols / 关键符号: `register_amd_ci`
