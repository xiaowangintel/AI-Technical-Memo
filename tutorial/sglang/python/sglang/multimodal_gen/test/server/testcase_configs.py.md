# testcase_configs.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/server/testcase_configs.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates testcase configs with focused assertions and fixtures. Key symbols include `ToleranceConfig`, `ScenarioConfig`, `BaselineConfig`. / 该测试模块通过有针对性的断言与夹具，验证 testcase configs 的实现。 关键符号包括 `ToleranceConfig`, `ScenarioConfig`, `BaselineConfig`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: Imports and module setup / 导入与模块初始化
```python
"""
Configuration and data structures for diffusion performance tests.

Usage:

pytest python/sglang/multimodal_gen/test/server/test_server_1_gpu.py
# for a single testcase, look for the name of the testcase in ONE_GPU_CASES,
# ONE_GPU_MODELOPT_FP8_CASES, ONE_GPU_B200_CASES, or TWO_GPU_CASES
pytest python/sglang/multimodal_gen/test/server/test_server_1_gpu.py -k qwen_image_t2i


To add a new testcase:
1. add your testcase with case-id: `my_new_test_case_id` to `ONE_GPU_CASES`, `ONE_GPU_MODELOPT_FP8_CASES`, `ONE_GPU_B200_CASES`, or `TWO_GPU_CASES`
2. run `SGLANG_GEN_BASELINE=1 pytest -s python/sglang/multimodal_gen/test/server/ -k my_new_test_case_id`
# ...

from sglang.multimodal_gen.configs.pipeline_configs.base import ModelTaskType
from sglang.multimodal_gen.registry import get_model_info
from sglang.multimodal_gen.runtime.utils.perf_logger import RequestPerfRecord
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 35-84: Class `ToleranceConfig` / 类 `ToleranceConfig`
```python
@dataclass
class ToleranceConfig:
    """Tolerance ratios for performance validation."""

    e2e: float
    denoise_stage: float
    non_denoise_stage: float
    denoise_step: float
    denoise_agg: float

    @classmethod
    def load_profile(cls, all_tolerances: dict, profile_name: str) -> ToleranceConfig:
        """Load a specific tolerance profile from a dictionary of profiles."""
        # Support both flat structure (backward compatibility) and profiled structure
# ...
            denoise_agg=float(
                os.getenv("SGLANG_DENOISE_AGG_TOLERANCE", tol_data["denoise_agg"])
            ),
        )
```
**EN:** This class models `ToleranceConfig`. Tolerance ratios for performance validation. Important methods include `load_profile`.
**CN:** 该类实现 `ToleranceConfig`。 文档字符串指出：Tolerance ratios for performance validation. 其中较重要的方法包括 `load_profile`。

### Lines 87-96: Class `ScenarioConfig` / 类 `ScenarioConfig`
```python
@dataclass
class ScenarioConfig:
    """Expected performance metrics for a test scenario."""

    stages_ms: dict[str, float]
    denoise_step_ms: dict[int, float]
    expected_e2e_ms: float
    expected_avg_denoise_ms: float
    expected_median_denoise_ms: float
    estimated_full_test_time_s: float | None = None
```
**EN:** This class models `ScenarioConfig`. Expected performance metrics for a test scenario.
**CN:** 该类实现 `ScenarioConfig`。 文档字符串指出：Expected performance metrics for a test scenario.

### Lines 99-157: Class `BaselineConfig` / 类 `BaselineConfig`
```python
@dataclass
class BaselineConfig:
    """Full baseline configuration."""

    scenarios: dict[str, ScenarioConfig]
    step_fractions: Sequence[float]
    tolerances: ToleranceConfig
    improvement_threshold: float

    @classmethod
    def load(cls, path: Path) -> BaselineConfig:
        """Load baseline configuration from JSON file."""
        with path.open("r", encoding="utf-8") as fh:
            data = json.load(fh)
# ...
            )

        self.scenarios.update(scenarios_new)
        return self
