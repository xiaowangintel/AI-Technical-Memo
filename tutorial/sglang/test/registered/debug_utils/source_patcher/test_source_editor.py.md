# test_source_editor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/debug_utils/source_patcher/test_source_editor.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module focuses on source patcher source editor in SGLang. It captures the setup, helper logic, and expectations required by this scenario. / 该测试模块用于分析 SGLang 中与 source patcher source editor 相关的实现或行为。 它概括了该场景所需的初始化、辅助逻辑与预期结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-6: Import dependencies
```python
import pytest
from pydantic import ValidationError

from sglang.srt.debug_utils.source_patcher.source_editor import apply_edits
from sglang.srt.debug_utils.source_patcher.types import EditSpec, PatchApplicationError
from sglang.test.ci.ci_register import register_cpu_ci
```
**EN:** This block imports the libraries, fixtures, and helpers that the rest of the file relies on.
**CN:** 该代码块导入了此文件后续逻辑依赖的库、夹具与辅助工具。

### Lines 8-8: Register CI metadata
```python
register_cpu_ci(est_time=10, suite="base-a-test-cpu", nightly=True)
```
**EN:** This block registers the test case in the CI matrix together with execution-time or runner metadata.
**CN:** 该代码块把测试用例注册到 CI 矩阵中，并附带执行耗时或运行器元数据。

### Lines 11-11: Define class TestApplyEdits
```python
class TestApplyEdits:
```
**EN:** This declaration introduces the `TestApplyEdits` test class and its inheritance-based responsibilities.
**CN:** 该声明引入了 `TestApplyEdits` 测试类，并说明它通过继承承担的职责。

### Lines 12-12: Document the class `TestApplyEdits`
```python
    """Tests for the apply_edits() source text transformation function."""
```
**EN:** This block stores descriptive text that explains the intent of the surrounding class `TestApplyEdits`.
**CN:** 该代码块保存说明性文本，用于解释周围class `TestApplyEdits`的设计意图。

### Lines 14-25: Run test: single line match to multiline replacement
```python
    def test_single_line_match_to_multiline_replacement(self) -> None:
        source = "def foo():\n" "    x = compute()\n" "    return x\n"
        edits = [
            EditSpec(
                match="x = compute()",
                replacement="x = compute()\nprint(x)",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "def foo():\n" "    x = compute()\n" "    print(x)\n" "    return x\n"
        )
```
**EN:** This test method exercises single line match to multiline replacement and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 single line match to multiline replacement 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 27-36: Run test: pure insertion
```python
    def test_pure_insertion(self) -> None:
        source = "def foo():\n" "    a = 1\n" "    b = 2\n"
        edits = [
            EditSpec(
                match="a = 1",
                replacement="a = 1\nprint(a)",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == ("def foo():\n" "    a = 1\n" "    print(a)\n" "    b = 2\n")
```
**EN:** This test method exercises pure insertion and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 pure insertion 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 38-47: Run test: pure deletion via empty replacement
```python
    def test_pure_deletion_via_empty_replacement(self) -> None:
        source = "def foo():\n" "    debug_log()\n" "    return 42\n"
        edits = [
            EditSpec(
                match="debug_log()",
                replacement="",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == ("def foo():\n" "    return 42\n")
```
**EN:** This test method exercises pure deletion via empty replacement and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 pure deletion via empty replacement 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 49-58: Run test: deletion fewer lines
```python
    def test_deletion_fewer_lines(self) -> None:
        source = "def foo():\n" "    a = 1\n" "    b = 2\n" "    c = 3\n"
        edits = [
            EditSpec(
                match="a = 1\nb = 2",
                replacement="ab = 3",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == ("def foo():\n" "    ab = 3\n" "    c = 3\n")
```
**EN:** This test method exercises deletion fewer lines and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 deletion fewer lines 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 60-84: Run test: multiline match to multiline replacement
```python
    def test_multiline_match_to_multiline_replacement(self) -> None:
        source = (
            "def foo():\n"
            "    result = self.attn(\n"
            "        q=q,\n"
            "        k=k,\n"
            "    )\n"
            "    return result\n"
        )
        edits = [
            EditSpec(
                match="result = self.attn(\n    q=q,\n    k=k,\n)",
                replacement="result = self.attn(\n    q=q,\n    k=k,\n    v=v,\n)",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "def foo():\n"
            "    result = self.attn(\n"
            "        q=q,\n"
            "        k=k,\n"
            "        v=v,\n"
            "    )\n"
            "    return result\n"
        )
```
**EN:** This test method exercises multiline match to multiline replacement and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiline match to multiline replacement 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 86-108: Run test: indent alignment deep nesting
```python
    def test_indent_alignment_deep_nesting(self) -> None:
        source = (
            "class Foo:\n"
            "    class Bar:\n"
            "        def method(self):\n"
            "            x = compute()\n"
            "            return x\n"
        )
        edits = [
            EditSpec(
                match="x = compute()",
                replacement="x = compute()\nprint(x)",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "class Foo:\n"
            "    class Bar:\n"
            "        def method(self):\n"
            "            x = compute()\n"
            "            print(x)\n"
            "            return x\n"
        )
```
**EN:** This test method exercises indent alignment deep nesting and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 indent alignment deep nesting 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 110-114: Run test: match not found raises
```python
    def test_match_not_found_raises(self) -> None:
        source = "def foo():\n    return 1\n"
        edits = [EditSpec(match="nonexistent_call()", replacement="replaced()")]
        with pytest.raises(PatchApplicationError, match="not found"):
            apply_edits(source=source, edits=edits)
```
**EN:** This test method exercises match not found raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 match not found raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 116-120: Run test: match found multiple times raises
```python
    def test_match_found_multiple_times_raises(self) -> None:
        source = "def foo():\n" "    print(1)\n" "    print(1)\n"
        edits = [EditSpec(match="print(1)", replacement="print(2)")]
        with pytest.raises(PatchApplicationError, match="multiple"):
            apply_edits(source=source, edits=edits)
```
**EN:** This test method exercises match found multiple times raises and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 match found multiple times raises 场景，并验证观测到的行为是否符合预期契约。

