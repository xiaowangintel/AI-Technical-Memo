# test_formatter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/comparator/tensor_comparator/test_formatter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on tensor comparator formatter in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 tensor comparator formatter 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-59: Import dependencies
```python
import sys

import pytest
from registered.debug_utils.comparator.testing_helpers import (
    assert_rich_tags_balanced,
)
from registered.debug_utils.comparator.testing_helpers import make_diff as _make_diff
from registered.debug_utils.comparator.testing_helpers import make_stats as _make_stats
from registered.debug_utils.comparator.testing_helpers import (
    make_tensor_info as _make_tensor_info,
)

from sglang.srt.debug_utils.comparator.aligner.axis_aligner import AxisAlignerPlan
from sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types import (
    TracedAlignerPlan,
    TracedSidePlan,
    TracedStepPlan,
    TracedSubPlan,
)
from sglang.srt.debug_utils.comparator.aligner.entrypoint.types import (
    AlignerPerStepPlan,
    AlignerPlan,
)
from sglang.srt.debug_utils.comparator.aligner.reorderer.types import (
    ReordererPlan,
    ZigzagToNaturalParams,
)
from sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types import (
    TokenAlignerPlan,
    TokenLocator,
)
from sglang.srt.debug_utils.comparator.aligner.unsharder.types import (
    ConcatParams,
    UnsharderPlan,
)
from sglang.srt.debug_utils.comparator.dims_spec import ParallelAxis, TokenLayout
from sglang.srt.debug_utils.comparator.output_types import (
    BundleFileInfo,
    BundleSideInfo,
    ComparisonTensorRecord,
    ReplicatedCheckResult,
    ShapeSnapshot,
)
from sglang.srt.debug_utils.comparator.tensor_comparator.formatter import (
    _format_abs_diff_percentiles_rich,
    _format_bundle_section,
    _format_plan_section_rich,
    _format_stats_rich,
    format_comparison,
    format_comparison_rich,
    format_replicated_checks,
)
from sglang.srt.debug_utils.comparator.tensor_comparator.types import (
    DiffInfo,
    TensorComparisonInfo,
    TensorStats,
)
from sglang.srt.debug_utils.comparator.utils import Pair
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also checks concrete expectations with assertions.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会通过断言检查明确的预期。

### Lines 61-61: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 63-77: Define module constants
```python
_DEFAULT_PERCENTILE_LINES: list[str] = [
    "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]",
    "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]",
    "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]",
    "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]",
    "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]",
]

_DEFAULT_PERCENTILE_LINES: list[str] = [
    "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]",
    "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]",
    "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]",
    "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]",
    "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]",
]
```
**EN:** This block stores configuration values or reusable constants for the rest of the module.
**CN:** 该代码块保存模块其余部分会复用的配置值或常量。

### Lines 84-84: Define class TestFormatComparison
```python
class TestFormatComparison:
```
**EN:** This declaration introduces the `TestFormatComparison` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatComparison` 测试类，并说明它通过继承承担的职责。

