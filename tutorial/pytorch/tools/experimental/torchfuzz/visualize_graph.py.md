# visualize_graph.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `tools/experimental/torchfuzz/visualize_graph.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements experimental fuzzing helpers used to stress PyTorch operators or APIs.
- **Purpose (CN)**: 实现实验性模糊测试辅助逻辑，用于压测 PyTorch 算子或 API。
## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```python
# mypy: ignore-errors

"""
Visualization tools for operation stacks and graphs as DAGs.
"""

import subprocess

from torchfuzz.ops_fuzzer import OperationGraph
from torchfuzz.tensor_fuzzer import TensorSpec
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as subprocess; external packages such as torchfuzz.ops_fuzzer, torchfuzz.tensor_fuzzer. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 subprocess；外部依赖包，如 torchfuzz.ops_fuzzer、torchfuzz.tensor_fuzzer。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 12-22
```python

def save_and_render_dot(dot_content: str, filename: str = "operation_stack"):
    """
    Save DOT content to file and render as PNG/PDF.

    Args:
        dot_content: DOT format string
        filename: Base filename (without extension)
    """
    import os
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as os. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `save_and_render_dot`, which generates derived source text, templates, or metadata outputs.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 os。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `save_and_render_dot`，其作用是生成派生源码文本、模板或元数据输出。

### Lines 23-32
```python
    dot_file = f"{filename}.dot"
    png_file = f"{filename}.png"

    # Get absolute path for clickable link
    abs_png = os.path.abspath(png_file)

    # Save DOT file
    with open(dot_file, "w") as f:
        f.write(dot_content)
```
- **EN**: This chunk continues `save_and_render_dot` and expands its internal control flow or data movement. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `save_and_render_dot`，进一步展开其内部控制流或数据流转。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 33-41
```python
    # Render to PNG
    try:
        subprocess.run(["dot", "-Tpng", dot_file, "-o", png_file], check=True)
        print(f"🖼️  View: file://{abs_png}")
    except (subprocess.CalledProcessError, FileNotFoundError):
        pass


def operation_graph_to_dot(
```
- **EN**: This chunk defines `operation_graph_to_dot`, which implements a focused step inside the torch fuzzing pipeline. Subprocess orchestration bridges this script to external build tools, linters, or system commands. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段定义了 `operation_graph_to_dot`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 子进程编排把该脚本连接到外部构建工具、lint 或系统命令。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 42-50
```python
    graph: OperationGraph, title: str = "Operation Graph"
) -> str:
    """
    Convert an operation graph to Graphviz DOT format for visualization.

    Args:
        graph: OperationGraph instance
        title: Title for the graph
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `operation_graph_to_dot` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `operation_graph_to_dot`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 51-62
```python
    Returns:
        DOT format string
    """
    dot_lines = [
        "digraph OperationGraph {",
        f'    label="{title}";',
        "    rankdir=TB;",  # Top to bottom layout
        "    node [shape=box, style=filled, fontsize=10];",
        "    edge [fontsize=8];",
        "",
    ]
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk continues `operation_graph_to_dot` and expands its internal control flow or data movement.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段延续了 `operation_graph_to_dot`，进一步展开其内部控制流或数据流转。

