# test_minifier_common.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/_dynamo/test_minifier_common.py`
- **Repository / 仓库**: `pytorch/pytorch`
- **Purpose (EN)**: Implements torch.compile front-end capture, guard management, and graph/lowering helpers in TorchDynamo. The module docstring emphasizes: Common utilities for testing Dynamo's minifier functionality.
- **Purpose (CN)**: 实现 TorchDynamo 中的 torch.compile 前端捕获、守卫管理与图/降级辅助逻辑。 模块文档字符串重点说明了该实现的职责。

## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
"""Common utilities for testing Dynamo's minifier functionality.

This module provides the base infrastructure for running minification tests in Dynamo.
It includes:
- MinifierTestResult: A dataclass for storing and processing minifier test results
- MinifierTestBase: A base test class with utilities for:
  - Running tests in isolated environments
  - Managing temporary directories and configurations
  - Executing minifier launcher scripts
  - Running and validating reproduction scripts
  - Supporting both compile-time and runtime error testing

The minifier helps reduce failing Dynamo compilations to minimal reproductions.
"""

import dataclasses
import io
import logging
```
- **EN**: This docstring explains the surrounding API surface and gives readers the semantic context for the implementation.
- **CN**: 这一文档字符串解释了周边 API 的语义背景，帮助读者理解实现。

### Lines 19-41
```python
import os
import re
import shutil
import subprocess
import sys
import tempfile
import traceback
from collections.abc import Sequence
from typing import Any
from unittest.mock import patch

import torch
import torch._dynamo
import torch._dynamo.test_case
from torch._dynamo.trace_rules import _as_posix_path
from torch.utils._traceback import report_compile_source_on_error


@dataclasses.dataclass
class MinifierTestResult:
    minifier_code: str
    repro_code: str
```
- **EN**: These decorators register or transform the following definition so it can trace Python execution into an intermediate graph representation.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将 Python 执行过程跟踪为中间图表示。

### Lines 42-68
```python
    def _get_module(self, t: str) -> str:
        match = re.search(r"class Repro\(torch\.nn\.Module\):\s+([ ].*\n| *\n)+", t)
        assert match is not None, "failed to find module"
        r = match.group(0)
        r = re.sub(r"\s+$", "\n", r, flags=re.MULTILINE)
        r = re.sub(r"\n{3,}", "\n\n", r)
        return r.strip()

    def get_exported_program_path(self) -> str | None:
        # Extract the exported program file path from AOTI minifier's repro.py
        # Regular expression pattern to match the file path
        pattern = r'torch\.export\.load\(\s*["\'](.*?)["\']\s*\)'
        # Search for the pattern in the text
        match = re.search(pattern, self.repro_code)
        # Extract and print the file path if a match is found
        if match:
            file_path = match.group(1)
            return file_path
        return None

    def minifier_module(self) -> str:
        return self._get_module(self.minifier_code)

    def repro_module(self) -> str:
        return self._get_module(self.repro_code)
```
- **EN**: This module-level block helps organize reusable module behavior and state. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 这个模块级代码块用于组织可复用的模块行为与状态。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 69-94
```python
class MinifierTestBase(torch._dynamo.test_case.TestCase):
    DEBUG_DIR = tempfile.mkdtemp()

    @classmethod
    def setUpClass(cls) -> None:
        super().setUpClass()
        if not os.path.exists(cls.DEBUG_DIR):
            cls.DEBUG_DIR = tempfile.mkdtemp()
        cls._exit_stack.enter_context(  # type: ignore[attr-defined]
            torch._dynamo.config.patch(debug_dir_root=cls.DEBUG_DIR)
        )
        # These configurations make new process startup slower.  Disable them
        # for the minification tests to speed them up.
        cls._exit_stack.enter_context(  # type: ignore[attr-defined]
            torch._inductor.config.patch(
                {
                    # https://github.com/pytorch/pytorch/issues/100376
                    "pattern_matcher": False,
                    # multiprocess compilation takes a long time to warmup
                    "compile_threads": 1,
                    # https://github.com/pytorch/pytorch/issues/100378
                    "cpp.vec_isa_ok": False,
                }
            )
        )
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 95-115
```python
    @classmethod
    def tearDownClass(cls) -> None:
        if os.getenv("PYTORCH_KEEP_TMPDIR", "0") != "1":
            shutil.rmtree(cls.DEBUG_DIR)
        else:
            print(f"test_minifier_common tmpdir kept at: {cls.DEBUG_DIR}")
        cls._exit_stack.close()  # type: ignore[attr-defined]

    def _gen_codegen_fn_patch_code(self, device: str, bug_type: str) -> str:
        assert bug_type in ("compile_error", "runtime_error", "accuracy")
        return f"""\
{torch._dynamo.config.codegen_config()}
{torch._inductor.config.codegen_config()}
torch._inductor.config.{"cpp" if device == "cpu" else "triton"}.inject_relu_bug_TESTING_ONLY = {bug_type!r}
"""

    def _maybe_subprocess_run(
        self, args: Sequence[Any], *, isolate: bool, cwd: str | None = None
    ) -> subprocess.CompletedProcess[bytes]:
        from torch._inductor.cpp_builder import normalize_path_separator
```
- **EN**: These decorators register or transform the following definition so it can hand work to a compiler/backend pipeline.
- **CN**: 这些装饰器会注册或改造后续定义，使其能够将工作移交给编译器或后端流水线。

