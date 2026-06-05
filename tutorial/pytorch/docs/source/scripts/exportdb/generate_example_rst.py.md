# generate_example_rst.py — Code Analysis / 代码分析
## Source / 来源
- File: `docs/source/scripts/exportdb/generate_example_rst.py`
- Repository: `/root/xw/pytorch`
- Purpose (EN): Generate documentation assets or intermediate reStructuredText related to `generate_example_rst`.
- 用途 (CN): 生成与 `generate_example_rst` 相关的文档资源或中间 reStructuredText 文件。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1 / 第 1-1 行
```python
import inspect
```
- EN: This segment imports `inspect`, establishing dependencies needed by the rest of the script.
- CN: 这一段导入了 `inspect`，为脚本后续逻辑建立依赖。

### Lines 2-2 / 第 2-2 行
```python
import os
```
- EN: This segment imports `os`, establishing dependencies needed by the rest of the script.
- CN: 这一段导入了 `os`，为脚本后续逻辑建立依赖。

### Lines 3-3 / 第 3-3 行
```python
import re
```
- EN: This segment imports `re`, establishing dependencies needed by the rest of the script.
- CN: 这一段导入了 `re`，为脚本后续逻辑建立依赖。

### Lines 4-4 / 第 4-4 行
```python
from pathlib import Path
```
- EN: This segment imports `pathlib.Path`, establishing dependencies needed by the rest of the script. It manipulates filesystem paths and generated files.
- CN: 这一段导入了 `pathlib.Path`，为脚本后续逻辑建立依赖。 它会处理文件系统路径以及生成出的文件。

### Lines 6-6 / 第 6-6 行
```python
import torch
```
- EN: This segment imports `torch`, establishing dependencies needed by the rest of the script. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段导入了 `torch`，为脚本后续逻辑建立依赖。 它会与 PyTorch 模块、符号或示例交互。

### Lines 7-7 / 第 7-7 行
```python
import torch._dynamo as torchdynamo
```
- EN: This segment imports `torch._dynamo`, establishing dependencies needed by the rest of the script. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段导入了 `torch._dynamo`，为脚本后续逻辑建立依赖。 它会与 PyTorch 模块、符号或示例交互。

### Lines 8-8 / 第 8-8 行
```python
from torch._export.db.case import ExportCase
```
- EN: This segment imports `torch._export.db.case.ExportCase`, establishing dependencies needed by the rest of the script. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段导入了 `torch._export.db.case.ExportCase`，为脚本后续逻辑建立依赖。 它会与 PyTorch 模块、符号或示例交互。

### Lines 9-9 / 第 9-9 行
```python
from torch._export.db.examples import all_examples
```
- EN: This segment imports `torch._export.db.examples.all_examples`, establishing dependencies needed by the rest of the script. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段导入了 `torch._export.db.examples.all_examples`，为脚本后续逻辑建立依赖。 它会与 PyTorch 模块、符号或示例交互。

### Lines 10-10 / 第 10-10 行
```python
from torch.export import export
```
- EN: This segment imports `torch.export.export`, establishing dependencies needed by the rest of the script. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段导入了 `torch.export.export`，为脚本后续逻辑建立依赖。 它会与 PyTorch 模块、符号或示例交互。

### Lines 13-13 / 第 13-13 行
```python
PWD = Path(__file__).absolute().parent
```
- EN: This segment defines or updates `PWD`, which shapes later behavior in the file. It manipulates filesystem paths and generated files.
- CN: 这一段定义或更新了 `PWD`，从而影响文件后续行为。 它会处理文件系统路径以及生成出的文件。

### Lines 14-14 / 第 14-14 行
```python
ROOT = Path(__file__).absolute().parents[3]
```
- EN: This segment defines or updates `ROOT`, which shapes later behavior in the file. It manipulates filesystem paths and generated files.
- CN: 这一段定义或更新了 `ROOT`，从而影响文件后续行为。 它会处理文件系统路径以及生成出的文件。

### Lines 15-15 / 第 15-15 行
```python
SOURCE = ROOT / "source"
```
- EN: This segment defines or updates `SOURCE`, which shapes later behavior in the file.
- CN: 这一段定义或更新了 `SOURCE`，从而影响文件后续行为。

