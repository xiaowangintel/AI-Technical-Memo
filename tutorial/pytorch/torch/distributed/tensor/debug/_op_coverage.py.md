# _op_coverage.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/tensor/debug/_op_coverage.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed tensor layouts, placements, and operators. Its main entry points include fwd_bwd_compiler, get_inductor_decomp_graphs.
- **用途 (CN)**: 该模块聚焦于分布式张量布局、放置与算子逻辑，其主要入口包括 fwd_bwd_compiler, get_inductor_decomp_graphs。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
# mypy: allow-untyped-defs
from operator import itemgetter

import torch
import torch.fx
import torch.nn as nn
from functorch.compile import make_boxed_func
from torch._functorch.compilers import aot_module
from torch._inductor.decomposition import select_decomp_table
from torch.distributed.tensor import DTensor


inductor_decomps = select_decomp_table()

graphs: list[torch.fx.GraphModule] = []


def fwd_bwd_compiler(fx_g, _):
    graphs.append(fx_g)
    return make_boxed_func(fx_g)
````

- **L1** EN: Keeps the inline comment or directive: mypy: allow-untyped-defs | CN: 保留这一行注释或指令：mypy: allow-untyped-defs
- **L2** EN: Imports selected names from `operator`. | CN: 从 `operator` 导入指定名称。
- **L3** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L4** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L5** EN: Imports module dependencies: `torch.fx`. | CN: 导入模块依赖：`torch.fx`。
- **L6** EN: Imports module dependencies: `torch.nn as nn`. | CN: 导入模块依赖：`torch.nn as nn`。
- **L7** EN: Imports selected names from `functorch.compile`. | CN: 从 `functorch.compile` 导入指定名称。
- **L8** EN: Imports selected names from `torch._functorch.compilers`. | CN: 从 `torch._functorch.compilers` 导入指定名称。
- **L9** EN: Imports selected names from `torch._inductor.decomposition`. | CN: 从 `torch._inductor.decomposition` 导入指定名称。
- **L10** EN: Imports selected names from `torch.distributed.tensor`. | CN: 从 `torch.distributed.tensor` 导入指定名称。
- **L11** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L12** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L13** EN: Assigns or updates `inductor_decomps`. | CN: 对 `inductor_decomps` 进行赋值或更新。
- **L14** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L15** EN: Assigns or updates `graphs`. | CN: 对 `graphs` 进行赋值或更新。
- **L16** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L17** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L18** EN: Defines function `fwd_bwd_compiler`. | CN: 定义函数 `fwd_bwd_compiler`。
- **L19** EN: Calls `graphs.append` as part of the current workflow. | CN: 在当前流程中调用 `graphs.append`。
- **L20** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 21-40 / 第 21-40 行

````python


def get_inductor_decomp_graphs(model: nn.Module, args, kwargs):
    """
    Obtain forward and backward graphs of a model with inductor decompositions using tracing and aot_module.

    Convenient util to get the fwd and bwd graphs of an arbitrary model
    with inductor decompositions. Note that this would simply do tracing
    with aot_module and don't ensure correctness. This is useful to track
    the ops needed in DTensor.
    """
    compiled_mod = aot_module(
        model, fw_compiler=fwd_bwd_compiler, decompositions=inductor_decomps
    )
    output = compiled_mod(*args, **kwargs)

    if output.ndim != 0:
        # if output is not a scalar tensor, by default sum it in order to
        # run backward
        output = output.sum()
````

- **L21** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L22** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L23** EN: Defines function `get_inductor_decomp_graphs`. | CN: 定义函数 `get_inductor_decomp_graphs`。
- **L24** EN: Starts the docstring for the function get_inductor_decomp_graphs. | CN: 开始定义 function get_inductor_decomp_graphs 的文档字符串。
- **L25** EN: Continues the docstring text for the function get_inductor_decomp_graphs. | CN: 继续补充 function get_inductor_decomp_graphs 的文档字符串内容。
- **L26** EN: Continues the docstring text for the function get_inductor_decomp_graphs. | CN: 继续补充 function get_inductor_decomp_graphs 的文档字符串内容。
- **L27** EN: Continues the docstring text for the function get_inductor_decomp_graphs. | CN: 继续补充 function get_inductor_decomp_graphs 的文档字符串内容。
- **L28** EN: Continues the docstring text for the function get_inductor_decomp_graphs. | CN: 继续补充 function get_inductor_decomp_graphs 的文档字符串内容。
- **L29** EN: Continues the docstring text for the function get_inductor_decomp_graphs. | CN: 继续补充 function get_inductor_decomp_graphs 的文档字符串内容。
- **L30** EN: Continues the docstring text for the function get_inductor_decomp_graphs. | CN: 继续补充 function get_inductor_decomp_graphs 的文档字符串内容。
- **L31** EN: Closes the docstring for the function get_inductor_decomp_graphs. | CN: 结束 function get_inductor_decomp_graphs 的文档字符串。
- **L32** EN: Assigns or updates `compiled_mod`. | CN: 对 `compiled_mod` 进行赋值或更新。
- **L33** EN: Assigns or updates `model, fw_compiler`. | CN: 对 `model, fw_compiler` 进行赋值或更新。
- **L34** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L35** EN: Assigns or updates `output`. | CN: 对 `output` 进行赋值或更新。
- **L36** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L37** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L38** EN: Keeps the inline comment or directive: if output is not a scalar tensor, by default sum it in order to | CN: 保留这一行注释或指令：if output is not a scalar tensor, by default sum it in order to
- **L39** EN: Keeps the inline comment or directive: run backward | CN: 保留这一行注释或指令：run backward
- **L40** EN: Assigns or updates `output`. | CN: 对 `output` 进行赋值或更新。

