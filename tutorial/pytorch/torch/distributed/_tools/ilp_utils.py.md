# ilp_utils.py — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/distributed/_tools/ilp_utils.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: This module focuses on distributed runtime helpers, APIs, and package wiring. Its main entry points include ModOrder, ModRuntime, aggregate_stats, parse_module_info.
- **用途 (CN)**: 该模块聚焦于分布式运行时辅助逻辑、API 与包级导出，其主要入口包括 ModOrder, ModRuntime, aggregate_stats, parse_module_info。

## Line-by-Line Analysis / 逐行分析

### Lines 1-20 / 第 1-20 行

````python
import copy
from collections import OrderedDict
from typing import cast, TypedDict

import numpy as np

import torch
from torch.distributed._tools.mem_tracker import (
    _MemRefType,
    _ModMemStats,
    _ModState,
    MemTracker,
)
from torch.distributed._tools.runtime_estimator import RuntimeEstimator
from torch.distributed._tools.sac_estimator import SACEstimator, SACTradeOffStats


class ModOrder(TypedDict):
    fw_pre_order: list[str]
    bw_pre_order: list[str]
````

- **L1** EN: Imports module dependencies: `copy`. | CN: 导入模块依赖：`copy`。
- **L2** EN: Imports selected names from `collections`. | CN: 从 `collections` 导入指定名称。
- **L3** EN: Imports selected names from `typing`. | CN: 从 `typing` 导入指定名称。
- **L4** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L5** EN: Imports module dependencies: `numpy as np`. | CN: 导入模块依赖：`numpy as np`。
- **L6** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L7** EN: Imports module dependencies: `torch`. | CN: 导入模块依赖：`torch`。
- **L8** EN: Imports selected names from `torch.distributed._tools.mem_tracker`. | CN: 从 `torch.distributed._tools.mem_tracker` 导入指定名称。
- **L9** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L10** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L11** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L12** EN: Continues the module-level implementation. | CN: 继续模块级实现逻辑。
- **L13** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L14** EN: Imports selected names from `torch.distributed._tools.runtime_estimator`. | CN: 从 `torch.distributed._tools.runtime_estimator` 导入指定名称。
- **L15** EN: Imports selected names from `torch.distributed._tools.sac_estimator`. | CN: 从 `torch.distributed._tools.sac_estimator` 导入指定名称。
- **L16** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L17** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L18** EN: Defines class `ModOrder`. | CN: 定义类 `ModOrder`。
- **L19** EN: Continues the implementation inside class `ModOrder`. | CN: 继续说明类 `ModOrder` 内部的实现。
- **L20** EN: Continues the implementation inside class `ModOrder`. | CN: 继续说明类 `ModOrder` 内部的实现。

### Lines 21-40 / 第 21-40 行

````python
    fw_post_order: list[str]
    bw_post_order: list[str]


class ModRuntime(TypedDict):
    fw: float
    bw: float


class ModStats(TypedDict):
    fqn: str
    # per-module params
    param_per_module: int
    # per-module grads
    grad_per_module: int
    # total accumulated gradients up to and including this module
    grad_total: int
    # per module fw activation size (excluding input and output)
    act_fw_per_module: int
    # per module bw activation size during peak_bw
````

- **L21** EN: Continues the implementation inside class `ModOrder`. | CN: 继续说明类 `ModOrder` 内部的实现。
- **L22** EN: Continues the implementation inside class `ModOrder`. | CN: 继续说明类 `ModOrder` 内部的实现。
- **L23** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L24** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L25** EN: Defines class `ModRuntime`. | CN: 定义类 `ModRuntime`。
- **L26** EN: Continues the implementation inside class `ModRuntime`. | CN: 继续说明类 `ModRuntime` 内部的实现。
- **L27** EN: Continues the implementation inside class `ModRuntime`. | CN: 继续说明类 `ModRuntime` 内部的实现。
- **L28** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L29** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L30** EN: Defines class `ModStats`. | CN: 定义类 `ModStats`。
- **L31** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L32** EN: Keeps the inline comment or directive: per-module params | CN: 保留这一行注释或指令：per-module params
- **L33** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L34** EN: Keeps the inline comment or directive: per-module grads | CN: 保留这一行注释或指令：per-module grads
- **L35** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L36** EN: Keeps the inline comment or directive: total accumulated gradients up to and including this module | CN: 保留这一行注释或指令：total accumulated gradients up to and including this module
- **L37** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L38** EN: Keeps the inline comment or directive: per module fw activation size (excluding input and output) | CN: 保留这一行注释或指令：per module fw activation size (excluding input and output)
- **L39** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L40** EN: Keeps the inline comment or directive: per module bw activation size during peak_bw | CN: 保留这一行注释或指令：per module bw activation size during peak_bw

