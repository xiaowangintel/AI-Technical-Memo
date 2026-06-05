# test_vision_chunked_prefill.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/vlm/test_vision_chunked_prefill.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates vision chunked prefill behavior in SGLang's vlm area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 vlm 领域中与 vision chunked prefill 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: module imports and dependencies / 模块导入与依赖
```python
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
```
**EN:** This block imports the modules needed by the rest of the file, including `sglang.test.ci.ci_register`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `sglang.test.ci.ci_register`。

### Lines 3-8: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=156, stage="base-b", runner_config="1-gpu-large")
register_amd_ci(est_time=270, suite="stage-b-test-1-gpu-small-amd")
"""
Usage:
python3 -m unittest test_vision_chunked_prefill.TestVisionChunkedPrefill.test_chunked_prefill
"""
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 10-30: module imports and dependencies / 模块导入与依赖
```python
import io
import logging
import os
import time
import unittest
from concurrent.futures import ThreadPoolExecutor
from typing import Union

import numpy as np
import pybase64
import requests
from PIL import Image

from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    calculate_rouge_l,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `io`, `logging`, `os`, `time`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `io`, `logging`, `os`, `time`。

### Lines 31-32: supporting source context / 辅助源码上下文
```python

# Configure logging to help diagnose CI timeouts
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 33-37: module-level constants and configuration / 模块级常量与配置
```python
logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s - %(levelname)s - %(message)s",
)
logger = logging.getLogger(__name__)
```
**EN:** This block defines shared names such as `logger`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `logger` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 40-41: class TestVisionChunkedPrefill declaration / 类 TestVisionChunkedPrefill 声明
```python
class TestVisionChunkedPrefill(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 42-77: method prepare video messages / 方法 prepare video messages
```python
    def prepare_video_messages(self, video_path, max_frames_num=8):
        from sglang.srt.utils.video_decoder import VideoDecoderWrapper

        decoder = VideoDecoderWrapper(video_path)
        total_frame_num = len(decoder)
        uniform_sampled_frames = np.linspace(
            0, total_frame_num - 1, max_frames_num, dtype=int
        )
        frame_idx = uniform_sampled_frames.tolist()
        frames = decoder.get_frames_at(frame_idx)

        base64_frames = []
        for frame in frames:
            pil_img = Image.fromarray(frame)
            buff = io.BytesIO()
            pil_img.save(buff, format="JPEG")
            base64_str = pybase64.b64encode(buff.getvalue()).decode("utf-8")
            base64_frames.append(base64_str)

        messages = [{"role": "user", "content": []}]
        frame_format = {
            "type": "image_url",
            "image_url": {"url": "data:image/jpeg;base64,{}"},
            "modalities": "video",
        }

        for base64_frame in base64_frames:
            frame_format["image_url"]["url"] = "data:image/jpeg;base64,{}".format(
                base64_frame
            )
            messages[0]["content"].append(frame_format.copy())

        prompt = {"type": "text", "text": "Please describe the video briefly."}
        messages[0]["content"].append(prompt)

        return messages
```
**EN:** This block implements `prepare_video_messages` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `prepare_video_messages`，承担模块行为中的一个聚焦逻辑片段。

### Lines 79-90: helper routine get prompt from messages / 辅助流程 get prompt from messages
```python
    def get_prompt_from_messages(self, messages):
        text = (
            "<|im_start|>system\nYou are a helpful assistant.<|im_end|>\n"
            "<|im_start|>user\n"
        )
        image_data = []
        for content in messages[0]["content"]:
            if content["type"] == "image_url":
                text += "<image>\n"
                image_data.append(content["image_url"]["url"])
        text += "Please describe the video briefly.<|im_end|>\n<|im_start|>assistant\n"
        return text, image_data
```
**EN:** This helper encapsulates `get_prompt_from_messages` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `get_prompt_from_messages`，以便周围测试复用准备、执行或校验逻辑。

### Lines 92-113: method generate / 方法 generate
```python
    def generate(self, text, image_data):
        num_images = len(image_data) if image_data else 0
        logger.info(f"Starting generate request with {num_images} images")
        start_time = time.time()
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": text,
                "image_data": image_data,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 32,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
                "modalities": ["multi-images"],
            },
            timeout=120,  # Add timeout to prevent hanging indefinitely
        ).json()
        elapsed = time.time() - start_time
        logger.info(f"Generate request completed in {elapsed:.2f}s")
        return response["text"]
