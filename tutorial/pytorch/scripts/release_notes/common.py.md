# common.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `scripts/release_notes/common.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements release-note tooling that classifies commits, applies categories, and assembles changelog inputs.
- **用途 (CN)**: 实现发布说明工具，用于分类提交、应用类别并组装变更日志输入。

## Line-by-Line Analysis / 逐行分析
### Lines 1-16 / 第 1-16 行

````python
import json
import locale
import os
import re
import subprocess
from collections import namedtuple
from dataclasses import dataclass
from pathlib import Path

import requests


@dataclass
class CategoryGroup:
    name: str
    categories: list
````

- EN: This block imports dependencies such as `json`, `locale`, `os`, `re`; introduces classes like `CategoryGroup`; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块导入当前模块运行所需的依赖；定义新的类来封装状态与行为；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 19-35 / 第 19-35 行

````python
frontend_categories = [
    "meta",
    "nn",
    "linalg",
    "cpp",
    "python",
    "complex",
    "vmap",
    "autograd",
    "build",
    "memory_format",
    "foreach",
    "dataloader",
    "sparse",
    "nested tensor",
    "optimizer",
]
````

- EN: This block implements local helper logic for common.
- CN: 该代码块实现与 common 相关的局部辅助逻辑。

### Lines 37-50 / 第 37-50 行

````python
pytorch_2_categories = [
    "dynamo",
    "inductor",
]

# These will all get mapped to quantization
quantization = CategoryGroup(
    name="quantization",
    categories=[
        "quantization",
        "AO frontend",
        "AO Pruning",
    ],
)
````

- EN: This block implements local helper logic for common.
- CN: 该代码块实现与 common 相关的局部辅助逻辑。

### Lines 52-64 / 第 52-64 行

````python
# Distributed has a number of release note labels we want to map to one
distributed = CategoryGroup(
    name="distributed",
    categories=[
        "distributed",
        "distributed (c10d)",
        "distributed (composable)",
        "distributed (ddp)",
        "distributed (fsdp)",
        "distributed (rpc)",
        "distributed (sharded)",
    ],
)
````

- EN: This block implements local helper logic for common.
- CN: 该代码块实现与 common 相关的局部辅助逻辑。

### Lines 66-83 / 第 66-83 行

````python
categories = (
    [
        "Uncategorized",
        "lazy",
        "hub",
        "mobile",
        "jit",
        "visualization",
        "onnx",
        "caffe2",
        "amd",
        "rocm",
        "cuda",
        "cpu",
        "cudnn",
        "xla",
        "benchmark",
        "profiler",
````

- EN: This block implements local helper logic for common.
- CN: 该代码块实现与 common 相关的局部辅助逻辑。

### Lines 84-101 / 第 84-101 行

````python
        "performance_as_product",
        "package",
        "dispatcher",
        "releng",
        "fx",
        "code_coverage",
        "vulkan",
        "skip",
        "composability",
        # 2.0 release
        "mps",
        "intel",
        "functorch",
        "gnn",
        "distributions",
        "serialization",
    ]
    + [f"{category}_frontend" for category in frontend_categories]
````

- EN: This block iterates through collections, records, or generated items.
- CN: 该代码块遍历集合、记录或生成项。

### Lines 102-119 / 第 102-119 行

````python
    + pytorch_2_categories
    + [quantization.name]
    + [distributed.name]
)


topics = [
    "bc breaking",
    "deprecation",
    "new features",
    "improvements",
    "bug fixes",
    "performance",
    "docs",
    "devs",
    "Untopiced",
    "not user facing",
    "security",
````

- EN: This block implements local helper logic for common.
- CN: 该代码块实现与 common 相关的局部辅助逻辑。

### Lines 120-137 / 第 120-137 行

````python
]


Features = namedtuple(
    "Features",
    ["title", "body", "pr_number", "files_changed", "labels", "author", "accepters"],
)


def dict_to_features(dct):
    return Features(
        title=dct["title"],
        body=dct["body"],
        pr_number=dct["pr_number"],
        files_changed=dct["files_changed"],
        labels=dct["labels"],
        author=dct["author"],
        accepters=tuple(dct["accepters"]),
````

- EN: This block defines callable units such as `dict_to_features`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 138-155 / 第 138-155 行

````python
    )


def features_to_dict(features):
    return dict(features._asdict())


