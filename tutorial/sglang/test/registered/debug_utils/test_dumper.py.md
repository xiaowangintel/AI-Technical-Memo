# test_dumper.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/test_dumper.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on debug utils dumper in SGLang. It prepares the runtime needed by the scenario and checks the resulting behavior with automated assertions. / 该测试模块用于分析 SGLang 中与 debug utils dumper 相关的实现或行为。 它会准备场景所需的运行环境，并通过自动化断言检查最终行为。

## Line-by-Line Analysis / 逐行分析
### Lines 1-50: Import dependencies
```python
import io
import multiprocessing
import os
import re
import sys
import threading
import time
from contextlib import contextmanager
from pathlib import Path
from typing import Optional

import pytest
import requests
import torch
import torch.distributed as dist

from sglang.srt.debug_utils.dumper import (
    DumperConfig,
    _collective_with_timeout,
    _compare_tensors_quick,
    _deepcopy_or_clone,
    _detect_recompute_status,
    _Dumper,
    _format_tags,
    _get_default_exp_name,
    _Grafter,
    _load_function,
    _log,
    _map_tensor,
    _materialize_value,
    _MegatronPlugin,
    _obj_to_dict,
    _RecomputeStatus,
    _register_forward_hook_or_replace_fn,
    _SGLangPlugin,
    _torch_save,
    dumper,
    get_tensor_info,
    get_truncated_value,
)
from sglang.srt.environ import temp_set_env
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    find_available_port,
    popen_launch_server,
    run_distributed_test,
)
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 52-53: Register CI metadata
```python
register_cuda_ci(est_time=30, suite="nightly-2-gpu", nightly=True)
register_amd_ci(est_time=60, suite="nightly-amd", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata. It also registers the case for CUDA CI coverage and registers the case for AMD CI coverage.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。 其中还会将该用例注册到 CUDA CI 覆盖范围，并将该用例注册到 AMD CI 覆盖范围。

### Lines 56-64: Define helper: capture stdout
```python
@contextmanager
def _capture_stdout():
    captured = io.StringIO()
    old_stdout = sys.stdout
    sys.stdout = captured
    try:
        yield captured
    finally:
        sys.stdout = old_stdout
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 67-67: Define class TestDumperConfig
```python
class TestDumperConfig:
```
**EN:** This declaration introduces the `TestDumperConfig` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumperConfig` 测试类，并说明它通过继承承担的职责。

### Lines 68-69: Run test: from env defaults match dataclass defaults
```python
    def test_from_env_defaults_match_dataclass_defaults(self):
        assert DumperConfig.from_env() == DumperConfig()
```
**EN:** This test method exercises from env defaults match dataclass defaults and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from env defaults match dataclass defaults 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 71-75: Run test: from env bool
```python
    def test_from_env_bool(self):
        with temp_set_env(DUMPER_ENABLE="1"):
            assert DumperConfig.from_env().enable is True
        with temp_set_env(DUMPER_ENABLE="false"):
            assert DumperConfig.from_env().enable is False
```
**EN:** This test method exercises from env bool and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from env bool 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 77-79: Run test: from env str
```python
    def test_from_env_str(self):
        with temp_set_env(DUMPER_FILTER="layer_id=0"):
            assert DumperConfig.from_env().filter == "layer_id=0"
```
**EN:** This test method exercises from env str and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from env str 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 81-83: Run test: from env dir
```python
    def test_from_env_dir(self):
        with temp_set_env(DUMPER_DIR="/my/dir"):
            assert DumperConfig.from_env().dir == "/my/dir"
```
**EN:** This test method exercises from env dir and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from env dir 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 85-87: Run test: from env int
```python
    def test_from_env_int(self):
        with temp_set_env(DUMPER_COLLECTIVE_TIMEOUT="120"):
            assert DumperConfig.from_env().collective_timeout == 120
```
**EN:** This test method exercises from env int and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from env int 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 89-94: Run test: configure overrides
```python
    def test_configure_overrides(self):
        d = _make_test_dumper("/tmp")
        d.configure(enable=False)
        assert d._config.enable is False
        d.configure(enable=True)
        assert d._config.enable is True
```
**EN:** This test method exercises configure overrides and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 configure overrides 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 96-104: Run test: type validation
```python
    def test_type_validation(self):
        with pytest.raises(TypeError, match="enable.*expected bool.*got str"):
            DumperConfig(enable="yes")
        with pytest.raises(
            TypeError, match="collective_timeout.*expected int.*got str"
        ):
            DumperConfig(collective_timeout="abc")
        with pytest.raises(TypeError, match="filter.*expected str.*got int"):
            DumperConfig(filter=123)
```
**EN:** This test method exercises type validation and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 type validation 场景，并验证观测到的行为是否符合预期契约。

### Lines 106-110: Run test: configure default skips when env set
```python
    def test_configure_default_skips_when_env_set(self):
        with temp_set_env(DUMPER_FILTER="from_env"):
            d = _Dumper(config=DumperConfig.from_env())
            d.configure_default(filter="from_code")
            assert d._config.filter == "from_env"
```
**EN:** This test method exercises configure default skips when env set and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 configure default skips when env set 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 112-115: Run test: configure default applies when no env
```python
    def test_configure_default_applies_when_no_env(self):
        d = _Dumper(config=DumperConfig.from_env())
        d.configure_default(filter="from_code")
        assert d._config.filter == "from_code"
```
**EN:** This test method exercises configure default applies when no env and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 configure default applies when no env 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 117-119: Run test: from env whitespace treated as unset
```python
    def test_from_env_whitespace_treated_as_unset(self):
        with temp_set_env(DUMPER_FILTER="   "):
            assert DumperConfig.from_env().filter is None
```
**EN:** This test method exercises from env whitespace treated as unset and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from env whitespace treated as unset 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 121-123: Run test: may enable default false
```python
    def test_may_enable_default_false(self):
        d = _Dumper(config=DumperConfig())
        assert d.may_enable is False
```
**EN:** This test method exercises may enable default false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 may enable default false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 125-127: Run test: may enable true when enabled
```python
    def test_may_enable_true_when_enabled(self):
        d = _Dumper(config=DumperConfig(enable=True))
        assert d.may_enable is True
```
**EN:** This test method exercises may enable true when enabled and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 may enable true when enabled 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 129-134: Run test: may enable true when server port set
```python
    def test_may_enable_true_when_server_port_set(self):
        d = _Dumper(config=DumperConfig(server_port="40000"))
        assert d.may_enable is True

        d2 = _Dumper(config=DumperConfig(server_port="reuse"))
        assert d2.may_enable is True
```
**EN:** This test method exercises may enable true when server port set and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 may enable true when server port set 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 137-137: Define class TestServerPortParsed
```python
class TestServerPortParsed:
```
**EN:** This declaration introduces the `TestServerPortParsed` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestServerPortParsed` 测试类，并说明它通过继承承担的职责。

### Lines 138-139: Run test: negative returns none
```python
    def test_negative_returns_none(self):
        assert DumperConfig(server_port="-1").server_port_parsed is None
```
**EN:** This test method exercises negative returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 negative returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 141-142: Run test: zero returns none
```python
    def test_zero_returns_none(self):
        assert DumperConfig(server_port="0").server_port_parsed is None
```
**EN:** This test method exercises zero returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 zero returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 144-147: Run test: positive returns int
```python
    def test_positive_returns_int(self):
        result = DumperConfig(server_port="40000").server_port_parsed
        assert result == 40000
        assert isinstance(result, int)
```
**EN:** This test method exercises positive returns int and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 positive returns int 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 149-150: Run test: reuse returns string
```python
    def test_reuse_returns_string(self):
        assert DumperConfig(server_port="reuse").server_port_parsed == "reuse"
```
**EN:** This test method exercises reuse returns string and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reuse returns string 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 153-153: Define class TestDefaultExpName
```python
class TestDefaultExpName:
```
**EN:** This declaration introduces the `TestDefaultExpName` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDefaultExpName` 测试类，并说明它通过继承承担的职责。

### Lines 154-156: Run test: starts with prefix
```python
    def test_starts_with_prefix(self):
        name = _get_default_exp_name(timeout_seconds=5)
        assert name.startswith("dump_")
```
**EN:** This test method exercises starts with prefix and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 starts with prefix 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 158-162: Run test: suffix format
```python
    def test_suffix_format(self):
        name = _get_default_exp_name(timeout_seconds=5)
        suffix = name[len("dump_") :]
        assert len(suffix) == 22
        assert suffix[8] == "_"
```
**EN:** This test method exercises suffix format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 suffix format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 165-165: Define class TestKvPairsParsing
```python
class TestKvPairsParsing:
```
**EN:** This declaration introduces the `TestKvPairsParsing` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestKvPairsParsing` 测试类，并说明它通过继承承担的职责。

### Lines 166-167: Run test: from kv pairs none returns defaults
```python
    def test_from_kv_pairs_none_returns_defaults(self):
        assert DumperConfig.from_kv_pairs(None) == DumperConfig()
```
**EN:** This test method exercises from kv pairs none returns defaults and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs none returns defaults 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 169-170: Run test: from kv pairs empty returns defaults
```python
    def test_from_kv_pairs_empty_returns_defaults(self):
        assert DumperConfig.from_kv_pairs([]) == DumperConfig()
```
**EN:** This test method exercises from kv pairs empty returns defaults and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs empty returns defaults 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 172-175: Run test: from kv pairs bool field
```python
    def test_from_kv_pairs_bool_field(self):
        cfg = DumperConfig.from_kv_pairs(["enable=true"])
        assert cfg.enable is True
        assert cfg.dir == "/tmp/dumper"
```
**EN:** This test method exercises from kv pairs bool field and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs bool field 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 177-179: Run test: from kv pairs bool numeric
```python
    def test_from_kv_pairs_bool_numeric(self):
        assert DumperConfig.from_kv_pairs(["enable=1"]).enable is True
        assert DumperConfig.from_kv_pairs(["enable=0"]).enable is False
```
**EN:** This test method exercises from kv pairs bool numeric and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs bool numeric 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 181-184: Run test: from kv pairs int field
```python
    def test_from_kv_pairs_int_field(self):
        cfg = DumperConfig.from_kv_pairs(["collective_timeout=120"])
        assert cfg.collective_timeout == 120
        assert type(cfg.collective_timeout) is int
```
**EN:** This test method exercises from kv pairs int field and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs int field 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 186-189: Run test: from kv pairs int field zero stays int
```python
    def test_from_kv_pairs_int_field_zero_stays_int(self):
        cfg = DumperConfig.from_kv_pairs(["collective_timeout=0"])
        assert cfg.collective_timeout == 0
        assert type(cfg.collective_timeout) is int
```
**EN:** This test method exercises from kv pairs int field zero stays int and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs int field zero stays int 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 191-194: Run test: from kv pairs str field not coerced
```python
    def test_from_kv_pairs_str_field_not_coerced(self):
        cfg = DumperConfig.from_kv_pairs(["server_port=0"])
        assert cfg.server_port == "0"
        assert type(cfg.server_port) is str
```
**EN:** This test method exercises from kv pairs str field not coerced and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs str field not coerced 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 196-199: Run test: from kv pairs str field one stays str
```python
    def test_from_kv_pairs_str_field_one_stays_str(self):
        cfg = DumperConfig.from_kv_pairs(["server_port=1"])
        assert cfg.server_port == "1"
        assert type(cfg.server_port) is str
```
**EN:** This test method exercises from kv pairs str field one stays str and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs str field one stays str 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 201-205: Run test: from kv pairs optional str field
```python
    def test_from_kv_pairs_optional_str_field(self):
        cfg = DumperConfig.from_kv_pairs(
            ["filter=layer_id is not None and layer_id < 3"]
        )
        assert cfg.filter == "layer_id is not None and layer_id < 3"
```
**EN:** This test method exercises from kv pairs optional str field and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs optional str field 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 207-209: Run test: from kv pairs optional str exp name
```python
    def test_from_kv_pairs_optional_str_exp_name(self):
        cfg = DumperConfig.from_kv_pairs(["exp_name=my_experiment"])
        assert cfg.exp_name == "my_experiment"
```
**EN:** This test method exercises from kv pairs optional str exp name and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs optional str exp name 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 211-225: Run test: from kv pairs multiple fields
```python
    def test_from_kv_pairs_multiple_fields(self):
        cfg = DumperConfig.from_kv_pairs(
            [
                "enable=true",
                "dir=/my/dir",
                "filter=name == 'foo'",
                "collective_timeout=30",
                "enable_grad=1",
            ]
        )
        assert cfg.enable is True
        assert cfg.dir == "/my/dir"
        assert cfg.filter == "name == 'foo'"
        assert cfg.collective_timeout == 30
        assert cfg.enable_grad is True
```
**EN:** This test method exercises from kv pairs multiple fields and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs multiple fields 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 227-229: Run test: from kv pairs missing equals raises
```python
    def test_from_kv_pairs_missing_equals_raises(self):
        with pytest.raises(ValueError, match="missing '='"):
            DumperConfig.from_kv_pairs(["enable"])
```
**EN:** This test method exercises from kv pairs missing equals raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 from kv pairs missing equals raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 231-233: Run test: from kv pairs unknown key raises
```python
    def test_from_kv_pairs_unknown_key_raises(self):
        with pytest.raises(ValueError, match="Unknown config key"):
            DumperConfig.from_kv_pairs(["nonexistent=true"])
```
**EN:** This test method exercises from kv pairs unknown key raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 from kv pairs unknown key raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 235-239: Run test: kv pairs to dict returns only explicit
```python
    def test_kv_pairs_to_dict_returns_only_explicit(self):
        d = DumperConfig._kv_pairs_to_dict(["enable=true", "dir=/x"])
        assert d == {"enable": True, "dir": "/x"}
        assert "filter" not in d
        assert "collective_timeout" not in d
```
**EN:** This test method exercises kv pairs to dict returns only explicit and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 kv pairs to dict returns only explicit 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 241-242: Run test: kv pairs to dict none returns empty
```python
    def test_kv_pairs_to_dict_none_returns_empty(self):
        assert DumperConfig._kv_pairs_to_dict(None) == {}
```
**EN:** This test method exercises kv pairs to dict none returns empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 kv pairs to dict none returns empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 244-245: Run test: kv pairs to dict empty returns empty
```python
    def test_kv_pairs_to_dict_empty_returns_empty(self):
        assert DumperConfig._kv_pairs_to_dict([]) == {}
```
**EN:** This test method exercises kv pairs to dict empty returns empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 kv pairs to dict empty returns empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 247-249: Run test: from kv pairs value with equals in value
```python
    def test_from_kv_pairs_value_with_equals_in_value(self):
        cfg = DumperConfig.from_kv_pairs(["filter=name == 'foo'"])
        assert cfg.filter == "name == 'foo'"
```
**EN:** This test method exercises from kv pairs value with equals in value and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from kv pairs value with equals in value 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 251-253: Run test: from kv pairs type validation still works
```python
    def test_from_kv_pairs_type_validation_still_works(self):
        with pytest.raises(TypeError, match="collective_timeout.*expected int"):
            DumperConfig.from_kv_pairs(["collective_timeout=not_a_number"])
```
**EN:** This test method exercises from kv pairs type validation still works and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 from kv pairs type validation still works 场景，并验证观测到的行为是否符合预期契约。

### Lines 256-256: Define class TestDumperPureFunctions
```python
class TestDumperPureFunctions:
```
**EN:** This declaration introduces the `TestDumperPureFunctions` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumperPureFunctions` 测试类，并说明它通过继承承担的职责。

### Lines 257-262: Run test: get truncated value
```python
    def test_get_truncated_value(self):
        assert get_truncated_value(None) is None
        assert get_truncated_value(42) == 42
        assert len(get_truncated_value((torch.randn(10), torch.randn(20)))) == 2
        assert get_truncated_value(torch.randn(10, 10)).shape == (10, 10)
        assert get_truncated_value(torch.randn(100, 100)).shape == (5, 5)
```
**EN:** This test method exercises get truncated value and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 get truncated value 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 264-275: Run test: obj to dict
```python
    def test_obj_to_dict(self):
        assert _obj_to_dict({"a": 1}) == {"a": 1}

        class Obj:
            x, y = 10, 20

            def method(self):
                pass

        result = _obj_to_dict(Obj())
        assert result["x"] == 10
        assert "method" not in result
```
**EN:** This test method exercises obj to dict and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 obj to dict 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 277-282: Run test: deepcopy or clone tensor
```python
    def test_deepcopy_or_clone_tensor(self):
        original = torch.randn(3, 3)
        cloned = _deepcopy_or_clone(original)
        assert torch.equal(cloned, original)
        original.fill_(999.0)
        assert not torch.equal(cloned, original)
```
**EN:** This test method exercises deepcopy or clone tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 deepcopy or clone tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 284-290: Run test: deepcopy or clone non tensor
```python
    def test_deepcopy_or_clone_non_tensor(self):
        original = {"a": [1, 2, 3]}
        cloned = _deepcopy_or_clone(original)
        assert cloned == original
        assert cloned is not original
        original["a"].append(4)
        assert len(cloned["a"]) == 3
```
**EN:** This test method exercises deepcopy or clone non tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 deepcopy or clone non tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 292-298: Run test: get tensor info
```python
    def test_get_tensor_info(self):
        info = get_tensor_info(torch.randn(10, 10))
        for key in ["shape=", "dtype=", "min=", "max=", "mean="]:
            assert key in info

        assert "value=42" in get_tensor_info(42)
        assert "min=None" in get_tensor_info(torch.tensor([]))
```
**EN:** This test method exercises get tensor info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 get tensor info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 301-301: Define class TestMapTensor
```python
class TestMapTensor:
```
**EN:** This declaration introduces the `TestMapTensor` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMapTensor` 测试类，并说明它通过继承承担的职责。

### Lines 302-305: Run test: bare tensor
```python
    def test_bare_tensor(self):
        t = torch.randn(4)
        result = _map_tensor(t, lambda x: x * 2)
        assert torch.equal(result, t * 2)
```
**EN:** This test method exercises bare tensor and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bare tensor 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 307-310: Run test: bare tensor no change
```python
    def test_bare_tensor_no_change(self):
        t = torch.randn(4)
        result = _map_tensor(t, lambda x: x)
        assert result is t
```
**EN:** This test method exercises bare tensor no change and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 bare tensor no change 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 312-321: Run test: dict with tensor values
```python
    def test_dict_with_tensor_values(self):
        t1 = torch.randn(3)
        t2 = torch.randn(5)
        value = {"a": t1, "b": t2, "meta": "not a tensor"}
        result = _map_tensor(value, lambda x: x.clone())
        assert torch.equal(result["a"], t1)
        assert torch.equal(result["b"], t2)
        assert result["a"] is not t1
        assert result["b"] is not t2
        assert result["meta"] == "not a tensor"
```
**EN:** This test method exercises dict with tensor values and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dict with tensor values 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 323-326: Run test: dict no tensors
```python
    def test_dict_no_tensors(self):
        value = {"a": 1, "b": "hello"}
        result = _map_tensor(value, lambda x: x.clone())
        assert result == value
```
**EN:** This test method exercises dict no tensors and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dict no tensors 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 328-336: Run test: nested dict
```python
    def test_nested_dict(self):
        inner_t = torch.randn(3)
        value = {"outer": {"inner": inner_t, "label": "ok"}, "top": torch.randn(2)}
        result = _map_tensor(value, lambda x: x.clone())
        assert torch.equal(result["outer"]["inner"], inner_t)
        assert result["outer"]["inner"] is not inner_t
        assert result["outer"]["label"] == "ok"
        assert result is not value
        assert result["outer"] is not value["outer"]
```
**EN:** This test method exercises nested dict and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 nested dict 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 338-340: Run test: non tensor non dict
```python
    def test_non_tensor_non_dict(self):
        result = _map_tensor(42, lambda x: x.clone())
        assert result == 42
```
**EN:** This test method exercises non tensor non dict and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non tensor non dict 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 343-343: Define class TestTorchSave
```python
class TestTorchSave:
```
**EN:** This declaration introduces the `TestTorchSave` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestTorchSave` 测试类，并说明它通过继承承担的职责。

### Lines 344-350: Run test: normal
```python
    def test_normal(self, tmp_path):
        path = str(tmp_path / "a.pt")
        tensor = torch.randn(3, 3)

        _torch_save(tensor, path)

        assert torch.equal(torch.load(path, weights_only=True), tensor)
```
**EN:** This test method exercises normal and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 normal 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 352-362: Run test: parameter fallback
```python
    def test_parameter_fallback(self, tmp_path):
        class BadParam(torch.nn.Parameter):
            def __reduce_ex__(self, protocol):
                raise RuntimeError("not pickleable")

        path = str(tmp_path / "b.pt")
        param = BadParam(torch.randn(4))

        _torch_save(param, path)

        assert torch.equal(torch.load(path, weights_only=True), param.data)
```
**EN:** This test method exercises parameter fallback and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 parameter fallback 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 364-378: Run test: shared storage not bloated
```python
    def test_shared_storage_not_bloated(self, tmp_path):
        big = torch.randn(1000, 1000)
        view = big[0]
        path = str(tmp_path / "view.pt")

        _torch_save({"value": view, "meta": {}}, path)

        file_size = Path(path).stat().st_size
        expected_max = view.nelement() * view.element_size() * 10
        assert file_size < expected_max, (
            f"File {file_size} bytes but view is only "
            f"{view.nelement() * view.element_size()} bytes — "
            f"torch.save likely serialized the full "
            f"{big.nelement() * big.element_size()} byte storage"
        )
```
**EN:** This test method exercises shared storage not bloated and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 shared storage not bloated 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 380-388: Run test: silent skip
```python
    def test_silent_skip(self, tmp_path, capsys):
        path = str(tmp_path / "c.pt")

        _torch_save({"fn": lambda: None}, path)

        captured = capsys.readouterr()
        assert "[Dumper, rank=" in captured.out
        assert "Observe error=" in captured.out
        assert "skip the tensor" in captured.out
```
**EN:** This test method exercises silent skip and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 silent skip 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 391-391: Define class TestLog
```python
class TestLog:
```
**EN:** This declaration introduces the `TestLog` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLog` 测试类，并说明它通过继承承担的职责。

### Lines 392-398: Run test: log format
```python
    def test_log_format(self):
        with _capture_stdout() as captured:
            _log("hello")
        out = captured.getvalue()
        assert "hello" in out, out
        assert "[Dumper, rank=" in out, out
        assert ", t=" in out, out
```
**EN:** This test method exercises log format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 log format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 401-401: Define class TestCompareTensorsQuick
```python
class TestCompareTensorsQuick:
```
**EN:** This declaration introduces the `TestCompareTensorsQuick` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCompareTensorsQuick` 测试类，并说明它通过继承承担的职责。

### Lines 402-406: Run test: identical
```python
    def test_identical(self):
        a = torch.tensor([1.0, 2.0, 3.0])
        s = _compare_tensors_quick(a, a.clone())
        assert "rel_diff=0" in s, s
        assert "max_abs=0" in s, s
```
**EN:** This test method exercises identical and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 identical 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 408-413: Run test: diverged
```python
    def test_diverged(self):
        a = torch.tensor([1.0, 2.0, 3.0])
        b = torch.tensor([1.0, 2.0, 4.0])  # last element differs by 1
        s = _compare_tensors_quick(a, b)
        assert "max_abs=1" in s, s
        assert "rel_diff=" in s, s
```
**EN:** This test method exercises diverged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 diverged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 415-417: Run test: shape mismatch
```python
    def test_shape_mismatch(self):
        s = _compare_tensors_quick(torch.zeros(3), torch.zeros(4))
        assert "shape mismatch" in s, s
```
**EN:** This test method exercises shape mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 shape mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 419-425: Run test: dtype unified
```python
    def test_dtype_unified(self):
        s = _compare_tensors_quick(
            torch.zeros(3, dtype=torch.float32),
            torch.zeros(3, dtype=torch.float64),
        )
        assert "rel_diff=" in s, s
        assert "max_abs=" in s, s
```
**EN:** This test method exercises dtype unified and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dtype unified 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 427-429: Run test: empty
```python
    def test_empty(self):
        s = _compare_tensors_quick(torch.zeros(0), torch.zeros(0))
        assert s == "empty"
```
**EN:** This test method exercises empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 432-432: Define class TestCollectiveTimeout
```python
class TestCollectiveTimeout:
```
**EN:** This declaration introduces the `TestCollectiveTimeout` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCollectiveTimeout` 测试类，并说明它通过继承承担的职责。

### Lines 433-457: Run test: watchdog fires on timeout
```python
    def test_watchdog_fires_on_timeout(self):
        block_event = threading.Event()
        output = ""

        def run_with_timeout():
            nonlocal output
            with _capture_stdout() as captured:
                _collective_with_timeout(
                    lambda: block_event.wait(),
                    operation_name="test_blocked_op",
                    timeout_seconds=2,
                )
            output = captured.getvalue()

        worker = threading.Thread(target=run_with_timeout)
        worker.start()

        time.sleep(4)
        block_event.set()
        worker.join(timeout=5)

        print(f"Captured output: {output!r}")
        assert "WARNING" in output
        assert "test_blocked_op" in output
        assert "2s" in output
```
**EN:** This test method exercises watchdog fires on timeout and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions and waits for asynchronous state transitions before rechecking results.
**CN:** 该测试方法会执行 watchdog fires on timeout 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期，并在再次检查结果前等待异步状态转换完成。

### Lines 460-460: Define class TestDumperDistributed
```python
class TestDumperDistributed:
```
**EN:** This declaration introduces the `TestDumperDistributed` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumperDistributed` 测试类，并说明它通过继承承担的职责。

### Lines 461-466: Run test: basic
```python
    def test_basic(self, tmp_path):
        with temp_set_env(
            DUMPER_ENABLE="1",
            DUMPER_DIR=str(tmp_path),
        ):
            run_distributed_test(self._test_basic_func, tmpdir=str(tmp_path))
```
**EN:** This test method exercises basic and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 basic 场景，并验证观测到的行为是否符合预期契约。

### Lines 468-494: Define helper: test basic func
```python
    @staticmethod
    def _test_basic_func(rank, tmpdir):
        tensor = torch.randn(10, 10, device=f"cuda:{rank}")

        dumper.dump("tensor_a", tensor, arg=100)
        dumper.step()

        dumper.set_ctx(ctx_arg=200)
        dumper.dump("tensor_b", tensor)
        dumper.set_ctx(ctx_arg=None)
        dumper.step()

        dumper.configure(filter="False")
        dumper.dump("tensor_skip", tensor)
        dumper.configure(filter=None)
        dumper.step()

        dumper.dump_dict("obj", {"a": torch.randn(3, device=f"cuda:{rank}"), "b": 42})
        dumper.step()

        dist.barrier()
        filenames = _get_filenames(tmpdir)
        _assert_files(
            filenames,
            exist=["tensor_a", "tensor_b", "arg=100", "ctx_arg=200", "obj_a", "obj_b"],
            not_exist=["tensor_skip"],
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDumperDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDumperDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 496-498: Run test: collective timeout
```python
    def test_collective_timeout(self):
        with temp_set_env(DUMPER_ENABLE="1"):
            run_distributed_test(self._test_collective_timeout_func)
```
**EN:** This test method exercises collective timeout and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 collective timeout 场景，并验证观测到的行为是否符合预期契约。

### Lines 500-519: Define helper: test collective timeout func
```python
    @staticmethod
    def _test_collective_timeout_func(rank):
        dumper = _Dumper(
            config=DumperConfig(
                enable=True,
                collective_timeout=3,
            ),
        )

        with _capture_stdout() as captured:
            if rank != 0:
                time.sleep(6)
            dumper.step()

        output = captured.getvalue()
        print(f"Rank {rank} captured output: {output!r}")

        if rank == 0:
            assert "WARNING" in output, f"Expected WARNING in rank 0 output: {output}"
            assert "has not completed after 3s" in output
```
**EN:** This helper function encapsulates reusable logic inside `TestDumperDistributed` so the scenario stays organized. It also checks concrete expectations with assertions and waits for asynchronous state transitions before rechecking results.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDumperDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期，并在再次检查结果前等待异步状态转换完成。

### Lines 521-526: Run test: file content correctness
```python
    def test_file_content_correctness(self, tmp_path):
        with temp_set_env(
            DUMPER_ENABLE="1",
            DUMPER_DIR=str(tmp_path),
        ):
            run_distributed_test(self._test_file_content_func, tmpdir=str(tmp_path))
```
**EN:** This test method exercises file content correctness and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 file content correctness 场景，并验证观测到的行为是否符合预期契约。

### Lines 528-542: Define helper: test file content func
```python
    @staticmethod
    def _test_file_content_func(rank, tmpdir):
        tensor = torch.arange(12, device=f"cuda:{rank}").reshape(3, 4).float()

        dumper.dump("content_check", tensor)
        dumper.step()

        dist.barrier()
        path = _find_dump_file(tmpdir, rank=rank, name="content_check")
        raw = _load_dump(path)
        assert isinstance(raw, dict), f"Expected dict, got {type(raw)}"
        assert "value" in raw and "meta" in raw
        assert torch.equal(raw["value"], tensor.cpu())
        assert raw["meta"]["name"] == "content_check"
        assert raw["meta"]["rank"] == rank
```
**EN:** This helper function encapsulates reusable logic inside `TestDumperDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDumperDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 545-545: Define class TestDumperFileWriteControl
```python
class TestDumperFileWriteControl:
```
**EN:** This declaration introduces the `TestDumperFileWriteControl` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumperFileWriteControl` 测试类，并说明它通过继承承担的职责。

### Lines 546-552: Run test: filter
```python
    def test_filter(self, tmp_path):
        with temp_set_env(
            DUMPER_ENABLE="1",
            DUMPER_DIR=str(tmp_path),
            DUMPER_FILTER="name.startswith('keep')",
        ):
            run_distributed_test(self._test_filter_func, tmpdir=str(tmp_path))
```
**EN:** This test method exercises filter and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 filter 场景，并验证观测到的行为是否符合预期契约。

### Lines 554-567: Define helper: test filter func
```python
    @staticmethod
    def _test_filter_func(rank, tmpdir):
        dumper.dump("keep_this", torch.randn(5, device=f"cuda:{rank}"))
        dumper.dump("skip_this", torch.randn(5, device=f"cuda:{rank}"))
        dumper.dump("not_keep_this", torch.randn(5, device=f"cuda:{rank}"))
        dumper.step()

        dist.barrier()
        filenames = _get_filenames(tmpdir)
        _assert_files(
            filenames,
            exist=["keep_this"],
            not_exist=["skip_this", "not_keep_this"],
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestDumperFileWriteControl` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDumperFileWriteControl` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 569-574: Run test: save false
```python
    def test_save_false(self, tmp_path):
        with temp_set_env(
            DUMPER_ENABLE="1",
            DUMPER_DIR=str(tmp_path),
        ):
            run_distributed_test(self._test_save_false_func, tmpdir=str(tmp_path))
```
**EN:** This test method exercises save false and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 save false 场景，并验证观测到的行为是否符合预期契约。

### Lines 576-582: Define helper: test save false func
```python
    @staticmethod
    def _test_save_false_func(rank, tmpdir):
        dumper.dump("no_save_tensor", torch.randn(5, device=f"cuda:{rank}"), save=False)
        dumper.step()

        dist.barrier()
        assert len(_get_filenames(tmpdir)) == 0
```
**EN:** This helper function encapsulates reusable logic inside `TestDumperFileWriteControl` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDumperFileWriteControl` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 585-585: Define class TestDumpEnableFlags
```python
class TestDumpEnableFlags:
```
**EN:** This declaration introduces the `TestDumpEnableFlags` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumpEnableFlags` 测试类，并说明它通过继承承担的职责。

### Lines 586-589: Run test: all enables false no output
```python
    def test_all_enables_false_no_output(self, tmp_path):
        d = _make_test_dumper(tmp_path, enable_value=False, enable_grad=False)
        d.dump("should_skip", torch.randn(3, 3))
        assert len(_get_filenames(tmp_path)) == 0
```
**EN:** This test method exercises all enables false no output and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all enables false no output 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 592-592: Define class TestOutputControl
```python
class TestOutputControl:
```
**EN:** This declaration introduces the `TestOutputControl` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestOutputControl` 测试类，并说明它通过继承承担的职责。

### Lines 593-597: Run test: file enabled by default
```python
    def test_file_enabled_by_default(self, tmp_path):
        d = _make_test_dumper(tmp_path)
        d.dump("file_on", torch.randn(3, 3))

        _assert_files(_get_filenames(tmp_path), exist=["file_on"])
```
**EN:** This test method exercises file enabled by default and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 file enabled by default 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 599-604: Run test: file disabled
```python
    def test_file_disabled(self, tmp_path, capsys):
        d = _make_test_dumper(tmp_path, enable_output_file=False)
        d.dump("file_off", torch.randn(3, 3))

        assert len(_get_filenames(tmp_path)) == 0
        assert "file_off" in capsys.readouterr().out
```
**EN:** This test method exercises file disabled and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 file disabled 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 606-612: Run test: console enabled by default
```python
    def test_console_enabled_by_default(self, tmp_path, capsys):
        d = _make_test_dumper(tmp_path)
        d.dump("console_on", torch.randn(3, 3))

        captured = capsys.readouterr()
        assert "[Dumper.Value]" in captured.out
        assert "console_on" in captured.out
```
**EN:** This test method exercises console enabled by default and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 console enabled by default 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 614-619: Run test: console disabled
```python
    def test_console_disabled(self, tmp_path, capsys):
        d = _make_test_dumper(tmp_path, enable_output_console=False)
        d.dump("console_off", torch.randn(3, 3))

        assert "console_off" not in capsys.readouterr().out
        _assert_files(_get_filenames(tmp_path), exist=["console_off"])
```
**EN:** This test method exercises console disabled and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 console disabled 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 621-631: Run test: capture output basic
```python
    def test_capture_output_basic(self, tmp_path):
        d = _make_test_dumper(tmp_path)
        tensor = torch.randn(4, 4)

        with d.capture_output() as captured:
            d.dump("cap_basic", tensor)

        assert "cap_basic" in captured
        assert set(captured["cap_basic"].keys()) == {"value", "meta"}
        assert torch.equal(captured["cap_basic"]["value"], tensor)
        assert captured["cap_basic"]["meta"]["name"] == "cap_basic"
```
**EN:** This test method exercises capture output basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 capture output basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 633-640: Run test: capture output no file
```python
    def test_capture_output_no_file(self, tmp_path):
        d = _make_test_dumper(tmp_path)

        with d.capture_output() as captured:
            d.dump("cap_no_file", torch.randn(3, 3))

        assert "cap_no_file" in captured
        assert len(_get_filenames(tmp_path)) == 0
```
**EN:** This test method exercises capture output no file and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 capture output no file 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 642-651: Run test: capture output multiple
```python
    def test_capture_output_multiple(self, tmp_path):
        d = _make_test_dumper(tmp_path)

        with d.capture_output() as captured:
            d.dump("first", torch.randn(2, 2))
            d.dump("second", torch.randn(3, 3))

        assert set(captured.keys()) == {"first", "second"}
        assert captured["first"]["value"].shape == (2, 2)
        assert captured["second"]["value"].shape == (3, 3)
```
**EN:** This test method exercises capture output multiple and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 capture output multiple 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 653-661: Run test: capture output value cloned
```python
    def test_capture_output_value_cloned(self, tmp_path):
        d = _make_test_dumper(tmp_path)
        tensor = torch.zeros(3, 3)

        with d.capture_output() as captured:
            d.dump("clone_check", tensor)

        tensor.fill_(999.0)
        assert torch.equal(captured["clone_check"]["value"], torch.zeros(3, 3))
```
**EN:** This test method exercises capture output value cloned and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 capture output value cloned 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 663-668: Run test: capture output nested raises
```python
    def test_capture_output_nested_raises(self, tmp_path):
        d = _make_test_dumper(tmp_path)
        with d.capture_output():
            with pytest.raises(AssertionError):
                with d.capture_output():
                    pass
```
**EN:** This test method exercises capture output nested raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 capture output nested raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 670-678: Run test: capture output respects filter
```python
    def test_capture_output_respects_filter(self, tmp_path):
        d = _make_test_dumper(tmp_path, filter="'keep' in name")

        with d.capture_output() as captured:
            d.dump("keep_this", torch.randn(3, 3))
            d.dump("skip_this", torch.randn(3, 3))

        assert "keep_this" in captured
        assert "skip_this" not in captured
```
**EN:** This test method exercises capture output respects filter and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 capture output respects filter 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 681-681: Define class TestDumpDictFormat
```python
class TestDumpDictFormat:
```
**EN:** This declaration introduces the `TestDumpDictFormat` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumpDictFormat` 测试类，并说明它通过继承承担的职责。

### Lines 682-682: Document the class `TestDumpDictFormat`
```python
    """Verify that dump files use the dict output format: {"value": ..., "meta": {...}}."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDumpDictFormat`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDumpDictFormat`的设计意图。

### Lines 684-701: Run test: dict format structure
```python
    def test_dict_format_structure(self, tmp_path):
        dumper = _make_test_dumper(tmp_path)
        tensor = torch.randn(4, 4)
        dumper.dump("fmt_test", tensor, custom_key="hello")

        path = _find_dump_file(str(tmp_path), rank=0, name="fmt_test")
        raw = _load_dump(path)

        assert isinstance(raw, dict)
        assert set(raw.keys()) == {"value", "meta"}
        assert torch.equal(raw["value"], tensor)

        meta = raw["meta"]
        assert meta["name"] == "fmt_test"
        assert meta["custom_key"] == "hello"
        assert "step" in meta
        assert "rank" in meta
        assert "dump_index" in meta
```
**EN:** This test method exercises dict format structure and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dict format structure 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 703-713: Run test: dict format with context
```python
    def test_dict_format_with_context(self, tmp_path):
        dumper = _make_test_dumper(tmp_path)
        dumper.set_ctx(ctx_val=42)
        tensor = torch.randn(2, 2)
        dumper.dump("ctx_fmt", tensor)

        path = _find_dump_file(str(tmp_path), rank=0, name="ctx_fmt")
        raw = _load_dump(path)

        assert raw["meta"]["ctx_val"] == 42
        assert torch.equal(raw["value"], tensor)
```
**EN:** This test method exercises dict format with context and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dict format with context 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 716-725: Define helper: make test dumper
```python
def _make_test_dumper(tmp_path, **overrides) -> _Dumper:
    """Create a _Dumper for CPU testing without distributed."""
    defaults = dict(
        enable=True,
        dir=str(tmp_path),
        exp_name="test",
    )
    defaults.update(overrides)
    config = DumperConfig(**defaults)
    return _Dumper(config=config)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 728-729: Define helper: get filenames
```python
def _get_filenames(tmpdir):
    return {f.name for f in Path(tmpdir).glob("*/*.pt")}
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 732-738: Define helper: assert files
```python
def _assert_files(filenames, *, exist=(), not_exist=()):
    for p in exist:
        assert any(p in f for f in filenames), f"{p} not found in {filenames}"
    for p in not_exist:
        assert not any(
            p in f for f in filenames
        ), f"{p} should not exist in {filenames}"
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 741-743: Define helper: load dump
```python
def _load_dump(path: Path) -> dict:
    """Load a dump file and return the raw dict (with 'value' and 'meta' keys)."""
    return torch.load(path, map_location="cpu", weights_only=False)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 746-755: Define helper: find dump file
```python
def _find_dump_file(tmpdir, *, rank: int = 0, name: str) -> Path:
    matches = [
        f
        for f in Path(tmpdir).glob("*/*.pt")
        if f"rank={rank}" in f.name and name in f.name
    ]
    assert (
        len(matches) == 1
    ), f"Expected 1 file matching rank={rank} name={name}, got {matches}"
    return matches[0]
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 758-758: Define class TestMaterializeValue
```python
class TestMaterializeValue:
```
**EN:** This declaration introduces the `TestMaterializeValue` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMaterializeValue` 测试类，并说明它通过继承承担的职责。

### Lines 759-762: Run test: materialize value callable
```python
    def test_materialize_value_callable(self):
        tensor = torch.randn(3, 3)
        result = _materialize_value(lambda: tensor)
        assert torch.equal(result, tensor)
```
**EN:** This test method exercises materialize value callable and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 materialize value callable 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 764-767: Run test: materialize value passthrough
```python
    def test_materialize_value_passthrough(self):
        tensor = torch.randn(3, 3)
        result = _materialize_value(tensor)
        assert result is tensor
```
**EN:** This test method exercises materialize value passthrough and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 materialize value passthrough 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 769-777: Run test: dump with callable value
```python
    def test_dump_with_callable_value(self, tmp_path):
        d = _make_test_dumper(tmp_path)
        tensor = torch.randn(4, 4)
        d.dump("lazy_tensor", lambda: tensor)

        _assert_files(_get_filenames(tmp_path), exist=["name=lazy_tensor"])

        path = _find_dump_file(tmp_path, rank=0, name="lazy_tensor")
        assert torch.equal(_load_dump(path)["value"], tensor)
```
**EN:** This test method exercises dump with callable value and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dump with callable value 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 780-780: Define class TestSaveValue
```python
class TestSaveValue:
```
**EN:** This declaration introduces the `TestSaveValue` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestSaveValue` 测试类，并说明它通过继承承担的职责。

### Lines 781-791: Run test: dump output format
```python
    def test_dump_output_format(self, tmp_path):
        dumper = _make_test_dumper(tmp_path)
        tensor = torch.randn(4, 4)

        dumper.dump("dict_test", tensor)

        path = _find_dump_file(tmp_path, rank=0, name="dict_test")
        loaded = _load_dump(path)
        assert torch.equal(loaded["value"], tensor)
        assert loaded["meta"]["name"] == "dict_test"
        assert loaded["meta"]["rank"] == 0
```
**EN:** This test method exercises dump output format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dump output format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 794-794: Define class TestStaticMetadata
```python
class TestStaticMetadata:
```
**EN:** This declaration introduces the `TestStaticMetadata` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestStaticMetadata` 测试类，并说明它通过继承承担的职责。

### Lines 795-801: Run test: static meta contains world info
```python
    def test_static_meta_contains_world_info(self):
        dumper = _make_test_dumper("/tmp")
        meta = dumper._static_meta
        assert "world_rank" in meta
        assert "world_size" in meta
        assert meta["world_rank"] == 0
        assert meta["world_size"] == 1
```
**EN:** This test method exercises static meta contains world info and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 static meta contains world info 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 803-807: Run test: static meta caching
```python
    def test_static_meta_caching(self):
        dumper = _make_test_dumper("/tmp")
        meta1 = dumper._static_meta
        meta2 = dumper._static_meta
        assert meta1 is meta2
```
**EN:** This test method exercises static meta caching and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 static meta caching 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 809-814: Run test: parallel info graceful fallback
```python
    def test_parallel_info_graceful_fallback(self):
        sglang_info = _SGLangPlugin().collect_parallel_info()
        assert isinstance(sglang_info, dict)

        megatron_info = _MegatronPlugin().collect_parallel_info()
        assert isinstance(megatron_info, dict)
```
**EN:** This test method exercises parallel info graceful fallback and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 parallel info graceful fallback 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 816-826: Run test: dump includes static meta
```python
    def test_dump_includes_static_meta(self, tmp_path):
        dumper = _make_test_dumper(tmp_path)
        tensor = torch.randn(2, 2)

        dumper.dump("meta_test", tensor)

        path = _find_dump_file(tmp_path, rank=0, name="meta_test")
        loaded = _load_dump(path)
        meta = loaded["meta"]
        assert "world_rank" in meta
        assert "world_size" in meta
```
**EN:** This test method exercises dump includes static meta and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dump includes static meta 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 829-829: Define class TestDumpGrad
```python
class TestDumpGrad:
```
**EN:** This declaration introduces the `TestDumpGrad` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumpGrad` 测试类，并说明它通过继承承担的职责。

### Lines 830-840: Run test: dump grad basic
```python
    def test_dump_grad_basic(self, tmp_path):
        d = _make_test_dumper(tmp_path, enable_grad=True)
        x = torch.randn(3, 3, requires_grad=True)
        y = (x * 2).sum()

        d.dump("test_tensor", x)
        y.backward()

        filenames = _get_filenames(tmp_path)
        assert any("name=test_tensor" in f and "grad__" not in f for f in filenames)
        _assert_files(filenames, exist=["grad__test_tensor"])
```
**EN:** This test method exercises dump grad basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dump grad basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 842-846: Run test: dump grad non tensor skipped
```python
    def test_dump_grad_non_tensor_skipped(self, tmp_path):
        d = _make_test_dumper(tmp_path, enable_grad=True)
        d.dump("not_tensor", 42)

        _assert_files(_get_filenames(tmp_path), not_exist=["grad__"])
```
**EN:** This test method exercises dump grad non tensor skipped and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dump grad non tensor skipped 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 848-857: Run test: dump grad no requires grad skipped
```python
    def test_dump_grad_no_requires_grad_skipped(self, tmp_path):
        d = _make_test_dumper(tmp_path, enable_grad=True)
        x = torch.randn(3, 3, requires_grad=False)
        d.dump("no_grad_tensor", x)

        _assert_files(
            _get_filenames(tmp_path),
            exist=["name=no_grad_tensor"],
            not_exist=["grad__"],
        )
```
**EN:** This test method exercises dump grad no requires grad skipped and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dump grad no requires grad skipped 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 859-870: Run test: dump grad captures step
```python
    def test_dump_grad_captures_step(self, tmp_path):
        d = _make_test_dumper(tmp_path, enable_grad=True)
        d._state.step = 42
        x = torch.randn(3, 3, requires_grad=True)
        y = (x * 2).sum()

        d.dump("id_test", x)
        d._state.step = 999
        y.backward()

        grad_file = _find_dump_file(tmp_path, name="grad__id_test")
        assert "step=42" in grad_file.name
```
**EN:** This test method exercises dump grad captures step and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dump grad captures step 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 872-882: Run test: dump grad file content
```python
    def test_dump_grad_file_content(self, tmp_path):
        d = _make_test_dumper(tmp_path, enable_grad=True)
        x = torch.tensor([[1.0, 2.0], [3.0, 4.0]], requires_grad=True)
        y = (x * 3).sum()

        d.dump("content_check", x)
        y.backward()

        grad_path = _find_dump_file(tmp_path, name="grad__content_check")
        expected_grad = torch.full((2, 2), 3.0)
        assert torch.equal(_load_dump(grad_path)["value"], expected_grad)
```
**EN:** This test method exercises dump grad file content and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dump grad file content 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 884-896: Run test: disable value
```python
    def test_disable_value(self, tmp_path):
        d = _make_test_dumper(tmp_path, enable_value=False, enable_grad=True)
        x = torch.randn(3, 3, requires_grad=True)
        y = (x * 2).sum()

        d.dump("fwd_disabled", x)
        y.backward()

        filenames = _get_filenames(tmp_path)
        assert not any(
            "name=fwd_disabled" in f and "grad__" not in f for f in filenames
        )
        _assert_files(filenames, exist=["grad__fwd_disabled"])
```
**EN:** This test method exercises disable value and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 disable value 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 898-910: Run test: disable grad
```python
    def test_disable_grad(self, tmp_path):
        d = _make_test_dumper(tmp_path, enable_grad=False)
        x = torch.randn(3, 3, requires_grad=True)
        y = (x * 2).sum()

        d.dump("grad_disabled", x)
        y.backward()

        _assert_files(
            _get_filenames(tmp_path),
            exist=["name=grad_disabled"],
            not_exist=["grad__"],
        )
```
**EN:** This test method exercises disable grad and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 disable grad 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 913-913: Define class TestKvFilter
```python
class TestKvFilter:
```
**EN:** This declaration introduces the `TestKvFilter` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestKvFilter` 测试类，并说明它通过继承承担的职责。

### Lines 914-916: Run test: format tags
```python
    def test_format_tags(self):
        assert _format_tags({"a": 1, "b": "hello"}) == "a=1___b=hello"
        assert _format_tags({}) == ""
```
**EN:** This test method exercises format tags and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 format tags 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 918-924: Run test: filter matches extra kwargs
```python
    def test_filter_matches_extra_kwargs(self, tmp_path):
        d = _make_test_dumper(tmp_path, filter="layer_id == 0")
        d.dump("tensor_a", torch.randn(3), layer_id=0)
        d.dump("tensor_b", torch.randn(3), layer_id=1)

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["tensor_a"], not_exist=["tensor_b"])
```
**EN:** This test method exercises filter matches extra kwargs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter matches extra kwargs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 926-934: Run test: filter matches global ctx
```python
    def test_filter_matches_global_ctx(self, tmp_path):
        d = _make_test_dumper(tmp_path, filter="ctx_arg == 200")
        d.set_ctx(ctx_arg=200)
        d.dump("tensor_a", torch.randn(3))
        d.set_ctx(ctx_arg=None)
        d.dump("tensor_b", torch.randn(3))

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["tensor_a"], not_exist=["tensor_b"])
```
**EN:** This test method exercises filter matches global ctx and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter matches global ctx 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 936-942: Run test: filter matches name
```python
    def test_filter_matches_name(self, tmp_path):
        d = _make_test_dumper(tmp_path, filter="'keep' in name")
        d.dump("keep_this", torch.randn(3))
        d.dump("skip_this", torch.randn(3))

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["keep_this"], not_exist=["skip_this"])
```
**EN:** This test method exercises filter matches name and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter matches name 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 944-951: Run test: filter expr range
```python
    def test_filter_expr_range(self, tmp_path):
        d = _make_test_dumper(tmp_path, filter="layer_id is not None and layer_id < 3")
        d.dump("t0", torch.randn(3), layer_id=0)
        d.dump("t1", torch.randn(3), layer_id=1)
        d.dump("t5", torch.randn(3), layer_id=5)

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["name=t0", "name=t1"], not_exist=["name=t5"])
```
**EN:** This test method exercises filter expr range and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter expr range 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 953-964: Run test: filter expr with none
```python
    def test_filter_expr_with_none(self, tmp_path):
        d = _make_test_dumper(tmp_path, filter="layer_id is None or layer_id < 3")
        d.dump("no_layer", torch.randn(3))
        d.dump("layer0", torch.randn(3), layer_id=0)
        d.dump("layer5", torch.randn(3), layer_id=5)

        filenames = _get_filenames(tmp_path)
        _assert_files(
            filenames,
            exist=["no_layer", "layer0"],
            not_exist=["layer5"],
        )