```
**EN:** This class models `BaselineConfig`. Full baseline configuration. Important methods include `load`, `update`.
**CN:** 该类实现 `BaselineConfig`。 文档字符串指出：Full baseline configuration. 其中较重要的方法包括 `load`, `update`。

### Lines 160-206: Class `DiffusionServerArgs` / 类 `DiffusionServerArgs`
```python
@dataclass
class DiffusionServerArgs:
    """Configuration for a single model/scenario test case."""

    model_path: str  # HF repo or local path
    modality: str | None = None  # auto-inferred: "image" or "video" or "3d"

    custom_validator: str | None = None  # auto-derived unless explicitly overridden
    # resources
    num_gpus: int = 1
    tp_size: int | None = None
    ulysses_degree: int | None = None
    ring_degree: int | None = None
    cfg_parallel: bool | None = None
# ...
        elif self.modality == "video":
            self.custom_validator = "video"
        elif self.modality == "3d":
            self.custom_validator = "mesh"
```
**EN:** This class models `DiffusionServerArgs`. Configuration for a single model/scenario test case. Important methods include `__post_init__`.
**CN:** 该类实现 `DiffusionServerArgs`。 文档字符串指出：Configuration for a single model/scenario test case. 其中较重要的方法包括 `__post_init__`。

### Lines 209-220: Function `_infer_modality_from_model_path` / 函数 `_infer_modality_from_model_path`
```python
@lru_cache(maxsize=None)
def _infer_modality_from_model_path(model_path: str) -> str:
    model_info = get_model_info(model_path)
    if model_info is None:
        raise ValueError(f"Could not resolve model info for {model_path!r}")

    task_type = model_info.pipeline_config_cls.task_type
    if task_type == ModelTaskType.I2M:
        return "3d"
    if task_type.is_image_gen():
        return "image"
    return "video"
```
**EN:** This function drives `_infer_modality_from_model_path` with inputs such as `model_path`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_infer_modality_from_model_path`，主要处理 `model_path` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 223-248: Class `DiffusionSamplingParams` / 类 `DiffusionSamplingParams`
```python
@dataclass(frozen=True)
class DiffusionSamplingParams:
    """Configuration for a single model/scenario test case."""

    output_size: str = ""

    # inputs and conditioning
    prompt: str | None = None  # text prompt for generation
    image_path: Path | str | None = None  # input image/video for editing (Path or URL)

    # duration
    seconds: int = 1  # for video: duration in seconds
    num_frames: int | None = None  # for video: number of frames
    fps: int | None = None  # for video: frames per second
# ...

    # Additional request-level parameters (e.g. enable_teacache, enable_upscaling, …)
    # merged directly into the OpenAI extra_body dict.
    extras: dict = field(default_factory=dict)
```
**EN:** This class models `DiffusionSamplingParams`. Configuration for a single model/scenario test case.
**CN:** 该类实现 `DiffusionSamplingParams`。 文档字符串指出：Configuration for a single model/scenario test case.

### Lines 251-291: Class `DiffusionTestCase` / 类 `DiffusionTestCase`
```python
@dataclass(frozen=True)
class DiffusionTestCase:
    """Configuration for a single model/scenario test case."""

    id: str  # pytest test id and scenario name
    server_args: DiffusionServerArgs
    sampling_params: DiffusionSamplingParams
    run_perf_check: bool = True
    run_consistency_check: bool = True
    run_component_accuracy_check: bool = True
    run_models_api_check: bool = True
    run_t2v_input_reference_check: bool = True
    run_lora_basic_api_check: bool = False
    run_lora_dynamic_load_check: bool = False
# ...
        if self.run_multi_lora_api_check and not (has_startup_lora and has_second_lora):
            raise ValueError(
                f"{self.id}: run_multi_lora_api_check requires lora_path and second_lora_path"
            )
```
**EN:** This class models `DiffusionTestCase`. Configuration for a single model/scenario test case. Important methods include `__post_init__`.
**CN:** 该类实现 `DiffusionTestCase`。 文档字符串指出：Configuration for a single model/scenario test case. 其中较重要的方法包括 `__post_init__`。

