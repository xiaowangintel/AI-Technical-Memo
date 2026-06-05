# test_session_control.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/sessions/test_session_control.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates session control behavior in SGLang's sessions area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 sessions 领域中与 session control 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-7: supporting statements / 辅助语句
```python
"""
Usage:
python3 -m unittest test_session_control.TestSessionControl.test_session_control
python3 -m unittest test_session_control.TestSessionControl.test_session_control_with_branching
python3 -m unittest test_session_control.TestSessionControl.test_session_control_backtrack_with_abort
python3 -m unittest test_session_control.TestSessionControlVision.test_session_control
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 9-25: module imports and dependencies / 模块导入与依赖
```python
import asyncio
import json
import unittest

import aiohttp
import requests

from sglang.srt.utils import is_hip, kill_process_tree
from sglang.srt.utils.hf_transformers_utils import get_tokenizer
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `asyncio`, `json`, `unittest`, `aiohttp`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `asyncio`, `json`, `unittest`, `aiohttp`。

### Lines 27-28: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=87, stage="extra-a", runner_config="1-gpu-large")
register_amd_ci(est_time=87, suite="stage-b-test-1-gpu-large-amd")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 31-32: function remove prefix / 函数 remove prefix
```python
def remove_prefix(text: str, prefix: str) -> str:
    return text[len(prefix) :] if text.startswith(prefix) else text
```
**EN:** This block implements `remove_prefix` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `remove_prefix`，承担模块行为中的一个聚焦逻辑片段。