### Lines 41-60 / 第 41-60 行

````python
    act_bw_per_module: int
    # per module activation grad size during peak_bw
    act_grad_per_module: int
    # total activation size up to but excluding the current module
    # includes input of the current module (i.e., output of previous module)
    act_total: int
    # Inputs to the module
    input_per_module: int
    # Outputs of the module
    output_per_module: int
    # Total fw run-time of the module
    fw_runtime_per_module: float
    # Total bw run-time of the module
    bw_runtime_per_module: float
    # Is this module a leaf module
    is_leaf: bool
    # Total ac run-time of the module
    sac_runtime: float
    # Total ac_memory for the module
    sac_memory: int
````

- **L41** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L42** EN: Keeps the inline comment or directive: per module activation grad size during peak_bw | CN: 保留这一行注释或指令：per module activation grad size during peak_bw
- **L43** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L44** EN: Keeps the inline comment or directive: total activation size up to but excluding the current module | CN: 保留这一行注释或指令：total activation size up to but excluding the current module
- **L45** EN: Keeps the inline comment or directive: includes input of the current module (i.e., output of previous module) | CN: 保留这一行注释或指令：includes input of the current module (i.e., output of previous module)
- **L46** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L47** EN: Keeps the inline comment or directive: Inputs to the module | CN: 保留这一行注释或指令：Inputs to the module
- **L48** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L49** EN: Keeps the inline comment or directive: Outputs of the module | CN: 保留这一行注释或指令：Outputs of the module
- **L50** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L51** EN: Keeps the inline comment or directive: Total fw run-time of the module | CN: 保留这一行注释或指令：Total fw run-time of the module
- **L52** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L53** EN: Keeps the inline comment or directive: Total bw run-time of the module | CN: 保留这一行注释或指令：Total bw run-time of the module
- **L54** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L55** EN: Keeps the inline comment or directive: Is this module a leaf module | CN: 保留这一行注释或指令：Is this module a leaf module
- **L56** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L57** EN: Keeps the inline comment or directive: Total ac run-time of the module | CN: 保留这一行注释或指令：Total ac run-time of the module
- **L58** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L59** EN: Keeps the inline comment or directive: Total ac_memory for the module | CN: 保留这一行注释或指令：Total ac_memory for the module
- **L60** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。

### Lines 61-80 / 第 61-80 行

````python
    # Number of piecewise-linear functions used for approximating ac tradeoff curve
    n_segments: int
    # Slopes of the of piecewise-linear functions
    slopes: list[float]
    # Intercepts of the of piecewise-linear functions
    intercepts: list[float]
    # X breakpoints of the of piecewise-linear functions
    breakpoints: list[float]
    # Original trade-off curves
    tradeoff_curve: OrderedDict[float, float]


class ModuleInfo(TypedDict):
    mod_order: ModOrder
    mod_stats: list[ModStats]


