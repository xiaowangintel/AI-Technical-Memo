# test_log_parser.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/debug_utils/test_log_parser.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `log parser` scenario in `test/manual/debug_utils`. It uses SGLang's shared test infrastructure to configure models or services and verify expected performance behavior. / 该手动测试模块覆盖 `test/manual/debug_utils` 中的 `log parser` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的性能表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: Imports and shared helpers / 导入与共享辅助项
```python
import json
import unittest

from sglang.srt.debug_utils import log_parser
from sglang.test.test_utils import CustomTestCase
```
**EN:** This range imports `json`, `unittest`, `sglang.srt.debug_utils` and `sglang.test.test_utils`. These dependencies provide the test harness, runtime clients, and reusable SGLang helpers used later in the file.
**CN:** 这一部分导入所需模块，为后续测试框架、运行时客户端以及可复用的 SGLang 辅助逻辑提供依赖。

### Lines 8-8: Class definition for TestLogParser / 类定义
```python
class TestLogParser(CustomTestCase):
```
**EN:** This range declares `TestLogParser`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 9-24: Test routines around test_log_parser / 测试例程
```python
    def test_log_parser(self):
        lines = """
(SGLangEngine pid=35555) [2025-10-31 03:45:20 TP0] Decode batch [51341], #running-req: 317, #token: 1094261, token usage: 0.67, cuda graph: True, gen throughput (token/s): 14806.57, #queue-req: 0,
(SGLangEngine pid=111711, ip=10.15.36.1) [2025-10-31 03:45:20 TP0] Decode batch [39913], #running-req: 78, #token: 432100, token usage: 0.27, cuda graph: True, gen throughput (token/s): 7269.16, #queue-req: 0,
[2025-11-03 14:31:10 DP6 TP6 EP6] Decode batch, #running-req: 251, #token: 2811200, token usage: 1.00, cuda graph: True, gen throughput (token/s): 2055.94, #queue-req: 655,
"""
        expect_rows = json.loads(
            """[{"line":"(SGLangEngine pid=35555) [2025-10-31 03:45:20 TP0] Decode batch [51341], #running-req: 317, #token: 1094261, token usage: 0.67, cuda graph: True, gen throughput (token/s): 14806.57, #queue-req: 0,","1":"(SGLangEngine pid=35555)","pid":35555,"ip":null,"time":"2025-10-31 03:45:20","dp_rank":null,"tp_rank":0,"ep_rank":null,"pp_rank":null,"9":" [51341]","num_running_req":317,"num_token":1094261,"token_usage":0.67,"gen_throughput":14806.57,"queue_req":0},{"line":"(SGLangEngine pid=111711, ip=10.15.36.1) [2025-10-31 03:45:20 TP0] Decode batch [39913], #running-req: 78, #token: 432100, token usage: 0.27, cuda graph: True, gen throughput (token/s): 7269.16, #queue-req: 0,","1":"(SGLangEngine pid=111711, ip=10.15.36.1)","pid":111711,"ip":"10.15.36.1","time":"2025-10-31 03:45:20","dp_rank":null,"tp_rank":0,"ep_rank":null,"pp_rank":null,"9":" [39913]","num_running_req":78,"num_token":432100,"token_usage":0.27,"gen_throughput":7269.16,"queue_req":0},{"line":"[2025-11-03 14:31:10 DP6 TP6 EP6] Decode batch, #running-req: 251, #token: 2811200, token usage: 1.00, cuda graph: True, gen throughput (token/s): 2055.94, #queue-req: 655,","1":null,"pid":null,"ip":null,"time":"2025-11-03 14:31:10","dp_rank":6,"tp_rank":6,"ep_rank":6,"pp_rank":null,"9":null,"num_running_req":251,"num_token":2811200,"token_usage":1.0,"gen_throughput":2055.94,"queue_req":655}]""",
        )

        df = log_parser.parse(lines)
        print(df)
        print(df.write_json())

        assert len(df) == len(lines.strip().splitlines()), f"{len(df)=}"
        self.assertEqual(json.loads(df.write_json()), expect_rows)
```
**EN:** This range defines concrete test routine(s) `test_log_parser`. The logic drives the target scenario and encodes the expected acceptance criteria. Assertions in this block enforce the intended outcome. Representative call sites include `throughput`, `loads`, `parse` and `write_json`.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 25-28: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Performance benchmarking / 性能基准测试
- Multi-GPU orchestration / 多 GPU 编排
- Token-level inspection / Token 级分析
- Disaggregated serving / 解耦式服务
- Structured outputs / 结构化输出

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `json`, `unittest`
- **Third-party / 第三方库**: None / 无
- **Project Modules / 项目模块**: `sglang.srt.debug_utils`, `sglang.test.test_utils`
