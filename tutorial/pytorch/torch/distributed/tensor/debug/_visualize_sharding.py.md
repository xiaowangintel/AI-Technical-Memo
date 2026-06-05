# _visualize_sharding.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/debug/_visualize_sharding.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include _create_table, make_color_iter.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 _create_table, make_color_iter。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
import importlib.util

import numpy as np

from torch._prims_common import ShapeType
from torch.distributed.tensor._utils import _compute_local_shape_and_global_offset


__all__ = ["visualize_sharding"]

Color = tuple[float, float, float]


def _create_table(
    shards: list[tuple[tuple[int, int], tuple[int, int], int]], device_kind: str = ""
):
    """
    Creates a tabulate table given row and column ranges with device name
    """
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Imports module dependencies: `importlib.util`. | CN: 导入模块依赖：`importlib.util`。
- **L3** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L4** EN: Imports module dependencies: `numpy as np`. | CN: 导入模块依赖：`numpy as np`。
- **L5** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L6** EN: Imports selected names from `torch._prims_common`. | CN: 从 `torch._prims_common` 导入指定名称。
- **L7** EN: Imports selected names from `torch.distributed.tensor._utils`. | CN: 从 `torch.distributed.tensor._utils` 导入指定名称。
- **L8** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L9** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L10** EN: Assigns or updates `__all__`. | CN: 对 `__all__` 进行赋值或更新。
- **L11** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L12** EN: Assigns or updates `Color`. | CN: 对 `Color` 进行赋值或更新。
- **L13** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L14** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L15** EN: Defines function `_create_table`. | CN: 定义函数 `_create_table`。
- **L16** EN: Assigns or updates `shards`. | CN: 对 `shards` 进行赋值或更新。
- **L17** EN: Continues the implementation inside function `_create_table`. | CN: 继续说明函数 `_create_table` 内部的实现。
- **L18** EN: Starts the docstring for the function _create_table. | CN: 开始定义 function _create_table 的文档字符串。
- **L19** EN: Continues the docstring text for the function _create_table. | CN: 继续补充 function _create_table 的文档字符串内容。
- **L20** EN: Closes the docstring for the function _create_table. | CN: 结束 function _create_table 的文档字符串。

### Lines 21-40 / 第 21-40 行

````python
    from tabulate import tabulate

    # Extract unique row and column ranges
    row_ranges = sorted({block[0] for block in shards})
    col_ranges = sorted({block[1] for block in shards})

    # Create a matrix initialized with empty strings
    matrix = [["" for _ in col_ranges] for _ in row_ranges]

    # Fill the matrix with values
    for block in shards:
        row_index = row_ranges.index(block[0])
        col_index = col_ranges.index(block[1])
        if matrix[row_index][col_index] == "":
            matrix[row_index][col_index] = device_kind + ":" + str(block[2])
        else:
            matrix[row_index][col_index] += "," + str(block[2])

    # Prepare headers
    row_headers = [f"Row {r[0]}-{r[1]}" for r in row_ranges]
````

- **L21** EN: Imports selected names from `tabulate`. | CN: 从 `tabulate` 导入指定名称。
- **L22** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L23** EN: Keeps the inline comment or directive: Extract unique row and column ranges | CN: 保留这一行注释或指令：Extract unique row and column ranges
- **L24** EN: Assigns or updates `row_ranges`. | CN: 对 `row_ranges` 进行赋值或更新。
- **L25** EN: Assigns or updates `col_ranges`. | CN: 对 `col_ranges` 进行赋值或更新。
- **L26** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L27** EN: Keeps the inline comment or directive: Create a matrix initialized with empty strings | CN: 保留这一行注释或指令：Create a matrix initialized with empty strings
- **L28** EN: Assigns or updates `matrix`. | CN: 对 `matrix` 进行赋值或更新。
- **L29** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L30** EN: Keeps the inline comment or directive: Fill the matrix with values | CN: 保留这一行注释或指令：Fill the matrix with values
- **L31** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L32** EN: Assigns or updates `row_index`. | CN: 对 `row_index` 进行赋值或更新。
- **L33** EN: Assigns or updates `col_index`. | CN: 对 `col_index` 进行赋值或更新。
- **L34** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L35** EN: Assigns or updates `matrix[row_index][col_index]`. | CN: 对 `matrix[row_index][col_index]` 进行赋值或更新。
- **L36** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L37** EN: Continues the implementation inside function `_create_table`. | CN: 继续说明函数 `_create_table` 内部的实现。
- **L38** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L39** EN: Keeps the inline comment or directive: Prepare headers | CN: 保留这一行注释或指令：Prepare headers
- **L40** EN: Assigns or updates `row_headers`. | CN: 对 `row_headers` 进行赋值或更新。

