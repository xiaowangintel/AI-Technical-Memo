# test_piecewise_cuda_graph_support_1_gpu.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/piecewise_cuda_graph/test_piecewise_cuda_graph_support_1_gpu.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates piecewise cuda graph support 1 gpu behavior in SGLang's piecewise cuda graph area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 piecewise cuda graph 领域中与 piecewise cuda graph support 1 gpu 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang import Engine
from sglang.lang.chat_template import get_chat_template_by_model_path
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.run_eval import run_eval
from sglang.test.test_utils import (
    DEFAULT_IMAGE_URL,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    SimpleNamespace,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang`, `sglang.lang.chat_template`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang`, `sglang.lang.chat_template`。

### Lines 18-19: supporting source context / 辅助源码上下文
```python

# CI Registration
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 20-20: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=180, stage="base-b", runner_config="1-gpu-large")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 23-23: class TestPiecewiseCudaGraphQwen25VL declaration / 类 TestPiecewiseCudaGraphQwen25VL 声明
```python
class TestPiecewiseCudaGraphQwen25VL(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 24-24: supporting statements / 辅助语句
```python
    """Test piecewise CUDA graph with Qwen2.5-VL-7B-Instruct model"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 26-38: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "Qwen/Qwen2.5-VL-7B-Instruct"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--enforce-piecewise-cuda-graph",
                "--disable-radix-cache",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 40-42: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 44-56: test case gsm8k accuracy / 测试用例 gsm8k accuracy
```python
    def test_gsm8k_accuracy(self):
        args = SimpleNamespace(
            base_url=self.base_url,
            model=self.model,
            eval_name="gsm8k",
            num_examples=None,
            num_threads=1024,
        )

        metrics = run_eval(args)
        print(f"GSM8K Accuracy: {metrics['score']:.3f}")

        self.assertGreaterEqual(metrics["score"], 0.80)
```
**EN:** This test exercises `test_gsm8k_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_accuracy`。

### Lines 59-59: class TestPiecewiseCudaGraphQwen25VLEmbedding declaration / 类 TestPiecewiseCudaGraphQwen25VLEmbedding 声明
```python
class TestPiecewiseCudaGraphQwen25VLEmbedding(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 60-60: supporting statements / 辅助语句
```python
    """Test piecewise CUDA graph with Qwen2.5-VL-3B-Instruct embedding model"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 62-108: test case embedding / 测试用例 embedding
```python
    def test_embedding(self):
        model_path = "Qwen/Qwen2.5-VL-3B-Instruct"
        chat_template = get_chat_template_by_model_path(model_path)
        text = f"{chat_template.image_token}What is in this picture? Answer: "

        engine = Engine(
            model_path=model_path,
            enable_multimodal=True,
            is_embedding=True,
            enforce_piecewise_cuda_graph=True,
        )
        out = engine.encode([text], image_data=[DEFAULT_IMAGE_URL])[0]["embedding"]
        engine.shutdown()
        self.assertGreater(len(out), 0)

        engine = Engine(
            model_path=model_path,
            enable_multimodal=True,
            is_embedding=True,
            disable_piecewise_cuda_graph=True,
        )
        out_without_pcg = engine.encode([text], image_data=[DEFAULT_IMAGE_URL])[0][
            "embedding"
        ]
        engine.shutdown()
        self.assertGreater(len(out_without_pcg), 0)

        t_out = torch.tensor(out)
        t_out_without_pcg = torch.tensor(out_without_pcg)
        max_abs_diff = (t_out - t_out_without_pcg).abs().max().item()
        max_rel_diff = (
            ((t_out - t_out_without_pcg).abs() / (t_out_without_pcg.abs() + 1e-8))
            .max()
            .item()
        )
        print(
            f"PCG embedding diff: max_abs={max_abs_diff:.6f}, max_rel={max_rel_diff:.6f}"
        )
        self.assertTrue(
            torch.allclose(
                t_out,
                t_out_without_pcg,
                atol=1e-2,
                rtol=1e-2,
            ),
            f"Piecewise CUDA graph embedding mismatch: max_abs_diff={max_abs_diff}, max_rel_diff={max_rel_diff}",
        )
```
**EN:** This test exercises `test_embedding` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding`。

### Lines 111-112: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestPiecewiseCudaGraphQwen25VL`: Test piecewise CUDA graph with Qwen2.5-VL-7B-Instruct model / 用于组织相关测试、夹具或辅助方法。
- `TestPiecewiseCudaGraphQwen25VLEmbedding`: Test piecewise CUDA graph with Qwen2.5-VL-3B-Instruct embedding model / 用于组织相关测试、夹具或辅助方法。
- `TestPiecewiseCudaGraphQwen25VL.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestPiecewiseCudaGraphQwen25VL.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestPiecewiseCudaGraphQwen25VL.test_gsm8k_accuracy`: This test exercises `test_gsm8k_accuracy` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gsm8k_accuracy`。
- `TestPiecewiseCudaGraphQwen25VLEmbedding.test_embedding`: This test exercises `test_embedding` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_embedding`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang`, `sglang.lang.chat_template`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.run_eval`, `sglang.test.test_utils`

- **Total lines / 总行数**: 112
