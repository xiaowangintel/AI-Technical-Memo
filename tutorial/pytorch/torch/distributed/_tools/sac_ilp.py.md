# sac_ilp.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/_tools/sac_ilp.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed runtime helpers, APIs, and package wiring. Its main entry points include SACDecision, sac_milp, get_optimal_checkpointing_policy_per_module.
- **用途 (CN)**: 该模块聚焦于分布式运行时辅助逻辑、API 与包级导出，其主要入口包括 SACDecision, sac_milp, get_optimal_checkpointing_policy_per_module。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
import logging
import math
from enum import IntEnum

from torch.distributed._tools.ilp_utils import Graph, is_submodule
from torch.distributed._tools.sac_estimator import SACStats


try:
    from pulp import (  # type: ignore[import-untyped,import-not-found]
        lpDot,
        LpInteger,
        LpMaximize,
        LpMinimize,
        LpProblem,
        LpStatus,
        lpSum,
        LpVariable,
        PULP_CBC_CMD,
        value,
````

- **L1** EN: Imports module dependencies: `logging`. | CN: 导入模块依赖：`logging`。
- **L2** EN: Imports module dependencies: `math`. | CN: 导入模块依赖：`math`。
- **L3** EN: Imports selected names from `enum`. | CN: 从 `enum` 导入指定名称。
- **L4** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L5** EN: Imports selected names from `torch.distributed._tools.ilp_utils`. | CN: 从 `torch.distributed._tools.ilp_utils` 导入指定名称。
- **L6** EN: Imports selected names from `torch.distributed._tools.sac_estimator`. | CN: 从 `torch.distributed._tools.sac_estimator` 导入指定名称。
- **L7** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L8** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L9** EN: Starts a protected block for exception handling. | CN: 开始一个受保护的异常处理代码块。
- **L10** EN: Imports selected names from `pulp`. | CN: 从 `pulp` 导入指定名称。
- **L11** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L12** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L13** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L14** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L15** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L16** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L17** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L18** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L19** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L20** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。

### Lines 21-40 / 第 21-40 行

````python
    )
except ImportError as err:
    raise ImportError(
        "Please install pulp package. See: https://github.com/coin-or/pulp."
    ) from err

# Create a logger object
logger = logging.getLogger(__name__)

# Set the logging level to INFO
logger.setLevel(logging.INFO)


def sac_milp(
    graph: Graph,
    memory_budget: float,
    world_size: int = 1,
    ac_units: list[str] | None = None,
    fsdp_units: list[str] | None = None,
) -> tuple[dict[str, float], float, int]:
````

- **L21** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L22** EN: Handles a matching exception path. | CN: 处理匹配到的异常路径。
- **L23** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L24** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L25** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L26** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L27** EN: Keeps the inline comment or directive: Create a logger object | CN: 保留这一行注释或指令：Create a logger object
- **L28** EN: Assigns or updates `logger`. | CN: 对 `logger` 进行赋值或更新。
- **L29** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L30** EN: Keeps the inline comment or directive: Set the logging level to INFO | CN: 保留这一行注释或指令：Set the logging level to INFO
- **L31** EN: Calls `logger.setLevel` as part of the current workflow. | CN: 在当前流程中调用 `logger.setLevel`。
- **L32** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L33** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L34** EN: Defines function `sac_milp`. | CN: 定义函数 `sac_milp`。
- **L35** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L36** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L37** EN: Assigns or updates `world_size`. | CN: 对 `world_size` 进行赋值或更新。
- **L38** EN: Assigns or updates `ac_units`. | CN: 对 `ac_units` 进行赋值或更新。
- **L39** EN: Assigns or updates `fsdp_units`. | CN: 对 `fsdp_units` 进行赋值或更新。
- **L40** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。

### Lines 41-60 / 第 41-60 行