def aggregate_stats(
    model: torch.nn.Module,
    mem_tracker: MemTracker,
````

- **L61** EN: Keeps the inline comment or directive: Number of piecewise-linear functions used for approximating ac tradeoff curve | CN: 保留这一行注释或指令：Number of piecewise-linear functions used for approximating ac tradeoff curve
- **L62** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L63** EN: Keeps the inline comment or directive: Slopes of the of piecewise-linear functions | CN: 保留这一行注释或指令：Slopes of the of piecewise-linear functions
- **L64** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L65** EN: Keeps the inline comment or directive: Intercepts of the of piecewise-linear functions | CN: 保留这一行注释或指令：Intercepts of the of piecewise-linear functions
- **L66** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L67** EN: Keeps the inline comment or directive: X breakpoints of the of piecewise-linear functions | CN: 保留这一行注释或指令：X breakpoints of the of piecewise-linear functions
- **L68** EN: Exits the current loop immediately. | CN: 立即退出当前循环。
- **L69** EN: Keeps the inline comment or directive: Original trade-off curves | CN: 保留这一行注释或指令：Original trade-off curves
- **L70** EN: Continues the implementation inside class `ModStats`. | CN: 继续说明类 `ModStats` 内部的实现。
- **L71** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L72** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L73** EN: Defines class `ModuleInfo`. | CN: 定义类 `ModuleInfo`。
- **L74** EN: Continues the implementation inside class `ModuleInfo`. | CN: 继续说明类 `ModuleInfo` 内部的实现。
- **L75** EN: Continues the implementation inside class `ModuleInfo`. | CN: 继续说明类 `ModuleInfo` 内部的实现。
- **L76** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L77** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L78** EN: Defines function `aggregate_stats`. | CN: 定义函数 `aggregate_stats`。
- **L79** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L80** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。

### Lines 81-100 / 第 81-100 行

````python
    runtime_estimator: RuntimeEstimator,
    sac_estimator: SACEstimator,
    dev: torch.device,
) -> ModuleInfo:
    """
    Collect modulewise stats for a given model, including memory, runtime, and AC tradeoff stats.

    Args:
        model: nn.Module object
        runtime_estimator: RuntimeEstimator object with runtime stats
        mem_tracker: MemTracker object with memory stats
        sac_estimator: SACEstimator object with AC tradeoff stats
        dev: device the model was run on (used to extract memory stats from MemTracker)

    Returns:
        ModuleInfo: A dictionary with module order and module stats.
    """

    # Memory stats
    mod_mem_stats: dict[torch.nn.Module, _ModMemStats] = dict(
````

- **L81** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L82** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L83** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L84** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L85** EN: Starts the docstring for the function aggregate_stats. | CN: 开始定义 function aggregate_stats 的文档字符串。
- **L86** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L87** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L88** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L89** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L90** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L91** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L92** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L93** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L94** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L95** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L96** EN: Continues the docstring text for the function aggregate_stats. | CN: 继续补充 function aggregate_stats 的文档字符串内容。
- **L97** EN: Closes the docstring for the function aggregate_stats. | CN: 结束 function aggregate_stats 的文档字符串。
- **L98** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L99** EN: Keeps the inline comment or directive: Memory stats | CN: 保留这一行注释或指令：Memory stats
- **L100** EN: Assigns or updates `mod_mem_stats`. | CN: 对 `mod_mem_stats` 进行赋值或更新。

### Lines 101-120 / 第 101-120 行

````python
        copy.deepcopy(mem_tracker.memory_tracking)
    )

    # Runtime stats
    mod_runtime_stats: dict[str, ModRuntime] = {
        fqn: {"fw": v["fw"], "bw": v["bw"]}
        for fqn, v in runtime_estimator.mod_runtimes.items()
    }

    # Module order
    mod_order: ModOrder = {
        "fw_pre_order": list(runtime_estimator.mod_fw_pre_order),
        "bw_pre_order": list(runtime_estimator.mod_bw_pre_order),
        "fw_post_order": list(runtime_estimator.mod_fw_post_order),
        "bw_post_order": list(runtime_estimator.mod_bw_post_order),
    }

    # Selective Activation Checkpointing stats
    sac_estimator.pwlf_sac_tradeoff_curve()
    mod_sac_tradeoff_stats: dict[str, SACTradeOffStats] = copy.deepcopy(
````

- **L101** EN: Calls `copy.deepcopy` as part of the current workflow. | CN: 在当前流程中调用 `copy.deepcopy`。
- **L102** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L103** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L104** EN: Keeps the inline comment or directive: Runtime stats | CN: 保留这一行注释或指令：Runtime stats
- **L105** EN: Assigns or updates `mod_runtime_stats`. | CN: 对 `mod_runtime_stats` 进行赋值或更新。
- **L106** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L107** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L108** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L109** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L110** EN: Keeps the inline comment or directive: Module order | CN: 保留这一行注释或指令：Module order
- **L111** EN: Assigns or updates `mod_order`. | CN: 对 `mod_order` 进行赋值或更新。
- **L112** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L113** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L114** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L115** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L116** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L117** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L118** EN: Keeps the inline comment or directive: Selective Activation Checkpointing stats | CN: 保留这一行注释或指令：Selective Activation Checkpointing stats
- **L119** EN: Calls `sac_estimator.pwlf_sac_tradeoff_curve` as part of the current workflow. | CN: 在当前流程中调用 `sac_estimator.pwlf_sac_tradeoff_curve`。
- **L120** EN: Assigns or updates `mod_sac_tradeoff_stats`. | CN: 对 `mod_sac_tradeoff_stats` 进行赋值或更新。

### Lines 121-140 / 第 121-140 行

````python
        sac_estimator.sac_mod_tradeoff_stats
    )

    module_info: ModuleInfo = {
        "mod_order": mod_order,
        "mod_stats": [],
    }

    for mod in model.modules():
        if mod_mem_stat := mod_mem_stats.get(mod):
            if tradeoff_stats := mod_sac_tradeoff_stats.get(mod_mem_stat.mod_fqn):
                sac_runtime = tradeoff_stats.sac_runtime
                sac_memory = tradeoff_stats.sac_memory
                n_segments = tradeoff_stats.n_segments
                slopes = tradeoff_stats.slopes
                intercepts = tradeoff_stats.intercepts
                breakpoints = tradeoff_stats.fit_breaks
                tradeoff_curve = tradeoff_stats.tradeoff_curve
                is_leaf = False
            else:
````

- **L121** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L122** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L123** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L124** EN: Assigns or updates `module_info`. | CN: 对 `module_info` 进行赋值或更新。
- **L125** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L126** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L127** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L128** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L129** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L130** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L131** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L132** EN: Assigns or updates `sac_runtime`. | CN: 对 `sac_runtime` 进行赋值或更新。
- **L133** EN: Assigns or updates `sac_memory`. | CN: 对 `sac_memory` 进行赋值或更新。
- **L134** EN: Assigns or updates `n_segments`. | CN: 对 `n_segments` 进行赋值或更新。
- **L135** EN: Assigns or updates `slopes`. | CN: 对 `slopes` 进行赋值或更新。
- **L136** EN: Assigns or updates `intercepts`. | CN: 对 `intercepts` 进行赋值或更新。
- **L137** EN: Exits the current loop immediately. | CN: 立即退出当前循环。
- **L138** EN: Assigns or updates `tradeoff_curve`. | CN: 对 `tradeoff_curve` 进行赋值或更新。
- **L139** EN: Assigns or updates `is_leaf`. | CN: 对 `is_leaf` 进行赋值或更新。
- **L140** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。

### Lines 141-160 / 第 141-160 行

````python
                sac_runtime = sac_memory = n_segments = 0
                slopes = intercepts = breakpoints = []
                tradeoff_curve: OrderedDict[float, float] = OrderedDict()  # type: ignore[no-redef]
                is_leaf = True
            mod_stat: ModStats = {
                "fqn": mod_mem_stat.mod_fqn,
                "param_per_module": mod_mem_stat.parameter_mem,
                "grad_per_module": mod_mem_stat.parameter_mem,
                "grad_total": mod_mem_stat.snapshots[_ModState.PRE_BW][-1][dev][
                    _MemRefType.GRAD
                ],
                "act_fw_per_module": max(
                    0,
                    mod_mem_stat.snapshots[_ModState.POST_FW][-1][dev][_MemRefType.ACT]
                    - mod_mem_stat.snapshots[_ModState.PRE_FW][-1][dev][_MemRefType.ACT]
                    - mod_mem_stat.output_mem,
                ),
                "act_bw_per_module": max(
                    0,
                    mod_mem_stat.snapshots[_ModState.PEAK_BW][-1][dev][_MemRefType.ACT],
````

- **L141** EN: Assigns or updates `sac_runtime`. | CN: 对 `sac_runtime` 进行赋值或更新。
- **L142** EN: Assigns or updates `slopes`. | CN: 对 `slopes` 进行赋值或更新。
- **L143** EN: Assigns or updates `tradeoff_curve`. | CN: 对 `tradeoff_curve` 进行赋值或更新。
- **L144** EN: Assigns or updates `is_leaf`. | CN: 对 `is_leaf` 进行赋值或更新。
- **L145** EN: Assigns or updates `mod_stat`. | CN: 对 `mod_stat` 进行赋值或更新。
- **L146** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L147** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L148** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L149** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L150** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L151** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L152** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L153** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L154** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L155** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L156** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L157** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L158** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L159** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L160** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。

### Lines 161-180 / 第 161-180 行

````python
                ),
                "act_grad_per_module": (
                    mod_mem_stat.snapshots[_ModState.PEAK_BW][-1][dev][_MemRefType.TEMP]
                    - mod_mem_stat.snapshots[_ModState.PRE_BW][-1][dev][
                        _MemRefType.TEMP
                    ]
                ),
                "act_total": mod_mem_stat.snapshots[_ModState.POST_FW][-1][dev][
                    _MemRefType.ACT
                ],
                "input_per_module": mod_mem_stat.input_mem,
                "output_per_module": mod_mem_stat.output_mem,
                "fw_runtime_per_module": mod_runtime_stats[mod_mem_stat.mod_fqn]["fw"],
                "bw_runtime_per_module": mod_runtime_stats[mod_mem_stat.mod_fqn]["bw"],
                "is_leaf": is_leaf,
                "sac_runtime": sac_runtime,
                "sac_memory": sac_memory,
                "n_segments": n_segments,
                "slopes": slopes,
                "intercepts": intercepts,
````

- **L161** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L162** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L163** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L164** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L165** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L166** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L167** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L168** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L169** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L170** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L171** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L172** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L173** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L174** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L175** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L176** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L177** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L178** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L179** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L180** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。

### Lines 181-200 / 第 181-200 行

````python
                "breakpoints": breakpoints,
                "tradeoff_curve": tradeoff_curve,
            }
            module_info["mod_stats"].append(mod_stat)

    return module_info


