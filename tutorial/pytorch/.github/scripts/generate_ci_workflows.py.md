# generate_ci_workflows.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/generate_ci_workflows.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。

## Line-by-Line Analysis / 逐行分析
### Lines 1-17 / 第 1-17 行

````python
#!/usr/bin/env python3

import os
import sys
from collections.abc import Iterable
from dataclasses import asdict, dataclass, field
from pathlib import Path
from typing import Literal
from typing_extensions import TypedDict  # Python 3.11+

import generate_binary_build_matrix  # type: ignore[import]
import jinja2


Arch = Literal["windows", "linux", "macos"]

GITHUB_DIR = Path(__file__).resolve().parent.parent
````

- EN: This block imports dependencies such as `os`, `sys`, `collections.abc`, `dataclasses`; touches GitHub-related data or remote service interactions.
- CN: 该代码块导入当前模块运行所需的依赖；处理 GitHub 相关数据或远程服务交互。

### Lines 19-35 / 第 19-35 行

````python
LABEL_CIFLOW_TRUNK = "ciflow/trunk"
LABEL_CIFLOW_UNSTABLE = "ciflow/unstable"
LABEL_CIFLOW_BINARIES = "ciflow/binaries"
LABEL_CIFLOW_PERIODIC = "ciflow/periodic"
LABEL_CIFLOW_BINARIES_LIBTORCH = "ciflow/binaries_libtorch"
LABEL_CIFLOW_BINARIES_WHEEL = "ciflow/binaries_wheel"
LABEL_CIFLOW_S390 = "ciflow/s390"


@dataclass
class CIFlowConfig:
    # For use to enable workflows to run on pytorch/pytorch-canary
    run_on_canary: bool = False
    labels: set[str] = field(default_factory=set)
    # Certain jobs might not want to be part of the ciflow/[all,trunk] workflow
    isolated_workflow: bool = False
    unstable: bool = False
````

- EN: This block introduces classes like `CIFlowConfig`; iterates through collections, records, or generated items.
- CN: 该代码块定义新的类来封装状态与行为；遍历集合、记录或生成项。

### Lines 37-50 / 第 37-50 行

````python
    def __post_init__(self) -> None:
        if not self.isolated_workflow:
            if LABEL_CIFLOW_PERIODIC not in self.labels:
                self.labels.add(
                    LABEL_CIFLOW_TRUNK if not self.unstable else LABEL_CIFLOW_UNSTABLE
                )


class Config(TypedDict):
    num_shards: int
    runner: str


@dataclass
````

- EN: This block introduces classes like `Config`; defines callable units such as `__post_init__`; branches on runtime conditions or configuration choices.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支。

### Lines 51-67 / 第 51-67 行

````python
class BinaryBuildWorkflow:
    os: str
    build_configs: list[dict[str, str]]
    package_type: str

    # Optional fields
    build_environment: str = ""
    ciflow_config: CIFlowConfig = field(default_factory=CIFlowConfig)
    is_scheduled: str = ""
    branches: str = "nightly"
    # Mainly for macos
    macos_runner: str = "macos-14-xlarge"
    # Mainly used for libtorch builds
    build_variant: str = ""
    # Libtorch extraction configs: lightweight jobs that extract libtorch
    # from a wheel build instead of building libtorch from scratch
    libtorch_extraction_configs: list[dict[str, str]] = field(default_factory=list)
````

- EN: This block introduces classes like `BinaryBuildWorkflow`; iterates through collections, records, or generated items.
- CN: 该代码块定义新的类来封装状态与行为；遍历集合、记录或生成项。

### Lines 69-86 / 第 69-86 行

````python
    def __post_init__(self) -> None:
        if self.build_environment == "":
            self.build_environment = "-".join(
                item
                for item in [self.os, "binary", self.package_type, self.build_variant]
                if item != ""
            )

    def generate_workflow_file(self, workflow_template: jinja2.Template) -> None:
        output_file_path = (
            GITHUB_DIR
            / f"workflows/generated-{self.build_environment}-{self.branches}.yml"
        )
        with open(output_file_path, "w") as output_file:
            GENERATED = "generated"  # Note that please keep the variable GENERATED otherwise phabricator will hide the whole file
            output_file.writelines([f"# @{GENERATED} DO NOT EDIT MANUALLY\n"])
            try:
                content = workflow_template.render(asdict(self))
