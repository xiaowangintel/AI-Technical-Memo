# test_patch_tokenizer.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/utils/test_patch_tokenizer.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates patch tokenizer behavior in SGLang's unit / utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 工具 领域中与 patch tokenizer 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-12: module imports and dependencies / 模块导入与依赖
```python
import random
import unittest
from contextlib import contextmanager

from transformers import AutoTokenizer

from sglang.srt.utils.patch_tokenizer import (
    _SpecialTokensCachePatcher,
    decode_without_hf_kwargs,
    unpatch_tokenizer,
)
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `random`, `unittest`, `contextlib`, `transformers`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `random`, `unittest`, `contextlib`, `transformers`。

### Lines 14-14: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=30, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 17-17: class TestPatchTokenizerEndToEndTest declaration / 类 TestPatchTokenizerEndToEndTest 声明
```python
class TestPatchTokenizerEndToEndTest(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 18-27: test case patched produces same results as raw / 测试用例 patched produces same results as raw
```python
    def test_patched_produces_same_results_as_raw(self):
        tokenizer = _load_tokenizer()
        test_texts = self._generate_test_texts(tokenizer)
        raw_results = self._run_tokenizer_ops(tokenizer, test_texts)

        _SpecialTokensCachePatcher.patch(tokenizer)
        patched_results = self._run_tokenizer_ops(tokenizer, test_texts)
        unpatch_tokenizer(tokenizer)

        self.assertEqual(raw_results, patched_results)
```
**EN:** This test exercises `test_patched_produces_same_results_as_raw` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_patched_produces_same_results_as_raw`。

### Lines 29-48: method generate test texts / 方法 generate test texts
```python
    @classmethod
    def _generate_test_texts(cls, tokenizer):
        special_tokens = tokenizer.all_special_tokens
        return [
            "Hello, world!",
            "This is a longer sentence with multiple words.",
            "Numbers 12345 and symbols !@#$%",
            "    leading and trailing spaces    ",
            "\n\nMultiple\n\nNewlines\n\n",
            *[f"Text with {tok} inside" for tok in special_tokens],
            " ".join(special_tokens),
            *[
                cls._random_text_from_tokens(tokenizer, num_tokens=100)
                for _ in range(5)
            ],
            *[
                cls._random_text_from_tokens(tokenizer, num_tokens=1000)
                for _ in range(3)
            ],
        ]
```
**EN:** This block implements `_generate_test_texts` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_generate_test_texts`，承担模块行为中的一个聚焦逻辑片段。

### Lines 50-55: method random text from tokens / 方法 random text from tokens
```python
    @classmethod
    def _random_text_from_tokens(cls, tokenizer, num_tokens):
        token_ids = [
            random.randint(0, tokenizer.vocab_size - 1) for _ in range(num_tokens)
        ]
        return tokenizer.decode(token_ids)
```
**EN:** This block implements `_random_text_from_tokens` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_random_text_from_tokens`，承担模块行为中的一个聚焦逻辑片段。

### Lines 57-73: method run tokenizer ops / 方法 run tokenizer ops
```python
    @classmethod
    def _run_tokenizer_ops(cls, tokenizer, texts):
        encode_results = [tokenizer.encode(t) for t in texts]
        batch_encode_results = tokenizer(texts)["input_ids"]
        return {
            "encode": encode_results,
            "batch_encode": batch_encode_results,
            "decode": [
                tokenizer.decode(ids, skip_special_tokens=True)
                for ids in encode_results
            ],
            "batch_decode": tokenizer.batch_decode(
                encode_results, skip_special_tokens=True
            ),
            "special_tokens": tokenizer.all_special_tokens,
            "special_ids": tokenizer.all_special_ids,
        }
```
**EN:** This block implements `_run_tokenizer_ops` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_tokenizer_ops`，承担模块行为中的一个聚焦逻辑片段。

### Lines 76-76: class TestPatchTokenizerUnitTest declaration / 类 TestPatchTokenizerUnitTest 声明
```python
class TestPatchTokenizerUnitTest(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 77-105: test case patch unpatch restores original / 测试用例 patch unpatch restores original
```python
    def test_patch_unpatch_restores_original(self):
        tokenizer = _load_tokenizer()
        cls = type(tokenizer)

        original_ids = _get_class_attr_ids(cls)

        _SpecialTokensCachePatcher.patch(tokenizer)
        self.assertTrue(getattr(cls, "_sglang_special_tokens_patched", False))

        patched_ids = _get_class_attr_ids(cls)
        changed_attrs = [
            name
            for name in original_ids
            if name in patched_ids and patched_ids[name] != original_ids[name]
        ]
        self.assertGreater(len(changed_attrs), 0, "Patch should change some attributes")

        unpatch_tokenizer(tokenizer)
        self.assertFalse(getattr(cls, "_sglang_special_tokens_patched", False))

        restored_ids = _get_class_attr_ids(cls)
        for name in original_ids:
            if name.startswith("_sglang") or name.startswith("_original"):
                continue
            self.assertEqual(
                restored_ids.get(name),
                original_ids[name],
                f"Attribute {name} should be restored to original",
            )
```
**EN:** This test exercises `test_patch_unpatch_restores_original` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_patch_unpatch_restores_original`。

### Lines 107-115: test case patch caches special tokens / 测试用例 patch caches special tokens
```python
    def test_patch_caches_special_tokens(self):
        with _patched_tokenizer() as tokenizer:
            tokens1 = tokenizer.all_special_tokens
            ids1 = tokenizer.all_special_ids
            tokens2 = tokenizer.all_special_tokens
            ids2 = tokenizer.all_special_ids

            self.assertIs(tokens1, tokens2)
            self.assertIs(ids1, ids2)
```
**EN:** This test exercises `test_patch_caches_special_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_patch_caches_special_tokens`。

### Lines 117-123: test case patch blocks add special tokens / 测试用例 patch blocks add special tokens
```python
    def test_patch_blocks_add_special_tokens(self):
        with _patched_tokenizer() as tokenizer:
            with self.assertRaises(AssertionError) as ctx:
                tokenizer.add_special_tokens({"pad_token": "<pad>"})
            self.assertIn(
                "Cannot modify special tokens after patch", str(ctx.exception)
            )
```
**EN:** This test exercises `test_patch_blocks_add_special_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_patch_blocks_add_special_tokens`。

### Lines 125-131: test case patch blocks add tokens with special flag / 测试用例 patch blocks add tokens with special flag
```python
    def test_patch_blocks_add_tokens_with_special_flag(self):
        with _patched_tokenizer() as tokenizer:
            with self.assertRaises(AssertionError) as ctx:
                tokenizer.add_tokens(["<new>"], special_tokens=True)
            self.assertIn("Cannot add special tokens after patch", str(ctx.exception))

            tokenizer.add_tokens(["<regular>"], special_tokens=False)
```
**EN:** This test exercises `test_patch_blocks_add_tokens_with_special_flag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_patch_blocks_add_tokens_with_special_flag`。

### Lines 133-141: test case unpatch clears cache / 测试用例 unpatch clears cache
```python
    def test_unpatch_clears_cache(self):
        with _patched_tokenizer() as tokenizer:
            _ = tokenizer.all_special_tokens
            _ = tokenizer.all_special_ids
            self.assertTrue(hasattr(tokenizer, "_sglang_cached_special_tokens"))
            self.assertTrue(hasattr(tokenizer, "_sglang_cached_special_ids"))

        self.assertFalse(hasattr(tokenizer, "_sglang_cached_special_tokens"))
        self.assertFalse(hasattr(tokenizer, "_sglang_cached_special_ids"))
```
**EN:** This test exercises `test_unpatch_clears_cache` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unpatch_clears_cache`。

### Lines 143-152: test case double patch is idempotent / 测试用例 double patch is idempotent
```python
    def test_double_patch_is_idempotent(self):
        tokenizer = _load_tokenizer()
        _SpecialTokensCachePatcher.patch(tokenizer)
        _SpecialTokensCachePatcher.patch(tokenizer)

        self.assertTrue(
            getattr(type(tokenizer), "_sglang_special_tokens_patched", False)
        )

        unpatch_tokenizer(tokenizer)
```
**EN:** This test exercises `test_double_patch_is_idempotent` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_double_patch_is_idempotent`。

### Lines 154-165: test case decode without hf kwargs uses native decode / 测试用例 decode without hf kwargs uses native decode
```python
    def test_decode_without_hf_kwargs_uses_native_decode(self):
        tokenizer = _FakeDecodeTokenizer()

        self.assertEqual(
            decode_without_hf_kwargs(tokenizer, [1, 99, 2], True),
            "ab",
        )
        self.assertEqual(
            decode_without_hf_kwargs(tokenizer, [1, 99, 2], False),
            "a<special>b",
        )
        self.assertEqual(tokenizer.decode_calls, [[1, 2], [1, 99, 2]])
```
**EN:** This test exercises `test_decode_without_hf_kwargs_uses_native_decode` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_decode_without_hf_kwargs_uses_native_decode`。

### Lines 168-171: function get class attr ids / 函数 get class attr ids
```python
def _get_class_attr_ids(cls):
    return {
        n: id(v.fget if isinstance(v, property) else v) for n, v in vars(cls).items()
    }
```
**EN:** This block implements `_get_class_attr_ids` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_get_class_attr_ids`，承担模块行为中的一个聚焦逻辑片段。

### Lines 174-178: function load tokenizer / 函数 load tokenizer
```python
def _load_tokenizer():
    # The slowness is mainly observed in Kimi
    return AutoTokenizer.from_pretrained(
        "nvidia/Kimi-K2-Thinking-NVFP4", trust_remote_code=True
    )
```
**EN:** This block implements `_load_tokenizer` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_load_tokenizer`，承担模块行为中的一个聚焦逻辑片段。

### Lines 181-188: function patched tokenizer / 函数 patched tokenizer
```python
@contextmanager
def _patched_tokenizer():
    tokenizer = _load_tokenizer()
    _SpecialTokensCachePatcher.patch(tokenizer)
    try:
        yield tokenizer
    finally:
        unpatch_tokenizer(tokenizer)
```
**EN:** This block implements `_patched_tokenizer` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_patched_tokenizer`，承担模块行为中的一个聚焦逻辑片段。

### Lines 191-191: class _FakeDecodeTokenizer declaration / 类 _FakeDecodeTokenizer 声明
```python
class _FakeDecodeTokenizer:
```
**EN:** This section introduces the class and any class-level context used by later methods.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。

### Lines 192-192: class-level constants and configuration for `_FakeDecodeTokenizer` / 类级常量与配置
```python
    all_special_ids_set = {99}
```
**EN:** This block defines shared names such as `all_special_ids_set`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `all_special_ids_set` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 194-195: method init / 方法 init
```python
    def __init__(self):
        self.decode_calls = []
```
**EN:** This block implements `__init__` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `__init__`，承担模块行为中的一个聚焦逻辑片段。

### Lines 197-201: method decode / 方法 decode
```python
    def decode(self, token_ids):
        token_ids = list(token_ids)
        self.decode_calls.append(token_ids)
        token_text = {1: "a", 2: "b", 99: "<special>"}
        return "".join(token_text[token_id] for token_id in token_ids)
```
**EN:** This block implements `decode` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `decode`，承担模块行为中的一个聚焦逻辑片段。

### Lines 204-205: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPatchTokenizerEndToEndTest`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPatchTokenizerUnitTest`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_get_class_attr_ids`: This block implements `_get_class_attr_ids` and captures one focused piece of the module's behavior. / 该代码块实现 `_get_class_attr_ids`，承担模块行为中的一个聚焦逻辑片段。
- `_load_tokenizer`: This block implements `_load_tokenizer` and captures one focused piece of the module's behavior. / 该代码块实现 `_load_tokenizer`，承担模块行为中的一个聚焦逻辑片段。
- `_patched_tokenizer`: This block implements `_patched_tokenizer` and captures one focused piece of the module's behavior. / 该代码块实现 `_patched_tokenizer`，承担模块行为中的一个聚焦逻辑片段。
- `_FakeDecodeTokenizer`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestPatchTokenizerEndToEndTest.test_patched_produces_same_results_as_raw`: This test exercises `test_patched_produces_same_results_as_raw` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_patched_produces_same_results_as_raw`。
- `TestPatchTokenizerEndToEndTest._generate_test_texts`: This block implements `_generate_test_texts` and captures one focused piece of the module's behavior. / 该代码块实现 `_generate_test_texts`，承担模块行为中的一个聚焦逻辑片段。
- `TestPatchTokenizerEndToEndTest._random_text_from_tokens`: This block implements `_random_text_from_tokens` and captures one focused piece of the module's behavior. / 该代码块实现 `_random_text_from_tokens`，承担模块行为中的一个聚焦逻辑片段。
- `TestPatchTokenizerEndToEndTest._run_tokenizer_ops`: This block implements `_run_tokenizer_ops` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_tokenizer_ops`，承担模块行为中的一个聚焦逻辑片段。
- `TestPatchTokenizerUnitTest.test_patch_unpatch_restores_original`: This test exercises `test_patch_unpatch_restores_original` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_patch_unpatch_restores_original`。
- `TestPatchTokenizerUnitTest.test_patch_caches_special_tokens`: This test exercises `test_patch_caches_special_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_patch_caches_special_tokens`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `random`, `unittest`, `contextlib`
- **Third-party modules / 第三方模块**: `transformers`
- **Internal modules / 内部模块**: `sglang.srt.utils.patch_tokenizer`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 205