````python
    """
    MILP to decide which modules to AC and how much memory to discard.
    The objective is to minimize recomputation time.
    The constraint is to ensure peak memory is under budget.

    Args:
        graph: graph representation of the model as a module submodule tree
            where each node is a submodule with memory & runtime stats
        memory_budget: memory budget in GiB
        world_size: number of GPUs. In the case of FSDP, world_size will be
            used to compute the amount of parameter and gradient memory on each rank
        ac_units: a list of user-specified AC units.
        fsdp_units: a list of FSDP units. AC units cannot be supermodules of FSDP units.

    Returns:
        Dict[str, float]: the optimal SAC solution, mapping from module fqn to
            the percentage of activation memory to **discard**
        float: the recomputation time of the optimal SAC solution
        int: upper bound on the peak memory of the optimal SAC solution.
            note that value of -1 means that the ILP solver failed to find a solution.
````

- **L41** EN: Starts the docstring for the function sac_milp. | CN: 开始定义 function sac_milp 的文档字符串。
- **L42** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L43** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L44** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L45** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L46** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L47** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L48** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L49** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L50** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L51** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L52** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L53** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L54** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L55** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L56** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L57** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L58** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L59** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L60** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。

### Lines 61-80 / 第 61-80 行

````python

    """
    num_nodes = len(graph.nodes)
    M = 10**2  # note: numerical issue may occur if M is too big
    MEM_MULTIPLIER = 2**30

    # Create a MILP problem
    prob = LpProblem("SAC", LpMinimize)

    # Create decision variables
    # y_i: indicator for if module i is AC'ed
    y = LpVariable.matrix("y", list(range(num_nodes)), 0, 1, LpInteger)
    # r_i: percentage of discarded activation memory
    r = LpVariable.matrix("r", list(range(num_nodes)), 0, 1)
    # d_i: discarded activation memory for module i
    d = LpVariable.matrix("d", list(range(num_nodes)), 0)
    # a_i: total activation memory at module i
    a = LpVariable.matrix("a", list(range(num_nodes)), 0)
    # m_i: memory at module i, combining parameters, gradients, and activations
    m = LpVariable.matrix("m", list(range(num_nodes)), 0)
````

- **L61** EN: Continues the docstring text for the function sac_milp. | CN: 继续补充 function sac_milp 的文档字符串内容。
- **L62** EN: Closes the docstring for the function sac_milp. | CN: 结束 function sac_milp 的文档字符串。
- **L63** EN: Assigns or updates `num_nodes`. | CN: 对 `num_nodes` 进行赋值或更新。
- **L64** EN: Assigns or updates `M`. | CN: 对 `M` 进行赋值或更新。
- **L65** EN: Assigns or updates `MEM_MULTIPLIER`. | CN: 对 `MEM_MULTIPLIER` 进行赋值或更新。
- **L66** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L67** EN: Keeps the inline comment or directive: Create a MILP problem | CN: 保留这一行注释或指令：Create a MILP problem
- **L68** EN: Assigns or updates `prob`. | CN: 对 `prob` 进行赋值或更新。
- **L69** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L70** EN: Keeps the inline comment or directive: Create decision variables | CN: 保留这一行注释或指令：Create decision variables
- **L71** EN: Keeps the inline comment or directive: y_i: indicator for if module i is AC'ed | CN: 保留这一行注释或指令：y_i: indicator for if module i is AC'ed
- **L72** EN: Assigns or updates `y`. | CN: 对 `y` 进行赋值或更新。
- **L73** EN: Keeps the inline comment or directive: r_i: percentage of discarded activation memory | CN: 保留这一行注释或指令：r_i: percentage of discarded activation memory
- **L74** EN: Assigns or updates `r`. | CN: 对 `r` 进行赋值或更新。
- **L75** EN: Keeps the inline comment or directive: d_i: discarded activation memory for module i | CN: 保留这一行注释或指令：d_i: discarded activation memory for module i
- **L76** EN: Assigns or updates `d`. | CN: 对 `d` 进行赋值或更新。
- **L77** EN: Keeps the inline comment or directive: a_i: total activation memory at module i | CN: 保留这一行注释或指令：a_i: total activation memory at module i
- **L78** EN: Assigns or updates `a`. | CN: 对 `a` 进行赋值或更新。
- **L79** EN: Keeps the inline comment or directive: m_i: memory at module i, combining parameters, gradients, and activations | CN: 保留这一行注释或指令：m_i: memory at module i, combining parameters, gradients, and activations
- **L80** EN: Assigns or updates `m`. | CN: 对 `m` 进行赋值或更新。

### Lines 81-100 / 第 81-100 行