### Lines 16-16 / 第 16-16 行
```python
EXPORTDB_SOURCE = SOURCE / "generated" / "exportdb"
```
- EN: This segment defines or updates `EXPORTDB_SOURCE`, which shapes later behavior in the file.
- CN: 这一段定义或更新了 `EXPORTDB_SOURCE`，从而影响文件后续行为。

### Lines 19-95 / 第 19-95 行
```python
def generate_example_rst(example_case: ExportCase):
    """
    Generates the .rst files for all the examples in db/examples/
    """

    model = example_case.model

    tags = ", ".join(f":doc:`{tag} <{tag}>`" for tag in example_case.tags)

    source_file = (
        inspect.getfile(model.__class__)
        if isinstance(model, torch.nn.Module)
        else inspect.getfile(model)
    )
    with open(source_file) as file:
        source_code = file.read()
    source_code = source_code.replace("\n", "\n    ")
    splitted_source_code = re.split(r"@export_rewrite_case.*\n", source_code)

    if len(splitted_source_code) not in {1, 2}:
        raise AssertionError(
            f"more than one @export_rewrite_case decorator in {source_code}"
        )

    more_arguments = ""
    if example_case.example_kwargs:
        more_arguments += ", example_kwargs"
    if example_case.dynamic_shapes:
        more_arguments += ", dynamic_shapes=dynamic_shapes"

    # Generate contents of the .rst file
    title = f"{example_case.name}"
    doc_contents = f"""{title}
{"^" * (len(title))}

.. note::

    Tags: {tags}

    Support Level: {example_case.support_level.name}

Original source code:

.. code-block:: python

    {splitted_source_code[0]}

    torch.export.export(model, example_args{more_arguments})

Result:

.. code-block::

"""

    # Get resulting graph from dynamo trace
    try:
        exported_program = export(
            model,
            example_case.example_args,
            example_case.example_kwargs,
            dynamic_shapes=example_case.dynamic_shapes,
            strict=True,
        )
        graph_output = str(exported_program)
        graph_output = re.sub(r"        # File(.|\n)*?\n", "", graph_output)
        graph_output = graph_output.replace("\n", "\n    ")
        output = f"    {graph_output}"
    except torchdynamo.exc.Unsupported as e:
        output = "    Unsupported: " + str(e).split("\n")[0]
    except AssertionError as e:
        output = "    AssertionError: " + str(e).split("\n")[0]
    except RuntimeError as e:
        output = "    RuntimeError: " + str(e).split("\n")[0]

    doc_contents += output + "\n"
```
- EN: This is part 1/2 of function `generate_example_rst(example_case)`, so it continues the implementation rather than introducing a new top-level concept. The docstring summarizes its intent as: Generates the .rst files for all the examples in db/examples/ Notable calls include `', '.join`, `source_code.replace`, `re.split`, `isinstance`, `inspect.getfile`, `open`, which hint at the services this function relies on. It interacts with PyTorch modules, symbols, or examples. It uses pattern matching to transform or inspect text.
- CN: 这一段是函数 `generate_example_rst(example_case)` 的 1/2 部分，因此它是在继续实现，而不是引入新的顶层概念。 其文档字符串将意图概括为：Generates the .rst files for all the examples in db/examples/ 值得注意的调用包括 `', '.join`, `source_code.replace`, `re.split`, `isinstance`, `inspect.getfile`, `open`，这些调用揭示了该函数依赖的服务。 它会与 PyTorch 模块、符号或示例交互。 它使用模式匹配来转换或检查文本。

### Lines 96-106 / 第 96-106 行
```python
    if len(splitted_source_code) == 2:
        doc_contents += f"""\n
You can rewrite the example above to something like the following:

.. code-block:: python

{splitted_source_code[1]}

"""

    return doc_contents
```
- EN: This is part 2/2 of function `generate_example_rst(example_case)`, so it continues the implementation rather than introducing a new top-level concept. The docstring summarizes its intent as: Generates the .rst files for all the examples in db/examples/ Notable calls include `', '.join`, `source_code.replace`, `re.split`, `isinstance`, `inspect.getfile`, `open`, which hint at the services this function relies on.
- CN: 这一段是函数 `generate_example_rst(example_case)` 的 2/2 部分，因此它是在继续实现，而不是引入新的顶层概念。 其文档字符串将意图概括为：Generates the .rst files for all the examples in db/examples/ 值得注意的调用包括 `', '.join`, `source_code.replace`, `re.split`, `isinstance`, `inspect.getfile`, `open`，这些调用揭示了该函数依赖的服务。