### Lines 294-305: Function `sample_step_indices` / 函数 `sample_step_indices`
```python
def sample_step_indices(
    step_map: dict[int, float], fractions: Sequence[float]
) -> list[int]:
    if not step_map:
        return []
    max_idx = max(step_map.keys())
    indices = set()
    for fraction in fractions:
        idx = min(max_idx, max(0, int(round(fraction * max_idx))))
        if idx in step_map:
            indices.add(idx)
    return sorted(indices)
```
**EN:** This function drives `sample_step_indices` with inputs such as `step_map`, `fractions`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `sample_step_indices`，主要处理 `step_map`, `fractions` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 308-357: Class `PerformanceSummary` / 类 `PerformanceSummary`
```python
@dataclass
class PerformanceSummary:
    """Summary of performance of a request, built from RequestPerfRecord"""

    e2e_ms: float
    avg_denoise_ms: float
    median_denoise_ms: float
    # { "stage_1": time_1, "stage_2": time_2 }
    stage_metrics: dict[str, float]
    step_metrics: list[float]
    sampled_steps: dict[int, float]
    all_denoise_steps: dict[int, float]
    frames_per_second: float | None = None
    total_frames: int | None = None
# ...
            step_metrics=step_durations,
            sampled_steps=sampled_steps,
            all_denoise_steps=per_step,
        )
```
**EN:** This class models `PerformanceSummary`. Summary of performance of a request, built from RequestPerfRecord Important methods include `from_req_perf_record`.
**CN:** 该类实现 `PerformanceSummary`。 文档字符串指出：Summary of performance of a request, built from RequestPerfRecord 其中较重要的方法包括 `from_req_perf_record`。

### Lines 358-457: Top-level configuration / 顶层配置
```python


T2I_sampling_params = DiffusionSamplingParams(
    prompt="Doraemon is eating dorayaki",
    output_size="1024x1024",
)

MODELOPT_T2I_CI_sampling_params = DiffusionSamplingParams(
    prompt="Doraemon is eating dorayaki",
    output_size="768x768",
    extras={"num_inference_steps": 12, "seed": 0},
)

MODELOPT_TI2I_CI_sampling_params = DiffusionSamplingParams(
# ...
MODELOPT_WAN22_NVFP4_TRANSFORMER = (
    "lmsys/wan22-t2v-a14b-modelopt-nvfp4-sglang-transformer"
)
MODELOPT_NVFP4_B200_ENV_VARS = {"SGLANG_DIFFUSION_FLASHINFER_FP4_GEMM_BACKEND": "cudnn"}
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 460-482: Function `_make_modelopt_ci_case` / 函数 `_make_modelopt_ci_case`
```python
def _make_modelopt_ci_case(
    case_id: str,
    *,
    model_path: str,
    modality: str,
    sampling_params: DiffusionSamplingParams,
    extras: list[str],
    env_vars: dict[str, str] | None = None,
) -> DiffusionTestCase:
    return DiffusionTestCase(
        case_id,
        DiffusionServerArgs(
            model_path=model_path,
            modality=modality,
# ...
        run_perf_check=False,
        run_consistency_check=False,
        run_component_accuracy_check=False,
    )
```
**EN:** This function drives `_make_modelopt_ci_case` with inputs such as `case_id`, `model_path`, `modality`, `sampling_params`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_make_modelopt_ci_case`，主要处理 `case_id`, `model_path`, `modality`, `sampling_params` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 485-491: Function `_with_default_num_gpus` / 函数 `_with_default_num_gpus`
```python
def _with_default_num_gpus(
    cases: list[DiffusionTestCase], num_gpus: int
) -> list[DiffusionTestCase]:
    return [
        replace(case, server_args=replace(case.server_args, num_gpus=num_gpus))
        for case in cases
    ]
```
**EN:** This function drives `_with_default_num_gpus` with inputs such as `cases`, `num_gpus`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_with_default_num_gpus`，主要处理 `cases`, `num_gpus` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 492-499: Top-level configuration / 顶层配置
```python


# Load global configuration
BASELINE_CONFIG = (
    BaselineConfig.load(Path(__file__).with_name("perf_baselines.json"))
    .update(Path(__file__).parent / "ascend" / "perf_baselines_npu.json")
    .update(Path(__file__).parent / "musa" / "perf_baselines_musa.json")
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- Pipeline orchestration / 流水线编排
- Caching strategy / 缓存策略
- Sampling parameter control / 采样参数控制
- Automated verification / 自动化验证
- Pytest-based assertions / 基于 Pytest 的断言

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.pipeline_configs.base`, `sglang.multimodal_gen.registry`, `sglang.multimodal_gen.runtime.utils.perf_logger`
- **External / 外部**: `__future__`
- **Stdlib / 标准库**: `json`, `os`, `statistics`, `dataclasses`, `functools`, `pathlib`, `typing`