````python
    # rcp_i: percentage of recomputation time
    rcp = LpVariable.matrix("rcp", list(range(num_nodes)), 0)
    # rct_i: recomputation time for module i (in ms)
    rct = LpVariable.matrix("rct", list(range(num_nodes)), 0)
    # max_m: peak memory
    max_m = LpVariable("max_m", 0)

    # Add constraints
    # [Constraint] User specified AC units
    if ac_units:
        ac_units_set = set(ac_units)
        for i in range(num_nodes):
            if graph.nodes[i]["fqn"] not in ac_units_set:
                prob += y[i] == 0

    # [Constraint] AC units cannot be supmodules of user specified FSDP units
    if fsdp_units:
        for i in range(num_nodes):
            if any(
                is_submodule(fsdp_unit, graph.nodes[i]["fqn"])
````

- **L81** EN: Keeps the inline comment or directive: rcp_i: percentage of recomputation time | CN: 保留这一行注释或指令：rcp_i: percentage of recomputation time
- **L82** EN: Assigns or updates `rcp`. | CN: 对 `rcp` 进行赋值或更新。
- **L83** EN: Keeps the inline comment or directive: rct_i: recomputation time for module i (in ms) | CN: 保留这一行注释或指令：rct_i: recomputation time for module i (in ms)
- **L84** EN: Assigns or updates `rct`. | CN: 对 `rct` 进行赋值或更新。
- **L85** EN: Keeps the inline comment or directive: max_m: peak memory | CN: 保留这一行注释或指令：max_m: peak memory
- **L86** EN: Assigns or updates `max_m`. | CN: 对 `max_m` 进行赋值或更新。
- **L87** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L88** EN: Keeps the inline comment or directive: Add constraints | CN: 保留这一行注释或指令：Add constraints
- **L89** EN: Keeps the inline comment or directive: [Constraint] User specified AC units | CN: 保留这一行注释或指令：[Constraint] User specified AC units
- **L90** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L91** EN: Assigns or updates `ac_units_set`. | CN: 对 `ac_units_set` 进行赋值或更新。
- **L92** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L93** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L94** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L95** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L96** EN: Keeps the inline comment or directive: [Constraint] AC units cannot be supmodules of user specified FSDP units | CN: 保留这一行注释或指令：[Constraint] AC units cannot be supmodules of user specified FSDP units
- **L97** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L98** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L99** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L100** EN: Calls `is_submodule` as part of the current workflow. | CN: 在当前流程中调用 `is_submodule`。

### Lines 101-120 / 第 101-120 行

````python
                for fsdp_unit in fsdp_units
            ):
                prob += y[i] == 0

    # [Constraint] No nested AC units
    for i in range(num_nodes):
        for j in range(i + 1, num_nodes):
            if graph.ad_matrix[i][j] == 1:
                prob += y[i] + y[j] <= 1

    # [Constraint] Do not AC leaf modules
    for i in range(num_nodes):
        if graph.nodes[i]["is_leaf"]:
            prob += y[i] == 0

    # [Constraint] Express amount of discarded activation memory
    for i in range(num_nodes):
        # There are two measures for activation memory: ACM and IA
        # 1. IA is the activation memory saved when not using AC
        # 2. ACM is the total activation memory, including those
````

- **L101** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L102** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L103** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L104** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L105** EN: Keeps the inline comment or directive: [Constraint] No nested AC units | CN: 保留这一行注释或指令：[Constraint] No nested AC units
- **L106** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L107** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L108** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L109** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L110** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L111** EN: Keeps the inline comment or directive: [Constraint] Do not AC leaf modules | CN: 保留这一行注释或指令：[Constraint] Do not AC leaf modules
- **L112** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L113** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L114** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L115** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L116** EN: Keeps the inline comment or directive: [Constraint] Express amount of discarded activation memory | CN: 保留这一行注释或指令：[Constraint] Express amount of discarded activation memory
- **L117** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L118** EN: Keeps the inline comment or directive: There are two measures for activation memory: ACM and IA | CN: 保留这一行注释或指令：There are two measures for activation memory: ACM and IA
- **L119** EN: Keeps the inline comment or directive: 1. IA is the activation memory saved when not using AC | CN: 保留这一行注释或指令：1. IA is the activation memory saved when not using AC
- **L120** EN: Keeps the inline comment or directive: 2. ACM is the total activation memory, including those | CN: 保留这一行注释或指令：2. ACM is the total activation memory, including those

### Lines 121-140 / 第 121-140 行

````python
        #    that are not typically saved when not using AC
        # Note: ACM >= IA
        if (not graph.nodes[i]["is_leaf"]) and graph.nodes[i][
            "sac_memory"
        ] < graph.nodes[i]["act_fw_per_module"]:
            logger.warning("For module {%s}: ", graph.nodes[i]["fqn"])
            logger.warning(
                "activation memory from memory tracker is {%d},",
                graph.nodes[i]["act_fw_per_module"],
            )
            logger.warning(
                "activation memory from SAC estimator is {%d}.",
                graph.nodes[i]["sac_memory"],
            )
            logger.warning("Something is wrong. Please check!")
            logger.warning("Overriding the latter with the former.")
            graph.nodes[i]["sac_memory"] = graph.nodes[i]["act_fw_per_module"]
        ACM_i = graph.nodes[i]["sac_memory"] / MEM_MULTIPLIER
        IA_i = graph.nodes[i]["act_fw_per_module"] / MEM_MULTIPLIER
        prob += d[i] == ACM_i * r[i] - (ACM_i - IA_i) * y[i]
