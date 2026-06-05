# test_fused_moe.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/moe/test_fused_moe.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates fused moe behavior in SGLang's moe area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 MoE 领域中与 fused moe 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-14: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch
from tqdm import tqdm

from sglang.srt.layers.activation import SiluAndMul
from sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe import fused_moe
from sglang.srt.layers.moe.topk import TopKConfig, select_experts
from sglang.srt.layers.quantization.fp8_kernel import is_fp8_fnuz
from sglang.srt.layers.quantization.fp8_utils import normalize_e4m3fn_to_e4m3fnuz
from sglang.srt.server_args import ServerArgs, set_global_server_args_for_scheduler
from sglang.srt.utils import get_device, get_device_capability, is_hip
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import CustomTestCase, empty_gpu_cache
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `tqdm`, `sglang.srt.layers.activation`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `tqdm`, `sglang.srt.layers.activation`。

### Lines 16-20: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=87, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=30, suite="stage-b-test-1-gpu-small-amd")

_is_hip = is_hip()
_is_fp8_fnuz = is_fp8_fnuz()
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci, is_hip.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci, is_hip 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 23-23: class TestFusedMOE declaration / 类 TestFusedMOE 声明
```python
class TestFusedMOE(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 24-25: class-level constants and configuration for `TestFusedMOE` / 类级常量与配置
```python
    NUM_EXPERTS = [8, 64]
    TOP_KS = [2, 6]
```
**EN:** This block defines shared names such as `NUM_EXPERTS`, `TOP_KS`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `NUM_EXPERTS`, `TOP_KS` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 27-40: helper routine create random gpu tensor / 辅助流程 create random gpu tensor
```python
    @staticmethod
    def create_random_gpu_tensor(shape, dtype, mean=0, std=0.01):
        """Create a random Torch(device) tensor

        Args:
            shape: Tensor shape
            dtype: Data type
            mean: Mean value
            std: Standard deviation

        Returns:
            torch.Tensor: Randomly initialized Torch(device) tensor
        """
        return torch.empty(shape, dtype=dtype, device=get_device()).normal_(mean, std)
```
**EN:** Create a random Torch(device) tensor Args: shape: Tensor shape dtype: Data type mean: Mean value std: Standard deviation Returns: torch.Tensor: Randomly initialized Torch(device) tensor This helper encapsulates `create_random_gpu_tensor` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Create a random Torch(device) tensor Args: shape: Tensor shape dtype: Data type mean: Mean value std: Standard deviation Returns: torch.Tensor: Randomly initialized Torch(device) tensor 该辅助函数封装了 `create_random_gpu_tensor`，以便周围测试复用准备、执行或校验逻辑。

### Lines 42-56: helper routine get tolerance / 辅助流程 get tolerance
```python
    def get_tolerance(self, dtype):
        """Get tolerance values for different data types

        Args:
            dtype: Data type

        Returns:
            tuple: (relative tolerance, absolute tolerance)
        """
        if dtype == torch.float32:
            return 1e-3, 1e-5
        elif dtype in [torch.float16, torch.bfloat16]:
            return 1e-1, 1e-2
        else:
            return 1e-2, 1e-2  # Default values for other types