```
**EN:** This test method exercises filter expr with none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter expr with none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 966-977: Run test: filter expr with re search
```python
    def test_filter_expr_with_re_search(self, tmp_path):
        d = _make_test_dumper(tmp_path, filter="search(r'attn|mlp', name)")
        d.dump("self_attn", torch.randn(3))
        d.dump("mlp_proj", torch.randn(3))
        d.dump("layernorm", torch.randn(3))

        filenames = _get_filenames(tmp_path)
        _assert_files(
            filenames,
            exist=["self_attn", "mlp_proj"],
            not_exist=["layernorm"],
        )
```
**EN:** This test method exercises filter expr with re search and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter expr with re search 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 979-982: Run test: filter expr syntax error
```python
    def test_filter_expr_syntax_error(self, tmp_path):
        d = _make_test_dumper(tmp_path, filter="layer_id ===")
        with pytest.raises(SyntaxError):
            d.dump("tensor", torch.randn(3))
```
**EN:** This test method exercises filter expr syntax error and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 filter expr syntax error 场景，并验证观测到的行为是否符合预期契约。

### Lines 984-990: Run test: no filter dumps all
```python
    def test_no_filter_dumps_all(self, tmp_path):
        d = _make_test_dumper(tmp_path)
        d.dump("a", torch.randn(3))
        d.dump("b", torch.randn(3))

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["name=a", "name=b"])
```
**EN:** This test method exercises no filter dumps all and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no filter dumps all 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 993-993: Define class TestDumpModel
```python
class TestDumpModel:
```
**EN:** This declaration introduces the `TestDumpModel` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumpModel` 测试类，并说明它通过继承承担的职责。

