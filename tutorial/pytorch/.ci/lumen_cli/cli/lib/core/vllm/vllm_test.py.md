# vllm_test.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `.ci/lumen_cli/cli/lib/core/vllm/vllm_test.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements Python-side logic used for benchmarking, tooling, validation, or repository automation. Backend or dtype coverage is a visible axis of variation in the implementation.
- **Purpose (CN)**: 实现用于基准、工具链、校验或仓库自动化的 Python 侧逻辑。 后端或 dtype 覆盖是该实现中明显的变化维度。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11 / 第 1-11 行

```python
import logging
import os
import re
import subprocess
import sys
from collections.abc import Iterable
from dataclasses import dataclass
from enum import Enum
from pathlib import Path
from typing import Any

```

- **EN:** This range mainly pulls in the imports/includes that the later benchmark or integration logic depends on.
- **CN:** 这一段主要引入后续基准或集成逻辑依赖的 import/include。
- **EN:** Important local symbols in this block include import.
- **CN:** 该代码块中的重要局部符号包括 import。

### Lines 12-23 / 第 12-23 行

```python
from cli.lib.common.cli_helper import BaseRunner
from cli.lib.common.envs_helper import env_path_field, env_str_field, get_env
from cli.lib.common.path_helper import copy, get_path, remove_dir
from cli.lib.common.pip_helper import (
    pip_install_first_match,
    pip_install_packages,
    pkg_exists,
    run_python,
)
from cli.lib.common.utils import run_command, working_directory
from cli.lib.core.vllm.lib import clone_vllm, run_test_plan, sample_vllm_test_library

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 24-32 / 第 24-32 行

```python

logger = logging.getLogger(__name__)


@dataclass
class VllmTestParameters:
    """
    Parameters defining the vllm external test input

```

- **EN:** Important local symbols in this block include VllmTestParameters.
- **CN:** 该代码块中的重要局部符号包括 VllmTestParameters。

### Lines 33-43 / 第 33-43 行

```python
    !!!DO NOT ADD SECRETS IN THIS CLASS!!!
    you can put environment variable name in VllmTestParameters if it's not the same as the secret one
    fetch secrests directly from env variables during runtime
    """

    torch_whls_path: Path = env_path_field("WHEELS_PATH", "./dist")

    vllm_whls_path: Path = env_path_field(
        "VLLM_WHEELS_PATH", "./dist/external/vllm/wheels"
    )

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 44-51 / 第 44-51 行

```python
    torch_cuda_arch_list: str = env_str_field("TORCH_CUDA_ARCH_LIST", "8.9")

    cleaning_script: Path = env_path_field(
        "cleaning_script", ".github/ci_configs/vllm/use_existing_torch.py"
    )

    def __post_init__(self):
        if not self.torch_whls_path.exists():
```

- **EN:** Important local symbols in this block include __post_init__.
- **CN:** 该代码块中的重要局部符号包括 __post_init__。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 52-60 / 第 52-60 行

```python
            raise ValueError("missing torch_whls_path")
        if not self.vllm_whls_path.exists():
            raise ValueError("missing vllm_whls_path")


class TestInpuType(Enum):
    TEST_PLAN = "test_plan"
    UNKNOWN = "unknown"

```

- **EN:** Important local symbols in this block include TestInpuType.
- **CN:** 该代码块中的重要局部符号包括 TestInpuType。

### Lines 61-70 / 第 61-70 行

```python

class VllmTestRunner(BaseRunner):
    def __init__(self, args: Any):
        self.work_directory = "vllm"
        self.test_plan = ""
        self.test_type = TestInpuType.UNKNOWN

        self.shard_id = args.shard_id
        self.num_shards = args.num_shards

```

- **EN:** Important local symbols in this block include VllmTestRunner, __init__.
- **CN:** 该代码块中的重要局部符号包括 VllmTestRunner、__init__。

### Lines 71-82 / 第 71-82 行

