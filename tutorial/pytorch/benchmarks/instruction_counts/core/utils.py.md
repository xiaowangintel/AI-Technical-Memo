# utils.py — Code Analysis / 代码分析

## Source / 来源
- File: `benchmarks/instruction_counts/core/utils.py`
- Repository: `pytorch`
- Purpose (EN): Implements benchmark definitions, helpers, or runners for performance measurement.
- 用途 (CN): 实现用于性能测量的基准定义、辅助工具或运行器。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````python
 1: # mypy: ignore-errors
 2: import atexit
 3: import re
 4: import shutil
 5: import textwrap
 6: 
 7: from core.api import GroupedBenchmark, TimerArgs
 8: from core.types import Definition, FlatIntermediateDefinition, Label
 9: 
10: from torch.utils.benchmark.utils.common import _make_temp_dir
11: 
12: 
13: _TEMPDIR: str | None = None
14: 
15: 
16: def get_temp_dir() -> str:
17:     global _TEMPDIR
18:     if _TEMPDIR is None:
19:         _TEMPDIR = _make_temp_dir(
20:             prefix="instruction_count_microbenchmarks", gc_dev_shm=True
21:         )
22:         atexit.register(shutil.rmtree, path=_TEMPDIR)
23:     return _TEMPDIR
24: 
25: 
26: def _flatten(
27:     key_prefix: Label, sub_schema: Definition, result: FlatIntermediateDefinition
28: ) -> None:
29:     for k, value in sub_schema.items():
30:         if isinstance(k, tuple):
31:             if not all(isinstance(ki, str) for ki in k):
32:                 raise AssertionError(
33:                     f"expected all elements of key tuple to be str, got {k}"
34:                 )
35:             key_suffix: Label = k
36:         elif k is None:
37:             key_suffix = ()
38:         else:
39:             if not isinstance(k, str):
40:                 raise AssertionError(f"expected key to be str, got {type(k)}")
````
- EN: Handles module imports such as `atexit`, `re`, `shutil`, `textwrap`.
- CN: 处理模块导入，例如 `atexit`, `re`, `shutil`, `textwrap`。
- EN: Implements callable logic such as `get_temp_dir`, `_flatten`.
- CN: 实现可调用逻辑，例如 `get_temp_dir`, `_flatten`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````python
41:             key_suffix = (k,)
42: 
43:         key: Label = key_prefix + key_suffix
44:         if isinstance(value, (TimerArgs, GroupedBenchmark)):
45:             if key in result:
46:                 raise AssertionError(f"duplicate key: {key}")
47:             result[key] = value
48:         else:
49:             if not isinstance(value, dict):
50:                 raise AssertionError(f"expected value to be dict, got {type(value)}")
51:             _flatten(key_prefix=key, sub_schema=value, result=result)
52: 
53: 
54: def flatten(schema: Definition) -> FlatIntermediateDefinition:
55:     """See types.py for an explanation of nested vs. flat definitions."""
56:     result: FlatIntermediateDefinition = {}
57:     _flatten(key_prefix=(), sub_schema=schema, result=result)
58: 
59:     # Ensure that we produced a valid flat definition.
60:     for k, v in result.items():
61:         if not isinstance(k, tuple):
62:             raise AssertionError(f"expected key to be tuple, got {type(k)}")
63:         if not all(isinstance(ki, str) for ki in k):
64:             raise AssertionError(
65:                 f"expected all elements of key tuple to be str, got {k}"
66:             )
67:         if not isinstance(v, (TimerArgs, GroupedBenchmark)):
68:             raise AssertionError(
69:                 f"expected value to be TimerArgs or GroupedBenchmark, got {type(v)}"
70:             )
71:     return result
72: 
73: 
74: def parse_stmts(stmts: str) -> tuple[str, str]:
75:     """Helper function for side-by-side Python and C++ stmts.
76: 
77:     For more complex statements, it can be useful to see Python and C++ code
78:     side by side. To this end, we provide an **extremely restricted** way
79:     to define Python and C++ code side-by-side. The schema should be mostly
80:     self explanatory, with the following non-obvious caveats:
````
- EN: Implements callable logic such as `flatten`, `parse_stmts`.
- CN: 实现可调用逻辑，例如 `flatten`, `parse_stmts`。
- EN: Contains benchmark setup, registration, or performance-oriented execution paths.
- CN: 包含基准测试设置、注册或面向性能的执行路径。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````python
 81:       - Width for the left (Python) column MUST be 40 characters.
 82:       - The column separator is " | ", not "|". Whitespace matters.
 83:     """
 84:     stmts = textwrap.dedent(stmts).strip()
 85:     lines: list[str] = stmts.splitlines(keepends=False)
 86:     if len(lines) < 3:
 87:         raise AssertionError(f"Invalid string (expected at least 3 lines):\n{stmts}")
 88: 
 89:     column_header_pattern = r"^Python\s{35}\| C\+\+(\s*)$"
 90:     signature_pattern = r"^: f\((.*)\)( -> (.+))?\s*$"  # noqa: F841
 91:     separation_pattern = r"^[-]{40} | [-]{40}$"
 92:     code_pattern = r"^(.{40}) \|($| (.*)$)"
 93: 
 94:     column_match = re.search(column_header_pattern, lines[0])
 95:     if column_match is None:
 96:         raise ValueError(
 97:             f"Column header `{lines[0]}` "
 98:             f"does not match pattern `{column_header_pattern}`"
 99:         )
100: 
101:     if not re.search(separation_pattern, lines[1]):
102:         raise AssertionError(
103:             f"Separation line `{lines[1]}` does not match pattern `{separation_pattern}`"
104:         )
105: 
106:     py_lines: list[str] = []
107:     cpp_lines: list[str] = []
108:     for l in lines[2:]:
109:         l_match = re.search(code_pattern, l)
110:         if l_match is None:
111:             raise ValueError(f"Invalid line `{l}`")
112:         py_lines.append(l_match.groups()[0])
113:         cpp_lines.append(l_match.groups()[2] or "")
114: 
115:         # Make sure we can round trip for correctness.
116:         l_from_stmts = f"{py_lines[-1]:<40} | {cpp_lines[-1]:<40}".rstrip()
117:         if l_from_stmts != l.rstrip():
118:             raise AssertionError(f"Failed to round trip `{l}`")
119: 
120:     return "\n".join(py_lines), "\n".join(cpp_lines)
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Performance benchmarking / 性能基准测试
- Symbol `get_temp_dir` / 符号 `get_temp_dir`
- Symbol `_flatten` / 符号 `_flatten`
- Symbol `flatten` / 符号 `flatten`
- Symbol `parse_stmts` / 符号 `parse_stmts`

## Dependencies / 依赖关系
- Python imports: `atexit`, `re`, `shutil`, `textwrap`, `core.api`, `core.types`, `torch.utils.benchmark.utils.common`
- Python 导入: `atexit`, `re`, `shutil`, `textwrap`, `core.api`, `core.types`, `torch.utils.benchmark.utils.common`