### Lines 85-118: Run test: normal
```python
    def test_normal(self):
        info = TensorComparisonInfo(
            name="test",
            baseline=_make_tensor_info(
                stats=_make_stats(mean=0.1, std=1.0, min=-2.0, max=2.0),
            ),
            target=_make_tensor_info(
                stats=_make_stats(mean=0.1001, std=1.0001, min=-2.0001, max=2.0001),
            ),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(),
        )

        assert format_comparison(info) == (
            "Raw [shape] [4, 8] vs [4, 8]\t"
            "[dtype] torch.float32 vs torch.float32\n"
            "After unify [shape] [4, 8] vs [4, 8]\t"
            "[dtype] torch.float32 vs torch.float32\n"
            "[mean] 0.1000 vs 0.1001 (diff: 0.0001)\n"
            "[abs_mean] 0.8000 vs 0.8000 (diff: 0.0000)\n"
            "[std] 1.0000 vs 1.0001 (diff: 0.0001)\n"
            "[min] -2.0000 vs -2.0001 (diff: -0.0001)\n"
            "[max] 2.0000 vs 2.0001 (diff: 0.0001)\n"
            "[p1] -1.8000 vs -1.8000 (diff: 0.0000)\n"
            "[p5] -1.5000 vs -1.5000 (diff: 0.0000)\n"
            "[p50] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[p95] 1.5000 vs 1.5000 (diff: 0.0000)\n"
            "[p99] 1.8000 vs 1.8000 (diff: 0.0000)\n"
            "✅ rel_diff=0.0001\tmax_abs_diff=0.0005\tmean_abs_diff=0.0002\n"
            "max_abs_diff happens at coord=[2, 3] with "
            "baseline=1.0 target=1.0005\n"
            "[abs_diff] p1=0.0001 p5=0.0001 p50=0.0002 p95=0.0004 p99=0.0005"
        )
```
**EN:** This test method exercises normal and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 normal 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 120-145: Run test: shape mismatch
```python
    def test_shape_mismatch(self):
        info = TensorComparisonInfo(
            name="mismatch",
            baseline=_make_tensor_info(shape=[3, 4]),
            target=_make_tensor_info(shape=[5, 6]),
            unified_shape=[3, 4],
            shape_mismatch=True,
        )

        assert format_comparison(info) == (
            "Raw [shape] [3, 4] vs [5, 6]\t"
            "[dtype] torch.float32 vs torch.float32\n"
            "After unify [shape] [3, 4] vs [5, 6]\t"
            "[dtype] torch.float32 vs torch.float32\n"
            "[mean] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[abs_mean] 0.8000 vs 0.8000 (diff: 0.0000)\n"
            "[std] 1.0000 vs 1.0000 (diff: 0.0000)\n"
            "[min] -2.0000 vs -2.0000 (diff: 0.0000)\n"
            "[max] 2.0000 vs 2.0000 (diff: 0.0000)\n"
            "[p1] -1.8000 vs -1.8000 (diff: 0.0000)\n"
            "[p5] -1.5000 vs -1.5000 (diff: 0.0000)\n"
            "[p50] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[p95] 1.5000 vs 1.5000 (diff: 0.0000)\n"
            "[p99] 1.8000 vs 1.8000 (diff: 0.0000)\n"
            "⚠️ Shape mismatch"
        )
```
**EN:** This test method exercises shape mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 shape mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 147-187: Run test: with downcast
```python
    def test_with_downcast(self):
        info = TensorComparisonInfo(
            name="downcast",
            baseline=_make_tensor_info(),
            target=_make_tensor_info(dtype="torch.bfloat16"),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(
                rel_diff=0.002, max_abs_diff=0.005, mean_abs_diff=0.001, passed=False
            ),
            diff_downcast=_make_diff(
                rel_diff=0.0001, max_abs_diff=0.0005, mean_abs_diff=0.0002, passed=True
            ),
            downcast_dtype="torch.bfloat16",
        )

        assert format_comparison(info) == (
            "Raw [shape] [4, 8] vs [4, 8]\t"
            "[🟠dtype] torch.float32 vs torch.bfloat16\n"
            "After unify [shape] [4, 8] vs [4, 8]\t"
            "[dtype] torch.float32 vs torch.bfloat16\n"
            "[mean] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[abs_mean] 0.8000 vs 0.8000 (diff: 0.0000)\n"
            "[std] 1.0000 vs 1.0000 (diff: 0.0000)\n"
            "[min] -2.0000 vs -2.0000 (diff: 0.0000)\n"
            "[max] 2.0000 vs 2.0000 (diff: 0.0000)\n"
            "[p1] -1.8000 vs -1.8000 (diff: 0.0000)\n"
            "[p5] -1.5000 vs -1.5000 (diff: 0.0000)\n"
            "[p50] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[p95] 1.5000 vs 1.5000 (diff: 0.0000)\n"
            "[p99] 1.8000 vs 1.8000 (diff: 0.0000)\n"
            "❌ rel_diff=0.002\tmax_abs_diff=0.005\tmean_abs_diff=0.001\n"
            "max_abs_diff happens at coord=[2, 3] with "
            "baseline=1.0 target=1.0005\n"
            "[abs_diff] p1=0.0001 p5=0.0001 p50=0.0002 p95=0.0004 p99=0.0005\n"
            "When downcast to torch.bfloat16: "
            "✅ rel_diff=0.0001\tmax_abs_diff=0.0005\tmean_abs_diff=0.0002\n"
            "max_abs_diff happens at coord=[2, 3] with "
            "baseline=1.0 target=1.0005\n"
            "[abs_diff] p1=0.0001 p5=0.0001 p50=0.0002 p95=0.0004 p99=0.0005"
        )
```
**EN:** This test method exercises with downcast and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with downcast 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 189-220: Run test: with shape unification
```python
    def test_with_shape_unification(self):
        info = TensorComparisonInfo(
            name="unify",
            baseline=_make_tensor_info(shape=[1, 1, 4, 8]),
            target=_make_tensor_info(),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(),
        )

        assert format_comparison(info) == (
            "Raw [shape] [1, 1, 4, 8] vs [4, 8]\t"
            "[dtype] torch.float32 vs torch.float32\n"
            "Unify shape: [1, 1, 4, 8] -> [4, 8] "
            "(to match [4, 8])\n"
            "After unify [shape] [4, 8] vs [4, 8]\t"
            "[dtype] torch.float32 vs torch.float32\n"
            "[mean] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[abs_mean] 0.8000 vs 0.8000 (diff: 0.0000)\n"
            "[std] 1.0000 vs 1.0000 (diff: 0.0000)\n"
            "[min] -2.0000 vs -2.0000 (diff: 0.0000)\n"
            "[max] 2.0000 vs 2.0000 (diff: 0.0000)\n"
            "[p1] -1.8000 vs -1.8000 (diff: 0.0000)\n"
            "[p5] -1.5000 vs -1.5000 (diff: 0.0000)\n"
            "[p50] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[p95] 1.5000 vs 1.5000 (diff: 0.0000)\n"
            "[p99] 1.8000 vs 1.8000 (diff: 0.0000)\n"
            "✅ rel_diff=0.0001\tmax_abs_diff=0.0005\tmean_abs_diff=0.0002\n"
            "max_abs_diff happens at coord=[2, 3] with "
            "baseline=1.0 target=1.0005\n"
            "[abs_diff] p1=0.0001 p5=0.0001 p50=0.0002 p95=0.0004 p99=0.0005"
        )
```
**EN:** This test method exercises with shape unification and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with shape unification 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 222-253: Run test: with samples
```python
    def test_with_samples(self):
        info = TensorComparisonInfo(
            name="samples",
            baseline=_make_tensor_info(sample="tensor([0.1, 0.2, ...])"),
            target=_make_tensor_info(sample="tensor([0.1, 0.3, ...])"),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(),
        )

        assert format_comparison(info) == (
            "Raw [shape] [4, 8] vs [4, 8]\t"
            "[dtype] torch.float32 vs torch.float32\n"
            "After unify [shape] [4, 8] vs [4, 8]\t"
            "[dtype] torch.float32 vs torch.float32\n"
            "[mean] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[abs_mean] 0.8000 vs 0.8000 (diff: 0.0000)\n"
            "[std] 1.0000 vs 1.0000 (diff: 0.0000)\n"
            "[min] -2.0000 vs -2.0000 (diff: 0.0000)\n"
            "[max] 2.0000 vs 2.0000 (diff: 0.0000)\n"
            "[p1] -1.8000 vs -1.8000 (diff: 0.0000)\n"
            "[p5] -1.5000 vs -1.5000 (diff: 0.0000)\n"
            "[p50] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[p95] 1.5000 vs 1.5000 (diff: 0.0000)\n"
            "[p99] 1.8000 vs 1.8000 (diff: 0.0000)\n"
            "✅ rel_diff=0.0001\tmax_abs_diff=0.0005\tmean_abs_diff=0.0002\n"
            "max_abs_diff happens at coord=[2, 3] with "
            "baseline=1.0 target=1.0005\n"
            "[abs_diff] p1=0.0001 p5=0.0001 p50=0.0002 p95=0.0004 p99=0.0005\n"
            "x_baseline(sample)=tensor([0.1, 0.2, ...])\n"
            "x_target(sample)=tensor([0.1, 0.3, ...])"
        )
```
**EN:** This test method exercises with samples and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with samples 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 255-280: Run test: empty percentiles
```python
    def test_empty_percentiles(self):
        stats_no_quantiles = _make_stats(percentiles={})

        info = TensorComparisonInfo(
            name="no_quantiles",
            baseline=_make_tensor_info(stats=stats_no_quantiles),
            target=_make_tensor_info(stats=stats_no_quantiles),
            unified_shape=[4, 8],
            shape_mismatch=False,
            diff=_make_diff(abs_diff_percentiles={}),
        )

        assert format_comparison(info) == (
            "Raw [shape] [4, 8] vs [4, 8]\t"
            "[dtype] torch.float32 vs torch.float32\n"
            "After unify [shape] [4, 8] vs [4, 8]\t"
            "[dtype] torch.float32 vs torch.float32\n"
            "[mean] 0.0000 vs 0.0000 (diff: 0.0000)\n"
            "[abs_mean] 0.8000 vs 0.8000 (diff: 0.0000)\n"
            "[std] 1.0000 vs 1.0000 (diff: 0.0000)\n"
            "[min] -2.0000 vs -2.0000 (diff: 0.0000)\n"
            "[max] 2.0000 vs 2.0000 (diff: 0.0000)\n"
            "✅ rel_diff=0.0001\tmax_abs_diff=0.0005\tmean_abs_diff=0.0002\n"
            "max_abs_diff happens at coord=[2, 3] with "
            "baseline=1.0 target=1.0005"
        )
```
**EN:** This test method exercises empty percentiles and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty percentiles 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 283-309: Define helper: make comparison record
```python
def _make_comparison_record(
    name: str = "hidden_states",
    shape: list[int] | None = None,
    dtype: str = "torch.float32",
    diff: DiffInfo | None = None,
    shape_mismatch: bool = False,
    sample: str | None = None,
    diff_downcast: DiffInfo | None = None,
    downcast_dtype: str | None = None,
    replicated_checks: list[ReplicatedCheckResult] | None = None,
    raw_bundle_info: Pair[BundleSideInfo] | None = None,
    traced_plan: TracedAlignerPlan | None = None,
) -> ComparisonTensorRecord:
    s: list[int] = shape if shape is not None else [4, 8]
    return ComparisonTensorRecord(
        name=name,
        baseline=_make_tensor_info(shape=s, dtype=dtype, sample=sample),
        target=_make_tensor_info(shape=s, dtype=dtype, sample=sample),
        unified_shape=s,
        shape_mismatch=shape_mismatch,
        diff=diff,
        diff_downcast=diff_downcast,
        downcast_dtype=downcast_dtype,
        replicated_checks=replicated_checks or [],
        raw_bundle_info=raw_bundle_info,
        traced_plan=traced_plan,
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 312-326: Define helper: make bundle side info
```python
def _make_bundle_side_info(
    num_files: int = 2,
    shape: list[int] | None = None,
    dtype: str = "torch.float32",
    dims: str | None = None,
    with_parallel_info: bool = False,
) -> BundleSideInfo:
    s: list[int] = shape if shape is not None else [2, 4096]
    files: list[BundleFileInfo] = []
    for i in range(num_files):
        par: dict[str, str] | None = (
            {"tp": f"{i}/{num_files}"} if with_parallel_info else None
        )
        files.append(BundleFileInfo(shape=s, dtype=dtype, rank=i, parallel_info=par))
    return BundleSideInfo(num_files=num_files, files=files, dims=dims)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 329-383: Define helper: make simple aligner plan