### Lines 41-60 / 第 41-60 行

````python
    col_headers = [f"Col {c[0]}-{c[1]}" for c in col_ranges]

    return tabulate(matrix, headers=col_headers, showindex=row_headers)


def make_color_iter(color_map, num_rows, num_cols):
    num_colors = num_rows * num_cols
    for idx in range(num_colors):
        yield color_map(idx)


def _canonicalize_color(color: Color) -> str:
    if isinstance(color, str):
        return color
    r, g, b = (int(a * 255) for a in color)
    return f"#{r:02X}{g:02X}{b:02X}"


def _get_text_color(color: str) -> str:
    r, g, b = map(lambda x: int(x, 16), (color[1:3], color[3:5], color[5:7]))  # noqa: C417
````

- **L41** EN: Assigns or updates `col_headers`. | CN: 对 `col_headers` 进行赋值或更新。
- **L42** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L43** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L44** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L45** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L46** EN: Defines function `make_color_iter`. | CN: 定义函数 `make_color_iter`。
- **L47** EN: Assigns or updates `num_colors`. | CN: 对 `num_colors` 进行赋值或更新。
- **L48** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L49** EN: Yields a value from a generator-style callable. | CN: 从生成器风格的可调用对象中产出一个值。
- **L50** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L51** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L52** EN: Defines function `_canonicalize_color`. | CN: 定义函数 `_canonicalize_color`。
- **L53** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L54** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L55** EN: Assigns or updates `r, g, b`. | CN: 对 `r, g, b` 进行赋值或更新。
- **L56** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L57** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L58** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L59** EN: Defines function `_get_text_color`. | CN: 定义函数 `_get_text_color`。
- **L60** EN: Assigns or updates `r, g, b`. | CN: 对 `r, g, b` 进行赋值或更新。

### Lines 61-80 / 第 61-80 行

````python
    if (r * 0.299 + g * 0.587 + b * 0.114) > 186:
        return "#000000"
    return "#ffffff"


def _create_rich_table(
    shape: ShapeType,
    shards: list[tuple[tuple[int, int], tuple[int, int], int]],
    device_kind: str = "",
    scale: float = 1.0,
    min_width: int = 9,
    max_width: int = 80,
):
    import matplotlib
    import rich.align
    import rich.box
    import rich.console
    import rich.padding
    import rich.style
    import rich.table
````

- **L61** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L62** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L63** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L64** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L65** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L66** EN: Defines function `_create_rich_table`. | CN: 定义函数 `_create_rich_table`。
- **L67** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L68** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L69** EN: Assigns or updates `device_kind`. | CN: 对 `device_kind` 进行赋值或更新。
- **L70** EN: Assigns or updates `scale`. | CN: 对 `scale` 进行赋值或更新。
- **L71** EN: Assigns or updates `min_width`. | CN: 对 `min_width` 进行赋值或更新。
- **L72** EN: Assigns or updates `max_width`. | CN: 对 `max_width` 进行赋值或更新。
- **L73** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L74** EN: Imports module dependencies: `matplotlib`. | CN: 导入模块依赖：`matplotlib`。
- **L75** EN: Imports module dependencies: `rich.align`. | CN: 导入模块依赖：`rich.align`。
- **L76** EN: Imports module dependencies: `rich.box`. | CN: 导入模块依赖：`rich.box`。
- **L77** EN: Imports module dependencies: `rich.console`. | CN: 导入模块依赖：`rich.console`。
- **L78** EN: Imports module dependencies: `rich.padding`. | CN: 导入模块依赖：`rich.padding`。
- **L79** EN: Imports module dependencies: `rich.style`. | CN: 导入模块依赖：`rich.style`。
- **L80** EN: Imports module dependencies: `rich.table`. | CN: 导入模块依赖：`rich.table`。