````

- **L121** EN: Keeps the inline comment or directive: that are not typically saved when not using AC | CN: 保留这一行注释或指令：that are not typically saved when not using AC
- **L122** EN: Keeps the inline comment or directive: Note: ACM >= IA | CN: 保留这一行注释或指令：Note: ACM >= IA
- **L123** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L124** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L125** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L126** EN: Calls `logger.warning` as part of the current workflow. | CN: 在当前流程中调用 `logger.warning`。
- **L127** EN: Calls `logger.warning` as part of the current workflow. | CN: 在当前流程中调用 `logger.warning`。
- **L128** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L129** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L130** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L131** EN: Calls `logger.warning` as part of the current workflow. | CN: 在当前流程中调用 `logger.warning`。
- **L132** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L133** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L134** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L135** EN: Calls `logger.warning` as part of the current workflow. | CN: 在当前流程中调用 `logger.warning`。
- **L136** EN: Calls `logger.warning` as part of the current workflow. | CN: 在当前流程中调用 `logger.warning`。
- **L137** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L138** EN: Assigns or updates `ACM_i`. | CN: 对 `ACM_i` 进行赋值或更新。
- **L139** EN: Assigns or updates `IA_i`. | CN: 对 `IA_i` 进行赋值或更新。
- **L140** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。

### Lines 141-160 / 第 141-160 行

````python

    # [Constraint] Ensure correctness of r_i
    # There are two parts to its correctness
    # 1. r_i > 0 only if y_i == 1 (discard only if it is an AC unit)
    # 2. r_i needs to be large enough to cover the difference between
    #    ACM and IA. Otherwise, we are not saving any memory
    for i in range(num_nodes):
        prob += y[i] >= r[i]
        if graph.nodes[i]["is_leaf"]:
            continue
        ACM_i = graph.nodes[i]["sac_memory"] / MEM_MULTIPLIER
        IA_i = graph.nodes[i]["act_fw_per_module"] / MEM_MULTIPLIER
        prob += r[i] >= (ACM_i - IA_i) / ACM_i * y[i]

    # [Constraint] Express total activation memory in the backward pass
    for i in range(num_nodes):
        AG_i = graph.nodes[i]["act_grad_per_module"] / MEM_MULTIPLIER
        TA_i = graph.nodes[i]["act_total"] / MEM_MULTIPLIER
        # related to discarded amount of memory
        pos = graph.nodes[i]["pos_fw_post_order"]
````

- **L141** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L142** EN: Keeps the inline comment or directive: [Constraint] Ensure correctness of r_i | CN: 保留这一行注释或指令：[Constraint] Ensure correctness of r_i
- **L143** EN: Keeps the inline comment or directive: There are two parts to its correctness | CN: 保留这一行注释或指令：There are two parts to its correctness
- **L144** EN: Keeps the inline comment or directive: 1. r_i > 0 only if y_i == 1 (discard only if it is an AC unit) | CN: 保留这一行注释或指令：1. r_i > 0 only if y_i == 1 (discard only if it is an AC unit)
- **L145** EN: Keeps the inline comment or directive: 2. r_i needs to be large enough to cover the difference between | CN: 保留这一行注释或指令：2. r_i needs to be large enough to cover the difference between
- **L146** EN: Keeps the inline comment or directive: ACM and IA. Otherwise, we are not saving any memory | CN: 保留这一行注释或指令：ACM and IA. Otherwise, we are not saving any memory
- **L147** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L148** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L149** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L150** EN: Skips to the next loop iteration. | CN: 跳到下一次循环迭代。
- **L151** EN: Assigns or updates `ACM_i`. | CN: 对 `ACM_i` 进行赋值或更新。
- **L152** EN: Assigns or updates `IA_i`. | CN: 对 `IA_i` 进行赋值或更新。
- **L153** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L154** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L155** EN: Keeps the inline comment or directive: [Constraint] Express total activation memory in the backward pass | CN: 保留这一行注释或指令：[Constraint] Express total activation memory in the backward pass
- **L156** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L157** EN: Assigns or updates `AG_i`. | CN: 对 `AG_i` 进行赋值或更新。
- **L158** EN: Assigns or updates `TA_i`. | CN: 对 `TA_i` 进行赋值或更新。
- **L159** EN: Keeps the inline comment or directive: related to discarded amount of memory | CN: 保留这一行注释或指令：related to discarded amount of memory
- **L160** EN: Assigns or updates `pos`. | CN: 对 `pos` 进行赋值或更新。