class Node(ModStats):
    index: int  # index according to forward pre-order
    pos_fw_post_order: int  # index according to forward post-order


class Graph:
    def __init__(self, n: int) -> None:
        self.nodes: list[Node] = []
        self.name2node: dict[str, Node] = {}
        self.ad_matrix = np.zeros((n, n))
        self.fw_post_order: list[str] = []

````

- **L181** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L182** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L183** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L184** EN: Continues the implementation inside function `aggregate_stats`. | CN: 继续说明函数 `aggregate_stats` 内部的实现。
- **L185** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L186** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L187** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L188** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L189** EN: Defines class `Node`. | CN: 定义类 `Node`。
- **L190** EN: Continues the implementation inside class `Node`. | CN: 继续说明类 `Node` 内部的实现。
- **L191** EN: Continues the implementation inside class `Node`. | CN: 继续说明类 `Node` 内部的实现。
- **L192** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L193** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L194** EN: Defines class `Graph`. | CN: 定义类 `Graph`。
- **L195** EN: Defines function `__init__`. | CN: 定义函数 `__init__`。
- **L196** EN: Assigns or updates `self.nodes`. | CN: 对 `self.nodes` 进行赋值或更新。
- **L197** EN: Assigns or updates `self.name2node`. | CN: 对 `self.name2node` 进行赋值或更新。
- **L198** EN: Assigns or updates `self.ad_matrix`. | CN: 对 `self.ad_matrix` 进行赋值或更新。
- **L199** EN: Assigns or updates `self.fw_post_order`. | CN: 对 `self.fw_post_order` 进行赋值或更新。
- **L200** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。

### Lines 201-220 / 第 201-220 行

````python
    def add_node(self, node: Node) -> None:
        self.nodes.append(node)
        self.name2node[node["fqn"]] = node


def parse_module_info(module_info: ModuleInfo) -> Graph:
    """
    Parse module info and create a graph (tree) of modules. The graph will be
    used by MILP solver to find optimal SAC and/or FSDP configurations.
    """
    mod_stats = module_info["mod_stats"]
    fw_pre_order = module_info["mod_order"]["fw_pre_order"]
    # assertion and number of nodes
    if len(mod_stats) != len(fw_pre_order):
        raise AssertionError
    n_nodes = len(mod_stats)

    # create graph
    g = Graph(n_nodes)
    g.fw_post_order = module_info["mod_order"]["fw_post_order"]
````