### Lines 994-1008: Run test: grad basic
```python
    def test_grad_basic(self, tmp_path):
        d = _make_test_dumper(
            tmp_path, enable_model_grad=True, enable_model_value=False
        )
        model = torch.nn.Linear(4, 2)
        x = torch.randn(3, 4)
        y = model(x).sum()
        y.backward()

        d.dump_model(model, name_prefix="model")

        _assert_files(
            _get_filenames(tmp_path),
            exist=["grad__model__weight", "grad__model__bias"],
        )
```
**EN:** This test method exercises grad basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 grad basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1010-1021: Run test: value basic
```python
    def test_value_basic(self, tmp_path):
        d = _make_test_dumper(
            tmp_path, enable_model_value=True, enable_model_grad=False
        )
        model = torch.nn.Linear(4, 2, bias=False)

        d.dump_model(model, name_prefix="model")

        _assert_files(
            _get_filenames(tmp_path),
            exist=["model__weight"],
        )
```
**EN:** This test method exercises value basic and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 value basic 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1023-1032: Run test: no grad skipped
```python
    def test_no_grad_skipped(self, tmp_path):
        d = _make_test_dumper(
            tmp_path, enable_model_grad=True, enable_model_value=False
        )
        model = torch.nn.Linear(4, 2)

        d.dump_model(model, name_prefix="model")

        filenames = _get_filenames(tmp_path)
        assert len(filenames) == 0
```
**EN:** This test method exercises no grad skipped and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no grad skipped 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1034-1052: Run test: filter
```python
    def test_filter(self, tmp_path):
        d = _make_test_dumper(
            tmp_path,
            enable_model_value=True,
            enable_model_grad=True,
            filter="'weight' in name",
        )
        model = torch.nn.Linear(4, 2)
        x = torch.randn(3, 4)
        y = model(x).sum()
        y.backward()

        d.dump_model(model, name_prefix="model")

        _assert_files(
            _get_filenames(tmp_path),
            exist=["model__weight", "grad__model__weight"],
            not_exist=["model__bias", "grad__model__bias"],
        )
```
**EN:** This test method exercises filter and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1054-1066: Run test: grad file content
```python
    def test_grad_file_content(self, tmp_path):
        d = _make_test_dumper(
            tmp_path, enable_model_grad=True, enable_model_value=False
        )
        model = torch.nn.Linear(4, 2, bias=False)
        x = torch.ones(1, 4)
        y = model(x).sum()
        y.backward()

        d.dump_model(model, name_prefix="p")

        path = _find_dump_file(tmp_path, name="grad__p__weight")
        assert torch.equal(_load_dump(path)["value"], model.weight.grad)
```
**EN:** This test method exercises grad file content and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 grad file content 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1068-1080: Run test: disable model grad
```python
    def test_disable_model_grad(self, tmp_path):
        d = _make_test_dumper(
            tmp_path, enable_model_value=True, enable_model_grad=False
        )
        model = torch.nn.Linear(4, 2)
        x = torch.randn(3, 4)
        y = model(x).sum()
        y.backward()

        d.dump_model(model, name_prefix="model")

        filenames = _get_filenames(tmp_path)
        assert all("grad" not in f for f in filenames)
```
**EN:** This test method exercises disable model grad and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 disable model grad 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1082-1093: Run test: parameter saved as parameter
```python
    def test_parameter_saved_as_parameter(self, tmp_path):
        d = _make_test_dumper(
            tmp_path, enable_model_value=True, enable_model_grad=False
        )
        model = torch.nn.Linear(4, 2, bias=False)

        d.dump_model(model, name_prefix="p")

        path = _find_dump_file(tmp_path, name="p__weight")
        loaded = _load_dump(path)
        assert isinstance(loaded["value"], torch.nn.Parameter)
        assert torch.equal(loaded["value"], model.weight)
```
**EN:** This test method exercises parameter saved as parameter and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 parameter saved as parameter 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1095-1112: Run test: unpicklable parameter falls back to data
```python
    def test_unpicklable_parameter_falls_back_to_data(self, tmp_path):
        class BadParam(torch.nn.Parameter):
            def __reduce_ex__(self, protocol):
                raise RuntimeError("not pickleable")

        d = _make_test_dumper(
            tmp_path, enable_model_value=True, enable_model_grad=False
        )
        model = torch.nn.Linear(4, 2, bias=False)
        model.weight = BadParam(model.weight.data)

        d.dump_model(model, name_prefix="p")

        path = _find_dump_file(tmp_path, name="p__weight")
        loaded = _load_dump(path)
        assert isinstance(loaded["value"], torch.Tensor)
        assert not isinstance(loaded["value"], torch.nn.Parameter)
        assert torch.equal(loaded["value"], model.weight.data)
```
**EN:** This test method exercises unpicklable parameter falls back to data and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 unpicklable parameter falls back to data 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1114-1126: Run test: disable model value
```python
    def test_disable_model_value(self, tmp_path):
        d = _make_test_dumper(
            tmp_path, enable_model_grad=True, enable_model_value=False
        )
        model = torch.nn.Linear(4, 2, bias=False)
        x = torch.ones(1, 4)
        y = model(x).sum()
        y.backward()

        d.dump_model(model, name_prefix="model")

        filenames = _get_filenames(tmp_path)
        assert all("grad" in f for f in filenames)
```
**EN:** This test method exercises disable model value and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 disable model value 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1129-1129: Define class TestCleanup
```python
class TestCleanup:
```
**EN:** This declaration introduces the `TestCleanup` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCleanup` 测试类，并说明它通过继承承担的职责。

### Lines 1130-1139: Run test: cleanup removes old dumps
```python
    def test_cleanup_removes_old_dumps(self, tmp_path):
        old_dir = tmp_path / "dump_old"
        old_dir.mkdir()
        (old_dir / "dummy.pt").touch()

        dumper = _make_test_dumper(tmp_path, cleanup_previous=True)
        dumper.dump("new_tensor", torch.randn(3, 3))

        assert not old_dir.exists()
        _assert_files(_get_filenames(tmp_path), exist=["new_tensor"])
```
**EN:** This test method exercises cleanup removes old dumps and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cleanup removes old dumps 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1141-1151: Run test: cleanup removes exp name dir
```python
    def test_cleanup_removes_exp_name_dir(self, tmp_path):
        exp_name = "my_custom_exp"
        old_exp_dir = tmp_path / exp_name
        old_exp_dir.mkdir()
        (old_exp_dir / "old_data.pt").touch()

        dumper = _make_test_dumper(tmp_path, exp_name=exp_name, cleanup_previous=True)
        dumper.dump("new_tensor", torch.randn(3, 3))

        assert not (tmp_path / exp_name / "old_data.pt").exists()
        _assert_files(_get_filenames(tmp_path), exist=["new_tensor"])
```
**EN:** This test method exercises cleanup removes exp name dir and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cleanup removes exp name dir 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1153-1168: Run test: cleanup removes both dump prefix and exp name
```python
    def test_cleanup_removes_both_dump_prefix_and_exp_name(self, tmp_path):
        old_dump = tmp_path / "dump_old"
        old_dump.mkdir()
        (old_dump / "dummy.pt").touch()

        exp_name = "custom_run"
        old_exp = tmp_path / exp_name
        old_exp.mkdir()
        (old_exp / "stale.pt").touch()

        dumper = _make_test_dumper(tmp_path, exp_name=exp_name, cleanup_previous=True)
        dumper.dump("new_tensor", torch.randn(3, 3))

        assert not old_dump.exists()
        assert not (tmp_path / exp_name / "stale.pt").exists()
        _assert_files(_get_filenames(tmp_path), exist=["new_tensor"])
```
**EN:** This test method exercises cleanup removes both dump prefix and exp name and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cleanup removes both dump prefix and exp name 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1170-1179: Run test: no cleanup by default
```python
    def test_no_cleanup_by_default(self, tmp_path):
        old_dir = tmp_path / "dump_old"
        old_dir.mkdir()
        (old_dir / "dummy.pt").touch()

        dumper = _make_test_dumper(tmp_path)
        dumper.dump("new_tensor", torch.randn(3, 3))

        assert old_dir.exists()
        _assert_files(_get_filenames(tmp_path), exist=["new_tensor"])
```
**EN:** This test method exercises no cleanup by default and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no cleanup by default 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1182-1182: Define class TestReset
```python
class TestReset:
```
**EN:** This declaration introduces the `TestReset` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestReset` 测试类，并说明它通过继承承担的职责。