### Lines 81-100 / 第 81-100 行

````python

    dtensor_height = shape[0]
    dtensor_width = shape[1] if len(shape) == 2 else 1

    row_ranges = sorted({s[0] for s in shards})
    col_ranges = sorted({s[1] for s in shards})
    num_rows, num_cols = len(row_ranges), len(col_ranges)

    console = rich.console.Console(width=max_width)
    use_color = console.color_system
    color_iter = make_color_iter(matplotlib.colormaps["tab20b"], num_rows, num_cols)

    base_height = int(10 * scale)
    aspect_ratio = (shape[1] if len(shape) == 2 else 1) / shape[0]
    base_width = int(base_height * aspect_ratio)
    height_to_width_ratio = 2.5

    table = rich.table.Table(
        show_header=False,
        show_lines=not use_color,
````

- **L81** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L82** EN: Assigns or updates `dtensor_height`. | CN: 对 `dtensor_height` 进行赋值或更新。
- **L83** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L84** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L85** EN: Assigns or updates `row_ranges`. | CN: 对 `row_ranges` 进行赋值或更新。
- **L86** EN: Assigns or updates `col_ranges`. | CN: 对 `col_ranges` 进行赋值或更新。
- **L87** EN: Assigns or updates `num_rows, num_cols`. | CN: 对 `num_rows, num_cols` 进行赋值或更新。
- **L88** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L89** EN: Assigns or updates `console`. | CN: 对 `console` 进行赋值或更新。
- **L90** EN: Assigns or updates `use_color`. | CN: 对 `use_color` 进行赋值或更新。
- **L91** EN: Assigns or updates `color_iter`. | CN: 对 `color_iter` 进行赋值或更新。
- **L92** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L93** EN: Assigns or updates `base_height`. | CN: 对 `base_height` 进行赋值或更新。
- **L94** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L95** EN: Assigns or updates `base_width`. | CN: 对 `base_width` 进行赋值或更新。
- **L96** EN: Assigns or updates `height_to_width_ratio`. | CN: 对 `height_to_width_ratio` 进行赋值或更新。
- **L97** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L98** EN: Assigns or updates `table`. | CN: 对 `table` 进行赋值或更新。
- **L99** EN: Assigns or updates `show_header`. | CN: 对 `show_header` 进行赋值或更新。
- **L100** EN: Assigns or updates `show_lines`. | CN: 对 `show_lines` 进行赋值或更新。

### Lines 101-120 / 第 101-120 行

````python
        padding=0,
        highlight=not use_color,
        pad_edge=False,
        box=rich.box.SQUARE if not use_color else None,
    )
    for row in range(num_rows):
        table_row = []
        for col in range(num_cols):
            entry = (
                device_kind
                + ":"
                + ",".join(
                    [
                        str(device_id)
                        for row_range, col_range, device_id in shards
                        if row_range == row_ranges[row] and col_range == col_ranges[col]
                    ]
                )
            )
            width = (col_ranges[col][1] - col_ranges[col][0]) / dtensor_width
````