### Lines 116-134
```python
        if not isolate:
            assert len(args) >= 2, args
            assert args[0] == "python3", args
            if args[1] == "-c":
                assert len(args) == 3, args
                code = args[2]
                args = ["-c"]
            else:
                assert len(args) >= 2, args
                with open(args[1]) as f:
                    # Need normalize path of the code.
                    code = normalize_path_separator(f.read())
                args = args[1:]

            # WARNING: This is not a perfect simulation of running
            # the program out of tree.  We only interpose on things we KNOW we
            # need to handle for tests.  If you need more stuff, you will
            # need to augment this appropriately.
```
- **EN**: This block continues `MinifierTestBase._maybe_subprocess_run` and works to capture Python execution for torch.compile and maintain compiler state. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `MinifierTestBase._maybe_subprocess_run`，用于为 torch.compile 捕获 Python 执行并维护编译器状态。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 135-152
```python
            # NB: Can't use save_config because that will omit some fields,
            # but we must save and reset ALL fields
            dynamo_config = torch._dynamo.config.get_config_copy()
            inductor_config = torch._inductor.config.get_config_copy()
            try:
                stderr = io.StringIO()
                log_handler = logging.StreamHandler(stderr)
                log = logging.getLogger("torch._dynamo")
                log.addHandler(log_handler)
                try:
                    prev_cwd = _as_posix_path(os.getcwd())
                    if cwd is not None:
                        cwd = _as_posix_path(cwd)
                        os.chdir(cwd)
                    with patch("sys.argv", args), report_compile_source_on_error():
                        exec(code, {"__name__": "__main__", "__compile_source__": code})
                    rc = 0
                except Exception:
```
- **EN**: This block continues `MinifierTestBase._maybe_subprocess_run` and works to hand work to a compiler/backend pipeline. It also branches on runtime conditions; uses a scoped helper/context manager.
- **CN**: 该代码块继续实现 `MinifierTestBase._maybe_subprocess_run`，用于将工作移交给编译器或后端流水线。 同时它还会根据运行时条件分支处理；使用带作用域的辅助对象或上下文管理器。