### Lines 63-78
```python
    # Add nodes with styling based on operation type
    for node_id, node in graph.nodes.items():
        # Choose color and shape based on operation type
        if node.op_name.startswith("arg_"):
            color = "lightblue"
            shape = "ellipse"
        elif node.op_name == "constant":
            color = "lightgreen"
            shape = "ellipse"
        elif "aten" in node.op_name:
            color = "lightyellow"
            shape = "box"
        else:
            color = "lightgray"
            shape = "box"
```
- **EN**: This chunk continues `operation_graph_to_dot` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `operation_graph_to_dot`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 79-88
```python
        # Create comprehensive label
        if node.op_name.startswith("arg_"):
            label_parts = [node.op_name]
        else:
            label_parts = [node_id, node.op_name, f"depth {node.depth}"]

        if hasattr(node.output_spec, "dtype"):
            dtype_str = str(node.output_spec.dtype).replace("torch.", "")
            label_parts.append(dtype_str)
```
- **EN**: This chunk continues `operation_graph_to_dot` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `operation_graph_to_dot`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 89-98
```python
        # Only add size for TensorSpec, not ScalarSpec
        if isinstance(node.output_spec, TensorSpec) and node.output_spec.size:
            size_str = "x".join(map(str, node.output_spec.size))
            label_parts.append(f"size {size_str}")

        label = "\\n".join(label_parts)

        # Special highlighting for root node
        extra_style = ""
        if node_id == graph.root_node_id:
```
- **EN**: This chunk continues `operation_graph_to_dot` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `operation_graph_to_dot`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 99-108
```python
            extra_style = ", penwidth=3, color=red"

        dot_lines.append(
            f'    {node_id} [label="{label}", fillcolor="{color}", shape="{shape}"{extra_style}];'
        )

    dot_lines.append("")

    # Add edges based on the graph structure
    for node_id, node in graph.nodes.items():
```
- **EN**: This chunk continues `operation_graph_to_dot` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `operation_graph_to_dot`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 109-126
```python
        for i, input_node_id in enumerate(node.input_nodes):
            # Add edge from input node to current node with input position label
            edge_label = f"input_{i}"
            dot_lines.append(
                f'    {input_node_id} -> {node_id} [label="{edge_label}"];'
            )

    dot_lines.extend(
        [
            "",
            "    // Legend",
            "    subgraph cluster_legend {",
            '        label="Legend";',
            "        style=filled;",
            "        fillcolor=white;",
            '        legend_arg [label="arg", fillcolor=lightblue, shape=ellipse];',
            '        legend_const [label="constant", fillcolor=lightgreen, shape=ellipse];',
            '        legend_aten [label="aten ops", fillcolor=lightyellow, shape=box];',
```
- **EN**: This chunk continues `operation_graph_to_dot` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `operation_graph_to_dot`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 127-135
```python
            '        legend_root [label="root", fillcolor=orange, shape=box, penwidth=3, color=red];',
            "    }",
            "}",
        ]
    )

    return "\n".join(dot_lines)
```
- **EN**: This chunk continues `operation_graph_to_dot` and expands its internal control flow or data movement. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段延续了 `operation_graph_to_dot`，进一步展开其内部控制流或数据流转。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 136-151
```python
def visualize_operation_graph(
    graph: OperationGraph,
    title: str = "Operation Graph",
    output_folder: str = ".",
):
    """
    Complete visualization pipeline for an operation graph.

    Args:
        graph: OperationGraph instance
        title: Title for the visualization
        output_folder: Folder where to save the visualization files
    """
    # Generate DOT content
    dot_content = operation_graph_to_dot(graph, title)
```
- **EN**: This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `visualize_operation_graph`, which implements a focused step inside the torch fuzzing pipeline. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `visualize_operation_graph`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 152-164
```python
    # Save and render in the specified folder
    import os

    filename = os.path.join(output_folder, "operation_graph")
    save_and_render_dot(dot_content, filename)


def operation_graph_to_networkx(graph: OperationGraph):
    """
    Convert operation graph to NetworkX graph for Python visualization.
    Requires: pip install networkx matplotlib
    """
    try:
```
- **EN**: This block assembles the Python-side dependencies, importing Python standard-library modules such as os. This docstring block explains the tool entry point, generation contract, or usage expectations before executable logic begins. This chunk defines `operation_graph_to_networkx`, which implements a focused step inside the torch fuzzing pipeline. Filesystem handling code normalizes paths and keeps generated or discovered artifacts in predictable locations. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics.
- **CN**: 这一段组织 Python 侧依赖，引入了Python 标准库模块，如 os。 这一段文档字符串在可执行逻辑开始前说明了工具入口、生成约定或使用方式。 这一段定义了 `operation_graph_to_networkx`，其作用是实现Torch 模糊测试流水线中的一个关键步骤。 文件系统处理逻辑负责规范化路径，并把生成或发现的产物放在可预测的位置。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。

### Lines 165-175
```python
        import matplotlib.pyplot as plt
        import networkx as nx
    except ImportError:
        print(
            "⚠️  NetworkX/Matplotlib not installed. Run: pip install networkx matplotlib"
        )
        return

    # Create directed graph
    G = nx.DiGraph()
```
- **EN**: This block assembles the Python-side dependencies, importing external packages such as matplotlib.pyplot, networkx. Configuration constants such as G centralize defaults so later functions share the same policy knobs. Exception-handling paths convert tool failures into clearer control flow or user-facing diagnostics. The tail returns computed state or forwards the updated result to the caller.
- **CN**: 这一段组织 Python 侧依赖，引入了外部依赖包，如 matplotlib.pyplot、networkx。 G 等配置常量集中定义默认值，使后续函数共享同一套策略开关。 异常处理路径把工具失败转换为更清晰的控制流或面向用户的诊断信息。 末尾会返回计算得到的状态，或把更新后的结果继续传给调用方。

