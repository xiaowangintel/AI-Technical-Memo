# filter-failed.py — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/utils/lit/tests/filter-failed.py` | `llvm/utils/lit/tests/filter-failed.py` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | Checks that --filter-failed only runs tests that previously failed. | 实现 LLVM 的 lit 测试基础设施、运行器、辅助工具或测试夹具。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-12

````python
# Checks that --filter-failed only runs tests that previously failed.

# RUN: rm -rf %t
# RUN: cp -rL %{inputs}%{fs-sep}filter-failed %t
#
# RUN: not %{lit} %t
#
# RUN: echo "RUN: false" > %t%{fs-sep}new-fail.txt
# RUN: echo "RUN: true"  > %t%{fs-sep}new-pass.txt
#
# RUN: not %{lit} --filter-failed %t | FileCheck %s

````
- **L1 EN**: Comment documents nearby script behavior: `Checks that --filter-failed only runs tests that previously failed.`.
  **L1 CN**: 注释说明了附近脚本逻辑：`Checks that --filter-failed only runs tests that previously failed.`。
- **L2 EN**: Blank line separates nearby declarations or logic blocks.
  **L2 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3 EN**: Comment documents nearby script behavior: `RUN: rm -rf %t`.
  **L3 CN**: 注释说明了附近脚本逻辑：`RUN: rm -rf %t`。
- **L4 EN**: Comment documents nearby script behavior: `RUN: cp -rL %{inputs}%{fs-sep}filter-failed %t`.
  **L4 CN**: 注释说明了附近脚本逻辑：`RUN: cp -rL %{inputs}%{fs-sep}filter-failed %t`。
- **L5 EN**: Comment documents nearby script behavior: ``.
  **L5 CN**: 注释说明了附近脚本逻辑：``。
- **L6 EN**: Comment documents nearby script behavior: `RUN: not %{lit} %t`.
  **L6 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} %t`。
- **L7 EN**: Comment documents nearby script behavior: ``.
  **L7 CN**: 注释说明了附近脚本逻辑：``。
- **L8 EN**: Comment documents nearby script behavior: `RUN: echo "RUN: false" > %t%{fs-sep}new-fail.txt`.
  **L8 CN**: 注释说明了附近脚本逻辑：`RUN: echo "RUN: false" > %t%{fs-sep}new-fail.txt`。
- **L9 EN**: Comment documents nearby script behavior: `RUN: echo "RUN: true" > %t%{fs-sep}new-pass.txt`.
  **L9 CN**: 注释说明了附近脚本逻辑：`RUN: echo "RUN: true" > %t%{fs-sep}new-pass.txt`。
- **L10 EN**: Comment documents nearby script behavior: ``.
  **L10 CN**: 注释说明了附近脚本逻辑：``。
- **L11 EN**: Comment documents nearby script behavior: `RUN: not %{lit} --filter-failed %t | FileCheck %s`.
  **L11 CN**: 注释说明了附近脚本逻辑：`RUN: not %{lit} --filter-failed %t | FileCheck %s`。
- **L12 EN**: Blank line separates nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 13-23

````python
# CHECK: Testing: 3 of 7 tests
# CHECK-DAG: FAIL: filter-failed :: fail.txt
# CHECK-DAG: UNRESOLVED: filter-failed :: unresolved.txt
# CHECK-DAG: XPASS: filter-failed :: xpass.txt

# CHECK: Testing Time:
# CHECK: Total Discovered Tests:
# CHECK-NEXT:   Excluded : 4 {{\([0-9]*\.[0-9]*%\)}}
# CHECK-NEXT:   Unresolved : 1 {{\([0-9]*\.[0-9]*%\)}}
# CHECK-NEXT:   Failed : 1 {{\([0-9]*\.[0-9]*%\)}}
# CHECK-NEXT:   Unexpectedly Passed: 1 {{\([0-9]*\.[0-9]*%\)}}
````
- **L13 EN**: Comment documents nearby script behavior: `CHECK: Testing: 3 of 7 tests`.
  **L13 CN**: 注释说明了附近脚本逻辑：`CHECK: Testing: 3 of 7 tests`。
- **L14 EN**: Comment documents nearby script behavior: `CHECK-DAG: FAIL: filter-failed :: fail.txt`.
  **L14 CN**: 注释说明了附近脚本逻辑：`CHECK-DAG: FAIL: filter-failed :: fail.txt`。
- **L15 EN**: Comment documents nearby script behavior: `CHECK-DAG: UNRESOLVED: filter-failed :: unresolved.txt`.
  **L15 CN**: 注释说明了附近脚本逻辑：`CHECK-DAG: UNRESOLVED: filter-failed :: unresolved.txt`。
- **L16 EN**: Comment documents nearby script behavior: `CHECK-DAG: XPASS: filter-failed :: xpass.txt`.
  **L16 CN**: 注释说明了附近脚本逻辑：`CHECK-DAG: XPASS: filter-failed :: xpass.txt`。
- **L17 EN**: Blank line separates nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L18 EN**: Comment documents nearby script behavior: `CHECK: Testing Time:`.
  **L18 CN**: 注释说明了附近脚本逻辑：`CHECK: Testing Time:`。
- **L19 EN**: Comment documents nearby script behavior: `CHECK: Total Discovered Tests:`.
  **L19 CN**: 注释说明了附近脚本逻辑：`CHECK: Total Discovered Tests:`。
- **L20 EN**: Comment documents nearby script behavior: `CHECK-NEXT: Excluded : 4 {{\([0-9]*\.[0-9]*%\)}}`.
  **L20 CN**: 注释说明了附近脚本逻辑：`CHECK-NEXT: Excluded : 4 {{\([0-9]*\.[0-9]*%\)}}`。
- **L21 EN**: Comment documents nearby script behavior: `CHECK-NEXT: Unresolved : 1 {{\([0-9]*\.[0-9]*%\)}}`.
  **L21 CN**: 注释说明了附近脚本逻辑：`CHECK-NEXT: Unresolved : 1 {{\([0-9]*\.[0-9]*%\)}}`。
- **L22 EN**: Comment documents nearby script behavior: `CHECK-NEXT: Failed : 1 {{\([0-9]*\.[0-9]*%\)}}`.
  **L22 CN**: 注释说明了附近脚本逻辑：`CHECK-NEXT: Failed : 1 {{\([0-9]*\.[0-9]*%\)}}`。
- **L23 EN**: Comment documents nearby script behavior: `CHECK-NEXT: Unexpectedly Passed: 1 {{\([0-9]*\.[0-9]*%\)}}`.
  **L23 CN**: 注释说明了附近脚本逻辑：`CHECK-NEXT: Unexpectedly Passed: 1 {{\([0-9]*\.[0-9]*%\)}}`。

## Key Concepts / 关键概念

- EN: lit test runner infrastructure
  - CN: lit 测试运行基础设施
- EN: pattern-based output verification
  - CN: 基于模式的输出校验
- EN: test harness behavior
  - CN: 测试框架行为
- EN: Python utility scripting
  - CN: Python 工具脚本

## Dependencies / 依赖关系

- EN: No direct include/import dependencies are declared in this file.
  - CN: 该文件中没有声明直接的 include/import 依赖。