```python
        if args.test_plan:
            self.test_plan = args.test_plan
            self.test_type = TestInpuType.TEST_PLAN

        # Matches the structeur in the artifacts.zip from torcb build
        self.TORCH_WHL_PATH_REGEX = "torch*.whl"
        self.TORCH_WHL_EXTRA = "opt-einsum"
        self.TORCH_ADDITIONAL_WHLS_REGEX = [
            "vision/torchvision*.whl",
            "audio/torchaudio*.whl",
        ]

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 83-95 / 第 83-95 行

```python
        # Match the structure of the artifacts.zip from vllm external build
        self.VLLM_TEST_WHLS_REGEX = [
            "vllm/vllm*.whl",
        ]

    def prepare(self):
        """
        prepare test environment for vllm. This includes clone vllm repo, install all wheels, test dependencies and set env
        """
        params = VllmTestParameters()
        logger.info("Display VllmTestParameters %s", params)
        self._set_envs(params)

```

- **EN:** Important local symbols in this block include prepare.
- **CN:** 该代码块中的重要局部符号包括 prepare。

### Lines 96-103 / 第 96-103 行

```python
        clone_vllm(dst=self.work_directory)
        self.cp_torch_cleaning_script(params)
        with working_directory(self.work_directory):
            remove_dir(Path("vllm"))
            self._install_wheels(params)
            self._install_dependencies()
        # verify the torches are not overridden by test dependencies

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 104-113 / 第 104-113 行

```python
        check_versions()

    def run(self):
        """
        main function to run vllm test
        """
        self.prepare()
        try:
            with working_directory(self.work_directory):
                if self.test_type == TestInpuType.TEST_PLAN:
```

- **EN:** Important local symbols in this block include run.
- **CN:** 该代码块中的重要局部符号包括 run。

### Lines 114-129 / 第 114-129 行

```python
                    if self.num_shards > 1:
                        run_test_plan(
                            self.test_plan,
                            "vllm",
                            sample_vllm_test_library(),
                            self.shard_id,
                            self.num_shards,
                        )
                    else:
                        run_test_plan(
                            self.test_plan, "vllm", sample_vllm_test_library()
                        )
                else:
                    raise ValueError(f"Unknown test type {self.test_type}")
        finally:
            # double check the torches are not overridden by other packages
```

- **EN:** Conditional branches separate validation paths, special cases, or backend-specific behavior.
- **CN:** 条件分支用于区分校验路径、特殊情况或后端专用行为。

### Lines 130-137 / 第 130-137 行

```python
            check_versions()

    def cp_torch_cleaning_script(self, params: VllmTestParameters):
        script = get_path(params.cleaning_script, resolve=True)
        vllm_script = Path(f"./{self.work_directory}/use_existing_torch.py")
        copy(script, vllm_script)

    def _install_wheels(self, params: VllmTestParameters):
```

- **EN:** Important local symbols in this block include cp_torch_cleaning_script, _install_wheels.
- **CN:** 该代码块中的重要局部符号包括 cp_torch_cleaning_script、_install_wheels。

### Lines 138-146 / 第 138-146 行