### Lines 122-131: Run test: multiple edits applied sequentially
```python
    def test_multiple_edits_applied_sequentially(self) -> None:
        source = "def foo():\n" "    a = 1\n" "    b = 2\n" "    return a + b\n"
        edits = [
            EditSpec(match="a = 1", replacement="a = 10"),
            EditSpec(match="b = 2", replacement="b = 20"),
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "def foo():\n" "    a = 10\n" "    b = 20\n" "    return a + b\n"
        )
```
**EN:** This test method exercises multiple edits applied sequentially and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 multiple edits applied sequentially 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 133-142: Run test: strip matching ignores leading trailing whitespace
```python
    def test_strip_matching_ignores_leading_trailing_whitespace(self) -> None:
        source = "def foo():\n" "    x = compute()\n" "    return x\n"
        edits = [
            EditSpec(
                match="  x = compute()  ",
                replacement="x = replaced()",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == ("def foo():\n" "    x = replaced()\n" "    return x\n")
```
**EN:** This test method exercises strip matching ignores leading trailing whitespace and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 strip matching ignores leading trailing whitespace 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 144-159: Run test: replacement indented text realigned
```python
    def test_replacement_indented_text_realigned(self) -> None:
        """replacement text with its own indentation gets realigned to match source."""
        source = "def foo():\n" "        x = compute()\n" "        return x\n"
        edits = [
            EditSpec(
                match="x = compute()",
                replacement="x = compute()\nprint(x)",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "def foo():\n"
            "        x = compute()\n"
            "        print(x)\n"
            "        return x\n"
        )
```
**EN:** This test method exercises replacement indented text realigned and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 replacement indented text realigned 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 161-178: Run test: replacement with existing indent realigned
```python
    def test_replacement_with_existing_indent_realigned(self) -> None:
        """replacement text already has indentation that should be rebased."""
        source = "def foo():\n" "    if True:\n" "        x = 1\n" "        return x\n"
        edits = [
            EditSpec(
                match="x = 1",
                replacement="x = 1\nif x > 0:\n    print(x)",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "def foo():\n"
            "    if True:\n"
            "        x = 1\n"
            "        if x > 0:\n"
            "            print(x)\n"
            "        return x\n"
        )
```
**EN:** This test method exercises replacement with existing indent realigned and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 replacement with existing indent realigned 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 180-186: Run test: append keeps match and adds after
```python
    def test_append_keeps_match_and_adds_after(self) -> None:
        source = "def foo():\n" "    x = compute()\n" "    return x\n"
        edits = [EditSpec(match="x = compute()", append="print(x)")]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "def foo():\n" "    x = compute()\n" "    print(x)\n" "    return x\n"
        )
```
**EN:** This test method exercises append keeps match and adds after and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 append keeps match and adds after 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 188-212: Run test: append multiline match
```python
    def test_append_multiline_match(self) -> None:
        source = (
            "def foo():\n"
            "    result = call(\n"
            "        a=1,\n"
            "        b=2,\n"
            "    )\n"
            "    return result\n"
        )
        edits = [
            EditSpec(
                match="result = call(\n    a=1,\n    b=2,\n)",
                append="dumper.dump('result', result)",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "def foo():\n"
            "    result = call(\n"
            "        a=1,\n"
            "        b=2,\n"
            "    )\n"
            "    dumper.dump('result', result)\n"
            "    return result\n"
        )
```
**EN:** This test method exercises append multiline match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 append multiline match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 214-223: Run test: prepend adds before match
```python
    def test_prepend_adds_before_match(self) -> None:
        source = "def foo():\n" "    x = compute()\n" "    return x\n"
        edits = [EditSpec(match="x = compute()", prepend="print('before')")]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "def foo():\n"
            "    print('before')\n"
            "    x = compute()\n"
            "    return x\n"
        )
```
**EN:** This test method exercises prepend adds before match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prepend adds before match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 225-229: Run test: prepend multiline
```python
    def test_prepend_multiline(self) -> None:
        source = "def foo():\n" "    return x\n"
        edits = [EditSpec(match="return x", prepend="a = 1\nb = 2")]
        result = apply_edits(source=source, edits=edits)
        assert result == ("def foo():\n" "    a = 1\n" "    b = 2\n" "    return x\n")
```
**EN:** This test method exercises prepend multiline and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prepend multiline 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 231-246: Run test: prepend deep indent
```python
    def test_prepend_deep_indent(self) -> None:
        source = (
            "class Foo:\n"
            "    class Bar:\n"
            "        def method(self):\n"
            "            return x\n"
        )
        edits = [EditSpec(match="return x", prepend="dumper.dump('x', x)")]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "class Foo:\n"
            "    class Bar:\n"
            "        def method(self):\n"
            "            dumper.dump('x', x)\n"
            "            return x\n"
        )
```
**EN:** This test method exercises prepend deep indent and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prepend deep indent 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 248-270: Run test: prepend multiline match
```python
    def test_prepend_multiline_match(self) -> None:
        source = (
            "def foo():\n"
            "    result = call(\n"
            "        a=1,\n"
            "    )\n"
            "    return result\n"
        )
        edits = [
            EditSpec(
                match="result = call(\n    a=1,\n)",
                prepend="dumper.dump('before', x)",
            )
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == (
            "def foo():\n"
            "    dumper.dump('before', x)\n"
            "    result = call(\n"
            "        a=1,\n"
            "    )\n"
            "    return result\n"
        )
```
**EN:** This test method exercises prepend multiline match and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 prepend multiline match 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 272-274: Run test: replacement and append mutually exclusive
```python
    def test_replacement_and_append_mutually_exclusive(self) -> None:
        with pytest.raises(ValidationError, match="only one of"):
            EditSpec(match="x = 1", replacement="x = 2", append="print(x)")
```
**EN:** This test method exercises replacement and append mutually exclusive and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 replacement and append mutually exclusive 场景，并验证观测到的行为是否符合预期契约。

