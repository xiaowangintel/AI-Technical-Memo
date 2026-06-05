# test_load_plugins.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/plugins/test_load_plugins.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates load plugins behavior in SGLang's unit / plugins area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 插件 领域中与 load plugins 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-9: supporting statements / 辅助语句
```python
"""
Unit tests for the plugin loading flow.

Covers: idempotency, apply_hooks invocation, exception resilience,
SGLANG_PLUGINS whitelist, SGLANG_PLATFORM exclusion logic,
and _current_plugin_source context var reset.

Run:  python -m pytest test/registered/unit/plugins/test_load_plugins.py -v
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 11-20: module imports and dependencies / 模块导入与依赖
```python
from unittest.mock import MagicMock, patch

from sglang.srt.plugins import (
    _current_plugin_source,
    _get_excluded_dists,
    load_plugins,
    load_plugins_by_group,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest.mock`, `sglang.srt.plugins`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest.mock`, `sglang.srt.plugins`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`。

### Lines 22-22: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 25-36: function make ep / 函数 make ep
```python
def _make_ep(name, dist_name=None, load_fn=None):
    """Create a mock entry point."""
    ep = MagicMock()
    ep.name = name
    ep.value = f"fake_module:{name}"
    ep.dist = MagicMock()
    ep.dist.name = dist_name or f"{name}-dist"
    if load_fn is not None:
        ep.load.return_value = load_fn
    else:
        ep.load.return_value = MagicMock()
    return ep
```
**EN:** Create a mock entry point. This block implements `_make_ep` and captures one focused piece of the module's behavior.
**CN:** Create a mock entry point. 该代码块实现 `_make_ep`，承担模块行为中的一个聚焦逻辑片段。

### Lines 39-43: function reset plugins loaded / 函数 reset plugins loaded
```python
def _reset_plugins_loaded():
    """Reset the _plugins_loaded flag so load_plugins() can run again."""
    import sglang.srt.plugins as plugins_mod

    plugins_mod._plugins_loaded = False
```
**EN:** Reset the _plugins_loaded flag so load_plugins() can run again. This block implements `_reset_plugins_loaded` and captures one focused piece of the module's behavior.
**CN:** Reset the _plugins_loaded flag so load_plugins() can run again. 该代码块实现 `_reset_plugins_loaded`，承担模块行为中的一个聚焦逻辑片段。

### Lines 46-46: class TestLoadPlugins declaration / 类 TestLoadPlugins 声明
```python
class TestLoadPlugins(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 47-47: supporting statements / 辅助语句
```python
    """Tests for load_plugins() and related helpers."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 49-50: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        _reset_plugins_loaded()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 52-53: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        _reset_plugins_loaded()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 55-69: test case load plugins idempotent and calls apply / 测试用例 load plugins idempotent and calls apply
```python
    @patch("sglang.srt.plugins.HookRegistry")
    @patch("sglang.srt.plugins.envs")
    @patch("sglang.srt.plugins.entry_points", return_value=[])
    def test_load_plugins_idempotent_and_calls_apply(
        self, mock_eps, mock_envs, mock_registry
    ):
        """Second call is a no-op; first call invokes apply_hooks."""
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        mock_envs.SGLANG_PLUGINS.get.return_value = ""

        load_plugins()
        self.assertEqual(mock_registry.apply_hooks.call_count, 1)

        load_plugins()  # should be skipped
        self.assertEqual(mock_registry.apply_hooks.call_count, 1)
```
**EN:** Second call is a no-op; first call invokes apply_hooks. This test exercises `test_load_plugins_idempotent_and_calls_apply` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Second call is a no-op; first call invokes apply_hooks. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_load_plugins_idempotent_and_calls_apply`。

### Lines 71-98: test case plugin exception does not crash / 测试用例 plugin exception does not crash
```python
    @patch("sglang.srt.plugins.HookRegistry")
    @patch("sglang.srt.plugins.envs")
    @patch("sglang.srt.plugins.entry_points")
    def test_plugin_exception_does_not_crash(self, mock_eps, mock_envs, mock_registry):
        """A failing plugin should not prevent others from loading."""
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        mock_envs.SGLANG_PLUGINS.get.return_value = ""

        def bad_plugin():
            raise RuntimeError("boom")

        good_call_log = []

        def good_plugin():
            good_call_log.append("ok")

        eps = [
            _make_ep("bad", load_fn=bad_plugin),
            _make_ep("good", load_fn=good_plugin),
        ]
        mock_eps.return_value = eps

        with self.assertLogs("sglang.srt.plugins", level="ERROR") as cm:
            load_plugins()

        self.assertTrue(any("boom" in msg for msg in cm.output))
        self.assertEqual(good_call_log, ["ok"])
        mock_registry.apply_hooks.assert_called_once()
```
**EN:** A failing plugin should not prevent others from loading. This test exercises `test_plugin_exception_does_not_crash` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** A failing plugin should not prevent others from loading. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_plugin_exception_does_not_crash`。

### Lines 100-121: test case sglang plugins whitelist / 测试用例 sglang plugins whitelist
```python
    @patch("sglang.srt.plugins.entry_points")
    @patch("sglang.srt.plugins.envs")
    def test_sglang_plugins_whitelist(self, mock_envs, mock_eps):
        """Only plugins named in SGLANG_PLUGINS should be loaded."""
        mock_envs.SGLANG_PLUGINS.get.return_value = "alpha,gamma"
        mock_envs.SGLANG_PLATFORM.get.return_value = ""

        alpha_fn = MagicMock()
        beta_fn = MagicMock()
        gamma_fn = MagicMock()

        eps = [
            _make_ep("alpha", load_fn=alpha_fn),
            _make_ep("beta", load_fn=beta_fn),
            _make_ep("gamma", load_fn=gamma_fn),
        ]
        mock_eps.return_value = eps

        result = load_plugins_by_group("test.group")
        self.assertIn("alpha", result)
        self.assertNotIn("beta", result)
        self.assertIn("gamma", result)
```
**EN:** Only plugins named in SGLANG_PLUGINS should be loaded. This test exercises `test_sglang_plugins_whitelist` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Only plugins named in SGLANG_PLUGINS should be loaded. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sglang_plugins_whitelist`。

### Lines 123-139: test case excluded dists / 测试用例 excluded dists
```python
    @patch("sglang.srt.plugins.entry_points")
    @patch("sglang.srt.plugins.envs")
    def test_excluded_dists(self, mock_envs, mock_eps):
        """SGLANG_PLATFORM excludes other platform dists; empty when unset."""
        # Case 1: no env set → empty
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        self.assertEqual(_get_excluded_dists(), set())

        # Case 2: env set → exclude other dists
        mock_envs.SGLANG_PLATFORM.get.return_value = "kunlun"
        ep_kunlun = _make_ep("kunlun", dist_name="kunlun-pkg")
        ep_other = _make_ep("other_hw", dist_name="other-pkg")
        mock_eps.return_value = [ep_kunlun, ep_other]

        excluded = _get_excluded_dists()
        self.assertNotIn("kunlun-pkg", excluded)
        self.assertIn("other-pkg", excluded)
```
**EN:** SGLANG_PLATFORM excludes other platform dists; empty when unset. This test exercises `test_excluded_dists` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** SGLANG_PLATFORM excludes other platform dists; empty when unset. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_excluded_dists`。

### Lines 141-163: test case current plugin source set during and reset after / 测试用例 current plugin source set during and reset after
```python
    @patch("sglang.srt.plugins.HookRegistry")
    @patch("sglang.srt.plugins.envs")
    @patch("sglang.srt.plugins.entry_points")
    def test_current_plugin_source_set_during_and_reset_after(
        self, mock_eps, mock_envs, mock_registry
    ):
        """_current_plugin_source is set during plugin execution, reset after."""
        sources_seen = []

        def spy_plugin():
            sources_seen.append(_current_plugin_source.get())

        mock_eps.return_value = [_make_ep("spy", load_fn=spy_plugin)]
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        mock_envs.SGLANG_PLUGINS.get.return_value = ""

        load_plugins()
        # During execution: source was set (not None)
        self.assertEqual(len(sources_seen), 1)
        self.assertIsNotNone(sources_seen[0])
        self.assertEqual(sources_seen[0].plugin_name, "spy")
        # After execution: source is back to None
        self.assertIsNone(_current_plugin_source.get())
```
**EN:** _current_plugin_source is set during plugin execution, reset after. This test exercises `test_current_plugin_source_set_during_and_reset_after` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** _current_plugin_source is set during plugin execution, reset after. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_current_plugin_source_set_during_and_reset_after`。

### Lines 165-181: test case current plugin source reset after exception / 测试用例 current plugin source reset after exception
```python
    @patch("sglang.srt.plugins.HookRegistry")
    @patch("sglang.srt.plugins.envs")
    @patch("sglang.srt.plugins.entry_points")
    def test_current_plugin_source_reset_after_exception(
        self, mock_eps, mock_envs, mock_registry
    ):
        """_current_plugin_source is reset to None even when a plugin raises."""
        mock_envs.SGLANG_PLATFORM.get.return_value = ""
        mock_envs.SGLANG_PLUGINS.get.return_value = ""

        def bad_plugin():
            raise RuntimeError("boom")

        mock_eps.return_value = [_make_ep("bad", load_fn=bad_plugin)]

        load_plugins()
        self.assertIsNone(_current_plugin_source.get())
```
**EN:** _current_plugin_source is reset to None even when a plugin raises. This test exercises `test_current_plugin_source_reset_after_exception` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** _current_plugin_source is reset to None even when a plugin raises. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_current_plugin_source_reset_after_exception`。

### Lines 184-187: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    import unittest

    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_make_ep`: Create a mock entry point. / 该代码块实现 `_make_ep`，承担模块行为中的一个聚焦逻辑片段。
- `_reset_plugins_loaded`: Reset the _plugins_loaded flag so load_plugins() can run again. / 该代码块实现 `_reset_plugins_loaded`，承担模块行为中的一个聚焦逻辑片段。
- `TestLoadPlugins`: Tests for load_plugins() and related helpers. / 用于组织相关测试、夹具或辅助方法。
- `TestLoadPlugins.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestLoadPlugins.tearDown`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestLoadPlugins.test_load_plugins_idempotent_and_calls_apply`: Second call is a no-op; first call invokes apply_hooks. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_load_plugins_idempotent_and_calls_apply`。
- `TestLoadPlugins.test_plugin_exception_does_not_crash`: A failing plugin should not prevent others from loading. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_plugin_exception_does_not_crash`。
- `TestLoadPlugins.test_sglang_plugins_whitelist`: Only plugins named in SGLANG_PLUGINS should be loaded. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sglang_plugins_whitelist`。
- `TestLoadPlugins.test_excluded_dists`: SGLANG_PLATFORM excludes other platform dists; empty when unset. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_excluded_dists`。
- `TestLoadPlugins.test_current_plugin_source_set_during_and_reset_after`: _current_plugin_source is set during plugin execution, reset after. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_current_plugin_source_set_during_and_reset_after`。
- `TestLoadPlugins.test_current_plugin_source_reset_after_exception`: _current_plugin_source is reset to None even when a plugin raises. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_current_plugin_source_reset_after_exception`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest.mock`
- **Internal modules / 内部模块**: `sglang.srt.plugins`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 187