- **L101** EN: Assigns or updates `padding`. | CN: 对 `padding` 进行赋值或更新。
- **L102** EN: Assigns or updates `highlight`. | CN: 对 `highlight` 进行赋值或更新。
- **L103** EN: Assigns or updates `pad_edge`. | CN: 对 `pad_edge` 进行赋值或更新。
- **L104** EN: Assigns or updates `box`. | CN: 对 `box` 进行赋值或更新。
- **L105** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L106** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L107** EN: Assigns or updates `table_row`. | CN: 对 `table_row` 进行赋值或更新。
- **L108** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L109** EN: Assigns or updates `entry`. | CN: 对 `entry` 进行赋值或更新。
- **L110** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L111** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L112** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L113** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L114** EN: Calls `str` as part of the current workflow. | CN: 在当前流程中调用 `str`。
- **L115** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L116** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L117** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L118** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L119** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L120** EN: Assigns or updates `width`. | CN: 对 `width` 进行赋值或更新。

### Lines 121-140 / 第 121-140 行

````python
            width = int(width * base_width * height_to_width_ratio)
            height = (row_ranges[row][1] - row_ranges[row][0]) / dtensor_height
            height = int(height * base_height)
            left_padding, remainder = divmod(width - len(entry) - 2, 2)
            right_padding = left_padding + remainder
            top_padding, remainder = divmod(height - 2, 2)
            bottom_padding = top_padding + remainder
            if use_color:
                color = _canonicalize_color(next(color_iter)[:3])
                text_color = _get_text_color(color)
                top_padding += 1
                bottom_padding += 1
                left_padding += 1
                right_padding += 1
            else:
                color = None
                text_color = None
            padding = (
                max(top_padding, 0),
                max(right_padding, 0),
````

- **L121** EN: Assigns or updates `width`. | CN: 对 `width` 进行赋值或更新。
- **L122** EN: Assigns or updates `height`. | CN: 对 `height` 进行赋值或更新。
- **L123** EN: Assigns or updates `height`. | CN: 对 `height` 进行赋值或更新。
- **L124** EN: Assigns or updates `left_padding, remainder`. | CN: 对 `left_padding, remainder` 进行赋值或更新。
- **L125** EN: Assigns or updates `right_padding`. | CN: 对 `right_padding` 进行赋值或更新。
- **L126** EN: Assigns or updates `top_padding, remainder`. | CN: 对 `top_padding, remainder` 进行赋值或更新。
- **L127** EN: Assigns or updates `bottom_padding`. | CN: 对 `bottom_padding` 进行赋值或更新。
- **L128** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L129** EN: Assigns or updates `color`. | CN: 对 `color` 进行赋值或更新。
- **L130** EN: Assigns or updates `text_color`. | CN: 对 `text_color` 进行赋值或更新。
- **L131** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L132** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L133** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L134** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L135** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L136** EN: Assigns or updates `color`. | CN: 对 `color` 进行赋值或更新。
- **L137** EN: Assigns or updates `text_color`. | CN: 对 `text_color` 进行赋值或更新。
- **L138** EN: Assigns or updates `padding`. | CN: 对 `padding` 进行赋值或更新。
- **L139** EN: Calls `max` as part of the current workflow. | CN: 在当前流程中调用 `max`。
- **L140** EN: Calls `max` as part of the current workflow. | CN: 在当前流程中调用 `max`。

### Lines 141-160 / 第 141-160 行

````python
                max(bottom_padding, 0),
                max(left_padding, 0),
            )
            table_row.append(
                rich.padding.Padding(
                    rich.align.Align(entry, "center", vertical="middle"),
                    padding,
                    style=rich.style.Style(bgcolor=color, color=text_color),
                )
            )
        table.add_row(*table_row)
    console.print(table, end="\n\n")