### Lines 1183-1192: Run test: reset clears state
```python
    def test_reset_clears_state(self, tmp_path):
        d = _make_test_dumper(tmp_path)
        d.set_ctx(layer_id=1)
        d.dump("before_reset", torch.randn(3, 3))

        d.reset()

        assert d._state.dump_index == 0
        assert d._state.step == 0
        assert d._state.global_ctx == {}
```
**EN:** This test method exercises reset clears state and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reset clears state 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1194-1204: Run test: dump works after reset
```python
    def test_dump_works_after_reset(self, tmp_path):
        d = _make_test_dumper(tmp_path)
        d.dump("pre", torch.randn(3, 3))

        d.reset()
        d.dump("post", torch.randn(3, 3))

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["pre", "post"])
        post_file = _find_dump_file(tmp_path, name="post")
        assert "dump_index=1" in post_file.name
```
**EN:** This test method exercises dump works after reset and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dump works after reset 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1206-1226: Run test: cleanup previous re triggers after reset
```python
    def test_cleanup_previous_re_triggers_after_reset(self, tmp_path):
        """Miles pattern: reset() + configure(cleanup_previous=True) should re-clean."""
        exp_alpha = "exp_alpha"
        exp_beta = "exp_beta"

        (tmp_path / exp_alpha).mkdir()
        (tmp_path / exp_alpha / "stale.pt").touch()
        (tmp_path / exp_beta).mkdir()
        (tmp_path / exp_beta / "stale.pt").touch()

        d = _make_test_dumper(tmp_path, exp_name=exp_alpha, cleanup_previous=True)
        d.dump("phase1", torch.randn(2, 2))

        d.reset()
        d.configure(exp_name=exp_beta, cleanup_previous=True)
        d.dump("phase2", torch.randn(2, 2))

        assert not (tmp_path / exp_alpha / "stale.pt").exists()
        assert not (tmp_path / exp_beta / "stale.pt").exists()
        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["phase1", "phase2"])
```
**EN:** This test method exercises cleanup previous re triggers after reset and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 cleanup previous re triggers after reset 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1228-1238: Run test: no cleanup when config false
```python
    def test_no_cleanup_when_config_false(self, tmp_path):
        """cleanup_previous=False: handled stays False but no cleanup runs."""
        old_dir = tmp_path / "dump_old"
        old_dir.mkdir()
        (old_dir / "dummy.pt").touch()

        d = _make_test_dumper(tmp_path, cleanup_previous=False)
        d.dump("tensor", torch.randn(2, 2))

        assert old_dir.exists()
        assert d._state.cleanup_previous_handled is False
```
**EN:** This test method exercises no cleanup when config false and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 no cleanup when config false 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1240-1259: Run test: multi phase switch
```python
    def test_multi_phase_switch(self, tmp_path):
        """Simulate Miles multi-phase: configure → dump → reset → configure new phase → dump."""
        d = _make_test_dumper(tmp_path, cleanup_previous=True)

        d.configure(exp_name="fwd_only")
        d.dump("weight", torch.randn(2, 2))
        d.step()
        d.configure(enable=False)

        d.reset()
        d.configure(exp_name="fwd_bwd", enable=True, cleanup_previous=True)
        d.dump("weight", torch.randn(2, 2))
        d.step()

        fwd_only_files = list(Path(tmp_path).glob("fwd_only/*.pt"))
        fwd_bwd_files = list(Path(tmp_path).glob("fwd_bwd/*.pt"))
        assert len(fwd_only_files) > 0
        assert len(fwd_bwd_files) > 0
        assert d._state.step == 1
        assert d._state.dump_index == 1
```
**EN:** This test method exercises multi phase switch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi phase switch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1261-1280: Run test: reset removes non intrusive hooks
```python
    def test_reset_removes_non_intrusive_hooks(self, tmp_path):
        model = torch.nn.Sequential(
            torch.nn.Linear(4, 4),
            torch.nn.ReLU(),
            torch.nn.Linear(4, 4),
        )
        d = _make_test_dumper(tmp_path, non_intrusive_mode="all")
        d.register_non_intrusive_dumper(model)

        x = torch.randn(2, 4)
        with d.capture_output() as captured:
            model(x)
        assert len(captured) > 0

        d.reset()
        d.configure(enable=True, dir=str(tmp_path), non_intrusive_mode="all")

        with d.capture_output() as captured_after:
            model(x)
        assert len(captured_after) == 0
```
**EN:** This test method exercises reset removes non intrusive hooks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reset removes non intrusive hooks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1282-1310: Run test: reset removes non intrusive hooks multiple models
```python
    def test_reset_removes_non_intrusive_hooks_multiple_models(self, tmp_path):
        model_a = torch.nn.Sequential(
            torch.nn.Linear(4, 4),
            torch.nn.ReLU(),
        )
        model_b = torch.nn.Sequential(
            torch.nn.Linear(4, 4),
            torch.nn.ReLU(),
        )
        d = _make_test_dumper(tmp_path, non_intrusive_mode="all")
        d.register_non_intrusive_dumper(model_a)
        d.register_non_intrusive_dumper(model_b)

        x = torch.randn(2, 4)
        with d.capture_output() as captured:
            model_a(x)
            model_b(x)
        assert len(captured) > 0

        d.reset()
        d.configure(enable=True, dir=str(tmp_path), non_intrusive_mode="all")

        with d.capture_output() as captured_a:
            model_a(x)
        assert len(captured_a) == 0

        with d.capture_output() as captured_b:
            model_b(x)
        assert len(captured_b) == 0
```
**EN:** This test method exercises reset removes non intrusive hooks multiple models and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reset removes non intrusive hooks multiple models 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1313-1317: Define helper: dumper worker
```python
def _dumper_worker(rank, http_port: int, stop_event):
    """Minimal distributed dumper worker: configure, step (triggers ZMQ setup), then wait."""
    dumper.configure(enable=False, server_port=str(http_port))
    dumper.step()
    stop_event.wait()
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 1320-1328: Define helper: wait for dumper http
```python
def _wait_for_dumper_http(url: str, timeout: float = 30) -> None:
    deadline = time.time() + timeout
    while time.time() < deadline:
        try:
            requests.post(f"{url}/dumper/configure", json={}, timeout=2)
            return
        except requests.ConnectionError:
            time.sleep(0.5)
    raise TimeoutError(f"Dumper HTTP server not reachable at {url}")
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 1331-1331: Define class TestZmqPortIsolation
```python
class TestZmqPortIsolation:
```
**EN:** This declaration introduces the `TestZmqPortIsolation` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestZmqPortIsolation` 测试类，并说明它通过继承承担的职责。

### Lines 1332-1332: Document the class `TestZmqPortIsolation`
```python
    """Multiple independent dumper instances (each with 2 ranks) must not conflict on ZMQ ports."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestZmqPortIsolation`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestZmqPortIsolation`的设计意图。

### Lines 1334-1334: Declare TestZmqPortIsolation configuration
```python
    NUM_INSTANCES = 3
```
**EN:** This block defines class-level settings that are shared across the `TestZmqPortIsolation` test methods.
**CN:** 该代码块定义了 `TestZmqPortIsolation` 各测试方法共享的类级配置。

### Lines 1336-1372: Run test: concurrent instances no port conflict
```python
    def test_concurrent_instances_no_port_conflict(self):
        ports = [
            find_available_port(40000 + i * 1000) for i in range(self.NUM_INSTANCES)
        ]
        stop_events = []
        threads = []
        ctx = multiprocessing.get_context("spawn")

        for port in ports:
            stop_event = ctx.Event()
            stop_events.append(stop_event)
            thread = threading.Thread(
                target=run_distributed_test,
                args=(_dumper_worker,),
                kwargs={"http_port": port, "stop_event": stop_event},
            )
            thread.start()
            threads.append(thread)

        try:
            for port in ports:
                _wait_for_dumper_http(f"http://127.0.0.1:{port}")

            for i, port in enumerate(ports):
                resp = requests.post(
                    f"http://127.0.0.1:{port}/dumper/get_state", json={}
                )
                resp.raise_for_status()
                states = resp.json()
                assert (
                    len(states) == 2
                ), f"Instance {i} (port {port}): expected 2 ranks, got {len(states)}"
        finally:
            for event in stop_events:
                event.set()
            for thread in threads:
                thread.join(timeout=10)
```
**EN:** This test method exercises concurrent instances no port conflict and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 concurrent instances no port conflict 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 1375-1375: Define class TestDumperHttp
```python
class TestDumperHttp:
```
**EN:** This declaration introduces the `TestDumperHttp` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumperHttp` 测试类，并说明它通过继承承担的职责。

### Lines 1376-1376: Document the class `TestDumperHttp`
```python
    """Test /dumper/* HTTP control — parametrized over standalone vs sglang server."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestDumperHttp`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestDumperHttp`的设计意图。

### Lines 1378-1409: Define helper: dumper http url
```python
    @pytest.fixture(scope="class", params=["standalone", "sglang"])
    def dumper_http_url(self, request):
        if request.param == "standalone":
            http_port = find_available_port(40000)
            base_url = f"http://127.0.0.1:{http_port}"
            stop_event = multiprocessing.get_context("spawn").Event()
            thread = threading.Thread(
                target=run_distributed_test,
                args=(_dumper_worker,),
                kwargs={"http_port": http_port, "stop_event": stop_event},
            )
            thread.start()
            try:
                _wait_for_dumper_http(base_url)
                yield base_url
            finally:
                stop_event.set()
                thread.join(timeout=10)
        else:
            base_url = DEFAULT_URL_FOR_TEST
            env = {**os.environ, "DUMPER_SERVER_PORT": "reuse"}
            proc = popen_launch_server(
                "Qwen/Qwen3-0.6B",
                base_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=["--max-total-tokens", "128"],
                env=env,
            )
            try:
                yield base_url
            finally:
                kill_process_tree(proc.pid)
```
**EN:** This helper function encapsulates reusable logic inside `TestDumperHttp` so the scenario stays organized. It also launches a model server for the scenario and releases spawned processes after the checks finish.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDumperHttp` 内部调用，从而让场景结构更清晰。 其中还会启动场景所需的模型服务，并在检查完成后释放已启动的进程。

### Lines 1411-1417: Define helper: post
```python
    @staticmethod
    def _post(base_url: str, method: str, **kwargs) -> list[dict]:
        resp = requests.post(f"{base_url}/dumper/{method}", json=kwargs or None)
        resp.raise_for_status()
        states = resp.json()
        assert isinstance(states, list) and len(states) >= 1
        return states
```
**EN:** This helper function encapsulates reusable logic inside `TestDumperHttp` so the scenario stays organized. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDumperHttp` 内部调用，从而让场景结构更清晰。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 1419-1429: Define helper: assert all ranks
```python
    @staticmethod
    def _assert_all_ranks(states: list[dict], path: str, expected):
        """Assert that ``state[path]`` equals ``expected`` on every rank."""
        keys = path.split(".")
        for rank, state in enumerate(states):
            val = state
            for k in keys:
                val = val[k]
            assert (
                val == expected
            ), f"rank {rank}: {path}={val!r}, expected {expected!r}"
```
**EN:** This helper function encapsulates reusable logic inside `TestDumperHttp` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestDumperHttp` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 1431-1435: Run test: configure enable toggle
```python
    def test_configure_enable_toggle(self, dumper_http_url: str):
        for enable in [True, False]:
            self._post(dumper_http_url, "configure", enable=enable)
            states = self._post(dumper_http_url, "get_state")
            self._assert_all_ranks(states, "config.enable", enable)
```
**EN:** This test method exercises configure enable toggle and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 configure enable toggle 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1437-1448: Run test: configure multi field
```python
    def test_configure_multi_field(self, dumper_http_url: str):
        self._post(
            dumper_http_url,
            "configure",
            enable=True,
            filter="layer_id == 0",
            dir="/tmp/test_http",
        )
        states = self._post(dumper_http_url, "get_state")
        self._assert_all_ranks(states, "config.enable", True)
        self._assert_all_ranks(states, "config.filter", "layer_id == 0")
        self._assert_all_ranks(states, "config.dir", "/tmp/test_http")
```
**EN:** This test method exercises configure multi field and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 configure multi field 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1450-1454: Run test: configure clear optional
```python
    def test_configure_clear_optional(self, dumper_http_url: str):
        self._post(dumper_http_url, "configure", filter="layer_id == 0")
        self._post(dumper_http_url, "configure", filter=None)
        states = self._post(dumper_http_url, "get_state")
        self._assert_all_ranks(states, "config.filter", None)
```
**EN:** This test method exercises configure clear optional and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 configure clear optional 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1456-1461: Run test: reset
```python
    def test_reset(self, dumper_http_url: str):
        self._post(dumper_http_url, "configure", enable=True)
        self._post(dumper_http_url, "reset")
        states = self._post(dumper_http_url, "get_state")
        self._assert_all_ranks(states, "dump_index", 0)
        self._assert_all_ranks(states, "step", 0)
```
**EN:** This test method exercises reset and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 reset 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1463-1477: Run test: get state
```python
    def test_get_state(self, dumper_http_url: str):
        self._post(
            dumper_http_url,
            "configure",
            enable=True,
            filter="layer_id is not None and layer_id < 3",
        )
        states = self._post(dumper_http_url, "get_state")
        self._assert_all_ranks(states, "config.enable", True)
        self._assert_all_ranks(
            states, "config.filter", "layer_id is not None and layer_id < 3"
        )
        for state in states:
            assert "dump_index" in state
            assert "step" in state
```
**EN:** This test method exercises get state and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 get state 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1479-1484: Run test: all ranks consistent
```python
    def test_all_ranks_consistent(self, dumper_http_url: str):
        self._post(dumper_http_url, "configure", enable=True, dir="/tmp/multi")
        states = self._post(dumper_http_url, "get_state")
        configs = [s["config"] for s in states]
        for rank_config in configs[1:]:
            assert rank_config == configs[0], f"rank configs diverged: {configs}"
```
**EN:** This test method exercises all ranks consistent and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all ranks consistent 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1486-1491: Run test: error unknown field
```python
    def test_error_unknown_field(self, dumper_http_url: str):
        resp = requests.post(
            f"{dumper_http_url}/dumper/configure",
            json={"nonexistent_field": 123},
        )
        assert resp.status_code == 400
```
**EN:** This test method exercises error unknown field and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 error unknown field 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 1493-1498: Run test: error unknown method
```python
    def test_error_unknown_method(self, dumper_http_url: str):
        resp = requests.post(
            f"{dumper_http_url}/dumper/nonexistent",
            json={},
        )
        assert resp.status_code == 400
```
**EN:** This test method exercises error unknown method and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 error unknown method 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 1500-1505: Run test: error wrong type
```python
    def test_error_wrong_type(self, dumper_http_url: str):
        resp = requests.post(
            f"{dumper_http_url}/dumper/configure",
            json={"enable": "not_a_bool"},
        )
        assert resp.status_code == 400
```
**EN:** This test method exercises error wrong type and verifies that the observed behavior matches the expected contract. It also issues HTTP POST requests against the exposed endpoint and talks to the server through its HTTP interface.
**CN:** 该测试方法会执行 error wrong type 场景，并验证观测到的行为是否符合预期契约。 其中还会向暴露的端点发起 HTTP POST 请求，并通过 HTTP 接口与服务交互。

### Lines 1508-1508: Define class TestRegisterForwardHookOrReplaceFn
```python
class TestRegisterForwardHookOrReplaceFn:
```
**EN:** This declaration introduces the `TestRegisterForwardHookOrReplaceFn` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestRegisterForwardHookOrReplaceFn` 测试类，并说明它通过继承承担的职责。

### Lines 1509-1517: Run test: unknown mode raises
```python
    def test_unknown_mode_raises(self):
        module = torch.nn.Linear(4, 4)
        with pytest.raises(ValueError, match="Unknown mode"):
            _register_forward_hook_or_replace_fn(
                module,
                pre_hook=lambda _mod, _input: None,
                hook=lambda _mod, _input, _output: None,
                mode="bad",
            )
```
**EN:** This test method exercises unknown mode raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 unknown mode raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 1520-1520: Define class _NonIntrusiveTestBase
```python
class _NonIntrusiveTestBase:
```
**EN:** This declaration introduces the `_NonIntrusiveTestBase` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `_NonIntrusiveTestBase` 测试类，并说明它通过继承承担的职责。

### Lines 1521-1521: Declare _NonIntrusiveTestBase configuration
```python
    _PREFIX = "non_intrusive__"
```
**EN:** This block defines class-level settings that are shared across the `_NonIntrusiveTestBase` test methods.
**CN:** 该代码块定义了 `_NonIntrusiveTestBase` 各测试方法共享的类级配置。

### Lines 1523-1529: Define helper: assert captured contains
```python
    @staticmethod
    def _assert_captured_contains(
        captured: dict, expected: list[str], prefix: str = "non_intrusive__"
    ) -> None:
        for suffix in expected:
            key = f"{prefix}{suffix}"
            assert key in captured, f"missing {key}"
```
**EN:** This helper function encapsulates reusable logic inside `_NonIntrusiveTestBase` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `_NonIntrusiveTestBase` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 1531-1543: Define helper: wrap as outer
```python
    @staticmethod
    def _wrap_as_outer(inner_cls: type) -> torch.nn.Module:
        """Wrap an inner module class as OuterModel.model, mimicking typical model nesting."""

        class OuterModel(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.model = inner_cls()

            def forward(self, *args, **kwargs):
                return self.model(*args, **kwargs)

        return OuterModel()
```
**EN:** This helper function encapsulates reusable logic inside `_NonIntrusiveTestBase` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `_NonIntrusiveTestBase` 内部调用，从而让场景结构更清晰。

### Lines 1545-1547: Define helper: make dumper
```python
    @staticmethod
    def _make_dumper(tmp_path, **overrides) -> "_Dumper":
        return _make_test_dumper(tmp_path, non_intrusive_mode="all", **overrides)
```
**EN:** This helper function encapsulates reusable logic inside `_NonIntrusiveTestBase` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `_NonIntrusiveTestBase` 内部调用，从而让场景结构更清晰。

### Lines 1549-1556: Define helper: run
```python
    def _run(self, tmp_path, inner_cls, **dumper_overrides):
        d = self._make_dumper(tmp_path, **dumper_overrides)
        model = self._wrap_as_outer(inner_cls)
        d.register_non_intrusive_dumper(model)
        x = torch.randn(2, 4)
        with d.capture_output() as captured:
            output = model(x)
        return captured, x, output
```
**EN:** This helper function encapsulates reusable logic inside `_NonIntrusiveTestBase` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `_NonIntrusiveTestBase` 内部调用，从而让场景结构更清晰。

### Lines 1559-1559: Define class TestNonIntrusiveDumper
```python
class TestNonIntrusiveDumper(_NonIntrusiveTestBase):
```
**EN:** This declaration introduces the `TestNonIntrusiveDumper` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNonIntrusiveDumper` 测试类，并说明它通过继承承担的职责。

### Lines 1560-1560: Document the class `TestNonIntrusiveDumper`
```python
    """Tests for mode='all' — hooks on every module, non_intrusive__ prefix."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNonIntrusiveDumper`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNonIntrusiveDumper`的设计意图。

### Lines 1562-1588: Run test: basic inputs and outputs
```python
    def test_basic_inputs_and_outputs(self, tmp_path):
        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.linear = torch.nn.Linear(4, 4)
                self.relu = torch.nn.ReLU()

            def forward(self, x):
                return self.relu(self.linear(x))

        captured, x, output = self._run(tmp_path, Inner)

        self._assert_captured_contains(
            captured,
            [
                "output",
                "inputs.0",
                "model.output",
                "model.inputs.0",
                "model.linear.output",
                "model.linear.inputs.0",
                "model.relu.output",
                "model.relu.inputs.0",
            ],
        )
        P = self._PREFIX
        assert torch.allclose(captured[f"{P}output"]["value"], output)
```
**EN:** This test method exercises basic inputs and outputs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 basic inputs and outputs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1590-1625: Run test: inputs dumped before forward
```python
    def test_inputs_dumped_before_forward(self, tmp_path):
        """Inputs are captured *before* forward(); in-place mutation must not affect them."""

        class Mutator(torch.nn.Module):
            def forward(self, x):
                x.fill_(999.0)
                return x

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.mutator = Mutator()

            def forward(self, x):
                return self.mutator(x)

        d = self._make_dumper(tmp_path)
        model = self._wrap_as_outer(Inner)
        d.register_non_intrusive_dumper(model)

        x = torch.randn(2, 4)
        original_x = x.clone()
        with d.capture_output() as captured:
            model(x)

        P = self._PREFIX
        dumped_input = captured[f"{P}model.mutator.inputs.0"]["value"]
        assert torch.allclose(dumped_input, original_x), (
            f"pre-hook should capture inputs before forward mutates them; "
            f"got {dumped_input} but expected {original_x}"
        )

        dumped_output = captured[f"{P}model.mutator.output"]["value"]
        assert (
            dumped_output == 999.0
        ).all(), "post-hook should capture outputs after forward"
```
**EN:** This test method exercises inputs dumped before forward and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 inputs dumped before forward 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1627-1676: Run test: hooks all module levels
```python
    def test_hooks_all_module_levels(self, tmp_path):
        class Attention(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.qkv_proj = torch.nn.Linear(4, 12)
                self.o_proj = torch.nn.Linear(4, 4)

            def forward(self, x):
                _qkv = self.qkv_proj(x)
                return self.o_proj(x)

        class Layer(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.self_attn = Attention()
                self.mlp = torch.nn.Linear(4, 4)

            def forward(self, x):
                x = self.self_attn(x)
                return self.mlp(x)

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.layers = torch.nn.ModuleList([Layer()])

            def forward(self, x):
                for layer in self.layers:
                    x = layer(x)
                return x

        captured, x, output = self._run(tmp_path, Inner)

        self._assert_captured_contains(
            captured,
            [
                "output",
                "model.output",
                "model.layers.0.output",
                "model.layers.0.self_attn.output",
                "model.layers.0.self_attn.qkv_proj.output",
                "model.layers.0.self_attn.o_proj.output",
                "model.layers.0.mlp.output",
                "model.layers.0.self_attn.qkv_proj.inputs.0",
                "model.layers.0.self_attn.o_proj.inputs.0",
                "model.layers.0.mlp.inputs.0",
            ],
        )
        P = self._PREFIX
        assert f"{P}model.layers.output" not in captured
```
**EN:** This test method exercises hooks all module levels and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 hooks all module levels 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1678-1699: Run test: multi tensor tuple output
```python
    def test_multi_tensor_tuple_output(self, tmp_path):
        class TupleModule(torch.nn.Module):
            def forward(self, x):
                return x, x * 2

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.split = TupleModule()
                self.linear = torch.nn.Linear(4, 4)

            def forward(self, x):
                a, b = self.split(x)
                return self.linear(a + b)

        captured, x, output = self._run(tmp_path, Inner)

        assert "non_intrusive__model.split.output.0" in captured
        assert "non_intrusive__model.split.output.1" in captured
        assert torch.allclose(
            captured["non_intrusive__model.split.output.0"]["value"], x
        )
```
**EN:** This test method exercises multi tensor tuple output and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multi tensor tuple output 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1701-1717: Run test: single tensor tuple collapses
```python
    def test_single_tensor_tuple_collapses(self, tmp_path):
        class SingleTupleModule(torch.nn.Module):
            def forward(self, x):
                return (x * 3,)

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.wrap = SingleTupleModule()

            def forward(self, x):
                return self.wrap(x)[0]

        captured, x, output = self._run(tmp_path, Inner)

        assert "non_intrusive__model.wrap.output" in captured
        assert "non_intrusive__model.wrap.output.0" not in captured
```
**EN:** This test method exercises single tensor tuple collapses and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single tensor tuple collapses 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1719-1736: Run test: multiple forward inputs
```python
    def test_multiple_forward_inputs(self, tmp_path):
        class TwoInputModule(torch.nn.Module):
            def forward(self, x, mask):
                return x * mask

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.mul = TwoInputModule()

            def forward(self, x):
                mask = torch.ones_like(x)
                return self.mul(x, mask)

        captured, x, output = self._run(tmp_path, Inner)

        assert "non_intrusive__model.mul.inputs.0" in captured
        assert "non_intrusive__model.mul.inputs.1" in captured
```
**EN:** This test method exercises multiple forward inputs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple forward inputs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1738-1757: Run test: none output only dumps inputs
```python
    def test_none_output_only_dumps_inputs(self, tmp_path):
        class NoneModule(torch.nn.Module):
            def forward(self, x):
                return None

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.sink = NoneModule()

            def forward(self, x):
                self.sink(x)
                return x

        captured, x, output = self._run(tmp_path, Inner)

        assert "non_intrusive__model.sink.inputs.0" in captured
        assert not any(
            k.startswith("non_intrusive__model.sink.output") for k in captured
        )
```
**EN:** This test method exercises none output only dumps inputs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 none output only dumps inputs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1759-1778: Run test: non tensor value silently skipped
```python
    def test_non_tensor_value_silently_skipped(self, tmp_path):
        class IntModule(torch.nn.Module):
            def forward(self, x):
                return 42

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.const = IntModule()

            def forward(self, x):
                self.const(x)
                return x

        captured, x, output = self._run(tmp_path, Inner)

        assert "non_intrusive__model.const.inputs.0" in captured
        assert not any(
            k.startswith("non_intrusive__model.const.output") for k in captured
        )
```
**EN:** This test method exercises non tensor value silently skipped and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non tensor value silently skipped 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1780-1794: Run test: root module name no malformed dots
```python
    def test_root_module_name_no_malformed_dots(self, tmp_path):
        d = self._make_dumper(tmp_path)
        model = torch.nn.Linear(4, 4)
        d.register_non_intrusive_dumper(model)

        x = torch.randn(2, 4)
        with d.capture_output() as captured:
            model(x)

        for key in captured:
            assert not key.startswith("non_intrusive__."), f"malformed key: {key}"
            assert ".." not in key, f"double dot in key: {key}"

        assert "non_intrusive__output" in captured
        assert "non_intrusive__inputs.0" in captured
```
**EN:** This test method exercises root module name no malformed dots and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 root module name no malformed dots 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1796-1812: Run test: respects dumper filter
```python
    def test_respects_dumper_filter(self, tmp_path):
        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.linear = torch.nn.Linear(4, 4)
                self.relu = torch.nn.ReLU()

            def forward(self, x):
                return self.relu(self.linear(x))

        captured, x, output = self._run(
            tmp_path, Inner, filter="name == 'non_intrusive__model.linear.output'"
        )

        assert "non_intrusive__model.linear.output" in captured
        assert "non_intrusive__model.relu.output" not in captured
        assert "non_intrusive__model.linear.inputs.0" not in captured
```
**EN:** This test method exercises respects dumper filter and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 respects dumper filter 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1814-1832: Run test: disabled dumper no output
```python
    def test_disabled_dumper_no_output(self, tmp_path):
        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.linear = torch.nn.Linear(4, 4)

            def forward(self, x):
                return self.linear(x)

        d = self._make_dumper(tmp_path)
        d.configure(enable=False)
        model = self._wrap_as_outer(Inner)
        d.register_non_intrusive_dumper(model)

        x = torch.randn(2, 4)
        with d.capture_output() as captured:
            model(x)

        assert len(captured) == 0
```
**EN:** This test method exercises disabled dumper no output and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 disabled dumper no output 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1835-1847: Define helper: make forward batch
```python
def _make_forward_batch():
    from sglang.srt.model_executor.forward_batch_info import ForwardBatch, ForwardMode

    return ForwardBatch(
        forward_mode=ForwardMode.DECODE,
        batch_size=2,
        input_ids=torch.tensor([10, 20]),
        req_pool_indices=torch.zeros(2, dtype=torch.long),
        seq_lens=torch.tensor([5, 6]),
        out_cache_loc=torch.zeros(2, dtype=torch.long),
        seq_lens_sum=11,
        positions=torch.tensor([0, 1]),
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 1850-1850: Define class TestNonIntrusiveDumperConfigMode
```python
class TestNonIntrusiveDumperConfigMode(_NonIntrusiveTestBase):
```
**EN:** This declaration introduces the `TestNonIntrusiveDumperConfigMode` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNonIntrusiveDumperConfigMode` 测试类，并说明它通过继承承担的职责。

### Lines 1851-1871: Define helper: build model
```python
    @staticmethod
    def _build_model() -> torch.nn.Module:
        class SubLayer(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.linear = torch.nn.Linear(4, 4)

            def forward(self, forward_batch):
                return self.linear(
                    forward_batch.input_ids.float().unsqueeze(-1).expand(-1, 4)
                )

        class Root(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.layer = SubLayer()

            def forward(self, forward_batch):
                return self.layer(forward_batch)

        return Root()
```
**EN:** This helper function encapsulates reusable logic inside `TestNonIntrusiveDumperConfigMode` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNonIntrusiveDumperConfigMode` 内部调用，从而让场景结构更清晰。

### Lines 1873-1880: Define helper: run
```python
    def _run(self, tmp_path, mode: str) -> tuple:
        d = _make_test_dumper(tmp_path, non_intrusive_mode=mode)
        model = self._build_model()
        d.register_non_intrusive_dumper(model)
        forward_batch = _make_forward_batch()
        with d.capture_output() as captured:
            model(forward_batch)
        return captured, forward_batch
```
**EN:** This helper function encapsulates reusable logic inside `TestNonIntrusiveDumperConfigMode` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestNonIntrusiveDumperConfigMode` 内部调用，从而让场景结构更清晰。

### Lines 1882-1884: Run test: off mode
```python
    def test_off_mode(self, tmp_path):
        captured, _ = self._run(tmp_path, "off")
        assert len(captured) == 0
```
**EN:** This test method exercises off mode and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 off mode 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1886-1898: Run test: core mode
```python
    def test_core_mode(self, tmp_path):
        captured, fb = self._run(tmp_path, "core")

        # core fields dumped with clean names
        assert "input_ids" in captured
        assert "positions" in captured
        assert "seq_lens" in captured
        assert torch.equal(captured["input_ids"]["value"], fb.input_ids)
        assert torch.equal(captured["positions"]["value"], fb.positions)
        assert torch.equal(captured["seq_lens"]["value"], fb.seq_lens)

        # nothing with non_intrusive__ prefix
        assert not any(k.startswith("non_intrusive__") for k in captured)
```
**EN:** This test method exercises core mode and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 core mode 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1900-1925: Run test: all mode
```python
    def test_all_mode(self, tmp_path):
        captured, fb = self._run(tmp_path, "all")

        # core fields dumped with clean names
        assert "input_ids" in captured
        assert "positions" in captured
        assert "seq_lens" in captured
        assert torch.equal(captured["input_ids"]["value"], fb.input_ids)
        assert torch.equal(captured["positions"]["value"], fb.positions)
        assert torch.equal(captured["seq_lens"]["value"], fb.seq_lens)

        # core fields NOT duplicated with prefix
        for field in ("input_ids", "positions", "seq_lens"):
            assert not any(
                k.startswith("non_intrusive__") and k.endswith(field) for k in captured
            )

        # ForwardBatch skipped on sub-modules (no duplication)
        assert not any(
            k.startswith("non_intrusive__layer.inputs.") and "seq_lens" in k
            for k in captured
        ), f"ForwardBatch skipped on sub-module, got: {list(captured.keys())}"

        # regular tensor outputs on sub-modules still dumped
        assert "non_intrusive__layer.linear.output" in captured
        assert "non_intrusive__layer.output" in captured
```
**EN:** This test method exercises all mode and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 all mode 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1928-1928: Define class _LayerWithNumber
```python
class _LayerWithNumber(torch.nn.Module):
```
**EN:** This declaration introduces the `_LayerWithNumber` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `_LayerWithNumber` 测试类，并说明它通过继承承担的职责。

### Lines 1929-1929: Document the class `_LayerWithNumber`
```python
    """Test helper: module with a ``layer_number`` attribute (Megatron style)."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `_LayerWithNumber`.
**CN:** 该代码块保存说明性文本，用于解释周围class `_LayerWithNumber`的设计意图。

### Lines 1931-1934: Define helper: init
```python
    def __init__(self, layer_number: int):
        super().__init__()
        self.layer_number = layer_number
        self.linear = torch.nn.Linear(4, 4)
```
**EN:** This helper function encapsulates reusable logic inside `_LayerWithNumber` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `_LayerWithNumber` 内部调用，从而让场景结构更清晰。

### Lines 1936-1937: Define helper: forward
```python
    def forward(self, x):
        return self.linear(x)
```
**EN:** This helper function encapsulates reusable logic inside `_LayerWithNumber` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `_LayerWithNumber` 内部调用，从而让场景结构更清晰。

### Lines 1940-1940: Define class TestNonIntrusiveLayerIdCtx
```python
class TestNonIntrusiveLayerIdCtx(_NonIntrusiveTestBase):
```
**EN:** This declaration introduces the `TestNonIntrusiveLayerIdCtx` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNonIntrusiveLayerIdCtx` 测试类，并说明它通过继承承担的职责。

### Lines 1941-1941: Document the class `TestNonIntrusiveLayerIdCtx`
```python
    """Tests for automatic layer_id context injection via set_ctx."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestNonIntrusiveLayerIdCtx`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestNonIntrusiveLayerIdCtx`的设计意图。

### Lines 1943-1969: Run test: layer id from layer number
```python
    def test_layer_id_from_layer_number(self, tmp_path):
        """Megatron PP: layer_number (1-based global) -> layer_id = layer_number - 1."""

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.layers = torch.nn.ModuleList(
                    [_LayerWithNumber(10), _LayerWithNumber(11)]
                )

            def forward(self, x):
                for layer in self.layers:
                    x = layer(x)
                return x

        captured, x, output = self._run(tmp_path, Inner)

        layer0_key = "non_intrusive__model.layers.0.linear.output"
        layer1_key = "non_intrusive__model.layers.1.linear.output"
        assert layer0_key in captured
        assert layer1_key in captured
        assert captured[layer0_key]["meta"]["layer_id"] == 9
        assert captured[layer1_key]["meta"]["layer_id"] == 10

        root_key = "non_intrusive__output"
        assert root_key in captured
        assert "layer_id" not in captured[root_key]["meta"]
```
**EN:** This test method exercises layer id from layer number and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 layer id from layer number 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1971-1997: Run test: layer id from layer id attr
```python
    def test_layer_id_from_layer_id_attr(self, tmp_path):
        """SGLang style: module has layer_id attribute directly."""

        class Layer(torch.nn.Module):
            def __init__(self, layer_id: int):
                super().__init__()
                self.layer_id = layer_id
                self.linear = torch.nn.Linear(4, 4)

            def forward(self, x):
                return self.linear(x)

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.layers = torch.nn.ModuleList([Layer(5)])

            def forward(self, x):
                for layer in self.layers:
                    x = layer(x)
                return x

        captured, x, output = self._run(tmp_path, Inner)

        layer_key = "non_intrusive__model.layers.0.linear.output"
        assert layer_key in captured
        assert captured[layer_key]["meta"]["layer_id"] == 5
```
**EN:** This test method exercises layer id from layer id attr and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 layer id from layer id attr 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 1999-2027: Run test: layer id fallback from module name
```python
    def test_layer_id_fallback_from_module_name(self, tmp_path):
        """layers.N modules without layer_number/layer_id -> layer_id from module name."""

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.layers = torch.nn.ModuleList(
                    [torch.nn.Linear(4, 4), torch.nn.Linear(4, 4)]
                )

            def forward(self, x):
                for layer in self.layers:
                    x = layer(x)
                return x

        captured, x, output = self._run(tmp_path, Inner)

        assert len(captured) > 0
        input_keys: list[str] = [
            k for k in captured if "model.layers." in k and "inputs" in k
        ]
        assert len(input_keys) > 0
        for key in input_keys:
            meta = captured[key]["meta"]
            assert "layer_id" in meta, f"{key} missing layer_id"
            if "layers.0" in key:
                assert meta["layer_id"] == 0
            elif "layers.1" in key:
                assert meta["layer_id"] == 1
```
**EN:** This test method exercises layer id fallback from module name and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 layer id fallback from module name 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2029-2049: Run test: filter by layer id
```python
    def test_filter_by_layer_id(self, tmp_path):
        """filter='layer_id == 0' keeps only layer 0 dumps."""

        class Inner(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.layers = torch.nn.ModuleList(
                    [_LayerWithNumber(1), _LayerWithNumber(2)]
                )

            def forward(self, x):
                for layer in self.layers:
                    x = layer(x)
                return x

        captured, x, output = self._run(tmp_path, Inner, filter="layer_id == 0")

        layer0_keys = [k for k in captured if "layers.0" in k]
        layer1_keys = [k for k in captured if "layers.1" in k]
        assert len(layer0_keys) > 0, "layer 0 dumps should be kept"
        assert len(layer1_keys) == 0, f"layer 1 dumps should be filtered: {layer1_keys}"
```
**EN:** This test method exercises filter by layer id and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter by layer id 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2052-2052: Define class TestDumperE2E
```python
class TestDumperE2E:
```
**EN:** This declaration introduces the `TestDumperE2E` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumperE2E` 测试类，并说明它通过继承承担的职责。

### Lines 2053-2108: Run test: step and non intrusive hooks (part 1)
```python
    def test_step_and_non_intrusive_hooks(self, tmp_path):
        base_url = DEFAULT_URL_FOR_TEST
        dump_dir = str(tmp_path)
        env = {
            **os.environ,
            "DUMPER_SERVER_PORT": "reuse",
        }
        proc = popen_launch_server(
            "Qwen/Qwen3-0.6B",
            base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=["--tp", "2", "--max-total-tokens", "128"],
            env=env,
        )
        try:
            states = requests.post(f"{base_url}/dumper/get_state", json={}).json()
            assert len(states) == 2, f"Expected 2 ranks (tp=2), got {len(states)}"
            for state in states:
                assert state["config"]["enable"] is False
                assert state["step"] == 0

            requests.post(
                f"{base_url}/dumper/configure",
                json={"enable": True, "dir": dump_dir},
            ).raise_for_status()

            states = requests.post(f"{base_url}/dumper/get_state", json={}).json()
            assert len(states) == 2
            for rank, state in enumerate(states):
                assert (
                    state["config"]["enable"] is True
                ), f"rank {rank}: enable should be True after configure"
                assert state["config"]["dir"] == dump_dir

            resp = requests.post(
                f"{base_url}/generate",
                json={"text": "Hello", "sampling_params": {"max_new_tokens": 8}},
            )
            assert resp.status_code == 200, f"Generate failed: {resp.text}"

            states = requests.post(f"{base_url}/dumper/get_state", json={}).json()
            assert len(states) == 2
            steps = [s["step"] for s in states]
            for rank, step in enumerate(steps):
                assert step > 0, f"rank {rank}: step should be > 0, got {step}"
            assert steps[0] == steps[1], f"step mismatch across ranks: {steps}"

            dump_files = list(Path(dump_dir).glob("dump_*/*.pt"))
            assert len(dump_files) > 0, f"No dump files in {dump_dir}"
            filenames = {f.name for f in dump_files}

            for field in ("input_ids", "positions", "rids"):
                assert any(f"name={field}" in f for f in filenames), (
                    f"Missing {field} dump from non-intrusive hooks, "
                    f"got: {sorted(filenames)[:10]}"
                )
