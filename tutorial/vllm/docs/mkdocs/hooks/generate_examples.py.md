# generate_examples.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `docs/mkdocs/hooks/generate_examples.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: Build-time documentation hook or generator for MkDocs / 面向 MkDocs 的构建期文档 hook 或生成器

## Line-by-Line Analysis / 逐行分析
### Module setup
```python
# SPDX-License-Identifier: Apache-2.0
# SPDX-FileCopyrightText: Copyright contributors to the vLLM project
import itertools
import logging
from dataclasses import dataclass
from functools import cached_property
from pathlib import Path
from typing import Literal

import regex as re

logger = logging.getLogger("mkdocs")

ROOT_DIR = Path(__file__).parent.parent.parent.parent
ROOT_DIR_RELATIVE = "../../../../.."
EXAMPLE_DIR = ROOT_DIR / "examples"
EXAMPLE_DOC_DIR = ROOT_DIR / "docs/examples"
```
**EN:** The opening block establishes imports, constants, globals, and module-level context for later hooks or helpers.
**CN:** 开头部分建立导入、常量、全局变量以及后续 hook/辅助函数所需的模块上下文。

### title
```python
def title(text: str) -> str:
    # Default title case
    text = text.replace("_", " ").replace("/", " - ").title()
    # Custom substitutions
    subs = {
        "io": "IO",
        "rl": "RL",
        "api(s?)": r"API\1",
        "cli": "CLI",
        "cpu": "CPU",
        "ipc": "IPC",
        "llm": "LLM",
        "mae": "MAE",
        "ner": "NER",
        "tpu": "TPU",
        "gguf": "GGUF",
        "lora": "LoRA",
        "nccl": "NCCL",
        "rlhf": "RLHF",
        "vllm": "vLLM",
        "openai": "OpenAI",
        "lmcache": "LMCache",
        "multilora": "MultiLoRA",
        "mlpspeculator": "MLPSpeculator",
        r"fp\d+": lambda x: x.group(0).upper(),  # e.g. fp16, fp32
        r"int\d+": lambda x: x.group(0).upper(),  # e.g. int8, int16
    }
    for pattern, repl in subs.items():
        text = re.sub(rf"\b{pattern}\b", repl, text, flags=re.IGNORECASE)
    return text
```
**EN:** This function normalizes presentation details so generated docs are easier to read.
**CN:** 该function负责规范展示细节，使生成文档更易读。

### Example
```python
class Example:
    """
    Example class for generating documentation content from a given path.

    Attributes:
        path (Path): The path to the main directory or file.
        category (str): The category of the document.

    Properties::
        main_file() -> Path | None: Determines the main file in the given path.
        other_files() -> list[Path]: Determines other files in the directory excluding
        the main file.
        title() -> str: Determines the title of the document.

    Methods:
        generate() -> str: Generates the documentation content.
    """

    path: Path
    category: str

    @cached_property
    def main_file(self) -> Path | None:
        """Determines the main file in the given path.

        If path is a file, it returns the path itself. If path is a directory, it
        searches for Markdown files (*.md) in the directory and returns the first one
        found. If no Markdown files are found, it returns None."""
        # Single file example
        if self.path.is_file():
            return self.path
        # Multi file example with a README
        if md_paths := list(self.path.glob("*.md")):
            return md_paths[0]
        # Multi file example without a README
        return None

    @cached_property
    def other_files(self) -> list[Path]:
        """Determine other files in the directory excluding the main file.

        If path is a file, it returns an empty list. Otherwise, it returns every file
        in the directory except the main file in a list."""
        # Single file example
        if self.path.is_file():
            return []
        # Multi file example
        is_other_file = lambda file: file.is_file() and file != self.main_file
        return sorted(file for file in self.path.rglob("*") if is_other_file(file))

    @cached_property
    def is_code(self) -> bool:
        return self.main_file is not None and self.main_file.suffix != ".md"

    @cached_property
    def title(self) -> str:
        # Generate title from filename if no main md file found
        if self.main_file is None or self.is_code:
            return title(self.path.stem)
        # Specify encoding for building on Windows
        with open(self.main_file, encoding="utf-8") as f:
            first_line = f.readline().strip()
        match = re.match(r"^#\s+(?P<title>.+)$", first_line)
        if match:
            return match.group("title")
        raise ValueError(f"Title not found in {self.main_file}")

    def fix_relative_links(self, content: str) -> str:
        """
        Fix relative links in markdown content by converting them to gh-file
        format.

        Args:
            content (str): The markdown content to process

        Returns:
            str: Content with relative links converted to gh-file format
        """
        # Regex to match markdown links [text](relative_path)
        # This matches links that don't start with http, https, ftp, or #
        link_pattern = r"\[([^\]]*)\]\((?!(?:https?|ftp)://|#)([^)]+)\)"

        def replace_link(match):
            link_text = match.group(1)
            relative_path = match.group(2)

            # Make relative to repo root
            gh_file = (self.main_file.parent / relative_path).resolve()
            gh_file = gh_file.relative_to(ROOT_DIR)

            # Make GitHub URL
            url = "https://github.com/vllm-project/vllm/"
            url += "tree/main" if self.path.is_dir() else "blob/main"
            gh_url = f"{url}/{gh_file}"

            return f"[{link_text}]({gh_url})"

        return re.sub(link_pattern, replace_link, content)

    def generate(self) -> str:
        content = f"# {self.title}\n\n"
        url = "https://github.com/vllm-project/vllm/"
        url += "tree/main" if self.path.is_dir() else "blob/main"
        content += f"Source <{url}/{self.path.relative_to(ROOT_DIR)}>.\n\n"

        # Use long code fence to avoid issues with
        # included files containing code fences too
        code_fence = "``````"

        if self.main_file is not None:
            # Single file example or multi file example with a README
            if self.is_code:
                content += (
                    f"{code_fence}{self.main_file.suffix[1:]}\n"
                    f'--8<-- "{self.main_file}"\n'
                    f"{code_fence}\n"
                )
            else:
                with open(self.main_file, encoding="utf-8") as f:
                    # Skip the title from md snippets as it's been included above
                    main_content = f.readlines()[1:]
                content += self.fix_relative_links("".join(main_content))
            content += "\n"
        else:
            # Multi file example without a README
            for file in self.other_files:
                file_title = title(str(file.relative_to(self.path).with_suffix("")))
                content += f"## {file_title}\n\n"
                content += (
                    f'{code_fence}{file.suffix[1:]}\n--8<-- "{file}"\n{code_fence}\n\n'
                )
            return content

        if not self.other_files:
            return content

        content += "## Example materials\n\n"
        for file in self.other_files:
            content += f'??? abstract "{file.relative_to(self.path)}"\n'
            if file.suffix != ".md":
                content += f"    {code_fence}{file.suffix[1:]}\n"
            content += f'    --8<-- "{file}"\n'
            if file.suffix != ".md":
                content += f"    {code_fence}\n"

        return content
```
**EN:** This class contains one of the main logical units in the file and supports the documented generation pipeline.
**CN:** 该class是文件中的主要逻辑单元之一，用来支撑对应的文档生成流程。

