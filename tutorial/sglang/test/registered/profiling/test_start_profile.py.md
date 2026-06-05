# test_start_profile.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/profiling/test_start_profile.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates start profile behavior in SGLang's profiling area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 profiling 领域中与 start profile 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-11: supporting statements / 辅助语句
```python
"""
Usage:
# From the test/srt directory:
cd test/srt
python3 -m unittest test_start_profile.TestStartProfile
python3 -m unittest test_start_profile.TestStartProfileWithNsys

# Run specific tests:
python3 -m unittest test_start_profile.TestStartProfile.test_start_profile_1
python3 -m unittest test_start_profile.TestStartProfileWithNsys.test_start_profile_cuda_profiler
"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 13-30: module imports and dependencies / 模块导入与依赖
```python
import os
import shutil
import subprocess
import time
import unittest

import requests

from sglang.srt.environ import envs
from sglang.srt.utils import kill_process_tree
from sglang.test.ci.ci_register import register_amd_ci, register_cuda_ci
from sglang.test.test_utils import (
    DEFAULT_SMALL_MODEL_NAME_FOR_TEST,
    DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
    DEFAULT_URL_FOR_TEST,
    CustomTestCase,
    popen_launch_server,
)
```
**EN:** This block imports the modules needed by the rest of the file, including `os`, `shutil`, `subprocess`, `time`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `os`, `shutil`, `subprocess`, `time`。

### Lines 32-35: CI registration and metadata / CI 注册与元数据
```python
register_cuda_ci(est_time=42, stage="base-b", runner_config="1-gpu-small")
register_amd_ci(est_time=60, suite="stage-b-test-1-gpu-small-amd")

OUTPUT_DIR = "./profiler_dir"
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cuda_ci, register_amd_ci.
**CN:** 该代码块通过 register_cuda_ci, register_amd_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 38-44: function is nsys available / 函数 is nsys available
```python
def _is_nsys_available():
    """Check if nsys (Nsight Systems) is available on the system."""
    try:
        result = subprocess.run(["nsys", "--version"], capture_output=True, timeout=5)
        return result.returncode == 0
    except (FileNotFoundError, subprocess.TimeoutExpired):
        return False
```
**EN:** Check if nsys (Nsight Systems) is available on the system. This block implements `_is_nsys_available` and captures one focused piece of the module's behavior.
**CN:** Check if nsys (Nsight Systems) is available on the system. 该代码块实现 `_is_nsys_available`，承担模块行为中的一个聚焦逻辑片段。

### Lines 47-48: class TestStartProfile declaration / 类 TestStartProfile 声明
```python
class TestStartProfile(CustomTestCase):

```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 49-58: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        envs.SGLANG_TORCH_PROFILER_DIR.set(OUTPUT_DIR)
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        cls.base_url = DEFAULT_URL_FOR_TEST
        cls.process = popen_launch_server(
            cls.model,
            cls.base_url,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        )
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 60-62: tearDownClass cleanup routine / tearDownClass 清理流程
```python
    @classmethod
    def tearDownClass(cls):
        kill_process_tree(cls.process.pid)
```
**EN:** This routine releases resources and restores state after the related tests finish.
**CN:** 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 64-65: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        self._clear_profile_dir()
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 67-73: test case start profile 1 / 测试用例 start profile 1
```python
    def test_start_profile_1(self):
        """Test /start_profile with start_step and num_steps argument. This have to be the first test for start_step to work"""
        response = self._start_profile(start_step="15", num_steps=5)

        self._post_request()

        self._check_non_empty_profile_dir()
```
**EN:** Test /start_profile with start_step and num_steps argument. This test exercises `test_start_profile_1` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test /start_profile with start_step and num_steps argument. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_profile_1`。

### Lines 75-88: test case start profile 2 / 测试用例 start profile 2
```python
    def test_start_profile_2(self):
        """Test /start_profile with no argument"""
        response = self._start_profile()

        self._post_request()

        # Before /stop_profile, the profile directory should be empty
        self._check_empty_profile_dir()

        # Post /stop_profile and check the profile directory is non-empty
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/stop_profile",
        )
        self._check_non_empty_profile_dir()
```
**EN:** Test /start_profile with no argument This test exercises `test_start_profile_2` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test /start_profile with no argument 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_profile_2`。

