# test_complement.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `test/python/pycute/test_complement.py`
- **EN:** This module tests `pycute.complement()` by verifying that the original layout and its complement have disjoint codomains except for the shared zero location.
- **CN:** 该模块通过验证原布局与其 `complement()` 结果的值域互斥（除了共享的零位置）来测试 `pycute.complement()`。

## Line-by-Line Analysis / 逐行分析

### Lines 33-42

```python
"""
Unit tests for pycute.complement
"""

import logging
import unittest

from pycute import *

_LOGGER = logging.getLogger(__name__)
```

**EN:** The file sets up logging, unittest, and wildcard-imported PyCuTe helpers, matching the style of the other small PyCuTe property-test modules.

**CN:** 文件先设置 logging、unittest，以及通过通配符导入的 PyCuTe 辅助接口，整体风格与其他小型 PyCuTe 性质测试模块保持一致。

### Lines 45-55

```python
class TestComplement(unittest.TestCase):
  def helper_test_complement(self, layout):
    layoutR = complement(layout)

    _LOGGER.debug(f"{layout}  =>  {layoutR}")

    # Post-condition: test disjointness of the codomains
    for a in range(size(layout)):
      for b in range(size(layoutR)):
        assert (layout(a) != layoutR(b)) or (layout(a) == 0 and layoutR(b) == 0)

```

**EN:** `helper_test_complement()` computes the complement layout, logs the transformation, and then performs a nested-loop codomain check. For every coordinate in the original layout and every coordinate in the complement, the mapped values must differ unless both are exactly zero.

**CN:** `helper_test_complement()` 会先求出补布局，记录变换结果，然后通过双层循环检查值域关系：对原布局中的每个坐标和补布局中的每个坐标，映射值都必须不同，除非两者都恰好为零。

### Lines 56-89

```python
  def test_complement(self):
    test = Layout(1,0)
    self.helper_test_complement(test)

    test = Layout(1,1)
    self.helper_test_complement(test)

    test = Layout(4,0)
    self.helper_test_complement(test)

    test = Layout((2,4),(1,2))
    self.helper_test_complement(test)

    test = Layout((2,3),(1,2))
    self.helper_test_complement(test)

    test = Layout((2,4),(1,4))
    self.helper_test_complement(test)

    test = Layout((2,4,8),(8,1,64))
    self.helper_test_complement(test)

    test = Layout(((2,2),(2,2)),((1,4),(8,32)))
    self.helper_test_complement(test)

    test = Layout((2,(3,4)),(3,(1,6)))
    self.helper_test_complement(test)

    test = Layout((4,6),(1,6))
    self.helper_test_complement(test)

    test = Layout((4,10),(1,10))
    self.helper_test_complement(test)

```

**EN:** `test_complement()` enumerates a diverse set of shapes and strides, including degenerate stride-zero layouts, simple 2D cases, deeper tuples, and nested tuple structures. Each case is passed through the same disjointness property checker.

**CN:** `test_complement()` 枚举了多种形状与步幅组合，包括退化的零步幅布局、简单二维情况、更深的元组，以及嵌套元组结构。每个样本都复用同一个互斥性质检查器。

### Lines 91-92

```python
if __name__ == "__main__":
  unittest.main()
```

**EN:** The standalone unittest entry point mirrors the rest of the PyCuTe test files.

**CN:** 独立的 unittest 入口与其余 PyCuTe 测试文件保持一致。

## Key Concepts / 关键概念
- **EN:** The test is about codomain separation rather than direct structural equality.
  **CN:** 该测试关注的是值域分离，而不是结构上的直接相等。
- **EN:** A special case is carved out for zero, which both layouts may legitimately map to.
  **CN:** 零值被单独视为特例，因为两个布局都可能合法地映射到它。
- **EN:** The property is checked extensionally by iterating over all valid coordinates.
  **CN:** 该性质通过遍历所有合法坐标进行外延式验证。

## Dependencies / 依赖关系
- **EN:** Standard library: `logging`, `unittest`.
  **CN:** 标准库：`logging`、`unittest`。
- **EN:** PyCuTe API imported via wildcard, especially `Layout`, `complement`, and `size`.
  **CN:** 通过通配符导入的 PyCuTe API，尤其是 `Layout`、`complement` 与 `size`。
