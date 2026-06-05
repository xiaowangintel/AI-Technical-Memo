# test_io_struct.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/managers/test_io_struct.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates io struct behavior in SGLang's unit / managers area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / managers 领域中与 io struct 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-10: module imports and dependencies / 模块导入与依赖
```python
import copy
import unittest

from sglang.srt.managers.io_struct import GenerateReqInput
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `copy`, `unittest`, `sglang.srt.managers.io_struct`, `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `copy`, `unittest`, `sglang.srt.managers.io_struct`, `sglang.test.ci.ci_register`。

### Lines 12-13: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=8, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=8, suite="stage-b-test-1-gpu-small-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 16-16: class TestGenerateReqInputNormalization declaration / 类 TestGenerateReqInputNormalization 声明
```python
class TestGenerateReqInputNormalization(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 17-17: supporting statements / 辅助语句
```python
    """Test the normalization of GenerateReqInput for batch processing and different input formats."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 19-22: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 24-30: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        # Common setup for all tests
        self.base_req = GenerateReqInput(
            text=["Hello", "World"],
            sampling_params=[{}, {}],
            rid=["id1", "id2"],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 32-47: test case single image to list of lists / 测试用例 single image to list of lists
```python
    def test_single_image_to_list_of_lists(self):
        """Test that a single image is converted to a list of single-image lists."""
        req = copy.deepcopy(self.base_req)
        req.image_data = "single_image.jpg"  # A single image (non-list)

        req.normalize_batch_and_arguments()

        # Should be converted to [[image], [image]]
        self.assertEqual(len(req.image_data), 2)
        self.assertEqual(len(req.image_data[0]), 1)
        self.assertEqual(len(req.image_data[1]), 1)
        self.assertEqual(req.image_data[0][0], "single_image.jpg")
        self.assertEqual(req.image_data[1][0], "single_image.jpg")

        # Check modalities
        self.assertEqual(req.modalities, ["image", "image"])
```
**EN:** Test that a single image is converted to a list of single-image lists. This test exercises `test_single_image_to_list_of_lists` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that a single image is converted to a list of single-image lists. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_image_to_list_of_lists`。

### Lines 49-64: test case list of images to list of lists / 测试用例 list of images to list of lists
```python
    def test_list_of_images_to_list_of_lists(self):
        """Test that a list of images is converted to a list of single-image lists."""
        req = copy.deepcopy(self.base_req)
        req.image_data = ["image1.jpg", "image2.jpg"]  # List of images

        req.normalize_batch_and_arguments()

        # Should be converted to [[image1], [image2]]
        self.assertEqual(len(req.image_data), 2)
        self.assertEqual(len(req.image_data[0]), 1)
        self.assertEqual(len(req.image_data[1]), 1)
        self.assertEqual(req.image_data[0][0], "image1.jpg")
        self.assertEqual(req.image_data[1][0], "image2.jpg")

        # Check modalities
        self.assertEqual(req.modalities, ["image", "image"])
```
**EN:** Test that a list of images is converted to a list of single-image lists. This test exercises `test_list_of_images_to_list_of_lists` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that a list of images is converted to a list of single-image lists. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_list_of_images_to_list_of_lists`。

### Lines 66-82: test case list of lists with different modalities / 测试用例 list of lists with different modalities
```python
    def test_list_of_lists_with_different_modalities(self):
        """Test handling of list of lists of images with different modalities."""
        req = copy.deepcopy(self.base_req)
        req.image_data = [
            ["image1.jpg"],  # Single image (image modality)
            ["image2.jpg", "image3.jpg"],  # Multiple images (multi-images modality)
        ]

        req.normalize_batch_and_arguments()

        # Structure should remain the same
        self.assertEqual(len(req.image_data), 2)
        self.assertEqual(len(req.image_data[0]), 1)
        self.assertEqual(len(req.image_data[1]), 2)

        # Check modalities
        self.assertEqual(req.modalities, ["image", "multi-images"])
```
**EN:** Test handling of list of lists of images with different modalities. This test exercises `test_list_of_lists_with_different_modalities` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of list of lists of images with different modalities. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_list_of_lists_with_different_modalities`。

### Lines 84-100: test case list of lists with none values / 测试用例 list of lists with none values
```python
    def test_list_of_lists_with_none_values(self):
        """Test handling of list of lists with None values."""
        req = copy.deepcopy(self.base_req)
        req.image_data = [
            [None],  # None value
            ["image.jpg"],  # Single image
        ]

        req.normalize_batch_and_arguments()

        # Structure should remain the same
        self.assertEqual(len(req.image_data), 2)
        self.assertEqual(len(req.image_data[0]), 1)
        self.assertEqual(len(req.image_data[1]), 1)

        # Check modalities
        self.assertEqual(req.modalities, [None, "image"])
```
**EN:** Test handling of list of lists with None values. This test exercises `test_list_of_lists_with_none_values` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of list of lists with None values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_list_of_lists_with_none_values`。

### Lines 102-129: test case expanding parallel sample correlation / 测试用例 expanding parallel sample correlation
```python
    def test_expanding_parallel_sample_correlation(self):
        """Test that when expanding with parallel samples, prompts, images and modalities are properly correlated."""
        req = copy.deepcopy(self.base_req)
        req.text = ["Prompt 1", "Prompt 2"]
        req.image_data = [
            ["image1.jpg"],
            ["image2.jpg", "image3.jpg"],
        ]
        req.sampling_params = {"n": 3}  # All prompts get 3 samples

        # Define expected values before normalization
        expected_text = req.text * 3
        expected_images = req.image_data * 3
        expected_modalities = ["image", "multi-images"] * 3

        req.normalize_batch_and_arguments()

        # Should be expanded to 6 items (2 original * 3 parallel)
        self.assertEqual(len(req.image_data), 6)

        # Check that images are properly expanded
        self.assertEqual(req.image_data, expected_images)

        # Check modalities
        self.assertEqual(req.modalities, expected_modalities)

        # Ensure that text items are properly duplicated too
        self.assertEqual(req.text, expected_text)
```
**EN:** Test that when expanding with parallel samples, prompts, images and modalities are properly correlated. This test exercises `test_expanding_parallel_sample_correlation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that when expanding with parallel samples, prompts, images and modalities are properly correlated. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_expanding_parallel_sample_correlation`。

### Lines 131-160: test case specific parallel n per sample / 测试用例 specific parallel n per sample
```python
    def test_specific_parallel_n_per_sample(self):
        """Test parallel expansion when different samples have different n values."""
        req = copy.deepcopy(self.base_req)
        req.text = ["Prompt 1", "Prompt 2"]
        req.image_data = [
            ["image1.jpg"],
            ["image2.jpg", "image3.jpg"],
        ]
        req.sampling_params = [
            {"n": 2},
            {"n": 2},
        ]  # First prompt gets 2 samples, second prompt gets 2 samples

        expected_images = req.image_data * 2
        expected_modalities = ["image", "multi-images"] * 2
        expected_text = req.text * 2

        req.normalize_batch_and_arguments()

        # Should be expanded to 4 items (2 original * 2 parallel)
        self.assertEqual(len(req.image_data), 4)

        # Check that the first 2 are copies for the first prompt
        self.assertEqual(req.image_data, expected_images)

        # Check modalities
        self.assertEqual(req.modalities, expected_modalities)

        # Check text expansion
        self.assertEqual(req.text, expected_text)
```
**EN:** Test parallel expansion when different samples have different n values. This test exercises `test_specific_parallel_n_per_sample` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test parallel expansion when different samples have different n values. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_specific_parallel_n_per_sample`。

### Lines 162-190: test case mixed none and images with parallel samples / 测试用例 mixed none and images with parallel samples
```python
    def test_mixed_none_and_images_with_parallel_samples(self):
        """Test that when some batch items have images and others None, parallel expansion works correctly."""
        req = copy.deepcopy(self.base_req)
        req.text = ["Prompt 1", "Prompt 2", "Prompt 3"]
        req.rid = ["id1", "id2", "id3"]
        req.image_data = [
            ["image1.jpg"],
            None,
            ["image3_1.jpg", "image3_2.jpg"],
        ]
        req.sampling_params = {"n": 2}  # All prompts get 2 samples

        expected_images = req.image_data * 2
        expected_modalities = ["image", None, "multi-images"] * 2
        expected_text = req.text * 2

        req.normalize_batch_and_arguments()

        # Should be expanded to 6 items (3 original * 2 parallel)
        self.assertEqual(len(req.image_data), 6)

        # Check image data
        self.assertEqual(req.image_data, expected_images)

        # Check modalities
        self.assertEqual(req.modalities, expected_modalities)

        # Check text expansion
        self.assertEqual(req.text, expected_text)
```
**EN:** Test that when some batch items have images and others None, parallel expansion works correctly. This test exercises `test_mixed_none_and_images_with_parallel_samples` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that when some batch items have images and others None, parallel expansion works correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_none_and_images_with_parallel_samples`。

### Lines 192-221: test case correlation with sampling params / 测试用例 correlation with sampling params
```python
    def test_correlation_with_sampling_params(self):
        """Test that sampling parameters are correctly correlated with prompts during expansion."""
        req = copy.deepcopy(self.base_req)
        req.text = ["Prompt 1", "Prompt 2"]
        req.image_data = [
            ["image1.jpg"],
            ["image2.jpg"],
        ]
        req.sampling_params = [
            {"temperature": 0.7, "n": 2},
            {"temperature": 0.9, "n": 2},
        ]

        req.normalize_batch_and_arguments()

        # Check sampling params expansion
        self.assertEqual(len(req.sampling_params), 4)
        self.assertEqual(req.sampling_params[0]["temperature"], 0.7)
        self.assertEqual(req.sampling_params[1]["temperature"], 0.9)
        self.assertEqual(req.sampling_params[2]["temperature"], 0.7)
        self.assertEqual(req.sampling_params[3]["temperature"], 0.9)

        # Should be expanded to 4 items (2 original * 2 parallel)
        self.assertEqual(len(req.image_data), 4)

        # Check correlation with images
        self.assertEqual(req.image_data[0], ["image1.jpg"])
        self.assertEqual(req.image_data[1], ["image2.jpg"])
        self.assertEqual(req.image_data[2], ["image1.jpg"])
        self.assertEqual(req.image_data[3], ["image2.jpg"])
```
**EN:** Test that sampling parameters are correctly correlated with prompts during expansion. This test exercises `test_correlation_with_sampling_params` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that sampling parameters are correctly correlated with prompts during expansion. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_correlation_with_sampling_params`。

### Lines 223-234: test case single example with image / 测试用例 single example with image
```python
    def test_single_example_with_image(self):
        """Test handling of single example with image."""
        req = GenerateReqInput(
            text="Hello",
            image_data="single_image.jpg",
        )

        req.normalize_batch_and_arguments()

        # For single examples, image_data doesn't get processed into lists
        self.assertEqual(req.image_data, "single_image.jpg")
        self.assertIsNone(req.modalities)  # Modalities isn't set for single examples
```
**EN:** Test handling of single example with image. This test exercises `test_single_example_with_image` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of single example with image. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_example_with_image`。

### Lines 236-259: test case single to batch with parallel sampling / 测试用例 single to batch with parallel sampling
```python
    def test_single_to_batch_with_parallel_sampling(self):
        """Test single example converted to batch with parallel sampling."""
        req = GenerateReqInput(
            text="Hello",
            image_data="single_image.jpg",
            sampling_params={"n": 3},  # parallel_sample_num = 3
        )

        # Define expected values before normalization
        expected_text = ["Hello"] * 3

        req.normalize_batch_and_arguments()

        # Should be converted to batch with text=["Hello"]
        self.assertEqual(req.text, expected_text)

        # Image should be automatically wrapped to list of lists with length 1*3=3
        self.assertEqual(len(req.image_data), 3)
        self.assertEqual(req.image_data[0][0], "single_image.jpg")
        self.assertEqual(req.image_data[1][0], "single_image.jpg")
        self.assertEqual(req.image_data[2][0], "single_image.jpg")

        # Modalities should be set for all 3 examples
        self.assertEqual(req.modalities, ["image", "image", "image"])
```
**EN:** Test single example converted to batch with parallel sampling. This test exercises `test_single_to_batch_with_parallel_sampling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test single example converted to batch with parallel sampling. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_to_batch_with_parallel_sampling`。

### Lines 261-282: test case audio data handling / 测试用例 audio data handling
```python
    def test_audio_data_handling(self):
        """Test handling of audio_data."""
        req = copy.deepcopy(self.base_req)
        req.audio_data = "audio.mp3"  # Single audio

        req.normalize_batch_and_arguments()

        # Should be converted to ["audio.mp3", "audio.mp3"]
        self.assertEqual(len(req.audio_data), 2)
        self.assertEqual(req.audio_data[0], "audio.mp3")
        self.assertEqual(req.audio_data[1], "audio.mp3")

        # Test with list
        req = copy.deepcopy(self.base_req)
        req.audio_data = ["audio1.mp3", "audio2.mp3"]

        req.normalize_batch_and_arguments()

        # Should remain the same
        self.assertEqual(len(req.audio_data), 2)
        self.assertEqual(req.audio_data[0], "audio1.mp3")
        self.assertEqual(req.audio_data[1], "audio2.mp3")
```
**EN:** Test handling of audio_data. This test exercises `test_audio_data_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of audio_data. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_audio_data_handling`。

### Lines 284-303: test case input ids normalization / 测试用例 input ids normalization
```python
    def test_input_ids_normalization(self):
        """Test normalization of input_ids instead of text."""
        # Test single input_ids
        req = GenerateReqInput(input_ids=[1, 2, 3])
        req.normalize_batch_and_arguments()
        self.assertTrue(req.is_single)
        self.assertEqual(req.batch_size, 1)

        # Test batch input_ids
        req = GenerateReqInput(input_ids=[[1, 2, 3], [4, 5, 6]])
        req.normalize_batch_and_arguments()
        self.assertFalse(req.is_single)
        self.assertEqual(req.batch_size, 2)

        # Test with parallel sampling
        req = GenerateReqInput(
            input_ids=[[1, 2, 3], [4, 5, 6]], sampling_params={"n": 2}
        )
        req.normalize_batch_and_arguments()
        self.assertEqual(len(req.input_ids), 4)  # 2 original * 2 parallel
```
**EN:** Test normalization of input_ids instead of text. This test exercises `test_input_ids_normalization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test normalization of input_ids instead of text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_input_ids_normalization`。

### Lines 305-317: test case input embeds normalization / 测试用例 input embeds normalization
```python
    def test_input_embeds_normalization(self):
        """Test normalization of input_embeds."""
        # Test single input_embeds
        req = GenerateReqInput(input_embeds=[[0.1, 0.2], [0.3, 0.4]])
        req.normalize_batch_and_arguments()
        self.assertTrue(req.is_single)
        self.assertEqual(req.batch_size, 1)

        # Test batch input_embeds
        req = GenerateReqInput(input_embeds=[[[0.1, 0.2]], [[0.3, 0.4]]])
        req.normalize_batch_and_arguments()
        self.assertFalse(req.is_single)
        self.assertEqual(req.batch_size, 2)
```
**EN:** Test normalization of input_embeds. This test exercises `test_input_embeds_normalization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test normalization of input_embeds. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_input_embeds_normalization`。

### Lines 319-355: test case input embeds with parallel sampling / 测试用例 input embeds with parallel sampling
```python
    def test_input_embeds_with_parallel_sampling(self):
        """Test input_embeds normalization with parallel sampling (n > 1)."""
        # Test single input_embeds with parallel sampling
        req = GenerateReqInput(
            input_embeds=[[0.1, 0.2]],  # single embedding vector
            sampling_params={"n": 2},
        )
        req.normalize_batch_and_arguments()

        # Should be converted from single to batch and then expanded
        self.assertFalse(req.is_single)
        self.assertEqual(len(req.input_embeds), 2)
        # Both should be the same input_embeds
        self.assertEqual(req.input_embeds[0], [[0.1, 0.2]])
        self.assertEqual(req.input_embeds[1], [[0.1, 0.2]])

        # Test batch input_embeds with parallel sampling
        req = GenerateReqInput(
            input_embeds=[[[0.1, 0.2]], [[0.3, 0.4]]], sampling_params={"n": 3}
        )
        req.normalize_batch_and_arguments()

        # Should be expanded
        self.assertFalse(req.is_single)
        self.assertEqual(len(req.input_embeds), 6)

        # Check that the expansion is correct
        expected_embeds = [[[0.1, 0.2]], [[0.3, 0.4]]] * 3
        self.assertEqual(req.input_embeds, expected_embeds)

        # Test with different n values per sample (should raise error)
        req = GenerateReqInput(
            input_embeds=[[[0.1, 0.2]], [[0.3, 0.4]]],
            sampling_params=[{"n": 2}, {"n": 3}],
        )
        with self.assertRaises(ValueError):
            req.normalize_batch_and_arguments()
```
**EN:** Test input_embeds normalization with parallel sampling (n > 1). This test exercises `test_input_embeds_with_parallel_sampling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test input_embeds normalization with parallel sampling (n > 1). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_input_embeds_with_parallel_sampling`。

### Lines 357-382: test case input embeds single to batch conversion / 测试用例 input embeds single to batch conversion
```python
    def test_input_embeds_single_to_batch_conversion(self):
        """Test that single input_embeds are properly converted to batch when using parallel sampling."""
        # Test the specific case that was fixed: single input_embeds with n > 1
        req = GenerateReqInput(
            input_embeds=[[0.1, 0.2, 0.3]], sampling_params={"n": 2}  # Single embedding
        )
        req.normalize_batch_and_arguments()

        # Should convert single to batch and then expand
        self.assertFalse(req.is_single)
        self.assertEqual(len(req.input_embeds), 2)

        # Both should be the same single embedding
        self.assertEqual(req.input_embeds[0], [[0.1, 0.2, 0.3]])
        self.assertEqual(req.input_embeds[1], [[0.1, 0.2, 0.3]])

        # Test with higher n value
        req = GenerateReqInput(input_embeds=[[0.1, 0.2, 0.3]], sampling_params={"n": 5})
        req.normalize_batch_and_arguments()

        self.assertFalse(req.is_single)
        self.assertEqual(len(req.input_embeds), 5)

        # All should be the same
        for i in range(5):
            self.assertEqual(req.input_embeds[i], [[0.1, 0.2, 0.3]])
```
**EN:** Test that single input_embeds are properly converted to batch when using parallel sampling. This test exercises `test_input_embeds_single_to_batch_conversion` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that single input_embeds are properly converted to batch when using parallel sampling. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_input_embeds_single_to_batch_conversion`。

### Lines 384-415: test case lora path normalization / 测试用例 lora path normalization
```python
    def test_lora_path_normalization(self):
        """Test normalization of lora_path."""
        # Test single lora_path with batch input
        req = GenerateReqInput(text=["Hello", "World"], lora_path="path/to/lora")

        # Define expected lora_paths before normalization
        expected_lora_paths = ["path/to/lora", "path/to/lora"]

        req.normalize_batch_and_arguments()
        self.assertEqual(req.lora_path, expected_lora_paths)

        # Test list of lora_paths
        req = GenerateReqInput(text=["Hello", "World"], lora_path=["path1", "path2"])

        # Define expected lora_paths before normalization
        expected_lora_paths = ["path1", "path2"]

        req.normalize_batch_and_arguments()
        self.assertEqual(req.lora_path, expected_lora_paths)

        # Test with parallel sampling
        req = GenerateReqInput(
            text=["Hello", "World"],
            lora_path=["path1", "path2"],
            sampling_params={"n": 2},
        )

        # Define expected lora_paths before normalization
        expected_lora_paths = ["path1", "path2"] * 2

        req.normalize_batch_and_arguments()
        self.assertEqual(req.lora_path, expected_lora_paths)
```
**EN:** Test normalization of lora_path. This test exercises `test_lora_path_normalization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test normalization of lora_path. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_lora_path_normalization`。

### Lines 417-461: test case logprob parameters normalization / 测试用例 logprob parameters normalization
```python
    def test_logprob_parameters_normalization(self):
        """Test normalization of logprob-related parameters."""
        # Test single example
        req = GenerateReqInput(
            text="Hello",
            return_logprob=True,
            logprob_start_len=10,
            top_logprobs_num=5,
            token_ids_logprob=[7, 8, 9],
        )
        req.normalize_batch_and_arguments()
        self.assertEqual(req.return_logprob, True)
        self.assertEqual(req.logprob_start_len, 10)
        self.assertEqual(req.top_logprobs_num, 5)
        self.assertEqual(req.token_ids_logprob, [7, 8, 9])

        # Test batch with scalar values
        req = GenerateReqInput(
            text=["Hello", "World"],
            return_logprob=True,
            logprob_start_len=10,
            top_logprobs_num=5,
            token_ids_logprob=[7, 8, 9],
        )
        req.normalize_batch_and_arguments()
        self.assertEqual(req.return_logprob, [True, True])
        self.assertEqual(req.logprob_start_len, [10, 10])
        self.assertEqual(req.top_logprobs_num, [5, 5])
        self.assertEqual(req.token_ids_logprob, [[7, 8, 9], [7, 8, 9]])

        # Test batch with list values
        req = GenerateReqInput(
            text=["Hello", "World"],
            return_logprob=[True, False],
            logprob_start_len=[10, 5],
            top_logprobs_num=[5, 3],
            token_ids_logprob=[[7, 8, 9], [4, 5, 6]],
            return_hidden_states=[False, False, True],
        )
        req.normalize_batch_and_arguments()
        self.assertEqual(req.return_logprob, [True, False])
        self.assertEqual(req.logprob_start_len, [10, 5])
        self.assertEqual(req.top_logprobs_num, [5, 3])
        self.assertEqual(req.token_ids_logprob, [[7, 8, 9], [4, 5, 6]])
        self.assertEqual(req.return_hidden_states, [False, False, True])
```
**EN:** Test normalization of logprob-related parameters. This test exercises `test_logprob_parameters_normalization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test normalization of logprob-related parameters. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_logprob_parameters_normalization`。

### Lines 463-479: test case custom logit processor normalization / 测试用例 custom logit processor normalization
```python
    def test_custom_logit_processor_normalization(self):
        """Test normalization of custom_logit_processor."""
        # Test single processor
        req = GenerateReqInput(
            text=["Hello", "World"], custom_logit_processor="serialized_processor"
        )
        req.normalize_batch_and_arguments()
        self.assertEqual(
            req.custom_logit_processor, ["serialized_processor", "serialized_processor"]
        )

        # Test list of processors
        req = GenerateReqInput(
            text=["Hello", "World"], custom_logit_processor=["processor1", "processor2"]
        )
        req.normalize_batch_and_arguments()
        self.assertEqual(req.custom_logit_processor, ["processor1", "processor2"])
```
**EN:** Test normalization of custom_logit_processor. This test exercises `test_custom_logit_processor_normalization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test normalization of custom_logit_processor. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_logit_processor_normalization`。

### Lines 481-496: test case session params handling / 测试用例 session params handling
```python
    def test_session_params_handling(self):
        """Test handling of session_params."""
        # Test with dict
        req = GenerateReqInput(
            text=["Hello", "World"], session_params={"id": "session1", "offset": 10}
        )
        req.normalize_batch_and_arguments()
        self.assertEqual(req.session_params, {"id": "session1", "offset": 10})

        # Test with list of dicts
        req = GenerateReqInput(
            text=["Hello", "World"],
            session_params=[{"id": "session1"}, {"id": "session2"}],
        )
        req.normalize_batch_and_arguments()
        self.assertEqual(req.session_params, [{"id": "session1"}, {"id": "session2"}])
```
**EN:** Test handling of session_params. This test exercises `test_session_params_handling` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test handling of session_params. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_params_handling`。

### Lines 498-535: test case getitem method / 测试用例 getitem method
```python
    def test_getitem_method(self):
        """Test the __getitem__ method."""
        req = GenerateReqInput(
            text=["Hello", "World"],
            image_data=[["img1.jpg"], ["img2.jpg"]],
            audio_data=["audio1.mp3", "audio2.mp3"],
            sampling_params=[{"temp": 0.7}, {"temp": 0.8}],
            rid=["id1", "id2"],
            return_logprob=[True, False],
            logprob_start_len=[10, 5],
            top_logprobs_num=[5, 3],
            token_ids_logprob=[[7, 8, 9], [4, 5, 6]],
            stream=True,
            log_metrics=True,
            modalities=["image", "image"],
            lora_path=["path1", "path2"],
            custom_logit_processor=["processor1", "processor2"],
            return_hidden_states=True,
        )
        req.normalize_batch_and_arguments()

        # Get the first item
        item0 = req[0]
        self.assertEqual(item0.text, "Hello")
        self.assertEqual(item0.image_data, ["img1.jpg"])
        self.assertEqual(item0.audio_data, "audio1.mp3")
        self.assertEqual(item0.sampling_params, {"temp": 0.7})
        self.assertEqual(item0.rid, "id1")
        self.assertEqual(item0.return_logprob, True)
        self.assertEqual(item0.logprob_start_len, 10)
        self.assertEqual(item0.top_logprobs_num, 5)
        self.assertEqual(item0.token_ids_logprob, [7, 8, 9])
        self.assertEqual(item0.stream, True)
        self.assertEqual(item0.log_metrics, True)
        self.assertEqual(item0.modalities, "image")
        self.assertEqual(item0.lora_path, "path1")
        self.assertEqual(item0.custom_logit_processor, "processor1")
        self.assertEqual(item0.return_hidden_states, True)
```
**EN:** Test the __getitem__ method. This test exercises `test_getitem_method` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the __getitem__ method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_getitem_method`。

### Lines 537-546: test case regenerate rid / 测试用例 regenerate rid
```python
    def test_regenerate_rid(self):
        """Test the regenerate_rid method."""
        req = GenerateReqInput(text="Hello")
        req.normalize_batch_and_arguments()

        original_rid = req.rid
        new_rid = req.regenerate_rid()

        self.assertNotEqual(original_rid, new_rid)
        self.assertEqual(req.rid, new_rid)
```
**EN:** Test the regenerate_rid method. This test exercises `test_regenerate_rid` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test the regenerate_rid method. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_regenerate_rid`。

### Lines 548-560: test case error cases / 测试用例 error cases
```python
    def test_error_cases(self):
        """Test various error cases."""
        # Test when neither text, input_ids, nor input_embeds is provided
        with self.assertRaises(ValueError):
            req = GenerateReqInput()
            req.normalize_batch_and_arguments()

        # Test when all of text, input_ids, and input_embeds are provided
        with self.assertRaises(ValueError):
            req = GenerateReqInput(
                text="Hello", input_ids=[1, 2, 3], input_embeds=[[0.1, 0.2]]
            )
            req.normalize_batch_and_arguments()
```
**EN:** Test various error cases. This test exercises `test_error_cases` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test various error cases. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_error_cases`。

### Lines 562-577: test case multiple input formats / 测试用例 multiple input formats
```python
    def test_multiple_input_formats(self):
        """Test different combinations of input formats."""
        # Test with text only
        req = GenerateReqInput(text="Hello")
        req.normalize_batch_and_arguments()
        self.assertTrue(req.is_single)

        # Test with input_ids only
        req = GenerateReqInput(input_ids=[1, 2, 3])
        req.normalize_batch_and_arguments()
        self.assertTrue(req.is_single)

        # Test with input_embeds only
        req = GenerateReqInput(input_embeds=[[0.1, 0.2]])
        req.normalize_batch_and_arguments()
        self.assertTrue(req.is_single)
```
**EN:** Test different combinations of input formats. This test exercises `test_multiple_input_formats` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test different combinations of input formats. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_input_formats`。

### Lines 580-581: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestGenerateReqInputNormalization`: Test the normalization of GenerateReqInput for batch processing and different input formats. / 用于组织相关测试、夹具或辅助方法。
- `TestGenerateReqInputNormalization.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestGenerateReqInputNormalization.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestGenerateReqInputNormalization.test_single_image_to_list_of_lists`: Test that a single image is converted to a list of single-image lists. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_image_to_list_of_lists`。
- `TestGenerateReqInputNormalization.test_list_of_images_to_list_of_lists`: Test that a list of images is converted to a list of single-image lists. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_list_of_images_to_list_of_lists`。
- `TestGenerateReqInputNormalization.test_list_of_lists_with_different_modalities`: Test handling of list of lists of images with different modalities. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_list_of_lists_with_different_modalities`。
- `TestGenerateReqInputNormalization.test_list_of_lists_with_none_values`: Test handling of list of lists with None values. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_list_of_lists_with_none_values`。
- `TestGenerateReqInputNormalization.test_expanding_parallel_sample_correlation`: Test that when expanding with parallel samples, prompts, images and modalities are properly correlated. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_expanding_parallel_sample_correlation`。
- `TestGenerateReqInputNormalization.test_specific_parallel_n_per_sample`: Test parallel expansion when different samples have different n values. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_specific_parallel_n_per_sample`。
- `TestGenerateReqInputNormalization.test_mixed_none_and_images_with_parallel_samples`: Test that when some batch items have images and others None, parallel expansion works correctly. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_none_and_images_with_parallel_samples`。
- `TestGenerateReqInputNormalization.test_correlation_with_sampling_params`: Test that sampling parameters are correctly correlated with prompts during expansion. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_correlation_with_sampling_params`。
- `TestGenerateReqInputNormalization.test_single_example_with_image`: Test handling of single example with image. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_single_example_with_image`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `copy`, `unittest`
- **Internal modules / 内部模块**: `sglang.srt.managers.io_struct`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 581