### Lines 35-35: class TestSessionControl declaration / 类 TestSessionControl 声明
```python
class TestSessionControl(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 36-50: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=[
                "--attention-backend",
                "triton",
                "--disable-cuda-graph",
                "--disable-piecewise-cuda-graph",
            ],
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 52-54: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 56-135: test case session control (part 1/4) / 测试用例 session control（第 1/4 部分）
```python
    def test_session_control(self, gen_len=12):
        chunks = [
            "Let me tell you something about France.",
            "The capital of France is",
            "The population of the city is",
            "A brief history about that city is",
        ]
        tokenizer = get_tokenizer(self.model)
        chunks_ids = [tokenizer.encode(x) for x in chunks]
        for i in range(1, len(chunks_ids)):
            if chunks_ids[i][0] == tokenizer.bos_token_id:
                chunks_ids[i] = chunks_ids[i][1:]

        # 1. using session control
        requests.post(self.base_url + "/flush_cache")
        session_id = requests.post(
            self.base_url + "/open_session",
            json={"capacity_of_str_len": 1000},
        ).json()
        rid = None

        # open an existing session, should get session_id as None
        ret = requests.post(
            self.base_url + "/open_session",
            json={"capacity_of_str_len": 1000, "session_id": session_id},
        )
        self.assertNotEqual(ret.status_code, 200)

        first_rid = None
        outputs_from_session = []
        logprobs_from_session = []
        cur_logprob_start_len = 0
        for i, chunk_ids in enumerate(chunks_ids):
            max_new_tokens = gen_len if i > 0 else 1  # prefill only for the first chunk
            response = requests.post(
                self.base_url + "/generate",
                json={
                    "input_ids": chunk_ids,
                    "session_params": {
                        "id": session_id,
                        "rid": rid,
                        "offset": -1,
                        "replace": True,
                    },
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": max_new_tokens,
                        "no_stop_trim": True,
                        "skip_special_tokens": False,
                    },
                    "return_logprob": True,
                    "logprob_start_len": cur_logprob_start_len - 1,
                },
            ).json()
            rid = response["meta_info"]["id"]
            if i == 0:
                first_rid = rid
            if i > 0:
                outputs_from_session.append(response["text"])
                logprobs_from_session.extend(
                    [
                        round(sublist[0], 2)
                        for sublist in response["meta_info"]["output_token_logprobs"]
                    ]
                )
            cur_logprob_start_len += len(chunk_ids) + max_new_tokens

        # query with a logprob_start_len longer than the request, should see error
        ret = requests.post(
            self.base_url + "/generate",
            json={
                "input_ids": chunk_ids,
                "session_params": {
                    "id": session_id,
                    "rid": rid,
                    "offset": -1,
                    "replace": True,
                },
                "sampling_params": {
                    "temperature": 0,
```
**EN:** This test exercises `test_session_control` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control`。 这一段对应同一逻辑块的第 1 部分。

### Lines 136-215: test case session control (part 2/4) / 测试用例 session control（第 2/4 部分）
```python
                    "max_new_tokens": max_new_tokens,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
                "return_logprob": True,
                "logprob_start_len": cur_logprob_start_len + len(chunk_ids),
            },
        )
        self.assertNotEqual(ret.status_code, 200)

        # backtrack to the first request and regenerate
        cur_logprob_start_len = 0
        response = requests.post(
            self.base_url + "/generate",
            json={
                "input_ids": chunks_ids[-1],
                "session_params": {
                    "id": session_id,
                    "rid": first_rid,
                    "offset": -1,
                    "replace": True,
                },
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": gen_len,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
                "return_logprob": True,
                "logprob_start_len": cur_logprob_start_len,
            },
        ).json()
        outputs_from_session.append(response["text"])
        logprobs_from_session.extend(
            [
                round(sublist[0], 2)
                for sublist in response["meta_info"]["output_token_logprobs"]
            ]
        )

        # query with a non-existing rid (the last one should be disappeared because of backtrack), should see abort
        ret = requests.post(
            self.base_url + "/generate",
            json={
                "input_ids": chunks_ids[-1],
                "session_params": {
                    "id": session_id,
                    "rid": rid,
                    "offset": -1,
                    "replace": True,
                },
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": gen_len,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
                "return_logprob": True,
            },
        )
        self.assertNotEqual(ret.status_code, 200)

        ret = requests.post(
            self.base_url + "/close_session",
            json={"session_id": session_id},
        )
        self.assertEqual(ret.status_code, 200)

        # send a request to a closed session, should see abort
        ret = requests.post(
            self.base_url + "/generate",
            json={
                "input_ids": chunks_ids[-1],
                "session_params": {
                    "id": session_id,
                    "rid": first_rid,
                    "offset": -1,
                    "replace": True,
                },
                "sampling_params": {
```
**EN:** This test exercises `test_session_control` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control`。 这一段对应同一逻辑块的第 2 部分。

### Lines 216-295: test case session control (part 3/4) / 测试用例 session control（第 3/4 部分）
```python
                    "temperature": 0,
                    "max_new_tokens": gen_len,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
                "return_logprob": True,
            },
        )
        self.assertNotEqual(ret.status_code, 200)

        # 2. not use session control
        requests.post(self.base_url + "/flush_cache")

        input_ids_first_req = None
        input_ids = []
        outputs_normal = []
        logprobs_normal = []
        for i, chunk_ids in enumerate(chunks_ids):
            input_ids += chunk_ids
            response = requests.post(
                self.base_url + "/generate",
                json={
                    "input_ids": input_ids,
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": (
                            gen_len if i > 0 else 1
                        ),  # prefill only for the first chunk
                        "no_stop_trim": True,
                        "skip_special_tokens": False,
                    },
                    "return_logprob": True,
                },
            ).json()
            if i > 0:
                output_ids = tokenizer.encode(response["text"])
                if output_ids[0] == tokenizer.bos_token_id:
                    output_ids = output_ids[1:]
                input_ids += output_ids[:-1]
                outputs_normal.append(response["text"])
                logprobs_normal.extend(
                    [
                        round(sublist[0], 2)
                        for sublist in response["meta_info"]["output_token_logprobs"]
                    ]
                )
            if i == 0:
                input_ids_first_req = input_ids.copy()

        input_ids_first_req += chunks_ids[-1]
        response = requests.post(
            self.base_url + "/generate",
            json={
                "input_ids": input_ids_first_req,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": gen_len,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
                "return_logprob": True,
            },
        ).json()
        outputs_normal.append(response["text"])
        logprobs_normal.extend(
            [
                round(sublist[0], 2)
                for sublist in response["meta_info"]["output_token_logprobs"]
            ]
        )

        print("outputs from chunked queries with session control:")
        print(outputs_from_session)
        print("outputs from normal queries:")
        print(outputs_normal)
        self.assertEqual(outputs_from_session, outputs_normal)
        print("logprobs from chunked queries with session control:")
        print(logprobs_from_session)
        print("logprobs from normal queries:")
        print(logprobs_normal)