### on_startup
```python
def on_startup(command: Literal["build", "gh-deploy", "serve"], dirty: bool):
    # Monkey-patch dirname_to_title in awesome-nav so that sub-directory names are
    # title-cased (e.g. "Offline Inference" instead of "Offline inference").
    import mkdocs_awesome_nav.nav.directory as _nav_dir

    _nav_dir.dirname_to_title = title
    logger.info("Generating example documentation")
    logger.debug("Root directory: %s", ROOT_DIR.resolve())
    logger.debug("Example directory: %s", EXAMPLE_DIR.resolve())
    logger.debug("Example document directory: %s", EXAMPLE_DOC_DIR.resolve())

    # Create the EXAMPLE_DOC_DIR if it doesn't exist
    if not EXAMPLE_DOC_DIR.exists():
        EXAMPLE_DOC_DIR.mkdir(parents=True)

    categories = sorted(p for p in EXAMPLE_DIR.iterdir() if p.is_dir())

    examples = []
    glob_patterns = ["*.py", "*.md", "*.sh"]
    # Find categorised examples
    for category in categories:
        logger.info("Processing category: %s", category.stem)
        globs = [category.glob(pattern) for pattern in glob_patterns]
        for path in itertools.chain(*globs):
            examples.append(Example(path, category.stem))
        # Find examples in subdirectories
        globs = [category.glob(f"*/{pattern}") for pattern in glob_patterns]
        for path in itertools.chain(*globs):
            examples.append(Example(path.parent, category.stem))

    # Generate the example documentation
    for example in sorted(examples, key=lambda e: e.path.stem):
        example_name = f"{example.path.stem}.md"
        doc_path = EXAMPLE_DOC_DIR / example.category / example_name
        if not doc_path.parent.exists():
            doc_path.parent.mkdir(parents=True)
        # Specify encoding for building on Windows
        with open(doc_path, "w+", encoding="utf-8") as f:
            f.write(example.generate())
        logger.debug("Example generated: %s", doc_path.relative_to(ROOT_DIR))
    logger.info("Total examples generated: %d", len(examples))
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
- **EN:** `itertools` is imported directly and participates in the hook or generation flow.
  **CN:** `itertools` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `logging` is imported directly and participates in the hook or generation flow.
  **CN:** `logging` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `dataclasses` is imported directly and participates in the hook or generation flow.
  **CN:** `dataclasses` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `functools` is imported directly and participates in the hook or generation flow.
  **CN:** `functools` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `pathlib` is imported directly and participates in the hook or generation flow.
  **CN:** `pathlib` 被直接导入，并参与该 hook 或生成流程。
- **EN:** `typing` is imported directly and participates in the hook or generation flow.
  **CN:** `typing` 被直接导入，并参与该 hook 或生成流程。
