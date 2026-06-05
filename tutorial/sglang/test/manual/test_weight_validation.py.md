# test_weight_validation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/test_weight_validation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `weight validation` scenario in `test/manual`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual` 中的 `weight validation` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-19: Scenario logic / 场景逻辑
```python
"""
Unit tests for weight validation and cache cleanup logic.

Tests the fix for issue #14754 - ensuring that missing shards do not trigger
entire cache deletion, which can cause race conditions in multi-process scenarios.
"""

import json
import os
import struct
import tempfile
import unittest

from sglang.srt.model_loader.ci_weight_validation import (
    _check_index_files_exist,
    _validate_sharded_model,
)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 20-22: Class definition for TestWeightValidation / 类定义
```python
class TestWeightValidation(unittest.TestCase):
    """Tests for weight validation functions."""
```
**EN:** This range declares `TestWeightValidation`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 23-44: Test routines around test_validate_sharded_model_missing_shard / 测试例程
```python
    def test_validate_sharded_model_missing_shard(self):
        """
        Test that missing shards are detected correctly.

        This is the core test for issue #14754 fix: when a shard is missing,
        the validation should return is_valid=False with an error message
        containing "Missing", but corrupted_files should be empty (indicating
        this is a missing shard issue, not a corruption issue).

        This distinction is critical because:
        - Missing shards: should NOT delete cache (other processes may be using it)
        - Corrupted files: should delete only the corrupted files selectively
        """
        with tempfile.TemporaryDirectory() as tmpdir:
            # Create partial shards (missing shard 3)
            for i in [1, 2]:  # Missing shard 3
                open(
                    os.path.join(tmpdir, f"model-0000{i}-of-00003.safetensors"), "w"
                ).close()

            # Create index file
            index_data = {
```
**EN:** This range defines concrete test routine(s) `test_validate_sharded_model_missing_shard`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `empty`, `cache`, `TemporaryDirectory` and `shards`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 45-47: Scenario logic / 场景逻辑
```python
                "weight_map": {
                    "layer1": "model-00001-of-00003.safetensors",
                    "layer2": "model-00002-of-00003.safetensors",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 48-67: Assertions and result checks / 断言与结果检查
```python
                    "layer3": "model-00003-of-00003.safetensors",
                }
            }
            with open(os.path.join(tmpdir, "model.safetensors.index.json"), "w") as f:
                json.dump(index_data, f)

            weight_files = [
                os.path.join(tmpdir, f"model-0000{i}-of-00003.safetensors")
                for i in [1, 2]
            ]

            is_valid, error_msg, corrupted_files = _validate_sharded_model(
                tmpdir, weight_files
            )

            self.assertFalse(is_valid)
            self.assertIn("Missing", error_msg)
            # CRITICAL: corrupted_files should be empty for missing shards
            # This is what prevents entire cache deletion
            self.assertEqual(corrupted_files, [])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `join`, `dump`, `_validate_sharded_model` and `assertFalse`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 69-69: Test routines around test_validate_sharded_model_all_present / 测试例程
```python
    def test_validate_sharded_model_all_present(self):
```
**EN:** This range defines concrete test routine(s) `test_validate_sharded_model_all_present`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 70-93: Scenario logic / 场景逻辑
```python
        """Test that complete shards pass validation."""
        with tempfile.TemporaryDirectory() as tmpdir:
            # Create all shards with valid safetensors header
            for i in [1, 2, 3]:
                filepath = os.path.join(tmpdir, f"model-0000{i}-of-00003.safetensors")
                # Create a minimal valid safetensors file
                # Header: 8 bytes for header size + JSON header
                header = b'{"__metadata__":{}}'
                header_size = len(header)
                with open(filepath, "wb") as f:
                    f.write(struct.pack("<Q", header_size))
                    f.write(header)

            # Create index file
            index_data = {
                "weight_map": {
                    "layer1": "model-00001-of-00003.safetensors",
                    "layer2": "model-00002-of-00003.safetensors",
                    "layer3": "model-00003-of-00003.safetensors",
                }
            }
            with open(os.path.join(tmpdir, "model.safetensors.index.json"), "w") as f:
                json.dump(index_data, f)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `TemporaryDirectory`, `join`, `write` and `pack`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 94-94: Scenario logic / 场景逻辑
```python
            weight_files = [
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 95-105: Assertions and result checks / 断言与结果检查
```python
                os.path.join(tmpdir, f"model-0000{i}-of-00003.safetensors")
                for i in [1, 2, 3]
            ]

            is_valid, error_msg, corrupted_files = _validate_sharded_model(
                tmpdir, weight_files
            )

            self.assertTrue(is_valid)
            self.assertIsNone(error_msg)
            self.assertEqual(corrupted_files, [])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `join`, `_validate_sharded_model`, `assertTrue` and `assertIsNone`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 107-119: Test routines around test_validate_sharded_model_corrupted_shard / 测试例程
```python
    def test_validate_sharded_model_corrupted_shard(self):
        """
        Test that corrupted shards are detected and returned in corrupted_files.

        This tests the other branch: when a file exists but is corrupted
        (invalid safetensors format), it should be added to corrupted_files
        so that selective cleanup can remove just that file.
        """
        with tempfile.TemporaryDirectory() as tmpdir:
            # Create shard 1 as valid
            filepath1 = os.path.join(tmpdir, "model-00001-of-00003.safetensors")
            header = b'{"__metadata__":{}}'
            with open(filepath1, "wb") as f:
```
**EN:** This range defines concrete test routine(s) `test_validate_sharded_model_corrupted_shard`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `corrupted`, `TemporaryDirectory` and `join`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 120-131: Scenario logic / 场景逻辑
```python
                f.write(struct.pack("<Q", len(header)))
                f.write(header)

            # Create shard 2 as corrupted (invalid header)
            filepath2 = os.path.join(tmpdir, "model-00002-of-00003.safetensors")
            with open(filepath2, "wb") as f:
                f.write(b"invalid data that is not a valid safetensors file")

            # Create shard 3 as valid
            filepath3 = os.path.join(tmpdir, "model-00003-of-00003.safetensors")
            with open(filepath3, "wb") as f:
                f.write(struct.pack("<Q", len(header)))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `write`, `pack`, `corrupted` and `join`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 132-144: Scenario logic / 场景逻辑
```python
                f.write(header)

            # Create index file
            index_data = {
                "weight_map": {
                    "layer1": "model-00001-of-00003.safetensors",
                    "layer2": "model-00002-of-00003.safetensors",
                    "layer3": "model-00003-of-00003.safetensors",
                }
            }
            with open(os.path.join(tmpdir, "model.safetensors.index.json"), "w") as f:
                json.dump(index_data, f)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `write`, `join` and `dump`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 145-155: Assertions and result checks / 断言与结果检查
```python
            weight_files = [filepath1, filepath2, filepath3]

            is_valid, error_msg, corrupted_files = _validate_sharded_model(
                tmpdir, weight_files
            )

            self.assertFalse(is_valid)
            self.assertIn("Corrupt", error_msg)
            # The corrupted file should be identified
            self.assertEqual(len(corrupted_files), 1)
            self.assertIn("model-00002-of-00003.safetensors", corrupted_files[0])
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `_validate_sharded_model`, `assertFalse`, `assertIn` and `assertEqual`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 157-169: Test routines around test_broken_index_symlink_detected / 测试例程
```python
    def test_broken_index_symlink_detected(self):
        """
        Test that broken index symlinks are detected and cause validation to fail.

        When an index file is a symlink pointing to a non-existent blob,
        validation should fail (to trigger re-download) rather than silently
        continuing and causing timeout during actual loading.
        """
        with tempfile.TemporaryDirectory() as tmpdir:
            # Create a broken symlink for the index file
            index_path = os.path.join(tmpdir, "model.safetensors.index.json")
            non_existent_blob = os.path.join(tmpdir, "blobs", "nonexistent_hash")
            os.symlink(non_existent_blob, index_path)
```
**EN:** This range defines concrete test routine(s) `test_broken_index_symlink_detected`. The logic drives the target scenario and encodes the expected acceptance criteria. Representative call sites include `fail`, `TemporaryDirectory`, `join` and `symlink`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 170-181: Assertions and result checks / 断言与结果检查
```python

            # Verify it's a broken symlink
            self.assertTrue(os.path.islink(index_path))
            self.assertFalse(os.path.exists(index_path))

            # Check should fail for broken symlink
            is_valid, error_msg = _check_index_files_exist(tmpdir)

            self.assertFalse(is_valid)
            self.assertIn("Broken", error_msg)
            # The broken symlink should have been cleaned up
            self.assertFalse(os.path.exists(index_path))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertTrue`, `islink`, `assertFalse` and `exists`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 182-182: Assertions and result checks / 断言与结果检查
```python
            self.assertFalse(os.path.islink(index_path))
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `assertFalse` and `islink`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 183-186: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Prefix caching / 前缀缓存
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `os`, `struct`, `tempfile`, `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.model_loader.ci_weight_validation`