### Lines 90-96: test case start profile 3 / 测试用例 start profile 3
```python
    def test_start_profile_3(self):
        """Test /start_profile with num_steps argument"""
        response = self._start_profile(num_steps=5)

        self._post_request()

        self._check_non_empty_profile_dir()
```
**EN:** Test /start_profile with num_steps argument This test exercises `test_start_profile_3` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test /start_profile with num_steps argument 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_profile_3`。

### Lines 98-104: method start profile / 方法 start profile
```python
    def _start_profile(self, **kwargs):
        """Start profiling with optional parameters."""
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/start_profile",
            json=kwargs if kwargs else None,
        )
        self.assertEqual(response.status_code, 200)
```
**EN:** Start profiling with optional parameters. This block implements `_start_profile` and captures one focused piece of the module's behavior.
**CN:** Start profiling with optional parameters. 该代码块实现 `_start_profile`，承担模块行为中的一个聚焦逻辑片段。

### Lines 106-117: method post request / 方法 post request
```python
    def _post_request(self):
        response = requests.post(
            f"{DEFAULT_URL_FOR_TEST}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 32,
                },
            },
        )
        self.assertEqual(response.status_code, 200)
```
**EN:** This block implements `_post_request` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_post_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 119-122: method clear profile dir / 方法 clear profile dir
```python
    def _clear_profile_dir(self):
        if os.path.isdir(OUTPUT_DIR):
            # Remove the directory and all its contents
            shutil.rmtree(OUTPUT_DIR)
```
**EN:** This block implements `_clear_profile_dir` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_clear_profile_dir`，承担模块行为中的一个聚焦逻辑片段。

### Lines 124-128: method check non empty profile dir / 方法 check non empty profile dir
```python
    def _check_non_empty_profile_dir(self):
        self.assertTrue(os.path.isdir(OUTPUT_DIR), "Output directory does not exist.")
        self.assertNotEqual(
            len(os.listdir(OUTPUT_DIR)), 0, "Output directory is empty!"
        )
```
**EN:** This block implements `_check_non_empty_profile_dir` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_check_non_empty_profile_dir`，承担模块行为中的一个聚焦逻辑片段。

### Lines 130-134: method check empty profile dir / 方法 check empty profile dir
```python
    def _check_empty_profile_dir(self):
        if os.path.isdir(OUTPUT_DIR):
            self.assertEqual(
                len(os.listdir(OUTPUT_DIR)), 0, "Output directory is non-empty!"
            )
```
**EN:** This block implements `_check_empty_profile_dir` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_check_empty_profile_dir`，承担模块行为中的一个聚焦逻辑片段。

### Lines 137-137: class TestStartProfileWithNsys declaration / 类 TestStartProfileWithNsys 声明
```python
class TestStartProfileWithNsys(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 138-141: supporting statements / 辅助语句
```python
    """Test /start_profile with CUDA_PROFILER (requires nsys wrapper)

    Each test starts its own clean server instance with nsys profiling.
    """
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 143-151: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        if not _is_nsys_available():
            raise unittest.SkipTest("nsys (Nsight Systems) is not available")

        envs.SGLANG_TORCH_PROFILER_DIR.set(OUTPUT_DIR)
        cls.model = DEFAULT_SMALL_MODEL_NAME_FOR_TEST
        # Use a different port to avoid conflicts with other tests
        cls.base_url = "http://127.0.0.1:21100"
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 153-176: setUp setup routine / setUp 初始化流程
```python
    def setUp(self):
        """Start a clean server with nsys for each test"""
        # Kill any existing processes on this port
        self._kill_existing_server()

        # Clean up old profile files for this test
        test_name = self.id().split(".")[-1]  # Get test method name
        self.nsys_output_file = f"nsys_profile_{test_name}"

        if os.path.isdir(OUTPUT_DIR):
            profile_file = os.path.join(OUTPUT_DIR, f"{self.nsys_output_file}.nsys-rep")
            if os.path.exists(profile_file):
                try:
                    os.remove(profile_file)
                except OSError:
                    pass

        # Launch server with nsys wrapper
        self.process = self._popen_launch_server_with_nsys(
            self.model,
            self.base_url,
            self.nsys_output_file,
            timeout=DEFAULT_TIMEOUT_FOR_SERVER_LAUNCH,
        )
```
**EN:** Start a clean server with nsys for each test This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** Start a clean server with nsys for each test 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 178-198: tearDown cleanup routine / tearDown 清理流程
```python
    def tearDown(self):
        """Kill server and verify profile was created"""

        # Kill server first to let nsys finalize the .nsys-rep file
        kill_process_tree(self.process.pid)

        # Also ensure nsys agent processes are killed
        try:
            subprocess.run(
                ["pkill", "-f", "nsys.*--start-agent"],
                timeout=5,
                stderr=subprocess.DEVNULL,
            )
        except (subprocess.TimeoutExpired, FileNotFoundError):
            pass

        # Give nsys a moment to finalize the report
        time.sleep(3)

        # Verify the .nsys-rep file was created
        self._verify_nsys_profile_created()
```
**EN:** Kill server and verify profile was created This routine releases resources and restores state after the related tests finish.
**CN:** Kill server and verify profile was created 该流程会在相关测试结束后释放资源并恢复状态。