### Lines 41-60 / 第 41-60 行

````python

    output.backward()

    # one fwd, one bwd graph
    if len(graphs) != 2:
        raise AssertionError
    return graphs


def print_op_coverage_summary(model: nn.Module, args, kwargs, *, output_csv=False):
    """
    Util to print the operator coverage summary of a certain model with tabulute.

    Must have tabulate module installed.
    """
    # python module required for summary
    import csv

    from tabulate import tabulate

````

- **L41** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L42** EN: Calls `output.backward` as part of the current workflow. | CN: 在当前流程中调用 `output.backward`。
- **L43** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L44** EN: Keeps the inline comment or directive: one fwd, one bwd graph | CN: 保留这一行注释或指令：one fwd, one bwd graph
- **L45** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L46** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L47** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L48** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L49** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L50** EN: Defines function `print_op_coverage_summary`. | CN: 定义函数 `print_op_coverage_summary`。
- **L51** EN: Starts the docstring for the function print_op_coverage_summary. | CN: 开始定义 function print_op_coverage_summary 的文档字符串。
- **L52** EN: Continues the docstring text for the function print_op_coverage_summary. | CN: 继续补充 function print_op_coverage_summary 的文档字符串内容。
- **L53** EN: Continues the docstring text for the function print_op_coverage_summary. | CN: 继续补充 function print_op_coverage_summary 的文档字符串内容。
- **L54** EN: Continues the docstring text for the function print_op_coverage_summary. | CN: 继续补充 function print_op_coverage_summary 的文档字符串内容。
- **L55** EN: Closes the docstring for the function print_op_coverage_summary. | CN: 结束 function print_op_coverage_summary 的文档字符串。
- **L56** EN: Keeps the inline comment or directive: python module required for summary | CN: 保留这一行注释或指令：python module required for summary
- **L57** EN: Imports module dependencies: `csv`. | CN: 导入模块依赖：`csv`。
- **L58** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L59** EN: Imports selected names from `tabulate`. | CN: 从 `tabulate` 导入指定名称。
- **L60** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 61-80 / 第 61-80 行

````python
    fwd_graph, bwd_graph = get_inductor_decomp_graphs(model, args, kwargs)

    op_counts = {}

    for node in fwd_graph.graph.nodes:
        if node.op == "call_function" and isinstance(
            node.target, torch._ops.OpOverload
        ):
            if node.target not in op_counts:
                op_counts[node.target] = 0

            op_counts[node.target] += 1

    for node in bwd_graph.graph.nodes:
        if node.op == "call_function" and isinstance(
            node.target, torch._ops.OpOverload
        ):
            if node.target not in op_counts:
                op_counts[node.target] = 0

````

- **L61** EN: Assigns or updates `fwd_graph, bwd_graph`. | CN: 对 `fwd_graph, bwd_graph` 进行赋值或更新。
- **L62** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L63** EN: Assigns or updates `op_counts`. | CN: 对 `op_counts` 进行赋值或更新。
- **L64** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L65** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L66** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L67** EN: Continues the implementation inside function `print_op_coverage_summary`. | CN: 继续说明函数 `print_op_coverage_summary` 内部的实现。
- **L68** EN: Continues the implementation inside function `print_op_coverage_summary`. | CN: 继续说明函数 `print_op_coverage_summary` 内部的实现。
- **L69** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L70** EN: Assigns or updates `op_counts[node.target]`. | CN: 对 `op_counts[node.target]` 进行赋值或更新。
- **L71** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L72** EN: Continues the implementation inside function `print_op_coverage_summary`. | CN: 继续说明函数 `print_op_coverage_summary` 内部的实现。
- **L73** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L74** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L75** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L76** EN: Continues the implementation inside function `print_op_coverage_summary`. | CN: 继续说明函数 `print_op_coverage_summary` 内部的实现。
- **L77** EN: Continues the implementation inside function `print_op_coverage_summary`. | CN: 继续说明函数 `print_op_coverage_summary` 内部的实现。
- **L78** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L79** EN: Assigns or updates `op_counts[node.target]`. | CN: 对 `op_counts[node.target]` 进行赋值或更新。
- **L80** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 81-100 / 第 81-100 行