### Lines 161-180 / 第 161-180 行

````python
        coeff = [0] * num_nodes
        for p in range(pos):
            j = graph.name2node[graph.fw_post_order[p]]["index"]
            coeff[j] = 1
        prob += a[i] == TA_i + AG_i - lpDot(coeff, d)

    # [Constraint] Express the total amount of memory at each module
    # Note that unsharded parameters and gradients are not included here
    P_1 = graph.nodes[0]["param_per_module"] / MEM_MULTIPLIER
    for i in range(num_nodes):
        TG_i = graph.nodes[i]["grad_total"] / MEM_MULTIPLIER
        prob += m[i] == a[i] + (P_1 + TG_i) / world_size

    # [Constraint] Express peak memory
    for i in range(num_nodes):
        prob += max_m >= m[i]

    # [Constraint] Express percentage of recomputation time
    for i in range(num_nodes):
        for s in range(graph.nodes[i]["n_segments"]):
````

- **L161** EN: Assigns or updates `coeff`. | CN: 对 `coeff` 进行赋值或更新。
- **L162** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L163** EN: Assigns or updates `j`. | CN: 对 `j` 进行赋值或更新。
- **L164** EN: Assigns or updates `coeff[j]`. | CN: 对 `coeff[j]` 进行赋值或更新。
- **L165** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L166** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L167** EN: Keeps the inline comment or directive: [Constraint] Express the total amount of memory at each module | CN: 保留这一行注释或指令：[Constraint] Express the total amount of memory at each module
- **L168** EN: Keeps the inline comment or directive: Note that unsharded parameters and gradients are not included here | CN: 保留这一行注释或指令：Note that unsharded parameters and gradients are not included here
- **L169** EN: Assigns or updates `P_1`. | CN: 对 `P_1` 进行赋值或更新。
- **L170** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L171** EN: Assigns or updates `TG_i`. | CN: 对 `TG_i` 进行赋值或更新。
- **L172** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L173** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L174** EN: Keeps the inline comment or directive: [Constraint] Express peak memory | CN: 保留这一行注释或指令：[Constraint] Express peak memory
- **L175** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L176** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L177** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L178** EN: Keeps the inline comment or directive: [Constraint] Express percentage of recomputation time | CN: 保留这一行注释或指令：[Constraint] Express percentage of recomputation time
- **L179** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L180** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。

### Lines 181-200 / 第 181-200 行

````python
            slope = graph.nodes[i]["slopes"][s]
            intercept = graph.nodes[i]["intercepts"][s]
            prob += rcp[i] >= slope * r[i] + intercept

    # [Constraint] Express recomputation time
    # rct_i = (rcp_i * ACT_i) if y_i == 1 else 0
    for i in range(num_nodes):
        ACT_i = graph.nodes[i]["sac_runtime"]
        prob += rct[i] <= M * y[i]
        prob += rct[i] <= ACT_i * rcp[i]
        prob += rct[i] >= ACT_i * rcp[i] - M * (1 - y[i])

    # [Constraint] Peak memory should be below budget
    prob += max_m <= memory_budget

    # Set Objeictive
    prob += lpSum(rct)

    # Solve
    solver = PULP_CBC_CMD(gapRel=0.05, timeLimit=180, msg=0)
````

- **L181** EN: Assigns or updates `slope`. | CN: 对 `slope` 进行赋值或更新。
- **L182** EN: Assigns or updates `intercept`. | CN: 对 `intercept` 进行赋值或更新。
- **L183** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L184** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L185** EN: Keeps the inline comment or directive: [Constraint] Express recomputation time | CN: 保留这一行注释或指令：[Constraint] Express recomputation time
- **L186** EN: Keeps the inline comment or directive: rct_i = (rcp_i * ACT_i) if y_i == 1 else 0 | CN: 保留这一行注释或指令：rct_i = (rcp_i * ACT_i) if y_i == 1 else 0
- **L187** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L188** EN: Assigns or updates `ACT_i`. | CN: 对 `ACT_i` 进行赋值或更新。
- **L189** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L190** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L191** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L192** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L193** EN: Keeps the inline comment or directive: [Constraint] Peak memory should be below budget | CN: 保留这一行注释或指令：[Constraint] Peak memory should be below budget
- **L194** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L195** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L196** EN: Keeps the inline comment or directive: Set Objeictive | CN: 保留这一行注释或指令：Set Objeictive
- **L197** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L198** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L199** EN: Keeps the inline comment or directive: Solve | CN: 保留这一行注释或指令：Solve
- **L200** EN: Assigns or updates `solver`. | CN: 对 `solver` 进行赋值或更新。