```
**EN:** This test method exercises step and non intrusive hooks and verifies that the observed behavior matches the expected contract. It also launches a model server for the scenario and issues HTTP POST requests against the exposed endpoint.
**CN:** 该测试方法会执行 step and non intrusive hooks 场景，并验证观测到的行为是否符合预期契约。 其中还会启动场景所需的模型服务，并向暴露的端点发起 HTTP POST 请求。

### Lines 2110-2150: Run test: step and non intrusive hooks (part 2)
```python
            for rank in range(2):
                assert any(
                    f"rank={rank}" in f for f in filenames
                ), f"No dump files for rank {rank}"

            sample_file = dump_files[0]
            loaded = torch.load(sample_file, map_location="cpu", weights_only=False)
            assert isinstance(loaded, dict), f"Expected dict, got {type(loaded)}"
            assert (
                "value" in loaded and "meta" in loaded
            ), f"Missing value/meta keys: {loaded.keys()}"
            assert "name" in loaded["meta"]
            assert "rank" in loaded["meta"]
            assert "step" in loaded["meta"]

            par = loaded["meta"].get("sglang_parallel_info", {})
            expected_keys = [
                "tp_rank",
                "tp_size",
                "pp_rank",
                "pp_size",
                "moe_ep_rank",
                "moe_ep_size",
                "moe_tp_rank",
                "moe_tp_size",
                "moe_dp_rank",
                "moe_dp_size",
                "enable_dp_attention",
                "attn_tp_rank",
                "attn_tp_size",
                "attn_dp_rank",
                "attn_dp_size",
                "local_attn_dp_rank",
                "local_attn_dp_size",
                "attn_cp_rank",
                "attn_cp_size",
            ]
            for key in expected_keys:
                assert (
                    key in par
                ), f"Missing {key} in sglang_parallel_info, got: {sorted(par)}"
```
**EN:** This test method exercises step and non intrusive hooks and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 step and non intrusive hooks 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2152-2165: Run test: step and non intrusive hooks (part 3)
```python
            rids_files = [f for f in dump_files if "name=rids" in f.name]
            rids_loaded = torch.load(
                rids_files[0], map_location="cpu", weights_only=False
            )
            rids_value = rids_loaded["value"]
            assert isinstance(
                rids_value, list
            ), f"rids should be a list, got {type(rids_value)}"
            assert len(rids_value) > 0, "rids should be non-empty"
            assert all(
                isinstance(r, str) for r in rids_value
            ), f"each rid should be a str, got {[type(r) for r in rids_value]}"
        finally:
            kill_process_tree(proc.pid)
```
**EN:** This test method exercises step and non intrusive hooks and verifies that the observed behavior matches the expected contract. It also releases spawned processes after the checks finish and checks concrete expectations with assertions.
**CN:** 该测试方法会执行 step and non intrusive hooks 场景，并验证观测到的行为是否符合预期契约。 其中还会在检查完成后释放已启动的进程，并通过断言检查明确的预期。

### Lines 2168-2168: Define class TestRegisterForwardHook
```python
class TestRegisterForwardHook:
```
**EN:** This declaration introduces the `TestRegisterForwardHook` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestRegisterForwardHook` 测试类，并说明它通过继承承担的职责。

### Lines 2169-2202: Run test: handles removable
```python
    @pytest.mark.parametrize("mode", ["hook", "replace_fn"])
    def test_handles_removable(self, mode):
        call_log: list[str] = []

        def pre_hook(_module, _args, _kwargs):
            call_log.append("pre")

        def hook(_module, _input, _output):
            call_log.append("post")

        module = torch.nn.Linear(4, 4)
        handles = _register_forward_hook_or_replace_fn(
            module,
            pre_hook=pre_hook,
            hook=hook,
            mode=mode,
        )

        x = torch.randn(2, 4)
        if mode == "hook":
            module(x)
        else:
            module.forward(x)
        assert call_log == ["pre", "post"]

        call_log.clear()
        for h in handles:
            h.remove()

        if mode == "hook":
            module(x)
        else:
            module.forward(x)
        assert call_log == []
```
**EN:** This test method exercises handles removable and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 handles removable 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2204-2236: Run test: kwargs passed to pre hook
```python
    @pytest.mark.parametrize("mode", ["hook", "replace_fn"])
    def test_kwargs_passed_to_pre_hook(self, mode):
        received: list[tuple] = []

        class KwargsModule(torch.nn.Module):
            def forward(self, x, *, scale=1.0):
                return x * scale

        def pre_hook(_module, _args, _kwargs):
            received.append((_args, _kwargs))

        def hook(_module, _input, _output):
            pass

        module = KwargsModule()
        _register_forward_hook_or_replace_fn(
            module,
            pre_hook=pre_hook,
            hook=hook,
            mode=mode,
        )

        x = torch.randn(2, 4)
        if mode == "hook":
            module(x, scale=2.0)
        else:
            module.forward(x, scale=2.0)

        assert len(received) == 1
        args, kwargs = received[0]
        assert len(args) == 1
        assert torch.equal(args[0], x)
        assert kwargs == {"scale": 2.0}
```
**EN:** This test method exercises kwargs passed to pre hook and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 kwargs passed to pre hook 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2238-2250: Run test: replace fn remove asserts on rewrap
```python
    def test_replace_fn_remove_asserts_on_rewrap(self):
        module = torch.nn.Linear(4, 4)
        handles = _register_forward_hook_or_replace_fn(
            module,
            pre_hook=lambda _m, _a, _kw: None,
            hook=lambda _m, _i, _o: None,
            mode="replace_fn",
        )

        module.forward = lambda *a, **kw: None

        with pytest.raises(AssertionError):
            handles[0].remove()
```
**EN:** This test method exercises replace fn remove asserts on rewrap and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 replace fn remove asserts on rewrap 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2253-2253: Define class TestPluginCoreFields
```python
class TestPluginCoreFields:
```
**EN:** This declaration introduces the `TestPluginCoreFields` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestPluginCoreFields` 测试类，并说明它通过继承承担的职责。

### Lines 2254-2258: Run test: sglang core fields
```python
    def test_sglang_core_fields(self):
        plugin = _SGLangPlugin()
        assert plugin.core_fields() == frozenset(
            {"input_ids", "positions", "seq_lens", "req_pool_indices", "rids"}
        )
```
**EN:** This test method exercises sglang core fields and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 sglang core fields 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2260-2264: Run test: megatron core fields
```python
    def test_megatron_core_fields(self):
        plugin = _MegatronPlugin()
        assert plugin.core_fields() == frozenset(
            {"input_ids", "position_ids", "cu_seqlens_q", "cu_seqlens_kv", "qkv_format"}
        )
```
**EN:** This test method exercises megatron core fields and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 megatron core fields 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2267-2267: Define class TestMegatronConvertValue
```python
class TestMegatronConvertValue:
```
**EN:** This declaration introduces the `TestMegatronConvertValue` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestMegatronConvertValue` 测试类，并说明它通过继承承担的职责。

### Lines 2268-2279: Define helper: patch megatron
```python
    @pytest.fixture(autouse=True)
    def _patch_megatron(self, monkeypatch):
        class FakePackedSeqParams:
            def __init__(self, **kwargs):
                for k, v in kwargs.items():
                    setattr(self, k, v)

        monkeypatch.setattr(_MegatronPlugin, "_available", True)
        monkeypatch.setattr(
            _MegatronPlugin, "PackedSeqParams", FakePackedSeqParams, raising=False
        )
        self._FakePackedSeqParams = FakePackedSeqParams
```
**EN:** This helper function encapsulates reusable logic inside `TestMegatronConvertValue` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestMegatronConvertValue` 内部调用，从而让场景结构更清晰。

### Lines 2281-2293: Run test: extracts packed seq params
```python
    def test_extracts_packed_seq_params(self):
        plugin = _MegatronPlugin()
        cu_q = torch.tensor([0, 3, 7])
        cu_kv = torch.tensor([0, 3, 7])
        value = self._FakePackedSeqParams(
            cu_seqlens_q=cu_q, cu_seqlens_kv=cu_kv, qkv_format="thd"
        )

        result = plugin.convert_value(value, skip_forward_batch=False)
        assert set(result.keys()) == {"cu_seqlens_q", "cu_seqlens_kv", "qkv_format"}
        assert torch.equal(result["cu_seqlens_q"], cu_q)
        assert torch.equal(result["cu_seqlens_kv"], cu_kv)
        assert result["qkv_format"] == "thd"
```
**EN:** This test method exercises extracts packed seq params and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 extracts packed seq params 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2295-2298: Run test: non packed returns none
```python
    def test_non_packed_returns_none(self):
        plugin = _MegatronPlugin()
        assert plugin.convert_value(torch.randn(4), skip_forward_batch=False) is None
        assert plugin.convert_value("hello", skip_forward_batch=False) is None
```
**EN:** This test method exercises non packed returns none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non packed returns none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2301-2301: Define class TestNonIntrusiveKwargsModel
```python
class TestNonIntrusiveKwargsModel(_NonIntrusiveTestBase):
```
**EN:** This declaration introduces the `TestNonIntrusiveKwargsModel` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestNonIntrusiveKwargsModel` 测试类，并说明它通过继承承担的职责。

### Lines 2302-2319: Run test: kwargs core fields
```python
    def test_kwargs_core_fields(self, tmp_path):
        class KwargsModel(torch.nn.Module):
            def forward(self, *, input_ids, position_ids):
                return input_ids + position_ids

        model = KwargsModel()
        d = _make_test_dumper(tmp_path, non_intrusive_mode="core")
        d.register_non_intrusive_dumper(model)

        ids = torch.randn(4)
        pos = torch.randn(4)
        with d.capture_output() as captured:
            model(input_ids=ids, position_ids=pos)

        assert "input_ids" in captured
        assert "position_ids" in captured
        assert torch.equal(captured["input_ids"]["value"], ids)
        assert torch.equal(captured["position_ids"]["value"], pos)
```
**EN:** This test method exercises kwargs core fields and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 kwargs core fields 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2321-2340: Run test: kwargs all mode
```python
    def test_kwargs_all_mode(self, tmp_path):
        class KwargsModel(torch.nn.Module):
            def forward(self, *, input_ids, position_ids, custom_value):
                return input_ids + position_ids + custom_value

        model = KwargsModel()
        d = _make_test_dumper(tmp_path, non_intrusive_mode="all")
        d.register_non_intrusive_dumper(model)

        ids = torch.randn(4)
        pos = torch.randn(4)
        custom = torch.randn(4)
        with d.capture_output() as captured:
            model(input_ids=ids, position_ids=pos, custom_value=custom)

        assert "input_ids" in captured
        assert "position_ids" in captured

        P = self._PREFIX
        assert f"{P}inputs.custom_value" in captured
```
**EN:** This test method exercises kwargs all mode and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 kwargs all mode 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2342-2359: Run test: mixed args and kwargs
```python
    def test_mixed_args_and_kwargs(self, tmp_path):
        class MixedModel(torch.nn.Module):
            def forward(self, x, *, input_ids):
                return x + input_ids

        model = MixedModel()
        d = _make_test_dumper(tmp_path, non_intrusive_mode="all")
        d.register_non_intrusive_dumper(model)

        x = torch.randn(4)
        ids = torch.randn(4)
        with d.capture_output() as captured:
            model(x, input_ids=ids)

        assert "input_ids" in captured

        P = self._PREFIX
        assert f"{P}inputs.0" in captured
```
**EN:** This test method exercises mixed args and kwargs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 mixed args and kwargs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2361-2396: Run test: packed seq params core fields
```python
    def test_packed_seq_params_core_fields(self, tmp_path, monkeypatch):
        class FakePackedSeqParams:
            def __init__(self, **kwargs):
                for k, v in kwargs.items():
                    setattr(self, k, v)

        monkeypatch.setattr(_MegatronPlugin, "_available", True)
        monkeypatch.setattr(
            _MegatronPlugin, "PackedSeqParams", FakePackedSeqParams, raising=False
        )

        class MegatronLikeModel(torch.nn.Module):
            def forward(self, *, input_ids, packed_seq_params):
                return input_ids

        model = MegatronLikeModel()
        d = _make_test_dumper(tmp_path, non_intrusive_mode="core")
        d.register_non_intrusive_dumper(model)

        ids = torch.randn(4)
        cu_q = torch.tensor([0, 3, 7])
        cu_kv = torch.tensor([0, 3, 7])
        psp = FakePackedSeqParams(
            cu_seqlens_q=cu_q, cu_seqlens_kv=cu_kv, qkv_format="thd"
        )
        with d.capture_output() as captured:
            model(input_ids=ids, packed_seq_params=psp)

        assert "input_ids" in captured
        assert torch.equal(captured["input_ids"]["value"], ids)
        assert "cu_seqlens_q" in captured
        assert torch.equal(captured["cu_seqlens_q"]["value"], cu_q)
        assert "cu_seqlens_kv" in captured
        assert torch.equal(captured["cu_seqlens_kv"]["value"], cu_kv)
        assert "qkv_format" in captured
        assert captured["qkv_format"]["value"] == "thd"
```
**EN:** This test method exercises packed seq params core fields and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 packed seq params core fields 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2399-2399: Define class TestDumperDims
```python
class TestDumperDims:
```
**EN:** This declaration introduces the `TestDumperDims` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestDumperDims` 测试类，并说明它通过继承承担的职责。

### Lines 2400-2414: Run test: dims in meta not filename
```python
    def test_dims_in_meta_not_filename(self, tmp_path) -> None:
        dumper = _make_test_dumper(tmp_path)
        tensor = torch.randn(4, 8)
        dumper.dump("hidden", tensor, dims="b h(tp)")
        dumper.step()

        exp_dir = tmp_path / dumper._config.exp_name
        pt_files = list(exp_dir.glob("*.pt"))
        assert len(pt_files) == 1

        assert "dims" not in pt_files[0].stem

        data = torch.load(pt_files[0], weights_only=False)
        assert "dims" in data["meta"]
        assert data["meta"]["dims"] == "b h(tp)"
```
**EN:** This test method exercises dims in meta not filename and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dims in meta not filename 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2416-2443: Run test: dims grad override
```python
    def test_dims_grad_override(self, tmp_path) -> None:
        dumper = _Dumper(
            config=DumperConfig(
                enable=True,
                dir=str(tmp_path),
                enable_grad=True,
            )
        )

        tensor = torch.randn(4, 8, requires_grad=True)
        dumper.dump("hidden", tensor, dims="b h(tp)", dims_grad="b h(tp:partial)")
        dumper.step()

        tensor.backward(torch.ones_like(tensor))

        exp_dir = tmp_path / dumper._config.exp_name
        pt_files = sorted(exp_dir.glob("*.pt"))
        assert len(pt_files) == 2

        value_file = [f for f in pt_files if "grad__" not in f.stem][0]
        grad_file = [f for f in pt_files if "grad__" in f.stem][0]

        value_data = torch.load(value_file, weights_only=False)
        assert value_data["meta"]["dims"] == "b h(tp)"
        assert value_data["meta"]["dims_grad"] == "b h(tp:partial)"

        grad_data = torch.load(grad_file, weights_only=False)
        assert grad_data["meta"]["dims"] == "b h(tp:partial)"
```
**EN:** This test method exercises dims grad override and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dims grad override 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2445-2463: Run test: dims grad inherits
```python
    def test_dims_grad_inherits(self, tmp_path) -> None:
        dumper = _Dumper(
            config=DumperConfig(
                enable=True,
                dir=str(tmp_path),
                enable_grad=True,
            )
        )

        tensor = torch.randn(4, 8, requires_grad=True)
        dumper.dump("hidden", tensor, dims="b h(tp)")
        dumper.step()

        tensor.backward(torch.ones_like(tensor))

        exp_dir = tmp_path / dumper._config.exp_name
        grad_file = [f for f in exp_dir.glob("*.pt") if "grad__" in f.stem][0]
        grad_data = torch.load(grad_file, weights_only=False)
        assert grad_data["meta"]["dims"] == "b h(tp)"
```
**EN:** This test method exercises dims grad inherits and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dims grad inherits 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2466-2466: Define class TestCtxDecorator
```python
class TestCtxDecorator:
```
**EN:** This declaration introduces the `TestCtxDecorator` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCtxDecorator` 测试类，并说明它通过继承承担的职责。

### Lines 2467-2483: Run test: ctx dynamic lambda
```python
    def test_ctx_dynamic_lambda(self, tmp_path: Path) -> None:
        d = _make_test_dumper(tmp_path)

        class FakeLayer:
            def __init__(self, layer_id: int) -> None:
                self.layer_id = layer_id

            @d.ctx(lambda self: dict(layer_id=self.layer_id))
            def forward(self, x: torch.Tensor) -> torch.Tensor:
                d.dump("hidden", x)
                return x

        layer = FakeLayer(layer_id=42)
        layer.forward(torch.randn(3))

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["layer_id=42"])
```
**EN:** This test method exercises ctx dynamic lambda and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 ctx dynamic lambda 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2485-2496: Run test: ctx static kwargs
```python
    def test_ctx_static_kwargs(self, tmp_path: Path) -> None:
        d = _make_test_dumper(tmp_path)

        @d.ctx(phase="decode")
        def decode_step(x: torch.Tensor) -> torch.Tensor:
            d.dump("step_out", x)
            return x

        decode_step(torch.randn(3))

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["phase=decode"])
```
**EN:** This test method exercises ctx static kwargs and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 ctx static kwargs 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2498-2508: Run test: ctx clears on exception
```python
    def test_ctx_clears_on_exception(self, tmp_path: Path) -> None:
        d = _make_test_dumper(tmp_path)

        @d.ctx(phase="train")
        def buggy_fn() -> None:
            raise RuntimeError("boom")

        with pytest.raises(RuntimeError, match="boom"):
            buggy_fn()

        assert d._state.global_ctx == {}
```
**EN:** This test method exercises ctx clears on exception and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 ctx clears on exception 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2510-2514: Run test: ctx rejects mixed args
```python
    def test_ctx_rejects_mixed_args(self) -> None:
        d = _make_test_dumper("/tmp")

        with pytest.raises(ValueError, match="cannot mix"):
            d.ctx(lambda self: dict(a=1), phase="x")
```
**EN:** This test method exercises ctx rejects mixed args and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 ctx rejects mixed args 场景，并验证观测到的行为是否符合预期契约。

### Lines 2516-2520: Run test: ctx rejects empty args
```python
    def test_ctx_rejects_empty_args(self) -> None:
        d = _make_test_dumper("/tmp")

        with pytest.raises(ValueError, match="must provide"):
            d.ctx()
```
**EN:** This test method exercises ctx rejects empty args and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 ctx rejects empty args 场景，并验证观测到的行为是否符合预期契约。