def run(command):
    """Returns (return-code, stdout, stderr)"""
    p = subprocess.Popen(
        command, stdout=subprocess.PIPE, stderr=subprocess.PIPE, shell=True
    )
    output, err = p.communicate()
    rc = p.returncode
    enc = locale.getpreferredencoding()
    output = output.decode(enc)
    err = err.decode(enc)
    return rc, output.strip(), err.strip()
````

- EN: This block defines callable units such as `features_to_dict`, `run`; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；向调用方返回计算结果。

### Lines 158-173 / 第 158-173 行

````python
def commit_body(commit_hash):
    cmd = f"git log -n 1 --pretty=format:%b {commit_hash}"
    ret, out, err = run(cmd)
    return out if ret == 0 else None


def commit_title(commit_hash):
    cmd = f"git log -n 1 --pretty=format:%s {commit_hash}"
    ret, out, err = run(cmd)
    return out if ret == 0 else None


def commit_files_changed(commit_hash):
    cmd = f"git diff-tree --no-commit-id --name-only -r {commit_hash}"
    ret, out, err = run(cmd)
    return out.split("\n") if ret == 0 else None
````

- EN: This block defines callable units such as `commit_body`, `commit_title`, `commit_files_changed`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 176-186 / 第 176-186 行

````python
def parse_pr_number(body, commit_hash, title):
    regex = r"Pull Request resolved: https://github.com/pytorch/pytorch/pull/([0-9]+)"
    matches = re.findall(regex, body)
    if len(matches) == 0:
        if "revert" not in title.lower() and "updating submodules" not in title.lower():
            print(f"[{commit_hash}: {title}] Could not parse PR number, ignoring PR")
        return None
    if len(matches) > 1:
        print(f"[{commit_hash}: {title}] Got two PR numbers, using the first one")
        return matches[0]
    return matches[0]
````

- EN: This block defines callable units such as `parse_pr_number`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 189-205 / 第 189-205 行

````python
def get_ghstack_token():
    pattern = "github_oauth = (.*)"
    with open(Path("~/.ghstackrc").expanduser(), "r+") as f:
        config = f.read()
    matches = re.findall(pattern, config)
    if len(matches) == 0:
        raise RuntimeError("Can't find a github oauth token")
    return matches[0]


def get_token():
    env_token = os.environ.get("GITHUB_TOKEN")
    if env_token is not None:
        print("using GITHUB_TOKEN from environment variable")
        return env_token
    else:
        return get_ghstack_token()
````

- EN: This block defines callable units such as `get_ghstack_token`, `get_token`; branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互。

### Lines 208-224 / 第 208-224 行

````python
token = get_token()

headers = {"Authorization": f"token {token}"}


def run_query(query):
    request = requests.post(
        "https://api.github.com/graphql",  # @lint-ignore
        json={"query": query},
        headers=headers,
    )
    if request.status_code == 200:
        return request.json()
    else:
        raise Exception(  # noqa: TRY002
            f"Query failed to run by returning code of {request.status_code}. {request.json()}"
        )
````

- EN: This block defines callable units such as `run_query`; branches on runtime conditions or configuration choices; returns computed values to callers; validates assumptions and surfaces failures explicitly; touches GitHub-related data or remote service interactions; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；校验前提条件并显式暴露失败；处理 GitHub 相关数据或远程服务交互；序列化或解析结构化数据文件。

### Lines 227-244 / 第 227-244 行

````python
_ERRORS = []
_MAX_ERROR_LEN = 20


def github_data(pr_number):
    query = (
        """
    {
      repository(owner: "pytorch", name: "pytorch") {
        pullRequest(number: %s ) {
          author {
            login
          }
          reviews(last: 5, states: APPROVED) {
            nodes {
              author {
                login
              }
````

- EN: This block defines callable units such as `github_data`; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；处理 GitHub 相关数据或远程服务交互。

### Lines 245-262 / 第 245-262 行

````python
            }
          }
          labels(first: 10) {
            edges {
              node {
                name
              }
            }
          }
        }
      }
    }
    """  # noqa: UP031
        % pr_number
    )
    query = run_query(query)
    if query.get("errors"):
        global _ERRORS
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。

### Lines 263-278 / 第 263-278 行

