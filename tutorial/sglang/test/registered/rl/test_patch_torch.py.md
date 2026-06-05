# test_patch_torch.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/rl/test_patch_torch.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates patch torch behavior in SGLang's rl area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 rl 领域中与 patch torch 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and dependencies / 模块导入与依赖
```python
import os
import traceback
import unittest
from typing import List

import torch
import torch.multiprocessing as mp

from sglang.srt.utils.patch_torch import monkey_patch_torch_reductions
from sglang.test.ci.ci_register import register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `traceback`, `unittest`, `typing`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `traceback`, `unittest`, `typing`。

### Lines 12-12: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=15, stage="base-b", runner_config="2-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 15-15: class TestReleaseMemoryOccupation declaration / 类 TestReleaseMemoryOccupation 声明
```python
class TestReleaseMemoryOccupation(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 16-64: test case monkey patch torch reductions / 测试用例 monkey patch torch reductions
```python
    def test_monkey_patch_torch_reductions(self):
        mp.set_start_method("spawn", force=True)

        cuda_visible_devices_list: List[int] = [
            int(x)
            for x in os.environ.get("CUDA_VISIBLE_DEVICES", "0,1,2,3,4,5,6,7").split(
                ","
            )
        ]

        # Sender's cuda:1 and receiver's cuda:0 map to the same physical device.
        # With the patch, the IPC tensor must land on receiver's cuda:0.
        sender_info = dict(visible_devices=[0, 1], tensor_device=1)
        receiver_info = dict(visible_devices=[1, 0], tensor_device=0)

        processes = []
        output_reader, output_writer = mp.Pipe(duplex=False)
        # Split into SPSC queues; a single shared mp.Queue lets the sender's
        # get() pop its own put before the receiver wakes (CUDA IPC self-reopen fails).
        tensor_queue = mp.Queue()
        ack_queue = mp.Queue()
        for role, info in [
            ("sender", sender_info),
            ("receiver", receiver_info),
        ]:
            os.environ["CUDA_VISIBLE_DEVICES"] = ",".join(
                str(cuda_visible_devices_list[device])
                for device in info["visible_devices"]
            )
            p = mp.Process(
                target=_run_subprocess,
                kwargs=dict(
                    role=role,
                    tensor_queue=tensor_queue,
                    ack_queue=ack_queue,
                    output_writer=output_writer,
                    tensor_device=info["tensor_device"],
                ),
            )
            p.start()
            processes.append(p)

        for _ in range(len(processes)):
            self.assertTrue(
                output_reader.recv(), f"Subprocess has error, please see logs above."
            )

        for p in processes:
            p.join()
```
**EN:** This test exercises `test_monkey_patch_torch_reductions` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_monkey_patch_torch_reductions`。

### Lines 67-102: function run subprocess / 函数 run subprocess
```python
def _run_subprocess(
    role: str,
    tensor_queue: mp.Queue,
    ack_queue: mp.Queue,
    output_writer,
    tensor_device: int,
):
    print(
        f'subprocess[{role}] start {os.environ.get("CUDA_VISIBLE_DEVICES")=}',
        flush=True,
    )

    monkey_patch_torch_reductions()

    try:
        if role == "sender":
            tensor = torch.tensor([1.0, 2.0], device=f"cuda:{tensor_device}")
            print(f"sender tensor_queue.put {tensor=} {tensor.device=}")
            tensor_queue.put(tensor)
            assert ack_queue.get() == "done"
        elif role == "receiver":
            tensor = tensor_queue.get()
            print(f"receiver tensor_queue.get {tensor=} {tensor.device=}")
            assert str(tensor.device) == f"cuda:{tensor_device}"
            ack_queue.put("done")
        else:
            raise NotImplementedError

        execution_ok = True
    except Exception as e:
        print(f"subprocess[{role}] has error: {e}", flush=True)
        traceback.print_exc()
        execution_ok = False

    output_writer.send(execution_ok)
    output_writer.close()
```
**EN:** This block implements `_run_subprocess` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_run_subprocess`，承担模块行为中的一个聚焦逻辑片段。

### Lines 105-106: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestReleaseMemoryOccupation`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `_run_subprocess`: This block implements `_run_subprocess` and captures one focused piece of the module's behavior. / 该代码块实现 `_run_subprocess`，承担模块行为中的一个聚焦逻辑片段。
- `TestReleaseMemoryOccupation.test_monkey_patch_torch_reductions`: This test exercises `test_monkey_patch_torch_reductions` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_monkey_patch_torch_reductions`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `traceback`, `unittest`, `typing`
- **Third-party modules / 第三方模块**: `torch`, `torch.multiprocessing`
- **Internal modules / 内部模块**: `sglang.srt.utils.patch_torch`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 106
