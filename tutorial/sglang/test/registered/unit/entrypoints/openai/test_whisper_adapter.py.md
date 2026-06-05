# test_whisper_adapter.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/entrypoints/openai/test_whisper_adapter.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates whisper adapter behavior in SGLang's unit / entrypoints / openai area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 入口 / OpenAI 领域中与 whisper adapter 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-8: supporting statements / 辅助语句
```python
"""Unit tests for the Whisper transcription adapter.

Focused on ``WhisperAdapter.parse_fused_output`` — a pure static method
that parses the fused auto-detect output into ``(language, user_visible_text)``.
``visible=None`` means "forced prefix not yet locatable; streaming callers
should keep buffering, non-streaming callers should fall back to a
best-effort scrub".
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 10-22: module imports and dependencies / 模块导入与依赖
```python
import re
import unittest
from typing import Any

from sglang.srt.entrypoints.openai.protocol import TranscriptionRequest
from sglang.srt.entrypoints.openai.transcription_adapters.whisper import (
    WHISPER_AUTODETECT_REGEX,
    WHISPER_AUTODETECT_TS_REGEX,
    WHISPER_LANG_TOKEN_CODES,
    WhisperAdapter,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `re`, `unittest`, `typing`, `sglang.srt.entrypoints.openai.protocol`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `re`, `unittest`, `typing`, `sglang.srt.entrypoints.openai.protocol`。

### Lines 24-24: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=2, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 27-27: class TestWhisperParseFusedOutput declaration / 类 TestWhisperParseFusedOutput 声明
```python
class TestWhisperParseFusedOutput(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 28-28: supporting statements / 辅助语句
```python
    """parse_fused_output: (language, visible) where visible=None means defer."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 30-34: test case happy english / 测试用例 happy english
```python
    def test_happy_english(self):
        lang, visible = WhisperAdapter.parse_fused_output(
            "<|en|><|transcribe|><|notimestamps|> Hello world"
        )
        self.assertEqual((lang, visible), ("en", "Hello world"))
```
**EN:** This test exercises `test_happy_english` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_happy_english`。

### Lines 36-40: test case happy non english / 测试用例 happy non english
```python
    def test_happy_non_english(self):
        lang, visible = WhisperAdapter.parse_fused_output(
            "<|zh|><|transcribe|><|notimestamps|>你好世界"
        )
        self.assertEqual((lang, visible), ("zh", "你好世界"))
```
**EN:** This test exercises `test_happy_non_english` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_happy_non_english`。

### Lines 42-48: test case missing language prefix defers / 测试用例 missing language prefix defers
```python
    def test_missing_language_prefix_defers(self):
        # Partial prefix or raw untagged text — streaming callers should
        # keep buffering; non-streaming callers fall back to best-effort.
        self.assertEqual(
            WhisperAdapter.parse_fused_output("raw untagged output"), (None, None)
        )
        self.assertEqual(WhisperAdapter.parse_fused_output(""), (None, None))
```
**EN:** This test exercises `test_missing_language_prefix_defers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_missing_language_prefix_defers`。

### Lines 50-52: test case missing sentinel defers / 测试用例 missing sentinel defers
```python
    def test_missing_sentinel_defers(self):
        # Reviewer's repro: <|zh|> Hi — language tag in but no sentinel.
        self.assertEqual(WhisperAdapter.parse_fused_output("<|zh|> Hi"), (None, None))
```
**EN:** This test exercises `test_missing_sentinel_defers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_missing_sentinel_defers`。

### Lines 54-57: test case truncated after transcribe defers / 测试用例 truncated after transcribe defers
```python
    def test_truncated_after_transcribe_defers(self):
        self.assertEqual(
            WhisperAdapter.parse_fused_output("<|en|><|transcribe|>"), (None, None)
        )
```
**EN:** This test exercises `test_truncated_after_transcribe_defers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_truncated_after_transcribe_defers`。

### Lines 59-65: test case unsupported language code defers / 测试用例 unsupported language code defers
```python
    def test_unsupported_language_code_defers(self):
        # FSM regex only allows ISO639_1_SUPPORTED_LANGS. A bypassed-FSM
        # <|xx|> must not leak through as a valid detection.
        self.assertEqual(
            WhisperAdapter.parse_fused_output("<|xx|><|transcribe|><|notimestamps|>hi"),
            (None, None),
        )
```
**EN:** This test exercises `test_unsupported_language_code_defers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unsupported_language_code_defers`。

### Lines 67-78: test case malformed prefix without transcribe defers / 测试用例 malformed prefix without transcribe defers
```python
    def test_malformed_prefix_without_transcribe_defers(self):
        # The parse must match the exact 3-token forced prefix, not
        # "lang tag + sentinel somewhere". A bypassed-FSM string that
        # skips <|transcribe|> must not parse as a valid detection.
        self.assertEqual(
            WhisperAdapter.parse_fused_output("<|en|>junk<|notimestamps|>text"),
            (None, None),
        )
        self.assertEqual(
            WhisperAdapter.parse_fused_output("<|en|><|0.00|> text"),
            (None, None),
        )
```
**EN:** This test exercises `test_malformed_prefix_without_transcribe_defers` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_malformed_prefix_without_transcribe_defers`。

### Lines 80-91: test case sentinel in but whitespace only returns empty visible / 测试用例 sentinel in but whitespace only returns empty visible
```python
    def test_sentinel_in_but_whitespace_only_returns_empty_visible(self):
        # Prefix arrived at a chunk boundary before the first word. The
        # .strip() collapses to "" so streaming callers see no delta yet;
        # the language is still reported as soon as the sentinel lands.
        self.assertEqual(
            WhisperAdapter.parse_fused_output("<|en|><|transcribe|><|notimestamps|>"),
            ("en", ""),
        )
        self.assertEqual(
            WhisperAdapter.parse_fused_output("<|en|><|transcribe|><|notimestamps|>  "),
            ("en", ""),
        )
```
**EN:** This test exercises `test_sentinel_in_but_whitespace_only_returns_empty_visible` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sentinel_in_but_whitespace_only_returns_empty_visible`。

### Lines 93-97: test case trailing endoftext scrubbed / 测试用例 trailing endoftext scrubbed
```python
    def test_trailing_endoftext_scrubbed(self):
        lang, visible = WhisperAdapter.parse_fused_output(
            "<|en|><|transcribe|><|notimestamps|> Hello world<|endoftext|>"
        )
        self.assertEqual((lang, visible), ("en", "Hello world"))
```
**EN:** This test exercises `test_trailing_endoftext_scrubbed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_trailing_endoftext_scrubbed`。

### Lines 99-110: test case embedded timestamp tokens scrubbed / 测试用例 embedded timestamp tokens scrubbed
```python
    def test_embedded_timestamp_tokens_scrubbed(self):
        # Defensive: in the ts variant Whisper's tokenizer normally
        # decodes <|X.XX|> tokens to "" so they never reach this path,
        # but if a future tokenizer leaks them through they must be
        # scrubbed from the user-visible text. verbose_json segment
        # timing comes from _parse_segments over output_ids on a
        # separate path.
        lang, visible = WhisperAdapter.parse_fused_output(
            "<|en|><|transcribe|><|0.00|> Hello<|5.00|> world<|10.00|><|endoftext|>",
            ts_variant=True,
        )
        self.assertEqual((lang, visible), ("en", "Hello world"))
```
**EN:** This test exercises `test_embedded_timestamp_tokens_scrubbed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedded_timestamp_tokens_scrubbed`。

### Lines 112-129: test case ts variant realistic decoded text / 测试用例 ts variant realistic decoded text
```python
    def test_ts_variant_realistic_decoded_text(self):
        # Real Whisper tokenizer decodes every <|X.XX|> timestamp token
        # (id 50365+) to "" even with skip_special_tokens=False, so for
        # the ts variant the cumulative text is just <|en|><|transcribe|>
        # followed directly by the BPE-decoded transcription. Asserts
        # that the parser handles this shape — without ts_variant=True
        # it would (correctly) defer because <|notimestamps|> is missing.
        lang, visible = WhisperAdapter.parse_fused_output(
            "<|en|><|transcribe|> Hello world<|endoftext|>", ts_variant=True
        )
        self.assertEqual((lang, visible), ("en", "Hello world"))
        # Same input under non-ts contract correctly defers.
        self.assertEqual(
            WhisperAdapter.parse_fused_output(
                "<|en|><|transcribe|> Hello world<|endoftext|>"
            ),
            (None, None),
        )
```
**EN:** This test exercises `test_ts_variant_realistic_decoded_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_ts_variant_realistic_decoded_text`。

### Lines 131-146: test case visible grows monotonically across snapshots / 测试用例 visible grows monotonically across snapshots
```python
    def test_visible_grows_monotonically_across_snapshots(self):
        # Streaming property: cumulative text produces cumulative visible.
        snapshots = [
            "<|en|><|transcribe|>",
            "<|en|><|transcribe|><|notimestamps|>",
            "<|en|><|transcribe|><|notimestamps|> Hello",
            "<|en|><|transcribe|><|notimestamps|> Hello world",
            "<|en|><|transcribe|><|notimestamps|> Hello world<|endoftext|>",
        ]
        visibles = [WhisperAdapter.parse_fused_output(s)[1] for s in snapshots]
        # (None, "", "Hello", "Hello world", "Hello world")
        self.assertEqual(visibles, [None, "", "Hello", "Hello world", "Hello world"])
        # Every non-None entry is a prefix of the next non-None entry.
        real = [v for v in visibles if v is not None]
        for a, b in zip(real, real[1:]):
            self.assertTrue(b.startswith(a), f"monotonicity broken: {a!r} -> {b!r}")
```
**EN:** This test exercises `test_visible_grows_monotonically_across_snapshots` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_visible_grows_monotonically_across_snapshots`。

### Lines 149-149: class TestWhisperLangTokenCoverage declaration / 类 TestWhisperLangTokenCoverage 声明
```python
class TestWhisperLangTokenCoverage(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 150-151: supporting statements / 辅助语句
```python
    """The FSM regex must cover every Whisper language token, not just the
    narrower ISO639_1_SUPPORTED_LANGS set used for input validation."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 153-162: test case three letter codes parse / 测试用例 three letter codes parse
```python
    def test_three_letter_codes_parse(self):
        # yue (Cantonese, v3), haw (Hawaiian), jw (Javanese, two-letter but
        # missing from ISO639_1_SUPPORTED_LANGS) — reviewer's flagged examples.
        for code in ("yue", "haw", "jw"):
            with self.subTest(lang=code):
                lang, visible = WhisperAdapter.parse_fused_output(
                    f"<|{code}|><|transcribe|><|notimestamps|> Hi"
                )
                self.assertEqual(lang, code)
                self.assertEqual(visible, "Hi")
```
**EN:** This test exercises `test_three_letter_codes_parse` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_three_letter_codes_parse`。

### Lines 164-167: test case known whisper langs in allowlist / 测试用例 known whisper langs in allowlist
```python
    def test_known_whisper_langs_in_allowlist(self):
        # Spot-check: codes the reviewer named + common 3-letter tokens.
        for code in ("yue", "haw", "jw", "su", "ba", "tt", "ln", "lo"):
            self.assertIn(code, WHISPER_LANG_TOKEN_CODES)
```
**EN:** This test exercises `test_known_whisper_langs_in_allowlist` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_known_whisper_langs_in_allowlist`。

### Lines 169-174: test case fsm regex includes three letter alternatives / 测试用例 fsm regex includes three letter alternatives
```python
    def test_fsm_regex_includes_three_letter_alternatives(self):
        # Defensive: the regex alternation must spell out the 3-letter codes
        # so xgrammar's FSM admits the <|yue|> / <|haw|> single-token path.
        for code in ("yue", "haw"):
            self.assertIn(re.escape(code), WHISPER_AUTODETECT_REGEX)
            self.assertIn(re.escape(code), WHISPER_AUTODETECT_TS_REGEX)
```
**EN:** This test exercises `test_fsm_regex_includes_three_letter_alternatives` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fsm_regex_includes_three_letter_alternatives`。

### Lines 176-187: test case autodetect codes round trip through input validator / 测试用例 autodetect codes round trip through input validator
```python
    def test_autodetect_codes_round_trip_through_input_validator(self):
        # A code returned by fused autodetect must be accepted as
        # ``language=`` on a follow-up request. Before the fix,
        # ``normalize_language_to_code("yue")`` raised ValueError even
        # though verbose_json could report ``"yue"`` from the same server.
        from sglang.srt.multimodal.processors.whisper import (
            normalize_language_to_code,
        )

        for code in ("yue", "haw", "jw", "ba", "su", "tt"):
            with self.subTest(lang=code):
                self.assertEqual(normalize_language_to_code(code), code)
```
**EN:** This test exercises `test_autodetect_codes_round_trip_through_input_validator` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_autodetect_codes_round_trip_through_input_validator`。

### Lines 189-219: test case unknown language token id raises clean error / 测试用例 unknown language token id raises clean error
```python
    def test_unknown_language_token_id_raises_clean_error(self):
        # Some Whisper codes (yue, v3-only) aren't in older checkpoints'
        # vocabs. The explicit-language path must raise a clean ValueError
        # in that case instead of silently feeding the unk token into the
        # decoder and producing garbage. Mocks cover both "returns None"
        # and "returns unk_token_id" tokenizer behaviors.
        from unittest.mock import Mock

        from sglang.srt.multimodal.processors.whisper import WhisperProcessor

        proc = WhisperProcessor.__new__(WhisperProcessor)
        # Tokenizer where <|yue|> is not in the vocab → returns unk_id.
        tok = Mock()
        tok.convert_tokens_to_ids = Mock(return_value=100)  # arbitrary unk
        tok.unk_token_id = 100
        proc._tokenizer = tok
        with self.assertRaises(ValueError) as ctx:
            proc._get_language_token_id("yue")
        self.assertIn("yue", str(ctx.exception))

        # Known code (English) on the same tokenizer still works.
        tok.convert_tokens_to_ids = Mock(return_value=50259)  # <|en|>
        self.assertEqual(proc._get_language_token_id("en"), 50259)

        # Some tokenizers return None for unknown tokens instead of unk_id.
        tok2 = Mock()
        tok2.convert_tokens_to_ids = Mock(return_value=None)
        tok2.unk_token_id = 100
        proc._tokenizer = tok2
        with self.assertRaises(ValueError):
            proc._get_language_token_id("yue")
```
**EN:** This test exercises `test_unknown_language_token_id_raises_clean_error` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_language_token_id_raises_clean_error`。

### Lines 222-222: class TestWhisperStripSpecialTokens declaration / 类 TestWhisperStripSpecialTokens 声明
```python
class TestWhisperStripSpecialTokens(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 223-223: supporting statements / 辅助语句
```python
    """Fallback scrub used when parse_fused_output defers."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 225-231: test case strips all whisper specials / 测试用例 strips all whisper specials
```python
    def test_strips_all_whisper_specials(self):
        self.assertEqual(
            WhisperAdapter.strip_special_tokens(
                "<|en|><|transcribe|><|0.00|>hi<|5.00|>world<|endoftext|>"
            ),
            "hiworld",
        )
```
**EN:** This test exercises `test_strips_all_whisper_specials` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_strips_all_whisper_specials`。

### Lines 233-237: test case identity on plain text / 测试用例 identity on plain text
```python
    def test_identity_on_plain_text(self):
        self.assertEqual(
            WhisperAdapter.strip_special_tokens("plain text"), "plain text"
        )
        self.assertEqual(WhisperAdapter.strip_special_tokens(""), "")
```
**EN:** This test exercises `test_identity_on_plain_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_identity_on_plain_text`。

### Lines 239-254: test case preserves spoken angle bracket sequences / 测试用例 preserves spoken angle bracket sequences
```python
    def test_preserves_spoken_angle_bracket_sequences(self):
        # The scrub must only remove actual Whisper special-token literals
        # (lang / control / <|X.XX|> timestamps), not arbitrary ``<|...|>``
        # patterns that can appear in transcribed speech (someone reading a
        # token name aloud, an AI-safety demo, code dictation, etc.).
        self.assertEqual(
            WhisperAdapter.strip_special_tokens("the token <|foo|> is unused"),
            "the token <|foo|> is unused",
        )
        # Real specials still scrubbed even when interleaved with bogus ones.
        self.assertEqual(
            WhisperAdapter.strip_special_tokens(
                "<|en|>hello <|foo|> world<|endoftext|>"
            ),
            "hello <|foo|> world",
        )
```
**EN:** This test exercises `test_preserves_spoken_angle_bracket_sequences` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preserves_spoken_angle_bracket_sequences`。

### Lines 256-261: test case parse preserves spoken angle bracket sequences / 测试用例 parse preserves spoken angle bracket sequences
```python
    def test_parse_preserves_spoken_angle_bracket_sequences(self):
        # Same for the per-chunk scrub inside parse_fused_output.
        lang, visible = WhisperAdapter.parse_fused_output(
            "<|en|><|transcribe|><|notimestamps|> look at <|foo|><|endoftext|>"
        )
        self.assertEqual((lang, visible), ("en", "look at <|foo|>"))
```
**EN:** This test exercises `test_parse_preserves_spoken_angle_bracket_sequences` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_parse_preserves_spoken_angle_bracket_sequences`。

### Lines 264-264: class TestWhisperBuildFusedAutodetectParams declaration / 类 TestWhisperBuildFusedAutodetectParams 声明
```python
class TestWhisperBuildFusedAutodetectParams(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 265-265: supporting statements / 辅助语句
```python
    """build_fused_autodetect_params picks the right regex + propagates ts param."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 267-270: method request / 方法 request
```python
    def _request(self, **kwargs: Any) -> TranscriptionRequest:
        base: dict[str, Any] = dict(model="whisper", temperature=0.0)
        base.update(kwargs)
        return TranscriptionRequest(**base)
```
**EN:** This block implements `_request` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 272-275: test case no timestamps uses notimestamps regex / 测试用例 no timestamps uses notimestamps regex
```python
    def test_no_timestamps_uses_notimestamps_regex(self):
        params = WhisperAdapter().build_fused_autodetect_params(self._request())
        self.assertEqual(params["regex"], WHISPER_AUTODETECT_REGEX)
        self.assertNotIn("timestamp_granularities", params)
```
**EN:** This test exercises `test_no_timestamps_uses_notimestamps_regex` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_timestamps_uses_notimestamps_regex`。

### Lines 277-281: test case timestamps uses ts regex and propagates granularities / 测试用例 timestamps uses ts regex and propagates granularities
```python
    def test_timestamps_uses_ts_regex_and_propagates_granularities(self):
        req = self._request(timestamp_granularities=["segment"])
        params = WhisperAdapter().build_fused_autodetect_params(req)
        self.assertEqual(params["regex"], WHISPER_AUTODETECT_TS_REGEX)
        self.assertEqual(params["timestamp_granularities"], ["segment"])
```
**EN:** This test exercises `test_timestamps_uses_ts_regex_and_propagates_granularities` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_timestamps_uses_ts_regex_and_propagates_granularities`。

### Lines 283-288: test case empty timestamps list uses notimestamps regex / 测试用例 empty timestamps list uses notimestamps regex
```python
    def test_empty_timestamps_list_uses_notimestamps_regex(self):
        # Empty list is falsy — treat as "no timestamps requested".
        req = self._request(timestamp_granularities=[])
        params = WhisperAdapter().build_fused_autodetect_params(req)
        self.assertEqual(params["regex"], WHISPER_AUTODETECT_REGEX)
        self.assertNotIn("timestamp_granularities", params)
```
**EN:** This test exercises `test_empty_timestamps_list_uses_notimestamps_regex` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_timestamps_list_uses_notimestamps_regex`。

### Lines 290-299: test case spaces between special tokens is false / 测试用例 spaces between special tokens is false
```python
    def test_spaces_between_special_tokens_is_false(self):
        # parse_fused_output assumes a zero-space forced prefix. Slow
        # Whisper tokenizers otherwise insert a space between adjacent
        # special tokens, which would silently break the parse path.
        for req in (
            self._request(),
            self._request(timestamp_granularities=["segment"]),
        ):
            params = WhisperAdapter().build_fused_autodetect_params(req)
            self.assertIs(params["spaces_between_special_tokens"], False)
```
**EN:** This test exercises `test_spaces_between_special_tokens_is_false` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_spaces_between_special_tokens_is_false`。

### Lines 301-314: test case fused params survive sampling params construction / 测试用例 fused params survive sampling params construction
```python
    def test_fused_params_survive_sampling_params_construction(self):
        # Regression: the multimodal processor's fused branch used to skip
        # popping `timestamp_granularities`, leaking the key into
        # SamplingParams(**kwargs) → TypeError on any language=None +
        # timestamp_granularities request. Mirrors what the processor does
        # before constructing SamplingParams.
        from sglang.srt.sampling.sampling_params import SamplingParams

        req = self._request(timestamp_granularities=["segment"])
        params = WhisperAdapter().build_fused_autodetect_params(req)
        # Fields the processor pops before SamplingParams(**kwargs).
        params.pop("_detect_language", None)
        params.pop("timestamp_granularities", None)
        SamplingParams(**params)
```
**EN:** This test exercises `test_fused_params_survive_sampling_params_construction` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_fused_params_survive_sampling_params_construction`。

### Lines 317-318: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestWhisperParseFusedOutput`: parse_fused_output: (language, visible) where visible=None means defer. / 用于组织相关测试、夹具或辅助方法。
- `TestWhisperLangTokenCoverage`: The FSM regex must cover every Whisper language token, not just the narrower ISO639_1_SUPPORTED_LANGS set used for input validation. / 用于组织相关测试、夹具或辅助方法。
- `TestWhisperStripSpecialTokens`: Fallback scrub used when parse_fused_output defers. / 用于组织相关测试、夹具或辅助方法。
- `TestWhisperBuildFusedAutodetectParams`: build_fused_autodetect_params picks the right regex + propagates ts param. / 用于组织相关测试、夹具或辅助方法。
- `TestWhisperParseFusedOutput.test_happy_english`: This test exercises `test_happy_english` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_happy_english`。
- `TestWhisperParseFusedOutput.test_happy_non_english`: This test exercises `test_happy_non_english` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_happy_non_english`。
- `TestWhisperParseFusedOutput.test_missing_language_prefix_defers`: This test exercises `test_missing_language_prefix_defers` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_missing_language_prefix_defers`。
- `TestWhisperParseFusedOutput.test_missing_sentinel_defers`: This test exercises `test_missing_sentinel_defers` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_missing_sentinel_defers`。
- `TestWhisperParseFusedOutput.test_truncated_after_transcribe_defers`: This test exercises `test_truncated_after_transcribe_defers` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_truncated_after_transcribe_defers`。
- `TestWhisperParseFusedOutput.test_unsupported_language_code_defers`: This test exercises `test_unsupported_language_code_defers` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unsupported_language_code_defers`。
- `TestWhisperParseFusedOutput.test_malformed_prefix_without_transcribe_defers`: This test exercises `test_malformed_prefix_without_transcribe_defers` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_malformed_prefix_without_transcribe_defers`。
- `TestWhisperParseFusedOutput.test_sentinel_in_but_whitespace_only_returns_empty_visible`: This test exercises `test_sentinel_in_but_whitespace_only_returns_empty_visible` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_sentinel_in_but_whitespace_only_returns_empty_visible`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `re`, `unittest`, `typing`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.entrypoints.openai.transcription_adapters.whisper`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 318