````python
        _ERRORS.append(query.get("errors"))
        if len(_ERRORS) < _MAX_ERROR_LEN:
            return [], "None", ()
        else:
            raise Exception(  # noqa: TRY002
                f"Got {_MAX_ERROR_LEN} errors: {_ERRORS}, please check if"
                " there is something wrong"
            )
    edges = query["data"]["repository"]["pullRequest"]["labels"]["edges"]
    labels = [edge["node"]["name"] for edge in edges]
    author = query["data"]["repository"]["pullRequest"]["author"]["login"]
    nodes = query["data"]["repository"]["pullRequest"]["reviews"]["nodes"]

    # using set to dedup multiple accepts from same accepter
    accepters = {node["author"]["login"] for node in nodes}
    accepters = tuple(sorted(accepters))
````

- EN: This block branches on runtime conditions or configuration choices; iterates through collections, records, or generated items; returns computed values to callers; validates assumptions and surfaces failures explicitly.
- CN: 该代码块依据运行时条件或配置选择进行分支；遍历集合、记录或生成项；向调用方返回计算结果；校验前提条件并显式暴露失败。

### Lines 280-296 / 第 280-296 行

````python
    return labels, author, accepters


def get_features(commit_hash):
    title, body, files_changed = (
        commit_title(commit_hash),
        commit_body(commit_hash),
        commit_files_changed(commit_hash),
    )
    pr_number = parse_pr_number(body, commit_hash, title)
    labels = []
    author = ""
    accepters = ()
    if pr_number is not None:
        labels, author, accepters = github_data(pr_number)
    result = Features(title, body, pr_number, files_changed, labels, author, accepters)
    return result
````

- EN: This block defines callable units such as `get_features`; branches on runtime conditions or configuration choices; returns computed values to callers; touches GitHub-related data or remote service interactions.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；处理 GitHub 相关数据或远程服务交互。

### Lines 299-309 / 第 299-309 行

````python
_commit_data_cache = None


def get_commit_data_cache(path="results/data.json"):
    global _commit_data_cache
    if _commit_data_cache is None:
        _commit_data_cache = _CommitDataCache(path)
    return _commit_data_cache


class _CommitDataCache:
````

- EN: This block introduces classes like `_CommitDataCache`; defines callable units such as `get_commit_data_cache`; branches on runtime conditions or configuration choices; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 310-323 / 第 310-323 行

````python
    def __init__(self, path):
        self.path = path
        self.data = {}
        if os.path.exists(path):
            self.data = self.read_from_disk()
        else:
            os.makedirs(Path(path).parent, exist_ok=True)

    def get(self, commit):
        if commit not in self.data.keys():
            # Fetch and cache the data
            self.data[commit] = get_features(commit)
            self.write_to_disk()
        return self.data[commit]
````

- EN: This block defines callable units such as `__init__`, `get`; branches on runtime conditions or configuration choices; returns computed values to callers.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；依据运行时条件或配置选择进行分支；向调用方返回计算结果。

### Lines 325-329 / 第 325-329 行

````python
    def read_from_disk(self):
        with open(self.path) as f:
            data = json.load(f)
            data = {commit: dict_to_features(dct) for commit, dct in data.items()}
        return data
````

- EN: This block defines callable units such as `read_from_disk`; iterates through collections, records, or generated items; returns computed values to callers; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；向调用方返回计算结果；序列化或解析结构化数据文件。

### Lines 331-334 / 第 331-334 行

````python
    def write_to_disk(self):
        data = {commit: features._asdict() for commit, features in self.data.items()}
        with open(self.path, "w") as f:
            json.dump(data, f)
````

- EN: This block defines callable units such as `write_to_disk`; iterates through collections, records, or generated items; serializes or parses structured data files.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；序列化或解析结构化数据文件。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Configures native build inputs, optional backends, and dependency discovery at configure time.
  CN: 在配置阶段设置原生构建输入、可选后端以及依赖发现逻辑。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Processes generated artifacts into summarized reports or machine-consumable metrics.
  CN: 把生成的产物处理为汇总报告或机器可消费的指标。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `json`, `locale`, `os`, `re`, `subprocess`, `collections`, `dataclasses`, `pathlib`
- Internal imports / 内部导入: none
- External imports / 外部导入: `requests`
- Classes / 类: `CategoryGroup`, `_CommitDataCache`
- Functions / 函数: `dict_to_features`, `features_to_dict`, `run`, `commit_body`, `commit_title`, `commit_files_changed`, `parse_pr_number`, `get_ghstack_token`, `get_token`, `run_query`, ...
- Test entry points / 测试入口: none
