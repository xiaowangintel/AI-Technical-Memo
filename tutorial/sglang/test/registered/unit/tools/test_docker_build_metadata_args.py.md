# test_docker_build_metadata_args.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/tools/test_docker_build_metadata_args.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates docker build metadata args behavior in SGLang's unit / tools area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / tools 领域中与 docker build metadata args 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: module imports and dependencies / 模块导入与依赖
```python
import importlib.util
import json
import subprocess
import unittest
from pathlib import Path
```
**EN:** This block imports the modules needed by the rest of the file, including `importlib.util`, `json`, `subprocess`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `importlib.util`, `json`, `subprocess`, `unittest`。

### Lines 7-11: module-level constants and configuration / 模块级常量与配置
```python
REPO_ROOT = Path(__file__).resolve().parents[4]
CI_REGISTER_PATH = REPO_ROOT / "python" / "sglang" / "test" / "ci" / "ci_register.py"
HELPER_PATH = REPO_ROOT / "scripts" / "ci" / "utils" / "docker_build_metadata_args.py"
DOCKERFILE_PATH = REPO_ROOT / "docker" / "Dockerfile"
WORKFLOW_PATH = REPO_ROOT / ".github" / "workflows" / "_docker-build-and-publish.yml"
```
**EN:** This block defines shared names such as `REPO_ROOT`, `CI_REGISTER_PATH`, `HELPER_PATH`, `DOCKERFILE_PATH`, `WORKFLOW_PATH`. These values centralize configuration that later tests or helpers reuse.
**CN:** 该代码块定义了 `REPO_ROOT`, `CI_REGISTER_PATH`, `HELPER_PATH`, `DOCKERFILE_PATH`, `WORKFLOW_PATH` 等共享名称，用于集中保存后续测试或辅助逻辑会复用的配置。