### Lines 201-220 / 第 201-220 行

````python
    status = prob.solve(solver)

    # If solver fails, print status and return empty solution
    if status != 1:
        logger.error("Solver failed to find a solution: %s", LpStatus[status])
        return {}, 0, -1

    # Gather and return solution if optimal solution is found
    ac_decisions = {}
    for i in range(num_nodes):
        if round(y[i].varValue) == 1:
            ac_decisions[graph.nodes[i]["fqn"]] = round(r[i].varValue, 4)
    recomputation_time = round(value(prob.objective), 2)
    peak_mem = round(max_m.varValue * MEM_MULTIPLIER)

    return ac_decisions, recomputation_time, peak_mem


class SACDecision(IntEnum):
    RECOMPUTE = 0
````

- **L201** EN: Assigns or updates `status`. | CN: 对 `status` 进行赋值或更新。
- **L202** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L203** EN: Keeps the inline comment or directive: If solver fails, print status and return empty solution | CN: 保留这一行注释或指令：If solver fails, print status and return empty solution
- **L204** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L205** EN: Calls `logger.error` as part of the current workflow. | CN: 在当前流程中调用 `logger.error`。
- **L206** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L207** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L208** EN: Keeps the inline comment or directive: Gather and return solution if optimal solution is found | CN: 保留这一行注释或指令：Gather and return solution if optimal solution is found
- **L209** EN: Assigns or updates `ac_decisions`. | CN: 对 `ac_decisions` 进行赋值或更新。
- **L210** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L211** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L212** EN: Continues the implementation inside function `sac_milp`. | CN: 继续说明函数 `sac_milp` 内部的实现。
- **L213** EN: Assigns or updates `recomputation_time`. | CN: 对 `recomputation_time` 进行赋值或更新。
- **L214** EN: Assigns or updates `peak_mem`. | CN: 对 `peak_mem` 进行赋值或更新。
- **L215** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L216** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L217** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L218** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L219** EN: Defines class `SACDecision`. | CN: 定义类 `SACDecision`。
- **L220** EN: Assigns or updates `RECOMPUTE`. | CN: 对 `RECOMPUTE` 进行赋值或更新。

### Lines 221-240 / 第 221-240 行

````python
    SAVE = 1


def get_optimal_checkpointing_policy_per_module(
    sac_stats: SACStats, memory_budget: float
) -> list[int]:
    """
    This is adapted from --
    https://github.com/facebookresearch/xformers/blob/c6c0ac31f1b08542a0bc27278c6ed10f825f6963/xformers/checkpoint.py#L375

    Given the SACStats of a module, including list of operators, their memory, runtimes, and metadata,
    decide via MILP an optimal set of operators to checkpoint under a given ``memory_budget``.

    Args:
        sac_stats: the SACStats object of the module
        memory_budget: a float between zero and one

    Returns:
        List[int]: the decision whether each operator should be saved (1) or recomptued (0).
    """
````

- **L221** EN: Assigns or updates `SAVE`. | CN: 对 `SAVE` 进行赋值或更新。
- **L222** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L223** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L224** EN: Defines function `get_optimal_checkpointing_policy_per_module`. | CN: 定义函数 `get_optimal_checkpointing_policy_per_module`。
- **L225** EN: Continues the implementation inside function `get_optimal_checkpointing_policy_per_module`. | CN: 继续说明函数 `get_optimal_checkpointing_policy_per_module` 内部的实现。
- **L226** EN: Continues the implementation inside function `get_optimal_checkpointing_policy_per_module`. | CN: 继续说明函数 `get_optimal_checkpointing_policy_per_module` 内部的实现。
- **L227** EN: Starts the docstring for the function get_optimal_checkpointing_policy_per_module. | CN: 开始定义 function get_optimal_checkpointing_policy_per_module 的文档字符串。
- **L228** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L229** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L230** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L231** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L232** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L233** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L234** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L235** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L236** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L237** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L238** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L239** EN: Continues the docstring text for the function get_optimal_checkpointing_policy_per_module. | CN: 继续补充 function get_optimal_checkpointing_policy_per_module 的文档字符串内容。
- **L240** EN: Closes the docstring for the function get_optimal_checkpointing_policy_per_module. | CN: 结束 function get_optimal_checkpointing_policy_per_module 的文档字符串。