````

- EN: This block defines callable units such as `__post_init__`, `generate_workflow_file`; branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互。

### Lines 87-103 / 第 87-103 行

````python
            except Exception as e:
                print(f"Failed on template: {workflow_template}", file=sys.stderr)
                raise e
            output_file.write(content)
            if content[-1] != "\n":
                output_file.write("\n")
        print(output_file_path)


class OperatingSystem:
    LINUX = "linux"
    WINDOWS = "windows"
    WINDOWS_ARM64 = "windows-arm64"
    MACOS = "macos"
    MACOS_ARM64 = "macos-arm64"
    LINUX_AARCH64 = "linux-aarch64"
    LINUX_S390X = "linux-s390x"
````

- EN: This block introduces classes like `OperatingSystem`; branches on runtime conditions or configuration choices; validates assumptions and surfaces failures explicitly.
- CN: 该代码块定义新的类来封装状态与行为；依据运行时条件或配置选择进行分支；校验前提条件并显式暴露失败。

### Lines 106-123 / 第 106-123 行

````python
_LINUX_WHEEL_CONFIGS = generate_binary_build_matrix.generate_wheels_matrix(
    OperatingSystem.LINUX
)

LINUX_BINARY_BUILD_WORFKLOWS = [
    BinaryBuildWorkflow(
        os=OperatingSystem.LINUX,
        package_type="manywheel",
        build_configs=_LINUX_WHEEL_CONFIGS,
        ciflow_config=CIFlowConfig(
            labels={
                LABEL_CIFLOW_BINARIES,
                LABEL_CIFLOW_BINARIES_WHEEL,
                LABEL_CIFLOW_BINARIES_LIBTORCH,
            },
            isolated_workflow=True,
        ),
        libtorch_extraction_configs=generate_binary_build_matrix.generate_libtorch_extraction_configs(
````

- EN: This block implements local helper logic for generate ci workflows.
- CN: 该代码块实现与 generate ci workflows 相关的局部辅助逻辑。

### Lines 124-137 / 第 124-137 行

````python
            OperatingSystem.LINUX,
            _LINUX_WHEEL_CONFIGS,
        ),
    ),
]

_WINDOWS_WHEEL_CONFIGS = generate_binary_build_matrix.generate_wheels_matrix(
    OperatingSystem.WINDOWS
)
_WINDOWS_ARM64_WHEEL_CONFIGS = generate_binary_build_matrix.generate_wheels_matrix(
    OperatingSystem.WINDOWS_ARM64,
    arches=["cpu"],
    python_versions=["3.11", "3.12", "3.13"],
)
````

- EN: This block implements local helper logic for generate ci workflows.
- CN: 该代码块实现与 generate ci workflows 相关的局部辅助逻辑。

### Lines 139-156 / 第 139-156 行