````python
            op_counts[node.target] += 1

    op_infos = []

    for op, count in op_counts.items():
        supported = op in DTensor._op_dispatcher.sharding_propagator.op_to_rules
        op_infos.append([op, str(op._schema), count, supported])

    # sort the op info base on the total count index
    count_idx = 2
    op_infos.sort(key=itemgetter(count_idx), reverse=True)

    headers = ["Operator", "Schema", "Total Count", "Supported"]
    print(tabulate(op_infos, headers=headers))

    if output_csv:
        # Open a CSV file for writing
        with open("op_summary.csv", "w", newline="") as csv_file:
            # Create a CSV writer object
            csv_writer = csv.writer(csv_file)
````

- **L81** EN: Continues the implementation inside function `print_op_coverage_summary`. | CN: 继续说明函数 `print_op_coverage_summary` 内部的实现。
- **L82** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L83** EN: Assigns or updates `op_infos`. | CN: 对 `op_infos` 进行赋值或更新。
- **L84** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L85** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L86** EN: Assigns or updates `supported`. | CN: 对 `supported` 进行赋值或更新。
- **L87** EN: Calls `op_infos.append` as part of the current workflow. | CN: 在当前流程中调用 `op_infos.append`。
- **L88** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L89** EN: Keeps the inline comment or directive: sort the op info base on the total count index | CN: 保留这一行注释或指令：sort the op info base on the total count index
- **L90** EN: Assigns or updates `count_idx`. | CN: 对 `count_idx` 进行赋值或更新。
- **L91** EN: Calls `op_infos.sort` as part of the current workflow. | CN: 在当前流程中调用 `op_infos.sort`。
- **L92** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L93** EN: Assigns or updates `headers`. | CN: 对 `headers` 进行赋值或更新。
- **L94** EN: Calls `print` as part of the current workflow. | CN: 在当前流程中调用 `print`。
- **L95** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L96** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L97** EN: Keeps the inline comment or directive: Open a CSV file for writing | CN: 保留这一行注释或指令：Open a CSV file for writing
- **L98** EN: Enters a context manager scope. | CN: 进入上下文管理器作用域。
- **L99** EN: Keeps the inline comment or directive: Create a CSV writer object | CN: 保留这一行注释或指令：Create a CSV writer object
- **L100** EN: Assigns or updates `csv_writer`. | CN: 对 `csv_writer` 进行赋值或更新。

### Lines 101-105 / 第 101-105 行

````python

            csv_writer.writerow(headers)
            # Write each table row to the CSV file
            for row in op_infos:
                csv_writer.writerow(row)
````

- **L101** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L102** EN: Calls `csv_writer.writerow` as part of the current workflow. | CN: 在当前流程中调用 `csv_writer.writerow`。
- **L103** EN: Keeps the inline comment or directive: Write each table row to the CSV file | CN: 保留这一行注释或指令：Write each table row to the CSV file
- **L104** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L105** EN: Calls `csv_writer.writerow` as part of the current workflow. | CN: 在当前流程中调用 `csv_writer.writerow`。

## Key Concepts / 关键概念

- **EN**: distributed tensor layouts, placements, and operators  
  **CN**: 分布式张量布局、放置与算子逻辑
- **EN**: sharding  
  **CN**: 分片
- **EN**: distributed tensors  
  **CN**: 分布式张量
- **EN**: Core callables: fwd_bwd_compiler, get_inductor_decomp_graphs, print_op_coverage_summary  
  **CN**: 核心可调用对象：fwd_bwd_compiler, get_inductor_decomp_graphs, print_op_coverage_summary

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed.tensor`
- **PyTorch / PyTorch**: `torch`, `torch._functorch.compilers`, `torch._inductor.decomposition`, `torch.fx`, `torch.nn`
- **Python Stdlib / Python 标准库**: `csv`, `operator`
- **Third-party / 第三方**: `functorch.compile`, `tabulate`

