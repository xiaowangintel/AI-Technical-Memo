# test_sleep.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/entrypoints/serve/instrumentator/test_sleep.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers serve subsystem behavior. The file defines 1 test(s), 0 fixture(s), and 1 helper/class block(s) to validate this area. / [CN] 该文件覆盖服务子系统行为。它定义了 1 个测试、0 个 fixture，以及 1 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L4-L7)
```python
import requests
from prometheus_client.parser import text_string_to_metric_families

from tests.utils import RemoteOpenAIServer
```
**EN:** Imports third-party packages like `prometheus_client.parser.text_string_to_metric_families`, `requests`, project helpers such as `tests.utils.RemoteOpenAIServer`.
**CN:** 导入第三方包（如 `prometheus_client.parser.text_string_to_metric_families`、`requests`）、项目内辅助模块（如 `tests.utils.RemoteOpenAIServer`）。

### Module setup / 模块级配置: MODEL_NAME (L9-L9)
```python
MODEL_NAME = "meta-llama/Llama-3.2-1B"
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `MODEL_NAME`. Later helpers and tests reuse these values to keep scenarios concise.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `MODEL_NAME`。后续辅助函数和测试会复用这些值，以减少重复。

### Test / 测试: test_sleep_mode (L12-L86)
```python
def test_sleep_mode():
    # dtype, max-len etc set so that this can run in CI
    args = [
        "--dtype",
        "bfloat16",
        "--max-model-len",
        "8192",
        "--max-num-seqs",
        "128",
        "--enable-sleep-mode",
    ]

    with RemoteOpenAIServer(
        MODEL_NAME,
        args,
        env_dict={"VLLM_SERVER_DEV_MODE": "1", "CUDA_VISIBLE_DEVICES": "0"},
    ) as remote_server:
        response = requests.post(remote_server.url_for("sleep"), params={"level": "1"})
# ... 49 lines omitted for brevity ...

        # check sleep metrics
        response = requests.get(remote_server.url_for("metrics"))
        assert response.status_code == 200
        awake, weights_offloaded, discard_all = _get_sleep_metrics_from_api(response)
        assert awake == 1
        assert weights_offloaded == 0
        assert discard_all == 0
```
**EN:** This test validates `test_sleep_mode`. It launches or uses a `RemoteOpenAIServer` instance to exercise a served endpoint. The main assertion is `response.status_code == 200` and `response.status_code == 200`.
**CN:** 这个测试验证 `test_sleep_mode`。 它启动或使用 `RemoteOpenAIServer` 实例来覆盖真实服务端点。 核心断言是 `response.status_code == 200` and `response.status_code == 200`。

### Helper / 辅助函数: _get_sleep_metrics_from_api (L89-L110)
```python
def _get_sleep_metrics_from_api(response: requests.Response):
    """Return (awake, weights_offloaded, discard_all)"""

    awake, weights_offloaded, discard_all = None, None, None

    for family in text_string_to_metric_families(response.text):
        if family.name == "vllm:engine_sleep_state":
            for sample in family.samples:
                if sample.name == "vllm:engine_sleep_state":
                    for label_name, label_value in sample.labels.items():
                        if label_value == "awake":
                            awake = sample.value
                        elif label_value == "weights_offloaded":
                            weights_offloaded = sample.value
                        elif label_value == "discard_all":
                            discard_all = sample.value

    assert awake is not None
    assert weights_offloaded is not None
    assert discard_all is not None

    return awake, weights_offloaded, discard_all
```
**EN:** This helper encapsulates reusable logic in `_get_sleep_metrics_from_api`. Key inputs are `response`. It returns computed state or helper objects back to the caller. The main assertion is `awake is not None` and `weights_offloaded is not None`.
**CN:** 这个辅助函数将可复用逻辑封装在 `_get_sleep_metrics_from_api` 中。 关键输入包括 `response`。 它把计算得到的状态或辅助对象返回给调用方。 核心断言是 `awake is not None` and `weights_offloaded is not None`。

## Key Concepts / 关键概念
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Third-party / 第三方**: `prometheus_client.parser.text_string_to_metric_families`, `requests`
- **Project / 项目内**: `tests.utils.RemoteOpenAIServer`
