# url_schemes.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `docs/mkdocs/hooks/url_schemes.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Build-time documentation hook or generator for MkDocs / 面向 MkDocs 的构建期文档 hook 或生成器

## Line-by-Line Analysis / 逐行分析
### Module setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
"""
MkDocs hook + markdown extension to enable the following links to render correctly,
including inside content included via pymdownx.snippets:

- Relative file links outside of the `docs/` directory, e.g.:
    - [Text](../some_file.py)
    - [Directory](../../some_directory/)
- GitHub URLs for issues, pull requests, and projects, e.g.:
    - Adds GitHub icon before links
    - Replaces raw links with descriptive text,
        e.g. <...pull/123> -> [Pull Request #123](.../pull/123)
    - Works for external repos too by including the `owner/repo` in the link title

The link replacement runs as a markdown preprocessor (priority 25) so that it executes
after pymdownx.snippets (priority 32) has expanded all included content.
The on_page_markdown hook passes the current page context to the preprocessor before
each page is converted.
"""

from pathlib import Path

import regex as re
from markdown import Extension
from markdown.preprocessors import Preprocessor
from mkdocs.config.defaults import MkDocsConfig
from mkdocs.structure.files import Files
from mkdocs.structure.pages import Page

ROOT_DIR = Path(__file__).parent.parent.parent.parent.resolve()
DOC_DIR = ROOT_DIR / "docs"

gh_icon = ":octicons-mark-github-16:"

# Regex pieces
TITLE = r"(?P<title>[^\[\]<>]+?)"
REPO = r"(?P<repo>.+?/.+?)"
TYPE = r"(?P<type>issues|pull|projects)"
NUMBER = r"(?P<number>\d+)"
PATH = r"(?P<path>[^\s]+?)"
FRAGMENT = r"(?P<fragment>#[^\s]+)?"
URL = f"https://github.com/{REPO}/{TYPE}/{NUMBER}{FRAGMENT}"
RELATIVE = rf"(?!(https?|ftp)://|#){PATH}{FRAGMENT}"

# Common titles to use for GitHub links when none is provided in the link.
TITLES = {"issues": "Issue ", "pull": "Pull Request ", "projects": "Project "}

# Regex to match GitHub issue, PR, and project links with optional titles.
github_link = re.compile(rf"(\[{TITLE}\]\(|<){URL}(\)|>)")
# Regex to match relative file links with optional titles.
relative_link = re.compile(rf"\[{TITLE}\]\({RELATIVE}\)")
```
**EN:** The opening block establishes imports, constants, globals, and module-level context for later hooks or helpers.
**CN:** 开头部分建立导入、常量、全局变量以及后续 hook/辅助函数所需的模块上下文。

### UrlSchemesPreprocessor
```python
class UrlSchemesPreprocessor(Preprocessor):
    """Preprocessor that runs after pymdownx.snippets to process all links."""

    def __init__(self, md, ext):
        super().__init__(md)
        self.ext = ext

    def run(self, lines):
        page = self.ext.page
        if page is None or getattr(page.file, "abs_src_path", None) is None:
            return lines

        def replace_relative_link(match: re.Match) -> str:
            """
            Replace relative file links with URLs if they point outside the docs dir.
            """
            title = match.group("title")
            path = match.group("path")
            path = (Path(page.file.abs_src_path).parent / path).resolve()
            fragment = match.group("fragment") or ""

            # Check if the path exists and is outside the docs dir
            if not path.exists() or path.is_relative_to(DOC_DIR):
                return match.group(0)

            # Files and directories have different URL schemes on GitHub
            slug = "tree/main" if path.is_dir() else "blob/main"

            path = path.relative_to(ROOT_DIR)
            url = f"https://github.com/vllm-project/vllm/{slug}/{path}{fragment}"
            return f"[{gh_icon} {title}]({url})"

        def replace_github_link(match: re.Match) -> str:
            """
            Replace GitHub issue, PR, and project links with enhanced Markdown links.
            """
            repo = match.group("repo")
            type = match.group("type")
            number = match.group("number")
            # Title and fragment could be None
            title = match.group("title") or ""
            fragment = match.group("fragment") or ""

            # Use default titles for raw links
            if not title:
                title = TITLES[type]
                if "vllm-project" not in repo:
                    title += repo
                title += f"#{number}"

            url = f"https://github.com/{repo}/{type}/{number}{fragment}"
            return f"[{gh_icon} {title}]({url})"

        markdown = "\n".join(lines)
        markdown = relative_link.sub(replace_relative_link, markdown)
        markdown = github_link.sub(replace_github_link, markdown)
        return markdown.split("\n")
```
**EN:** This class plugs custom transformation logic into the Markdown/MkDocs pipeline.
**CN:** 该class把自定义转换逻辑接入 Markdown/MkDocs 处理流水线。

### UrlSchemesExtension
```python
class UrlSchemesExtension(Extension):
    """Markdown extension that registers the URL schemes preprocessor."""

    def __init__(self, **kwargs):
        self.page = None
        super().__init__(**kwargs)

    def extendMarkdown(self, md):
        # Priority 25 runs after pymdownx.snippets (priority 32)
        md.preprocessors.register(UrlSchemesPreprocessor(md, self), "url_schemes", 25)
```
**EN:** This class plugs custom transformation logic into the Markdown/MkDocs pipeline.
**CN:** 该class把自定义转换逻辑接入 Markdown/MkDocs 处理流水线。

### on_config
```python
def on_config(config: MkDocsConfig) -> MkDocsConfig:
    """Register the URL schemes markdown extension."""
    config["markdown_extensions"].append(_ext)
    return config
```
**EN:** This function is a MkDocs hook entrypoint that is executed during build/render events.
**CN:** 该function是 MkDocs 的钩子入口，会在构建或渲染事件中执行。

### on_page_markdown
```python
def on_page_markdown(
    markdown: str, *, page: Page, config: MkDocsConfig, files: Files
) -> str:
    """Pass the current page context to the preprocessor."""
    _ext.page = page
    return markdown
```
**EN:** This function is a MkDocs hook entrypoint that is executed during build/render events.
**CN:** 该function是 MkDocs 的钩子入口，会在构建或渲染事件中执行。

## Key Concepts / 关键概念
- **EN:** The file is build-time tooling: it shapes generated docs or page rendering rather than model execution.
  **CN:** 该文件属于构建期工具：它影响生成文档或页面渲染，而不是模型执行本身。
- **EN:** MkDocs hook functions are the entrypoints that connect the module to the documentation build lifecycle.
  **CN:** MkDocs hook 函数是把该模块接入文档构建生命周期的入口。
- **EN:** Classes in the file encapsulate reusable parsing, formatting, or preprocessing behavior.
  **CN:** 文件中的类封装了可复用的解析、格式化或预处理行为。

## Dependencies / 依赖关系
- **EN:** `pathlib` is imported directly and participates in the hook or generation flow.
  **CN:** `pathlib` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `regex` is imported directly and participates in the hook or generation flow.
  **CN:** `regex` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `markdown` is imported directly and participates in the hook or generation flow.
  **CN:** `markdown` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `markdown.preprocessors` is imported directly and participates in the hook or generation flow.
  **CN:** `markdown.preprocessors` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `mkdocs.config.defaults` is imported directly and participates in the hook or generation flow.
  **CN:** `mkdocs.config.defaults` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `mkdocs.structure.files` is imported directly and participates in the hook or generation flow.
  **CN:** `mkdocs.structure.files` 被直接导入，并参与该 hook 或生成流程。