### Lines 200-216: method kill existing server / 方法 kill existing server
```python
    def _kill_existing_server(self):
        """Kill any existing server process on our port and orphaned nsys agents"""
        try:
            # Kill server on our port
            subprocess.run(["lsof", "-ti", ":21100"], capture_output=True, timeout=5)
            subprocess.run(["pkill", "-f", "sglang.launch_server.*21100"], timeout=5)

            # Kill any orphaned nsys agent processes
            subprocess.run(
                ["pkill", "-f", "nsys.*--start-agent"],
                timeout=5,
                stderr=subprocess.DEVNULL,  # Suppress "no process found" errors
            )

            time.sleep(2)  # Wait for cleanup
        except (subprocess.TimeoutExpired, FileNotFoundError):
            pass
```
**EN:** Kill any existing server process on our port and orphaned nsys agents This block implements `_kill_existing_server` and captures one focused piece of the module's behavior.
**CN:** Kill any existing server process on our port and orphaned nsys agents 该代码块实现 `_kill_existing_server`，承担模块行为中的一个聚焦逻辑片段。

### Lines 218-285: method popen launch server with nsys / 方法 popen launch server with nsys
```python
    def _popen_launch_server_with_nsys(self, model, base_url, output_file, timeout):
        """Launch server wrapped with nsys profile -c cudaProfilerApi

        Each test gets its own output file for complete isolation.
        """
        _, host, port = base_url.split(":")
        host = host[2:]

        # Build the server launch command
        command = [
            "nsys",
            "profile",
            "-c",
            "cudaProfilerApi",
            "--capture-range-end",
            "stop",  # Stop after first cudaProfilerStop()
            "-o",
            os.path.join(OUTPUT_DIR, output_file),
            "python3",
            "-m",
            "sglang.launch_server",
            "--model-path",
            model,
            "--host",
            host,
            "--port",
            port,
        ]

        # Create output directory if it doesn't exist
        os.makedirs(OUTPUT_DIR, exist_ok=True)

        # Launch the process - capture output to keep test output clean
        process = subprocess.Popen(
            command,
            stdout=subprocess.PIPE,
            stderr=subprocess.STDOUT,
        )

        # Wait for server to be ready
        start_time = time.perf_counter()
        elapsed = 0
        with requests.Session() as session:
            while elapsed < timeout:
                elapsed = time.perf_counter() - start_time

                return_code = process.poll()
                if return_code is not None:
                    raise Exception(
                        f"Server process exited with code {return_code}. "
                        "Check server logs above for errors."
                    )

                try:
                    response = session.get(f"{base_url}/health_generate", timeout=5)
                    if response.status_code == 200:
                        return process
                except (requests.RequestException, requests.Timeout):
                    pass

                time.sleep(5)

        # Timeout reached
        kill_process_tree(process.pid)
        raise TimeoutError(
            f"Server failed to start within {timeout} seconds. "
            f"Check the server logs above for more information."
        )
```
**EN:** Launch server wrapped with nsys profile -c cudaProfilerApi Each test gets its own output file for complete isolation. This block implements `_popen_launch_server_with_nsys` and captures one focused piece of the module's behavior.
**CN:** Launch server wrapped with nsys profile -c cudaProfilerApi Each test gets its own output file for complete isolation. 该代码块实现 `_popen_launch_server_with_nsys`，承担模块行为中的一个聚焦逻辑片段。

### Lines 287-300: method verify nsys profile created / 方法 verify nsys profile created
```python
    def _verify_nsys_profile_created(self):
        """Verify that the .nsys-rep file was created after server shutdown."""
        if not os.path.isdir(OUTPUT_DIR):
            raise AssertionError("Output directory does not exist.")

        expected_file = f"{self.nsys_output_file}.nsys-rep"
        profile_path = os.path.join(OUTPUT_DIR, expected_file)

        if not os.path.exists(profile_path):
            files = os.listdir(OUTPUT_DIR)
            raise AssertionError(
                f"Expected profile file '{expected_file}' not found. "
                f"Files present: {files}"
            )
```
**EN:** Verify that the .nsys-rep file was created after server shutdown. This block implements `_verify_nsys_profile_created` and captures one focused piece of the module's behavior.
**CN:** Verify that the .nsys-rep file was created after server shutdown. 该代码块实现 `_verify_nsys_profile_created`，承担模块行为中的一个聚焦逻辑片段。