### Lines 2523-2523: Define class TestRecomputeStatus
```python
class TestRecomputeStatus:
```
**EN:** This declaration introduces the `TestRecomputeStatus` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestRecomputeStatus` 测试类，并说明它通过继承承担的职责。

### Lines 2524-2530: Run test: disabled by default
```python
    def test_disabled_by_default(self, tmp_path: Path) -> None:
        d = _make_test_dumper(tmp_path)
        tensor = torch.randn(3, 3)
        d.dump("test_tensor", tensor)

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["recompute_status=disabled"])
```
**EN:** This test method exercises disabled by default and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 disabled by default 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2532-2539: Run test: recompute status in embedded meta
```python
    def test_recompute_status_in_embedded_meta(self, tmp_path: Path) -> None:
        d = _make_test_dumper(tmp_path)
        tensor = torch.randn(3, 3)
        d.dump("test_tensor", tensor)

        path = _find_dump_file(tmp_path, rank=0, name="test_tensor")
        raw = _load_dump(path)
        assert raw["meta"]["recompute_status"] == "disabled"
```
**EN:** This test method exercises recompute status in embedded meta and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 recompute status in embedded meta 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2541-2559: Run test: recompute status recompute
```python
    def test_recompute_status_recompute(self, tmp_path: Path, monkeypatch) -> None:
        import sglang.srt.debug_utils.dumper as dumper_mod

        monkeypatch.setattr(
            dumper_mod, "_detect_recompute_status", lambda: _RecomputeStatus.RECOMPUTE
        )

        d = _make_test_dumper(tmp_path)
        tensor = torch.randn(3, 3)
        d.dump("test_tensor", tensor)

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["recompute_status=recompute"])

        path = _find_dump_file(tmp_path, rank=0, name="test_tensor")
        raw = _load_dump(path)
        assert raw["meta"]["recompute_status"] == "recompute"
        assert raw["meta"]["recompute_pseudo_rank"] == 1
        assert raw["meta"]["recompute_pseudo_size"] == 2
```
**EN:** This test method exercises recompute status recompute and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 recompute status recompute 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2561-2581: Run test: recompute status original
```python
    def test_recompute_status_original(self, tmp_path: Path, monkeypatch) -> None:
        import sglang.srt.debug_utils.dumper as dumper_mod

        monkeypatch.setattr(
            dumper_mod,
            "_detect_recompute_status",
            lambda: _RecomputeStatus.ORIGINAL,
        )

        d = _make_test_dumper(tmp_path)
        tensor = torch.randn(3, 3)
        d.dump("test_tensor", tensor)

        filenames = _get_filenames(tmp_path)
        _assert_files(filenames, exist=["recompute_status=original"])

        path = _find_dump_file(tmp_path, rank=0, name="test_tensor")
        raw = _load_dump(path)
        assert raw["meta"]["recompute_status"] == "original"
        assert raw["meta"]["recompute_pseudo_rank"] == 0
        assert raw["meta"]["recompute_pseudo_size"] == 2
```
**EN:** This test method exercises recompute status original and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 recompute status original 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2583-2591: Run test: disabled no recompute pseudo fields
```python
    def test_disabled_no_recompute_pseudo_fields(self, tmp_path: Path) -> None:
        d = _make_test_dumper(tmp_path)
        tensor = torch.randn(3, 3)
        d.dump("test_tensor", tensor)

        path = _find_dump_file(tmp_path, rank=0, name="test_tensor")
        raw = _load_dump(path)
        assert "recompute_pseudo_rank" not in raw["meta"]
        assert "recompute_pseudo_size" not in raw["meta"]
```
**EN:** This test method exercises disabled no recompute pseudo fields and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 disabled no recompute pseudo fields 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2593-2603: Run test: grad hook has no recompute status
```python
    def test_grad_hook_has_no_recompute_status(self, tmp_path: Path) -> None:
        d = _make_test_dumper(tmp_path, enable_grad=True)
        x = torch.randn(3, 3, requires_grad=True)
        y = (x * 2).sum()

        d.dump("test_tensor", x)
        y.backward()

        grad_files = [f for f in _get_filenames(tmp_path) if "grad__test_tensor" in f]
        assert len(grad_files) == 1
        assert "recompute_status" not in grad_files[0]
```
**EN:** This test method exercises grad hook has no recompute status and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 grad hook has no recompute status 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2605-2625: Run test: non intrusive hooks have recompute status
```python
    def test_non_intrusive_hooks_have_recompute_status(self, tmp_path: Path) -> None:
        class Simple(torch.nn.Module):
            def __init__(self):
                super().__init__()
                self.linear = torch.nn.Linear(4, 4)

            def forward(self, x: torch.Tensor) -> torch.Tensor:
                return self.linear(x)

        model = Simple()
        d = _make_test_dumper(tmp_path, non_intrusive_mode="all")
        d.register_non_intrusive_dumper(model)

        with d.capture_output() as captured:
            model(torch.randn(2, 4))

        for key, data in captured.items():
            assert (
                "recompute_status" in data["meta"]
            ), f"missing recompute_status in {key}"
            assert data["meta"]["recompute_status"] == "disabled"
```
**EN:** This test method exercises non intrusive hooks have recompute status and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 non intrusive hooks have recompute status 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2627-2628: Run test: detect recompute status default
```python
    def test_detect_recompute_status_default(self) -> None:
        assert _detect_recompute_status() == _RecomputeStatus.DISABLED
```
**EN:** This test method exercises detect recompute status default and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 detect recompute status default 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2631-2631: Define class TestGrafterConfig
```python
class TestGrafterConfig:
```
**EN:** This declaration introduces the `TestGrafterConfig` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGrafterConfig` 测试类，并说明它通过继承承担的职责。

### Lines 2632-2639: Run test: from env parses filters
```python
    def test_from_env_parses_filters(self):
        with temp_set_env(
            DUMPER_GRAFTER_B2T_FILTER="name == 'x'",
            DUMPER_GRAFTER_T2B_FILTER="name == 'y'",
        ):
            cfg = DumperConfig.from_env()
            assert cfg.grafter_b2t_filter == "name == 'x'"
            assert cfg.grafter_t2b_filter == "name == 'y'"
```
**EN:** This test method exercises from env parses filters and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from env parses filters 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2641-2653: Run test: from env parses int fields
```python
    def test_from_env_parses_int_fields(self):
        with temp_set_env(
            DUMPER_GRAFTER_BASELINE_WORLD_SIZE="8",
            DUMPER_GRAFTER_TARGET_WORLD_SIZE="8",
            DUMPER_GRAFTER_MASTER_PORT="29999",
            DUMPER_GRAFTER_TIMEOUT="120",
        ):
            cfg = DumperConfig.from_env()
            assert cfg.grafter_baseline_world_size == 8
            assert type(cfg.grafter_baseline_world_size) is int
            assert cfg.grafter_target_world_size == 8
            assert cfg.grafter_master_port == 29999
            assert cfg.grafter_timeout == 120
```
**EN:** This test method exercises from env parses int fields and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from env parses int fields 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2655-2657: Run test: from env role
```python
    def test_from_env_role(self):
        with temp_set_env(DUMPER_GRAFTER_ROLE="baseline"):
            assert DumperConfig.from_env().grafter_role == "baseline"
```
**EN:** This test method exercises from env role and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from env role 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2659-2673: Run test: from env enable flag
```python
    def test_from_env_enable_flag(self):
        # enable=True requires all of role, master_address/port, world sizes,
        # and at least one filter per DumperConfig.__post_init__.
        with temp_set_env(
            DUMPER_GRAFTER_ENABLE="1",
            DUMPER_GRAFTER_ROLE="baseline",
            DUMPER_GRAFTER_MASTER_ADDRESS="127.0.0.1",
            DUMPER_GRAFTER_MASTER_PORT="29999",
            DUMPER_GRAFTER_BASELINE_WORLD_SIZE="1",
            DUMPER_GRAFTER_TARGET_WORLD_SIZE="1",
            DUMPER_GRAFTER_B2T_FILTER="name == 'x'",
        ):
            assert DumperConfig.from_env().grafter_enable is True
        with temp_set_env(DUMPER_GRAFTER_ENABLE="false"):
            assert DumperConfig.from_env().grafter_enable is False
```
**EN:** This test method exercises from env enable flag and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 from env enable flag 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2675-2701: Run test: enable without required fields raises
```python
    def test_enable_without_required_fields_raises(self):
        with pytest.raises(AssertionError, match=r"grafter_role"):
            DumperConfig(grafter_enable=True)
        with pytest.raises(AssertionError, match=r"grafter_master_address"):
            DumperConfig(grafter_enable=True, grafter_role="baseline")
        with pytest.raises(AssertionError, match=r"grafter_master_port"):
            DumperConfig(
                grafter_enable=True,
                grafter_role="baseline",
                grafter_master_address="127.0.0.1",
            )
        with pytest.raises(AssertionError, match=r"grafter_baseline_world_size"):
            DumperConfig(
                grafter_enable=True,
                grafter_role="baseline",
                grafter_master_address="127.0.0.1",
                grafter_master_port=12345,
            )
        with pytest.raises(AssertionError, match=r"neither grafter_b2t_filter nor"):
            DumperConfig(
                grafter_enable=True,
                grafter_role="baseline",
                grafter_master_address="127.0.0.1",
                grafter_master_port=12345,
                grafter_baseline_world_size=1,
                grafter_target_world_size=1,
            )
```
**EN:** This test method exercises enable without required fields raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 enable without required fields raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 2703-2706: Run test: env name for grafter field
```python
    def test_env_name_for_grafter_field(self):
        assert (
            DumperConfig._env_name("grafter_b2t_filter") == "DUMPER_GRAFTER_B2T_FILTER"
        )
```
**EN:** This test method exercises env name for grafter field and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 env name for grafter field 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2709-2727: Define helper: unit grafter config
```python
def _unit_grafter_config(**overrides) -> DumperConfig:
    """Build a fully-valid DumperConfig for unit-test use.

    All grafter_* required fields default to dummy values; overrides patch
    individual fields (e.g., grafter_enable=False or filter strings).
    Dummy values are never reached because these unit tests short-circuit
    before _ensure_group runs.
    """
    base = dict(
        grafter_enable=True,
        grafter_role="baseline",
        grafter_master_address="127.0.0.1",
        grafter_master_port=12345,
        grafter_baseline_world_size=1,
        grafter_target_world_size=1,
        grafter_b2t_filter="name == 'x'",
    )
    base.update(overrides)
    return DumperConfig(**base)
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 2730-2730: Define class TestLog
```python
class TestLog:
```
**EN:** This declaration introduces the `TestLog` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestLog` 测试类，并说明它通过继承承担的职责。

### Lines 2731-2737: Run test: log format
```python
    def test_log_format(self):
        with _capture_stdout() as captured:
            _log("hello")
        out = captured.getvalue()
        assert "hello" in out, out
        assert "[Dumper, rank=" in out, out
        assert ", t=" in out, out
```
**EN:** This test method exercises log format and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 log format 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2740-2740: Define class TestCompareTensorsQuick
```python
class TestCompareTensorsQuick:
```
**EN:** This declaration introduces the `TestCompareTensorsQuick` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestCompareTensorsQuick` 测试类，并说明它通过继承承担的职责。

### Lines 2741-2745: Run test: identical
```python
    def test_identical(self):
        a = torch.tensor([1.0, 2.0, 3.0])
        s = _compare_tensors_quick(a, a.clone())
        assert "rel_diff=0" in s, s
        assert "max_abs=0" in s, s
```
**EN:** This test method exercises identical and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 identical 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2747-2753: Run test: diverged
```python
    def test_diverged(self):
        a = torch.tensor([1.0, 2.0, 3.0])
        b = torch.tensor([1.0, 2.0, 4.0])  # last element differs by 1
        s = _compare_tensors_quick(a, b)
        # rel_diff > 0 implies divergence; max_abs should equal 1.0
        assert "max_abs=1" in s, s
        assert "rel_diff=" in s, s
```
**EN:** This test method exercises diverged and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 diverged 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2755-2757: Run test: shape mismatch
```python
    def test_shape_mismatch(self):
        s = _compare_tensors_quick(torch.zeros(3), torch.zeros(4))
        assert "shape mismatch" in s, s
```
**EN:** This test method exercises shape mismatch and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 shape mismatch 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2759-2766: Run test: dtype unified
```python
    def test_dtype_unified(self):
        # Different dtypes should NOT error — both are cast to fp32 internally.
        s = _compare_tensors_quick(
            torch.zeros(3, dtype=torch.float32),
            torch.zeros(3, dtype=torch.float64),
        )
        assert "rel_diff=" in s, s
        assert "max_abs=" in s, s
```
**EN:** This test method exercises dtype unified and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 dtype unified 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2768-2770: Run test: empty
```python
    def test_empty(self):
        s = _compare_tensors_quick(torch.zeros(0), torch.zeros(0))
        assert s == "empty"
```
**EN:** This test method exercises empty and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 empty 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2773-2773: Define class TestGrafterFilterMatching
```python
class TestGrafterFilterMatching:
```
**EN:** This declaration introduces the `TestGrafterFilterMatching` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGrafterFilterMatching` 测试类，并说明它通过继承承担的职责。

### Lines 2774-2778: Document the class `TestGrafterFilterMatching`
```python
    """Unit tests for the filter-matching short-circuit logic.

    These don't initialize a process group, so the network-related fields
    are dummy values via _unit_grafter_config.
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestGrafterFilterMatching`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestGrafterFilterMatching`的设计意图。

### Lines 2780-2783: Run test: disabled returns silently
```python
    def test_disabled_returns_silently(self):
        grafter = _Grafter(config=_unit_grafter_config(grafter_enable=False))
        grafter.maybe_intercept(value=torch.zeros(2), tags={"name": "x"})
        assert grafter._pg is None  # never initialized
```
**EN:** This test method exercises disabled returns silently and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 disabled returns silently 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2785-2791: Run test: unmatched non tensor silent
```python
    def test_unmatched_non_tensor_silent(self):
        """Non-tensor + unmatched name → silent skip, no print."""
        grafter = _Grafter(config=_unit_grafter_config())
        with _capture_stdout() as captured:
            grafter.maybe_intercept(value=42, tags={"name": "other"})
        assert grafter._pg is None
        assert "[Grafter]" not in captured.getvalue(), captured.getvalue()
```
**EN:** This test method exercises unmatched non tensor silent and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 unmatched non tensor silent 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2793-2804: Run test: matched non tensor prints and skips
```python
    def test_matched_non_tensor_prints_and_skips(self):
        """Non-tensor that matches a filter → print explanation, then skip.

        This catches misconfigured filters (e.g. matching a name that maps to a
        dict/list at some call sites) without silently masking the issue."""
        grafter = _Grafter(config=_unit_grafter_config())
        with _capture_stdout() as captured:
            grafter.maybe_intercept(value={"not": "a tensor"}, tags={"name": "x"})
        output = captured.getvalue()
        assert grafter._pg is None  # still no PG init
        assert "value is not a torch.Tensor" in output, output
        assert "type=dict" in output, output
```
**EN:** This test method exercises matched non tensor prints and skips and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 matched non tensor prints and skips 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2806-2811: Run test: unmatched name returns silently
```python
    def test_unmatched_name_returns_silently(self):
        grafter = _Grafter(
            config=_unit_grafter_config(grafter_t2b_filter="name == 'y'")
        )
        grafter.maybe_intercept(value=torch.zeros(2), tags={"name": "z"})
        assert grafter._pg is None
```
**EN:** This test method exercises unmatched name returns silently and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 unmatched name returns silently 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2813-2824: Run test: overlap filters raise
```python
    def test_overlap_filters_raise(self):
        grafter = _Grafter(
            config=_unit_grafter_config(
                grafter_b2t_filter="name == 'x'",
                grafter_t2b_filter="name == 'x'",
            )
        )
        with pytest.raises(
            RuntimeError,
            match=r"matched BOTH grafter_b2t_filter and grafter_t2b_filter",
        ):
            grafter.maybe_intercept(value=torch.zeros(2), tags={"name": "x"})
```
**EN:** This test method exercises overlap filters raise and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 overlap filters raise 场景，并验证观测到的行为是否符合预期契约。

### Lines 2826-2842: Run test: filter expression uses extra tags
```python
    def test_filter_expression_uses_extra_tags(self):
        """Filter expressions can reference any tag key, not just 'name'."""
        grafter = _Grafter(
            config=_unit_grafter_config(
                grafter_b2t_filter="name == 'x' and layer_id < 3",
                grafter_t2b_filter="name == 'x' and layer_id < 3",
            )
        )
        # layer_id=1 → both filters match → overlap raise (proves filter saw layer_id).
        with pytest.raises(RuntimeError, match=r"matched BOTH"):
            grafter.maybe_intercept(
                value=torch.zeros(2),
                tags={"name": "x", "layer_id": 1},
            )
        # layer_id=5 → neither filter matches → silent skip.
        grafter.maybe_intercept(value=torch.zeros(2), tags={"name": "x", "layer_id": 5})
        assert grafter._pg is None
```
**EN:** This test method exercises filter expression uses extra tags and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter expression uses extra tags 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2844-2846: Run test: load function bad module
```python
    def test_load_function_bad_module(self):
        with pytest.raises(ModuleNotFoundError):
            _load_function("no_such_pkg.no_such_module.transform")
```
**EN:** This test method exercises load function bad module and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 load function bad module 场景，并验证观测到的行为是否符合预期契约。

### Lines 2848-2851: Run test: load function missing attr
```python
    def test_load_function_missing_attr(self):
        # `os.path` exists but has no `definitely_no_such_attr`.
        with pytest.raises(AttributeError):
            _load_function("os.path.definitely_no_such_attr")
```
**EN:** This test method exercises load function missing attr and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 load function missing attr 场景，并验证观测到的行为是否符合预期契约。

### Lines 2853-2855: Run test: load function no dotted prefix
```python
    def test_load_function_no_dotted_prefix(self):
        with pytest.raises(ValueError, match=r"missing dotted prefix"):
            _load_function("only_one_segment")
```
**EN:** This test method exercises load function no dotted prefix and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 load function no dotted prefix 场景，并验证观测到的行为是否符合预期契约。

### Lines 2857-2864: Run test: load function non callable resolves but call fails
```python
    def test_load_function_non_callable_resolves_but_call_fails(self):
        """`_load_function` itself only does attribute lookup — it doesn't
        verify the result is callable. A non-callable target manifests at
        call time as TypeError; we still want the failure to be debuggable."""
        sep = _load_function("os.path.sep")  # str, not a callable
        assert isinstance(sep, str)
        with pytest.raises(TypeError):
            sep()
```
**EN:** This test method exercises load function non callable resolves but call fails and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 load function non callable resolves but call fails 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2866-2877: Run test: filter expression only uses non name tag
```python
    def test_filter_expression_only_uses_non_name_tag(self):
        """A filter that doesn't reference `name` at all is still valid; it
        should match purely on the other tag(s)."""
        grafter = _Grafter(
            config=_unit_grafter_config(
                grafter_b2t_filter=None,
                grafter_t2b_filter="layer_id < 3",
            )
        )
        # layer_id absent → resolves to None; `None < 3` raises TypeError in py3.
        with pytest.raises(TypeError):
            grafter.maybe_intercept(value=torch.zeros(2), tags={"name": "x"})
```
**EN:** This test method exercises filter expression only uses non name tag and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 filter expression only uses non name tag 场景，并验证观测到的行为是否符合预期契约。

### Lines 2879-2892: Run test: filter expression unknown tag resolves to none
```python
    def test_filter_expression_unknown_tag_resolves_to_none(self):
        """Unknown tag keys resolve to None inside filter expressions, so
        `layer_id is None` works as an "absent" probe without raising."""
        grafter = _Grafter(
            config=_unit_grafter_config(
                grafter_b2t_filter=None,
                grafter_t2b_filter="layer_id is None and name == 'x'",
            )
        )
        # No `layer_id` in tags → resolves to None → filter matches → tries
        # to init the recv group (which we can't actually do here without a
        # real PG, so we expect the assertion failure from _ensure_group).
        with pytest.raises(AssertionError, match="default torch.distributed"):
            grafter.maybe_intercept(value=torch.zeros(2), tags={"name": "x"})
```
**EN:** This test method exercises filter expression unknown tag resolves to none and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter expression unknown tag resolves to none 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2894-2901: Run test: filter expression syntax error raises
```python
    def test_filter_expression_syntax_error_raises(self):
        """A filter string that isn't valid Python should surface as a
        SyntaxError so the misconfiguration is loud, not silent."""
        grafter = _Grafter(
            config=_unit_grafter_config(grafter_b2t_filter="name == "),
        )
        with pytest.raises(SyntaxError):
            grafter.maybe_intercept(value=torch.zeros(2), tags={"name": "x"})
```
**EN:** This test method exercises filter expression syntax error raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 filter expression syntax error raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 2903-2916: Run test: filter expression undefined helper raises
```python
    def test_filter_expression_undefined_helper_raises(self):
        """Referencing an undefined helper inside a filter (e.g. a function
        the user expected to be in scope) should NOT be silently treated as
        False. The filter namespace is a `_DefaultNoneDict` (unknown keys
        resolve to None), so calling an undefined helper raises TypeError
        (`'NoneType' object is not callable`) — loud enough to surface the
        misconfiguration."""
        grafter = _Grafter(
            config=_unit_grafter_config(
                grafter_b2t_filter="totally_undefined_helper(name)"
            ),
        )
        with pytest.raises(TypeError, match=r"NoneType.* not callable"):
            grafter.maybe_intercept(value=torch.zeros(2), tags={"name": "x"})
```
**EN:** This test method exercises filter expression undefined helper raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 filter expression undefined helper raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 2918-2932: Run test: filter can use re search
```python
    def test_filter_can_use_re_search(self):
        """`re.search` is exposed inside filter expressions as `search()`."""
        grafter = _Grafter(
            config=_unit_grafter_config(
                grafter_b2t_filter="search(r'attn.*', name) is not None",
                grafter_t2b_filter=None,
            )
        )
        # name='attn_input' matches /attn.*/ → tries to init group (hits
        # the no-default-PG assertion, proving the regex matched).
        with pytest.raises(AssertionError, match="default torch.distributed"):
            grafter.maybe_intercept(value=torch.zeros(2), tags={"name": "attn_input"})
        # name='other' does not match → silent skip.
        grafter.maybe_intercept(value=torch.zeros(2), tags={"name": "other"})
        assert grafter._pg is None
