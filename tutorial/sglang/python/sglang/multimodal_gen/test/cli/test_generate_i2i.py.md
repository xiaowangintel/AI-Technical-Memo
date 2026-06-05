# test_generate_i2i.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/cli/test_generate_i2i.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates generate i2i with focused assertions and fixtures. Key symbols include `TestQwenImageEditI2I`. / 该测试模块通过有针对性的断言与夹具，验证 generate i2i 的实现。 关键符号包括 `TestQwenImageEditI2I`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: Imports and module setup / 导入与模块初始化
```python
import os
import unittest

from PIL import Image

from sglang.multimodal_gen.configs.sample.sampling_params import DataType
from sglang.multimodal_gen.test.cli.test_generate_common import CLIBase, run_command
from sglang.multimodal_gen.test.test_utils import (
    DEFAULT_QWEN_IMAGE_EDIT_2511_MODEL_NAME_FOR_TEST,
    check_image_size,
)
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 14-125: Class `TestQwenImageEditI2I` / 类 `TestQwenImageEditI2I`
```python
class TestQwenImageEditI2I(CLIBase):
    model_path: str = DEFAULT_QWEN_IMAGE_EDIT_2511_MODEL_NAME_FOR_TEST
    data_type: DataType = DataType.IMAGE
    width: int = 512
    height: int = 512

    test_image_urls = [
        "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-Image/edit2509/edit2509_1.jpg",
        "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen-Image/edit2509/edit2509_2.jpg",
    ]

    def get_base_command(self):
        return [
            "sglang",
# ...

        succeed = run_command(command)
        self.assertTrue(succeed, f"{name} command failed")
        self.verify_multi_output(name, 2)
```
**EN:** This class models `TestQwenImageEditI2I` as a specialization of `CLIBase`. Important methods include `get_base_command`, `verify_multi_output`, `test_single_prompt_single_image`, `test_single_prompt_multi_image`.
**CN:** 该类实现 `TestQwenImageEditI2I`，并继承/扩展 `CLIBase`。 其中较重要的方法包括 `get_base_command`, `verify_multi_output`, `test_single_prompt_single_image`, `test_single_prompt_multi_image`。

### Lines 126-132: Top-level configuration / 顶层配置
```python


del CLIBase


if __name__ == "__main__":
    unittest.main()
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- Sampling parameter control / 采样参数控制
- Automated verification / 自动化验证
- Command-line interface / 命令行接口
- Image generation flow / 图像生成流程
- Symbol `TestQwenImageEditI2I` anchors the module API / 符号 `TestQwenImageEditI2I` 构成该模块的核心 API

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.configs.sample.sampling_params`, `sglang.multimodal_gen.test.cli.test_generate_common`, `sglang.multimodal_gen.test.test_utils`
- **External / 外部**: `unittest`, `PIL`
- **Stdlib / 标准库**: `os`
