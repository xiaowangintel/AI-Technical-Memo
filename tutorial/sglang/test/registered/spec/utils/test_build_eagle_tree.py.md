# test_build_eagle_tree.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/spec/utils/test_build_eagle_tree.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates build eagle tree behavior in SGLang's spec / utils area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 推测解码 / 工具 领域中与 build eagle tree 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and dependencies / 模块导入与依赖
```python
import unittest

import torch

from sglang.srt.speculative.eagle_utils import (
    build_tree_kernel_efficient,
    organize_draft_results,
)
from sglang.srt.utils import get_device
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `unittest`, `torch`, `sglang.srt.speculative.eagle_utils`, `sglang.srt.utils`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `unittest`, `torch`, `sglang.srt.speculative.eagle_utils`, `sglang.srt.utils`。

### Lines 12-13: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=6, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=3, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 16-16: class TestBuildEagleTree declaration / 类 TestBuildEagleTree 声明
```python
class TestBuildEagleTree(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 17-17: supporting statements / 辅助语句
```python
    """Unit tests for build_eagle_tree functionality."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 19-98: test case build tree kernel efficient (part 1/4) / 测试用例 build tree kernel efficient（第 1/4 部分）
```python
    def test_build_tree_kernel_efficient(self):
        """Test the build_tree_kernel_efficient function with known inputs and expected outputs."""
        bonus_tokens = torch.tensor([29974, 13], device=get_device(), dtype=torch.int32)
        score_list = [
            torch.tensor(
                [
                    [[7.1127e-01, 2.8292e-01, 2.2995e-03, 1.7357e-03]],
                    [[9.7476e-01, 2.2219e-02, 6.5031e-04, 1.3212e-04]],
                ],
                dtype=torch.float32,
                device=get_device(),
            ),
            torch.tensor(
                [
                    [
                        [6.9142e-01, 1.2863e-02, 1.6873e-03, 1.1871e-03],
                        [2.4787e-01, 1.8818e-02, 1.4204e-02, 9.2235e-04],
                        [2.2971e-03, 1.6700e-06, 1.8737e-07, 8.3146e-08],
                        [1.2771e-03, 2.4374e-04, 1.7832e-04, 1.1947e-05],
                    ],
                    [
                        [8.4832e-02, 6.6068e-02, 5.8304e-02, 5.7851e-02],
                        [2.3616e-03, 1.1243e-03, 5.4368e-04, 2.7768e-04],
                        [2.5286e-04, 1.5578e-04, 2.8817e-05, 1.2888e-05],
                        [1.2834e-04, 2.5417e-06, 1.1279e-06, 1.6088e-08],
                    ],
                ],
                dtype=torch.float32,
                device=get_device(),
            ),
            torch.tensor(
                [
                    [
                        [6.6438e-01, 2.6997e-02, 2.4236e-05, 4.0821e-06],
                        [2.4402e-01, 2.8409e-03, 5.0935e-04, 2.9022e-04],
                        [1.6178e-02, 2.0567e-03, 4.5892e-04, 3.0034e-05],
                        [1.3023e-02, 5.0497e-04, 3.6371e-04, 8.7750e-05],
                    ],
                    [
                        [2.3263e-02, 2.0054e-02, 9.3990e-03, 2.7783e-03],
                        [6.4156e-02, 5.5506e-04, 1.0429e-04, 9.7211e-05],
                        [4.9950e-02, 5.0630e-03, 9.0068e-04, 3.3656e-04],
                        [7.5817e-03, 8.5731e-04, 6.9972e-04, 6.0793e-04],
                    ],
                ],
                dtype=torch.float32,
                device=get_device(),
            ),
            torch.tensor(
                [
                    [
                        [6.6420e-01, 1.0525e-04, 6.5864e-05, 1.2253e-06],
                        [1.3019e-01, 1.0461e-01, 5.2083e-03, 1.6777e-03],
                        [2.0103e-02, 6.7335e-03, 1.2625e-04, 1.0364e-05],
                        [1.5142e-02, 7.0819e-04, 9.6595e-05, 8.7951e-05],
                    ],
                    [
                        [5.8608e-02, 1.8840e-03, 7.8535e-04, 4.4400e-04],
                        [1.2185e-02, 2.0684e-03, 1.7418e-03, 1.4327e-03],
                        [6.2455e-03, 6.1487e-03, 2.6862e-03, 1.8034e-03],
                        [1.8590e-03, 1.6151e-03, 1.2481e-03, 3.6038e-04],
                    ],
                ],
                dtype=torch.float32,
                device=get_device(),
            ),
        ]
        token_list = [
            torch.tensor(
                [[29896, 29906, 29900, 29945], [13, 2, 29871, 28956]],
                dtype=torch.int64,
                device=get_device(),
            ),
            torch.tensor(
                [
                    [
                        29889,
                        29974,
                        29945,
                        29900,
```
**EN:** Test the build_tree_kernel_efficient function with known inputs and expected outputs. This test exercises `test_build_tree_kernel_efficient` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** Test the build_tree_kernel_efficient function with known inputs and expected outputs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_tree_kernel_efficient`。 这一段对应同一逻辑块的第 1 部分。

### Lines 99-178: test case build tree kernel efficient (part 2/4) / 测试用例 build tree kernel efficient（第 2/4 部分）
```python
                        29974,
                        29922,
                        29930,
                        29958,
                        29889,
                        29974,
                        29930,
                        29945,
                        29974,
                        29922,
                        29930,
                        29958,
                    ],
                    [
                        22550,
                        4136,
                        16492,
                        8439,
                        29871,
                        2,
                        3001,
                        13,
                        2,
                        13,
                        29906,
                        29946,
                        2,
                        13,
                        29871,
                        259,
                    ],
                ],
                device=get_device(),
            ),
            torch.tensor(
                [
                    [
                        29946,
                        29945,
                        29953,
                        29906,
                        29896,
                        29945,
                        29900,
                        29906,
                        29896,
                        29945,
                        29906,
                        29953,
                        29896,
                        29945,
                        29906,
                        29946,
                    ],
                    [
                        29871,
                        2,
                        29901,
                        29889,
                        29871,
                        2,
                        395,
                        259,
                        29901,
                        29871,
                        2,
                        29889,
                        3001,
                        1234,
                        7146,
                        2186,
                    ],
                ],
                device=get_device(),
            ),
            torch.tensor(
                [
                    [
                        29946,
                        29974,
```
**EN:** Test the build_tree_kernel_efficient function with known inputs and expected outputs. This test exercises `test_build_tree_kernel_efficient` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** Test the build_tree_kernel_efficient function with known inputs and expected outputs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_tree_kernel_efficient`。 这一段对应同一逻辑块的第 2 部分。

### Lines 179-258: test case build tree kernel efficient (part 3/4) / 测试用例 build tree kernel efficient（第 3/4 部分）
```python
                        29945,
                        29930,
                        29889,
                        29922,
                        29974,
                        29930,
                        29974,
                        29946,
                        29930,
                        29922,
                        29889,
                        29974,
                        29945,
                        29922,
                    ],
                    [
                        29941,
                        29906,
                        2,
                        29946,
                        29871,
                        450,
                        319,
                        14990,
                        29946,
                        29941,
                        2,
                        29906,
                        29871,
                        2,
                        3001,
                        13,
                    ],
                ],
                device=get_device(),
            ),
        ]
        parents_list = [
            torch.tensor(
                [[-1, 0, 1, 2, 3], [-1, 0, 1, 2, 3]], dtype=torch.int64, device="cuda"
            ),
            torch.tensor(
                [[4, 8, 9, 10], [4, 5, 6, 7]], dtype=torch.int64, device="cuda"
            ),
            torch.tensor(
                [[20, 24, 21, 28], [24, 28, 20, 21]], dtype=torch.int64, device="cuda"
            ),
            torch.tensor(
                [[36, 40, 41, 44], [36, 40, 44, 45]], dtype=torch.int64, device="cuda"
            ),
        ]
        seq_lens = torch.tensor([5, 10], dtype=torch.int64, device="cuda")
        topk = 4
        depth = 4
        num_draft_token = 8

        parent_list, top_scores_index, draft_tokens = organize_draft_results(
            score_list, token_list, parents_list, num_draft_token
        )

        (
            tree_mask,
            position,
            retrieve_index,
            retrieve_next_token,
            retrieve_next_sibling,
            draft_tokens,
        ) = build_tree_kernel_efficient(
            bonus_tokens=bonus_tokens,
            parent_list=parent_list,
            top_scores_index=top_scores_index,
            draft_tokens=draft_tokens,
            seq_lens=seq_lens,
            seq_lens_sum=torch.sum(seq_lens).item(),
            topk=topk,
            spec_steps=depth,
            num_verify_tokens=num_draft_token,
        )

        # Verify expected outputs
```
**EN:** Test the build_tree_kernel_efficient function with known inputs and expected outputs. This test exercises `test_build_tree_kernel_efficient` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 3 of the same logical block.
**CN:** Test the build_tree_kernel_efficient function with known inputs and expected outputs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_tree_kernel_efficient`。 这一段对应同一逻辑块的第 3 部分。

### Lines 259-309: test case build tree kernel efficient (part 4/4) / 测试用例 build tree kernel efficient（第 4/4 部分）
```python
        self.assertEqual(
            position.tolist(),
            [5, 6, 6, 7, 7, 8, 8, 9, 10, 11, 12, 12, 12, 12, 13, 14],
            "Position tensor does not match expected values",
        )
        self.assertEqual(
            retrieve_index.tolist(),
            [
                [0, 1, 2, 3, 4, 5, 6, 7],
                [8, 9, 10, 11, 12, 13, 14, 15],
            ],
            "Retrieve index tensor does not match expected values",
        )
        self.assertEqual(
            retrieve_next_token.tolist(),
            [
                [1, 3, 4, 5, 6, 7, -1, -1],
                [1, 2, -1, 6, -1, -1, 7, -1],
            ],
            "Retrieve next token tensor does not match expected values",
        )
        self.assertEqual(
            retrieve_next_sibling.tolist(),
            [
                [-1, 2, -1, -1, -1, -1, -1, -1],
                [-1, -1, 3, 4, 5, -1, -1, -1],
            ],
            "Retrieve next sibling tensor does not match expected values",
        )
        self.assertEqual(
            draft_tokens.tolist(),
            [
                29974,
                29896,
                29906,
                29889,
                29974,
                29946,
                29896,
                29946,
                13,
                13,
                22550,
                4136,
                16492,
                8439,
                29871,
                29941,
            ],
            "Draft tokens tensor does not match expected values",
        )
```
**EN:** Test the build_tree_kernel_efficient function with known inputs and expected outputs. This test exercises `test_build_tree_kernel_efficient` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 4 of the same logical block.
**CN:** Test the build_tree_kernel_efficient function with known inputs and expected outputs. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_tree_kernel_efficient`。 这一段对应同一逻辑块的第 4 部分。

### Lines 312-313: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestBuildEagleTree`: Unit tests for build_eagle_tree functionality. / 用于组织相关测试、夹具或辅助方法。
- `TestBuildEagleTree.test_build_tree_kernel_efficient`: Test the build_tree_kernel_efficient function with known inputs and expected outputs. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_build_tree_kernel_efficient`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `unittest`
- **Third-party modules / 第三方模块**: `torch`
- **Internal modules / 内部模块**: `sglang.srt.speculative.eagle_utils`, `sglang.srt.utils`, `sglang.test.ci.ci_register`

- **Total lines / 总行数**: 313