- **L201** EN: Defines function `add_node`. | CN: 定义函数 `add_node`。
- **L202** EN: Calls `self.nodes.append` as part of the current workflow. | CN: 在当前流程中调用 `self.nodes.append`。
- **L203** EN: Continues the implementation inside function `add_node`. | CN: 继续说明函数 `add_node` 内部的实现。
- **L204** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L205** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L206** EN: Defines function `parse_module_info`. | CN: 定义函数 `parse_module_info`。
- **L207** EN: Starts the docstring for the function parse_module_info. | CN: 开始定义 function parse_module_info 的文档字符串。
- **L208** EN: Continues the docstring text for the function parse_module_info. | CN: 继续补充 function parse_module_info 的文档字符串内容。
- **L209** EN: Continues the docstring text for the function parse_module_info. | CN: 继续补充 function parse_module_info 的文档字符串内容。
- **L210** EN: Closes the docstring for the function parse_module_info. | CN: 结束 function parse_module_info 的文档字符串。
- **L211** EN: Assigns or updates `mod_stats`. | CN: 对 `mod_stats` 进行赋值或更新。
- **L212** EN: Assigns or updates `fw_pre_order`. | CN: 对 `fw_pre_order` 进行赋值或更新。
- **L213** EN: Keeps the inline comment or directive: assertion and number of nodes | CN: 保留这一行注释或指令：assertion and number of nodes
- **L214** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L215** EN: Raises an exception to signal an error or unsupported path. | CN: 抛出异常，用于报告错误或不支持的路径。
- **L216** EN: Assigns or updates `n_nodes`. | CN: 对 `n_nodes` 进行赋值或更新。
- **L217** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L218** EN: Keeps the inline comment or directive: create graph | CN: 保留这一行注释或指令：create graph
- **L219** EN: Assigns or updates `g`. | CN: 对 `g` 进行赋值或更新。
- **L220** EN: Assigns or updates `g.fw_post_order`. | CN: 对 `g.fw_post_order` 进行赋值或更新。

### Lines 221-240 / 第 221-240 行

````python

    # sort the modules by pre-order and add them to the graph
    module_info["mod_stats"] = sorted(
        mod_stats, key=lambda x: fw_pre_order.index(x["fqn"])
    )
    for i, one_mod_stats in enumerate(mod_stats):
        node: Node = cast(Node, one_mod_stats)
        node["index"] = i
        node["pos_fw_post_order"] = g.fw_post_order.index(node["fqn"])
        g.add_node(node)

    # set up ancestor-descendant matrix
    for i in range(n_nodes):
        for j in range(i, n_nodes):
            if is_self_or_submodule(g.nodes[j]["fqn"], g.nodes[i]["fqn"]):
                g.ad_matrix[i][j] = 1
            else:
                break

    return g
````

- **L221** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L222** EN: Keeps the inline comment or directive: sort the modules by pre-order and add them to the graph | CN: 保留这一行注释或指令：sort the modules by pre-order and add them to the graph
- **L223** EN: Continues the implementation inside function `parse_module_info`. | CN: 继续说明函数 `parse_module_info` 内部的实现。
- **L224** EN: Assigns or updates `mod_stats, key`. | CN: 对 `mod_stats, key` 进行赋值或更新。
- **L225** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L226** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L227** EN: Assigns or updates `node`. | CN: 对 `node` 进行赋值或更新。
- **L228** EN: Continues the implementation inside function `parse_module_info`. | CN: 继续说明函数 `parse_module_info` 内部的实现。
- **L229** EN: Continues the implementation inside function `parse_module_info`. | CN: 继续说明函数 `parse_module_info` 内部的实现。
- **L230** EN: Calls `g.add_node` as part of the current workflow. | CN: 在当前流程中调用 `g.add_node`。
- **L231** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L232** EN: Keeps the inline comment or directive: set up ancestor-descendant matrix | CN: 保留这一行注释或指令：set up ancestor-descendant matrix
- **L233** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L234** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L235** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L236** EN: Assigns or updates `g.ad_matrix[i][j]`. | CN: 对 `g.ad_matrix[i][j]` 进行赋值或更新。
- **L237** EN: Defines the fallback branch of the condition. | CN: 定义条件判断的兜底分支。
- **L238** EN: Exits the current loop immediately. | CN: 立即退出当前循环。
- **L239** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L240** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

### Lines 241-260 / 第 241-260 行

````python


