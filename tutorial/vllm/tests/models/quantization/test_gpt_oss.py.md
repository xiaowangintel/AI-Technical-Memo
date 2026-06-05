# test_gpt_oss.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/quantization/test_gpt_oss.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers quantized model support and model-facing behavior. The file defines 1 test(s), 0 fixture(s), and 4 helper/class block(s) to validate this area. / [CN] 该文件覆盖量化模型支持与面向模型的行为。它定义了 1 个测试、0 个 fixture，以及 4 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L15-L25)
```python
import importlib.metadata
import importlib.util
from dataclasses import dataclass

import huggingface_hub
import lm_eval
import pytest
from packaging import version

from vllm.platforms import current_platform
from vllm.platforms.rocm import on_gfx950
```
**EN:** Imports standard-library modules such as `dataclasses.dataclass`, `importlib.metadata`, `importlib.util`, third-party packages like `huggingface_hub`, `lm_eval`, `packaging.version`, project helpers such as `vllm.platforms.current_platform`, `vllm.platforms.rocm.on_gfx950`.
**CN:** 导入标准库模块（如 `dataclasses.dataclass`、`importlib.metadata`、`importlib.util`）、第三方包（如 `huggingface_hub`、`lm_eval`、`packaging.version`）、项目内辅助模块（如 `vllm.platforms.current_platform`、`vllm.platforms.rocm.on_gfx950`）。

### Module setup / 模块级配置: MODEL_ACCURACIES, QUARK_MXFP4_AVAILABLE (L27-L38)
```python
MODEL_ACCURACIES = {
    # Full quantization: attention linears and MoE linears
    "amd/gpt-oss-20b-WFP8-AFP8-KVFP8": 0.89,
    # MoE linears only quantization
    "amd/gpt-oss-20b-MoE-Quant-W-MXFP4-A-FP8-KV-FP8": 0.89,
    # MoE linears only quantization
    # "amd/gpt-oss-20b-MoE-Quant-W-MXFP4-A-MXFP4-KV-FP8": 0.90,
}

QUARK_MXFP4_AVAILABLE = importlib.util.find_spec("quark") is not None and version.parse(
    importlib.metadata.version("amd-quark")
) >= version.parse("0.9.0")
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_ACCURACIES`, `QUARK_MXFP4_AVAILABLE`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_ACCURACIES`、`QUARK_MXFP4_AVAILABLE`。后续辅助函数和测试会复用这些值，以减少重复。

### Helper / 辅助函数: has_huggingface_access (L41-L46)
```python
def has_huggingface_access(repo):
    try:
        huggingface_hub.list_repo_refs(repo)
        return True
    except huggingface_hub.errors.RepositoryNotFoundError:
        return False
```
**EN:** This helper encapsulates reusable logic in `has_huggingface_access`. Key inputs are `repo`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `has_huggingface_access` 中。 关键输入包括 `repo`。 它把计算得到的状态或辅助对象返回给调用方。

### Module setup / 模块级配置: HF_HUB_AMD_ORG_ACCESS (L49-L51)
```python
HF_HUB_AMD_ORG_ACCESS = all(
    [has_huggingface_access(model_name) for model_name in MODEL_ACCURACIES]
)
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `HF_HUB_AMD_ORG_ACCESS`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `HF_HUB_AMD_ORG_ACCESS`。后续辅助函数和测试会复用这些值，以减少重复。

### Class / 类: ModelCase (L54-L57)
```python
@dataclass
class ModelCase:
    model_id: str
    tp: int
```
**EN:** This class groups related scenarios in `ModelCase`. Decorators such as `@dataclass` make it a compact metadata container. The body is mostly structural and does not define direct methods.
**CN:** 该类将与 `ModelCase` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 其主体主要是结构性定义，没有直接方法实现。