```
**EN:** This block implements `generate` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `generate`，承担模块行为中的一个聚焦逻辑片段。

### Lines 115-162: method generate for video / 方法 generate for video
```python
    def generate_for_video(self, batch, num_frame) -> Union[str, list[str]]:
        logger.info(
            f"generate_for_video called with batch={batch}, num_frame={num_frame}"
        )

        # prepare the video input about Steven introducing ipod nano
        url = "https://raw.githubusercontent.com/evolvinglmms-lab/sglang/dev/onevision_local/assets/jobs.mp4"
        cache_dir = os.path.expanduser("~/.cache")
        file_path = os.path.join(cache_dir, "jobs.mp4")
        os.makedirs(cache_dir, exist_ok=True)
        if not os.path.exists(file_path):
            logger.info(f"Downloading video from {url}")
            start_time = time.time()
            response = requests.get(url, timeout=60)
            response.raise_for_status()
            with open(file_path, "wb") as f:
                f.write(response.content)
            elapsed = time.time() - start_time
            logger.info(
                f"Video downloaded in {elapsed:.2f}s, size={len(response.content)} bytes"
            )
        else:
            logger.info(f"Using cached video at {file_path}")

        if not batch:
            assert isinstance(num_frame, int)
            logger.info(f"Processing single video with {num_frame} frames")
            messages = self.prepare_video_messages(file_path, max_frames_num=num_frame)
            text, image_data = self.get_prompt_from_messages(messages)
            return self.generate(text, image_data)
        else:
            assert isinstance(num_frame, list)
            logger.info(f"Processing batch of videos with frame counts: {num_frame}")
            func_args = []
            for max_frames_num in num_frame:
                messages = self.prepare_video_messages(
                    file_path,
                    max_frames_num=max_frames_num,
                )
                text, image_data = self.get_prompt_from_messages(messages)
                func_args.append((text, image_data))

            logger.info(f"Starting batch generation with {len(func_args)} requests")
            with ThreadPoolExecutor(max_workers=10) as executor:
                responses = list(executor.map(lambda p: self.generate(*p), func_args))
            logger.info(f"Batch generation completed")

            return responses
```
**EN:** This block implements `generate_for_video` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `generate_for_video`，承担模块行为中的一个聚焦逻辑片段。

### Lines 164-178: helper routine launch server / 辅助流程 launch server
```python
    def launch_server(self, chunked_prefill_size) -> int:
        # launch server
        model = "lmms-lab/llava-onevision-qwen2-7b-ov"
        # model = "meta-llama/Llama-3.2-11B-Vision-Instruct"
        self.base_url = DEFAULT_URL_FOR_TEST
        process = popen_launch_server(
            model,
            self.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--chunked-prefill-size",
                f"{chunked_prefill_size}",
            ],
        )
        return process.pid