```
**EN:** This test exercises `test_session_control` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 3 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control`。 这一段对应同一逻辑块的第 3 部分。

### Lines 296-300: test case session control (part 4/4) / 测试用例 session control（第 4/4 部分）
```python
        assert len(logprobs_from_session) == len(
            logprobs_normal
        ), "logprobs must have equal length"
        for a, b in zip(logprobs_from_session, logprobs_normal):
            assert abs(a - b) <= 0.15, f"logprobs {a} and {b} differ by more than 0.15"
```
**EN:** This test exercises `test_session_control` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 4 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control`。 这一段对应同一逻辑块的第 4 部分。

### Lines 302-321: method async generate / 方法 async generate
```python
    async def async_generate(self, payload):
        url = self.base_url + "/generate"
        async with aiohttp.ClientSession() as session:
            async with session.post(url=url, json=payload) as response:
                assert response.status == 200
                async for chunk_bytes in response.content:
                    chunk_bytes = chunk_bytes.strip()
                    if not chunk_bytes:
                        continue
                    chunk = remove_prefix(chunk_bytes.decode("utf-8"), "data: ")
                    if chunk == "[DONE]":
                        yield "", None, ""
                    else:
                        data = json.loads(chunk)
                        finish_reason = (
                            data["meta_info"]["finish_reason"]["type"]
                            if data["meta_info"]["finish_reason"]
                            else ""
                        )
                        yield data["text"], data["meta_info"]["id"], finish_reason
```
**EN:** This block implements `async_generate` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `async_generate`，承担模块行为中的一个聚焦逻辑片段。

### Lines 323-402: helper routine run session control backtrack with abort (part 1/2) / 辅助流程 run session control backtrack with abort（第 1/2 部分）
```python
    async def run_session_control_backtrack_with_abort(self, replace):
        chunks = [
            "Let me tell you something about France.",
            "The capital of France is",
        ]
        tokenizer = get_tokenizer(self.model)
        chunks_ids = [tokenizer.encode(x) for x in chunks]
        for i in range(1, len(chunks_ids)):
            if chunks_ids[i][0] == tokenizer.bos_token_id:
                chunks_ids[i] = chunks_ids[i][1:]

        # 1. using session control
        requests.post(self.base_url + "/flush_cache")
        session_id = requests.post(
            self.base_url + "/open_session",
            json={"capacity_of_str_len": 1000},
        ).json()
        rid = None

        payload = {
            "input_ids": chunks_ids[0],
            "session_params": {
                "id": session_id,
                "rid": rid,
                "offset": -1,
                "replace": True,
            },
            "sampling_params": {
                "temperature": 0,
                "max_new_tokens": 100,
                "no_stop_trim": True,
                "skip_special_tokens": False,
                "ignore_eos": True,
            },
            "stream": True,
        }
        gen_so_far = ""
        finish_reason = ""
        second_output = ""
        async for chunk, rid, finish_reason_chunk in self.async_generate(payload):
            gen_so_far += chunk
            if finish_reason == "":
                finish_reason = finish_reason_chunk
            if len(gen_so_far) > 50 and second_output == "":
                payload2 = {
                    "input_ids": chunks_ids[1],
                    "session_params": {
                        "id": session_id,
                        "rid": rid,
                        "offset": 50,
                        "replace": replace,
                    },
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": 32,
                        "no_stop_trim": True,
                        "skip_special_tokens": False,
                    },
                    "stream": False,
                    "stream_output": True,
                }
                response = requests.post(
                    url=self.base_url + "/generate", json=payload2
                ).json()
                second_output = response["text"]
        if replace:
            assert finish_reason == "abort"
        print("first request output:")
        print(gen_so_far)
        print("second request output:")
        print(second_output)

        # close the session
        ret = requests.post(
            self.base_url + "/close_session",
            json={"session_id": session_id},
        )
        assert ret.status_code == 200

        if not replace:
```
**EN:** This helper encapsulates `run_session_control_backtrack_with_abort` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 1 of the same logical block.
**CN:** 该辅助函数封装了 `run_session_control_backtrack_with_abort`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 1 部分。

### Lines 403-431: helper routine run session control backtrack with abort (part 2/2) / 辅助流程 run session control backtrack with abort（第 2/2 部分）
```python
            assert response["meta_info"]["finish_reason"]["type"] == "abort"
        else:
            # 2. not using session control
            requests.post(self.base_url + "/flush_cache")
            output_ids = tokenizer.encode(gen_so_far)
            if output_ids[0] == tokenizer.bos_token_id:
                output_ids = output_ids[1:]
            input_ids = chunks_ids[0] + output_ids
            input_ids = input_ids[:50] + chunks_ids[1]
            payload = {
                "input_ids": input_ids,
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 32,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
                "stream": False,
                "stream_output": True,
            }
            response = requests.post(
                url=self.base_url + "/generate", json=payload
            ).json()
            output_no_session = response["text"]
            print("second request output without session:")
            print(output_no_session)
            assert (
                second_output == output_no_session
            ), f"second_output: {second_output}, output_no_session: {output_no_session}"
```
**EN:** This helper encapsulates `run_session_control_backtrack_with_abort` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 2 of the same logical block.
**CN:** 该辅助函数封装了 `run_session_control_backtrack_with_abort`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 2 部分。

### Lines 433-436: test case session control backtrack with abort / 测试用例 session control backtrack with abort
```python
    @unittest.skip("broken")
    def test_session_control_backtrack_with_abort(self):
        asyncio.run(self.run_session_control_backtrack_with_abort(replace=True))
        asyncio.run(self.run_session_control_backtrack_with_abort(replace=False))
```
**EN:** This test exercises `test_session_control_backtrack_with_abort` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control_backtrack_with_abort`。

### Lines 438-517: helper routine run session control with branching (part 1/2) / 辅助流程 run session control with branching（第 1/2 部分）
```python
    def run_session_control_with_branching(
        self, root_prompt, chunks_per_step, gen_len=16
    ):
        for x in chunks_per_step:
            assert len(x) == len(chunks_per_step[0])

        # 1. using session control
        requests.post(self.base_url + "/flush_cache")
        session_id = requests.post(
            self.base_url + "/open_session",
            json={"capacity_of_str_len": 1000},
        ).json()

        outputs_from_session = []
        # send the root prompt
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": root_prompt,
                "session_params": {
                    "id": session_id,
                    "rid": None,
                    "offset": 0,
                    "replace": False,
                },
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": gen_len,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
            },
        ).json()
        rid_per_branch = [response["meta_info"]["id"]] * len(chunks_per_step[0])
        outputs_from_session.append(response["text"])

        # send the prompts in branches
        for chunks_for_branches in chunks_per_step:
            for j, chunk in enumerate(chunks_for_branches):
                response = requests.post(
                    self.base_url + "/generate",
                    json={
                        "text": chunk,
                        "session_params": {
                            "id": session_id,
                            "rid": rid_per_branch[j],
                            "offset": 0,
                            "replace": False,
                        },
                        "sampling_params": {
                            "temperature": 0,
                            "max_new_tokens": gen_len,
                            "no_stop_trim": True,
                            "skip_special_tokens": False,
                        },
                    },
                ).json()
                rid = response["meta_info"]["id"]
                rid_per_branch[j] = rid
                outputs_from_session.append(response["text"])

        # close the session
        ret = requests.post(
            self.base_url + "/close_session",
            json={"session_id": session_id},
        )
        assert ret.status_code == 200

        # 2. not use session control
        requests.post(self.base_url + "/flush_cache")

        outputs_normal = []
        input_texts = [root_prompt] * len(chunks_per_step[0])
        # send the root prompt
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": root_prompt,
                "sampling_params": {
                    "temperature": 0,
```
**EN:** This helper encapsulates `run_session_control_with_branching` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 1 of the same logical block.
**CN:** 该辅助函数封装了 `run_session_control_with_branching`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 1 部分。