````python
WINDOWS_BINARY_BUILD_WORKFLOWS = [
    BinaryBuildWorkflow(
        os=OperatingSystem.WINDOWS,
        package_type="wheel",
        build_configs=_WINDOWS_WHEEL_CONFIGS,
        ciflow_config=CIFlowConfig(
            labels={
                LABEL_CIFLOW_BINARIES,
                LABEL_CIFLOW_BINARIES_WHEEL,
                LABEL_CIFLOW_BINARIES_LIBTORCH,
            },
            isolated_workflow=True,
        ),
        libtorch_extraction_configs=generate_binary_build_matrix.generate_libtorch_extraction_configs(
            OperatingSystem.WINDOWS,
            _WINDOWS_WHEEL_CONFIGS,
        ),
    ),
````

- EN: This block implements local helper logic for generate ci workflows.
- CN: 该代码块实现与 generate ci workflows 相关的局部辅助逻辑。

### Lines 157-174 / 第 157-174 行

````python
    BinaryBuildWorkflow(
        os=OperatingSystem.WINDOWS,
        package_type="libtorch",
        build_variant=generate_binary_build_matrix.DEBUG,
        build_configs=generate_binary_build_matrix.generate_libtorch_matrix(
            OperatingSystem.WINDOWS,
            generate_binary_build_matrix.DEBUG,
            libtorch_variants=["shared-with-deps"],
        ),
        ciflow_config=CIFlowConfig(
            labels={LABEL_CIFLOW_BINARIES, LABEL_CIFLOW_BINARIES_LIBTORCH},
            isolated_workflow=True,
        ),
    ),
    BinaryBuildWorkflow(
        os=OperatingSystem.WINDOWS_ARM64,
        package_type="wheel",
        build_configs=_WINDOWS_ARM64_WHEEL_CONFIGS,
````

- EN: This block implements local helper logic for generate ci workflows.
- CN: 该代码块实现与 generate ci workflows 相关的局部辅助逻辑。

### Lines 175-192 / 第 175-192 行

````python
        ciflow_config=CIFlowConfig(
            labels={
                LABEL_CIFLOW_BINARIES,
                LABEL_CIFLOW_BINARIES_WHEEL,
                LABEL_CIFLOW_BINARIES_LIBTORCH,
            },
            isolated_workflow=True,
        ),
        libtorch_extraction_configs=generate_binary_build_matrix.generate_libtorch_extraction_configs(
            OperatingSystem.WINDOWS_ARM64,
            _WINDOWS_ARM64_WHEEL_CONFIGS,
        ),
    ),
    BinaryBuildWorkflow(
        os=OperatingSystem.WINDOWS_ARM64,
        package_type="libtorch",
        build_variant=generate_binary_build_matrix.DEBUG,
        build_configs=generate_binary_build_matrix.generate_libtorch_matrix(
````

- EN: This block implements local helper logic for generate ci workflows.
- CN: 该代码块实现与 generate ci workflows 相关的局部辅助逻辑。

### Lines 193-207 / 第 193-207 行

````python
            OperatingSystem.WINDOWS_ARM64,
            generate_binary_build_matrix.DEBUG,
            arches=["cpu"],
            libtorch_variants=["shared-with-deps"],
        ),
        ciflow_config=CIFlowConfig(
            labels={LABEL_CIFLOW_BINARIES, LABEL_CIFLOW_BINARIES_LIBTORCH},
            isolated_workflow=True,
        ),
    ),
]

_MACOS_ARM64_WHEEL_CONFIGS = generate_binary_build_matrix.generate_wheels_matrix(
    OperatingSystem.MACOS_ARM64
)
````

- EN: This block implements local helper logic for generate ci workflows.
- CN: 该代码块实现与 generate ci workflows 相关的局部辅助逻辑。

### Lines 209-226 / 第 209-226 行

````python
MACOS_BINARY_BUILD_WORKFLOWS = [
    BinaryBuildWorkflow(
        os=OperatingSystem.MACOS_ARM64,
        package_type="wheel",
        build_configs=_MACOS_ARM64_WHEEL_CONFIGS,
        macos_runner="macos-26-xlarge",
        ciflow_config=CIFlowConfig(
            labels={
                LABEL_CIFLOW_BINARIES,
                LABEL_CIFLOW_BINARIES_WHEEL,
                LABEL_CIFLOW_BINARIES_LIBTORCH,
            },
            isolated_workflow=True,
        ),
        libtorch_extraction_configs=generate_binary_build_matrix.generate_libtorch_extraction_configs(
            OperatingSystem.MACOS_ARM64,
            _MACOS_ARM64_WHEEL_CONFIGS,
        ),
````

- EN: This block implements local helper logic for generate ci workflows.
- CN: 该代码块实现与 generate ci workflows 相关的局部辅助逻辑。

### Lines 227-242 / 第 227-242 行

````python
    ),
]

AARCH64_BINARY_BUILD_WORKFLOWS = [
    BinaryBuildWorkflow(
        os=OperatingSystem.LINUX_AARCH64,
        package_type="manywheel",
        build_configs=generate_binary_build_matrix.generate_wheels_matrix(
            OperatingSystem.LINUX_AARCH64
        ),
        ciflow_config=CIFlowConfig(
            labels={LABEL_CIFLOW_BINARIES, LABEL_CIFLOW_BINARIES_WHEEL},
            isolated_workflow=True,
        ),
    ),
]
````