```
**EN:** This test method exercises filter can use re search and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 filter can use re search 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 2935-2967: Define helper: run graft test
```python
def _run_graft_test(worker_func, **kwargs):
    """Spawn one GPU-using process per role (rank 0 = baseline, rank 1 = target).

    Limited to 1+1 because CI machines we can rely on have only 2 GPUs.
    Each process initializes its OWN default PG (nccl, world_size=1) from
    the start, mirroring production where baseline and target are
    independently launched.

    For asymmetric / multi-rank coverage that doesn't need GPU, see
    `_run_graft_test_cpu_multi` below.
    """
    import torch.multiprocessing as mp

    role_ports = [find_available_port(29700 + i * 100) for i in range(2)]

    ctx = mp.get_context("spawn")
    result_queue = ctx.Queue()
    processes = []
    for rank in range(2):
        p = ctx.Process(
            target=_graft_worker_entry,
            args=(rank, role_ports[rank], worker_func, result_queue, kwargs),
        )
        p.start()
        processes.append(p)

    for p in processes:
        p.join()

    errors = [result_queue.get() for _ in range(2)]
    errors = [e for e in errors if e]
    if errors:
        raise AssertionError("\n".join(errors))
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 2970-2986: Define helper: graft worker entry
```python
def _graft_worker_entry(rank, role_port, worker_func, result_queue, kwargs):
    import traceback

    torch.cuda.set_device(rank)
    dist.init_process_group(
        backend="nccl",
        init_method=f"tcp://127.0.0.1:{role_port}",
        world_size=1,
        rank=0,
    )
    try:
        worker_func(rank=rank, **kwargs)
        result_queue.put(None)
    except Exception as e:
        result_queue.put(f"rank={rank}: {e}\n{traceback.format_exc()}")
    finally:
        dist.destroy_process_group()
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 2989-3035: Define helper: run graft test split
```python
def _run_graft_test_split(worker_baseline, worker_target, **kwargs) -> dict:
    """Like `_run_graft_test`, but each role runs its OWN dedicated worker
    function (no `if rank == 0:` branching) and stdout is captured per role.

    Returns ``{"baseline": stdout_str, "target": stdout_str}`` so tests can
    snapshot/assert on the per-role logs. Used by the E2E example for
    educational clarity (each role's logic reads top-to-bottom) and to assert
    the user-visible log output matches expectations.
    """
    import torch.multiprocessing as mp

    role_ports = {
        "baseline": find_available_port(29700),
        "target": find_available_port(29800),
    }

    ctx = mp.get_context("spawn")
    result_queue = ctx.Queue()
    processes = []
    for global_rank, (role, worker) in enumerate(
        [("baseline", worker_baseline), ("target", worker_target)]
    ):
        p = ctx.Process(
            target=_graft_split_worker_entry,
            args=(global_rank, role, role_ports[role], worker, result_queue, kwargs),
        )
        p.start()
        processes.append(p)

    for p in processes:
        p.join()

    outputs: dict = {}
    errors: list = []
    for _ in range(2):
        role, error, captured = result_queue.get()
        outputs[role] = captured
        if error:
            errors.append(f"role={role}: {error}")
    if errors:
        raise AssertionError(
            "\n".join(errors)
            + "\nCaptured outputs:\n"
            + f"--- baseline ---\n{outputs.get('baseline', '')}\n"
            + f"--- target ---\n{outputs.get('target', '')}"
        )
    return outputs
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3038-3081: Define helper: graft split worker entry
```python
def _graft_split_worker_entry(
    global_rank, role, role_port, worker_func, result_queue, kwargs
):
    import io
    import traceback

    captured = io.StringIO()
    old_stdout = sys.stdout
    sys.stdout = captured
    error = None
    try:
        # Set per-role env BEFORE we (re)build the module-level `dumper`. The
        # parent left DUMPER_GRAFTER_ENABLE/ROLE unset because they vary per
        # child; we set them here, then rebuild the global so that worker
        # code can simply call `from sglang.srt.debug_utils.dumper import dumper`
        # and get a properly-configured Grafter — exactly mirroring how
        # production code uses the global.
        os.environ["DUMPER_GRAFTER_ENABLE"] = "1"
        os.environ["DUMPER_GRAFTER_ROLE"] = role
        import sglang.srt.debug_utils.dumper as _dumper_module

        _dumper_module.dumper = _dumper_module._Dumper(
            config=_dumper_module.DumperConfig.from_env()
        )

        torch.cuda.set_device(global_rank)
        dist.init_process_group(
            backend="nccl",
            init_method=f"tcp://127.0.0.1:{role_port}",
            world_size=1,
            rank=0,
        )
        try:
            worker_func(**kwargs)
        except Exception as e:
            error = f"{e}\n{traceback.format_exc()}"
        finally:
            try:
                dist.destroy_process_group()
            except Exception:
                pass
    finally:
        sys.stdout = old_stdout
    result_queue.put((role, error, captured.getvalue()))
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 3084-3139: Define helper: run graft test cpu multi
```python
def _run_graft_test_cpu_multi(
    worker_func, *, baseline_world: int, target_world: int, **kwargs
):
    """Spawn (baseline_world + target_world) CPU-only processes (gloo backend).

    Used to exercise asymmetric multi-rank cases (e.g. 4 baseline ranks and
    2 target ranks) that we can't run on the 2-GPU CI fleet. Each role gets
    its OWN default PG (gloo, world=role_world); the graft cross-system PG
    spans all ranks.

    The worker function receives (role, local_rank, **kwargs).
    """
    import torch.multiprocessing as mp

    # One default-PG port per role (baseline-side ranks share one PG, target
    # ranks share another). Allocated up-front to avoid child races.
    role_ports = {
        "baseline": find_available_port(29800),
        "target": find_available_port(29900),
    }

    ctx = mp.get_context("spawn")
    result_queue = ctx.Queue()
    processes = []
    total = baseline_world + target_world
    for global_rank in range(total):
        if global_rank < baseline_world:
            role = "baseline"
            local_rank = global_rank
            local_world = baseline_world
        else:
            role = "target"
            local_rank = global_rank - baseline_world
            local_world = target_world
        p = ctx.Process(
            target=_graft_cpu_worker_entry,
            args=(
                role,
                local_rank,
                local_world,
                role_ports[role],
                worker_func,
                result_queue,
                kwargs,
            ),
        )
        p.start()
        processes.append(p)

    for p in processes:
        p.join()

    errors = [result_queue.get() for _ in range(total)]
    errors = [e for e in errors if e]
    if errors:
        raise AssertionError("\n".join(errors))
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 3142-3161: Define helper: graft cpu worker entry
```python
def _graft_cpu_worker_entry(
    role, local_rank, local_world, port, worker_func, result_queue, kwargs
):
    import traceback

    dist.init_process_group(
        backend="gloo",
        init_method=f"tcp://127.0.0.1:{port}",
        world_size=local_world,
        rank=local_rank,
    )
    try:
        worker_func(role=role, local_rank=local_rank, **kwargs)
        result_queue.put(None)
    except Exception as e:
        result_queue.put(
            f"role={role} local_rank={local_rank}: {e}\n{traceback.format_exc()}"
        )
    finally:
        dist.destroy_process_group()
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 3164-3195: Define helper: make grafter test config
```python
def _make_grafter_test_config(
    *,
    rank: int,
    graft_port: int,
    group_name: str,
    timeout: int = 30,
    transform_path: Optional[str] = None,
    b2t_filter: Optional[str] = "name == 'x'",
    t2b_filter: Optional[str] = None,
) -> DumperConfig:
    """Helper for distributed grafter tests.

    Same b2t/t2b filters on both sides; only `grafter_role` differs (rank 0 =
    baseline, rank 1 = target). Both sides are world_size=1 within their own
    role's default PG.
    """
    role = "baseline" if rank == 0 else "target"
    return DumperConfig(
        grafter_enable=True,
        grafter_role=role,
        grafter_b2t_filter=b2t_filter,
        grafter_t2b_filter=t2b_filter,
        grafter_master_address="127.0.0.1",
        grafter_master_port=graft_port,
        grafter_baseline_world_size=1,
        grafter_target_world_size=1,
        grafter_group_name=group_name,
        grafter_timeout=timeout,
        # Loading the user transform on the recv side; for b2t the recv is
        # the target side (rank 1).
        grafter_transform_path=transform_path if rank == 1 else None,
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 3198-3198: Define class TestGrafterDistributed
```python
class TestGrafterDistributed:
```
**EN:** This declaration introduces the `TestGrafterDistributed` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGrafterDistributed` 测试类，并说明它通过继承承担的职责。

### Lines 3199-3204: Run test: b2t copy roundtrip
```python
    def test_b2t_copy_roundtrip(self):
        """Baseline (rank 0) sends 'x' to target (rank 1), target.copy_'s it."""
        graft_port = find_available_port(29600)
        _run_graft_test(
            self._test_b2t_func, graft_port=graft_port, group_name="grafter_b2t"
        )
```
**EN:** This test method exercises b2t copy roundtrip and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 b2t copy roundtrip 场景，并验证观测到的行为是否符合预期契约。

### Lines 3206-3226: Define helper: test b2t func
```python
    @staticmethod
    def _test_b2t_func(rank, graft_port, group_name):
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank, graft_port=graft_port, group_name=group_name
            )
        )
        try:
            if rank == 0:
                tensor = torch.tensor([1.0, 2.0, 3.0], device="cuda:0")
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                target = torch.zeros(3, device="cuda:1")
                with _capture_stdout() as captured:
                    grafter.maybe_intercept(value=target, tags={"name": "x"})
                assert target.tolist() == [1.0, 2.0, 3.0], f"got {target.tolist()}"
                # Success log must include the pre/new diff summary.
                assert "diff_pre_vs_new=" in captured.getvalue(), captured.getvalue()
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3228-3233: Run test: t2b copy roundtrip
```python
    def test_t2b_copy_roundtrip(self):
        """Target (rank 1) sends 'x' to baseline (rank 0), baseline.copy_'s it."""
        graft_port = find_available_port(29605)
        _run_graft_test(
            self._test_t2b_func, graft_port=graft_port, group_name="grafter_t2b"
        )
```
**EN:** This test method exercises t2b copy roundtrip and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 t2b copy roundtrip 场景，并验证观测到的行为是否符合预期契约。

### Lines 3235-3256: Define helper: test t2b func
```python
    @staticmethod
    def _test_t2b_func(rank, graft_port, group_name):
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank,
                graft_port=graft_port,
                group_name=group_name,
                b2t_filter=None,
                t2b_filter="name == 'x'",
            )
        )
        try:
            if rank == 1:
                tensor = torch.tensor([4.0, 5.0, 6.0], device="cuda:1")
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                target = torch.zeros(3, device="cuda:0")
                grafter.maybe_intercept(value=target, tags={"name": "x"})
                assert target.tolist() == [4.0, 5.0, 6.0], f"got {target.tolist()}"
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3258-3273: Run test: recv with user transform
```python
    def test_recv_with_user_transform(self, tmp_path: Path):
        # Write a tiny module that defines `transform(graft_input)`. The
        # worker prepends tmp_path to sys.path so import_module sees it.
        module_name = "_xform_user_basic"
        (tmp_path / f"{module_name}.py").write_text(
            "def transform(graft_input):\n"
            "    return graft_input.received_list[0] * 2\n"
        )
        graft_port = find_available_port(29610)
        _run_graft_test(
            self._test_user_transform_func,
            graft_port=graft_port,
            group_name="grafter_transform",
            transform_dir=str(tmp_path),
            transform_path=f"{module_name}.transform",
        )
```
**EN:** This test method exercises recv with user transform and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 recv with user transform 场景，并验证观测到的行为是否符合预期契约。

### Lines 3275-3298: Define helper: test user transform func
```python
    @staticmethod
    def _test_user_transform_func(
        rank, graft_port, group_name, transform_dir, transform_path
    ):
        sys.path.insert(0, transform_dir)
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank,
                graft_port=graft_port,
                group_name=group_name,
                transform_path=transform_path,
            )
        )
        try:
            if rank == 0:
                tensor = torch.tensor([1.0, 2.0, 3.0], device="cuda:0")
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                target = torch.zeros(3, device="cuda:1")
                grafter.maybe_intercept(value=target, tags={"name": "x"})
                assert target.tolist() == [2.0, 4.0, 6.0], f"got {target.tolist()}"
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3300-3306: Run test: unmatched name skipped
```python
    def test_unmatched_name_skipped(self):
        graft_port = find_available_port(29620)
        _run_graft_test(
            self._test_unmatched_func,
            graft_port=graft_port,
            group_name="grafter_unmatched",
        )
```
**EN:** This test method exercises unmatched name skipped and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 unmatched name skipped 场景，并验证观测到的行为是否符合预期契约。

### Lines 3308-3322: Define helper: test unmatched func
```python
    @staticmethod
    def _test_unmatched_func(rank, graft_port, group_name):
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank, graft_port=graft_port, group_name=group_name
            )
        )
        try:
            target = torch.tensor([7.0, 7.0, 7.0], device=f"cuda:{rank}")
            grafter.maybe_intercept(value=target, tags={"name": "other"})
            assert target.tolist() == [7.0, 7.0, 7.0], "tensor must not be modified"
            assert grafter._pg is None, "group must not init for unmatched name"
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3324-3332: Run test: default fallback shape mismatch does not crash
```python
    def test_default_fallback_shape_mismatch_does_not_crash(self):
        """When sender shape != target shape, default identity fallback raises;
        the grafter must catch it, log, and leave target unchanged."""
        graft_port = find_available_port(29615)
        _run_graft_test(
            self._test_shape_mismatch_func,
            graft_port=graft_port,
            group_name="grafter_shape_mismatch",
        )
```
**EN:** This test method exercises default fallback shape mismatch does not crash and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 default fallback shape mismatch does not crash 场景，并验证观测到的行为是否符合预期契约。

### Lines 3334-3359: Define helper: test shape mismatch func
```python
    @staticmethod
    def _test_shape_mismatch_func(rank, graft_port, group_name):
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank, graft_port=graft_port, group_name=group_name
            )
        )
        try:
            if rank == 0:
                # Baseline sends shape=(3,)
                tensor = torch.tensor([1.0, 2.0, 3.0], device="cuda:0")
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                # Target's local target has shape=(4,) — mismatch with sender.
                target = torch.tensor([7.0, 7.0, 7.0, 7.0], device="cuda:1")
                # No exception should propagate; tensor must stay unchanged.
                grafter.maybe_intercept(value=target, tags={"name": "x"})
                assert target.tolist() == [
                    7.0,
                    7.0,
                    7.0,
                    7.0,
                ], f"target should be unchanged after shape-mismatch graft, got {target.tolist()}"
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3361-3377: Run test: user transform exception does not crash
```python
    def test_user_transform_exception_does_not_crash(self, tmp_path: Path):
        """A user transform that raises must NOT bring down the system; the
        grafter logs and skips the copy_, leaving target unchanged."""
        module_name = "_xform_throws"
        (tmp_path / f"{module_name}.py").write_text(
            "def transform(graft_input):\n"
            "    raise RuntimeError('intentional test error from user transform')\n"
        )
        graft_port = find_available_port(29635)
        _run_graft_test(
            self._test_transform_throws_func,
            graft_port=graft_port,
            group_name="grafter_throws",
            transform_dir=str(tmp_path),
            transform_path=f"{module_name}.transform",
            module_name=module_name,
        )
```
**EN:** This test method exercises user transform exception does not crash and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 user transform exception does not crash 场景，并验证观测到的行为是否符合预期契约。

### Lines 3379-3413: Define helper: test transform throws func
```python
    @staticmethod
    def _test_transform_throws_func(
        rank, graft_port, group_name, transform_dir, transform_path, module_name
    ):
        sys.path.insert(0, transform_dir)
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank,
                graft_port=graft_port,
                group_name=group_name,
                transform_path=transform_path,
            )
        )
        try:
            if rank == 0:
                tensor = torch.tensor([1.0, 2.0, 3.0], device="cuda:0")
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                target = torch.tensor([9.0, 9.0, 9.0], device="cuda:1")
                with _capture_stdout() as captured:
                    grafter.maybe_intercept(value=target, tags={"name": "x"})
                assert target.tolist() == [
                    9.0,
                    9.0,
                    9.0,
                ], f"target must be unchanged when transform throws, got {target.tolist()}"
                output = captured.getvalue()
                assert "transform/copy_ raised RuntimeError" in output, output
                assert "intentional test error" in output, output
                # Full traceback must be included so the bug is debuggable.
                assert "Traceback (most recent call last)" in output, output
                assert f"{module_name}.py" in output, output
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3415-3432: Run test: extras flow to recv transform
```python
    def test_extras_flow_to_recv_transform(self, tmp_path: Path):
        """Sender attaches per-call grafter_extras; recv transform reads them
        and uses them to compute the override value."""
        module_name = "_xform_uses_extras"
        (tmp_path / f"{module_name}.py").write_text(
            "import torch\n"
            "def transform(graft_input):\n"
            "    fill = graft_input.received_extras_list[0]['fill_value']\n"
            "    return torch.full_like(graft_input.target, fill)\n"
        )
        graft_port = find_available_port(29645)
        _run_graft_test(
            self._test_extras_func,
            graft_port=graft_port,
            group_name="grafter_extras",
            transform_dir=str(tmp_path),
            transform_path=f"{module_name}.transform",
        )
```
**EN:** This test method exercises extras flow to recv transform and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 extras flow to recv transform 场景，并验证观测到的行为是否符合预期契约。

### Lines 3434-3464: Define helper: test extras func
```python
    @staticmethod
    def _test_extras_func(rank, graft_port, group_name, transform_dir, transform_path):
        sys.path.insert(0, transform_dir)
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank,
                graft_port=graft_port,
                group_name=group_name,
                transform_path=transform_path,
            )
        )
        try:
            if rank == 0:
                # Baseline (sender) attaches an extras dict.
                tensor = torch.tensor([1.0, 2.0, 3.0], device="cuda:0")
                grafter.maybe_intercept(
                    value=tensor,
                    tags={"name": "x"},
                    extras={"fill_value": 42.0},
                )
            else:
                target = torch.zeros(3, device="cuda:1")
                grafter.maybe_intercept(value=target, tags={"name": "x"})
                assert target.tolist() == [
                    42.0,
                    42.0,
                    42.0,
                ], f"target should be filled from sender extras, got {target.tolist()}"
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3466-3472: Run test: init timeout warns
```python
    def test_init_timeout_warns(self):
        graft_port = find_available_port(29630)
        _run_graft_test(
            self._test_init_timeout_func,
            graft_port=graft_port,
            group_name="grafter_timeout",
        )
```
**EN:** This test method exercises init timeout warns and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 init timeout warns 场景，并验证观测到的行为是否符合预期契约。

### Lines 3474-3499: Define helper: test init timeout func
```python
    @staticmethod
    def _test_init_timeout_func(rank, graft_port, group_name):
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank, graft_port=graft_port, group_name=group_name, timeout=2
            )
        )
        try:
            with _capture_stdout() as captured:
                if rank == 1:
                    time.sleep(4)
                tensor = torch.tensor([1.0, 2.0, 3.0], device=f"cuda:{rank}")
                if rank == 0:
                    grafter.maybe_intercept(value=tensor, tags={"name": "x"})
                else:
                    target = torch.zeros(3, device=f"cuda:{rank}")
                    grafter.maybe_intercept(value=target, tags={"name": "x"})
            output = captured.getvalue()
            if rank == 0:
                assert (
                    "WARNING" in output
                ), f"expected WARNING in rank 0 output: {output}"
                assert "has not completed after 2s" in output, output
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions and waits for asynchronous state transitions before rechecking results.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期，并在再次检查结果前等待异步状态转换完成。

### Lines 3501-3509: Run test: extras default none flow
```python
    def test_extras_default_none_flow(self):
        """When the sender omits `grafter_extras`, the recv transform sees a
        list of Nones — but len(received_extras_list) still matches n_senders."""
        graft_port = find_available_port(29650)
        _run_graft_test(
            self._test_extras_none_func,
            graft_port=graft_port,
            group_name="grafter_extras_none",
        )
```
**EN:** This test method exercises extras default none flow and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 extras default none flow 场景，并验证观测到的行为是否符合预期契约。

### Lines 3511-3534: Define helper: test extras none func
```python
    @staticmethod
    def _test_extras_none_func(rank, graft_port, group_name):
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank, graft_port=graft_port, group_name=group_name
            )
        )
        try:
            if rank == 0:
                tensor = torch.tensor([1.0, 2.0, 3.0], device="cuda:0")
                # Note: extras kwarg omitted entirely → None on the wire.
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                target = torch.zeros(3, device="cuda:1")
                with _capture_stdout() as captured:
                    grafter.maybe_intercept(value=target, tags={"name": "x"})
                # Default identity transform copies tensor through; recv log
                # must reflect that received_extras_list == [None].
                output = captured.getvalue()
                assert "sender_extras=[None]" in output, output
                assert target.tolist() == [1.0, 2.0, 3.0], target.tolist()
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3536-3545: Run test: group init is cached across calls
```python
    def test_group_init_is_cached_across_calls(self):
        """The graft process group is initialized lazily on the first
        matched dump() and cached afterwards — subsequent dumps must reuse
        the same `_pg` object, not re-init."""
        graft_port = find_available_port(29660)
        _run_graft_test(
            self._test_group_cache_func,
            graft_port=graft_port,
            group_name="grafter_cache",
        )
```
**EN:** This test method exercises group init is cached across calls and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 group init is cached across calls 场景，并验证观测到的行为是否符合预期契约。

### Lines 3547-3577: Define helper: test group cache func
```python
    @staticmethod
    def _test_group_cache_func(rank, graft_port, group_name):
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank, graft_port=graft_port, group_name=group_name
            )
        )
        try:
            if rank == 0:
                t1 = torch.tensor([1.0, 2.0, 3.0], device="cuda:0")
                t2 = torch.tensor([4.0, 5.0, 6.0], device="cuda:0")
                grafter.maybe_intercept(value=t1, tags={"name": "x"})
                pg_after_first = grafter._pg
                assert pg_after_first is not None
                grafter.maybe_intercept(value=t2, tags={"name": "x"})
                assert (
                    grafter._pg is pg_after_first
                ), "_pg must be cached across calls, not re-initialized"
            else:
                target1 = torch.zeros(3, device="cuda:1")
                target2 = torch.zeros(3, device="cuda:1")
                grafter.maybe_intercept(value=target1, tags={"name": "x"})
                pg_after_first = grafter._pg
                assert pg_after_first is not None
                grafter.maybe_intercept(value=target2, tags={"name": "x"})
                assert grafter._pg is pg_after_first
                assert target1.tolist() == [1.0, 2.0, 3.0]
                assert target2.tolist() == [4.0, 5.0, 6.0]
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3579-3598: Run test: copy failure does not crash
```python
    def test_copy_failure_does_not_crash(self, tmp_path: Path):
        """If the user transform returns a tensor whose shape doesn't match
        target, `value.copy_(value_to_override)` raises — and that error
        must be caught, logged with traceback, and target left unchanged
        (same robustness contract as transform-throws)."""
        module_name = "_xform_returns_wrong_shape"
        (tmp_path / f"{module_name}.py").write_text(
            "import torch\n"
            "def transform(graft_input):\n"
            "    # Deliberately return a shape that copy_ will reject.\n"
            "    return torch.zeros(99, device=graft_input.target.device)\n"
        )
        graft_port = find_available_port(29665)
        _run_graft_test(
            self._test_copy_failure_func,
            graft_port=graft_port,
            group_name="grafter_copy_fail",
            transform_dir=str(tmp_path),
            transform_path=f"{module_name}.transform",
        )