### Lines 518-552: helper routine run session control with branching (part 2/2) / 辅助流程 run session control with branching（第 2/2 部分）
```python
                    "max_new_tokens": gen_len,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
            },
        ).json()
        outputs_normal.append(response["text"])
        input_texts = [x + response["text"] for x in input_texts]

        # send the prompts in branches
        for chunks_for_branches in chunks_per_step:
            for j, chunk in enumerate(chunks_for_branches):
                input_texts[j] += chunk
                response = requests.post(
                    self.base_url + "/generate",
                    json={
                        "text": input_texts[j],
                        "sampling_params": {
                            "temperature": 0,
                            "max_new_tokens": gen_len,
                            "no_stop_trim": True,
                            "skip_special_tokens": False,
                        },
                    },
                ).json()
                outputs_normal.append(response["text"])
                input_texts[j] += response["text"]

        print("====== outputs from chunked queries with session control: =======")
        print(outputs_from_session)
        print("====== outputs from normal queries: =======")
        print(outputs_normal)
        assert (
            outputs_from_session == outputs_normal
        ), f"outputs_from_session: {outputs_from_session}, outputs_normal: {outputs_normal}"
```
**EN:** This helper encapsulates `run_session_control_with_branching` so the surrounding tests can reuse setup, execution, or validation logic. This chunk continues part 2 of the same logical block.
**CN:** 该辅助函数封装了 `run_session_control_with_branching`，以便周围测试复用准备、执行或校验逻辑。 这一段对应同一逻辑块的第 2 部分。