- EN: This block implements local helper logic for generate ci workflows.
- CN: 该代码块实现与 generate ci workflows 相关的局部辅助逻辑。

### Lines 244-260 / 第 244-260 行

````python
S390X_BINARY_BUILD_WORKFLOWS = [
    BinaryBuildWorkflow(
        os=OperatingSystem.LINUX_S390X,
        package_type="manywheel",
        build_configs=generate_binary_build_matrix.generate_wheels_matrix(
            OperatingSystem.LINUX_S390X
        ),
        ciflow_config=CIFlowConfig(
            labels={
                LABEL_CIFLOW_BINARIES,
                LABEL_CIFLOW_BINARIES_WHEEL,
                LABEL_CIFLOW_S390,
            },
            isolated_workflow=True,
        ),
    ),
]
````

- EN: This block implements local helper logic for generate ci workflows.
- CN: 该代码块实现与 generate ci workflows 相关的局部辅助逻辑。

### Lines 263-280 / 第 263-280 行

````python
def main() -> None:
    jinja_env = jinja2.Environment(
        variable_start_string="!{{",
        loader=jinja2.FileSystemLoader(str(GITHUB_DIR.joinpath("templates"))),
        undefined=jinja2.StrictUndefined,
    )

    # not ported yet
    template_and_workflows = [
        (
            jinja_env.get_template("linux_binary_build_workflow.yml.j2"),
            LINUX_BINARY_BUILD_WORFKLOWS,
        ),
        (
            jinja_env.get_template("linux_binary_build_workflow.yml.j2"),
            AARCH64_BINARY_BUILD_WORKFLOWS,
        ),
        (
````

- EN: This block defines callable units such as `main`; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互。

### Lines 281-298 / 第 281-298 行

````python
            jinja_env.get_template("linux_binary_build_workflow.yml.j2"),
            S390X_BINARY_BUILD_WORKFLOWS,
        ),
        (
            jinja_env.get_template("windows_binary_build_workflow.yml.j2"),
            WINDOWS_BINARY_BUILD_WORKFLOWS,
        ),
        (
            jinja_env.get_template("macos_binary_build_workflow.yml.j2"),
            MACOS_BINARY_BUILD_WORKFLOWS,
        ),
    ]
    # Delete the existing generated files first, this should align with .gitattributes file description.
    existing_workflows = GITHUB_DIR.glob("workflows/generated-*")
    for w in existing_workflows:
        try:
            os.remove(w)
        except Exception as e:
````

- EN: This block iterates through collections, records, or generated items; touches GitHub-related data or remote service interactions.
- CN: 该代码块遍历集合、记录或生成项；处理 GitHub 相关数据或远程服务交互。

### Lines 299-308 / 第 299-308 行

````python
            print(f"Error occurred when deleting file {w}: {e}")

    for template, workflows in template_and_workflows:
        # added Iterable check to appease the mypy gods
        if not isinstance(workflows, Iterable):
            raise Exception(  # noqa: TRY002
                f"How is workflows not iterable? {workflows}"
            )
        for workflow in workflows:
            workflow.generate_workflow_file(workflow_template=template)
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; validates assumptions and surfaces failures explicitly.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；校验前提条件并显式暴露失败。

### Lines 311-312 / 第 311-312 行

````python
if __name__ == "__main__":
    main()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Coordinates GitHub Actions automation, including workflow triggers, job graphs, and reusable actions.
  CN: 协调 GitHub Actions 自动化，包括工作流触发、作业图以及可复用 Action。
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Extends static analysis or typing precision so developer tooling understands PyTorch-specific patterns better.
  CN: 扩展静态分析或类型精度，使开发工具更好理解 PyTorch 特有模式。
- EN: Captures repository governance and contributor-facing policies in declarative configuration.
  CN: 以声明式配置的形式记录仓库治理规则和面向贡献者的策略。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `os`, `sys`, `collections.abc`, `dataclasses`, `pathlib`, `typing`
- Internal imports / 内部导入: none
- External imports / 外部导入: `typing_extensions`, `generate_binary_build_matrix`, `jinja2`
- Classes / 类: `CIFlowConfig`, `Config`, `BinaryBuildWorkflow`, `OperatingSystem`
- Functions / 函数: `main`
- Test entry points / 测试入口: none