### Lines 302-309: test case start profile cuda profiler with start step / 测试用例 start profile cuda profiler with start step
```python
    def test_start_profile_cuda_profiler_with_start_step(self):
        """Test /start_profile with CUDA_PROFILER, start_step, and num_steps"""
        # Use start_step to let server warm up before profiling
        response = self._start_profile(
            activities=["CUDA_PROFILER"], start_step=10, num_steps=3
        )

        self._post_request()
```
**EN:** Test /start_profile with CUDA_PROFILER, start_step, and num_steps This test exercises `test_start_profile_cuda_profiler_with_start_step` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test /start_profile with CUDA_PROFILER, start_step, and num_steps 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_profile_cuda_profiler_with_start_step`。

### Lines 310-312: supporting source context / 辅助源码上下文
```python

        # Profile verification happens in tearDown()

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 313-318: test case start profile cuda profiler / 测试用例 start profile cuda profiler
```python
    def test_start_profile_cuda_profiler(self):
        """Test /start_profile with CUDA_PROFILER activity (no start_step)"""
        # Simple num_steps test - profiling starts immediately
        response = self._start_profile(activities=["CUDA_PROFILER"], num_steps=5)

        self._post_request()
```
**EN:** Test /start_profile with CUDA_PROFILER activity (no start_step) This test exercises `test_start_profile_cuda_profiler` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test /start_profile with CUDA_PROFILER activity (no start_step) 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_profile_cuda_profiler`。

### Lines 319-321: supporting source context / 辅助源码上下文
```python

        # Profile verification happens in tearDown()

```
**EN:** This block preserves nearby source lines that connect the surrounding definitions.
**CN:** 该代码块保留衔接周围定义的相邻源码。

### Lines 322-329: method start profile / 方法 start profile
```python
    def _start_profile(self, **kwargs):
        """Start profiling with optional parameters."""
        response = requests.post(
            f"{self.base_url}/start_profile",
            json=kwargs if kwargs else None,
        )
        self.assertEqual(response.status_code, 200)
        return response
```
**EN:** Start profiling with optional parameters. This block implements `_start_profile` and captures one focused piece of the module's behavior.
**CN:** Start profiling with optional parameters. 该代码块实现 `_start_profile`，承担模块行为中的一个聚焦逻辑片段。

### Lines 331-342: method post request / 方法 post request
```python
    def _post_request(self):
        response = requests.post(
            f"{self.base_url}/generate",
            json={
                "text": "The capital of France is",
                "sampling_params": {
                    "temperature": 0,
                    "max_new_tokens": 32,
                },
            },
        )
        self.assertEqual(response.status_code, 200)
```
**EN:** This block implements `_post_request` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_post_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 345-346: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_is_nsys_available`: Check if nsys (Nsight Systems) is available on the system. / 该代码块实现 `_is_nsys_available`，承担模块行为中的一个聚焦逻辑片段。
- `TestStartProfile`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestStartProfileWithNsys`: Test /start_profile with CUDA_PROFILER (requires nsys wrapper) Each test starts its own clean server instance with nsys profiling. / 用于组织相关测试、夹具或辅助方法。
- `TestStartProfile.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestStartProfile.tearDownClass`: This routine releases resources and restores state after the related tests finish. / 该流程会在相关测试结束后释放资源并恢复状态。
- `TestStartProfile.setUp`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestStartProfile.test_start_profile_1`: Test /start_profile with start_step and num_steps argument. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_profile_1`。
- `TestStartProfile.test_start_profile_2`: Test /start_profile with no argument / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_profile_2`。
- `TestStartProfile.test_start_profile_3`: Test /start_profile with num_steps argument / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_start_profile_3`。
- `TestStartProfile._start_profile`: Start profiling with optional parameters. / 该代码块实现 `_start_profile`，承担模块行为中的一个聚焦逻辑片段。
- `TestStartProfile._post_request`: This block implements `_post_request` and captures one focused piece of the module's behavior. / 该代码块实现 `_post_request`，承担模块行为中的一个聚焦逻辑片段。
- `TestStartProfile._clear_profile_dir`: This block implements `_clear_profile_dir` and captures one focused piece of the module's behavior. / 该代码块实现 `_clear_profile_dir`，承担模块行为中的一个聚焦逻辑片段。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `os`, `shutil`, `subprocess`, `time`, `unittest`
- **Third-party modules / 第三方模块**: `requests`
- **Internal modules / 内部模块**: `sglang.srt.environ`, `sglang.srt.utils`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 346