def is_self_or_submodule(name_descendant: str, name_ancestor: str) -> bool:
    """
    check if name_descendant is a submodule of name_ancestor, or if they are the same
    """
    return name_descendant == name_ancestor or name_ancestor + "." in name_descendant


def is_submodule(name_descendant: str, name_ancestor: str) -> bool:
    """
    if name_descendant is a submodule of name_ancestor, but not the same
    """
    return name_ancestor + "." in name_descendant


def display_bytes(b: int, unit: str = "MiB") -> str:
    """
    return a string that represent the number of bytes in a desired unit
    """
````

- **L241** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L242** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L243** EN: Defines function `is_self_or_submodule`. | CN: 定义函数 `is_self_or_submodule`。
- **L244** EN: Starts the docstring for the function is_self_or_submodule. | CN: 开始定义 function is_self_or_submodule 的文档字符串。
- **L245** EN: Continues the docstring text for the function is_self_or_submodule. | CN: 继续补充 function is_self_or_submodule 的文档字符串内容。
- **L246** EN: Closes the docstring for the function is_self_or_submodule. | CN: 结束 function is_self_or_submodule 的文档字符串。
- **L247** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L248** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L249** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L250** EN: Defines function `is_submodule`. | CN: 定义函数 `is_submodule`。
- **L251** EN: Starts the docstring for the function is_submodule. | CN: 开始定义 function is_submodule 的文档字符串。
- **L252** EN: Continues the docstring text for the function is_submodule. | CN: 继续补充 function is_submodule 的文档字符串内容。
- **L253** EN: Closes the docstring for the function is_submodule. | CN: 结束 function is_submodule 的文档字符串。
- **L254** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L255** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L256** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L257** EN: Defines function `display_bytes`. | CN: 定义函数 `display_bytes`。
- **L258** EN: Starts the docstring for the function display_bytes. | CN: 开始定义 function display_bytes 的文档字符串。
- **L259** EN: Continues the docstring text for the function display_bytes. | CN: 继续补充 function display_bytes 的文档字符串内容。
- **L260** EN: Closes the docstring for the function display_bytes. | CN: 结束 function display_bytes 的文档字符串。

### Lines 261-280 / 第 261-280 行

````python
    if unit == "KiB":
        return f"{b / 2**10:.2f} KiB"
    if unit == "MiB":
        return f"{b / 2**20:.2f} MiB"
    if unit == "GiB":
        return f"{b / 2**30:.2f} GiB"
    return f"{b:.2f} bytes"


def get_peak_memory_runtime_baseline(graph: Graph) -> tuple[int, float]:
    """
    Get the baseline peak memory and runtime.
    Baseline here means there is no FSDP or AC.
    Memory includes the parameters, gradients, activations, and activation gradients.
    Memory does not include e.g., optimizer states, embedding tables, etc.

    Returns:
        int: peak memory in bytes
        float: compute time in ms
    """
````

- **L261** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L262** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L263** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L264** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L265** EN: Starts a conditional branch. | CN: 开始一个条件分支。
- **L266** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L267** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。
- **L268** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L269** EN: Blank line separating logical code blocks. | CN: 空行，用于分隔逻辑代码块。
- **L270** EN: Defines function `get_peak_memory_runtime_baseline`. | CN: 定义函数 `get_peak_memory_runtime_baseline`。
- **L271** EN: Starts the docstring for the function get_peak_memory_runtime_baseline. | CN: 开始定义 function get_peak_memory_runtime_baseline 的文档字符串。
- **L272** EN: Continues the docstring text for the function get_peak_memory_runtime_baseline. | CN: 继续补充 function get_peak_memory_runtime_baseline 的文档字符串内容。
- **L273** EN: Continues the docstring text for the function get_peak_memory_runtime_baseline. | CN: 继续补充 function get_peak_memory_runtime_baseline 的文档字符串内容。
- **L274** EN: Continues the docstring text for the function get_peak_memory_runtime_baseline. | CN: 继续补充 function get_peak_memory_runtime_baseline 的文档字符串内容。
- **L275** EN: Continues the docstring text for the function get_peak_memory_runtime_baseline. | CN: 继续补充 function get_peak_memory_runtime_baseline 的文档字符串内容。
- **L276** EN: Continues the docstring text for the function get_peak_memory_runtime_baseline. | CN: 继续补充 function get_peak_memory_runtime_baseline 的文档字符串内容。
- **L277** EN: Continues the docstring text for the function get_peak_memory_runtime_baseline. | CN: 继续补充 function get_peak_memory_runtime_baseline 的文档字符串内容。
- **L278** EN: Continues the docstring text for the function get_peak_memory_runtime_baseline. | CN: 继续补充 function get_peak_memory_runtime_baseline 的文档字符串内容。
- **L279** EN: Continues the docstring text for the function get_peak_memory_runtime_baseline. | CN: 继续补充 function get_peak_memory_runtime_baseline 的文档字符串内容。
- **L280** EN: Closes the docstring for the function get_peak_memory_runtime_baseline. | CN: 结束 function get_peak_memory_runtime_baseline 的文档字符串。