### Lines 109-148 / 第 109-148 行
```python
def generate_index_rst(example_cases, tag_to_modules, support_level_to_modules):
    """
    Generates the index.rst file
    """

    support_contents = ""
    for k, v in support_level_to_modules.items():
        support_level = k.name.lower().replace("_", " ").title()
        module_contents = "\n\n".join(v)
        support_contents += f"""
{support_level}
{"-" * (len(support_level))}

{module_contents}
"""

    tag_names = "\n    ".join(t for t in tag_to_modules)

    with open(os.path.join(PWD, "blurb.txt")) as file:
        blurb = file.read()

    # Generate contents of the .rst file
    doc_contents = f""".. _torch.export_db:

ExportDB
========

{blurb}

.. toctree::
    :maxdepth: 1
    :caption: Tags

    {tag_names}

{support_contents}
"""

    with open(os.path.join(EXPORTDB_SOURCE, "index.rst"), "w") as f:
        f.write(doc_contents)
```
- EN: This segment defines function `generate_index_rst(example_cases, tag_to_modules, support_level_to_modules)` and packages a reusable step in the documentation workflow. The docstring summarizes its intent as: Generates the index.rst file Notable calls include `support_level_to_modules.items`, `'\n    '.join`, `k.name.lower().replace('_', ' ').title`, `'\n\n'.join`, `open`, `file.read`, which hint at the services this function relies on. It manipulates filesystem paths and generated files. It interacts with PyTorch modules, symbols, or examples.
- CN: 这一段定义了函数 `generate_index_rst(example_cases, tag_to_modules, support_level_to_modules)`，用于封装文档流程中的可复用步骤。 其文档字符串将意图概括为：Generates the index.rst file 值得注意的调用包括 `support_level_to_modules.items`, `'\n    '.join`, `k.name.lower().replace('_', ' ').title`, `'\n\n'.join`, `open`, `file.read`，这些调用揭示了该函数依赖的服务。 它会处理文件系统路径以及生成出的文件。 它会与 PyTorch 模块、符号或示例交互。

### Lines 151-166 / 第 151-166 行
```python
def generate_tag_rst(tag_to_modules):
    """
    For each tag that shows up in each ExportCase.tag, generate an .rst file
    containing all the examples that have that tag.
    """

    for tag, modules_rst in tag_to_modules.items():
        doc_contents = f"{tag}\n{'=' * (len(tag) + 4)}\n"
        full_modules_rst = "\n\n".join(modules_rst)
        full_modules_rst = re.sub(
            r"={3,}", lambda match: "-" * len(match.group()), full_modules_rst
        )
        doc_contents += full_modules_rst

        with open(os.path.join(EXPORTDB_SOURCE, f"{tag}.rst"), "w") as f:
            f.write(doc_contents)
```
- EN: This segment defines function `generate_tag_rst(tag_to_modules)` and packages a reusable step in the documentation workflow. The docstring summarizes its intent as: For each tag that shows up in each ExportCase.tag, generate an .rst file containing all the examples that have that tag. Notable calls include `tag_to_modules.items`, `'\n\n'.join`, `re.sub`, `open`, `f.write`, `os.path.join`, which hint at the services this function relies on. It manipulates filesystem paths and generated files. It uses pattern matching to transform or inspect text.
- CN: 这一段定义了函数 `generate_tag_rst(tag_to_modules)`，用于封装文档流程中的可复用步骤。 其文档字符串将意图概括为：For each tag that shows up in each ExportCase.tag, generate an .rst file containing all the examples that have that tag. 值得注意的调用包括 `tag_to_modules.items`, `'\n\n'.join`, `re.sub`, `open`, `f.write`, `os.path.join`，这些调用揭示了该函数依赖的服务。 它会处理文件系统路径以及生成出的文件。 它使用模式匹配来转换或检查文本。