### Lines 554-582: test case session control with branching / 测试用例 session control with branching
```python
    @unittest.skipIf(
        is_hip(),
        "Session-branching produces a deterministic 1-token divergence from "
        "plain generation on AMD (greedy temperature=0). The other 4 of 5 "
        "branch outputs match exactly; passes on CUDA. Suspected ROCm "
        "numerical-precision difference in the session KV-cache reuse path. "
        "Re-enable when that divergence is fixed.",
    )
    def test_session_control_with_branching(self):
        root_prompt = "First, let me explain in one sentence about AI"
        chunks_per_step = [
            [
                "Then, briefly, the positive side of AI is",
                "But, briefly, AI could be harmful to human",
            ],
            ["For example", "For example"],
        ]
        self.run_session_control_with_branching(
            root_prompt=root_prompt, chunks_per_step=chunks_per_step, gen_len=8
        )

        root_prompt = "I have three apples."
        chunks_per_step = [
            ["I then give one apple to my friend", "My friend give me another apple."],
            ["I still have", "I now have"],
        ]
        self.run_session_control_with_branching(
            root_prompt=root_prompt, chunks_per_step=chunks_per_step, gen_len=8
        )
```
**EN:** This test exercises `test_session_control_with_branching` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control_with_branching`。

### Lines 585-585: class TestSessionControlVision declaration / 类 TestSessionControlVision 声明
```python
class TestSessionControlVision(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 586-594: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.model = "OpenGVLab/InternVL2-2B"
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 596-598: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 600-679: test case session control (part 1/3) / 测试用例 session control（第 1/3 部分）
```python
    def test_session_control(self):
        image_token = "<IMG_CONTEXT>"
        text_chunks = [
            "<|im_start|>system\nYou are a helpful assistant.<|im_end|>\n",
            f"<|im_start|>user\n{image_token}\nDescribe this image in a very short sentence.<|im_end|>\n<|im_start|>assistant\n",
            f"<|im_start|>user\n{image_token}\nIs this image same with one of the previous images?<|im_end|>\n<|im_start|>assistant\n",
            f"<|im_start|>user\n{image_token}\nIs this image same with one of the previous images?<|im_end|>\n<|im_start|>assistant\n",
            "<|im_start|>user\nDescribe this image in a very short sentence.<|im_end|>\n<|im_start|>assistant\n",
        ]
        image_chunks = [
            "https://raw.githubusercontent.com/sgl-project/sglang/main/examples/assets/example_image.png",
            "https://raw.githubusercontent.com/sgl-project/sglang/main/examples/assets/example_image.png",
            "https://raw.githubusercontent.com/sgl-project/sglang/main/assets/logo.png",
        ]

        self.assertEqual(
            len(text_chunks), len(image_chunks) + 2
        )  # the first and the last prompt does not contain images
        gen_len = 32

        # 1. using session control
        requests.post(self.base_url + "/flush_cache")
        session_id = requests.post(
            self.base_url + "/open_session",
            json={"capacity_of_str_len": 1000},
        ).json()
        rid = None

        # open an existing session, should get session_id as None
        ret = requests.post(
            self.base_url + "/open_session",
            json={"capacity_of_str_len": 1000, "session_id": session_id},
        )
        self.assertNotEqual(ret.status_code, 200)

        first_rid = None
        outputs_from_session = []
        for i in range(len(text_chunks[:-1])):
            response = requests.post(
                self.base_url + "/generate",
                json={
                    "text": text_chunks[i],
                    "image_data": image_chunks[i - 1] if i > 0 else None,
                    "modalities": ["multi-images"],
                    "session_params": {
                        "id": session_id,
                        "rid": rid,
                        "offset": 0,
                        "replace": True,
                    },
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": (
                            gen_len if i > 0 else 0
                        ),  # prefill only for the first chunk
                        "no_stop_trim": True,
                        "skip_special_tokens": False,
                    },
                },
            ).json()
            rid = response["meta_info"]["id"]
            if i == 0:
                first_rid = rid
            if i > 0:
                outputs_from_session.append(response["text"])

        # backtrack to the first request and regenerate
        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": text_chunks[-1],
                "session_params": {
                    "id": session_id,
                    "rid": first_rid,
                    "offset": 0,
                    "replace": True,
                },
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": gen_len,
```
**EN:** This test exercises `test_session_control` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 1 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control`。 这一段对应同一逻辑块的第 1 部分。

### Lines 680-759: test case session control (part 2/3) / 测试用例 session control（第 2/3 部分）
```python
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
            },
        ).json()
        outputs_from_session.append(response["text"])

        # query with a non-existing rid (the last one should be disappeared because of backtrack), should see abort
        ret = requests.post(
            self.base_url + "/generate",
            json={
                "text": text_chunks[-1],
                "session_params": {
                    "id": session_id,
                    "rid": rid,
                    "offset": 0,
                    "replace": True,
                },
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": gen_len,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
            },
        )
        self.assertNotEqual(ret.status_code, 200)

        ret = requests.post(
            self.base_url + "/close_session",
            json={"session_id": session_id},
        )
        self.assertEqual(ret.status_code, 200)

        # send a request to a closed session, should see abort
        ret = requests.post(
            self.base_url + "/generate",
            json={
                "text": text_chunks[-1],
                "session_params": {
                    "id": session_id,
                    "rid": first_rid,
                    "offset": 0,
                    "replace": True,
                },
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": gen_len,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
            },
        )
        self.assertNotEqual(ret.status_code, 200)

        # 2. not use session control
        requests.post(self.base_url + "/flush_cache")

        accumulated_text = ""
        first_req_text = None
        outputs_normal = []
        for i in range(len(text_chunks[:-1])):
            accumulated_text += text_chunks[i]
            image_data = image_chunks[:i] if i > 0 else None
            response = requests.post(
                self.base_url + "/generate",
                json={
                    "text": accumulated_text,
                    "image_data": image_data,
                    "modalities": ["multi-images"],
                    "sampling_params": {
                        "temperature": 0,
                        "max_new_tokens": (
                            gen_len if i > 0 else 0
                        ),  # prefill only for the first chunk
                        "no_stop_trim": True,
                        "skip_special_tokens": False,
                    },
                },
            ).json()