### Lines 153-177
```python
                    rc = 1
                    traceback.print_exc(file=stderr)
                finally:
                    log.removeHandler(log_handler)
                    if cwd is not None:
                        os.chdir(prev_cwd)  # type: ignore[possibly-undefined]
                    # Make sure we don't leave buggy compiled frames lying
                    # around
                    torch._dynamo.reset()
            finally:
                torch._dynamo.config.load_config(dynamo_config)
                torch._inductor.config.load_config(inductor_config)

            # TODO: return a more appropriate data structure here
            return subprocess.CompletedProcess(
                args,
                rc,
                b"",
                stderr.getvalue().encode("utf-8"),
            )
        else:
            if cwd is not None:
                cwd = _as_posix_path(cwd)
            return subprocess.run(args, capture_output=True, cwd=cwd, check=False)
```
- **EN**: This block continues `MinifierTestBase` and works to trace Python execution into an intermediate graph representation. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `MinifierTestBase`，用于将 Python 执行过程跟踪为中间图表示。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 178-196
```python
    # Run `code` in a separate python process.
    # Returns the completed process state and the directory containing the
    # minifier launcher script, if `code` outputted it.
    def _run_test_code(
        self, code: str, *, isolate: bool
    ) -> tuple[subprocess.CompletedProcess[bytes], str | Any]:
        proc = self._maybe_subprocess_run(
            ["python3", "-c", code], isolate=isolate, cwd=self.DEBUG_DIR
        )

        print("test stdout:", proc.stdout.decode("utf-8"))
        print("test stderr:", proc.stderr.decode("utf-8"))
        repro_dir_match = re.search(
            r"(\S+)minifier_launcher.py", proc.stderr.decode("utf-8")
        )
        if repro_dir_match is not None:
            return proc, repro_dir_match.group(1)
        return proc, None
```
- **EN**: Declares `MinifierTestBase(torch._dynamo.test_case.TestCase)`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `MinifierTestBase(torch._dynamo.test_case.TestCase)`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 197-222
```python
    # Runs the minifier launcher script in `repro_dir`
    def _run_minifier_launcher(
        self,
        repro_dir: str,
        isolate: bool,
        *,
        minifier_args: Sequence[Any] = (),
        repro_after: str | None = None,
    ) -> tuple[subprocess.CompletedProcess[bytes], str]:
        self.assertIsNotNone(repro_dir)
        launch_file = _as_posix_path(os.path.join(repro_dir, "minifier_launcher.py"))
        with open(launch_file) as f:
            launch_code = f.read()

        self.assertTrue(os.path.exists(launch_file))

        args = ["python3", launch_file, "minify", *minifier_args]
        if not isolate and repro_after != "aot_inductor":
            # AOTI minifier doesn't have --no-isolate flag.
            # Everything in AOTI minifier is in no-isolate mode.
            args.append("--no-isolate")
        launch_proc = self._maybe_subprocess_run(args, isolate=isolate, cwd=repro_dir)
        print("minifier stdout:", launch_proc.stdout.decode("utf-8"))
        stderr = launch_proc.stderr.decode("utf-8")
        print("minifier stderr:", stderr)
```
- **EN**: Declares `MinifierTestBase(torch._dynamo.test_case.TestCase)`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `MinifierTestBase(torch._dynamo.test_case.TestCase)`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 223-244
```python
        self.assertNotIn("Input graph did not fail the tester", stderr)

        return launch_proc, launch_code

    # Runs the repro script in `repro_dir`
    def _run_repro(
        self, repro_dir: str, *, isolate: bool = True
    ) -> tuple[subprocess.CompletedProcess[bytes], str]:
        self.assertIsNotNone(repro_dir)
        repro_file = _as_posix_path(os.path.join(repro_dir, "repro.py"))
        with open(repro_file) as f:
            repro_code = f.read()

        self.assertTrue(os.path.exists(repro_file))

        repro_proc = self._maybe_subprocess_run(
            ["python3", repro_file], isolate=isolate, cwd=repro_dir
        )
        print("repro stdout:", repro_proc.stdout.decode("utf-8"))
        print("repro stderr:", repro_proc.stderr.decode("utf-8"))
        return repro_proc, repro_code
```
- **EN**: Declares `MinifierTestBase(torch._dynamo.test_case.TestCase)`; this class packages state and methods that build, traverse, or rewrite graph structures and their metadata.
- **CN**: 声明 `MinifierTestBase(torch._dynamo.test_case.TestCase)`；该类封装了状态与方法，用于构建、遍历或改写图结构及其元数据。

### Lines 245-271
```python
    # Template for testing code.
    # `run_code` is the code to run for the test case.
    # `patch_code` is the code to be patched in every generated file; usually
    # just use this to turn on bugs via the config
    def _gen_test_code(self, run_code: str, repro_after: str, repro_level: int) -> str:
        repro_after_line = ""
        if repro_after == "aot_inductor":
            repro_after_line = (
                "torch._inductor.config.aot_inductor.dump_aoti_minifier = True"
            )
        elif repro_after:
            repro_after_line = f"""\
torch._dynamo.config.repro_after = "{repro_after}"
        """
        return f"""\
import torch
import torch._dynamo
import torch._inductor
{_as_posix_path(torch._dynamo.config.codegen_config())}
{_as_posix_path(torch._inductor.config.codegen_config())}
{repro_after_line}
torch._dynamo.config.repro_level = {repro_level}
torch._inductor.config.aot_inductor.repro_level = {repro_level}
torch._dynamo.config.debug_dir_root = "{_as_posix_path(self.DEBUG_DIR)}"
{run_code}
"""
```
- **EN**: Declares `MinifierTestBase(torch._dynamo.test_case.TestCase)`; this class packages state and methods that capture Python execution for torch.compile and maintain compiler state.
- **CN**: 声明 `MinifierTestBase(torch._dynamo.test_case.TestCase)`；该类封装了状态与方法，用于为 torch.compile 捕获 Python 执行并维护编译器状态。