```
**EN:** Get tolerance values for different data types Args: dtype: Data type Returns: tuple: (relative tolerance, absolute tolerance) This helper encapsulates `get_tolerance` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Get tolerance values for different data types Args: dtype: Data type Returns: tuple: (relative tolerance, absolute tolerance) 该辅助函数封装了 `get_tolerance`，以便周围测试复用准备、执行或校验逻辑。

### Lines 58-105: method torch naive moe / 方法 torch naive moe
```python
    def torch_naive_moe(
        self,
        a,
        w1,
        w2,
        score,
        topk,
        w1_scale=None,
        w2_scale=None,
        a1_scale=None,
        a2_scale=None,
    ):
        set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))

        B, D = a.shape
        a = a.view(B, -1, D).repeat(1, topk, 1).reshape(-1, D)
        out = torch.zeros(B * topk, w2.shape[1], dtype=a.dtype, device=a.device)
        score = torch.softmax(score, dim=-1, dtype=torch.float32)
        topk_weight, topk_ids = torch.topk(score, topk)
        topk_weight = topk_weight.view(-1)
        topk_ids = topk_ids.view(-1)

        if w1.dtype in [torch.float8_e4m3fn, torch.float8_e4m3fnuz]:
            w1_compute = w1.to(a.dtype)
            w2_compute = w2.to(a.dtype)

            if w1_scale is not None:
                w1_compute = (w1_compute * w1_scale.view(-1, 1, 1)).to(a.dtype)
            if w2_scale is not None:
                w2_compute = (w2_compute * w2_scale.view(-1, 1, 1)).to(a.dtype)
            if a1_scale is not None:
                a = (a * a1_scale).to(a.dtype)
            if a2_scale is not None:
                a = (a * a2_scale).to(a.dtype)
        else:
            w1_compute = w1
            w2_compute = w2

        for i in range(w1_compute.shape[0]):
            mask = topk_ids == i
            if mask.sum():
                out[mask] = SiluAndMul()(
                    a[mask] @ w1_compute[i].transpose(0, 1)
                ) @ w2_compute[i].transpose(0, 1)

        return (
            out.view(B, -1, w2.shape[1]) * topk_weight.view(B, -1, 1).to(out.dtype)
        ).sum(dim=1)
```
**EN:** This block implements `torch_naive_moe` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `torch_naive_moe`，承担模块行为中的一个聚焦逻辑片段。

### Lines 107-186: method test case (part 1/2) / 方法 test case（第 1/2 部分）
```python
    def _test_case(self, m, n, k, e, topk, dtype, use_fp8_w8a8=False):
        rtol, atol = self.get_tolerance(dtype)

        if use_fp8_w8a8:
            # AssertionError: fp8e4nv data type is not supported on CUDA arch < 89
            capability = get_device_capability()
            if not _is_hip and not (capability[0] >= 9 or capability == (8, 9)):
                return

            a = self.create_random_gpu_tensor((m, k), dtype)
            w1 = self.create_random_gpu_tensor((e, 2 * n, k), dtype)
            w2 = self.create_random_gpu_tensor((e, k, n), dtype)
            w1 = w1.to(torch.float8_e4m3fn)
            w2 = w2.to(torch.float8_e4m3fn)
            score = self.create_random_gpu_tensor((m, e), dtype)
            w1_scale = self.create_random_gpu_tensor(e, torch.float32)
            w2_scale = self.create_random_gpu_tensor(e, torch.float32)
            a1_scale = self.create_random_gpu_tensor(1, torch.float32)
            a2_scale = self.create_random_gpu_tensor(1, torch.float32)

            # Handle HIP case: normalize float8 weights so fused kernel doesn't break
            # on ROCm.
            if _is_fp8_fnuz:
                # Normalize to e4m3fnuz on HIP
                w1, w1_scale, _ = normalize_e4m3fn_to_e4m3fnuz(
                    weight=w1,
                    weight_scale=w1_scale,
                    input_scale=a1_scale,
                )
                w2, w2_scale, _ = normalize_e4m3fn_to_e4m3fnuz(
                    weight=w2,
                    weight_scale=w2_scale,
                    input_scale=a2_scale,
                )

            topk_output = select_experts(
                hidden_states=a,
                router_logits=score,
                topk_config=TopKConfig(top_k=topk, renormalize=False),
            )

            torch_output = self.torch_naive_moe(
                a,
                w1,
                w2,
                score,
                topk,
                w1_scale,
                w2_scale,
                a1_scale,
                a2_scale,
            )

            sglang_output = fused_moe(
                a,
                w1,
                w2,
                topk_output,
                use_fp8_w8a8=True,
                w1_scale=w1_scale,
                w2_scale=w2_scale,
                a1_scale=a1_scale,
                a2_scale=a2_scale,
            )
            torch.testing.assert_close(
                sglang_output, torch_output, rtol=rtol, atol=atol
            )
        else:
            a = self.create_random_gpu_tensor((m, k), dtype)
            w1 = self.create_random_gpu_tensor((e, 2 * n, k), dtype)
            w2 = self.create_random_gpu_tensor((e, k, n), dtype)
            score = self.create_random_gpu_tensor((m, e), dtype)

            topk_output = select_experts(
                hidden_states=a,
                router_logits=score,
                topk_config=TopKConfig(top_k=topk, renormalize=False),
            )

            triton_output = fused_moe(a, w1, w2, topk_output)