### Lines 169-187 / 第 169-187 行
```python
def generate_rst():
    if not os.path.exists(EXPORTDB_SOURCE):
        os.makedirs(EXPORTDB_SOURCE)

    example_cases = all_examples()
    tag_to_modules = {}
    support_level_to_modules = {}
    for example_case in example_cases.values():
        doc_contents = generate_example_rst(example_case)

        for tag in example_case.tags:
            tag_to_modules.setdefault(tag, []).append(doc_contents)

        support_level_to_modules.setdefault(example_case.support_level, []).append(
            doc_contents
        )

    generate_tag_rst(tag_to_modules)
    generate_index_rst(example_cases, tag_to_modules, support_level_to_modules)
```
- EN: This segment defines function `generate_rst()` and packages a reusable step in the documentation workflow. Notable calls include `all_examples`, `example_cases.values`, `generate_tag_rst`, `generate_index_rst`, `os.path.exists`, `os.makedirs`, which hint at the services this function relies on. It manipulates filesystem paths and generated files.
- CN: 这一段定义了函数 `generate_rst()`，用于封装文档流程中的可复用步骤。 值得注意的调用包括 `all_examples`, `example_cases.values`, `generate_tag_rst`, `generate_index_rst`, `os.path.exists`, `os.makedirs`，这些调用揭示了该函数依赖的服务。 它会处理文件系统路径以及生成出的文件。

### Lines 190-191 / 第 190-191 行
```python
if __name__ == "__main__":
    generate_rst()
```
- EN: This conditional branch evaluates `__name__ == '__main__'` and gates behavior on that runtime or configuration check.
- CN: 这一条件分支会评估 `__name__ == '__main__'`，并据此控制相应的运行时或配置行为。

## Key Concepts / 关键概念
- EN: `inspect` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `inspect` —— 在标题、示例或行内强调中反复出现的主题。
- EN: `os` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `os` —— 在标题、示例或行内强调中反复出现的主题。
- EN: `re` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `re` —— 在标题、示例或行内强调中反复出现的主题。
- EN: `pathlib.Path` — API symbol or namespace repeatedly referenced by the file.
  CN: `pathlib.Path` —— 文件中反复引用的 API 符号或命名空间。
- EN: `torch` — API symbol or namespace repeatedly referenced by the file.
  CN: `torch` —— 文件中反复引用的 API 符号或命名空间。
- EN: `torch._dynamo` — API symbol or namespace repeatedly referenced by the file.
  CN: `torch._dynamo` —— 文件中反复引用的 API 符号或命名空间。
- EN: `torch._export.db.case.ExportCase` — API symbol or namespace repeatedly referenced by the file.
  CN: `torch._export.db.case.ExportCase` —— 文件中反复引用的 API 符号或命名空间。
- EN: `torch._export.db.examples.all_examples` — API symbol or namespace repeatedly referenced by the file.
  CN: `torch._export.db.examples.all_examples` —— 文件中反复引用的 API 符号或命名空间。
- EN: `torch.export.export` — API symbol or namespace repeatedly referenced by the file.
  CN: `torch.export.export` —— 文件中反复引用的 API 符号或命名空间。
- EN: `generate_example_rst` — Recurring topic surfaced by headings, examples, or inline emphasis.
  CN: `generate_example_rst` —— 在标题、示例或行内强调中反复出现的主题。

## Dependencies / 依赖关系
- EN: Depends on `inspect` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `inspect`。
- EN: Depends on `os` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `os`。
- EN: Depends on `re` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `re`。
- EN: Depends on `pathlib.Path` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `pathlib.Path`。
- EN: Depends on `torch` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `torch`。
- EN: Depends on `torch._dynamo` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `torch._dynamo`。
- EN: Depends on `torch._export.db.case.ExportCase` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `torch._export.db.case.ExportCase`。
- EN: Depends on `torch._export.db.examples.all_examples` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `torch._export.db.examples.all_examples`。
- EN: Depends on `torch.export.export` through imports, environment access, or tool invocation.
  CN: 通过导入、环境变量访问或工具调用依赖 `torch.export.export`。