```
**EN:** This helper encapsulates `launch_server` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `launch_server`，以便周围测试复用准备、执行或校验逻辑。

### Lines 180-242: method test chunked prefill / 方法 test chunked prefill
```python
    def _test_chunked_prefill(self, batches, num_frames):
        logger.info("=" * 60)
        logger.info("Starting chunked prefill test")
        logger.info("=" * 60)

        # Chunked
        logger.info("Phase 1: Testing with chunked_prefill_size=1024")
        chunked_server_pid = self.launch_server(chunked_prefill_size=1024)
        logger.info(f"Chunked server started with pid={chunked_server_pid}")
        try:
            outputs_chunked = []
            for i, (batch, num_frame) in enumerate(zip(batches, num_frames)):
                logger.info(f"Chunked test iteration {i+1}/{len(batches)}")
                output_chunked = self.generate_for_video(
                    batch=batch, num_frame=num_frame
                )
                outputs_chunked += [output_chunked]
                logger.info(f"Chunked test iteration {i+1} completed")
        finally:
            logger.info(f"Killing chunked server pid={chunked_server_pid}")
            kill_process_tree(chunked_server_pid)
            logger.info("Chunked server killed")
            time.sleep(4)

        # None-chunked
        logger.info("Phase 2: Testing with chunked_prefill_size=-1 (no chunking)")
        try:
            no_chunked_server_pid = self.launch_server(chunked_prefill_size=-1)
            logger.info(f"Non-chunked server started with pid={no_chunked_server_pid}")
            outputs_no_chunked = []
            for i, (batch, num_frame) in enumerate(zip(batches, num_frames)):
                logger.info(f"Non-chunked test iteration {i+1}/{len(batches)}")
                output_no_chunked = self.generate_for_video(
                    batch=batch, num_frame=num_frame
                )
                outputs_no_chunked += [output_no_chunked]
                logger.info(f"Non-chunked test iteration {i+1} completed")

        finally:
            logger.info(f"Killing non-chunked server pid={no_chunked_server_pid}")
            kill_process_tree(no_chunked_server_pid)
            logger.info("Non-chunked server killed")
            time.sleep(4)

        for output_chunked, output_no_chunked in zip(
            outputs_chunked, outputs_no_chunked
        ):
            print("output with chunked prefill:")
            print(output_chunked)
            print("output without chunked prefill:")
            print(output_no_chunked)
            self.assertEqual(len(output_chunked), len(output_no_chunked))
            rouge_scores = calculate_rouge_l(output_chunked, output_no_chunked)
            avg_score = sum(rouge_scores) / len(rouge_scores)
            print(f"ROUGE-L scores: {rouge_scores}")
            print(f"Average ROUGE-L score: {avg_score:.4f}")
            # Allow for occasional divergence in one item while maintaining overall output quality
            self.assertGreater(
                avg_score,
                0.90,
                f"Average ROUGE-L score too low: {avg_score:.4f}. "
                f"Individual scores: {rouge_scores}",
            )
```
**EN:** This block implements `_test_chunked_prefill` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_test_chunked_prefill`，承担模块行为中的一个聚焦逻辑片段。

### Lines 244-245: test case chunked prefill / 测试用例 chunked prefill
```python
    def test_chunked_prefill(self):
        self._test_chunked_prefill(batches=[False, True], num_frames=[1, [2, 6, 8, 10]])
```
**EN:** This test exercises `test_chunked_prefill` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chunked_prefill`。

### Lines 248-249: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestVisionChunkedPrefill`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestVisionChunkedPrefill.prepare_video_messages`: This block implements `prepare_video_messages` and captures one focused piece of the module's behavior. / 该代码块实现 `prepare_video_messages`，承担模块行为中的一个聚焦逻辑片段。
- `TestVisionChunkedPrefill.get_prompt_from_messages`: This helper encapsulates `get_prompt_from_messages` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `get_prompt_from_messages`，以便周围测试复用准备、执行或校验逻辑。
- `TestVisionChunkedPrefill.generate`: This block implements `generate` and captures one focused piece of the module's behavior. / 该代码块实现 `generate`，承担模块行为中的一个聚焦逻辑片段。
- `TestVisionChunkedPrefill.generate_for_video`: This block implements `generate_for_video` and captures one focused piece of the module's behavior. / 该代码块实现 `generate_for_video`，承担模块行为中的一个聚焦逻辑片段。
- `TestVisionChunkedPrefill.launch_server`: This helper encapsulates `launch_server` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `launch_server`，以便周围测试复用准备、执行或校验逻辑。
- `TestVisionChunkedPrefill._test_chunked_prefill`: This block implements `_test_chunked_prefill` and captures one focused piece of the module's behavior. / 该代码块实现 `_test_chunked_prefill`，承担模块行为中的一个聚焦逻辑片段。
- `TestVisionChunkedPrefill.test_chunked_prefill`: This test exercises `test_chunked_prefill` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chunked_prefill`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `io`, `logging`, `os`, `time`, `unittest`, `concurrent.futures`, `typing`
- **Third-party modules / 第三方模块**: `numpy`, `pybase64`, `requests`, `PIL`
- **Internal modules / 内部模块**: `sglang.test.ci.ci_register`, `sglang.srt.utils`, `sglang.test.test_utils`

- **Total lines / 总行数**: 249