### Lines 281-293 / 第 281-293 行

````python
    P_1 = graph.nodes[0]["param_per_module"]
    num_nodes = len(graph.nodes)
    peak_mem = 0
    for i in range(num_nodes):
        TG_i = graph.nodes[i]["grad_total"]
        AG_i = graph.nodes[i]["act_grad_per_module"]
        TA_i = graph.nodes[i]["act_total"]
        peak_mem = max(peak_mem, P_1 + TG_i + AG_i + TA_i)
    compute_time = (
        graph.nodes[0]["fw_runtime_per_module"]
        + graph.nodes[0]["bw_runtime_per_module"]
    )
    return (peak_mem, compute_time)
````

- **L281** EN: Assigns or updates `P_1`. | CN: 对 `P_1` 进行赋值或更新。
- **L282** EN: Assigns or updates `num_nodes`. | CN: 对 `num_nodes` 进行赋值或更新。
- **L283** EN: Assigns or updates `peak_mem`. | CN: 对 `peak_mem` 进行赋值或更新。
- **L284** EN: Starts a loop over an iterable input. | CN: 开始遍历可迭代对象的循环。
- **L285** EN: Assigns or updates `TG_i`. | CN: 对 `TG_i` 进行赋值或更新。
- **L286** EN: Assigns or updates `AG_i`. | CN: 对 `AG_i` 进行赋值或更新。
- **L287** EN: Assigns or updates `TA_i`. | CN: 对 `TA_i` 进行赋值或更新。
- **L288** EN: Assigns or updates `peak_mem`. | CN: 对 `peak_mem` 进行赋值或更新。
- **L289** EN: Assigns or updates `compute_time`. | CN: 对 `compute_time` 进行赋值或更新。
- **L290** EN: Continues the implementation inside function `get_peak_memory_runtime_baseline`. | CN: 继续说明函数 `get_peak_memory_runtime_baseline` 内部的实现。
- **L291** EN: Continues the implementation inside function `get_peak_memory_runtime_baseline`. | CN: 继续说明函数 `get_peak_memory_runtime_baseline` 内部的实现。
- **L292** EN: Closes the surrounding multi-line expression or container. | CN: 结束外围的多行表达式或容器。
- **L293** EN: Returns a value or exits the current callable. | CN: 返回结果，或结束当前可调用对象的执行。

## Key Concepts / 关键概念

- **EN**: distributed runtime helpers, APIs, and package wiring  
  **CN**: 分布式运行时辅助逻辑、API 与包级导出
- **EN**: checkpointing  
  **CN**: 检查点
- **EN**: optimizer coordination  
  **CN**: 优化器协同
- **EN**: Primary classes: ModOrder, ModRuntime, ModStats, ModuleInfo, Node  
  **CN**: 主要类：ModOrder, ModRuntime, ModStats, ModuleInfo, Node
- **EN**: Core callables: aggregate_stats, parse_module_info, is_self_or_submodule, is_submodule, display_bytes  
  **CN**: 核心可调用对象：aggregate_stats, parse_module_info, is_self_or_submodule, is_submodule, display_bytes

## Dependencies / 依赖关系

- **Internal / 内部**: `torch.distributed._tools.mem_tracker`, `torch.distributed._tools.runtime_estimator`, `torch.distributed._tools.sac_estimator`
- **PyTorch / PyTorch**: `torch`
- **Python Stdlib / Python 标准库**: `collections`, `copy`, `typing`
- **Third-party / 第三方**: `numpy`

