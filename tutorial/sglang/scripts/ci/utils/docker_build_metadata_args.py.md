# docker_build_metadata_args.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `scripts/ci/utils/docker_build_metadata_args.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This python module supports the `docker_build_metadata_args` workflow in SGLang. It mainly handles CI orchestration, test execution. / 该Python 模块用于支撑 SGLang 中的 `docker_build_metadata_args` 流程，主要负责CI 编排、测试执行。它属于 `utils` 自动化路径的一部分。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Imports dependencies and shared utilities / 导入依赖与共享工具
```python
import argparse
import datetime
import json
import sys
```
**EN:** This block loads argparse, datetime, json, sys. Keeping imports together makes the rest of the script easier to read and clarifies its runtime dependencies.
**CN:** 该代码块导入了 argparse, datetime, json, sys。把导入集中放在一起，有助于读者快速识别运行时依赖。

### Lines 6-6: Declares module-level constants and defaults / 声明模块级常量与默认值
```python
MOVING_TAGS = {"dev", "dev-cu12", "dev-cu13", "latest"}
```
**EN:** This section defines MOVING_TAGS, which centralize reusable defaults for the rest of the module.
**CN:** 该部分定义了 MOVING_TAGS，用于把可复用的默认值集中在模块顶部。

### Lines 9-14: Defines the `render_tag_template` routine / 定义 `render_tag_template` 例程
```python
def render_tag_template(tag: str, version: str, date: str, short_sha: str) -> str:
    return (
        tag.replace("{version}", version)
        .replace("{date}", date)
        .replace("{short_sha}", short_sha)
    )
```
**EN:** This block defines `render_tag_template`. It accepts 4 parameter(s): tag, version, date, short_sha. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `render_tag_template`。它接收 4 个参数：tag, version, date, short_sha。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 17-18: Defines the `is_moving_tag` routine / 定义 `is_moving_tag` 例程
```python
def is_moving_tag(tag: str) -> bool:
    return tag in MOVING_TAGS or tag.startswith("latest-")
```
**EN:** This block defines `is_moving_tag`. It accepts 1 parameter(s): tag. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `is_moving_tag`。它接收 1 个参数：tag。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 21-41: Defines the `select_tag` routine / 定义 `select_tag` 例程
```python
def select_tag(
    tag_config: str, cuda: str, version: str, date: str, short_sha: str
) -> str:
    entries = json.loads(tag_config)
    for entry in entries:
        if entry.get("cuda") != cuda:
            continue

        tags = [
            render_tag_template(tag, version, date, short_sha)
            for tag in entry.get("tags", [])
        ]
        if not tags:
            raise ValueError(f"No tags configured for CUDA variant {cuda}")

        for tag in tags:
            if not is_moving_tag(tag):
                return tag
        return tags[0]

    raise ValueError(f"CUDA variant {cuda} not found in tag_config")
```
**EN:** This block defines `select_tag`. It accepts 5 parameter(s): tag_config, cuda, version, date, short_sha. Internally it loads structured data.
**CN:** 该代码块定义了 `select_tag`。它接收 5 个参数：tag_config, cuda, version, date, short_sha。内部会加载结构化数据。

### Lines 44-65: Defines the `build_arg_tokens` routine / 定义 `build_arg_tokens` 例程
```python
def build_arg_tokens(
    *,
    cuda: str,
    tag_config: str,
    image_repo: str,
    version: str,
    build_commit: str,
    build_url: str,
    date: str,
    short_sha: str,
) -> list[str]:
    image_tag = select_tag(tag_config, cuda, version, date, short_sha)
    build_args = {
        "SGLANG_BUILD_COMMIT": build_commit,
        "SGLANG_BUILD_URL": build_url,
        "SGLANG_IMAGE_TAG": f"{image_repo}:{image_tag}",
    }

    tokens = []
    for key, value in build_args.items():
        tokens.extend(["--build-arg", f"{key}={value}"])
    return tokens
```
**EN:** This block defines `build_arg_tokens`. It accepts 8 parameter(s): cuda, tag_config, image_repo, version, build_commit, build_url. It packages one focused unit of automation for the surrounding workflow.
**CN:** 该代码块定义了 `build_arg_tokens`。它接收 8 个参数：cuda, tag_config, image_repo, version, build_commit, build_url。它把周边流程中的一个独立自动化步骤封装成可复用逻辑。

### Lines 68-92: Defines the `parse_args` routine / 定义 `parse_args` 例程
```python
def parse_args() -> argparse.Namespace:
    parser = argparse.ArgumentParser(
        description="Emit docker build arguments for SGLang image metadata."
    )
    parser.add_argument("--cuda", required=True, help="CUDA variant from tag_config.")
    parser.add_argument("--tag-config", required=True, help="Docker tag JSON config.")
    parser.add_argument("--image-repo", required=True, help="Docker image repository.")
    parser.add_argument("--sgl-version", default="", help="SGLang release version.")
    parser.add_argument(
        "--build-commit",
        required=True,
        help="Commit checked out for the Docker build.",
    )
    parser.add_argument("--build-url", default="", help="CI run URL.")
    parser.add_argument(
        "--date",
        default=datetime.datetime.now(datetime.timezone.utc).strftime("%Y%m%d"),
        help="Date used for {date} tag templates.",
    )
    parser.add_argument(
        "--short-sha",
        default="",
        help="Short SHA used for {short_sha}; defaults to build commit prefix.",
    )
    return parser.parse_args()
```
**EN:** This block defines `parse_args`. It takes no explicit parameters at the top level. Internally it parses CLI arguments.
**CN:** 该代码块定义了 `parse_args`。它在顶层定义中不接收显式参数。内部会解析命令行参数。

### Lines 95-115: Defines the `main` routine / 定义 `main` 例程
```python
def main() -> int:
    args = parse_args()
    short_sha = args.short_sha or args.build_commit[:8]

    try:
        tokens = build_arg_tokens(
            cuda=args.cuda,
            tag_config=args.tag_config,
            image_repo=args.image_repo,
            version=args.sgl_version,
            build_commit=args.build_commit,
            build_url=args.build_url,
            date=args.date,
            short_sha=short_sha,
        )
    except (json.JSONDecodeError, ValueError) as exc:
        print(f"error: {exc}", file=sys.stderr)
        return 1

    print("\n".join(tokens))
    return 0
```
**EN:** This block defines `main`. It takes no explicit parameters at the top level. Internally it parses CLI arguments, emits status messages.
**CN:** 该代码块定义了 `main`。它在顶层定义中不接收显式参数。内部会解析命令行参数、输出状态信息。

### Lines 118-119: Entrypoint wiring the main workflow / 连接主流程的入口逻辑
```python
if __name__ == "__main__":
    sys.exit(main())
```
**EN:** This guard runs only when the file is executed as a script, making it the operational entrypoint for the module. It controls process exit status.
**CN:** 该保护块只会在文件被直接执行时运行，因此它就是模块的实际入口。 它会控制进程退出状态。

## Key Concepts / 关键概念
- **CLI parsing** / 命令行解析
- **Environment management** / 环境管理
- **Process control** / 进程控制
- **Structured data handling** / 结构化数据处理
- **Container execution** / 容器执行

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `datetime`, `json`, `sys`