### Lines 176-184
```python
    # Add nodes
    for node_id, node in graph.nodes.items():
        label = f"{node_id}\n{node.op_name}\ndepth {node.depth}"
        G.add_node(node_id, label=label, node=node)

    # Add edges based on the graph structure
    for node_id, node in graph.nodes.items():
        for input_node_id in node.input_nodes:
            if input_node_id in graph.nodes:  # Only add edges to nodes in the graph
```
- **EN**: This chunk continues `operation_graph_to_networkx` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `operation_graph_to_networkx`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 185-193
```python
                G.add_edge(input_node_id, node_id)

    # Plot
    plt.figure(figsize=(12, 8))
    pos = nx.spring_layout(G, k=2, iterations=50)

    # Draw nodes with colors based on operation type
    node_colors = []
    for node_id in G.nodes():
```
- **EN**: This chunk continues `operation_graph_to_networkx` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `operation_graph_to_networkx`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 194-203
```python
        node = graph.nodes[node_id]
        if node.op_name.startswith("arg_"):
            node_colors.append("lightblue")
        elif node.op_name == "constant":
            node_colors.append("lightgreen")
        elif "aten" in node.op_name:
            node_colors.append("lightyellow")
        else:
            node_colors.append("lightgray")
```
- **EN**: This chunk continues `operation_graph_to_networkx` and expands its internal control flow or data movement. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `operation_graph_to_networkx`，进一步展开其内部控制流或数据流转。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 204-214
```python
    # Highlight root node
    node_sizes = []
    for node_id in G.nodes():
        if node_id == graph.root_node_id:
            node_sizes.append(2000)  # Larger size for root
        else:
            node_sizes.append(1500)

    nx.draw_networkx_nodes(G, pos, node_color=node_colors, node_size=node_sizes)
    nx.draw_networkx_edges(G, pos, edge_color="gray", arrows=True, arrowsize=20)
```
- **EN**: This chunk continues `operation_graph_to_networkx` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently. Conditional branches preserve platform, workflow, or metadata-specific invariants before proceeding.
- **CN**: 这一段延续了 `operation_graph_to_networkx`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。 条件分支在继续执行前维护平台、工作流或元数据相关的不变量。

### Lines 215-226
```python
    # Draw labels
    labels = {
        node_id: f"{node_id}\n{graph.nodes[node_id].op_name}" for node_id in G.nodes()
    }
    nx.draw_networkx_labels(G, pos, labels, font_size=8)

    plt.title("Operation Graph Visualization")
    plt.axis("off")
    plt.tight_layout()
    plt.savefig("operation_graph_networkx.png", dpi=300, bbox_inches="tight")
    plt.show()
```
- **EN**: This chunk continues `operation_graph_to_networkx` and expands its internal control flow or data movement. Looping logic walks files, schema entries, tests, or generated records so the same rule can be applied consistently.
- **CN**: 这一段延续了 `operation_graph_to_networkx`，进一步展开其内部控制流或数据流转。 循环逻辑遍历文件、schema 条目、测试或生成记录，从而把同一规则一致地应用到各项对象。

### Lines 227-227
```python
    print("✓ NetworkX graph visualization saved as operation_graph_networkx.png")
```
- **EN**: This chunk continues `operation_graph_to_networkx` and expands its internal control flow or data movement.
- **CN**: 这一段延续了 `operation_graph_to_networkx`，进一步展开其内部控制流或数据流转。

## Key Concepts / 关键概念

- **Torch fuzzing**
  - EN: This file belongs to the torch fuzzing layer and should be read in the context of neighboring scripts in the same subtree.
  - CN: 该文件属于Torch 模糊测试层，应结合同一子目录中的相邻脚本一起理解。
- **Process orchestration**
  - EN: The file launches external commands and translates their results back into Python control flow.
  - CN: 该文件负责启动外部命令，并把结果重新映射回 Python 控制流。
- **Fuzz testing**
  - EN: The implementation creates randomized or adversarial inputs to probe API robustness.
  - CN: 该实现构造随机或对抗性输入，以探测 API 的健壮性。
- **save_and_render_dot**
  - EN: `save_and_render_dot` is one of the main local symbols exposed or implemented here.
  - CN: `save_and_render_dot` 是此处暴露或实现的主要局部符号之一。
- **operation_graph_to_dot**
  - EN: `operation_graph_to_dot` is one of the main local symbols exposed or implemented here.
  - CN: `operation_graph_to_dot` 是此处暴露或实现的主要局部符号之一。
## Dependencies / 依赖关系

- **Python standard library / Python 标准库**: `subprocess`, `os`
- **External packages / 外部依赖包**: `torchfuzz.ops_fuzzer`, `torchfuzz.tensor_fuzzer`, `matplotlib.pyplot`, `networkx`
- **Primary symbols in this file / 本文件核心符号**: `save_and_render_dot`, `operation_graph_to_dot`, `visualize_operation_graph`, `operation_graph_to_networkx`