```
**EN:** This test method exercises copy failure does not crash and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 copy failure does not crash 场景，并验证观测到的行为是否符合预期契约。

### Lines 3600-3632: Define helper: test copy failure func
```python
    @staticmethod
    def _test_copy_failure_func(
        rank, graft_port, group_name, transform_dir, transform_path
    ):
        sys.path.insert(0, transform_dir)
        grafter = _Grafter(
            config=_make_grafter_test_config(
                rank=rank,
                graft_port=graft_port,
                group_name=group_name,
                transform_path=transform_path,
            )
        )
        try:
            if rank == 0:
                tensor = torch.tensor([1.0, 2.0, 3.0], device="cuda:0")
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                target = torch.tensor([7.0, 7.0, 7.0], device="cuda:1")
                with _capture_stdout() as captured:
                    grafter.maybe_intercept(value=target, tags={"name": "x"})
                # target must be unchanged; error must be logged with traceback.
                assert target.tolist() == [
                    7.0,
                    7.0,
                    7.0,
                ], f"target must be unchanged on copy_ failure, got {target.tolist()}"
                output = captured.getvalue()
                assert "transform/copy_ raised" in output, output
                assert "Traceback (most recent call last)" in output, output
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterDistributed` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterDistributed` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3635-3635: Define class TestGrafterMultiRankCpu
```python
class TestGrafterMultiRankCpu:
```
**EN:** This declaration introduces the `TestGrafterMultiRankCpu` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGrafterMultiRankCpu` 测试类，并说明它通过继承承担的职责。

### Lines 3636-3637: Document the class `TestGrafterMultiRankCpu`
```python
    """Coverage of asymmetric multi-rank cases via CPU/gloo (CI fleet has
    only 2 GPUs, which is too few for these cases)."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestGrafterMultiRankCpu`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestGrafterMultiRankCpu`的设计意图。

### Lines 3639-3663: Run test: 4 baseline 2 target b2t with user transform
```python
    def test_4_baseline_2_target_b2t_with_user_transform(self, tmp_path: Path):
        """4 baseline senders -> 2 target receivers via b2t graft.
        The user transform asserts received_list has length 4 with each
        sender's tensor matching its rank, then returns a marker tensor."""
        module_name = "_xform_assert_4_senders"
        (tmp_path / f"{module_name}.py").write_text(
            "import torch\n"
            "def transform(graft_input):\n"
            "    rl = graft_input.received_list\n"
            "    assert len(rl) == 4, f'expected 4 senders, got {len(rl)}'\n"
            "    for i, t in enumerate(rl):\n"
            "        v = float(t.flatten()[0].item())\n"
            "        assert v == float(i), f'rl[{i}][0]={v}, want {float(i)}'\n"
            "    return torch.full_like(graft_input.target, 999.0)\n"
        )
        graft_port = find_available_port(29655)
        _run_graft_test_cpu_multi(
            self._test_4b_2t_func,
            baseline_world=4,
            target_world=2,
            graft_port=graft_port,
            group_name="grafter_4b_2t",
            transform_dir=str(tmp_path),
            transform_path=f"{module_name}.transform",
        )
```
**EN:** This test method exercises 4 baseline 2 target b2t with user transform and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 4 baseline 2 target b2t with user transform 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 3665-3692: Define helper: test 4b 2t func
```python
    @staticmethod
    def _test_4b_2t_func(
        role, local_rank, graft_port, group_name, transform_dir, transform_path
    ):
        sys.path.insert(0, transform_dir)
        cfg = _make_multi_rank_config(
            role=role,
            graft_port=graft_port,
            group_name=group_name,
            baseline_world=4,
            target_world=2,
            transform_path=transform_path,
            direction="b2t",
        )
        grafter = _Grafter(config=cfg)
        try:
            if role == "baseline":
                # rank-i baseline contributes [i, i, i].
                tensor = torch.full((3,), float(local_rank))
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                # Target's local tensor (will be overwritten with 999s by transform).
                target = torch.full((3,), 99.0)
                grafter.maybe_intercept(value=target, tags={"name": "x"})
                assert target.tolist() == [999.0, 999.0, 999.0], target.tolist()
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterMultiRankCpu` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterMultiRankCpu` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3694-3720: Run test: 2 target 4 baseline t2b with user transform
```python
    def test_2_target_4_baseline_t2b_with_user_transform(self, tmp_path: Path):
        """Mirror image of the b2t case: 2 target senders -> 4 baseline
        receivers via t2b graft. Confirms the (role, direction) algebra and
        sender_slice work correctly when target is the SENDER side."""
        module_name = "_xform_assert_2_senders_t2b"
        (tmp_path / f"{module_name}.py").write_text(
            "import torch\n"
            "def transform(graft_input):\n"
            "    rl = graft_input.received_list\n"
            "    assert len(rl) == 2, f'expected 2 senders, got {len(rl)}'\n"
            "    for i, t in enumerate(rl):\n"
            "        v = float(t.flatten()[0].item())\n"
            "        assert v == float(i + 100), (\n"
            "            f'rl[{i}][0]={v}, want {float(i + 100)}'\n"
            "        )\n"
            "    return torch.full_like(graft_input.target, 7.0)\n"
        )
        graft_port = find_available_port(29670)
        _run_graft_test_cpu_multi(
            self._test_2t_4b_func,
            baseline_world=4,
            target_world=2,
            graft_port=graft_port,
            group_name="grafter_2t_4b",
            transform_dir=str(tmp_path),
            transform_path=f"{module_name}.transform",
        )
```
**EN:** This test method exercises 2 target 4 baseline t2b with user transform and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 2 target 4 baseline t2b with user transform 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 3722-3748: Define helper: test 2t 4b func
```python
    @staticmethod
    def _test_2t_4b_func(
        role, local_rank, graft_port, group_name, transform_dir, transform_path
    ):
        sys.path.insert(0, transform_dir)
        cfg = _make_multi_rank_config(
            role=role,
            graft_port=graft_port,
            group_name=group_name,
            baseline_world=4,
            target_world=2,
            transform_path=transform_path,
            direction="t2b",
        )
        grafter = _Grafter(config=cfg)
        try:
            if role == "target":
                # rank-i target contributes [i+100, i+100, i+100].
                tensor = torch.full((3,), float(local_rank + 100))
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                target = torch.full((3,), 99.0)
                grafter.maybe_intercept(value=target, tags={"name": "x"})
                assert target.tolist() == [7.0, 7.0, 7.0], target.tolist()
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterMultiRankCpu` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterMultiRankCpu` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3750-3762: Run test: default transform with asymmetric world logs and skips
```python
    def test_default_transform_with_asymmetric_world_logs_and_skips(self):
        """The default identity-by-rank fallback requires #senders == #recvs.
        With baseline=4 and target=2 and no user transform, the recv side
        must catch the RuntimeError, log it with traceback, and leave the
        target unchanged."""
        graft_port = find_available_port(29675)
        _run_graft_test_cpu_multi(
            self._test_default_asym_func,
            baseline_world=4,
            target_world=2,
            graft_port=graft_port,
            group_name="grafter_default_asym",
        )
```
**EN:** This test method exercises default transform with asymmetric world logs and skips and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 default transform with asymmetric world logs and skips 场景，并验证观测到的行为是否符合预期契约。

### Lines 3764-3795: Define helper: test default asym func
```python
    @staticmethod
    def _test_default_asym_func(role, local_rank, graft_port, group_name):
        cfg = _make_multi_rank_config(
            role=role,
            graft_port=graft_port,
            group_name=group_name,
            baseline_world=4,
            target_world=2,
            transform_path=None,  # default identity-by-rank fallback
            direction="b2t",
        )
        grafter = _Grafter(config=cfg)
        try:
            if role == "baseline":
                tensor = torch.full((3,), float(local_rank))
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                target = torch.full((3,), 42.0)
                with _capture_stdout() as captured:
                    grafter.maybe_intercept(value=target, tags={"name": "x"})
                assert target.tolist() == [42.0, 42.0, 42.0], (
                    f"target must be unchanged when default transform raises, "
                    f"got {target.tolist()}"
                )
                output = captured.getvalue()
                assert "transform/copy_ raised RuntimeError" in output, output
                # The error message must explain WHY the default fell through.
                assert "#senders=4" in output and "#recvs=2" in output, output
                assert "Traceback (most recent call last)" in output, output
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterMultiRankCpu` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterMultiRankCpu` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3797-3826: Run test: mixed shape senders via user transform
```python
    def test_mixed_shape_senders_via_user_transform(self, tmp_path: Path):
        """`all_gather_object` is pickle-routed, so sender ranks may
        contribute tensors with DIFFERENT shapes. The user transform sees
        the full list and is responsible for picking/reducing. Asserts that
        rank-i baseline's tensor has shape (i+1,) and the transform
        concatenates them on the recv side."""
        module_name = "_xform_concat_mixed_shape"
        (tmp_path / f"{module_name}.py").write_text(
            "import torch\n"
            "def transform(graft_input):\n"
            "    rl = graft_input.received_list\n"
            "    # Each baseline sent shape=(rank+1,) tensors filled with rank.\n"
            "    expected_shapes = [(i + 1,) for i in range(len(rl))]\n"
            "    actual_shapes = [tuple(t.shape) for t in rl]\n"
            "    assert actual_shapes == expected_shapes, (\n"
            "        f'shape mismatch: expected {expected_shapes}, got {actual_shapes}'\n"
            "    )\n"
            "    # Concat to length 1+2+3+4 = 10 == target's length.\n"
            "    return torch.cat(rl)\n"
        )
        graft_port = find_available_port(29680)
        _run_graft_test_cpu_multi(
            self._test_mixed_shape_func,
            baseline_world=4,
            target_world=2,
            graft_port=graft_port,
            group_name="grafter_mixed_shape",
            transform_dir=str(tmp_path),
            transform_path=f"{module_name}.transform",
        )
```
**EN:** This test method exercises mixed shape senders via user transform and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 mixed shape senders via user transform 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 3828-3856: Define helper: test mixed shape func
```python
    @staticmethod
    def _test_mixed_shape_func(
        role, local_rank, graft_port, group_name, transform_dir, transform_path
    ):
        sys.path.insert(0, transform_dir)
        cfg = _make_multi_rank_config(
            role=role,
            graft_port=graft_port,
            group_name=group_name,
            baseline_world=4,
            target_world=2,
            transform_path=transform_path,
            direction="b2t",
        )
        grafter = _Grafter(config=cfg)
        try:
            if role == "baseline":
                # rank-i baseline contributes shape=(i+1,) filled with i.
                tensor = torch.full((local_rank + 1,), float(local_rank))
                grafter.maybe_intercept(value=tensor, tags={"name": "x"})
            else:
                # 1 + 2 + 3 + 4 = 10 elements after concat.
                target = torch.zeros(10)
                grafter.maybe_intercept(value=target, tags={"name": "x"})
                expected = [0.0] + [1.0] * 2 + [2.0] * 3 + [3.0] * 4
                assert target.tolist() == expected, target.tolist()
        finally:
            if grafter._pg is not None:
                dist.destroy_process_group(grafter._pg)
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterMultiRankCpu` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterMultiRankCpu` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3859-3882: Define helper: make multi rank config
```python
def _make_multi_rank_config(
    *,
    role: str,
    graft_port: int,
    group_name: str,
    baseline_world: int,
    target_world: int,
    transform_path: Optional[str],
    direction: str,
) -> DumperConfig:
    return DumperConfig(
        grafter_enable=True,
        grafter_role=role,
        grafter_b2t_filter="name == 'x'" if direction == "b2t" else None,
        grafter_t2b_filter="name == 'x'" if direction == "t2b" else None,
        grafter_master_address="127.0.0.1",
        grafter_master_port=graft_port,
        grafter_baseline_world_size=baseline_world,
        grafter_target_world_size=target_world,
        grafter_backend="gloo",
        grafter_group_name=group_name,
        grafter_timeout=30,
        grafter_transform_path=transform_path,
    )
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。

### Lines 3885-3903: Define helper: e2e transform
```python
def _e2e_transform(graft_input):
    """User transform used by the E2E example test. Demonstrates the two
    customization hooks reviewers should learn from:

      1. The transform receives a `GraftTransformInput` and returns the
         tensor that the recv side will `.copy_()` into its local target.
      2. `graft_input.received_extras_list` carries whatever the sender
         passed via `grafter_extras={...}` — useful for any per-call
         metadata the recv side needs (layer ids, calibration knobs, ...).

    Here we keep the example minimal: the sender attaches a single dummy
    key/value so the recv side has something concrete to assert on, then
    the transform is just identity. Real workflows would compute a
    non-trivial override (scale, reshape, decode, ...) using the extras.
    """
    assert (
        graft_input.received_extras_list[0]["my_extra_key"] == "my_extra_value"
    ), graft_input.received_extras_list
    return graft_input.received_list[0]
```
**EN:** This helper function encapsulates reusable logic so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 3906-3906: Define class TestGrafterE2eExample
```python
class TestGrafterE2eExample:
```
**EN:** This declaration introduces the `TestGrafterE2eExample` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestGrafterE2eExample` 测试类，并说明它通过继承承担的职责。

### Lines 3907-3938: Document the class `TestGrafterE2eExample`
```python
    """End-to-end example: target has a (suspected) buggy attention kernel.

    Story: target's attention kernel produces wrong outputs and we want to
    test "if we replace target's attention with baseline's, does the rest of
    the model converge?". The full graft wiring is:

      - At the attention call site, target sends its inputs (q/k/v) to
        baseline → baseline's local inputs are overwritten by target's, so
        baseline runs its (known-good) attention against the same inputs.
        This is a t->b graft on `attn_input`.
      - Both sides run the kernel.
      - Baseline sends its outputs back to target → target's outputs are
        overwritten by baseline's, so target's downstream sees baseline's
        attention result. This is a b->t graft on `attn_output`.

    Net effect: target's attention is semantically replaced by baseline's,
    without modifying target's source beyond inserting `dumper.dump` at the
    input/output sites. This test additionally demonstrates two recv-side
    customization hooks via `_e2e_transform`:

      * `grafter_extras={...}` per dump call — arbitrary per-call metadata
        the recv side can consume.
      * `DUMPER_GRAFTER_TRANSFORM_PATH` — a user-supplied function that
        decides what value the recv side actually copy_'s in (defaults to
        identity-by-rank when unset).

    The remaining call-site code is exactly:

        dumper.dump("attn_input", q, grafter_extras={"layer_id": 7})  # t -> b
        out = target_attention_kernel(q, ...)
        dumper.dump("attn_output", out, grafter_extras={"scale": 0.5})  # b -> t
    """
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestGrafterE2eExample`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestGrafterE2eExample`的设计意图。

### Lines 3940-3966: Run test: e2e buggy attn replaced by baseline
```python
    def test_e2e_buggy_attn_replaced_by_baseline(self):
        graft_port = find_available_port(29640)
        # All non-role env is shared by both sides; we set it in the parent
        # so the spawned subprocesses inherit it. DUMPER_GRAFTER_ENABLE and
        # DUMPER_GRAFTER_ROLE are deliberately *not* set here — they are set
        # by `_run_graft_test_split` per-rank, after which the global
        # `dumper` is rebuilt (so workers can use the global directly).
        with temp_set_env(
            DUMPER_ENABLE="1",
            DUMPER_ENABLE_OUTPUT_FILE="false",  # skip disk I/O for the test
            DUMPER_ENABLE_OUTPUT_CONSOLE="false",
            # Pin exp_name so the dumper doesn't auto-pick + log "Choose
            # exp_name=..." into the captured snapshot.
            DUMPER_EXP_NAME="grafter_e2e_test",
            DUMPER_GRAFTER_MASTER_ADDRESS="127.0.0.1",
            DUMPER_GRAFTER_MASTER_PORT=str(graft_port),
            DUMPER_GRAFTER_BASELINE_WORLD_SIZE="1",
            DUMPER_GRAFTER_TARGET_WORLD_SIZE="1",
            DUMPER_GRAFTER_B2T_FILTER="name == 'attn_output'",
            DUMPER_GRAFTER_T2B_FILTER="name == 'attn_input'",
            DUMPER_GRAFTER_GROUP_NAME="grafter_e2e",
            DUMPER_GRAFTER_TIMEOUT="30",
            DUMPER_GRAFTER_TRANSFORM_PATH=f"{__name__}._e2e_transform",
        ):
            outputs = _run_graft_test_split(self._worker_baseline, self._worker_target)

        self._assert_e2e_snapshot(outputs)
```
**EN:** This test method exercises e2e buggy attn replaced by baseline and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 e2e buggy attn replaced by baseline 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 3968-4009: Define helper: assert e2e snapshot (part 1)
```python
    @staticmethod
    def _assert_e2e_snapshot(outputs: dict) -> None:
        """Snapshot of the FULL per-role log timeline.

        Volatile fields (timestamps, ports, float diff values, tensor
        min/max/mean/samples, struct addresses) are masked with ad-hoc regex
        placeholders so the snapshot stays stable while still pinning
        everything else. The snapshot doubles as documentation of the logs a
        reader will see when running this E2E setup.

        Captured logs are unconditionally printed before asserting so a
        snapshot failure doesn't require a re-run.
        """
        baseline_log = outputs["baseline"]
        target_log = outputs["target"]

        print("\n=========== captured baseline log ===========")
        print(baseline_log)
        print("=========== captured target log ===========")
        print(target_log)
        print("===========================================")

        # Convenience tokens for verbose volatile substrings.
        prefix = r"\[Dumper, rank=\d+, t=\d+\.\d+\] "
        # `get_tensor_info(t)` for our tensors expands to a long line; we
        # match the leading struct fields verbatim and let the trailing
        # min/max/mean/sample fields wildcard out.
        tinfo_f32_4 = (
            r"type=<class 'torch\.Tensor'> shape=torch\.Size\(\[4\]\) "
            r"dtype=torch\.float32 device=cuda:\d stride=\(1,\) "
            r"req_grad=False .*"
        )
        diff = r"rel_diff=[-\d.eE+]+ max_abs=[-\d.eE+]+ mean_abs=[-\d.eE+]+"

        # `_dump_inner` automatically annotates tags with `recompute_status`
        # (always present, value depends on whether autograd recompute is
        # active — "disabled" in this test env).
        attn_input_tags = r"\{'name': 'attn_input', 'recompute_status': 'disabled'\}"
        attn_output_tags = r"\{'name': 'attn_output', 'recompute_status': 'disabled'\}"

        # Same dummy extras dict travels in both directions.
        extras_lit = r"\{'my_extra_key': 'my_extra_value'\}"
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterE2eExample` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterE2eExample` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 4011-4044: Define helper: assert e2e snapshot (part 2)
```python
        baseline_pattern = (
            r"\A"
            f"{prefix}\\[Grafter\\] init group: role=baseline "
            r"baseline_world=1 target_world=1 rank=0 "
            r"init_method=tcp://127\.0\.0\.1:\d+ backend=nccl "
            r"name=grafter_e2e\n"
            f"{prefix}\\[Grafter\\] recv role=baseline dir=t2b "
            f"tags={attn_input_tags} n_senders=1 "
            f"sender_extras=\\[{extras_lit}\\] "
            f"before_overridden={tinfo_f32_4} "
            f"to_override={tinfo_f32_4} "
            f"diff_pre_vs_new={diff}\n"
            f"{prefix}\\[Grafter\\] send role=baseline dir=b2t "
            f"tags={attn_output_tags} extras={extras_lit} "
            f"local={tinfo_f32_4}\n"
            r"\Z"
        )
        target_pattern = (
            r"\A"
            f"{prefix}\\[Grafter\\] init group: role=target "
            r"baseline_world=1 target_world=1 rank=1 "
            r"init_method=tcp://127\.0\.0\.1:\d+ backend=nccl "
            r"name=grafter_e2e\n"
            f"{prefix}\\[Grafter\\] send role=target dir=t2b "
            f"tags={attn_input_tags} extras={extras_lit} "
            f"local={tinfo_f32_4}\n"
            f"{prefix}\\[Grafter\\] recv role=target dir=b2t "
            f"tags={attn_output_tags} n_senders=1 "
            f"sender_extras=\\[{extras_lit}\\] "
            f"before_overridden={tinfo_f32_4} "
            f"to_override={tinfo_f32_4} "
            f"diff_pre_vs_new={diff}\n"
            r"\Z"
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterE2eExample` so the scenario stays organized.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterE2eExample` 内部调用，从而让场景结构更清晰。

### Lines 4046-4055: Define helper: assert e2e snapshot (part 3)
```python
        assert re.fullmatch(baseline_pattern, baseline_log, flags=re.DOTALL), (
            f"baseline log did not match snapshot.\n"
            f"--- pattern ---\n{baseline_pattern}\n"
            f"--- actual ---\n{baseline_log}"
        )
        assert re.fullmatch(target_pattern, target_log, flags=re.DOTALL), (
            f"target log did not match snapshot.\n"
            f"--- pattern ---\n{target_pattern}\n"
            f"--- actual ---\n{target_log}"
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterE2eExample` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterE2eExample` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 4057-4084: Define helper: worker baseline
```python
    @staticmethod
    def _worker_baseline():
        # In production code, callers just `from sglang.srt.debug_utils.dumper
        # import dumper` and call `dumper.dump(name, value)` — the env
        # configures the global Grafter for them. We do the same here.
        from sglang.srt.debug_utils.dumper import dumper

        # Step 1: graft input. target sends its q to baseline; baseline's
        # `_e2e_transform` runs on the recv side, asserts the dummy extras
        # made it across, then returns target's q so baseline's local
        # placeholder is overwritten via .copy_().
        q = torch.tensor([99.0, 99.0, 99.0, 99.0], device="cuda:0")
        dumper.dump("attn_input", q)
        assert q.tolist() == [1.0, 2.0, 3.0, 4.0], (
            f"baseline's q should be overwritten by target's via the t->b graft, "
            f"got {q.tolist()}"
        )

        # Step 2: baseline runs the known-good attention kernel.
        attn_out = q * 10.0  # → [10, 20, 30, 40]

        # Step 3: graft output. baseline sends attn_out to target with a
        # dummy extras key the recv-side transform will assert on.
        dumper.dump(
            "attn_output",
            attn_out,
            grafter_extras={"my_extra_key": "my_extra_value"},
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterE2eExample` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterE2eExample` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 4086-4110: Define helper: worker target
```python
    @staticmethod
    def _worker_target():
        from sglang.srt.debug_utils.dumper import dumper

        # Step 1: graft input. target sends its real q to baseline along
        # with a dummy extras key the recv-side transform will assert on.
        q = torch.tensor([1.0, 2.0, 3.0, 4.0], device="cuda:1")
        dumper.dump(
            "attn_input",
            q,
            grafter_extras={"my_extra_key": "my_extra_value"},
        )

        # Step 2: target runs the (suspected buggy) attention kernel —
        # here it returns all zeros to mimic a broken implementation.
        attn_out = torch.zeros_like(q)

        # Step 3: graft output. baseline sends its (good) attn_out to
        # target; target's recv-side transform identity-passes it, so
        # target's local attn_out ends up = baseline's [10, 20, 30, 40].
        dumper.dump("attn_output", attn_out)
        assert attn_out.tolist() == [10.0, 20.0, 30.0, 40.0], (
            f"target's attn_out should be overwritten by baseline's via "
            f"the b->t graft, got {attn_out.tolist()}"
        )
```
**EN:** This helper function encapsulates reusable logic inside `TestGrafterE2eExample` so the scenario stays organized. It also checks concrete expectations with assertions.
**CN:** 该辅助函数封装了可复用逻辑，供 `TestGrafterE2eExample` 内部调用，从而让场景结构更清晰。 其中还会通过断言检查明确的预期。

### Lines 4113-4114: Expose unittest entrypoint
```python
if __name__ == "__main__":
    sys.exit(pytest.main([__file__]))
```
**EN:** This conditional keeps the module executable as a standalone unittest target.
**CN:** 该条件语句使模块可以作为独立的 unittest 目标直接运行。

## Key Concepts / 关键概念
- The file is structured as an automated regression test for a focused scenario. / 该文件按聚焦场景组织为自动化回归测试。
- CI registration keeps the case visible to automated test pipelines. / CI 注册使该用例能够被自动化测试流水线发现。
- Server lifecycle helpers start the target runtime before assertions begin. / 服务生命周期辅助函数会在断言开始前启动目标运行时。
- Explicit teardown avoids leaked child processes between runs. / 显式清理可以避免多次运行之间遗留子进程。
- HTTP requests validate the externally visible API contract rather than only internal helpers. / HTTP 请求验证的是对外暴露的 API 契约，而不只是内部辅助函数。

## Dependencies / 依赖关系
- Internal modules / 内部模块: `sglang.srt.debug_utils.dumper`, `sglang.srt.environ`, `sglang.srt.model_executor.forward_batch_info`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`
- External and stdlib modules / 外部与标准库模块: `contextlib`, `io`, `multiprocessing`, `os`, `pathlib`, `pytest`, `re`, `requests`, `sys`, `threading`, `time`, `torch`, `torch.distributed`, `torch.multiprocessing`, `traceback`, `typing`
- Notable symbols / 关键符号: `register_cuda_ci`, `register_amd_ci`, `popen_launch_server`, `kill_process_tree`, `requests.post`, `DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH`