### Class / 类: EvaluationConfig (L60-L76)
```python
@dataclass
class EvaluationConfig:
    model_name: str

    def get_model_args(self, tp_size: int):
        return {
            "pretrained": self.model_name,
            "chat_template_args": {"reasoning_effort": "low"},
            "enable_thinking": True,
            "think_end_token": "200008",
            "tensor_parallel_size": tp_size,
            "dtype": "auto",
            "gpu_memory_utilization": 0.95,
            "trust_remote_code": False,
            "enable_prefix_caching": False,
            "enforce_eager": False,
        }
```
**EN:** This class groups related scenarios in `EvaluationConfig`. Decorators such as `@dataclass` make it a compact metadata container. It contains 0 test method(s) and 1 supporting method(s). Representative methods include `get_model_args`.
**CN:** 该类将与 `EvaluationConfig` 相关的场景组织在一起。 像 `@dataclass` 这样的装饰器使它成为紧凑的元数据容器。 它包含 0 个测试方法和 1 个辅助方法。 代表性方法包括 `get_model_args`。

### Helper method / 辅助方法: EvaluationConfig.get_model_args (L64-L76)
```python
    def get_model_args(self, tp_size: int):
        return {
            "pretrained": self.model_name,
            "chat_template_args": {"reasoning_effort": "low"},
            "enable_thinking": True,
            "think_end_token": "200008",
            "tensor_parallel_size": tp_size,
            "dtype": "auto",
            "gpu_memory_utilization": 0.95,
            "trust_remote_code": False,
            "enable_prefix_caching": False,
            "enforce_eager": False,
        }
```
**EN:** This helper encapsulates reusable logic in `EvaluationConfig.get_model_args`. Key inputs are `tp_size`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `EvaluationConfig.get_model_args` 中。 关键输入包括 `tp_size`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_gpt_oss_attention_quantization (L79-L122)
```python
@pytest.mark.skipif(not QUARK_MXFP4_AVAILABLE, reason="amd-quark>=0.9 is not available")
@pytest.mark.skipif(
    not HF_HUB_AMD_ORG_ACCESS,
    reason="Read access to huggingface.co/amd is required for this test.",
)
@pytest.mark.parametrize("tp_size", [1, 2, 4, 8])
@pytest.mark.parametrize("model_name, expected_accuracy", MODEL_ACCURACIES.items())
def test_gpt_oss_attention_quantization(
    model_name: str,
    tp_size: int,
    expected_accuracy: float,
    monkeypatch: pytest.MonkeyPatch,
):
    if tp_size > current_platform.device_count():
        pytest.skip("Not enough GPUs to run this test case")

    if "amd/gpt-oss-20b-MoE-Quant-W-MXFP4-A-FP8-KV-FP8" in model_name and on_gfx950():
        monkeypatch.setenv("VLLM_ROCM_USE_AITER", "1")
# ... 18 lines omitted for brevity ...
        lm_eval_out["results"]["gsm8k_platinum"]["exact_match,flexible-extract"]
    )

    rtol = 0.02
    assert measured_accuracy >= expected_accuracy - rtol, (
        f"Accuracy {measured_accuracy:.4f} is below threshold "
        f"{expected_accuracy - rtol:.4f} (expected >= {expected_accuracy} - {rtol})"
    )
```
**EN:** This test validates `test_gpt_oss_attention_quantization`. It uses parameterization over `tp_size`. Relevant pytest markers include `skipif`. Key inputs are `model_name`, `tp_size`, `expected_accuracy`, `monkeypatch`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. The logic can skip unsupported environments when prerequisites are not satisfied. The main assertion is `measured_accuracy >= expected_accuracy - rtol`.
**CN:** 这个测试验证 `test_gpt_oss_attention_quantization`。 它通过参数化组合 `tp_size`。 相关的 pytest 标记包括 `skipif`。 关键输入包括 `model_name`、`tp_size`、`expected_accuracy`、`monkeypatch`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 当前置条件不满足时，这段逻辑会跳过不受支持的环境。 核心断言是 `measured_accuracy >= expected_accuracy - rtol`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `dataclasses.dataclass`, `importlib.metadata`, `importlib.util`
- **Third-party / 第三方**: `huggingface_hub`, `lm_eval`, `packaging.version`, `pytest`
- **Project / 项目内**: `vllm.platforms.current_platform`, `vllm.platforms.rocm.on_gfx950`