### Lines 14-18: function load module / 函数 load module
```python
def _load_module(name, path):
    spec = importlib.util.spec_from_file_location(name, path)
    module = importlib.util.module_from_spec(spec)
    spec.loader.exec_module(module)
    return module
```
**EN:** This block implements `_load_module` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_load_module`，承担模块行为中的一个聚焦逻辑片段。

### Lines 21-22: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci = _load_module("ci_register", CI_REGISTER_PATH).register_cpu_ci
register_cpu_ci(est_time=0, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through _load_module, register_cpu_ci.
**CN:** 该代码块通过 _load_module, register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 25-25: class TestDockerBuildMetadataArgs declaration / 类 TestDockerBuildMetadataArgs 声明
```python
class TestDockerBuildMetadataArgs(unittest.TestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `unittest.TestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `unittest.TestCase`。

### Lines 26-28: setUpClass setup routine / setUpClass 初始化流程
```python
    @classmethod
    def setUpClass(cls):
        cls.helper = _load_module("docker_build_metadata_args", HELPER_PATH)
```
**EN:** This routine prepares shared fixtures, models, or runtime state before the assertions execute.
**CN:** 该流程会在断言执行前准备共享夹具、模型或运行状态。

### Lines 30-64: helper routine run helper / 辅助流程 run helper
```python
    def run_helper(
        self,
        *,
        cuda: str,
        tag_config: list[dict[str, object]],
        image_repo: str = "lmsysorg/sglang",
        version: str = "0.6.0",
        build_commit: str = "abcdef1234567890",
        build_url: str = "https://github.com/sgl-project/sglang/actions/runs/1",
        date: str = "20260429",
    ) -> list[str]:
        result = subprocess.run(
            [
                "python3",
                str(HELPER_PATH),
                "--cuda",
                cuda,
                "--tag-config",
                json.dumps(tag_config),
                "--image-repo",
                image_repo,
                "--sgl-version",
                version,
                "--build-commit",
                build_commit,
                "--build-url",
                build_url,
                "--date",
                date,
            ],
            check=True,
            stdout=subprocess.PIPE,
            text=True,
        )
        return result.stdout.splitlines()
```
**EN:** This helper encapsulates `run_helper` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `run_helper`，以便周围测试复用准备、执行或校验逻辑。

### Lines 66-68: method option values / 方法 option values
```python
    @staticmethod
    def option_values(args: list[str], option: str) -> list[str]:
        return [args[i + 1] for i, arg in enumerate(args[:-1]) if arg == option]
```
**EN:** This block implements `option_values` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `option_values`，承担模块行为中的一个聚焦逻辑片段。

### Lines 70-75: helper routine build args / 辅助流程 build args
```python
    def build_args(self, args: list[str]) -> dict[str, str]:
        values = {}
        for value in self.option_values(args, "--build-arg"):
            key, arg_value = value.split("=", 1)
            values[key] = arg_value
        return values
```
**EN:** This helper encapsulates `build_args` so the surrounding tests can reuse setup, execution, or validation logic.
**CN:** 该辅助函数封装了 `build_args`，以便周围测试复用准备、执行或校验逻辑。

### Lines 77-95: test case release metadata prefers versioned tag / 测试用例 release metadata prefers versioned tag
```python
    def test_release_metadata_prefers_versioned_tag(self):
        args = self.run_helper(
            cuda="cu129",
            tag_config=[
                {"cuda": "cu129", "tags": ["v{version}", "latest"]},
                {"cuda": "cu130", "tags": ["v{version}-cu130", "latest-cu130"]},
            ],
        )

        self.assertEqual(
            self.build_args(args),
            {
                "SGLANG_BUILD_COMMIT": "abcdef1234567890",
                "SGLANG_BUILD_URL": (
                    "https://github.com/sgl-project/sglang/actions/runs/1"
                ),
                "SGLANG_IMAGE_TAG": "lmsysorg/sglang:v0.6.0",
            },
        )
```
**EN:** This test exercises `test_release_metadata_prefers_versioned_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_release_metadata_prefers_versioned_tag`。

### Lines 97-113: test case runtime metadata uses custom repo and runtime tag / 测试用例 runtime metadata uses custom repo and runtime tag
```python
    def test_runtime_metadata_uses_custom_repo_and_runtime_tag(self):
        args = self.run_helper(
            cuda="cu130",
            image_repo="lmsysorg/sglang-staging",
            tag_config=[
                {"cuda": "cu129", "tags": ["v{version}-runtime", "latest-runtime"]},
                {
                    "cuda": "cu130",
                    "tags": ["v{version}-cu130-runtime", "latest-cu130-runtime"],
                },
            ],
        )

        self.assertEqual(
            self.build_args(args)["SGLANG_IMAGE_TAG"],
            "lmsysorg/sglang-staging:v0.6.0-cu130-runtime",
        )
```
**EN:** This test exercises `test_runtime_metadata_uses_custom_repo_and_runtime_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_runtime_metadata_uses_custom_repo_and_runtime_tag`。

### Lines 115-136: test case dev nightly metadata prefers unique tag from checked out commit / 测试用例 dev nightly metadata prefers unique tag from checked out commit
```python
    def test_dev_nightly_metadata_prefers_unique_tag_from_checked_out_commit(self):
        args = self.run_helper(
            cuda="cu129",
            version="",
            build_commit="1234567890abcdef",
            tag_config=[
                {"cuda": "cu129", "tags": ["dev", "nightly-dev-{date}-{short_sha}"]},
                {
                    "cuda": "cu130",
                    "tags": ["dev-cu13", "nightly-dev-cu13-{date}-{short_sha}"],
                },
            ],
        )

        self.assertEqual(
            self.build_args(args)["SGLANG_IMAGE_TAG"],
            "lmsysorg/sglang:nightly-dev-20260429-12345678",
        )
        self.assertEqual(
            self.build_args(args)["SGLANG_BUILD_COMMIT"],
            "1234567890abcdef",
        )
```
**EN:** This test exercises `test_dev_nightly_metadata_prefers_unique_tag_from_checked_out_commit` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dev_nightly_metadata_prefers_unique_tag_from_checked_out_commit`。

### Lines 138-151: test case custom dev tag is treated as specific / 测试用例 custom dev tag is treated as specific
```python
    def test_custom_dev_tag_is_treated_as_specific(self):
        args = self.run_helper(
            cuda="cu130",
            version="",
            tag_config=[
                {"cuda": "cu129", "tags": ["dev-my-test"]},
                {"cuda": "cu130", "tags": ["dev-cu13-my-test"]},
            ],
        )

        self.assertEqual(
            self.build_args(args)["SGLANG_IMAGE_TAG"],
            "lmsysorg/sglang:dev-cu13-my-test",
        )
```
**EN:** This test exercises `test_custom_dev_tag_is_treated_as_specific` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_dev_tag_is_treated_as_specific`。

### Lines 153-161: test case missing cuda entry fails / 测试用例 missing cuda entry fails
```python
    def test_missing_cuda_entry_fails(self):
        with self.assertRaisesRegex(ValueError, "cu130"):
            self.helper.select_tag(
                json.dumps([{"cuda": "cu129", "tags": ["v{version}"]}]),
                "cu130",
                "0.6.0",
                "20260429",
                "abcdef12",
            )
```
**EN:** This test exercises `test_missing_cuda_entry_fails` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_missing_cuda_entry_fails`。

### Lines 163-190: test case final dockerfile stages embed metadata contract / 测试用例 final dockerfile stages embed metadata contract
```python
    def test_final_dockerfile_stages_embed_metadata_contract(self):
        dockerfile = DOCKERFILE_PATH.read_text()
        framework_stage = dockerfile.split("FROM framework AS framework_final", 1)[
            1
        ].split("FROM nvidia/cuda:${CUDA_VERSION}-cudnn-devel-ubuntu24.04 AS runtime")[
            0
        ]
        runtime_stage = dockerfile.split(
            "FROM nvidia/cuda:${CUDA_VERSION}-cudnn-devel-ubuntu24.04 AS runtime", 1
        )[1]

        for stage in (framework_stage, runtime_stage):
            for expected in (
                "ARG SGLANG_BUILD_COMMIT=unknown",
                "ARG SGLANG_BUILD_URL=",
                "ARG SGLANG_IMAGE_TAG=local/sglang:dev",
                "SGLANG_BUILD_COMMIT=${SGLANG_BUILD_COMMIT:-unknown}",
                "SGLANG_BUILD_URL=${SGLANG_BUILD_URL:-}",
                "SGLANG_IMAGE_TAG=${SGLANG_IMAGE_TAG:-local/sglang:dev}",
                'org.opencontainers.image.source="https://github.com/sgl-project/sglang"',
                'org.opencontainers.image.revision="${SGLANG_BUILD_COMMIT}"',
                'org.opencontainers.image.version="${SGLANG_IMAGE_TAG}"',
                'org.opencontainers.image.url="${SGLANG_BUILD_URL}"',
                'ai.sglang.build.commit="${SGLANG_BUILD_COMMIT}"',
                'ai.sglang.build.url="${SGLANG_BUILD_URL}"',
                'ai.sglang.image.tag="${SGLANG_IMAGE_TAG}"',
            ):
                self.assertIn(expected, stage)
```
**EN:** This test exercises `test_final_dockerfile_stages_embed_metadata_contract` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_final_dockerfile_stages_embed_metadata_contract`。

### Lines 192-198: test case shared docker workflow uses checked out commit / 测试用例 shared docker workflow uses checked out commit
```python
    def test_shared_docker_workflow_uses_checked_out_commit(self):
        workflow = WORKFLOW_PATH.read_text()

        self.assertIn("git rev-parse HEAD", workflow)
        self.assertIn("scripts/ci/utils/docker_build_metadata_args.py", workflow)
        self.assertIn("mapfile -t METADATA_ARGS", workflow)
        self.assertIn('"${METADATA_ARGS[@]}"', workflow)
```
**EN:** This test exercises `test_shared_docker_workflow_uses_checked_out_commit` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_shared_docker_workflow_uses_checked_out_commit`。

### Lines 201-202: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `_load_module`: This block implements `_load_module` and captures one focused piece of the module's behavior. / 该代码块实现 `_load_module`，承担模块行为中的一个聚焦逻辑片段。
- `TestDockerBuildMetadataArgs`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestDockerBuildMetadataArgs.setUpClass`: This routine prepares shared fixtures, models, or runtime state before the assertions execute. / 该流程会在断言执行前准备共享夹具、模型或运行状态。
- `TestDockerBuildMetadataArgs.run_helper`: This helper encapsulates `run_helper` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `run_helper`，以便周围测试复用准备、执行或校验逻辑。
- `TestDockerBuildMetadataArgs.option_values`: This block implements `option_values` and captures one focused piece of the module's behavior. / 该代码块实现 `option_values`，承担模块行为中的一个聚焦逻辑片段。
- `TestDockerBuildMetadataArgs.build_args`: This helper encapsulates `build_args` so the surrounding tests can reuse setup, execution, or validation logic. / 该辅助函数封装了 `build_args`，以便周围测试复用准备、执行或校验逻辑。
- `TestDockerBuildMetadataArgs.test_release_metadata_prefers_versioned_tag`: This test exercises `test_release_metadata_prefers_versioned_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_release_metadata_prefers_versioned_tag`。
- `TestDockerBuildMetadataArgs.test_runtime_metadata_uses_custom_repo_and_runtime_tag`: This test exercises `test_runtime_metadata_uses_custom_repo_and_runtime_tag` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_runtime_metadata_uses_custom_repo_and_runtime_tag`。
- `TestDockerBuildMetadataArgs.test_dev_nightly_metadata_prefers_unique_tag_from_checked_out_commit`: This test exercises `test_dev_nightly_metadata_prefers_unique_tag_from_checked_out_commit` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dev_nightly_metadata_prefers_unique_tag_from_checked_out_commit`。
- `TestDockerBuildMetadataArgs.test_custom_dev_tag_is_treated_as_specific`: This test exercises `test_custom_dev_tag_is_treated_as_specific` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_custom_dev_tag_is_treated_as_specific`。
- `TestDockerBuildMetadataArgs.test_missing_cuda_entry_fails`: This test exercises `test_missing_cuda_entry_fails` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_missing_cuda_entry_fails`。
- `TestDockerBuildMetadataArgs.test_final_dockerfile_stages_embed_metadata_contract`: This test exercises `test_final_dockerfile_stages_embed_metadata_contract` by arranging inputs, invoking the relevant path, and checking the expected outcome. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_final_dockerfile_stages_embed_metadata_contract`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `importlib.util`, `json`, `subprocess`, `unittest`, `pathlib`

- **Total lines / 总行数**: 202