```python
def _make_simple_aligner_plan(
    *,
    with_unsharder: bool = False,
    with_reorderer: bool = False,
    with_token_aligner: bool = False,
    with_axis_aligner: bool = False,
    axis_aligner_noop: bool = False,
) -> AlignerPlan:
    baseline_plans: list[AlignerPerStepPlan] = []
    target_plans: list[AlignerPerStepPlan] = []

    if with_unsharder:
        unsharder: UnsharderPlan = UnsharderPlan(
            axis=ParallelAxis.TP,
            params=ConcatParams(dim_name="h"),
            groups=[[0, 1]],
        )
        target_plans.append(
            AlignerPerStepPlan(
                step=0, input_object_indices=[0, 1], sub_plans=[unsharder]
            )
        )

    if with_reorderer:
        reorderer: ReordererPlan = ReordererPlan(
            params=ZigzagToNaturalParams(dim_name="s", cp_size=2),
        )
        target_plans.append(
            AlignerPerStepPlan(step=0, input_object_indices=[0], sub_plans=[reorderer])
        )

    token_aligner_plan: TokenAlignerPlan | None = None
    if with_token_aligner:
        token_aligner_plan = TokenAlignerPlan(
            locators=Pair(
                x=TokenLocator(steps=[0, 0, 0], token_index_in_step=[0, 1, 2]),
                y=TokenLocator(steps=[0, 0, 0], token_index_in_step=[0, 1, 2]),
            ),
            layouts=Pair(x=TokenLayout.T, y=TokenLayout.T),
        )

    axis_aligner_plan: AxisAlignerPlan | None = None
    if with_axis_aligner:
        if axis_aligner_noop:
            axis_aligner_plan = AxisAlignerPlan(pattern=Pair(x=None, y=None))
        else:
            axis_aligner_plan = AxisAlignerPlan(
                pattern=Pair(x="b s d -> s b d", y=None)
            )

    return AlignerPlan(
        per_step_plans=Pair(x=baseline_plans, y=target_plans),
        token_aligner_plan=token_aligner_plan,
        axis_aligner_plan=axis_aligner_plan,
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 386-427: Define helper: make traced plan
```python
def _make_traced_plan(
    plan: AlignerPlan,
    *,
    target_input_shapes: list[list[int]] | None = None,
    target_output_shapes: list[list[int]] | None = None,
) -> TracedAlignerPlan:
    """Build a TracedAlignerPlan by attaching snapshots to target sub_plans."""
    baseline_traced_steps: list[TracedStepPlan] = [
        TracedStepPlan(
            step=sp.step,
            input_object_indices=sp.input_object_indices,
            sub_plans=[TracedSubPlan(plan=sub) for sub in sp.sub_plans],
        )
        for sp in plan.per_step_plans.x
    ]

    target_traced_steps: list[TracedStepPlan] = []
    for sp in plan.per_step_plans.y:
        traced_subs: list[TracedSubPlan] = []
        for sub in sp.sub_plans:
            snapshot: ShapeSnapshot | None = None
            if target_input_shapes is not None or target_output_shapes is not None:
                snapshot = ShapeSnapshot(
                    input_shapes=target_input_shapes or [[2, 4096], [2, 4096]],
                    output_shapes=target_output_shapes or [[4, 4096]],
                )
            traced_subs.append(TracedSubPlan(plan=sub, snapshot=snapshot))
        target_traced_steps.append(
            TracedStepPlan(
                step=sp.step,
                input_object_indices=sp.input_object_indices,
                sub_plans=traced_subs,
            )
        )

    return TracedAlignerPlan(
        plan=plan,
        per_side=Pair(
            x=TracedSidePlan(step_plans=baseline_traced_steps),
            y=TracedSidePlan(step_plans=target_traced_steps),
        ),
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 435-435: Define class TestFormatComparisonRichMinimal
```python
class TestFormatComparisonRichMinimal:
```
**EN:** This declaration introduces the `TestFormatComparisonRichMinimal` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatComparisonRichMinimal` 测试类，并说明它通过继承承担的职责。

### Lines 436-436: Document the class `TestFormatComparisonRichMinimal`
```python
    """format_comparison_rich() with verbosity='minimal'."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFormatComparisonRichMinimal`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFormatComparisonRichMinimal`的设计意图。

### Lines 438-448: Run test: passed
```python
    def test_passed(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff(rel_diff=1e-4, passed=True),
        )
        result: str = format_comparison_rich(record, verbosity="minimal")
        assert_rich_tags_balanced(result)

        assert result == (
            "[green]✅[/] [bold green]hidden_states                 [/] "
            "rel_diff=1.00e-04"
        )
```
**EN:** This test method exercises passed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 passed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 450-460: Run test: failed
```python
    def test_failed(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff(rel_diff=0.5, passed=False),
        )
        result: str = format_comparison_rich(record, verbosity="minimal")
        assert_rich_tags_balanced(result)

        assert result == (
            "[red]❌[/] [bold red]hidden_states                 [/] "
            "rel_diff=5.00e-01"
        )
```
**EN:** This test method exercises failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 462-472: Run test: shape mismatch
```python
    def test_shape_mismatch(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            shape_mismatch=True,
        )
        result: str = format_comparison_rich(record, verbosity="minimal")
        assert_rich_tags_balanced(result)

        assert result == (
            "[red]❌[/] [bold red]hidden_states                 [/] "
            "[yellow]shape mismatch[/]"
        )
```
**EN:** This test method exercises shape mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 shape mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 474-479: Run test: no diff
```python
    def test_no_diff(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record()
        result: str = format_comparison_rich(record, verbosity="minimal")
        assert_rich_tags_balanced(result)

        assert result == ("[red]❌[/] [bold red]hidden_states                 [/]")
```
**EN:** This test method exercises no diff and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no diff 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 482-482: Define class TestFormatComparisonRichNormal
```python
class TestFormatComparisonRichNormal:
```
**EN:** This declaration introduces the `TestFormatComparisonRichNormal` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatComparisonRichNormal` 测试类，并说明它通过继承承担的职责。

### Lines 483-483: Document the class `TestFormatComparisonRichNormal`
```python
    """format_comparison_rich() with verbosity='normal'."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFormatComparisonRichNormal`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFormatComparisonRichNormal`的设计意图。

### Lines 485-507: Run test: passed
```python
    def test_passed(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff(rel_diff=1e-4, passed=True),
        )
        result: str = format_comparison_rich(record, verbosity="normal")
        assert_rich_tags_balanced(result)

        assert result == (
            "[green]✅[/] [bold green]hidden_states[/] [dim cyan]── float32  [4, 8][/]\n"
            "   [green]rel_diff=1.00e-04[/]  max_abs=5.00e-04  mean_abs=2.00e-04\n"
            "   [dim]Aligned[/]\n"
            "      [4, 8] vs [4, 8]   torch.float32 vs torch.float32\n"
            "   [dim]Stats[/]\n"
            "      [dim]             baseline       target       Δ[/]\n"
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]\n"
            "      [blue]range     [/] [-2.0000, 2.0000]   [-2.0000, 2.0000]\n"
            "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]\n"
            "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]"
        )
```
**EN:** This test method exercises passed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 passed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 509-536: Run test: failed
```python
    def test_failed(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff(
                rel_diff=0.5, max_abs_diff=1.0, mean_abs_diff=0.3, passed=False
            ),
        )
        result: str = format_comparison_rich(record, verbosity="normal")
        assert_rich_tags_balanced(result)

        assert result == (
            "[red]❌[/] [bold red]hidden_states[/] [dim cyan]── float32  [4, 8][/]\n"
            "   [bold red]rel_diff=5.00e-01[/]  max_abs=1.00e+00  mean_abs=3.00e-01\n"
            "   max_abs @ [2, 3]: baseline=1.0  target=1.0005\n"
            "   [dim]Aligned[/]\n"
            "      [4, 8] vs [4, 8]   torch.float32 vs torch.float32\n"
            "   [dim]Stats[/]\n"
            "      [dim]             baseline       target       Δ[/]\n"
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]\n"
            "      [blue]range     [/] [-2.0000, 2.0000]   [-2.0000, 2.0000]\n"
            "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]\n"
            "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]\n"
            "   [dim]Abs Diff Percentiles[/]\n"
            "      p1=1.00e-04  p5=1.00e-04  p50=2.00e-04  p95=4.00e-04  p99=5.00e-04"
        )
```
**EN:** This test method exercises failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 538-560: Run test: shape mismatch
```python
    def test_shape_mismatch(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            shape_mismatch=True,
        )
        result: str = format_comparison_rich(record, verbosity="normal")
        assert_rich_tags_balanced(result)

        assert result == (
            "[red]❌[/] [bold red]hidden_states[/] [dim cyan]── float32  [4, 8][/]\n"
            "   [yellow]⚠ Shape mismatch[/]\n"
            "   [dim]Aligned[/]\n"
            "      [4, 8] vs [4, 8]   torch.float32 vs torch.float32\n"
            "   [dim]Stats[/]\n"
            "      [dim]             baseline       target       Δ[/]\n"
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]\n"
            "      [blue]range     [/] [-2.0000, 2.0000]   [-2.0000, 2.0000]\n"
            "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]\n"
            "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]"
        )
```
**EN:** This test method exercises shape mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 shape mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 562-590: Run test: with downcast
```python
    def test_with_downcast(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff(rel_diff=0.01, passed=False),
            diff_downcast=_make_diff(rel_diff=1e-5, passed=True),
            downcast_dtype="torch.bfloat16",
        )
        result: str = format_comparison_rich(record, verbosity="normal")
        assert_rich_tags_balanced(result)

        assert result == (
            "[red]❌[/] [bold red]hidden_states[/] [dim cyan]── float32  [4, 8][/]\n"
            "   [bold red]rel_diff=1.00e-02[/]  max_abs=5.00e-04  mean_abs=2.00e-04\n"
            "   max_abs @ [2, 3]: baseline=1.0  target=1.0005\n"
            "   [green]✅[/] downcast to torch.bfloat16: rel_diff=1.00e-05\n"
            "   [dim]Aligned[/]\n"
            "      [4, 8] vs [4, 8]   torch.float32 vs torch.float32\n"
            "   [dim]Stats[/]\n"
            "      [dim]             baseline       target       Δ[/]\n"
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]\n"
            "      [blue]range     [/] [-2.0000, 2.0000]   [-2.0000, 2.0000]\n"
            "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]\n"
            "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]\n"
            "   [dim]Abs Diff Percentiles[/]\n"
            "      p1=1.00e-04  p5=1.00e-04  p50=2.00e-04  p95=4.00e-04  p99=5.00e-04"
        )
```
**EN:** This test method exercises with downcast and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with downcast 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 592-622: Run test: with bundle info
```python
    def test_with_bundle_info(self) -> None:
        bundle_info: Pair[BundleSideInfo] = Pair(
            x=_make_bundle_side_info(num_files=2, dims="b s h(tp) d"),
            y=_make_bundle_side_info(num_files=2, dims="b s h(tp) d"),
        )
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff(passed=True),
            raw_bundle_info=bundle_info,
        )
        result: str = format_comparison_rich(record, verbosity="normal")
        assert_rich_tags_balanced(result)

        assert result == (
            "[green]✅[/] [bold green]hidden_states[/] [dim cyan]── float32  [4, 8][/]\n"
            "   [green]rel_diff=1.00e-04[/]  max_abs=5.00e-04  mean_abs=2.00e-04\n"
            "   [dim]Bundle[/]\n"
            "      baseline  [cyan]2 files[/] × [2, 4096] float32  [dim]dims: b s h(tp) d[/]\n"
            "      target    [cyan]2 files[/] × [2, 4096] float32  [dim]dims: b s h(tp) d[/]\n"
            "   [dim]Aligned[/]\n"
            "      [4, 8] vs [4, 8]   torch.float32 vs torch.float32\n"
            "   [dim]Stats[/]\n"
            "      [dim]             baseline       target       Δ[/]\n"
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]\n"
            "      [blue]range     [/] [-2.0000, 2.0000]   [-2.0000, 2.0000]\n"
            "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]\n"
            "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]"
        )
```
**EN:** This test method exercises with bundle info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with bundle info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 624-651: Run test: with plan
```python
    def test_with_plan(self) -> None:
        plan: AlignerPlan = _make_simple_aligner_plan(with_unsharder=True)
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff(passed=True),
            traced_plan=_make_traced_plan(plan),
        )
        result: str = format_comparison_rich(record, verbosity="normal")
        assert_rich_tags_balanced(result)

        assert result == (
            "[green]✅[/] [bold green]hidden_states[/] [dim cyan]── float32  [4, 8][/]\n"
            "   [green]rel_diff=1.00e-04[/]  max_abs=5.00e-04  mean_abs=2.00e-04\n"
            "   [dim]Plan[/]\n"
            "      baseline  [dim](passthrough)[/]\n"
            "      target    [magenta]unsharder(tp)[/]\n"
            "   [dim]Aligned[/]\n"
            "      [4, 8] vs [4, 8]   torch.float32 vs torch.float32\n"
            "   [dim]Stats[/]\n"
            "      [dim]             baseline       target       Δ[/]\n"
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]\n"
            "      [blue]range     [/] [-2.0000, 2.0000]   [-2.0000, 2.0000]\n"
            "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]\n"
            "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]"
        )
```
**EN:** This test method exercises with plan and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with plan 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 654-654: Define class TestFormatComparisonRichVerbose
```python
class TestFormatComparisonRichVerbose:
```
**EN:** This declaration introduces the `TestFormatComparisonRichVerbose` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatComparisonRichVerbose` 测试类，并说明它通过继承承担的职责。

### Lines 655-655: Document the class `TestFormatComparisonRichVerbose`
```python
    """format_comparison_rich() with verbosity='verbose'."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFormatComparisonRichVerbose`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFormatComparisonRichVerbose`的设计意图。

### Lines 657-687: Run test: passed full detail
```python
    def test_passed_full_detail(self) -> None:
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff(rel_diff=1e-4, passed=True),
            sample="tensor([0.1, 0.2, ...])",
        )
        result: str = format_comparison_rich(record, verbosity="verbose")
        assert_rich_tags_balanced(result)

        assert result == (
            "[green]✅[/] [bold green]hidden_states[/] [dim cyan]── float32  [4, 8][/]\n"
            "   [green]rel_diff=1.00e-04[/]  max_abs=5.00e-04  mean_abs=2.00e-04\n"
            "   [dim]Aligned[/]\n"
            "      [4, 8] vs [4, 8]   torch.float32 vs torch.float32\n"
            "   [dim]Stats[/]\n"
            "      [dim]             baseline       target       Δ[/]\n"
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]abs_mean  [/]     0.8000       0.8000   [dim]+0.00e+00[/]\n"
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]\n"
            "      [blue]min       [/]    -2.0000      -2.0000   [dim]+0.00e+00[/]\n"
            "      [blue]max       [/]     2.0000       2.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]\n"
            "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]\n"
            "   [dim]Abs Diff Percentiles[/]\n"
            "      p1=1.00e-04  p5=1.00e-04  p50=2.00e-04  p95=4.00e-04  p99=5.00e-04\n"
            "   [dim]Samples[/]\n"
            "      baseline  tensor([0.1, 0.2, ...])\n"
            "      target    tensor([0.1, 0.2, ...])"
        )
```
**EN:** This test method exercises passed full detail and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 passed full detail 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 689-727: Run test: with bundle verbose
```python
    def test_with_bundle_verbose(self) -> None:
        bundle_info: Pair[BundleSideInfo] = Pair(
            x=_make_bundle_side_info(num_files=2, with_parallel_info=True),
            y=_make_bundle_side_info(num_files=2, with_parallel_info=True),
        )
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff(passed=True),
            raw_bundle_info=bundle_info,
        )
        result: str = format_comparison_rich(record, verbosity="verbose")
        assert_rich_tags_balanced(result)

        assert result == (
            "[green]✅[/] [bold green]hidden_states[/] [dim cyan]── float32  [4, 8][/]\n"
            "   [green]rel_diff=1.00e-04[/]  max_abs=5.00e-04  mean_abs=2.00e-04\n"
            "   [dim]Bundle[/]\n"
            "      baseline  [cyan]2 files[/] float32\n"
            "         [0] [2, 4096]  rank=0 tp=0/2\n"
            "         [1] [2, 4096]  rank=1 tp=1/2\n"
            "      target    [cyan]2 files[/] float32\n"
            "         [0] [2, 4096]  rank=0 tp=0/2\n"
            "         [1] [2, 4096]  rank=1 tp=1/2\n"
            "   [dim]Aligned[/]\n"
            "      [4, 8] vs [4, 8]   torch.float32 vs torch.float32\n"
            "   [dim]Stats[/]\n"
            "      [dim]             baseline       target       Δ[/]\n"
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]abs_mean  [/]     0.8000       0.8000   [dim]+0.00e+00[/]\n"
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]\n"
            "      [blue]min       [/]    -2.0000      -2.0000   [dim]+0.00e+00[/]\n"
            "      [blue]max       [/]     2.0000       2.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]\n"
            "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]\n"
            "   [dim]Abs Diff Percentiles[/]\n"
            "      p1=1.00e-04  p5=1.00e-04  p50=2.00e-04  p95=4.00e-04  p99=5.00e-04"
        )
```
**EN:** This test method exercises with bundle verbose and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with bundle verbose 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 729-764: Run test: with plan and traces
```python
    def test_with_plan_and_traces(self) -> None:
        plan: AlignerPlan = _make_simple_aligner_plan(with_unsharder=True)
        record: ComparisonTensorRecord = _make_comparison_record(
            diff=_make_diff(passed=True),
            traced_plan=_make_traced_plan(
                plan,
                target_input_shapes=[[2, 4096], [2, 4096]],
                target_output_shapes=[[4, 4096]],
            ),
        )
        result: str = format_comparison_rich(record, verbosity="verbose")
        assert_rich_tags_balanced(result)

        assert result == (
            "[green]✅[/] [bold green]hidden_states[/] [dim cyan]── float32  [4, 8][/]\n"
            "   [green]rel_diff=1.00e-04[/]  max_abs=5.00e-04  mean_abs=2.00e-04\n"
            "   [dim]Plan[/]\n"
            "      baseline  [dim](passthrough)[/]\n"
            "      target    [magenta]unsharder(tp)[/] (2×[2, 4096] → 1×[4, 4096])\n"
            "   [dim]Aligned[/]\n"
            "      [4, 8] vs [4, 8]   torch.float32 vs torch.float32\n"
            "   [dim]Stats[/]\n"
            "      [dim]             baseline       target       Δ[/]\n"
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]abs_mean  [/]     0.8000       0.8000   [dim]+0.00e+00[/]\n"
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]\n"
            "      [blue]min       [/]    -2.0000      -2.0000   [dim]+0.00e+00[/]\n"
            "      [blue]max       [/]     2.0000       2.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]\n"
            "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]\n"
            "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]\n"
            "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]\n"
            "   [dim]Abs Diff Percentiles[/]\n"
            "      p1=1.00e-04  p5=1.00e-04  p50=2.00e-04  p95=4.00e-04  p99=5.00e-04"
        )
```
**EN:** This test method exercises with plan and traces and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with plan and traces 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 767-767: Define class TestFormatBundleSection
```python
class TestFormatBundleSection:
```
**EN:** This declaration introduces the `TestFormatBundleSection` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatBundleSection` 测试类，并说明它通过继承承担的职责。

### Lines 768-768: Document the class `TestFormatBundleSection`
```python
    """_format_bundle_section() snapshot tests."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFormatBundleSection`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFormatBundleSection`的设计意图。

### Lines 770-780: Run test: single shape
```python
    def test_single_shape(self) -> None:
        bundle: Pair[BundleSideInfo] = Pair(
            x=_make_bundle_side_info(num_files=2, shape=[2, 4096]),
            y=_make_bundle_side_info(num_files=2, shape=[2, 4096]),
        )
        lines: list[str] = _format_bundle_section(bundle)

        assert lines == [
            "      baseline  [cyan]2 files[/] × [2, 4096] float32",
            "      target    [cyan]2 files[/] × [2, 4096] float32",
        ]
```
**EN:** This test method exercises single shape and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single shape 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 782-796: Run test: mixed shapes
```python
    def test_mixed_shapes(self) -> None:
        side: BundleSideInfo = BundleSideInfo(
            num_files=2,
            files=[
                BundleFileInfo(shape=[2, 4096], dtype="torch.float32", rank=0),
                BundleFileInfo(shape=[3, 4096], dtype="torch.float32", rank=1),
            ],
        )
        bundle: Pair[BundleSideInfo] = Pair(x=side, y=side)
        lines: list[str] = _format_bundle_section(bundle)

        assert lines == [
            "      baseline  [cyan]2 files[/] × mixed shapes float32",
            "      target    [cyan]2 files[/] × mixed shapes float32",
        ]
```
**EN:** This test method exercises mixed shapes and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 mixed shapes 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 798-806: Run test: no files
```python
    def test_no_files(self) -> None:
        empty: BundleSideInfo = BundleSideInfo(num_files=0, files=[])
        bundle: Pair[BundleSideInfo] = Pair(x=empty, y=empty)
        lines: list[str] = _format_bundle_section(bundle)

        assert lines == [
            "      baseline  [dim](no files)[/]",
            "      target    [dim](no files)[/]",
        ]
```
**EN:** This test method exercises no files and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no files 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 808-818: Run test: with dims
```python
    def test_with_dims(self) -> None:
        bundle: Pair[BundleSideInfo] = Pair(
            x=_make_bundle_side_info(num_files=1, dims="b s h(tp) d"),
            y=_make_bundle_side_info(num_files=1, dims="b s h(tp) d"),
        )
        lines: list[str] = _format_bundle_section(bundle)

        assert lines == [
            "      baseline  [cyan]1 files[/] × [2, 4096] float32  [dim]dims: b s h(tp) d[/]",
            "      target    [cyan]1 files[/] × [2, 4096] float32  [dim]dims: b s h(tp) d[/]",
        ]
```
**EN:** This test method exercises with dims and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with dims 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 821-821: Define class TestFormatBundleSectionVerbose
```python
class TestFormatBundleSectionVerbose:
```
**EN:** This declaration introduces the `TestFormatBundleSectionVerbose` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatBundleSectionVerbose` 测试类，并说明它通过继承承担的职责。

### Lines 822-822: Document the class `TestFormatBundleSectionVerbose`
```python
    """_format_bundle_section(verbose=True) snapshot tests."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFormatBundleSectionVerbose`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFormatBundleSectionVerbose`的设计意图。

### Lines 824-838: Run test: per file listing
```python
    def test_per_file_listing(self) -> None:
        bundle: Pair[BundleSideInfo] = Pair(
            x=_make_bundle_side_info(num_files=2, with_parallel_info=True),
            y=_make_bundle_side_info(num_files=2, with_parallel_info=True),
        )
        lines: list[str] = _format_bundle_section(bundle, verbose=True)

        assert lines == [
            "      baseline  [cyan]2 files[/] float32",
            "         [0] [2, 4096]  rank=0 tp=0/2",
            "         [1] [2, 4096]  rank=1 tp=1/2",
            "      target    [cyan]2 files[/] float32",
            "         [0] [2, 4096]  rank=0 tp=0/2",
            "         [1] [2, 4096]  rank=1 tp=1/2",
        ]
```
**EN:** This test method exercises per file listing and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 per file listing 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 840-848: Run test: no files
```python
    def test_no_files(self) -> None:
        empty: BundleSideInfo = BundleSideInfo(num_files=0, files=[])
        bundle: Pair[BundleSideInfo] = Pair(x=empty, y=empty)
        lines: list[str] = _format_bundle_section(bundle, verbose=True)

        assert lines == [
            "      baseline  [dim](no files)[/]",
            "      target    [dim](no files)[/]",
        ]
```
**EN:** This test method exercises no files and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no files 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 851-851: Define class TestFormatPlanSectionRich
```python
class TestFormatPlanSectionRich:
```
**EN:** This declaration introduces the `TestFormatPlanSectionRich` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatPlanSectionRich` 测试类，并说明它通过继承承担的职责。

### Lines 852-852: Document the class `TestFormatPlanSectionRich`
```python
    """_format_plan_section_rich() snapshot tests."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFormatPlanSectionRich`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFormatPlanSectionRich`的设计意图。

### Lines 854-862: Run test: passthrough
```python
    def test_passthrough(self) -> None:
        plan: AlignerPlan = _make_simple_aligner_plan()
        traced: TracedAlignerPlan = _make_traced_plan(plan)
        lines: list[str] = _format_plan_section_rich(traced_plan=traced)

        assert lines == [
            "      baseline  [dim](passthrough)[/]",
            "      target    [dim](passthrough)[/]",
        ]
```
**EN:** This test method exercises passthrough and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 passthrough 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 864-872: Run test: unsharder op
```python
    def test_unsharder_op(self) -> None:
        plan: AlignerPlan = _make_simple_aligner_plan(with_unsharder=True)
        traced: TracedAlignerPlan = _make_traced_plan(plan)
        lines: list[str] = _format_plan_section_rich(traced_plan=traced)

        assert lines == [
            "      baseline  [dim](passthrough)[/]",
            "      target    [magenta]unsharder(tp)[/]",
        ]
```
**EN:** This test method exercises unsharder op and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 unsharder op 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 874-882: Run test: reorderer op
```python
    def test_reorderer_op(self) -> None:
        plan: AlignerPlan = _make_simple_aligner_plan(with_reorderer=True)
        traced: TracedAlignerPlan = _make_traced_plan(plan)
        lines: list[str] = _format_plan_section_rich(traced_plan=traced)

        assert lines == [
            "      baseline  [dim](passthrough)[/]",
            "      target    [magenta]reorderer(zigzag_to_natural)[/]",
        ]
```
**EN:** This test method exercises reorderer op and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reorderer op 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 884-896: Run test: with shape traces
```python
    def test_with_shape_traces(self) -> None:
        plan: AlignerPlan = _make_simple_aligner_plan(with_unsharder=True)
        traced: TracedAlignerPlan = _make_traced_plan(
            plan,
            target_input_shapes=[[2, 4096], [2, 4096]],
            target_output_shapes=[[4, 4096]],
        )
        lines: list[str] = _format_plan_section_rich(traced_plan=traced)

        assert lines == [
            "      baseline  [dim](passthrough)[/]",
            "      target    [magenta]unsharder(tp)[/] (2×[2, 4096] → 1×[4, 4096])",
        ]
```
**EN:** This test method exercises with shape traces and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with shape traces 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 898-907: Run test: with token aligner
```python
    def test_with_token_aligner(self) -> None:
        plan: AlignerPlan = _make_simple_aligner_plan(with_token_aligner=True)
        traced: TracedAlignerPlan = _make_traced_plan(plan)
        lines: list[str] = _format_plan_section_rich(traced_plan=traced)

        assert lines == [
            "      baseline  [dim](passthrough)[/]",
            "      target    [dim](passthrough)[/]",
            "      token_aligner  [dim]3 tokens[/]",
        ]
```
**EN:** This test method exercises with token aligner and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with token aligner 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 909-918: Run test: with axis aligner
```python
    def test_with_axis_aligner(self) -> None:
        plan: AlignerPlan = _make_simple_aligner_plan(with_axis_aligner=True)
        traced: TracedAlignerPlan = _make_traced_plan(plan)
        lines: list[str] = _format_plan_section_rich(traced_plan=traced)

        assert lines == [
            "      baseline  [dim](passthrough)[/]",
            "      target    [dim](passthrough)[/]",
            "      axis_aligner  [dim]x=b s d -> s b d[/]",
        ]
```
**EN:** This test method exercises with axis aligner and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 with axis aligner 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 920-931: Run test: axis aligner noop
```python
    def test_axis_aligner_noop(self) -> None:
        plan: AlignerPlan = _make_simple_aligner_plan(
            with_axis_aligner=True, axis_aligner_noop=True
        )
        traced: TracedAlignerPlan = _make_traced_plan(plan)
        lines: list[str] = _format_plan_section_rich(traced_plan=traced)

        assert lines == [
            "      baseline  [dim](passthrough)[/]",
            "      target    [dim](passthrough)[/]",
            "      axis_aligner  [dim](no-op)[/]",
        ]
```
**EN:** This test method exercises axis aligner noop and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 axis aligner noop 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 934-934: Define class TestFormatStatsRich
```python
class TestFormatStatsRich:
```
**EN:** This declaration introduces the `TestFormatStatsRich` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatStatsRich` 测试类，并说明它通过继承承担的职责。

### Lines 935-935: Document the class `TestFormatStatsRich`
```python
    """_format_stats_rich() snapshot tests."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFormatStatsRich`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFormatStatsRich`的设计意图。

### Lines 937-951: Run test: basic
```python
    def test_basic(self) -> None:
        baseline: TensorStats = _make_stats(mean=0.0, std=1.0, min=-2.0, max=2.0)
        target: TensorStats = _make_stats(
            mean=0.0001, std=1.0001, min=-2.0001, max=2.0001
        )
        lines: list[str] = _format_stats_rich(baseline=baseline, target=target)
        assert_rich_tags_balanced("\n".join(lines))

        assert lines == [
            "      [dim]             baseline       target       Δ[/]",
            "      [blue]mean      [/]     0.0000       0.0001   [dim]+1.00e-04[/]",
            "      [blue]std       [/]     1.0000       1.0001   [dim]+1.00e-04[/]",
            "      [blue]range     [/] [-2.0000, 2.0000]   [-2.0001, 2.0001]",
            *_DEFAULT_PERCENTILE_LINES,
        ]
```
**EN:** This test method exercises basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 953-965: Run test: large delta
```python
    def test_large_delta(self) -> None:
        baseline: TensorStats = _make_stats(mean=0.0)
        target: TensorStats = _make_stats(mean=1.0)
        lines: list[str] = _format_stats_rich(baseline=baseline, target=target)
        assert_rich_tags_balanced("\n".join(lines))

        assert lines == [
            "      [dim]             baseline       target       Δ[/]",
            "      [blue]mean      [/]     0.0000       1.0000   [yellow]+1.00e+00[/]",
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]",
            "      [blue]range     [/] [-2.0000, 2.0000]   [-2.0000, 2.0000]",
            *_DEFAULT_PERCENTILE_LINES,
        ]
```
**EN:** This test method exercises large delta and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 large delta 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 967-979: Run test: small delta
```python
    def test_small_delta(self) -> None:
        baseline: TensorStats = _make_stats(mean=0.0)
        target: TensorStats = _make_stats(mean=0.001)
        lines: list[str] = _format_stats_rich(baseline=baseline, target=target)
        assert_rich_tags_balanced("\n".join(lines))

        assert lines == [
            "      [dim]             baseline       target       Δ[/]",
            "      [blue]mean      [/]     0.0000       0.0010   [dim]+1.00e-03[/]",
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]",
            "      [blue]range     [/] [-2.0000, 2.0000]   [-2.0000, 2.0000]",
            *_DEFAULT_PERCENTILE_LINES,
        ]
```
**EN:** This test method exercises small delta and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 small delta 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 982-982: Define class TestFormatStatsRichVerbose
```python
class TestFormatStatsRichVerbose:
```
**EN:** This declaration introduces the `TestFormatStatsRichVerbose` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatStatsRichVerbose` 测试类，并说明它通过继承承担的职责。

### Lines 983-983: Document the class `TestFormatStatsRichVerbose`
```python
    """_format_stats_rich(verbose=True) snapshot tests."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFormatStatsRichVerbose`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFormatStatsRichVerbose`的设计意图。

### Lines 985-1004: Run test: all stats with percentiles
```python
    def test_all_stats_with_percentiles(self) -> None:
        baseline: TensorStats = _make_stats()
        target: TensorStats = _make_stats()
        lines: list[str] = _format_stats_rich(
            baseline=baseline, target=target, verbose=True
        )

        assert lines == [
            "      [dim]             baseline       target       Δ[/]",
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]",
            "      [blue]abs_mean  [/]     0.8000       0.8000   [dim]+0.00e+00[/]",
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]",
            "      [blue]min       [/]    -2.0000      -2.0000   [dim]+0.00e+00[/]",
            "      [blue]max       [/]     2.0000       2.0000   [dim]+0.00e+00[/]",
            "      [blue]p1        [/]    -1.8000      -1.8000   [dim]+0.00e+00[/]",
            "      [blue]p5        [/]    -1.5000      -1.5000   [dim]+0.00e+00[/]",
            "      [blue]p50       [/]     0.0000       0.0000   [dim]+0.00e+00[/]",
            "      [blue]p95       [/]     1.5000       1.5000   [dim]+0.00e+00[/]",
            "      [blue]p99       [/]     1.8000       1.8000   [dim]+0.00e+00[/]",
        ]
```
**EN:** This test method exercises all stats with percentiles and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all stats with percentiles 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1006-1020: Run test: no percentiles
```python
    def test_no_percentiles(self) -> None:
        baseline: TensorStats = _make_stats(percentiles={})
        target: TensorStats = _make_stats(percentiles={})
        lines: list[str] = _format_stats_rich(
            baseline=baseline, target=target, verbose=True
        )

        assert lines == [
            "      [dim]             baseline       target       Δ[/]",
            "      [blue]mean      [/]     0.0000       0.0000   [dim]+0.00e+00[/]",
            "      [blue]abs_mean  [/]     0.8000       0.8000   [dim]+0.00e+00[/]",
            "      [blue]std       [/]     1.0000       1.0000   [dim]+0.00e+00[/]",
            "      [blue]min       [/]    -2.0000      -2.0000   [dim]+0.00e+00[/]",
            "      [blue]max       [/]     2.0000       2.0000   [dim]+0.00e+00[/]",
        ]
```
**EN:** This test method exercises no percentiles and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no percentiles 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1023-1023: Define class TestFormatAbsDiffPercentilesRich
```python
class TestFormatAbsDiffPercentilesRich:
```
**EN:** This declaration introduces the `TestFormatAbsDiffPercentilesRich` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatAbsDiffPercentilesRich` 测试类，并说明它通过继承承担的职责。

### Lines 1024-1024: Document the class `TestFormatAbsDiffPercentilesRich`
```python
    """_format_abs_diff_percentiles_rich() snapshot tests."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFormatAbsDiffPercentilesRich`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFormatAbsDiffPercentilesRich`的设计意图。

### Lines 1026-1032: Run test: normal values
```python
    def test_normal_values(self) -> None:
        diff: DiffInfo = _make_diff()
        result: str = _format_abs_diff_percentiles_rich(diff)

        assert result == (
            "p1=1.00e-04  p5=1.00e-04  p50=2.00e-04  " "p95=4.00e-04  p99=5.00e-04"
        )
```
**EN:** This test method exercises normal values and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 normal values 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1034-1040: Run test: high p99 coloring
```python
    def test_high_p99_coloring(self) -> None:
        diff: DiffInfo = _make_diff(
            abs_diff_percentiles={99: 0.5},
        )
        result: str = _format_abs_diff_percentiles_rich(diff)

        assert result == "[yellow]p99=5.00e-01[/]"
```
**EN:** This test method exercises high p99 coloring and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 high p99 coloring 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1042-1048: Run test: low p99 no coloring
```python
    def test_low_p99_no_coloring(self) -> None:
        diff: DiffInfo = _make_diff(
            abs_diff_percentiles={99: 0.01},
        )
        result: str = _format_abs_diff_percentiles_rich(diff)

        assert result == "p99=1.00e-02"
```
**EN:** This test method exercises low p99 no coloring and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 low p99 no coloring 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1051-1051: Define class TestFormatReplicatedChecks
```python
class TestFormatReplicatedChecks:
```
**EN:** This declaration introduces the `TestFormatReplicatedChecks` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestFormatReplicatedChecks` 测试类，并说明它通过继承承担的职责。

### Lines 1052-1052: Document the class `TestFormatReplicatedChecks`
```python
    """format_replicated_checks() snapshot tests."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestFormatReplicatedChecks`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestFormatReplicatedChecks`的设计意图。

### Lines 1054-1072: Run test: all passed
```python
    def test_all_passed(self) -> None:
        checks: list[ReplicatedCheckResult] = [
            ReplicatedCheckResult(
                axis="tp",
                group_index=0,
                compared_index=1,
                baseline_index=0,
                passed=True,
                atol=1e-3,
                diff=_make_diff(rel_diff=1e-6, max_abs_diff=1e-5, mean_abs_diff=1e-6),
            ),
        ]
        result: str = format_replicated_checks(checks)

        assert result == (
            "Replicated checks:\n"
            "  ✅ axis=tp group=0 idx=1 vs 0: "
            "rel_diff=1.000000e-06 max_abs_diff=1.000000e-05 mean_abs_diff=1.000000e-06"
        )
```
**EN:** This test method exercises all passed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all passed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1074-1092: Run test: one failed
```python
    def test_one_failed(self) -> None:
        checks: list[ReplicatedCheckResult] = [
            ReplicatedCheckResult(
                axis="tp",
                group_index=0,
                compared_index=1,
                baseline_index=0,
                passed=False,
                atol=1e-3,
                diff=_make_diff(rel_diff=0.5, max_abs_diff=1.0, mean_abs_diff=0.3),
            ),
        ]
        result: str = format_replicated_checks(checks)

        assert result == (
            "Replicated checks:\n"
            "  ❌ axis=tp group=0 idx=1 vs 0: "
            "rel_diff=5.000000e-01 max_abs_diff=1.000000e+00 mean_abs_diff=3.000000e-01"
        )
```
**EN:** This test method exercises one failed and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 one failed 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1094-1109: Run test: no diff
```python
    def test_no_diff(self) -> None:
        checks: list[ReplicatedCheckResult] = [
            ReplicatedCheckResult(
                axis="tp",
                group_index=0,
                compared_index=1,
                baseline_index=0,
                passed=True,
                atol=1e-3,
            ),
        ]
        result: str = format_replicated_checks(checks)

        assert result == (
            "Replicated checks:\n" "  ✅ axis=tp group=0 idx=1 vs 0: n/a diff"
        )
```
**EN:** This test method exercises no diff and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no diff 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1112-1113: Expose unittest entrypoint
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Assertions encode the behavioral contract that the scenario must satisfy. / 断言编码了该场景必须满足的行为契约。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.comparator.aligner.axis_aligner`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.traced_types`, `sglang.srt.debug_utils.comparator.aligner.entrypoint.types`, `sglang.srt.debug_utils.comparator.aligner.reorderer.types`, `sglang.srt.debug_utils.comparator.aligner.token_aligner.smart.types`, `sglang.srt.debug_utils.comparator.aligner.unsharder.types`, `sglang.srt.debug_utils.comparator.dims_spec`, `sglang.srt.debug_utils.comparator.output_types`, `sglang.srt.debug_utils.comparator.tensor_comparator.formatter`, `sglang.srt.debug_utils.comparator.tensor_comparator.types`, `sglang.srt.debug_utils.comparator.utils`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pytest`, `registered.debug_utils.comparator.testing_helpers`, `sys`
- Notable symbols / 关键符号: None / 无