### Lines 272-289
```python
    # Runs a full minifier test.
    # Minifier tests generally consist of 3 stages:
    # 1. Run the problematic code
    # 2. Run the generated minifier launcher script
    # 3. Run the generated repro script
    #
    # If possible, you should run the test with isolate=False; use
    # isolate=True only if the bug you're testing would otherwise
    # crash the process
    def _run_full_test(
        self,
        run_code: str,
        repro_after: str,
        expected_error: str | None,
        *,
        isolate: bool,
        minifier_args: Sequence[Any] = (),
    ) -> MinifierTestResult | None:
```
- **EN**: Declares `MinifierTestBase(torch._dynamo.test_case.TestCase)`; this class packages state and methods that validate invariants and surface meaningful failures.
- **CN**: 声明 `MinifierTestBase(torch._dynamo.test_case.TestCase)`；该类封装了状态与方法，用于校验不变量并给出有意义的失败信息。

### Lines 290-309
```python
        if isolate:
            repro_level = 3
        elif expected_error is None or expected_error == "AccuracyError":
            repro_level = 4
        else:
            repro_level = 2
        test_code = self._gen_test_code(run_code, repro_after, repro_level)
        print("running test", file=sys.stderr)
        test_proc, repro_dir = self._run_test_code(test_code, isolate=isolate)
        if expected_error is None:
            # Just check that there was no error
            self.assertEqual(test_proc.returncode, 0)

            self.assertIsNone(repro_dir)
            return None
        # NB: Intentionally do not test return code; we only care about
        # actually generating the repro, we don't have to crash

        self.assertIn(expected_error, test_proc.stderr.decode("utf-8"))
```
- **EN**: This block continues `MinifierTestBase._run_full_test` and works to validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value; branches on runtime conditions.
- **CN**: 该代码块继续实现 `MinifierTestBase._run_full_test`，用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果；根据运行时条件分支处理。

### Lines 310-323
```python
        self.assertIsNotNone(repro_dir)
        print("running minifier", file=sys.stderr)
        _minifier_proc, minifier_code = self._run_minifier_launcher(
            repro_dir,
            isolate=isolate,
            minifier_args=minifier_args,
            repro_after=repro_after,
        )
        print("running repro", file=sys.stderr)
        repro_proc, repro_code = self._run_repro(repro_dir, isolate=isolate)

        self.assertIn(expected_error, repro_proc.stderr.decode("utf-8"))
        self.assertNotEqual(repro_proc.returncode, 0)
        return MinifierTestResult(minifier_code=minifier_code, repro_code=repro_code)
```
- **EN**: This block continues `MinifierTestBase._run_full_test` and works to validate invariants and surface meaningful failures. It also computes and returns an intermediate/result value.
- **CN**: 该代码块继续实现 `MinifierTestBase._run_full_test`，用于校验不变量并给出有意义的失败信息。 同时它还会计算并返回中间值或结果。

## Key Concepts / 关键概念

- **Graph capture / 图捕获**
  - EN: The file participates in converting eager Python execution into compilable graph form.
  - CN: 该文件参与把即时执行的 Python 程序转换为可编译的图形式。
- **Graph IR / 图中间表示**
  - EN: The implementation manipulates graph nodes, edges, or metadata as an explicit intermediate representation.
  - CN: 该实现把图节点、边或元数据作为显式的中间表示来处理。
- **Tracing / 跟踪**
  - EN: The file captures Python-level operations and turns them into analyzable graph structures.
  - CN: 该文件会捕获 Python 层操作，并将其转换为可分析的图结构。
- **Module state / 模块状态**
  - EN: The code manages reusable module state, learnable parameters, or forward-path behavior.
  - CN: 代码负责管理可复用的模块状态、可学习参数或前向路径行为。
- **Compiler handoff / 编译器交接**
  - EN: The implementation prepares captured work for backend compilation or execution.
  - CN: 该实现会把捕获到的工作整理后交给后端编译或执行。
- **Python execution capture / Python 执行捕获**
  - EN: The file hooks or rewrites Python execution machinery to observe user programs.
  - CN: 该文件会挂接或改写 Python 执行机制，以观察用户程序。
## Dependencies / 依赖关系

- **Internal PyTorch modules / PyTorch 内部模块**: `torch`, `torch._dynamo`, `torch._dynamo.test_case`, `torch._dynamo.trace_rules`, `torch.utils._traceback`, `torch._inductor.cpp_builder`
- **Standard library / 标准库**: `dataclasses`, `io`, `logging`, `os`, `re`, `shutil`, `subprocess`, `sys`, `tempfile`, `traceback`, `collections.abc`, `typing`, `unittest.mock`
- **Primary symbols / 核心符号**: `MinifierTestResult`, `MinifierTestBase`
- **Relationship summary / 关系总结**: Dependencies center on graph capture, guard checking, bytecode analysis, and backend handoff. / 依赖关系主要围绕图捕获、守卫检查、字节码分析和后端交接展开。
