# test_epd_disaggregation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/distributed/test_epd_disaggregation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates epd disaggregation behavior in SGLang's distributed area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 分布式 领域中与 epd disaggregation 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-32: module imports and dependencies / 模块导入与依赖
```python
import io
import os
import re
import subprocess
import threading
import time
import unittest

import grpc
import openai
import zmq
from grpc_health.v1 import health_pb2, health_pb2_grpc

from sglang.srt.utils import kill_process_tree
from sglang.srt.utils.network import get_zmq_socket_on_host
from sglang.test.ci.ci_register import register_cuda_ci
from sglang.test.kits.mmmu_vlm_kit import MMMUMixin
from sglang.test.server_fixtures.disaggregation_fixture import (
    PDDisaggregationServerBase,
)
from sglang.test.test_utils import (
    DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    is_in_ci,
    popen_launch_server,
)
from sglang.test.vlm_utils import (
    AUDIO_TRUMP_SPEECH_URL,
    IMAGE_MAN_IRONING_URL,
    IMAGE_SGL_LOGO_URL,
    VIDEO_JOBS_URL,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `io`, `os`, `re`, `subprocess`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `io`, `os`, `re`, `subprocess`。

### Lines 33-34: supporting source context / 辅助源码上下文
```python

# Omni model for local testing; override via env var EPD_OMNI_MODEL
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 35-39: CI registration and metadata / CI 注册与元数据
```python
DEFAULT_OMNI_MODEL = "Qwen/Qwen3-Omni-30B-A3B-Instruct"
QWEN35_27B_MODEL = "Qwen/Qwen3.5-27B"


register_cuda_ci(est_time=97, suite="nightly-4-gpu", nightly=True)
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci.
**CN:** 该代码块通过 register_cuda_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 42-46: class TestEPDDisaggregationOmni declaration / 类 TestEPDDisaggregationOmni 声明
```python
@unittest.skipIf(
    is_in_ci(),
    "Omni model EPD test with image, video, and audio modalities, running locally only",
)
class TestEPDDisaggregationOmni(PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PDDisaggregationServerBase`。

### Lines 47-51: supporting statements / 辅助语句
```python
    """
    EPD disaggregation test for omni models (e.g. Qwen3-Omni). Covers image, video,
    and audio when server_type=http (encoder_transfer_backend: mooncake/zmq_to_scheduler/zmq_to_tokenizer).
    When server_type=grpc, only image is tested (gRPC encode is image-only).
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 53-115: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = os.environ.get("EPD_OMNI_MODEL", DEFAULT_OMNI_MODEL)
        cls.server_type = os.environ.get("EPD_ENCODE_SERVER_TYPE", "http")
        assert cls.server_type in (
            "grpc",
            "http",
        ), f"Invalid EPD_ENCODE_SERVER_TYPE: {cls.server_type}"
        cls.encoder_transfer_backend = os.environ.get(
            "EPD_ENCODER_TRANSFER_BACKEND", "zmq_to_scheduler"
        )
        assert cls.encoder_transfer_backend in (
            "mooncake",
            "zmq_to_scheduler",
            "zmq_to_tokenizer",
        ), f"Invalid EPD_ENCODER_TRANSFER_BACKEND: {cls.encoder_transfer_backend}"
        cls.enable_global_cache = (
            os.environ.get("MOONCAKE_MASTER") is not None
            or os.environ.get("MOONCAKE_CLIENT") is not None
        )
        if cls.server_type == "grpc":
            cls.encode_port = f"{int(cls.lb_port) + 305}"
            cls.encode_url = f"grpc://{cls.base_host}:{cls.encode_port}"
        else:
            cls.encode_port = f"{int(cls.lb_port) + 300}"
            cls.encode_url = f"http://{cls.base_host}:{cls.encode_port}"

        cls.image_man_ironing = IMAGE_MAN_IRONING_URL
        cls.image_sgl_logo = IMAGE_SGL_LOGO_URL
        cls.video_jobs = VIDEO_JOBS_URL
        cls.audio_trump = AUDIO_TRUMP_SPEECH_URL

        print(
            f"Setting up EPD Omni: model={cls.model}, encode={cls.encode_port}, "
            f"prefill={cls.prefill_port}, decode={cls.decode_port}, "
            f"server_type={cls.server_type}, backend={cls.encoder_transfer_backend}, "
            f"global_cache={cls.enable_global_cache}"
        )
        print(f"Data URLs: image={cls.image_man_ironing}, audio={cls.audio_trump}")

        cls.start_encode()
        prefill_thread = threading.Thread(target=cls.start_prefill)
        decode_thread = threading.Thread(target=cls.start_decode)
        prefill_thread.start()
        decode_thread.start()
        prefill_thread.join()
        decode_thread.join()

        if cls.server_type == "grpc":
            cls._wait_grpc_ready(cls.base_host, cls.encode_port, cls.process_encode)
        else:
            cls.wait_server_ready(
                cls.encode_url + "/health", process=cls.process_encode
            )
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()

        cls.api_key = "sk-123456"
        os.environ["OPENAI_API_KEY"] = cls.api_key
        os.environ["OPENAI_API_BASE"] = f"{cls.lb_url}/v1"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 117-163: helper routine start encode / 辅助流程 start encode
```python
    @classmethod
    def start_encode(cls):
        if cls.server_type == "grpc":
            cls.encode_stdout = io.StringIO()
            cls.encode_stderr = io.StringIO()
            cls.process_encode = subprocess.Popen(
                [
                    "python3",
                    "-m",
                    "sglang.launch_server",
                    "--model-path",
                    cls.model,
                    "--host",
                    cls.base_host,
                    "--port",
                    cls.encode_port,
                    "--trust-remote-code",
                    "--encoder-only",
                    "--grpc-mode",
                    "--encoder-transfer-backend",
                    "zmq_to_scheduler",
                    "--tp",
                    "1",
                ]
            )
        else:
            encode_args = [
                "--trust-remote-code",
                "--encoder-only",
                "--encoder-transfer-backend",
                cls.encoder_transfer_backend,
                "--tp",
                "1",
                "--port",
                cls.encode_port,
            ]
            if cls.enable_global_cache:
                encode_args.append("--enable-mm-global-cache")
            cls.encode_stdout = io.StringIO()
            cls.encode_stderr = io.StringIO()
            cls.process_encode = popen_launch_server(
                cls.model,
                base_url=cls.encode_url,
                timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
                other_args=encode_args,
                return_stdout_stderr=(cls.encode_stdout, cls.encode_stderr),
            )
```
**EN:** This helper encapsulates `start_encode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_encode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 165-199: helper routine start prefill / 辅助流程 start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--language-only",
            "--encoder-urls",
            cls.encode_url,
            "--encoder-transfer-backend",
            (
                "zmq_to_scheduler"
                if cls.server_type == "grpc"
                else cls.encoder_transfer_backend
            ),
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--base-gpu-id",
            "1",
            "--port",
            cls.prefill_port,
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        prefill_env = os.environ.copy()
        if cls.server_type == "grpc":
            prefill_env["SGLANG_ENCODER_MM_RECEIVER_MODE"] = "grpc"
        cls.process_prefill = popen_launch_server(
            cls.model,
            base_url=cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
            env=prefill_env,
        )
```
**EN:** This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。

### Lines 201-222: helper routine start decode / 辅助流程 start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--base-gpu-id",
            "2",
            "--port",
            cls.decode_port,
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_server(
            cls.model,
            base_url=cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 224-236: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        for process in [
            cls.process_lb,
            cls.process_decode,
            cls.process_prefill,
            cls.process_encode,
        ]:
            if process:
                try:
                    kill_process_tree(process.pid)
                except Exception as e:
                    print(f"Error killing process: {e}")
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 238-262: method wait grpc ready / 方法 wait grpc ready
```python
    @staticmethod
    def _wait_grpc_ready(
        host, port, process, timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH
    ):
        deadline = time.time() + timeout
        channel = grpc.insecure_channel(f"{host}:{port}")
        stub = health_pb2_grpc.HealthStub(channel)
        try:
            while time.time() < deadline:
                if process.poll() is not None:
                    raise RuntimeError(
                        f"gRPC encoder exited with code {process.returncode}"
                    )
                try:
                    response = stub.Check(
                        health_pb2.HealthCheckRequest(service=""), timeout=2
                    )
                    if response.status == health_pb2.HealthCheckResponse.SERVING:
                        return
                except grpc.RpcError:
                    pass
                time.sleep(1)
        finally:
            channel.close()
        raise RuntimeError(f"gRPC encoder not ready at {host}:{port} within {timeout}s")
```
**EN:** This block implements `_wait_grpc_ready` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_wait_grpc_ready`，承担模块行为中的一个聚焦逻辑片段。

### Lines 263-265: supporting source context / 辅助源码上下文
```python

    # ---- helpers ----

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 266-267: method client / 方法 client
```python
    def _client(self):
        return openai.Client(api_key=self.api_key, base_url=f"{self.lb_url}/v1")
```
**EN:** This block implements `_client` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_client`，承担模块行为中的一个聚焦逻辑片段。

### Lines 269-272: method skip if grpc / 方法 skip if grpc
```python
    def _skip_if_grpc(self, msg="gRPC encode is image-only"):
        """Skip this test when encode server is gRPC (image-only)."""
        if self.server_type == "grpc":
            self.skipTest(msg)
```
**EN:** Skip this test when encode server is gRPC (image-only). This block implements `_skip_if_grpc` and captures one focused piece of the module's behavior.
**CN:** Skip this test when encode server is gRPC (image-only). 该代码块实现 `_skip_if_grpc`，承担模块行为中的一个聚焦逻辑片段。

### Lines 274-284: method parse cache log / 方法 parse cache log
```python
    def _parse_cache_log(self):
        """Parse encode server logs and return list of (local_hits, global_hits, misses)
        tuples from '=== Multi-Level Cache Check ===' lines."""
        log = self.encode_stdout.getvalue() + self.encode_stderr.getvalue()
        pattern = re.compile(
            r"Multi-Level Cache Check.*?"
            r"Local Hits:\s*(\d+).*?"
            r"Global Hits:\s*(\d+).*?"
            r"Misses.*?:\s*(\d+)"
        )
        return [(int(m[1]), int(m[2]), int(m[3])) for m in pattern.finditer(log)]
```
**EN:** Parse encode server logs and return list of (local_hits, global_hits, misses) tuples from '=== Multi-Level Cache Check ===' lines. This block implements `_parse_cache_log` and captures one focused piece of the module's behavior.
**CN:** Parse encode server logs and return list of (local_hits, global_hits, misses) tuples from '=== Multi-Level Cache Check ===' lines. 该代码块实现 `_parse_cache_log`，承担模块行为中的一个聚焦逻辑片段。

### Lines 285-286: supporting source context / 辅助源码上下文
```python

    # ---- image ----
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 287-322: test case image / 测试用例 image
```python
    def test_image(self):
        client = self._client()
        response = client.chat.completions.create(
            model="default",
            messages=[
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "image_url",
                            "image_url": {"url": self.image_man_ironing},
                        },
                        {
                            "type": "text",
                            "text": "Describe this image in a sentence.",
                        },
                    ],
                },
            ],
            temperature=0,
            max_tokens=256,
        )
        text = response.choices[0].message.content
        print(f"[Omni EPD] Image response:\n{text}")
        self.assertIsNotNone(text)
        self.assertGreater(len(text), 0)

        text_lower = text.lower()
        self.assertTrue(
            any(w in text_lower for w in ("man", "person", "driver")),
            f"Image response should mention a person: {text}",
        )
        self.assertTrue(
            any(w in text_lower for w in ("iron", "cloth", "hang", "holding")),
            f"Image response should mention ironing/clothes: {text}",
        )
```
**EN:** This test exercises `test_image` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_image`。

### Lines 324-368: test case image cache hit / 测试用例 image cache hit
```python
    def test_image_cache_hit(self):
        """Send the same image twice; the second request should hit the global-mm-cache."""
        self._skip_if_grpc("gRPC encode is image-only; cache test uses HTTP path")
        if not self.enable_global_cache:
            self.skipTest("global-mm-cache not enabled (MOONCAKE_MASTER not set)")
        client = self._client()
        baseline = len(self._parse_cache_log())
        for i in range(2):
            response = client.chat.completions.create(
                model="default",
                messages=[
                    {
                        "role": "user",
                        "content": [
                            {
                                "type": "image_url",
                                "image_url": {"url": self.image_sgl_logo},
                            },
                            {
                                "type": "text",
                                "text": "What is shown in this image?",
                            },
                        ],
                    },
                ],
                temperature=0,
                max_tokens=128,
            )
            text = response.choices[0].message.content
            print(f"[Omni EPD] Image cache-hit round {i}: {text}")
            self.assertIsNotNone(text)
            self.assertGreater(len(text), 0)
            time.sleep(1)

        entries = self._parse_cache_log()[baseline:]
        print(f"[Omni EPD] Image cache log entries: {entries}")
        self.assertGreaterEqual(
            len(entries), 2, "Expected at least 2 cache-check log entries"
        )
        local_hits, global_hits, _ = entries[-1]
        self.assertGreater(
            local_hits + global_hits,
            0,
            f"Second image request should have cache hits, got: {entries[-1]}",
        )
```
**EN:** Send the same image twice; the second request should hit the global-mm-cache. This test exercises `test_image_cache_hit` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Send the same image twice; the second request should hit the global-mm-cache. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_image_cache_hit`。

### Lines 369-370: supporting source context / 辅助源码上下文
```python

    # ---- video ----
```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 371-438: test case video / 测试用例 video
```python
    def test_video(self):
        self._skip_if_grpc()
        client = self._client()
        response = client.chat.completions.create(
            model="default",
            messages=[
                {
                    "role": "user",
                    "content": [
                        {"type": "text", "text": "Describe the video."},
                        {
                            "type": "video_url",
                            "video_url": {"url": self.video_jobs},
                        },
                    ],
                },
            ],
            max_tokens=8192,
            stream=False,
        )
        text = response.choices[0].message.content
        print(f"[Omni EPD] Video response:\n{text}")
        self.assertIsNotNone(text)
        self.assertGreater(len(text), 0)

        text_lower = text.lower()
        self.assertTrue(
            any(
                w in text_lower
                for w in ("ipod", "device", "microphone", "smartphone", "phone")
            ),
            f"Video response should mention a device: {text}",
        )
        self.assertTrue(
            any(
                w in text_lower
                for w in (
                    "man",
                    "person",
                    "individual",
                    "speaker",
                    "presenter",
                    "steve",
                    "hand",
                    "hands",
                )
            ),
            f"Video response should mention a person: {text}",
        )
        self.assertTrue(
            any(
                w in text_lower
                for w in (
                    "present",
                    "presenting",
                    "examine",
                    "examining",
                    "display",
                    "displaying",
                    "hold",
                    "holding",
                    "gestur",
                    "speak",
                    "speaking",
                )
            ),
            f"Video response should mention an action: {text}",
        )
```
**EN:** This test exercises `test_video` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_video`。

### Lines 440-481: test case video cache hit / 测试用例 video cache hit
```python
    def test_video_cache_hit(self):
        """Send the same video twice; the second request should hit the global-mm-cache."""
        self._skip_if_grpc()
        if not self.enable_global_cache:
            self.skipTest("global-mm-cache not enabled (MOONCAKE_MASTER not set)")
        client = self._client()
        baseline = len(self._parse_cache_log())
        for i in range(2):
            response = client.chat.completions.create(
                model="default",
                messages=[
                    {
                        "role": "user",
                        "content": [
                            {"type": "text", "text": "Describe the video."},
                            {
                                "type": "video_url",
                                "video_url": {"url": self.video_jobs},
                            },
                        ],
                    },
                ],
                max_tokens=256,
                stream=False,
            )
            text = response.choices[0].message.content
            print(f"[Omni EPD] Video cache-hit round {i}: {text}")
            self.assertIsNotNone(text)
            self.assertGreater(len(text), 0)
            time.sleep(1)

        entries = self._parse_cache_log()[baseline:]
        print(f"[Omni EPD] Video cache log entries: {entries}")
        self.assertGreaterEqual(
            len(entries), 2, "Expected at least 2 cache-check log entries"
        )
        local_hits, global_hits, _ = entries[-1]
        self.assertGreater(
            local_hits + global_hits,
            0,
            f"Second video request should have cache hits, got: {entries[-1]}",
        )
```
**EN:** Send the same video twice; the second request should hit the global-mm-cache. This test exercises `test_video_cache_hit` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Send the same video twice; the second request should hit the global-mm-cache. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_video_cache_hit`。

### Lines 482-484: supporting source context / 辅助源码上下文
```python

    # ---- audio ----

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 485-520: test case audio / 测试用例 audio
```python
    def test_audio(self):
        self._skip_if_grpc()
        client = self._client()
        response = client.chat.completions.create(
            model="default",
            messages=[
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "audio_url",
                            "audio_url": {"url": self.audio_trump},
                        },
                        {
                            "type": "text",
                            "text": "Listen to this audio and write down the audio transcription in English.",
                        },
                    ],
                },
            ],
            temperature=0,
            max_tokens=256,
            stream=False,
        )
        text = response.choices[0].message.content
        print(f"[Omni EPD] Audio response:\n{text}")
        self.assertIsNotNone(text)
        self.assertGreater(len(text), 0)

        text_lower = text.lower()
        for keyword in ("thank you", "leader"):
            self.assertIn(
                keyword,
                text_lower,
                f"Audio response should contain '{keyword}': {text}",
            )
```
**EN:** This test exercises `test_audio` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_audio`。

### Lines 522-567: test case audio cache hit / 测试用例 audio cache hit
```python
    def test_audio_cache_hit(self):
        """Send the same audio twice; the second request should hit the global-mm-cache."""
        self._skip_if_grpc()
        if not self.enable_global_cache:
            self.skipTest("global-mm-cache not enabled (MOONCAKE_MASTER not set)")
        client = self._client()
        baseline = len(self._parse_cache_log())
        for i in range(2):
            response = client.chat.completions.create(
                model="default",
                messages=[
                    {
                        "role": "user",
                        "content": [
                            {
                                "type": "audio_url",
                                "audio_url": {"url": self.audio_trump},
                            },
                            {
                                "type": "text",
                                "text": "What is this audio about?",
                            },
                        ],
                    },
                ],
                temperature=0,
                max_tokens=128,
                stream=False,
            )
            text = response.choices[0].message.content
            print(f"[Omni EPD] Audio cache-hit round {i}: {text}")
            self.assertIsNotNone(text)
            self.assertGreater(len(text), 0)
            time.sleep(1)

        entries = self._parse_cache_log()[baseline:]
        print(f"[Omni EPD] Audio cache log entries: {entries}")
        self.assertGreaterEqual(
            len(entries), 2, "Expected at least 2 cache-check log entries"
        )
        local_hits, global_hits, _ = entries[-1]
        self.assertGreater(
            local_hits + global_hits,
            0,
            f"Second audio request should have cache hits, got: {entries[-1]}",
        )
```
**EN:** Send the same audio twice; the second request should hit the global-mm-cache. This test exercises `test_audio_cache_hit` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Send the same audio twice; the second request should hit the global-mm-cache. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_audio_cache_hit`。

### Lines 568-570: supporting source context / 辅助源码上下文
```python

    # ---- mixed modality ----

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 571-618: test case mixed image audio video / 测试用例 mixed image audio video
```python
    def test_mixed_image_audio_video(self):
        """Image + audio + video in one request to test multi-modal routing."""
        self._skip_if_grpc()
        client = self._client()
        response = client.chat.completions.create(
            model="default",
            messages=[
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "image_url",
                            "image_url": {"url": self.image_man_ironing},
                        },
                        {
                            "type": "audio_url",
                            "audio_url": {"url": self.audio_trump},
                        },
                        {
                            "type": "video_url",
                            "video_url": {"url": self.video_jobs},
                        },
                        {
                            "type": "text",
                            "text": (
                                "I have an image, an audio clip, and a video, which are not related at all. "
                                "Please: 1. Describe the image in a sentence, "
                                "2. Summarize the audio content briefly, "
                                "3. Describe what happens in the video."
                            ),
                        },
                    ],
                },
            ],
            temperature=0,
            max_tokens=512,
            stream=False,
        )
        text = response.choices[0].message.content
        print(f"[Omni EPD] Mixed image+audio+video response:\n{text}")
        self.assertIsNotNone(text)
        self.assertGreater(len(text), 0)

        text_lower = text.lower()
        self.assertTrue(
            any(w in text_lower for w in ("man", "person", "iron", "cloth")),
            f"Mixed response should describe the image: {text}",
        )
```
**EN:** Image + audio + video in one request to test multi-modal routing. This test exercises `test_mixed_image_audio_video` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Image + audio + video in one request to test multi-modal routing. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mixed_image_audio_video`。

### Lines 621-622: class TestEPDDisaggregationOneEncoder declaration / 类 TestEPDDisaggregationOneEncoder 声明
```python
@unittest.skipIf(is_in_ci(), "Skipping in CI to reduce multi-GPU runtime")
class TestEPDDisaggregationOneEncoder(MMMUMixin, PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `MMMUMixin`, `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `MMMUMixin`, `PDDisaggregationServerBase`。

### Lines 623-627: class-level constants and configuration for `TestEPDDisaggregationOneEncoder` / 类级常量与配置
```python
    """Test EPD disaggregation with single encode server"""

    # Qwen2.5-VL-3B-Instruct scores ~0.40 on the 50-sample MMMU subset.
    accuracy = 0.40
    mmmu_args = ["--limit", "50"]
```
**EN:** This block defines shared names such as `accuracy`, `mmmu_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `accuracy`, `mmmu_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 629-661: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST
        cls.base_url = cls.lb_url  # MMMUMixin reads this for OPENAI_API_BASE
        cls.encode_port = f"{int(cls.lb_port) + 300}"
        cls.encode_url = f"http://{cls.base_host}:{cls.encode_port}"

        print(
            f"Setting up EPD (one encoder): encode={cls.encode_port}, "
            f"prefill={cls.prefill_port}, decode={cls.decode_port}"
        )

        # Start servers in order: encode -> prefill/decode
        cls.start_encode()
        prefill_thread = threading.Thread(target=cls.start_prefill)
        decode_thread = threading.Thread(target=cls.start_decode)
        prefill_thread.start()
        decode_thread.start()
        prefill_thread.join()
        decode_thread.join()

        # Wait for all servers to be ready
        cls.wait_server_ready(cls.encode_url + "/health", process=cls.process_encode)
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()

        # Set OpenAI API key and base URL environment variables. Needed for lmms-eval to work.
        cls.api_key = "sk-123456"
        os.environ["OPENAI_API_KEY"] = cls.api_key
        os.environ["OPENAI_API_BASE"] = f"{cls.lb_url}/v1"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 663-682: helper routine start encode / 辅助流程 start encode
```python
    @classmethod
    def start_encode(cls):
        """Start encode server for multimodal processing"""
        encode_args = [
            "--trust-remote-code",
            "--encoder-only",
            "--encoder-transfer-backend",
            "zmq_to_scheduler",
            "--tp",
            "1",
            "--port",
            cls.encode_port,
            "--enable-prefix-mm-cache",
        ]
        cls.process_encode = popen_launch_server(
            cls.model,
            base_url=cls.encode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=encode_args,
        )
```
**EN:** Start encode server for multimodal processing This helper encapsulates `start_encode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Start encode server for multimodal processing 该辅助函数封装了 `start_encode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 684-711: helper routine start prefill / 辅助流程 start prefill
```python
    @classmethod
    def start_prefill(cls):
        """Start prefill server with language model only"""
        prefill_args = [
            "--trust-remote-code",
            "--language-only",
            "--encoder-urls",
            cls.encode_url,
            "--encoder-transfer-backend",
            "zmq_to_scheduler",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--base-gpu-id",
            "1",
            "--port",
            cls.prefill_port,
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_server(
            cls.model,
            base_url=cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** Start prefill server with language model only This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Start prefill server with language model only 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。

### Lines 713-735: helper routine start decode / 辅助流程 start decode
```python
    @classmethod
    def start_decode(cls):
        """Start decode server"""
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--base-gpu-id",
            "2",
            "--port",
            cls.decode_port,
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_server(
            cls.model,
            base_url=cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** Start decode server This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Start decode server 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 737-750: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        """Clean up all processes"""
        for process in [
            cls.process_lb,
            cls.process_decode,
            cls.process_prefill,
            cls.process_encode,
        ]:
            if process:
                try:
                    kill_process_tree(process.pid)
                except Exception as e:
                    print(f"Error killing process: {e}")
```
**EN:** Clean up all processes This routine releases resources and restores state after the related tests finish.
**CN:** Clean up all processes 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 753-757: class TestEPDDisaggregationQwen35 declaration / 类 TestEPDDisaggregationQwen35 声明
```python
@unittest.skipIf(
    is_in_ci(),
    "Qwen3.5 EPD image/video test runs locally only",
)
class TestEPDDisaggregationQwen35(PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PDDisaggregationServerBase`。

### Lines 758-758: supporting statements / 辅助语句
```python
    """EPD disaggregation test for Qwen3.5 image and video requests."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 760-782: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.process_encode = None
        cls.model = QWEN35_27B_MODEL
        cls.encode_port = f"{int(cls.lb_port) + 300}"
        cls.encode_url = f"http://{cls.base_host}:{cls.encode_port}"
        cls.language_url = cls.prefill_url
        cls.image_man_ironing = IMAGE_MAN_IRONING_URL
        cls.video_jobs = VIDEO_JOBS_URL

        print(
            f"Setting up Qwen3.5 encoder disaggregation: model={cls.model}, "
            f"encode={cls.encode_port}, language={cls.prefill_port}"
        )

        cls.start_encode()
        cls.start_prefill()

        cls.wait_server_ready(cls.encode_url + "/health", process=cls.process_encode)
        cls.wait_server_ready(cls.language_url + "/health", process=cls.process_prefill)

        cls.api_key = "sk-123456"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 784-805: helper routine start encode / 辅助流程 start encode
```python
    @classmethod
    def start_encode(cls):
        encode_args = [
            "--trust-remote-code",
            "--encoder-only",
            "--encoder-transfer-backend",
            "zmq_to_scheduler",
            "--tp",
            "1",
            "--port",
            cls.encode_port,
            "--reasoning-parser",
            "qwen3",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true,"num_threads": 64}',
        ]
        cls.process_encode = popen_launch_server(
            cls.model,
            base_url=cls.encode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=encode_args,
        )
```
**EN:** This helper encapsulates `start_encode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_encode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 807-832: helper routine start prefill / 辅助流程 start prefill
```python
    @classmethod
    def start_prefill(cls):
        language_args = [
            "--trust-remote-code",
            "--language-only",
            "--encoder-urls",
            cls.encode_url,
            "--encoder-transfer-backend",
            "zmq_to_scheduler",
            "--tp",
            "1",
            "--base-gpu-id",
            "1",
            "--port",
            cls.prefill_port,
            "--reasoning-parser",
            "qwen3",
            "--model-loader-extra-config",
            '{"enable_multithread_load": true,"num_threads": 64}',
        ]
        cls.process_prefill = popen_launch_server(
            cls.model,
            base_url=cls.language_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=language_args,
        )
```
**EN:** This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。

### Lines 834-843: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        if cls.process_lb:
            kill_process_tree(cls.process_lb.pid)
        if cls.process_decode:
            kill_process_tree(cls.process_decode.pid)
        if cls.process_prefill:
            kill_process_tree(cls.process_prefill.pid)
        if cls.process_encode:
            kill_process_tree(cls.process_encode.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 845-846: method client / 方法 client
```python
    def _client(self):
        return openai.Client(api_key=self.api_key, base_url=f"{self.language_url}/v1")
```
**EN:** This block implements `_client` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_client`，承担模块行为中的一个聚焦逻辑片段。

### Lines 848-884: test case image / 测试用例 image
```python
    def test_image(self):
        client = self._client()
        response = client.chat.completions.create(
            model="default",
            messages=[
                {
                    "role": "user",
                    "content": [
                        {
                            "type": "image_url",
                            "image_url": {"url": self.image_man_ironing},
                        },
                        {
                            "type": "text",
                            "text": "Describe this image in a sentence.",
                        },
                    ],
                },
            ],
            temperature=0,
            max_tokens=256,
            extra_body={"reasoning_effort": "none"},
        )
        text = response.choices[0].message.content
        print(f"[Qwen3.5 EPD] Image response:\n{text}")
        self.assertIsNotNone(text)
        self.assertGreater(len(text), 0)

        text_lower = text.lower()
        self.assertTrue(
            any(w in text_lower for w in ("man", "person", "driver")),
            f"Image response should mention a person: {text}",
        )
        self.assertTrue(
            any(w in text_lower for w in ("iron", "cloth", "hang", "holding")),
            f"Image response should mention ironing/clothes: {text}",
        )
```
**EN:** This test exercises `test_image` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_image`。

### Lines 886-933: test case video / 测试用例 video
```python
    def test_video(self):
        client = self._client()
        response = client.chat.completions.create(
            model="default",
            messages=[
                {
                    "role": "user",
                    "content": [
                        {"type": "text", "text": "Describe the video."},
                        {
                            "type": "video_url",
                            "video_url": {"url": self.video_jobs},
                        },
                    ],
                },
            ],
            max_tokens=1024,
            stream=False,
        )
        text = response.choices[0].message.content
        print(f"[Qwen3.5 EPD] Video response:\n{text}")
        self.assertIsNotNone(text)
        self.assertGreater(len(text), 0)

        text_lower = text.lower()
        self.assertTrue(
            any(
                w in text_lower
                for w in ("ipod", "device", "microphone", "smartphone", "phone")
            ),
            f"Video response should mention a device: {text}",
        )
        self.assertTrue(
            any(
                w in text_lower
                for w in (
                    "man",
                    "person",
                    "individual",
                    "speaker",
                    "presenter",
                    "steve",
                    "hand",
                    "hands",
                )
            ),
            f"Video response should mention a person: {text}",
        )
```
**EN:** This test exercises `test_video` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_video`。

### Lines 936-936: class TestEPDDisaggregationMultiEncoders declaration / 类 TestEPDDisaggregationMultiEncoders 声明
```python
class TestEPDDisaggregationMultiEncoders(MMMUMixin, PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `MMMUMixin`, `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `MMMUMixin`, `PDDisaggregationServerBase`。

### Lines 937-944: class-level constants and configuration for `TestEPDDisaggregationMultiEncoders` / 类级常量与配置
```python
    """
    Test EPD disaggregation with multiple encode servers for load balancing.
    Both encode servers run on GPU 0 (different ports) for testing load distribution.
    """

    # Qwen2.5-VL-3B-Instruct scores ~0.40 on the 50-sample MMMU subset.
    accuracy = 0.40
    mmmu_args = ["--limit", "50"]
```
**EN:** This block defines shared names such as `accuracy`, `mmmu_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `accuracy`, `mmmu_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 946-990: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST
        cls.base_url = cls.lb_url  # MMMUMixin reads this for OPENAI_API_BASE
        cls.encode_port1 = f"{int(cls.lb_port) + 300}"
        cls.encode_port2 = f"{int(cls.lb_port) + 301}"
        cls.encode_url1 = f"http://{cls.base_host}:{cls.encode_port1}"
        cls.encode_url2 = f"http://{cls.base_host}:{cls.encode_port2}"

        print(
            f"Setting up EPD (multiple encoders): encode1={cls.encode_port1}, "
            f"encode2={cls.encode_port2}, prefill={cls.prefill_port}, decode={cls.decode_port}"
        )

        # Start two encode servers on GPU 0/1
        encode1_thread = threading.Thread(
            target=cls.start_encode_server, args=(cls.encode_port1, 0)
        )
        encode2_thread = threading.Thread(
            target=cls.start_encode_server, args=(cls.encode_port2, 1)
        )
        encode1_thread.start()
        encode2_thread.start()
        encode1_thread.join()
        encode2_thread.join()

        prefill_thread = threading.Thread(target=cls.start_prefill)
        decode_thread = threading.Thread(target=cls.start_decode)
        prefill_thread.start()
        decode_thread.start()
        prefill_thread.join()
        decode_thread.join()

        cls.wait_server_ready(cls.encode_url1 + "/health", process=cls.process_encode1)
        cls.wait_server_ready(cls.encode_url2 + "/health", process=cls.process_encode2)
        cls.wait_server_ready(cls.prefill_url + "/health", process=cls.process_prefill)
        cls.wait_server_ready(cls.decode_url + "/health", process=cls.process_decode)

        cls.launch_lb()

        # Set OpenAI API key and base URL environment variables. Needed for lmms-eval to work.
        cls.api_key = "sk-123456"
        os.environ["OPENAI_API_KEY"] = cls.api_key
        os.environ["OPENAI_API_BASE"] = f"{cls.lb_url}/v1"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 992-1019: helper routine start encode server / 辅助流程 start encode server
```python
    @classmethod
    def start_encode_server(cls, port, gpu_id):
        """Start an encode server on specific port and GPU"""
        encode_args = [
            "--trust-remote-code",
            "--encoder-only",
            "--encoder-transfer-backend",
            "zmq_to_scheduler",
            "--tp",
            "1",
            "--port",
            port,
            "--enable-prefix-mm-cache",
        ]
        # Only set base-gpu-id if not using GPU 0
        if gpu_id != 0:
            encode_args.extend(["--base-gpu-id", str(gpu_id)])

        process = popen_launch_server(
            cls.model,
            base_url=f"http://{cls.base_host}:{port}",
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=encode_args,
        )
        if port == cls.encode_port1:
            cls.process_encode1 = process
        else:
            cls.process_encode2 = process
```
**EN:** Start an encode server on specific port and GPU This helper encapsulates `start_encode_server` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Start an encode server on specific port and GPU 该辅助函数封装了 `start_encode_server`，以便周围测试复用准备、执行或校验逻辑。

### Lines 1021-1049: helper routine start prefill / 辅助流程 start prefill
```python
    @classmethod
    def start_prefill(cls):
        """Start prefill server with multiple encode URLs"""
        prefill_args = [
            "--trust-remote-code",
            "--language-only",
            "--encoder-urls",
            cls.encode_url1,
            cls.encode_url2,
            "--encoder-transfer-backend",
            "zmq_to_scheduler",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--base-gpu-id",
            "2",
            "--port",
            cls.prefill_port,
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        cls.process_prefill = popen_launch_server(
            cls.model,
            base_url=cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
        )
```
**EN:** Start prefill server with multiple encode URLs This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Start prefill server with multiple encode URLs 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。

### Lines 1051-1073: helper routine start decode / 辅助流程 start decode
```python
    @classmethod
    def start_decode(cls):
        """Start decode server"""
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--base-gpu-id",
            "3",
            "--port",
            cls.decode_port,
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_server(
            cls.model,
            base_url=cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** Start decode server This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** Start decode server 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 1075-1089: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        """Clean up all processes"""
        for process in [
            cls.process_lb,
            cls.process_decode,
            cls.process_prefill,
            cls.process_encode1,
            cls.process_encode2,
        ]:
            if process:
                try:
                    kill_process_tree(process.pid)
                except Exception as e:
                    print(f"Error killing process: {e}")
```
**EN:** Clean up all processes This routine releases resources and restores state after the related tests finish.
**CN:** Clean up all processes 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 1092-1093: class TestEPDDisaggregationGrpcEncoderMMMU declaration / 类 TestEPDDisaggregationGrpcEncoderMMMU 声明
```python
@unittest.skipIf(is_in_ci(), "Skipping in CI to reduce multi-GPU runtime")
class TestEPDDisaggregationGrpcEncoderMMMU(MMMUMixin, PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `MMMUMixin`, `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `MMMUMixin`, `PDDisaggregationServerBase`。

### Lines 1094-1098: class-level constants and configuration for `TestEPDDisaggregationGrpcEncoderMMMU` / 类级常量与配置
```python
    """Test MMMU evaluation with gRPC encoder in EPD mode."""

    # Qwen2.5-VL-3B-Instruct scores ~0.40 on the 50-sample MMMU subset.
    accuracy = 0.40
    mmmu_args = ["--limit", "50"]
```
**EN:** This block defines shared names such as `accuracy`, `mmmu_args`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `accuracy`, `mmmu_args` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 1100-1129: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        cls.model = DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST
        cls.base_url = cls.lb_url  # MMMUMixin reads this for OPENAI_API_BASE
        cls.encode_port = f"{int(cls.lb_port) + 304}"
        cls.encode_url = f"grpc://{cls.base_host}:{cls.encode_port}"

        print(
            f"Setting up gRPC EPD (one encoder): encode={cls.encode_port}, "
            f"prefill={cls.prefill_port}, decode={cls.decode_port}"
        )

        cls.start_encode()
        prefill_thread = threading.Thread(target=cls.start_prefill)
        decode_thread = threading.Thread(target=cls.start_decode)
        prefill_thread.start()
        decode_thread.start()
        prefill_thread.join()
        decode_thread.join()

        cls.wait_grpc_ready(cls.base_host, cls.encode_port, cls.process_encode)
        cls.wait_server_ready(cls.prefill_url + "/health")
        cls.wait_server_ready(cls.decode_url + "/health")

        cls.launch_lb()

        cls.api_key = "sk-123456"
        os.environ["OPENAI_API_KEY"] = cls.api_key
        os.environ["OPENAI_API_BASE"] = f"{cls.lb_url}/v1"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 1131-1154: helper routine start encode / 辅助流程 start encode
```python
    @classmethod
    def start_encode(cls):
        encode_command = [
            "python3",
            "-m",
            "sglang.launch_server",
            "--model-path",
            cls.model,
            "--host",
            cls.base_host,
            "--port",
            cls.encode_port,
            "--trust-remote-code",
            "--encoder-only",
            "--grpc-mode",
            "--encoder-transfer-backend",
            "zmq_to_scheduler",
            "--tp",
            "1",
            "--base-gpu-id",
            "0",
            "--enable-prefix-mm-cache",
        ]
        cls.process_encode = subprocess.Popen(encode_command)
```
**EN:** This helper encapsulates `start_encode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_encode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 1156-1185: helper routine start prefill / 辅助流程 start prefill
```python
    @classmethod
    def start_prefill(cls):
        prefill_args = [
            "--trust-remote-code",
            "--language-only",
            "--encoder-urls",
            cls.encode_url,
            "--encoder-transfer-backend",
            "zmq_to_scheduler",
            "--disaggregation-mode",
            "prefill",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--base-gpu-id",
            "1",
            "--port",
            cls.prefill_port,
        ]
        prefill_args += cls.transfer_backend + cls.rdma_devices
        prefill_env = os.environ.copy()
        prefill_env["SGLANG_ENCODER_MM_RECEIVER_MODE"] = "grpc"
        cls.process_prefill = popen_launch_server(
            cls.model,
            base_url=cls.prefill_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=prefill_args,
            env=prefill_env,
        )
```
**EN:** This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。

### Lines 1187-1208: helper routine start decode / 辅助流程 start decode
```python
    @classmethod
    def start_decode(cls):
        decode_args = [
            "--trust-remote-code",
            "--disaggregation-mode",
            "decode",
            "--disaggregation-bootstrap-port",
            cls.bootstrap_port,
            "--tp",
            "1",
            "--base-gpu-id",
            "2",
            "--port",
            cls.decode_port,
        ]
        decode_args += cls.transfer_backend + cls.rdma_devices
        cls.process_decode = popen_launch_server(
            cls.model,
            base_url=cls.decode_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=decode_args,
        )
```
**EN:** This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 1210-1235: method wait grpc ready / 方法 wait grpc ready
```python
    @staticmethod
    def wait_grpc_ready(host, port, process, timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH):
        deadline = time.time() + timeout
        channel = grpc.insecure_channel(f"{host}:{port}")
        stub = health_pb2_grpc.HealthStub(channel)
        try:
            while time.time() < deadline:
                if process.poll() is not None:
                    raise RuntimeError(
                        f"gRPC encoder server exited with code {process.returncode}"
                    )
                try:
                    response = stub.Check(
                        health_pb2.HealthCheckRequest(service=""), timeout=2
                    )
                    if response.status == health_pb2.HealthCheckResponse.SERVING:
                        return
                except grpc.RpcError:
                    pass
                time.sleep(1)
        finally:
            channel.close()

        raise RuntimeError(
            f"gRPC encoder server not ready at {host}:{port} within {timeout}s"
        )
```
**EN:** This block implements `wait_grpc_ready` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `wait_grpc_ready`，承担模块行为中的一个聚焦逻辑片段。

### Lines 1237-1252: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        os.environ.pop("SGLANG_ENCODER_MM_RECEIVER_MODE", None)
        os.environ.pop("OPENAI_API_KEY", None)
        os.environ.pop("OPENAI_API_BASE", None)
        for process in [
            cls.process_lb,
            cls.process_decode,
            cls.process_prefill,
            cls.process_encode,
        ]:
            if process:
                try:
                    kill_process_tree(process.pid)
                except Exception as e:
                    print(f"Error killing process: {e}")
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 1255-1256: class TestEPDDisaggregationGrpcEncoderOnly declaration / 类 TestEPDDisaggregationGrpcEncoderOnly 声明
```python
@unittest.skipIf(is_in_ci(), "Skipping in CI to reduce multi-GPU runtime")
class TestEPDDisaggregationGrpcEncoderOnly(PDDisaggregationServerBase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `PDDisaggregationServerBase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `PDDisaggregationServerBase`。

### Lines 1257-1257: supporting statements / 辅助语句
```python
    """Test gRPC encoder server integration with zmq_to_scheduler transfers."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 1259-1269: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        super().setUpClass()
        os.environ["SGLANG_ENCODER_MM_RECEIVER_MODE"] = "grpc"
        cls.model = DEFAULT_SMALL_VLM_MODEL_NAME_FOR_TEST
        cls.encode_port = f"{int(cls.lb_port) + 302}"

        print(f"Setting up gRPC EPD encoder: encode={cls.encode_port}")

        cls.start_encode()
        cls.wait_grpc_ready(cls.base_host, cls.encode_port, cls.process_encode)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 1271-1294: helper routine start encode / 辅助流程 start encode
```python
    @classmethod
    def start_encode(cls):
        encode_command = [
            "python3",
            "-m",
            "sglang.launch_server",
            "--model-path",
            cls.model,
            "--host",
            cls.base_host,
            "--port",
            cls.encode_port,
            "--trust-remote-code",
            "--encoder-only",
            "--grpc-mode",
            "--encoder-transfer-backend",
            "zmq_to_scheduler",
            "--tp",
            "1",
            "--base-gpu-id",
            "0",
            "--enable-prefix-mm-cache",
        ]
        cls.process_encode = subprocess.Popen(encode_command)
```
**EN:** This helper encapsulates `start_encode` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `start_encode`，以便周围测试复用准备、执行或校验逻辑。

### Lines 1296-1321: method wait grpc ready / 方法 wait grpc ready
```python
    @staticmethod
    def wait_grpc_ready(host, port, process, timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH):
        deadline = time.time() + timeout
        channel = grpc.insecure_channel(f"{host}:{port}")
        stub = health_pb2_grpc.HealthStub(channel)
        try:
            while time.time() < deadline:
                if process.poll() is not None:
                    raise RuntimeError(
                        f"gRPC encoder server exited with code {process.returncode}"
                    )
                try:
                    response = stub.Check(
                        health_pb2.HealthCheckRequest(service=""), timeout=2
                    )
                    if response.status == health_pb2.HealthCheckResponse.SERVING:
                        return
                except grpc.RpcError:
                    pass
                time.sleep(1)
        finally:
            channel.close()

        raise RuntimeError(
            f"gRPC encoder server not ready at {host}:{port} within {timeout}s"
        )
```
**EN:** This block implements `wait_grpc_ready` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `wait_grpc_ready`，承担模块行为中的一个聚焦逻辑片段。

### Lines 1323-1331: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        os.environ.pop("SGLANG_ENCODER_MM_RECEIVER_MODE", None)
        if cls.process_encode:
            try:
                kill_process_tree(cls.process_encode.pid)
            except Exception as e:
                print(f"Error killing process: {e}")
        super().tearDownClass()
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 1333-1377: test case grpc encoder zmq to scheduler / 测试用例 grpc encoder zmq to scheduler
```python
    def test_grpc_encoder_zmq_to_scheduler(self):
        from smg_grpc_proto import sglang_encoder_pb2, sglang_encoder_pb2_grpc

        context = zmq.Context()
        recv_port, recv_socket = get_zmq_socket_on_host(
            context, zmq.PULL, host=self.base_host
        )
        channel = grpc.insecure_channel(f"{self.base_host}:{self.encode_port}")
        stub = sglang_encoder_pb2_grpc.SglangEncoderStub(channel)
        req_id = f"grpc-epd-{int(time.time() * 1000)}"
        image_path = os.path.abspath("examples/assets/example_image.png")

        try:
            stub.SchedulerReceiveUrl(
                sglang_encoder_pb2.SchedulerReceiveUrlRequest(
                    req_id=req_id,
                    receive_url=f"{self.base_host}:{recv_port}",
                    receive_count=1,
                ),
                timeout=60,
            )
            stub.Encode(
                sglang_encoder_pb2.EncodeRequest(
                    mm_items=[image_path],
                    req_id=req_id,
                    num_parts=1,
                    part_idx=0,
                ),
                timeout=300,
            )

            poller = zmq.Poller()
            poller.register(recv_socket, zmq.POLLIN)
            socks = dict(poller.poll(60000))
            self.assertIn(
                recv_socket,
                socks,
                "No embedding payload received from gRPC encoder server",
            )
            parts = recv_socket.recv_multipart()
            self.assertTrue(parts, "Empty embedding payload from gRPC encoder server")
        finally:
            recv_socket.close()
            context.term()
            channel.close()
```
**EN:** This test exercises `test_grpc_encoder_zmq_to_scheduler` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_grpc_encoder_zmq_to_scheduler`。

### Lines 1380-1381: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestEPDDisaggregationOmni`: EPD disaggregation test for omni models (e.g. / 用于组织相关测试、夹具或辅助方法。
- `TestEPDDisaggregationOneEncoder`: Test EPD disaggregation with single encode server / 用于组织相关测试、夹具或辅助方法。
- `TestEPDDisaggregationQwen35`: EPD disaggregation test for Qwen3.5 image and video requests. / 用于组织相关测试、夹具或辅助方法。
- `TestEPDDisaggregationMultiEncoders`: Test EPD disaggregation with multiple encode servers for load balancing. / 用于组织相关测试、夹具或辅助方法。
- `TestEPDDisaggregationGrpcEncoderMMMU`: Test MMMU evaluation with gRPC encoder in EPD mode. / 用于组织相关测试、夹具或辅助方法。
- `TestEPDDisaggregationGrpcEncoderOnly`: Test gRPC encoder server integration with zmq_to_scheduler transfers. / 用于组织相关测试、夹具或辅助方法。
- `TestEPDDisaggregationOmni.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestEPDDisaggregationOmni.start_encode`: This helper encapsulates `start_encode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_encode`，以便周围测试复用准备、执行或校验逻辑。
- `TestEPDDisaggregationOmni.start_prefill`: This helper encapsulates `start_prefill` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_prefill`，以便周围测试复用准备、执行或校验逻辑。
- `TestEPDDisaggregationOmni.start_decode`: This helper encapsulates `start_decode` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `start_decode`，以便周围测试复用准备、执行或校验逻辑。
- `TestEPDDisaggregationOmni.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestEPDDisaggregationOmni._wait_grpc_ready`: This block implements `_wait_grpc_ready` and captures one focused piece of the module's behavior. / 该代码块实现 `_wait_grpc_ready`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `io`, `os`, `re`, `subprocess`, `threading`, `time`, `unittest`
- **Third-party modules / 第三方模块**: `grpc`, `openai`, `zmq`, `grpc_health.v1`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.srt.utils.network`, `sglang.test.ci.ci_register`, `sglang.test.kits.mmmu_vlm_kit`, `sglang.test.server_fixtures.disaggregation_fixture`, `sglang.test.test_utils`, `sglang.test.vlm_utils`

- **Total lines / 总行数**: 1381