```python
        logger.info("Running vllm test with inputs: %s", params)
        if not pkg_exists("torch"):
            # install torch from local whls if it's not installed yet.
            torch_p = f"{str(params.torch_whls_path)}/{self.TORCH_WHL_PATH_REGEX}"
            pip_install_first_match(torch_p, self.TORCH_WHL_EXTRA)

        torch_whls_path = [
            f"{str(params.torch_whls_path)}/{whl_path}"
            for whl_path in self.TORCH_ADDITIONAL_WHLS_REGEX
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 147-155 / 第 147-155 行

```python
        ]
        for torch_whl in torch_whls_path:
            pip_install_first_match(torch_whl)
        logger.info("Done. Installed torch and other torch-related wheels ")

        logger.info("Installing vllm wheels")
        vllm_whls_path = [
            f"{str(params.vllm_whls_path)}/{whl_path}"
            for whl_path in self.VLLM_TEST_WHLS_REGEX
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 156-171 / 第 156-171 行

```python
        ]
        for vllm_whl in vllm_whls_path:
            pip_install_first_match(vllm_whl)
        logger.info("Done. Installed vllm wheels")

    def _install_test_dependencies(self):
        """
        This method replaces torch dependencies with local torch wheel info in
        requirements/test/cuda.in file from vllm repo. then generates the test.txt
        in runtime
        """
        logger.info(
            "generate test.txt from requirements/test/cuda.in with local torch whls"
        )
        preprocess_test_in()
        copy("requirements/test/cuda.txt", "snapshot_constraint.txt")
```

- **EN:** Important local symbols in this block include _install_test_dependencies.
- **CN:** 该代码块中的重要局部符号包括 _install_test_dependencies。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 172-182 / 第 172-182 行

```python

        run_command(
            f"{sys.executable} -m uv pip compile requirements/test/cuda.in "
            "-o test/cuda.txt "
            "--index-strategy unsafe-best-match "
            "--constraint snapshot_constraint.txt "
            "--torch-backend cu129"
        )
        pip_install_packages(requirements="test/cuda.txt", prefer_uv=True)
        logger.info("Done. installed requirements for test dependencies")

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 183-190 / 第 183-190 行

```python
    def _install_dependencies(self):
        pip_install_packages(packages=["-e", "tests/vllm_test_utils"], prefer_uv=True)
        pip_install_packages(packages=["hf_transfer"], prefer_uv=True)
        os.environ["HF_HUB_ENABLE_HF_TRANSFER"] = "1"

        # using script from vllm repo to remove all torch packages from requirements txt
        run_python("use_existing_torch.py")

```

- **EN:** Important local symbols in this block include _install_dependencies.
- **CN:** 该代码块中的重要局部符号包括 _install_dependencies。

### Lines 191-199 / 第 191-199 行

```python
        # install common packages
        for requirements in ["requirements/common.txt", "requirements/build/cuda.txt"]:
            pip_install_packages(
                requirements=requirements,
                prefer_uv=True,
            )
        # install test packages
        self._install_test_dependencies()

```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 200-208 / 第 200-208 行

```python
    def _set_envs(self, inputs: VllmTestParameters):
        os.environ["TORCH_CUDA_ARCH_LIST"] = inputs.torch_cuda_arch_list
        if not validate_cuda(get_env("TORCH_CUDA_ARCH_LIST")):
            logger.warning(
                "Missing supported TORCH_CUDA_ARCH_LIST. "
                "Currently support TORCH_CUDA_ARCH_LIST env var "
                "with supported arch [8.0, 8.9, 9.0]"
            )

```

- **EN:** Important local symbols in this block include _set_envs.
- **CN:** 该代码块中的重要局部符号包括 _set_envs。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 209-220 / 第 209-220 行

```python
        os.environ["HF_TOKEN"] = os.getenv("VLLM_TEST_HUGGING_FACE_TOKEN", "")
        if not get_env("HF_TOKEN"):
            raise ValueError(
                "missing required HF_TOKEN, please set VLLM_TEST_HUGGING_FACE_TOKEN env var"
            )
        if not get_env("TORCH_CUDA_ARCH_LIST"):
            raise ValueError(
                "missing required TORCH_CUDA_ARCH_LIST, please set TORCH_CUDA_ARCH_LIST env var"
            )
        # HF_HOME is absolutely needed on CI to avoid rate limit to HF, so explicitly fail
        # vLLM jobs when it's not set so that we know when it's missing
        if get_env("CI") and not get_env("HF_HOME"):
```

- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 221-236 / 第 221-236 行

```python
            raise ValueError(
                "missing required HF_HOME when running on CI, please set HF_HOME env var"
            )


def preprocess_test_in(
    target_file: str = "requirements/test/cuda.in",
    additional_packages: Iterable[str] = (),
):
    """
    This modifies the target_file file in place in vllm work directory.
    It removes torch and unwanted packages in target_file and replace with local torch whls
    package with format "$WHEEL_PACKAGE_NAME @ file://<LOCAL_PATH>"
    """
    additional_package_to_move = list(additional_packages or ())
    pkgs_to_remove = [
```

- **EN:** Important local symbols in this block include preprocess_test_in.
- **CN:** 该代码块中的重要局部符号包括 preprocess_test_in。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 237-244 / 第 237-244 行

```python
        "torch",
        "torchvision",
        "torchaudio",
    ] + additional_package_to_move
    # Read current requirements
    target_path = Path(target_file)
    lines = target_path.read_text().splitlines()

```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 245-256 / 第 245-256 行

```python
    pkgs_to_add = []

    # Remove lines starting with the package names (==, @, >=) — case-insensitive
    pattern = re.compile(rf"^({'|'.join(pkgs_to_remove)})\s*(==|@|>=)", re.IGNORECASE)
    kept_lines = [line for line in lines if not pattern.match(line)]

    # Get local installed torch/vision/audio from pip freeze
    # This is hacky, but it works
    pip_freeze = subprocess.check_output(["pip", "freeze"], text=True)
    header_lines = [
        line
        for line in pip_freeze.splitlines()
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 257-264 / 第 257-264 行

```python
        if re.match(
            r"^(torch|torchvision|torchaudio)\s*@\s*file://", line, re.IGNORECASE
        )
    ]

    # Write back: header_lines + blank + kept_lines
    out_lines = header_lines + [""] + kept_lines
    if pkgs_to_add:
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

### Lines 265-272 / 第 265-272 行

```python
        out_lines += [""] + pkgs_to_add

    out = "\n".join(out_lines) + "\n"
    target_path.write_text(out)
    logger.info("[INFO] Updated %s", target_file)


def validate_cuda(value: str) -> bool:
```

- **EN:** Important local symbols in this block include validate_cuda.
- **CN:** 该代码块中的重要局部符号包括 validate_cuda。
- **EN:** Type or device branching is visible, showing how the implementation adapts behavior across execution targets.
- **CN:** 这里可以看到类型或设备分支，说明实现会针对不同执行目标调整行为。

### Lines 273-283 / 第 273-283 行

```python
    VALID_VALUES = {"8.0", "8.9", "9.0"}
    return all(v in VALID_VALUES for v in value.split())


def check_versions():
    """
    check installed packages version
    """
    logger.info("Double check installed packages")
    patterns = ["torch", "torchvision", "torchaudio", "vllm"]
    for pkg in patterns:
```

- **EN:** Important local symbols in this block include check_versions.
- **CN:** 该代码块中的重要局部符号包括 check_versions。
- **EN:** Return statements finalize helper results or forward computed values to the caller.
- **CN:** 返回语句会收束辅助结果，或把已计算的值传递给调用方。

### Lines 284-285 / 第 284-285 行

```python
        pkg_exists(pkg)
    logger.info("Done. checked installed packages")
```

- **EN:** This section continues the surrounding implementation with local declarations, helper logic, or small control-flow steps.
- **CN:** 这一段延续了周边实现，包含局部声明、辅助逻辑或一些控制流步骤。

## Key Concepts / 关键概念

- **Repository support logic** — 仓库支撑逻辑
- **PyTorch tensor operations** — 覆盖或配置具体的 PyTorch 张量/算子行为。
- **Representative symbols: VllmTestParameters, TestInpuType, VllmTestRunner, __post_init__, __init__, prepare, run, cp_torch_cleaning_script** — 代表性符号：VllmTestParameters、TestInpuType、VllmTestRunner、__post_init__、__init__、prepare、run、cp_torch_cleaning_script

## Dependencies / 依赖关系

- `logging`
- `os`
- `re`
- `subprocess`
- `sys`
- `collections.abc`
- `dataclasses`
- `enum`
- `pathlib`
- `typing`
- `cli.lib.common.cli_helper`
- `cli.lib.common.envs_helper`
- `cli.lib.common.path_helper`
- `cli.lib.common.pip_helper`
- `cli.lib.common.utils`
- `cli.lib.core.vllm.lib`