### Lines 241-260 / 第 241-260 行

````python
    if not (0 <= memory_budget <= 1):
        raise ValueError(
            f"`memory_budget` must be a float between 0 and 1. Got {memory_budget}."
        )
    num_ops = len(sac_stats.func_names)

    # Create a MILP problem
    prob = LpProblem("SAC-per-module", LpMaximize)

    # Create decision variables
    # x[i] = 1 means the i-th operator should be saved, otherwise it should be recomputed
    x = LpVariable.matrix("x", list(range(num_ops)), 0, 1, LpInteger)

    # Add constraints
    # [Constraint] random ops should be saved if ``force_store_random`` is True
    # otherwise, random ops should either be all recomputed or all saved
    if sac_stats.force_store_random:
        for i in sac_stats.rand_ops:
            prob += x[i] == SACDecision.SAVE.value
    else:
````

- **L241** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L242** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L243** EN: Continues the implementation inside function `get_optimal_checkpointing_policy_per_module`. | CN: 继续说明函数 `get_optimal_checkpointing_policy_per_module` 内部的实现。
- **L244** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L245** EN: Assigns or updates `num_ops`. | CN: 对 `num_ops` 进行赋值或更新。
- **L246** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L247** EN: Keeps the inline comment or directive: Create a MILP problem | CN: 保留这一行注释或指令：Create a MILP problem
- **L248** EN: Assigns or updates `prob`. | CN: 对 `prob` 进行赋值或更新。
- **L249** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L250** EN: Keeps the inline comment or directive: Create decision variables | CN: 保留这一行注释或指令：Create decision variables
- **L251** EN: Keeps the inline comment or directive: x[i] = 1 means the i-th operator should be saved, otherwise it should be recompu | CN: 保留这一行注释或指令：x[i] = 1 means the i-th operator should be saved, otherwise it should be recompu
- **L252** EN: Assigns or updates `x`. | CN: 对 `x` 进行赋值或更新。
- **L253** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L254** EN: Keeps the inline comment or directive: Add constraints | CN: 保留这一行注释或指令：Add constraints
- **L255** EN: Keeps the inline comment or directive: [Constraint] random ops should be saved if ``force_store_random`` is True | CN: 保留这一行注释或指令：[Constraint] random ops should be saved if ``force_store_random`` is True
- **L256** EN: Keeps the inline comment or directive: otherwise, random ops should either be all recomputed or all saved | CN: 保留这一行注释或指令：otherwise, random ops should either be all recomputed or all saved
- **L257** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L258** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L259** EN: Continues the implementation inside function `get_optimal_checkpointing_policy_per_module`. | CN: 继续说明函数 `get_optimal_checkpointing_policy_per_module` 内部的实现。
- **L260** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。

### Lines 261-280 / 第 261-280 行

````python
        for i1, i2 in zip(sac_stats.rand_ops[:-1], sac_stats.rand_ops[1:]):
            prob += x[i1] == x[i2]

    # [Constraint] view-like ops should always be recomputed
    for i in sac_stats.view_like_ops:
        prob += x[i] == SACDecision.RECOMPUTE.value

    # [Constraint] inplace ops should always be done in conjunction with its parent op
    for op, op_parent in sac_stats.inplace_ops:
        if op != op_parent:
            prob += x[op] == x[op_parent]
        else:
            prob += x[op] == SACDecision.SAVE.value

    # [Constraint] saved memory should be under the ``memory_budget``
    max_memory = math.ceil(memory_budget * sum(sac_stats.memory))
    prob += lpDot(x, sac_stats.memory) <= max_memory

    # [Objective] minimize recomputation time, note the ILP is a maximization problem
    # because x[i] == 1 means the op is saved (not recomputed), and thus recomputation
````

