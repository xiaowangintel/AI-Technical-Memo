# test_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/release/test_utils.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `test_utils` workflow in SGLang. It mainly handles test execution, release automation. / 该Python 模块用于支撑 SGLang 中的 `test_utils` 流程，主要负责测试执行、发布自动化。它属于 `release` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-2: File header and usage notes / 文件头与使用说明
```python
#!/usr/bin/env python3
```
**EN:** These opening lines establish the file context, such as the shebang, encoding note, or top-level comments that tell operators how the script should be used.
**CN:** 这些起始行用于建立文件上下文，例如 shebang、编码说明或顶层注释，帮助使用者理解脚本的执行方式。

### Lines 3-6: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import unittest
from pathlib import Path

from utils import compare_versions, normalize_version, parse_version, validate_version
```
**EN:** This block loads unittest, pathlib, utils. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 unittest, pathlib, utils。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 9-155: Declares the `TestVersionUtils` class / 声明 `TestVersionUtils` 类
```python
class TestVersionUtils(unittest.TestCase):
    def test_normalize_version(self):
        """Test version normalization removes 'v' prefix."""
        self.assertEqual(normalize_version("v0.5.3"), "0.5.3")
        self.assertEqual(normalize_version("0.5.3"), "0.5.3")
        self.assertEqual(normalize_version("v0.5.3rc0"), "0.5.3rc0")
        self.assertEqual(normalize_version("0.5.3.post1"), "0.5.3.post1")

    def test_validate_version(self):
        """Test version format validation."""
        # Valid formats
        self.assertTrue(validate_version("0.5.3"))
        self.assertTrue(validate_version("0.5.3rc0"))
        self.assertTrue(validate_version("0.5.3rc1"))
        self.assertTrue(validate_version("0.5.3rc999"))
        self.assertTrue(validate_version("0.5.3.post1"))
        self.assertTrue(validate_version("0.5.3.post10"))
        self.assertTrue(validate_version("1.2.3"))
        self.assertTrue(validate_version("10.20.30"))

        # Invalid formats
        self.assertFalse(validate_version("0.5"))
        self.assertFalse(validate_version("0.5.3."))
        self.assertFalse(validate_version("0.5.3rc"))
        self.assertFalse(validate_version("0.5.3post1"))
        self.assertFalse(validate_version("0.5.3-rc0"))
        self.assertFalse(validate_version("v0.5.3"))
        self.assertFalse(validate_version("0.5.3beta1"))
        self.assertFalse(validate_version("0.5.3.rc0"))

    def test_parse_version_stable(self):
        """Test parsing stable version."""
        self.assertEqual(parse_version("0.5.3"), (0, 5, 3, 0, 0))
        self.assertEqual(parse_version("1.2.3"), (1, 2, 3, 0, 0))
        self.assertEqual(parse_version("10.20.30"), (10, 20, 30, 0, 0))

    def test_parse_version_rc(self):
        """Test parsing release candidate versions."""
        self.assertEqual(parse_version("0.5.3rc0"), (0, 5, 3, -1000, 0))
        self.assertEqual(parse_version("0.5.3rc1"), (0, 5, 3, -999, 0))
        self.assertEqual(parse_version("0.5.3rc2"), (0, 5, 3, -998, 0))
        self.assertEqual(parse_version("0.5.3rc10"), (0, 5, 3, -990, 0))

    def test_parse_version_post(self):
        """Test parsing post-release versions."""
        self.assertEqual(parse_version("0.5.3.post1"), (0, 5, 3, 0, 1))
        self.assertEqual(parse_version("0.5.3.post2"), (0, 5, 3, 0, 2))
        self.assertEqual(parse_version("0.5.3.post10"), (0, 5, 3, 0, 10))

    def test_parse_version_invalid(self):
        """Test parsing invalid versions raises error."""
        with self.assertRaises(ValueError):
            parse_version("0.5")
        with self.assertRaises(ValueError):
            parse_version("invalid")
        with self.assertRaises(ValueError):
            parse_version("v0.5.3")

    def test_compare_versions_equal(self):
        """Test comparing equal versions."""
        self.assertEqual(compare_versions("0.5.3", "0.5.3"), 0)
        self.assertEqual(compare_versions("0.5.3rc0", "0.5.3rc0"), 0)
        self.assertEqual(compare_versions("0.5.3.post1", "0.5.3.post1"), 0)

    def test_compare_versions_rc_ordering(self):
        """Test release candidate ordering: rc0 < rc1 < rc2 < stable."""
        # rc0 < rc1
        self.assertEqual(compare_versions("0.5.3rc0", "0.5.3rc1"), -1)
        self.assertEqual(compare_versions("0.5.3rc1", "0.5.3rc0"), 1)

        # rc1 < rc2
        self.assertEqual(compare_versions("0.5.3rc1", "0.5.3rc2"), -1)
        self.assertEqual(compare_versions("0.5.3rc2", "0.5.3rc1"), 1)

        # rc < stable
        self.assertEqual(compare_versions("0.5.3rc0", "0.5.3"), -1)
        self.assertEqual(compare_versions("0.5.3rc1", "0.5.3"), -1)
        self.assertEqual(compare_versions("0.5.3", "0.5.3rc0"), 1)

    def test_compare_versions_post_ordering(self):
        """Test post-release ordering: stable < post1 < post2."""
        # stable < post1
        self.assertEqual(compare_versions("0.5.3", "0.5.3.post1"), -1)
        self.assertEqual(compare_versions("0.5.3.post1", "0.5.3"), 1)

        # post1 < post2
        self.assertEqual(compare_versions("0.5.3.post1", "0.5.3.post2"), -1)
        self.assertEqual(compare_versions("0.5.3.post2", "0.5.3.post1"), 1)

    def test_compare_versions_full_ordering(self):
        """Test complete version ordering: rc < stable < post."""
        # rc < stable < post
        self.assertEqual(compare_versions("0.5.3rc0", "0.5.3"), -1)
        self.assertEqual(compare_versions("0.5.3", "0.5.3.post1"), -1)
        self.assertEqual(compare_versions("0.5.3rc0", "0.5.3.post1"), -1)

        # Verify transitivity: rc0 < rc1 < stable < post1 < post2
        versions = [
            "0.5.3rc0",
            "0.5.3rc1",
            "0.5.3",
            "0.5.3.post1",
            "0.5.3.post2",
        ]
        for i in range(len(versions) - 1):
            self.assertEqual(
                compare_versions(versions[i], versions[i + 1]),
                -1,
                f"{versions[i]} should be less than {versions[i + 1]}",
            )

    def test_compare_versions_different_patch(self):
        """Test comparing versions with different patch numbers."""
        # 0.5.3 < 0.5.4
        self.assertEqual(compare_versions("0.5.3", "0.5.4"), -1)
        self.assertEqual(compare_versions("0.5.4", "0.5.3"), 1)

        # rc of higher patch > stable of lower patch
        self.assertEqual(compare_versions("0.5.4rc0", "0.5.3"), 1)
        self.assertEqual(compare_versions("0.5.3.post1", "0.5.4rc0"), -1)

    def test_compare_versions_different_minor(self):
        """Test comparing versions with different minor numbers."""
        self.assertEqual(compare_versions("0.4.9", "0.5.0"), -1)
        self.assertEqual(compare_versions("0.5.0", "0.4.9"), 1)

    def test_compare_versions_different_major(self):
        """Test comparing versions with different major numbers."""
        self.assertEqual(compare_versions("0.9.9", "1.0.0"), -1)
        self.assertEqual(compare_versions("1.0.0", "0.9.9"), 1)

    def test_real_world_scenarios(self):
        """Test real-world version bump scenarios."""
        # Scenario 1: RC progression
        self.assertEqual(compare_versions("0.5.3rc0", "0.5.3rc1"), -1)

        # Scenario 2: RC to stable release
        self.assertEqual(compare_versions("0.5.3rc2", "0.5.3"), -1)

        # Scenario 3: Stable to post-release hotfix
        self.assertEqual(compare_versions("0.5.3", "0.5.3.post1"), -1)

        # Scenario 4: Post-release to next RC
        self.assertEqual(compare_versions("0.5.3.post1", "0.5.4rc0"), -1)

        # Scenario 5: Next stable version
        self.assertEqual(compare_versions("0.5.3", "0.5.4"), -1)
```
**EN:** This block declares the `TestVersionUtils` class. It extends unittest.TestCase. The class groups 14 method(s), including test_normalize_version, test_validate_version, test_parse_version_stable, test_parse_version_rc, test_parse_version_post.
**CN:** 该代码块声明了 `TestVersionUtils` 类。它继承自 unittest.TestCase。该类集中定义了 14 个方法，包括 test_normalize_version, test_validate_version, test_parse_version_stable, test_parse_version_rc, test_parse_version_post。

### Lines 158-159: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It hands control to the previously defined helpers.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会把控制流交给前面定义的辅助函数。

## Key Concepts / 关键概念
- **Filesystem coordination** / 文件系统协同
- **Environment management** / 环境管理

## Dependencies / 依赖关系
- **Standard library / 标准库**: `pathlib`, `unittest`
- **Third-party modules / 第三方模块**: `utils`