```
**EN:** This test exercises `test_session_control` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 2 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control`。 这一段对应同一逻辑块的第 2 部分。

### Lines 760-786: test case session control (part 3/3) / 测试用例 session control（第 3/3 部分）
```python
            if i > 0:
                accumulated_text += response["text"]
                outputs_normal.append(response["text"])
            if i == 0:
                first_req_text = accumulated_text

        response = requests.post(
            self.base_url + "/generate",
            json={
                "text": first_req_text + text_chunks[-1],
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": gen_len,
                    "no_stop_trim": True,
                    "skip_special_tokens": False,
                },
            },
        ).json()
        outputs_normal.append(response["text"])

        print("outputs from chunked queries with session control:")
        print(outputs_from_session)
        print("outputs from normal queries:")
        print(outputs_normal)
        assert (
            outputs_from_session == outputs_normal
        ), f"outputs_from_session: {outputs_from_session}, outputs_normal: {outputs_normal}"
```
**EN:** This test exercises `test_session_control` by arranging inputs, invoking the relevant path, and checking the expected outcome. This chunk continues part 3 of the same logical block.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control`。 这一段对应同一逻辑块的第 3 部分。

### Lines 789-790: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `remove_prefix`: This block implements `remove_prefix` and captures one focused piece of the module's behavior. / 该代码块实现 `remove_prefix`，承担模块行为中的一个聚焦逻辑片段。
- `TestSessionControl`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSessionControlVision`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestSessionControl.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestSessionControl.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestSessionControl.test_session_control`: This test exercises `test_session_control` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control`。
- `TestSessionControl.async_generate`: This block implements `async_generate` and captures one focused piece of the module's behavior. / 该代码块实现 `async_generate`，承担模块行为中的一个聚焦逻辑片段。
- `TestSessionControl.run_session_control_backtrack_with_abort`: This helper encapsulates `run_session_control_backtrack_with_abort` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_session_control_backtrack_with_abort`，以便周围测试复用准备、执行或校验逻辑。
- `TestSessionControl.test_session_control_backtrack_with_abort`: This test exercises `test_session_control_backtrack_with_abort` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control_backtrack_with_abort`。
- `TestSessionControl.run_session_control_with_branching`: This helper encapsulates `run_session_control_with_branching` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_session_control_with_branching`，以便周围测试复用准备、执行或校验逻辑。
- `TestSessionControl.test_session_control_with_branching`: This test exercises `test_session_control_with_branching` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_session_control_with_branching`。
- `TestSessionControlVision.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `asyncio`, `json`, `unittest`
- **Third-party modules / 第三方模块**: `aiohttp`, `requests`
- **Internal modules / 内部模块**: `sglang.srt.utils`, `sglang.srt.utils.hf_transformers_utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 790