```
**EN:** This block implements `_test_case` and captures one focused piece of the module's behavior. This chunk continues part 1 of the same logical block.
**CN:** 该代码块实现 `_test_case`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 1 部分。

### Lines 187-190: method test case (part 2/2) / 方法 test case（第 2/2 部分）
```python
            torch_output = self.torch_naive_moe(a, w1, w2, score, topk)
            torch.testing.assert_close(
                triton_output, torch_output, rtol=rtol, atol=atol
            )
```
**EN:** This block implements `_test_case` and captures one focused piece of the module's behavior. This chunk continues part 2 of the same logical block.
**CN:** 该代码块实现 `_test_case`，承担模块行为中的一个聚焦逻辑片段。 这一段对应同一逻辑块的第 2 部分。

### Lines 192-240: test case various configurations / 测试用例 various configurations
```python
    def test_various_configurations(self):
        m_values = [1, 33, 64, 222]
        n_values = [128, 1024]
        k_values = [128, 511, 1024]
        dtypes = [torch.float16, torch.bfloat16]
        fp8_modes = [False, True]

        set_global_server_args_for_scheduler(ServerArgs(model_path="dummy"))

        # Calculate total number of tests
        total_tests = (
            len(m_values)
            * len(n_values)
            * len(k_values)
            * len(self.NUM_EXPERTS)
            * len(self.TOP_KS)
            * len(dtypes)
            * len(fp8_modes)
        )

        # Create progress bar
        with tqdm(total=total_tests, desc="Running MoE tests") as pbar:
            for m in m_values:
                for n in n_values:
                    for k in k_values:
                        for e in self.NUM_EXPERTS:
                            for topk in self.TOP_KS:
                                for dtype in dtypes:
                                    for use_fp8_w8a8 in fp8_modes:
                                        with self.subTest(
                                            m=m,
                                            n=n,
                                            k=k,
                                            e=e,
                                            topk=topk,
                                            dtype=dtype,
                                            fp8=use_fp8_w8a8,
                                        ):
                                            self._test_case(
                                                m,
                                                n,
                                                k,
                                                e,
                                                topk,
                                                dtype,
                                                use_fp8_w8a8=use_fp8_w8a8,
                                            )
                                            empty_gpu_cache()
                                        pbar.update(1)
```
**EN:** This test exercises `test_various_configurations` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_various_configurations`。

### Lines 243-244: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestFusedMOE`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestFusedMOE.create_random_gpu_tensor`: Create a random Torch(device) tensor Args: shape: Tensor shape dtype: Data type mean: Mean value std: Standard deviation Returns: torch.Tensor: Randomly initialized Torch(device) tensor / 该辅助函数封装了 `create_random_gpu_tensor`，以便周围测试复用准备、执行或校验逻辑。
- `TestFusedMOE.get_tolerance`: Get tolerance values for different data types Args: dtype: Data type Returns: tuple: (relative tolerance, absolute tolerance) / 该辅助函数封装了 `get_tolerance`，以便周围测试复用准备、执行或校验逻辑。
- `TestFusedMOE.torch_naive_moe`: This block implements `torch_naive_moe` and captures one focused piece of the module's behavior. / 该代码块实现 `torch_naive_moe`，承担模块行为中的一个聚焦逻辑片段。
- `TestFusedMOE._test_case`: This block implements `_test_case` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_case`，承担模块行为中的一个聚焦逻辑片段。
- `TestFusedMOE.test_various_configurations`: This test exercises `test_various_configurations` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_various_configurations`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`, `tqdm`
- **Internal modules / 内部模块**: `sglang.srt.layers.activation`, `sglang.srt.layers.moe.moe_runner.triton_utils.fused_moe`, `sglang.srt.layers.moe.topk`, `sglang.srt.layers.quantization.fp8_kernel`, `sglang.srt.layers.quantization.fp8_utils`, `sglang.srt.server_args`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 244