def visualize_sharding(dtensor, header="", use_rich: bool = False):
    """
    Visualizes sharding in the terminal for :class:`DTensor` that are 1D or 2D.

    .. note:: This requires the ``tabulate`` package, or ``rich`` and ``matplotlib``.
              No sharding info will be printed for empty tensors
````

- **L141** EN: Calls `max` as part of the current workflow. | CN: 在当前流程中调用 `max`。
- **L142** EN: Calls `max` as part of the current workflow. | CN: 在当前流程中调用 `max`。
- **L143** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L144** EN: Calls `table_row.append` as part of the current workflow. | CN: 在当前流程中调用 `table_row.append`。
- **L145** EN: Calls `rich.padding.Padding` as part of the current workflow. | CN: 在当前流程中调用 `rich.padding.Padding`。
- **L146** EN: Calls `rich.align.Align` as part of the current workflow. | CN: 在当前流程中调用 `rich.align.Align`。
- **L147** EN: Continues the implementation inside function `_create_rich_table`. | CN: 继续说明函数 `_create_rich_table` 内部的实现。
- **L148** EN: Assigns or updates `style`. | CN: 对 `style` 进行赋值或更新。
- **L149** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L150** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L151** EN: Calls `table.add_row` as part of the current workflow. | CN: 在当前流程中调用 `table.add_row`。
- **L152** EN: Calls `console.print` as part of the current workflow. | CN: 在当前流程中调用 `console.print`。
- **L153** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L154** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L155** EN: Defines function `visualize_sharding`. | CN: 定义函数 `visualize_sharding`。
- **L156** EN: Starts the docstring for the function visualize_sharding. | CN: 开始定义 function visualize_sharding 的文档字符串。
- **L157** EN: Continues the docstring text for the function visualize_sharding. | CN: 继续补充 function visualize_sharding 的文档字符串内容。
- **L158** EN: Continues the docstring text for the function visualize_sharding. | CN: 继续补充 function visualize_sharding 的文档字符串内容。
- **L159** EN: Continues the docstring text for the function visualize_sharding. | CN: 继续补充 function visualize_sharding 的文档字符串内容。
- **L160** EN: Continues the docstring text for the function visualize_sharding. | CN: 继续补充 function visualize_sharding 的文档字符串内容。

### Lines 161-180 / 第 161-180 行

````python
    """
    if dtensor.numel() == 0:  # Do not print empty dtensors.
        return

    if len(dtensor.shape) >= 3:
        raise RuntimeError("visualize sharding supports only 1D or 2D DTensor")

    if dtensor.device_mesh.get_coordinate() is None:  # current rank is not in the mesh
        return

    # Only display the visualization once for each DTensor, on the rank whose
    # coordinate is 0 on all dimensions. For example, if the mesh is a full mesh,
    # we will only print on rank 0.
    local_rank_zero_on_all_dim = all(
        dtensor.device_mesh.get_local_rank(mesh_dim=dim) == 0
        for dim in range(dtensor.device_mesh.ndim)
    )
    if not local_rank_zero_on_all_dim:
        return

````

- **L161** EN: Closes the docstring for the function visualize_sharding. | CN: 结束 function visualize_sharding 的文档字符串。
- **L162** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L163** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L164** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L165** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L166** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L167** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L168** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L169** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L170** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L171** EN: Keeps the inline comment or directive: Only display the visualization once for each DTensor, on the rank whose | CN: 保留这一行注释或指令：Only display the visualization once for each DTensor, on the rank whose
- **L172** EN: Keeps the inline comment or directive: coordinate is 0 on all dimensions. For example, if the mesh is a full mesh, | CN: 保留这一行注释或指令：coordinate is 0 on all dimensions. For example, if the mesh is a full mesh,
- **L173** EN: Keeps the inline comment or directive: we will only print on rank 0. | CN: 保留这一行注释或指令：we will only print on rank 0.
- **L174** EN: Assigns or updates `local_rank_zero_on_all_dim`. | CN: 对 `local_rank_zero_on_all_dim` 进行赋值或更新。
- **L175** EN: Calls `dtensor.device_mesh.get_local_rank` as part of the current workflow. | CN: 在当前流程中调用 `dtensor.device_mesh.get_local_rank`。
- **L176** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L177** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L178** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L179** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L180** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 181-200 / 第 181-200 行