- **L261** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L262** EN: Continues the implementation inside function `get_optimal_checkpointing_policy_per_module`. | CN: 继续说明函数 `get_optimal_checkpointing_policy_per_module` 内部的实现。
- **L263** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L264** EN: Keeps the inline comment or directive: [Constraint] view-like ops should always be recomputed | CN: 保留这一行注释或指令：[Constraint] view-like ops should always be recomputed
- **L265** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L266** EN: Continues the implementation inside function `get_optimal_checkpointing_policy_per_module`. | CN: 继续说明函数 `get_optimal_checkpointing_policy_per_module` 内部的实现。
- **L267** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L268** EN: Keeps the inline comment or directive: [Constraint] inplace ops should always be done in conjunction with its parent op | CN: 保留这一行注释或指令：[Constraint] inplace ops should always be done in conjunction with its parent op
- **L269** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L270** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L271** EN: Continues the implementation inside function `get_optimal_checkpointing_policy_per_module`. | CN: 继续说明函数 `get_optimal_checkpointing_policy_per_module` 内部的实现。
- **L272** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L273** EN: Continues the implementation inside function `get_optimal_checkpointing_policy_per_module`. | CN: 继续说明函数 `get_optimal_checkpointing_policy_per_module` 内部的实现。
- **L274** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L275** EN: Keeps the inline comment or directive: [Constraint] saved memory should be under the ``memory_budget`` | CN: 保留这一行注释或指令：[Constraint] saved memory should be under the ``memory_budget``
- **L276** EN: Assigns or updates `max_memory`. | CN: 对 `max_memory` 进行赋值或更新。
- **L277** EN: Continues the implementation inside function `get_optimal_checkpointing_policy_per_module`. | CN: 继续说明函数 `get_optimal_checkpointing_policy_per_module` 内部的实现。
- **L278** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L279** EN: Keeps the inline comment or directive: [Objective] minimize recomputation time, note the ILP is a maximization problem | CN: 保留这一行注释或指令：[Objective] minimize recomputation time, note the ILP is a maximization problem
- **L280** EN: Keeps the inline comment or directive: because x[i] == 1 means the op is saved (not recomputed), and thus recomputation | CN: 保留这一行注释或指令：because x[i] == 1 means the op is saved (not recomputed), and thus recomputation

### Lines 281-294 / 第 281-294 行

````python
    # time is sum(sac_stats.runtimes) - lpDot(x, sac_stats.runtimes)
    prob += lpDot(x, sac_stats.runtimes)

    # Solve
    solver = PULP_CBC_CMD(gapRel=0.05, timeLimit=10, msg=0)
    status = prob.solve(solver)

    # If solver fails, print status and return empty solution
    if status != 1:
        logger.error("Solver failed to find a solution: %s", LpStatus[status])
        return []

    # Gather and return solution if optimal solution is found
    return [round(x[i].varValue) for i in range(num_ops)]
````

- **L281** EN: Keeps the inline comment or directive: time is sum(sac_stats.runtimes) - lpDot(x, sac_stats.runtimes) | CN: 保留这一行注释或指令：time is sum(sac_stats.runtimes) - lpDot(x, sac_stats.runtimes)
- **L282** EN: Continues the implementation inside function `get_optimal_checkpointing_policy_per_module`. | CN: 继续说明函数 `get_optimal_checkpointing_policy_per_module` 内部的实现。
- **L283** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L284** EN: Keeps the inline comment or directive: Solve | CN: 保留这一行注释或指令：Solve
- **L285** EN: Assigns or updates `solver`. | CN: 对 `solver` 进行赋值或更新。
- **L286** EN: Assigns or updates `status`. | CN: 对 `status` 进行赋值或更新。
- **L287** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L288** EN: Keeps the inline comment or directive: If solver fails, print status and return empty solution | CN: 保留这一行注释或指令：If solver fails, print status and return empty solution
- **L289** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L290** EN: Calls `logger.error` as part of the current workflow. | CN: 在当前流程中调用 `logger.error`。
- **L291** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L292** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L293** EN: Keeps the inline comment or directive: Gather and return solution if optimal solution is found | CN: 保留这一行注释或指令：Gather and return solution if optimal solution is found
- **L294** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed runtime helpers, APIs, and package wiring  
  **CN**: 分布式运行时辅助逻辑、API 与包级导出
- **EN**: sharding  
  **CN**: 分片
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: distributed stores  
  **CN**: 分布式存储
- **EN**: Primary classes: SACDecision  
  **CN**: 主要类：SACDecision
- **EN**: Core callables: sac_milp, get_optimal_checkpointing_policy_per_module  
  **CN**: 核心可调用对象：sac_milp, get_optimal_checkpointing_policy_per_module

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed._tools.ilp_utils`, `torch.distributed._tools.sac_estimator`
- **PyTorch / PyTorch**: None detected / 未检测到
- **Python Stdlib / Python 标准库**: `enum`, `logging`, `math`
- **Third-party / 第三方**: `pulp`