### Lines 276-278: Run test: replacement and prepend mutually exclusive
```python
    def test_replacement_and_prepend_mutually_exclusive(self) -> None:
        with pytest.raises(ValidationError, match="only one of"):
            EditSpec(match="x = 1", replacement="x = 2", prepend="print(x)")
```
**EN:** This test method exercises replacement and prepend mutually exclusive and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 replacement and prepend mutually exclusive 场景，并验证观测到的行为是否符合预期契约。

### Lines 280-282: Run test: prepend and append mutually exclusive
```python
    def test_prepend_and_append_mutually_exclusive(self) -> None:
        with pytest.raises(ValidationError, match="only one of"):
            EditSpec(match="x = 1", prepend="a()", append="b()")
```
**EN:** This test method exercises prepend and append mutually exclusive and verifies that the observed behavior matches the expected contract.
**CN:** 该测试方法会执行 prepend and append mutually exclusive 场景，并验证观测到的行为是否符合预期契约。

### Lines 284-292: Run test: second edit sees result of first
```python
    def test_second_edit_sees_result_of_first(self) -> None:
        """Edits are applied sequentially; second edit matches modified source."""
        source = "def foo():\n" "    x = 1\n" "    return x\n"
        edits = [
            EditSpec(match="x = 1", replacement="x = 1\ny = 2"),
            EditSpec(match="y = 2", replacement="y = 20"),
        ]
        result = apply_edits(source=source, edits=edits)
        assert result == ("def foo():\n" "    x = 1\n" "    y = 20\n" "    return x\n")
```
**EN:** This test method exercises second edit sees result of first and verifies that the observed behavior matches the expected contract. It also checks concrete expectations with assertions.
**CN:** 该测试方法会执行 second edit sees result of first 场景，并验证观测到的行为是否符合预期契约。 其中还会通过断言检查明确的预期。

### Lines 295-298: Expose unittest entrypoint
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
- Internal modules / 内部模块: `sglang.srt.debug_utils.source_patcher.source_editor`, `sglang.srt.debug_utils.source_patcher.types`, `sglang.test.ci.ci_register`
- External and stdlib modules / 外部与标准库模块: `pydantic`, `pytest`, `sys`
- Notable symbols / 关键符号: None / 无
