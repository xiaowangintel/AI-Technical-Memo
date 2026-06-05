# generate_doc.py — Code Analysis / 代码分析

## Source / 来源

- File / 文件: `bolt/docs/generate_doc.py`
- Repository / 仓库: `llvm-project`
- Purpose / 用途: !/usr/bin/env python3 A tool to parse the output of `llvm-bolt --help-hidden` and update the documentation in CommandLineArgumentReference.md automatically. Run from the directory in which this file is located to update the docs.. It also sits in code that supports BOLT documentation tooling and helper scripts. / 该文件支撑 BOLT 文档工具与辅助脚本。 源码头部说明其职责是：!/usr/bin/env python3 A tool to parse the output of `llvm-bolt --help-hidden` and update the documentation in CommandLineArgumentReference.md automatically. Run from the directory in which this file is located to update the docs.。

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

```python
#!/usr/bin/env python3
# A tool to parse the output of `llvm-bolt --help-hidden` and update the
# documentation in CommandLineArgumentReference.md automatically.
# Run from the directory in which this file is located to update the docs.

import subprocess
from textwrap import wrap
```

- EN: Imports Python modules such as `subprocess`, `from textwrap import wrap` to support scripting logic. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `subprocess`, `from textwrap import wrap`.
- CN: 这里导入 Python 模块，例如 `subprocess`, `from textwrap import wrap`，以支撑脚本逻辑。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `subprocess`, `from textwrap import wrap`。

### Lines 9-16

```python
LINE_LIMIT = 80


def wrap_text(text, indent, limit=LINE_LIMIT):
    wrapped_lines = wrap(text, width=limit - len(indent))
    wrapped_text = ("\n" + indent).join(wrapped_lines)
    return wrapped_text
```

- EN: Declares or implements routines including `wrap`. Notable symbols here include `wrap`.
- CN: 这里声明或实现函数，例如 `wrap`。这里较值得关注的符号包括 `wrap`。

### Lines 17-27

```python

def add_info(sections, section, option, description):
    indent = "  "
    wrapped_description = "\n".join(
        [
            wrap_text(line, indent) if len(line) > LINE_LIMIT else line
            for line in description
        ]
    )
    sections[section].append((option, indent + wrapped_description))
```

- EN: Declares or implements routines including `wrap_text`. Notable symbols here include `wrap_text`.
- CN: 这里声明或实现函数，例如 `wrap_text`。这里较值得关注的符号包括 `wrap_text`。

### Lines 28-39

```python

def parse_bolt_options(output):
    section_headers = [
        "Generic options:",
        "Output options:",
        "BOLT generic options:",
        "BOLT optimization options:",
        "BOLT options in relocation mode:",
        "BOLT instrumentation options:",
        "BOLT printing options:",
    ]
```

- EN: Continues the surrounding implementation, wiring declarations, state, and helper logic together.
- CN: 这里继续展开周边实现，衔接声明、状态与辅助逻辑。

### Lines 40-51

```python
    sections = {key: [] for key in section_headers}
    current_section, prev_section = None, None
    option, description = None, []

    for line in output.split("\n"):
        cleaned_line = line.strip()

        if cleaned_line.casefold() in map(str.casefold, section_headers):
            if prev_section is not None:  # Save last option from prev section
                add_info(sections, current_section, option, description)
                option, description = None, []
```

- EN: Declares or implements routines including `add_info`. Notable symbols here include `add_info`.
- CN: 这里声明或实现函数，例如 `add_info`。这里较值得关注的符号包括 `add_info`。

### Lines 52-59

```python
            cleaned_line = cleaned_line.split()
            # Apply lowercase to all words except the first one
            cleaned_line = [cleaned_line[0]] + [
                word.lower() for word in cleaned_line[1:]
            ]
            # Join the words back together into a string
            cleaned_line = " ".join(cleaned_line)
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 60-70

```python
            current_section = cleaned_line
            prev_section = current_section
            continue

        if cleaned_line.startswith("-"):
            if option and description:
                # Join description lines, adding an extra newline for
                # sub-options that start with '='
                add_info(sections, current_section, option, description)
                option, description = None, []
```

- EN: Declares or implements routines including `add_info`. Comments in this range record assumptions, invariants, or usage notes. Notable symbols here include `add_info`.
- CN: 这里声明或实现函数，例如 `add_info`。这里的注释记录了设计假设、不变量或使用说明。这里较值得关注的符号包括 `add_info`。

### Lines 71-82

```python
            parts = cleaned_line.split("  ", 1)
            if len(parts) > 1:
                option = parts[0].strip()
                descr = parts[1].strip()
                descr = descr[2].upper() + descr[3:]
                description = [descr]
                if option.startswith("--print") or option.startswith("--time"):
                    current_section = "BOLT printing options:"
                elif prev_section is not None:
                    current_section = prev_section
            continue