````python
    device_coords = {
        int(device_index.item()): list(coord)
        for coord, device_index in np.ndenumerate(
            np.array(dtensor.device_mesh.mesh.tolist())
        )
    }

    device_shard_shape_and_offsets = {
        device_index: _compute_local_shape_and_global_offset(
            dtensor.shape,
            dtensor.device_mesh.shape,
            lambda i: device_coords[device_index][i],
            dtensor.placements,
        )
        for device_index in device_coords
    }

    # Extend shards in a 1D tensor to 2D
    device_shard_shape_and_offsets = {
        device_index: (
````

- **L181** EN: Assigns or updates `device_coords`. | CN: 对 `device_coords` 进行赋值或更新。
- **L182** EN: Calls `int` as part of the current workflow. | CN: 在当前流程中调用 `int`。
- **L183** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L184** EN: Calls `np.array` as part of the current workflow. | CN: 在当前流程中调用 `np.array`。
- **L185** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L186** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L187** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L188** EN: Assigns or updates `device_shard_shape_and_offsets`. | CN: 对 `device_shard_shape_and_offsets` 进行赋值或更新。
- **L189** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L190** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L191** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L192** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L193** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L194** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L195** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L196** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L197** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L198** EN: Keeps the inline comment or directive: Extend shards in a 1D tensor to 2D | CN: 保留这一行注释或指令：Extend shards in a 1D tensor to 2D
- **L199** EN: Assigns or updates `device_shard_shape_and_offsets`. | CN: 对 `device_shard_shape_and_offsets` 进行赋值或更新。
- **L200** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。

### Lines 201-220 / 第 201-220 行

````python
            shape if len(shape) == 2 else (shape[0], 1),
            offset if len(offset) == 2 else (offset[0], 0),
        )
        for device_index, (shape, offset) in device_shard_shape_and_offsets.items()
    }

    shards = [
        (
            (offset[0], offset[0] + shape[0] - 1),
            (offset[1], offset[1] + shape[1] - 1),
            device_index,
        )
        for device_index, (shape, offset) in device_shard_shape_and_offsets.items()
    ]

    if (
        importlib.util.find_spec("rich")
        and importlib.util.find_spec("matplotlib")
        and use_rich
    ):
````

- **L201** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L202** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L203** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L204** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L205** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L206** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L207** EN: Assigns or updates `shards`. | CN: 对 `shards` 进行赋值或更新。
- **L208** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L209** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L210** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L211** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L212** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L213** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L214** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L215** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L216** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L217** EN: Calls `importlib.util.find_spec` as part of the current workflow. | CN: 在当前流程中调用 `importlib.util.find_spec`。
- **L218** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L219** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。
- **L220** EN: Continues the implementation inside function `visualize_sharding`. | CN: 继续说明函数 `visualize_sharding` 内部的实现。

### Lines 221-227 / 第 221-227 行

````python
        _create_rich_table(
            dtensor.shape, shards, device_kind=dtensor.device_mesh.device_type
        )
    elif importlib.util.find_spec("tabulate"):
        print(_create_table(shards, device_kind=dtensor.device_mesh.device_type))
    else:
        raise ValueError("`visualize_sharding` requires either `rich` or `tabulate`.")
````

- **L221** EN: Calls `_create_rich_table` as part of the current workflow. | CN: 在当前流程中调用 `_create_rich_table`。
- **L222** EN: Assigns or updates `dtensor.shape, shards, device_kind`. | CN: 对 `dtensor.shape, shards, device_kind` 进行赋值或更新。
- **L223** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L224** EN: Adds an alternate conditional branch. | CN: 补充一个备用条件分支。
- **L225** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L226** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L227** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: device mesh  
  **CN**: 设备网格
- **EN**: sharding  
  **CN**: 分片
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: placements  
  **CN**: 放置规则
- **EN**: Core callables: _create_table, make_color_iter, _canonicalize_color, _get_text_color, _create_rich_table  
  **CN**: 核心可调用对象：_create_table, make_color_iter, _canonicalize_color, _get_text_color, _create_rich_table

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.tensor._utils`
- **PyTorch / PyTorch**: `torch._prims_common`
- **Python Stdlib / Python 标准库**: `importlib.util`
- **Third-party / 第三方**: `matplotlib`, `numpy`, `rich.align`, `rich.box`, `rich.console`, `rich.padding`, `rich.style`, `rich.table`, `tabulate`

