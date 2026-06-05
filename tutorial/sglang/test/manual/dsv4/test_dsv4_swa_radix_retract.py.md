# test_dsv4_swa_radix_retract.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/manual/dsv4/test_dsv4_swa_radix_retract.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This manual test module covers the `dsv4 swa radix retract` scenario in `test/manual/dsv4`. It uses SGLang's shared test infrastructure to configure models or services and verify expected behavioral and API behavior. / 该手动测试模块覆盖 `test/manual/dsv4` 中的 `dsv4 swa radix retract` 场景。它利用 SGLang 的共享测试基础设施配置模型或服务，并验证预期的行为与接口表现。

## Line-by-Line Analysis / 逐行分析
### Lines 1-54: Constants and scenario settings / 常量与场景配置
```python
"""DSV4 stress test for SWA radix cache + tombstone + retract interaction.

Reproduces the assert in `swa_radix_cache.cache_unfinished_req`:
    assert old_prefix_len <= len(new_indices)

Trip conditions (all required):
  1. Fork-only SWA leaf early-release on (`SGLANG_OPT_SWA_RELEASE_LEAF_LOCK_AFTER_WINDOW=1`)
  2. Multiple requests share a long prefix (so one req's tombstoned leaf
     poisons match_prefix for others walking the same radix path).
  3. Memory pressure forces retract while at least one req has tombstoned
     its leaf (decode_batch_idx >= sliding_window_size at retract time).

After main #19427 changed `old_prefix_len = req.cache_protected_len`
(stable), tombstone-induced shrinks in match's `best_value_len` across
chunked-prefill rounds can make stale `cache_protected_len` exceed
current matchable length -> assert trips.

Test passes iff the scheduler does not crash under this stress workload.
"""

import random
import threading
import time
import unittest

import requests

from sglang.srt.utils import kill_process_tree
from sglang.test.test_utils import (
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)

DSV4_FLASH_MODEL_PATH = "sgl-project/DeepSeek-V4-Flash-FP8"

# Long shared prefix forces multi-chunk prefill and ensures cross-request
# prefix-cache hits so one req's tombstone affects later reqs.
SHARED_PREFIX_BLOCK = (
    "You are a careful, expert assistant. Answer concisely.\n"
    "Context: " + ("the quick brown fox jumps over the lazy dog. " * 600)
)

QUESTION_TAILS = [
    " Q: What is 17*23?\n",
    " Q: List three primary colors.\n",
    " Q: Where is Mount Everest?\n",
    " Q: Summarize gradient descent in two sentences.\n",
    " Q: Name two bodies of water in Africa.\n",
    " Q: What language is spoken in Brazil?\n",
]
```
**EN:** This range defines constants and per-scenario settings that parameterize the later execution path. Process management appears here to launch, monitor, or stop runtime components. Assertions in this block enforce the intended outcome. Representative call sites include `conditions`, `on`, `prefix` and `leaf`.
**CN:** 这一部分定义常量和场景参数，为后续执行路径提供配置。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 55-56: Class definition for TestDSV4FlashSWARadixRetract / 类定义
```python
class TestDSV4FlashSWARadixRetract(CustomTestCase):
    @classmethod
```
**EN:** This range declares `TestDSV4FlashSWARadixRetract`, which organizes the scenario as a reusable test-oriented class.
**CN:** 这一部分声明测试相关类，把当前场景组织为可复用的类结构。

### Lines 57-79: Lifecycle helpers / 生命周期辅助逻辑
```python
    def setUpClass(cls):
        cls.model = DSV4_FLASH_MODEL_PATH
        cls.base_url = DEFAULT_URL_FOR_TEST
        other_args = [
            "--trust-remote-code",
            "--tp",
            "4",
            "--dp",
            "4",
            "--enable-dp-attention",
            "--moe-a2a-backend",
            "deepep",
            "--cuda-graph-max-bs",
            "128",
            "--max-running-requests",
            "256",
            "--deepep-config",
            '{"normal_dispatch":{"num_sms":96},"normal_combine":{"num_sms":96}}',
            "--speculative-algorithm",
            "EAGLE",
            "--speculative-num-steps",
            "3",
            "--speculative-eagle-topk",
```
**EN:** This range implements lifecycle helper(s) `setUpClass` to prepare or clean up shared resources across the test flow. It interacts with request/response APIs to observe live model behavior.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。 它会与请求/响应 API 交互，以观察模型的实时行为。

### Lines 80-81: Scenario logic / 场景逻辑
```python
            "1",
            "--speculative-num-draft-tokens",
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 82-101: Process control logic / 进程控制逻辑
```python
            "4",
            # Tight static memory so SWA pool fills up under load and
            # retract is forced.
            "--mem-fraction-static",
            "0.7",
        ]
        env = {
            "SGLANG_DSV4_FP4_EXPERTS": "0",
            "SGLANG_DEEPEP_NUM_MAX_DISPATCH_TOKENS_PER_RANK": "1024",
            "SGLANG_OPT_SWA_RADIX_CACHE_COMPACT": "0",
            "SGLANG_TEST_RETRACT": "1",
            "SGLANG_TEST_RETRACT_INTERVAL": "3",
        }
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
            other_args=other_args,
            env=env,
        )
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. It interacts with request/response APIs to observe live model behavior. Representative call sites include `popen_launch_server`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 102-103: Scenario logic / 场景逻辑
```python

    @classmethod
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。

### Lines 104-104: Lifecycle helpers / 生命周期辅助逻辑
```python
    def tearDownClass(cls):