```

- EN: Contains local control flow that updates state or selects among execution branches.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。

### Lines 83-92

```python
        if cleaned_line.startswith("="):
            parts = cleaned_line.split(maxsplit=1)
            # Split into two parts: sub-option and description
            if len(parts) == 2:
                # Rejoin with a single space
                cleaned_line = parts[0] + " " + parts[1].rstrip()
            description.append(cleaned_line)
        elif cleaned_line:  # Multiline description continuation
            description.append(cleaned_line)
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 93-105

```python
    add_info(sections, current_section, option, description)
    return sections


def generate_markdown(sections):
    markdown_lines = [
        "# BOLT - a post-link optimizer developed to speed up large applications\n",
        "## SYNOPSIS\n",
        "`llvm-bolt <executable> [-o outputfile] <executable>.bolt "
        "[-data=perf.fdata] [options]`\n",
        "## OPTIONS",
    ]
```

- EN: Declares or implements routines including `add_info`. Notable symbols here include `add_info`.
- CN: 这里声明或实现函数，例如 `add_info`。这里较值得关注的符号包括 `add_info`。

### Lines 106-123

```python
    for section, options in sections.items():
        markdown_lines.append(f"\n### {section}")
        if section == "BOLT instrumentation options:":
            markdown_lines.append(
                f"\n`llvm-bolt <executable> -instrument"
                " [-o outputfile] <instrumented-executable>`"
            )
        for option, desc in options:
            markdown_lines.append(f"\n- `{option}`\n")
            # Split description into lines to handle sub-options
            desc_lines = desc.split("\n")
            for line in desc_lines:
                if line.startswith("="):
                    # Sub-option: correct formatting with bullet
                    sub_option, sub_desc = line[1:].split(" ", 1)
                    markdown_lines.append(f"  - `{sub_option}`: {sub_desc[4:]}")
                else:
                    # Regular line of description
```

- EN: Contains local control flow that updates state or selects among execution branches. Comments in this range record assumptions, invariants, or usage notes.
- CN: 这里包含局部控制流，用于更新状态或在不同执行分支间选择。这里的注释记录了设计假设、不变量或使用说明。

### Lines 124-140

```python
                    if line[2:].startswith("<"):
                        line = line.replace("<", "").replace(">", "")
                    markdown_lines.append(f"{line}")

    return "\n".join(markdown_lines)


def main():
    try:
        help_output = subprocess.run(
            ["llvm-bolt", "--help-hidden"], capture_output=True, text=True, check=True
        ).stdout
    except subprocess.CalledProcessError as e:
        print("Failed to execute llvm-bolt --help:")
        print(e)
        return
```

- EN: Declares or implements routines including `print`. Notable symbols here include `print`.
- CN: 这里声明或实现函数，例如 `print`。这里较值得关注的符号包括 `print`。

### Lines 141-149

```python
    sections = parse_bolt_options(help_output)
    markdown = generate_markdown(sections)

    with open("CommandLineArgumentReference.md", "w") as md_file:
        md_file.write(markdown)


if __name__ == "__main__":
    main()
```

- EN: Declares or implements routines including `parse_bolt_options`, `generate_markdown`, `main`. Notable symbols here include `parse_bolt_options`, `generate_markdown`, `main`.
- CN: 这里声明或实现函数，例如 `parse_bolt_options`, `generate_markdown`, `main`。这里较值得关注的符号包括 `parse_bolt_options`, `generate_markdown`, `main`。

## Key Concepts / 关键概念

- `wrap`: function or method entry point / 函数或方法入口
- `wrap_text`: function or method entry point / 函数或方法入口
- `add_info`: function or method entry point / 函数或方法入口
- `print`: function or method entry point / 函数或方法入口
- `parse_bolt_options`: function or method entry point / 函数或方法入口
- `subprocess`: imported dependency / 导入的依赖
- `from textwrap import wrap`: imported dependency / 导入的依赖

## Dependencies / 依赖关系

- Python imports / Python 导入: `subprocess`, `from textwrap import wrap`
- Directory context / 目录上下文: `bolt/docs` neighbors usually cooperate with this file to provide the surrounding subsystem / `bolt/docs` 下的相邻文件通常与本文件协作组成对应子系统