```
**EN:** This range implements lifecycle helper(s) `tearDownClass` to prepare or clean up shared resources across the test flow.
**CN:** 这一部分实现生命周期辅助逻辑，用于在测试流程中准备或清理共享资源。

### Lines 105-105: Process control logic / 进程控制逻辑
```python
        kill_process_tree(cls.process.pid)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Process management appears here to launch, monitor, or stop runtime components. Representative call sites include `kill_process_tree`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 这里还包含进程管理逻辑，用于启动、监控或停止运行时组件。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 107-127: Helper routines around _send_req / 辅助例程
```python
    def _send_req(self, prompt: str, max_new_tokens: int):
        try:
            resp = requests.post(
                self.base_url + "/generate",
                json={
                    "text": prompt,
                    "sampling_params": {
                        # Vary outputs slightly so reqs don't share decode
                        # paths perfectly; we want some to finish, some to
                        # be retracted under pressure.
                        "temperature": 0.7,
                        "max_new_tokens": max_new_tokens,
                    },
                },
                timeout=600,
            )
            # Per-request success is not the gate; some requests are
            # expected to be retracted/aborted under heavy pressure.
            return resp.status_code == 200
        except Exception:
            return False
```
**EN:** This range implements helper routine(s) `_send_req` so setup, transformation, or validation logic can be reused cleanly. It interacts with request/response APIs to observe live model behavior. Representative call sites include `post`.
**CN:** 这一部分实现辅助例程，使初始化、转换或校验逻辑能够被整洁地复用。 它会与请求/响应 API 交互，以观察模型的实时行为。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 129-129: Test routines around test_swa_tombstone_retract_does_not_crash / 测试例程
```python
    def test_swa_tombstone_retract_does_not_crash(self):
```
**EN:** This range defines concrete test routine(s) `test_swa_tombstone_retract_does_not_crash`. The logic drives the target scenario and encodes the expected acceptance criteria.
**CN:** 这一部分定义具体测试例程，用于驱动目标场景并写入预期的验收标准。

### Lines 130-153: Assertions and result checks / 断言与结果检查
```python
        """Stress: 64 concurrent long-prompt reqs with long generation force
        retract under SWA pool pressure. Reqs share a 30k+ token prefix so
        tombstoned leaves from retracted reqs are on the radix path of new
        reqs. Scheduler must not crash on the swa_radix_cache assert."""

        random.seed(0)
        concurrency = 64
        # Long enough generation to push past sliding_window_size -> fires
        # `dec_swa_lock_only` -> tombstones leaves. Combined with SWA pool
        # pressure this guarantees retract while tombstones are live.
        max_new_tokens = 1024

        threads = []
        for i in range(concurrency):
            tail = QUESTION_TAILS[i % len(QUESTION_TAILS)]
            # Add a small per-req suffix so reqs don't dedup at radix root
            # but still share the bulk of the prefix.
            prompt = SHARED_PREFIX_BLOCK + tail + f"(seed={i})"
            t = threading.Thread(target=self._send_req, args=(prompt, max_new_tokens))
            threads.append(t)
            t.start()
            # Stagger so requests enter prefill in waves; some are still in
            # decode (and have tombstoned leaves) when later waves of
            # chunked-prefill reqs walk the same radix path.
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `seed`, `Thread`, `append` and `start`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 154-154: Scenario logic / 场景逻辑
```python
            time.sleep(0.05)
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Representative call sites include `sleep`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 155-161: Assertions and result checks / 断言与结果检查
```python

        for t in threads:
            t.join(timeout=600)

        # The only invariant: scheduler survived. Per-request completion is
        # best-effort under retract pressure.
        self.assertIsNone(self.process.poll())
```
**EN:** This range contains intermediate control flow and scenario-specific implementation details that connect the surrounding setup and assertions. Assertions in this block enforce the intended outcome. Representative call sites include `join`, `assertIsNone` and `poll`.
**CN:** 这一部分包含中间控制流程以及场景相关实现细节，用来连接前后的准备与断言逻辑。 其中的断言会落实预期结果。 其中还会调用若干代表性的函数来串联完整流程。

### Lines 162-165: Script entry point / 脚本入口
```python


if __name__ == "__main__":
    unittest.main()
```
**EN:** This range exposes the module as a directly runnable script, usually by delegating to a test runner or helper entry point. Representative call sites include `main`.
**CN:** 这一部分把模块暴露为可直接运行的脚本，通常会委托给测试运行器或辅助入口。 其中还会调用若干代表性的函数来串联完整流程。

## Key Concepts / 关键概念
- Unit testing / 单元测试
- Multi-GPU orchestration / 多 GPU 编排
- Quantization configuration / 量化配置
- Process lifecycle management / 进程生命周期管理
- HTTP/API interaction / HTTP/API 交互
- Token-level inspection / Token 级分析

## Dependencies / 依赖关系
- **Standard Library / 标准库**: `random`, `threading`, `time`, `unittest`
- **Third-party / 第三方库**: `requests`
- **Project Modules / 项目模块**: `sglang.srt.utils`, `sglang.test.test_utils`
