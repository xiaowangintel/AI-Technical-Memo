# generateTriplets.py — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/utils/mlgo-utils/IR2Vec/generateTriplets.py` | `llvm/utils/mlgo-utils/IR2Vec/generateTriplets.py` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | IR2Vec/MIR2Vec Triplet Generator Generates IR2Vec or MIR2Vec triplets by applying random optimization levels to LLVM IR files (or processing MIR files) and extracting triplets using llvm-ir2vec. Automatically generate... | 实现 MLGO 工具脚本，用于语料生成、提取或面向模型的流程。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-14

````python
# Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.
# See https://llvm.org/LICENSE.txt for license information.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
"""IR2Vec/MIR2Vec Triplet Generator

Generates IR2Vec or MIR2Vec triplets by applying random optimization levels to
LLVM IR files (or processing MIR files) and extracting triplets using llvm-ir2vec.
Automatically generates preprocessed files (entity2id.txt, relation2id.txt, and
train2id.txt) necessary for training IR2Vec or MIR2Vec vocabularies.

Usage:
    For LLVM IR:
        python generateTriplets.py <llvm_build_dir> <num_optimizations> <ll_file_list> <output_dir>

````
- **L1 EN**: Comment documents nearby script behavior: `Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`.
  **L1 CN**: 注释说明了附近脚本逻辑：`Part of the LLVM Project, under the Apache License v2.0 with LLVM Exceptions.`。
- **L2 EN**: Comment documents nearby script behavior: `See https://llvm.org/LICENSE.txt for license information.`.
  **L2 CN**: 注释说明了附近脚本逻辑：`See https://llvm.org/LICENSE.txt for license information.`。
- **L3 EN**: Comment documents nearby script behavior: `SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`.
  **L3 CN**: 注释说明了附近脚本逻辑：`SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception`。
- **L4 EN**: Executes Python statement `"""IR2Vec/MIR2Vec Triplet Generator`.
  **L4 CN**: 执行 Python 语句 `"""IR2Vec/MIR2Vec Triplet Generator`。
- **L5 EN**: Blank line separates nearby declarations or logic blocks.
  **L5 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L6 EN**: Executes Python statement `Generates IR2Vec or MIR2Vec triplets by applying random optimization levels to`.
  **L6 CN**: 执行 Python 语句 `Generates IR2Vec or MIR2Vec triplets by applying random optimization levels to`。
- **L7 EN**: Executes Python statement `LLVM IR files (or processing MIR files) and extracting triplets using llvm-ir2vec.`.
  **L7 CN**: 执行 Python 语句 `LLVM IR files (or processing MIR files) and extracting triplets using llvm-ir2vec.`。
- **L8 EN**: Executes Python statement `Automatically generates preprocessed files (entity2id.txt, relation2id.txt, and`.
  **L8 CN**: 执行 Python 语句 `Automatically generates preprocessed files (entity2id.txt, relation2id.txt, and`。
- **L9 EN**: Executes Python statement `train2id.txt) necessary for training IR2Vec or MIR2Vec vocabularies.`.
  **L9 CN**: 执行 Python 语句 `train2id.txt) necessary for training IR2Vec or MIR2Vec vocabularies.`。
- **L10 EN**: Blank line separates nearby declarations or logic blocks.
  **L10 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L11 EN**: Executes Python statement `Usage:`.
  **L11 CN**: 执行 Python 语句 `Usage:`。
- **L12 EN**: Executes Python statement `For LLVM IR:`.
  **L12 CN**: 执行 Python 语句 `For LLVM IR:`。
- **L13 EN**: Executes Python statement `python generateTriplets.py <llvm_build_dir> <num_optimizations> <ll_file_list> <output_...`.
  **L13 CN**: 执行 Python 语句 `python generateTriplets.py <llvm_build_dir> <num_optimizations> <ll_file_list> <output_...`。
- **L14 EN**: Blank line separates nearby declarations or logic blocks.
  **L14 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 15-27

````python
    For Machine IR:
        python generateTriplets.py --mode=mir <llvm_build_dir> <mir_file_list> <output_dir>
"""

import argparse
import logging
import os
import random
import subprocess
from concurrent.futures import ThreadPoolExecutor, as_completed
from pathlib import Path
from typing import List, Set, Tuple

````
- **L15 EN**: Executes Python statement `For Machine IR:`.
  **L15 CN**: 执行 Python 语句 `For Machine IR:`。
- **L16 EN**: Assigns or updates `python generateTriplets.py --mode`.
  **L16 CN**: 对 `python generateTriplets.py --mode` 进行赋值或更新。
- **L17 EN**: Executes Python statement `"""`.
  **L17 CN**: 执行 Python 语句 `"""`。
- **L18 EN**: Blank line separates nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Imports Python module(s) `argparse` for supporting functionality.
  **L19 CN**: 导入 Python 模块 `argparse` 以提供辅助功能。
- **L20 EN**: Imports Python module(s) `logging` for supporting functionality.
  **L20 CN**: 导入 Python 模块 `logging` 以提供辅助功能。
- **L21 EN**: Imports Python module(s) `os` for supporting functionality.
  **L21 CN**: 导入 Python 模块 `os` 以提供辅助功能。
- **L22 EN**: Imports Python module(s) `random` for supporting functionality.
  **L22 CN**: 导入 Python 模块 `random` 以提供辅助功能。
- **L23 EN**: Imports Python module(s) `subprocess` for supporting functionality.
  **L23 CN**: 导入 Python 模块 `subprocess` 以提供辅助功能。
- **L24 EN**: Imports `ThreadPoolExecutor, as_completed` from module `concurrent.futures`.
  **L24 CN**: 从模块 `concurrent.futures` 导入 `ThreadPoolExecutor, as_completed`。
- **L25 EN**: Imports `Path` from module `pathlib`.
  **L25 CN**: 从模块 `pathlib` 导入 `Path`。
- **L26 EN**: Imports `List, Set, Tuple` from module `typing`.
  **L26 CN**: 从模块 `typing` 导入 `List, Set, Tuple`。
- **L27 EN**: Blank line separates nearby declarations or logic blocks.
  **L27 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 28-40

````python
# Configuration
OPT_LEVELS = ["O0", "O1", "O2", "O3", "Os", "Oz"]
DEFAULT_MAX_WORKERS = 100

logger = logging.getLogger(__name__)


# TODO: Change this to a dataclass with slots
# when Python 3.10+ is the minimum version
# https://docs.python.org/3/library/dataclasses.html#dataclasses.dataclass
class TripletResult:
    """Result from processing a single LLVM IR file"""

````
- **L28 EN**: Comment documents nearby script behavior: `Configuration`.
  **L28 CN**: 注释说明了附近脚本逻辑：`Configuration`。
- **L29 EN**: Assigns or updates `OPT_LEVELS`.
  **L29 CN**: 对 `OPT_LEVELS` 进行赋值或更新。
- **L30 EN**: Assigns or updates `DEFAULT_MAX_WORKERS`.
  **L30 CN**: 对 `DEFAULT_MAX_WORKERS` 进行赋值或更新。
- **L31 EN**: Blank line separates nearby declarations or logic blocks.
  **L31 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L32 EN**: Assigns or updates `logger`.
  **L32 CN**: 对 `logger` 进行赋值或更新。
- **L33 EN**: Blank line separates nearby declarations or logic blocks.
  **L33 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L34 EN**: Blank line separates nearby declarations or logic blocks.
  **L34 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L35 EN**: Comment documents nearby script behavior: `TODO: Change this to a dataclass with slots`.
  **L35 CN**: 注释说明了附近脚本逻辑：`TODO: Change this to a dataclass with slots`。
- **L36 EN**: Comment documents nearby script behavior: `when Python 3.10+ is the minimum version`.
  **L36 CN**: 注释说明了附近脚本逻辑：`when Python 3.10+ is the minimum version`。
- **L37 EN**: Comment documents nearby script behavior: `https://docs.python.org/3/library/dataclasses.html#dataclasses.dataclass`.
  **L37 CN**: 注释说明了附近脚本逻辑：`https://docs.python.org/3/library/dataclasses.html#dataclasses.dataclass`。
- **L38 EN**: Declares class `TripletResult` to group related state and behavior.
  **L38 CN**: 声明类 `TripletResult`，用于组织相关状态与行为。
- **L39 EN**: Executes Python statement `"""Result from processing a single LLVM IR file"""`.
  **L39 CN**: 执行 Python 语句 `"""Result from processing a single LLVM IR file"""`。
- **L40 EN**: Blank line separates nearby declarations or logic blocks.
  **L40 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 41-64

````python
    __slots__ = ["triplets", "max_relation"]

    def __init__(self, triplets: Set[str], max_relation: int):
        self.triplets = triplets
        self.max_relation = max_relation


class IR2VecTripletGenerator:
    """Main class for generating IR2Vec or MIR2Vec triplets"""

    def __init__(
        self,
        llvm_build_dir: Path,
        num_optimizations: int,
        output_dir: Path,
        max_workers: int = DEFAULT_MAX_WORKERS,
        mode: str = "llvm",
    ):
        self.llvm_build_dir = llvm_build_dir
        self.num_optimizations = num_optimizations
        self.output_dir = output_dir
        self.max_workers = max_workers
        self.mode = mode  # "llvm" or "mir"

````
- **L41 EN**: Assigns or updates `__slots__`.
  **L41 CN**: 对 `__slots__` 进行赋值或更新。
- **L42 EN**: Blank line separates nearby declarations or logic blocks.
  **L42 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L43 EN**: Declares function `__init__`.
  **L43 CN**: 声明函数 `__init__`。
- **L44 EN**: Assigns or updates `self.triplets`.
  **L44 CN**: 对 `self.triplets` 进行赋值或更新。
- **L45 EN**: Assigns or updates `self.max_relation`.
  **L45 CN**: 对 `self.max_relation` 进行赋值或更新。
- **L46 EN**: Blank line separates nearby declarations or logic blocks.
  **L46 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L47 EN**: Blank line separates nearby declarations or logic blocks.
  **L47 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L48 EN**: Declares class `IR2VecTripletGenerator` to group related state and behavior.
  **L48 CN**: 声明类 `IR2VecTripletGenerator`，用于组织相关状态与行为。
- **L49 EN**: Executes Python statement `"""Main class for generating IR2Vec or MIR2Vec triplets"""`.
  **L49 CN**: 执行 Python 语句 `"""Main class for generating IR2Vec or MIR2Vec triplets"""`。
- **L50 EN**: Blank line separates nearby declarations or logic blocks.
  **L50 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L51 EN**: Declares function `__init__`.
  **L51 CN**: 声明函数 `__init__`。
- **L52 EN**: Executes Python statement `self,`.
  **L52 CN**: 执行 Python 语句 `self,`。
- **L53 EN**: Executes Python statement `llvm_build_dir: Path,`.
  **L53 CN**: 执行 Python 语句 `llvm_build_dir: Path,`。
- **L54 EN**: Executes Python statement `num_optimizations: int,`.
  **L54 CN**: 执行 Python 语句 `num_optimizations: int,`。
- **L55 EN**: Executes Python statement `output_dir: Path,`.
  **L55 CN**: 执行 Python 语句 `output_dir: Path,`。
- **L56 EN**: Assigns or updates `max_workers: int`.
  **L56 CN**: 对 `max_workers: int` 进行赋值或更新。
- **L57 EN**: Assigns or updates `mode: str`.
  **L57 CN**: 对 `mode: str` 进行赋值或更新。
- **L58 EN**: Executes Python statement `):`.
  **L58 CN**: 执行 Python 语句 `):`。
- **L59 EN**: Assigns or updates `self.llvm_build_dir`.
  **L59 CN**: 对 `self.llvm_build_dir` 进行赋值或更新。
- **L60 EN**: Assigns or updates `self.num_optimizations`.
  **L60 CN**: 对 `self.num_optimizations` 进行赋值或更新。
- **L61 EN**: Assigns or updates `self.output_dir`.
  **L61 CN**: 对 `self.output_dir` 进行赋值或更新。
- **L62 EN**: Assigns or updates `self.max_workers`.
  **L62 CN**: 对 `self.max_workers` 进行赋值或更新。
- **L63 EN**: Assigns or updates `self.mode`.
  **L63 CN**: 对 `self.mode` 进行赋值或更新。
- **L64 EN**: Blank line separates nearby declarations or logic blocks.
  **L64 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 65-80

````python
        # Tool paths
        self.opt_binary = os.path.join(llvm_build_dir, "bin", "opt")
        self.ir2vec_binary = os.path.join(llvm_build_dir, "bin", "llvm-ir2vec")

        self._validate_setup()

        # Create output directory if it doesn't exist
        self.output_dir.mkdir(parents=True, exist_ok=True)

    def _validate_setup(self):
        """Validate that all required tools and paths exist"""
        if not self.llvm_build_dir.exists():
            raise FileNotFoundError(
                f"LLVM build directory not found: {self.llvm_build_dir}"
            )

````
- **L65 EN**: Comment documents nearby script behavior: `Tool paths`.
  **L65 CN**: 注释说明了附近脚本逻辑：`Tool paths`。
- **L66 EN**: Assigns or updates `self.opt_binary`.
  **L66 CN**: 对 `self.opt_binary` 进行赋值或更新。
- **L67 EN**: Assigns or updates `self.ir2vec_binary`.
  **L67 CN**: 对 `self.ir2vec_binary` 进行赋值或更新。
- **L68 EN**: Blank line separates nearby declarations or logic blocks.
  **L68 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L69 EN**: Executes Python statement `self._validate_setup()`.
  **L69 CN**: 执行 Python 语句 `self._validate_setup()`。
- **L70 EN**: Blank line separates nearby declarations or logic blocks.
  **L70 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L71 EN**: Comment documents nearby script behavior: `Create output directory if it doesn't exist`.
  **L71 CN**: 注释说明了附近脚本逻辑：`Create output directory if it doesn't exist`。
- **L72 EN**: Assigns or updates `self.output_dir.mkdir(parents`.
  **L72 CN**: 对 `self.output_dir.mkdir(parents` 进行赋值或更新。
- **L73 EN**: Blank line separates nearby declarations or logic blocks.
  **L73 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L74 EN**: Declares function `_validate_setup`.
  **L74 CN**: 声明函数 `_validate_setup`。
- **L75 EN**: Executes Python statement `"""Validate that all required tools and paths exist"""`.
  **L75 CN**: 执行 Python 语句 `"""Validate that all required tools and paths exist"""`。
- **L76 EN**: Controls Python flow with `if` logic.
  **L76 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L77 EN**: Raises an exception to signal an error path.
  **L77 CN**: 抛出异常以显式表示错误路径。
- **L78 EN**: Executes Python statement `f"LLVM build directory not found: {self.llvm_build_dir}"`.
  **L78 CN**: 执行 Python 语句 `f"LLVM build directory not found: {self.llvm_build_dir}"`。
- **L79 EN**: Executes Python statement `)`.
  **L79 CN**: 执行 Python 语句 `)`。
- **L80 EN**: Blank line separates nearby declarations or logic blocks.
  **L80 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 81-94

````python
        if not os.path.isfile(self.opt_binary) or not os.access(
            self.opt_binary, os.X_OK
        ):
            raise FileNotFoundError(
                f"opt binary not found or not executable: {self.opt_binary}"
            )

        if not os.path.isfile(self.ir2vec_binary) or not os.access(
            self.ir2vec_binary, os.X_OK
        ):
            raise FileNotFoundError(
                f"llvm-ir2vec binary not found or not executable: {self.ir2vec_binary}"
            )

````
- **L81 EN**: Controls Python flow with `if` logic.
  **L81 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L82 EN**: Executes Python statement `self.opt_binary, os.X_OK`.
  **L82 CN**: 执行 Python 语句 `self.opt_binary, os.X_OK`。
- **L83 EN**: Executes Python statement `):`.
  **L83 CN**: 执行 Python 语句 `):`。
- **L84 EN**: Raises an exception to signal an error path.
  **L84 CN**: 抛出异常以显式表示错误路径。
- **L85 EN**: Executes Python statement `f"opt binary not found or not executable: {self.opt_binary}"`.
  **L85 CN**: 执行 Python 语句 `f"opt binary not found or not executable: {self.opt_binary}"`。
- **L86 EN**: Executes Python statement `)`.
  **L86 CN**: 执行 Python 语句 `)`。
- **L87 EN**: Blank line separates nearby declarations or logic blocks.
  **L87 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L88 EN**: Controls Python flow with `if` logic.
  **L88 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L89 EN**: Executes Python statement `self.ir2vec_binary, os.X_OK`.
  **L89 CN**: 执行 Python 语句 `self.ir2vec_binary, os.X_OK`。
- **L90 EN**: Executes Python statement `):`.
  **L90 CN**: 执行 Python 语句 `):`。
- **L91 EN**: Raises an exception to signal an error path.
  **L91 CN**: 抛出异常以显式表示错误路径。
- **L92 EN**: Executes Python statement `f"llvm-ir2vec binary not found or not executable: {self.ir2vec_binary}"`.
  **L92 CN**: 执行 Python 语句 `f"llvm-ir2vec binary not found or not executable: {self.ir2vec_binary}"`。
- **L93 EN**: Executes Python statement `)`.
  **L93 CN**: 执行 Python 语句 `)`。
- **L94 EN**: Blank line separates nearby declarations or logic blocks.
  **L94 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 95-107

````python
        if self.mode not in ["llvm", "mir"]:
            raise ValueError(f"Mode must be 'llvm' or 'mir', got: {self.mode}")

        # For LLVM IR mode, validate optimization count
        if self.mode == "llvm" and not (1 <= self.num_optimizations <= len(OPT_LEVELS)):
            raise ValueError(
                f"Number of optimizations must be between 1-{len(OPT_LEVELS)}"
            )

    def _select_optimization_levels(self) -> List[str]:
        """Select unique random optimization levels"""
        return random.sample(OPT_LEVELS, self.num_optimizations)

````
- **L95 EN**: Controls Python flow with `if` logic.
  **L95 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L96 EN**: Raises an exception to signal an error path.
  **L96 CN**: 抛出异常以显式表示错误路径。
- **L97 EN**: Blank line separates nearby declarations or logic blocks.
  **L97 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L98 EN**: Comment documents nearby script behavior: `For LLVM IR mode, validate optimization count`.
  **L98 CN**: 注释说明了附近脚本逻辑：`For LLVM IR mode, validate optimization count`。
- **L99 EN**: Controls Python flow with `if` logic.
  **L99 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L100 EN**: Raises an exception to signal an error path.
  **L100 CN**: 抛出异常以显式表示错误路径。
- **L101 EN**: Executes Python statement `f"Number of optimizations must be between 1-{len(OPT_LEVELS)}"`.
  **L101 CN**: 执行 Python 语句 `f"Number of optimizations must be between 1-{len(OPT_LEVELS)}"`。
- **L102 EN**: Executes Python statement `)`.
  **L102 CN**: 执行 Python 语句 `)`。
- **L103 EN**: Blank line separates nearby declarations or logic blocks.
  **L103 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L104 EN**: Declares function `_select_optimization_levels`.
  **L104 CN**: 声明函数 `_select_optimization_levels`。
- **L105 EN**: Executes Python statement `"""Select unique random optimization levels"""`.
  **L105 CN**: 执行 Python 语句 `"""Select unique random optimization levels"""`。
- **L106 EN**: Returns a value or exits the current function.
  **L106 CN**: 返回一个值或结束当前函数。
- **L107 EN**: Blank line separates nearby declarations or logic blocks.
  **L107 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 108-131

````python
    def _process_single_file(self, input_file: Path) -> TripletResult:
        """Process a single LLVM IR or MIR file"""
        all_triplets = set()
        max_relation = 1

        if self.mode == "mir":
            # For MIR files, process directly without optimization
            triplets, file_max_relation = self._run_mir_pipeline(input_file)
            if triplets:
                all_triplets.update(triplets)
                max_relation = max(max_relation, file_max_relation)
                logger.debug(f"Generated {len(triplets)} triplets for {input_file}")
        else:
            # For LLVM IR files, apply multiple optimization levels
            opt_levels = self._select_optimization_levels()
            for opt_level in opt_levels:
                triplets, file_max_relation = self._run_pipeline(input_file, opt_level)
                if triplets:
                    all_triplets.update(triplets)
                    max_relation = max(max_relation, file_max_relation)
                    logger.debug(
                        f"Generated {len(triplets)} triplets for {input_file} with {opt_level}"
                    )

````
- **L108 EN**: Declares function `_process_single_file`.
  **L108 CN**: 声明函数 `_process_single_file`。
- **L109 EN**: Executes Python statement `"""Process a single LLVM IR or MIR file"""`.
  **L109 CN**: 执行 Python 语句 `"""Process a single LLVM IR or MIR file"""`。
- **L110 EN**: Assigns or updates `all_triplets`.
  **L110 CN**: 对 `all_triplets` 进行赋值或更新。
- **L111 EN**: Assigns or updates `max_relation`.
  **L111 CN**: 对 `max_relation` 进行赋值或更新。
- **L112 EN**: Blank line separates nearby declarations or logic blocks.
  **L112 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L113 EN**: Controls Python flow with `if` logic.
  **L113 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L114 EN**: Comment documents nearby script behavior: `For MIR files, process directly without optimization`.
  **L114 CN**: 注释说明了附近脚本逻辑：`For MIR files, process directly without optimization`。
- **L115 EN**: Assigns or updates `triplets, file_max_relation`.
  **L115 CN**: 对 `triplets, file_max_relation` 进行赋值或更新。
- **L116 EN**: Controls Python flow with `if` logic.
  **L116 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L117 EN**: Executes Python statement `all_triplets.update(triplets)`.
  **L117 CN**: 执行 Python 语句 `all_triplets.update(triplets)`。
- **L118 EN**: Assigns or updates `max_relation`.
  **L118 CN**: 对 `max_relation` 进行赋值或更新。
- **L119 EN**: Executes Python statement `logger.debug(f"Generated {len(triplets)} triplets for {input_file}")`.
  **L119 CN**: 执行 Python 语句 `logger.debug(f"Generated {len(triplets)} triplets for {input_file}")`。
- **L120 EN**: Controls Python flow with `else` logic.
  **L120 CN**: 使用 `else` 逻辑控制 Python 执行流程。
- **L121 EN**: Comment documents nearby script behavior: `For LLVM IR files, apply multiple optimization levels`.
  **L121 CN**: 注释说明了附近脚本逻辑：`For LLVM IR files, apply multiple optimization levels`。
- **L122 EN**: Assigns or updates `opt_levels`.
  **L122 CN**: 对 `opt_levels` 进行赋值或更新。
- **L123 EN**: Controls Python flow with `for` logic.
  **L123 CN**: 使用 `for` 逻辑控制 Python 执行流程。
- **L124 EN**: Assigns or updates `triplets, file_max_relation`.
  **L124 CN**: 对 `triplets, file_max_relation` 进行赋值或更新。
- **L125 EN**: Controls Python flow with `if` logic.
  **L125 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L126 EN**: Executes Python statement `all_triplets.update(triplets)`.
  **L126 CN**: 执行 Python 语句 `all_triplets.update(triplets)`。
- **L127 EN**: Assigns or updates `max_relation`.
  **L127 CN**: 对 `max_relation` 进行赋值或更新。
- **L128 EN**: Executes Python statement `logger.debug(`.
  **L128 CN**: 执行 Python 语句 `logger.debug(`。
- **L129 EN**: Executes Python statement `f"Generated {len(triplets)} triplets for {input_file} with {opt_level}"`.
  **L129 CN**: 执行 Python 语句 `f"Generated {len(triplets)} triplets for {input_file} with {opt_level}"`。
- **L130 EN**: Executes Python statement `)`.
  **L130 CN**: 执行 Python 语句 `)`。
- **L131 EN**: Blank line separates nearby declarations or logic blocks.
  **L131 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 132-144

````python
        return TripletResult(all_triplets, max_relation)

    def _run_pipeline(self, input_file: Path, opt_level: str) -> Tuple[Set[str], int]:
        """Run opt | llvm-ir2vec pipeline using subprocess pipes."""
        try:
            # Run opt first
            opt_proc = subprocess.Popen(
                [self.opt_binary, f"-{opt_level}", str(input_file), "-o", "-"],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True,
            )

````
- **L132 EN**: Returns a value or exits the current function.
  **L132 CN**: 返回一个值或结束当前函数。
- **L133 EN**: Blank line separates nearby declarations or logic blocks.
  **L133 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L134 EN**: Declares function `_run_pipeline`.
  **L134 CN**: 声明函数 `_run_pipeline`。
- **L135 EN**: Executes Python statement `"""Run opt | llvm-ir2vec pipeline using subprocess pipes."""`.
  **L135 CN**: 执行 Python 语句 `"""Run opt | llvm-ir2vec pipeline using subprocess pipes."""`。
- **L136 EN**: Controls Python flow with `try` logic.
  **L136 CN**: 使用 `try` 逻辑控制 Python 执行流程。
- **L137 EN**: Comment documents nearby script behavior: `Run opt first`.
  **L137 CN**: 注释说明了附近脚本逻辑：`Run opt first`。
- **L138 EN**: Assigns or updates `opt_proc`.
  **L138 CN**: 对 `opt_proc` 进行赋值或更新。
- **L139 EN**: Executes Python statement `[self.opt_binary, f"-{opt_level}", str(input_file), "-o", "-"],`.
  **L139 CN**: 执行 Python 语句 `[self.opt_binary, f"-{opt_level}", str(input_file), "-o", "-"],`。
- **L140 EN**: Assigns or updates `stdout`.
  **L140 CN**: 对 `stdout` 进行赋值或更新。
- **L141 EN**: Assigns or updates `stderr`.
  **L141 CN**: 对 `stderr` 进行赋值或更新。
- **L142 EN**: Assigns or updates `text`.
  **L142 CN**: 对 `text` 进行赋值或更新。
- **L143 EN**: Executes Python statement `)`.
  **L143 CN**: 执行 Python 语句 `)`。
- **L144 EN**: Blank line separates nearby declarations or logic blocks.
  **L144 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 145-157

````python
            # Run llvm-ir2vec with opt's output as input
            ir2vec_proc = subprocess.Popen(
                [self.ir2vec_binary, "triplets", "--mode=llvm", "-", "-o", "-"],
                stdin=opt_proc.stdout,
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True,
            )

            opt_proc.stdout.close()
            stdout, _ = ir2vec_proc.communicate()
            opt_proc.wait()

````
- **L145 EN**: Comment documents nearby script behavior: `Run llvm-ir2vec with opt's output as input`.
  **L145 CN**: 注释说明了附近脚本逻辑：`Run llvm-ir2vec with opt's output as input`。
- **L146 EN**: Assigns or updates `ir2vec_proc`.
  **L146 CN**: 对 `ir2vec_proc` 进行赋值或更新。
- **L147 EN**: Assigns or updates `[self.ir2vec_binary, "triplets", "--mode`.
  **L147 CN**: 对 `[self.ir2vec_binary, "triplets", "--mode` 进行赋值或更新。
- **L148 EN**: Assigns or updates `stdin`.
  **L148 CN**: 对 `stdin` 进行赋值或更新。
- **L149 EN**: Assigns or updates `stdout`.
  **L149 CN**: 对 `stdout` 进行赋值或更新。
- **L150 EN**: Assigns or updates `stderr`.
  **L150 CN**: 对 `stderr` 进行赋值或更新。
- **L151 EN**: Assigns or updates `text`.
  **L151 CN**: 对 `text` 进行赋值或更新。
- **L152 EN**: Executes Python statement `)`.
  **L152 CN**: 执行 Python 语句 `)`。
- **L153 EN**: Blank line separates nearby declarations or logic blocks.
  **L153 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L154 EN**: Executes Python statement `opt_proc.stdout.close()`.
  **L154 CN**: 执行 Python 语句 `opt_proc.stdout.close()`。
- **L155 EN**: Assigns or updates `stdout, _`.
  **L155 CN**: 对 `stdout, _` 进行赋值或更新。
- **L156 EN**: Executes Python statement `opt_proc.wait()`.
  **L156 CN**: 执行 Python 语句 `opt_proc.wait()`。
- **L157 EN**: Blank line separates nearby declarations or logic blocks.
  **L157 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 158-181

````python
            # Check if either process failed
            if opt_proc.returncode != 0 or ir2vec_proc.returncode != 0:
                return set(), 1

            return self._parse_triplet_output(stdout)
        except (subprocess.SubprocessError, OSError):
            return set(), 1

    def _run_mir_pipeline(self, input_file: Path) -> Tuple[Set[str], int]:
        """Run llvm-ir2vec pipeline for MIR files."""
        try:
            # Run llvm-ir2vec directly on MIR file
            result = subprocess.run(
                [
                    self.ir2vec_binary,
                    "triplets",
                    "--mode=mir",
                    str(input_file),
                    "-o",
                    "-",
                ],
                stdout=subprocess.PIPE,
                stderr=subprocess.PIPE,
                text=True,
````
- **L158 EN**: Comment documents nearby script behavior: `Check if either process failed`.
  **L158 CN**: 注释说明了附近脚本逻辑：`Check if either process failed`。
- **L159 EN**: Controls Python flow with `if` logic.
  **L159 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L160 EN**: Returns a value or exits the current function.
  **L160 CN**: 返回一个值或结束当前函数。
- **L161 EN**: Blank line separates nearby declarations or logic blocks.
  **L161 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L162 EN**: Returns a value or exits the current function.
  **L162 CN**: 返回一个值或结束当前函数。
- **L163 EN**: Controls Python flow with `except` logic.
  **L163 CN**: 使用 `except` 逻辑控制 Python 执行流程。
- **L164 EN**: Returns a value or exits the current function.
  **L164 CN**: 返回一个值或结束当前函数。
- **L165 EN**: Blank line separates nearby declarations or logic blocks.
  **L165 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L166 EN**: Declares function `_run_mir_pipeline`.
  **L166 CN**: 声明函数 `_run_mir_pipeline`。
- **L167 EN**: Executes Python statement `"""Run llvm-ir2vec pipeline for MIR files."""`.
  **L167 CN**: 执行 Python 语句 `"""Run llvm-ir2vec pipeline for MIR files."""`。
- **L168 EN**: Controls Python flow with `try` logic.
  **L168 CN**: 使用 `try` 逻辑控制 Python 执行流程。
- **L169 EN**: Comment documents nearby script behavior: `Run llvm-ir2vec directly on MIR file`.
  **L169 CN**: 注释说明了附近脚本逻辑：`Run llvm-ir2vec directly on MIR file`。
- **L170 EN**: Assigns or updates `result`.
  **L170 CN**: 对 `result` 进行赋值或更新。
- **L171 EN**: Executes Python statement `[`.
  **L171 CN**: 执行 Python 语句 `[`。
- **L172 EN**: Executes Python statement `self.ir2vec_binary,`.
  **L172 CN**: 执行 Python 语句 `self.ir2vec_binary,`。
- **L173 EN**: Executes Python statement `"triplets",`.
  **L173 CN**: 执行 Python 语句 `"triplets",`。
- **L174 EN**: Assigns or updates `"--mode`.
  **L174 CN**: 对 `"--mode` 进行赋值或更新。
- **L175 EN**: Executes Python statement `str(input_file),`.
  **L175 CN**: 执行 Python 语句 `str(input_file),`。
- **L176 EN**: Executes Python statement `"-o",`.
  **L176 CN**: 执行 Python 语句 `"-o",`。
- **L177 EN**: Executes Python statement `"-",`.
  **L177 CN**: 执行 Python 语句 `"-",`。
- **L178 EN**: Executes Python statement `],`.
  **L178 CN**: 执行 Python 语句 `],`。
- **L179 EN**: Assigns or updates `stdout`.
  **L179 CN**: 对 `stdout` 进行赋值或更新。
- **L180 EN**: Assigns or updates `stderr`.
  **L180 CN**: 对 `stderr` 进行赋值或更新。
- **L181 EN**: Assigns or updates `text`.
  **L181 CN**: 对 `text` 进行赋值或更新。

### Lines 182-196

````python
                check=False,
            )

            if result.returncode != 0:
                return set(), 1

            return self._parse_triplet_output(result.stdout)
        except (subprocess.SubprocessError, OSError):
            return set(), 1

    def _parse_triplet_output(self, output: str) -> Tuple[Set[str], int]:
        """Parse triplet output and extract max relation"""
        if not output.strip():
            return set(), 1

````
- **L182 EN**: Assigns or updates `check`.
  **L182 CN**: 对 `check` 进行赋值或更新。
- **L183 EN**: Executes Python statement `)`.
  **L183 CN**: 执行 Python 语句 `)`。
- **L184 EN**: Blank line separates nearby declarations or logic blocks.
  **L184 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L185 EN**: Controls Python flow with `if` logic.
  **L185 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L186 EN**: Returns a value or exits the current function.
  **L186 CN**: 返回一个值或结束当前函数。
- **L187 EN**: Blank line separates nearby declarations or logic blocks.
  **L187 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L188 EN**: Returns a value or exits the current function.
  **L188 CN**: 返回一个值或结束当前函数。
- **L189 EN**: Controls Python flow with `except` logic.
  **L189 CN**: 使用 `except` 逻辑控制 Python 执行流程。
- **L190 EN**: Returns a value or exits the current function.
  **L190 CN**: 返回一个值或结束当前函数。
- **L191 EN**: Blank line separates nearby declarations or logic blocks.
  **L191 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L192 EN**: Declares function `_parse_triplet_output`.
  **L192 CN**: 声明函数 `_parse_triplet_output`。
- **L193 EN**: Executes Python statement `"""Parse triplet output and extract max relation"""`.
  **L193 CN**: 执行 Python 语句 `"""Parse triplet output and extract max relation"""`。
- **L194 EN**: Controls Python flow with `if` logic.
  **L194 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L195 EN**: Returns a value or exits the current function.
  **L195 CN**: 返回一个值或结束当前函数。
- **L196 EN**: Blank line separates nearby declarations or logic blocks.
  **L196 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 197-208

````python
        lines = output.strip().split("\n")
        max_relation = 1

        # Extract max relation from metadata line
        if lines and lines[0].startswith("MAX_RELATION="):
            max_relation = int(lines[0].split("=")[1])
            lines = lines[1:]

        # Remove duplicate triplets by converting to a set
        return set(lines), max_relation

    def generate_triplets(self, file_list: Path) -> None:
````
- **L197 EN**: Assigns or updates `lines`.
  **L197 CN**: 对 `lines` 进行赋值或更新。
- **L198 EN**: Assigns or updates `max_relation`.
  **L198 CN**: 对 `max_relation` 进行赋值或更新。
- **L199 EN**: Blank line separates nearby declarations or logic blocks.
  **L199 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L200 EN**: Comment documents nearby script behavior: `Extract max relation from metadata line`.
  **L200 CN**: 注释说明了附近脚本逻辑：`Extract max relation from metadata line`。
- **L201 EN**: Controls Python flow with `if` logic.
  **L201 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L202 EN**: Assigns or updates `max_relation`.
  **L202 CN**: 对 `max_relation` 进行赋值或更新。
- **L203 EN**: Assigns or updates `lines`.
  **L203 CN**: 对 `lines` 进行赋值或更新。
- **L204 EN**: Blank line separates nearby declarations or logic blocks.
  **L204 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L205 EN**: Comment documents nearby script behavior: `Remove duplicate triplets by converting to a set`.
  **L205 CN**: 注释说明了附近脚本逻辑：`Remove duplicate triplets by converting to a set`。
- **L206 EN**: Returns a value or exits the current function.
  **L206 CN**: 返回一个值或结束当前函数。
- **L207 EN**: Blank line separates nearby declarations or logic blocks.
  **L207 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L208 EN**: Declares function `generate_triplets`.
  **L208 CN**: 声明函数 `generate_triplets`。

### Lines 209-224

````python
        """Main method to generate triplets from a list of LLVM IR or MIR files"""
        # Store file_list_path for later use in entity generation
        self.file_list_path = file_list

        input_files = self._read_file_list(file_list)

        if self.mode == "mir":
            logger.info(
                f"Processing {len(input_files)} MIR files using {self.max_workers} workers"
            )
        else:
            logger.info(
                f"Processing {len(input_files)} files with {self.num_optimizations} "
                f"optimization levels using {self.max_workers} workers"
            )

````
- **L209 EN**: Executes Python statement `"""Main method to generate triplets from a list of LLVM IR or MIR files"""`.
  **L209 CN**: 执行 Python 语句 `"""Main method to generate triplets from a list of LLVM IR or MIR files"""`。
- **L210 EN**: Comment documents nearby script behavior: `Store file_list_path for later use in entity generation`.
  **L210 CN**: 注释说明了附近脚本逻辑：`Store file_list_path for later use in entity generation`。
- **L211 EN**: Assigns or updates `self.file_list_path`.
  **L211 CN**: 对 `self.file_list_path` 进行赋值或更新。
- **L212 EN**: Blank line separates nearby declarations or logic blocks.
  **L212 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L213 EN**: Assigns or updates `input_files`.
  **L213 CN**: 对 `input_files` 进行赋值或更新。
- **L214 EN**: Blank line separates nearby declarations or logic blocks.
  **L214 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L215 EN**: Controls Python flow with `if` logic.
  **L215 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L216 EN**: Executes Python statement `logger.info(`.
  **L216 CN**: 执行 Python 语句 `logger.info(`。
- **L217 EN**: Executes Python statement `f"Processing {len(input_files)} MIR files using {self.max_workers} workers"`.
  **L217 CN**: 执行 Python 语句 `f"Processing {len(input_files)} MIR files using {self.max_workers} workers"`。
- **L218 EN**: Executes Python statement `)`.
  **L218 CN**: 执行 Python 语句 `)`。
- **L219 EN**: Controls Python flow with `else` logic.
  **L219 CN**: 使用 `else` 逻辑控制 Python 执行流程。
- **L220 EN**: Executes Python statement `logger.info(`.
  **L220 CN**: 执行 Python 语句 `logger.info(`。
- **L221 EN**: Executes Python statement `f"Processing {len(input_files)} files with {self.num_optimizations} "`.
  **L221 CN**: 执行 Python 语句 `f"Processing {len(input_files)} files with {self.num_optimizations} "`。
- **L222 EN**: Executes Python statement `f"optimization levels using {self.max_workers} workers"`.
  **L222 CN**: 执行 Python 语句 `f"optimization levels using {self.max_workers} workers"`。
- **L223 EN**: Executes Python statement `)`.
  **L223 CN**: 执行 Python 语句 `)`。
- **L224 EN**: Blank line separates nearby declarations or logic blocks.
  **L224 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 225-242

````python
        all_triplets = set()
        global_max_relation = 1

        with ThreadPoolExecutor(max_workers=self.max_workers) as executor:
            future_to_file = {
                executor.submit(self._process_single_file, file): file
                for file in input_files
            }

            for future in as_completed(future_to_file):
                try:
                    result = future.result()
                    all_triplets.update(result.triplets)
                    global_max_relation = max(global_max_relation, result.max_relation)
                except (subprocess.SubprocessError, OSError, ValueError) as e:
                    file_path = future_to_file[future]
                    logger.error(f"Error processing {file_path}: {e}")

````
- **L225 EN**: Assigns or updates `all_triplets`.
  **L225 CN**: 对 `all_triplets` 进行赋值或更新。
- **L226 EN**: Assigns or updates `global_max_relation`.
  **L226 CN**: 对 `global_max_relation` 进行赋值或更新。
- **L227 EN**: Blank line separates nearby declarations or logic blocks.
  **L227 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L228 EN**: Controls Python flow with `with` logic.
  **L228 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L229 EN**: Assigns or updates `future_to_file`.
  **L229 CN**: 对 `future_to_file` 进行赋值或更新。
- **L230 EN**: Executes Python statement `executor.submit(self._process_single_file, file): file`.
  **L230 CN**: 执行 Python 语句 `executor.submit(self._process_single_file, file): file`。
- **L231 EN**: Controls Python flow with `for` logic.
  **L231 CN**: 使用 `for` 逻辑控制 Python 执行流程。
- **L232 EN**: Executes Python statement `}`.
  **L232 CN**: 执行 Python 语句 `}`。
- **L233 EN**: Blank line separates nearby declarations or logic blocks.
  **L233 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L234 EN**: Controls Python flow with `for` logic.
  **L234 CN**: 使用 `for` 逻辑控制 Python 执行流程。
- **L235 EN**: Controls Python flow with `try` logic.
  **L235 CN**: 使用 `try` 逻辑控制 Python 执行流程。
- **L236 EN**: Assigns or updates `result`.
  **L236 CN**: 对 `result` 进行赋值或更新。
- **L237 EN**: Executes Python statement `all_triplets.update(result.triplets)`.
  **L237 CN**: 执行 Python 语句 `all_triplets.update(result.triplets)`。
- **L238 EN**: Assigns or updates `global_max_relation`.
  **L238 CN**: 对 `global_max_relation` 进行赋值或更新。
- **L239 EN**: Controls Python flow with `except` logic.
  **L239 CN**: 使用 `except` 逻辑控制 Python 执行流程。
- **L240 EN**: Assigns or updates `file_path`.
  **L240 CN**: 对 `file_path` 进行赋值或更新。
- **L241 EN**: Executes Python statement `logger.error(f"Error processing {file_path}: {e}")`.
  **L241 CN**: 执行 Python 语句 `logger.error(f"Error processing {file_path}: {e}")`。
- **L242 EN**: Blank line separates nearby declarations or logic blocks.
  **L242 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 243-257

````python
        self._generate_output_files(all_triplets, global_max_relation)
        logger.info("Processing completed successfully")

    def _read_file_list(self, file_list: Path) -> List[Path]:
        """Read and validate the list of input files"""
        input_files = []
        with open(file_list, "r") as f:
            for line_num, line in enumerate(f, 1):
                if line := line.strip():
                    file_path = Path(line)
                    if file_path.exists():
                        input_files.append(file_path)
                    else:
                        logger.warning(f"File not found (line {line_num}): {file_path}")

````
- **L243 EN**: Executes Python statement `self._generate_output_files(all_triplets, global_max_relation)`.
  **L243 CN**: 执行 Python 语句 `self._generate_output_files(all_triplets, global_max_relation)`。
- **L244 EN**: Executes Python statement `logger.info("Processing completed successfully")`.
  **L244 CN**: 执行 Python 语句 `logger.info("Processing completed successfully")`。
- **L245 EN**: Blank line separates nearby declarations or logic blocks.
  **L245 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L246 EN**: Declares function `_read_file_list`.
  **L246 CN**: 声明函数 `_read_file_list`。
- **L247 EN**: Executes Python statement `"""Read and validate the list of input files"""`.
  **L247 CN**: 执行 Python 语句 `"""Read and validate the list of input files"""`。
- **L248 EN**: Assigns or updates `input_files`.
  **L248 CN**: 对 `input_files` 进行赋值或更新。
- **L249 EN**: Controls Python flow with `with` logic.
  **L249 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L250 EN**: Controls Python flow with `for` logic.
  **L250 CN**: 使用 `for` 逻辑控制 Python 执行流程。
- **L251 EN**: Controls Python flow with `if` logic.
  **L251 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L252 EN**: Assigns or updates `file_path`.
  **L252 CN**: 对 `file_path` 进行赋值或更新。
- **L253 EN**: Controls Python flow with `if` logic.
  **L253 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L254 EN**: Executes Python statement `input_files.append(file_path)`.
  **L254 CN**: 执行 Python 语句 `input_files.append(file_path)`。
- **L255 EN**: Controls Python flow with `else` logic.
  **L255 CN**: 使用 `else` 逻辑控制 Python 执行流程。
- **L256 EN**: Executes Python statement `logger.warning(f"File not found (line {line_num}): {file_path}")`.
  **L256 CN**: 执行 Python 语句 `logger.warning(f"File not found (line {line_num}): {file_path}")`。
- **L257 EN**: Blank line separates nearby declarations or logic blocks.
  **L257 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 258-270

````python
        if not input_files:
            raise ValueError("No valid input files found")
        return input_files

    def _generate_output_files(self, all_triplets: Set[str], max_relation: int) -> None:
        """Generate the final output files"""
        logger.info(f"Generating output files with {len(all_triplets)} unique triplets")

        # Write all output files -- train2id.txt, entity2id.txt, relation2id.txt
        train2id_file = os.path.join(self.output_dir, "train2id.txt")
        entity2id_file = os.path.join(self.output_dir, "entity2id.txt")
        relation2id_file = os.path.join(self.output_dir, "relation2id.txt")

````
- **L258 EN**: Controls Python flow with `if` logic.
  **L258 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L259 EN**: Raises an exception to signal an error path.
  **L259 CN**: 抛出异常以显式表示错误路径。
- **L260 EN**: Returns a value or exits the current function.
  **L260 CN**: 返回一个值或结束当前函数。
- **L261 EN**: Blank line separates nearby declarations or logic blocks.
  **L261 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L262 EN**: Declares function `_generate_output_files`.
  **L262 CN**: 声明函数 `_generate_output_files`。
- **L263 EN**: Executes Python statement `"""Generate the final output files"""`.
  **L263 CN**: 执行 Python 语句 `"""Generate the final output files"""`。
- **L264 EN**: Executes Python statement `logger.info(f"Generating output files with {len(all_triplets)} unique triplets")`.
  **L264 CN**: 执行 Python 语句 `logger.info(f"Generating output files with {len(all_triplets)} unique triplets")`。
- **L265 EN**: Blank line separates nearby declarations or logic blocks.
  **L265 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L266 EN**: Comment documents nearby script behavior: `Write all output files -- train2id.txt, entity2id.txt, relation2id.txt`.
  **L266 CN**: 注释说明了附近脚本逻辑：`Write all output files -- train2id.txt, entity2id.txt, relation2id.txt`。
- **L267 EN**: Assigns or updates `train2id_file`.
  **L267 CN**: 对 `train2id_file` 进行赋值或更新。
- **L268 EN**: Assigns or updates `entity2id_file`.
  **L268 CN**: 对 `entity2id_file` 进行赋值或更新。
- **L269 EN**: Assigns or updates `relation2id_file`.
  **L269 CN**: 对 `relation2id_file` 进行赋值或更新。
- **L270 EN**: Blank line separates nearby declarations or logic blocks.
  **L270 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 271-286

````python
        with open(train2id_file, "w") as f:
            f.write(f"{len(all_triplets)}\n")
            f.writelines(f"{triplet}\n" for triplet in all_triplets)

        self._generate_entity2id(entity2id_file)
        self._generate_relation2id(relation2id_file, max_relation)

    def _generate_entity2id(self, output_file: Path) -> None:
        """Generate entity2id.txt using llvm-ir2vec"""
        if self.mode == "mir":
            # For MIR mode, we need to provide a sample MIR file to determine target
            # Use the first file from the processed list
            input_files = self._read_file_list(self.file_list_path)
            if not input_files:
                raise ValueError("No input files available for entity generation")

````
- **L271 EN**: Controls Python flow with `with` logic.
  **L271 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L272 EN**: Executes Python statement `f.write(f"{len(all_triplets)}\n")`.
  **L272 CN**: 执行 Python 语句 `f.write(f"{len(all_triplets)}\n")`。
- **L273 EN**: Executes Python statement `f.writelines(f"{triplet}\n" for triplet in all_triplets)`.
  **L273 CN**: 执行 Python 语句 `f.writelines(f"{triplet}\n" for triplet in all_triplets)`。
- **L274 EN**: Blank line separates nearby declarations or logic blocks.
  **L274 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L275 EN**: Executes Python statement `self._generate_entity2id(entity2id_file)`.
  **L275 CN**: 执行 Python 语句 `self._generate_entity2id(entity2id_file)`。
- **L276 EN**: Executes Python statement `self._generate_relation2id(relation2id_file, max_relation)`.
  **L276 CN**: 执行 Python 语句 `self._generate_relation2id(relation2id_file, max_relation)`。
- **L277 EN**: Blank line separates nearby declarations or logic blocks.
  **L277 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L278 EN**: Declares function `_generate_entity2id`.
  **L278 CN**: 声明函数 `_generate_entity2id`。
- **L279 EN**: Executes Python statement `"""Generate entity2id.txt using llvm-ir2vec"""`.
  **L279 CN**: 执行 Python 语句 `"""Generate entity2id.txt using llvm-ir2vec"""`。
- **L280 EN**: Controls Python flow with `if` logic.
  **L280 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L281 EN**: Comment documents nearby script behavior: `For MIR mode, we need to provide a sample MIR file to determine target`.
  **L281 CN**: 注释说明了附近脚本逻辑：`For MIR mode, we need to provide a sample MIR file to determine target`。
- **L282 EN**: Comment documents nearby script behavior: `Use the first file from the processed list`.
  **L282 CN**: 注释说明了附近脚本逻辑：`Use the first file from the processed list`。
- **L283 EN**: Assigns or updates `input_files`.
  **L283 CN**: 对 `input_files` 进行赋值或更新。
- **L284 EN**: Controls Python flow with `if` logic.
  **L284 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L285 EN**: Raises an exception to signal an error path.
  **L285 CN**: 抛出异常以显式表示错误路径。
- **L286 EN**: Blank line separates nearby declarations or logic blocks.
  **L286 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 287-310

````python
            subprocess.run(
                [
                    str(self.ir2vec_binary),
                    "entities",
                    "--mode=mir",
                    str(input_files[0]),
                    "-o",
                    str(output_file),
                ],
                check=True,
                capture_output=True,
            )
        else:
            subprocess.run(
                [
                    str(self.ir2vec_binary),
                    "entities",
                    "--mode=llvm",
                    "-o",
                    str(output_file),
                ],
                check=True,
                capture_output=True,
            )
````
- **L287 EN**: Executes Python statement `subprocess.run(`.
  **L287 CN**: 执行 Python 语句 `subprocess.run(`。
- **L288 EN**: Executes Python statement `[`.
  **L288 CN**: 执行 Python 语句 `[`。
- **L289 EN**: Executes Python statement `str(self.ir2vec_binary),`.
  **L289 CN**: 执行 Python 语句 `str(self.ir2vec_binary),`。
- **L290 EN**: Executes Python statement `"entities",`.
  **L290 CN**: 执行 Python 语句 `"entities",`。
- **L291 EN**: Assigns or updates `"--mode`.
  **L291 CN**: 对 `"--mode` 进行赋值或更新。
- **L292 EN**: Executes Python statement `str(input_files[0]),`.
  **L292 CN**: 执行 Python 语句 `str(input_files[0]),`。
- **L293 EN**: Executes Python statement `"-o",`.
  **L293 CN**: 执行 Python 语句 `"-o",`。
- **L294 EN**: Executes Python statement `str(output_file),`.
  **L294 CN**: 执行 Python 语句 `str(output_file),`。
- **L295 EN**: Executes Python statement `],`.
  **L295 CN**: 执行 Python 语句 `],`。
- **L296 EN**: Assigns or updates `check`.
  **L296 CN**: 对 `check` 进行赋值或更新。
- **L297 EN**: Assigns or updates `capture_output`.
  **L297 CN**: 对 `capture_output` 进行赋值或更新。
- **L298 EN**: Executes Python statement `)`.
  **L298 CN**: 执行 Python 语句 `)`。
- **L299 EN**: Controls Python flow with `else` logic.
  **L299 CN**: 使用 `else` 逻辑控制 Python 执行流程。
- **L300 EN**: Executes Python statement `subprocess.run(`.
  **L300 CN**: 执行 Python 语句 `subprocess.run(`。
- **L301 EN**: Executes Python statement `[`.
  **L301 CN**: 执行 Python 语句 `[`。
- **L302 EN**: Executes Python statement `str(self.ir2vec_binary),`.
  **L302 CN**: 执行 Python 语句 `str(self.ir2vec_binary),`。
- **L303 EN**: Executes Python statement `"entities",`.
  **L303 CN**: 执行 Python 语句 `"entities",`。
- **L304 EN**: Assigns or updates `"--mode`.
  **L304 CN**: 对 `"--mode` 进行赋值或更新。
- **L305 EN**: Executes Python statement `"-o",`.
  **L305 CN**: 执行 Python 语句 `"-o",`。
- **L306 EN**: Executes Python statement `str(output_file),`.
  **L306 CN**: 执行 Python 语句 `str(output_file),`。
- **L307 EN**: Executes Python statement `],`.
  **L307 CN**: 执行 Python 语句 `],`。
- **L308 EN**: Assigns or updates `check`.
  **L308 CN**: 对 `check` 进行赋值或更新。
- **L309 EN**: Assigns or updates `capture_output`.
  **L309 CN**: 对 `capture_output` 进行赋值或更新。
- **L310 EN**: Executes Python statement `)`.
  **L310 CN**: 执行 Python 语句 `)`。

### Lines 311-328

````python

    def _generate_relation2id(self, output_file: Path, max_relation: int) -> None:
        """Generate relation2id.txt from max relation"""
        max_relation = max(max_relation, 1)  # At least Next relation
        num_relations = max_relation + 1

        with open(output_file, "w") as f:
            f.write(f"{num_relations}\n")
            if self.mode == "llvm":
                # LLVM IR has Type relation at 0
                f.write("Type\t0\n")
                f.write("Next\t1\n")
                f.writelines(f"Arg{i-2}\t{i}\n" for i in range(2, num_relations))
            else:
                # MIR doesn't have Type relation, starts with Next at 0
                f.write("Next\t0\n")
                f.writelines(f"Arg{i-1}\t{i}\n" for i in range(1, num_relations))

````
- **L311 EN**: Blank line separates nearby declarations or logic blocks.
  **L311 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L312 EN**: Declares function `_generate_relation2id`.
  **L312 CN**: 声明函数 `_generate_relation2id`。
- **L313 EN**: Executes Python statement `"""Generate relation2id.txt from max relation"""`.
  **L313 CN**: 执行 Python 语句 `"""Generate relation2id.txt from max relation"""`。
- **L314 EN**: Assigns or updates `max_relation`.
  **L314 CN**: 对 `max_relation` 进行赋值或更新。
- **L315 EN**: Assigns or updates `num_relations`.
  **L315 CN**: 对 `num_relations` 进行赋值或更新。
- **L316 EN**: Blank line separates nearby declarations or logic blocks.
  **L316 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L317 EN**: Controls Python flow with `with` logic.
  **L317 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L318 EN**: Executes Python statement `f.write(f"{num_relations}\n")`.
  **L318 CN**: 执行 Python 语句 `f.write(f"{num_relations}\n")`。
- **L319 EN**: Controls Python flow with `if` logic.
  **L319 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L320 EN**: Comment documents nearby script behavior: `LLVM IR has Type relation at 0`.
  **L320 CN**: 注释说明了附近脚本逻辑：`LLVM IR has Type relation at 0`。
- **L321 EN**: Executes Python statement `f.write("Type\t0\n")`.
  **L321 CN**: 执行 Python 语句 `f.write("Type\t0\n")`。
- **L322 EN**: Executes Python statement `f.write("Next\t1\n")`.
  **L322 CN**: 执行 Python 语句 `f.write("Next\t1\n")`。
- **L323 EN**: Executes Python statement `f.writelines(f"Arg{i-2}\t{i}\n" for i in range(2, num_relations))`.
  **L323 CN**: 执行 Python 语句 `f.writelines(f"Arg{i-2}\t{i}\n" for i in range(2, num_relations))`。
- **L324 EN**: Controls Python flow with `else` logic.
  **L324 CN**: 使用 `else` 逻辑控制 Python 执行流程。
- **L325 EN**: Comment documents nearby script behavior: `MIR doesn't have Type relation, starts with Next at 0`.
  **L325 CN**: 注释说明了附近脚本逻辑：`MIR doesn't have Type relation, starts with Next at 0`。
- **L326 EN**: Executes Python statement `f.write("Next\t0\n")`.
  **L326 CN**: 执行 Python 语句 `f.write("Next\t0\n")`。
- **L327 EN**: Executes Python statement `f.writelines(f"Arg{i-1}\t{i}\n" for i in range(1, num_relations))`.
  **L327 CN**: 执行 Python 语句 `f.writelines(f"Arg{i-1}\t{i}\n" for i in range(1, num_relations))`。
- **L328 EN**: Blank line separates nearby declarations or logic blocks.
  **L328 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 329-352

````python

def main():
    """Main entry point"""
    parser = argparse.ArgumentParser(
        description="Generate IR2Vec or MIR2Vec triplets from LLVM IR or Machine IR files",
        formatter_class=argparse.RawDescriptionHelpFormatter,
    )

    parser.add_argument(
        "llvm_build_dir", type=Path, help="Path to LLVM build directory"
    )
    parser.add_argument(
        "num_optimizations",
        type=int,
        nargs="?",
        default=1,
        help="Number of optimization levels to apply (1-6) for LLVM IR mode",
    )
    parser.add_argument(
        "input_file_list",
        type=Path,
        help="File containing list of LLVM IR or MIR files to process",
    )
    parser.add_argument(
````
- **L329 EN**: Blank line separates nearby declarations or logic blocks.
  **L329 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L330 EN**: Declares function `main`.
  **L330 CN**: 声明函数 `main`。
- **L331 EN**: Executes Python statement `"""Main entry point"""`.
  **L331 CN**: 执行 Python 语句 `"""Main entry point"""`。
- **L332 EN**: Assigns or updates `parser`.
  **L332 CN**: 对 `parser` 进行赋值或更新。
- **L333 EN**: Assigns or updates `description`.
  **L333 CN**: 对 `description` 进行赋值或更新。
- **L334 EN**: Assigns or updates `formatter_class`.
  **L334 CN**: 对 `formatter_class` 进行赋值或更新。
- **L335 EN**: Executes Python statement `)`.
  **L335 CN**: 执行 Python 语句 `)`。
- **L336 EN**: Blank line separates nearby declarations or logic blocks.
  **L336 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L337 EN**: Executes Python statement `parser.add_argument(`.
  **L337 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L338 EN**: Assigns or updates `"llvm_build_dir", type`.
  **L338 CN**: 对 `"llvm_build_dir", type` 进行赋值或更新。
- **L339 EN**: Executes Python statement `)`.
  **L339 CN**: 执行 Python 语句 `)`。
- **L340 EN**: Executes Python statement `parser.add_argument(`.
  **L340 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L341 EN**: Executes Python statement `"num_optimizations",`.
  **L341 CN**: 执行 Python 语句 `"num_optimizations",`。
- **L342 EN**: Assigns or updates `type`.
  **L342 CN**: 对 `type` 进行赋值或更新。
- **L343 EN**: Assigns or updates `nargs`.
  **L343 CN**: 对 `nargs` 进行赋值或更新。
- **L344 EN**: Assigns or updates `default`.
  **L344 CN**: 对 `default` 进行赋值或更新。
- **L345 EN**: Assigns or updates `help`.
  **L345 CN**: 对 `help` 进行赋值或更新。
- **L346 EN**: Executes Python statement `)`.
  **L346 CN**: 执行 Python 语句 `)`。
- **L347 EN**: Executes Python statement `parser.add_argument(`.
  **L347 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L348 EN**: Executes Python statement `"input_file_list",`.
  **L348 CN**: 执行 Python 语句 `"input_file_list",`。
- **L349 EN**: Assigns or updates `type`.
  **L349 CN**: 对 `type` 进行赋值或更新。
- **L350 EN**: Assigns or updates `help`.
  **L350 CN**: 对 `help` 进行赋值或更新。
- **L351 EN**: Executes Python statement `)`.
  **L351 CN**: 执行 Python 语句 `)`。
- **L352 EN**: Executes Python statement `parser.add_argument(`.
  **L352 CN**: 执行 Python 语句 `parser.add_argument(`。

### Lines 353-375

````python
        "output_dir", type=Path, help="Output directory for generated files"
    )
    parser.add_argument(
        "--mode",
        type=str,
        choices=["llvm", "mir"],
        default="llvm",
        help="Operation mode: 'llvm' for LLVM IR (default) or 'mir' for Machine IR",
    )
    parser.add_argument(
        "-j",
        "--max-workers",
        type=int,
        default=DEFAULT_MAX_WORKERS,
        help=f"Maximum number of parallel workers (default: {DEFAULT_MAX_WORKERS})",
    )
    parser.add_argument(
        "-v", "--verbose", action="store_true", help="Enable debug logging"
    )
    parser.add_argument(
        "-q", "--quiet", action="store_true", help="Suppress all output except errors"
    )

````
- **L353 EN**: Assigns or updates `"output_dir", type`.
  **L353 CN**: 对 `"output_dir", type` 进行赋值或更新。
- **L354 EN**: Executes Python statement `)`.
  **L354 CN**: 执行 Python 语句 `)`。
- **L355 EN**: Executes Python statement `parser.add_argument(`.
  **L355 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L356 EN**: Executes Python statement `"--mode",`.
  **L356 CN**: 执行 Python 语句 `"--mode",`。
- **L357 EN**: Assigns or updates `type`.
  **L357 CN**: 对 `type` 进行赋值或更新。
- **L358 EN**: Assigns or updates `choices`.
  **L358 CN**: 对 `choices` 进行赋值或更新。
- **L359 EN**: Assigns or updates `default`.
  **L359 CN**: 对 `default` 进行赋值或更新。
- **L360 EN**: Assigns or updates `help`.
  **L360 CN**: 对 `help` 进行赋值或更新。
- **L361 EN**: Executes Python statement `)`.
  **L361 CN**: 执行 Python 语句 `)`。
- **L362 EN**: Executes Python statement `parser.add_argument(`.
  **L362 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L363 EN**: Executes Python statement `"-j",`.
  **L363 CN**: 执行 Python 语句 `"-j",`。
- **L364 EN**: Executes Python statement `"--max-workers",`.
  **L364 CN**: 执行 Python 语句 `"--max-workers",`。
- **L365 EN**: Assigns or updates `type`.
  **L365 CN**: 对 `type` 进行赋值或更新。
- **L366 EN**: Assigns or updates `default`.
  **L366 CN**: 对 `default` 进行赋值或更新。
- **L367 EN**: Assigns or updates `help`.
  **L367 CN**: 对 `help` 进行赋值或更新。
- **L368 EN**: Executes Python statement `)`.
  **L368 CN**: 执行 Python 语句 `)`。
- **L369 EN**: Executes Python statement `parser.add_argument(`.
  **L369 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L370 EN**: Assigns or updates `"-v", "--verbose", action`.
  **L370 CN**: 对 `"-v", "--verbose", action` 进行赋值或更新。
- **L371 EN**: Executes Python statement `)`.
  **L371 CN**: 执行 Python 语句 `)`。
- **L372 EN**: Executes Python statement `parser.add_argument(`.
  **L372 CN**: 执行 Python 语句 `parser.add_argument(`。
- **L373 EN**: Assigns or updates `"-q", "--quiet", action`.
  **L373 CN**: 对 `"-q", "--quiet", action` 进行赋值或更新。
- **L374 EN**: Executes Python statement `)`.
  **L374 CN**: 执行 Python 语句 `)`。
- **L375 EN**: Blank line separates nearby declarations or logic blocks.
  **L375 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 376-389

````python
    args = parser.parse_args()

    # Configure logging
    level = (
        logging.ERROR
        if args.quiet
        else (logging.DEBUG if args.verbose else logging.INFO)
    )
    logging.basicConfig(
        level=level,
        format="[%(asctime)s] %(levelname)s: %(message)s",
        datefmt="%H:%M:%S",
    )

````
- **L376 EN**: Assigns or updates `args`.
  **L376 CN**: 对 `args` 进行赋值或更新。
- **L377 EN**: Blank line separates nearby declarations or logic blocks.
  **L377 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L378 EN**: Comment documents nearby script behavior: `Configure logging`.
  **L378 CN**: 注释说明了附近脚本逻辑：`Configure logging`。
- **L379 EN**: Assigns or updates `level`.
  **L379 CN**: 对 `level` 进行赋值或更新。
- **L380 EN**: Executes Python statement `logging.ERROR`.
  **L380 CN**: 执行 Python 语句 `logging.ERROR`。
- **L381 EN**: Controls Python flow with `if` logic.
  **L381 CN**: 使用 `if` 逻辑控制 Python 执行流程。
- **L382 EN**: Controls Python flow with `else` logic.
  **L382 CN**: 使用 `else` 逻辑控制 Python 执行流程。
- **L383 EN**: Executes Python statement `)`.
  **L383 CN**: 执行 Python 语句 `)`。
- **L384 EN**: Executes Python statement `logging.basicConfig(`.
  **L384 CN**: 执行 Python 语句 `logging.basicConfig(`。
- **L385 EN**: Assigns or updates `level`.
  **L385 CN**: 对 `level` 进行赋值或更新。
- **L386 EN**: Assigns or updates `format`.
  **L386 CN**: 对 `format` 进行赋值或更新。
- **L387 EN**: Assigns or updates `datefmt`.
  **L387 CN**: 对 `datefmt` 进行赋值或更新。
- **L388 EN**: Executes Python statement `)`.
  **L388 CN**: 执行 Python 语句 `)`。
- **L389 EN**: Blank line separates nearby declarations or logic blocks.
  **L389 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 390-401

````python
    generator = IR2VecTripletGenerator(
        args.llvm_build_dir,
        args.num_optimizations,
        args.output_dir,
        args.max_workers,
        args.mode,
    )
    generator.generate_triplets(args.input_file_list)


if __name__ == "__main__":
    main()
````
- **L390 EN**: Assigns or updates `generator`.
  **L390 CN**: 对 `generator` 进行赋值或更新。
- **L391 EN**: Executes Python statement `args.llvm_build_dir,`.
  **L391 CN**: 执行 Python 语句 `args.llvm_build_dir,`。
- **L392 EN**: Executes Python statement `args.num_optimizations,`.
  **L392 CN**: 执行 Python 语句 `args.num_optimizations,`。
- **L393 EN**: Executes Python statement `args.output_dir,`.
  **L393 CN**: 执行 Python 语句 `args.output_dir,`。
- **L394 EN**: Executes Python statement `args.max_workers,`.
  **L394 CN**: 执行 Python 语句 `args.max_workers,`。
- **L395 EN**: Executes Python statement `args.mode,`.
  **L395 CN**: 执行 Python 语句 `args.mode,`。
- **L396 EN**: Executes Python statement `)`.
  **L396 CN**: 执行 Python 语句 `)`。
- **L397 EN**: Executes Python statement `generator.generate_triplets(args.input_file_list)`.
  **L397 CN**: 执行 Python 语句 `generator.generate_triplets(args.input_file_list)`。
- **L398 EN**: Blank line separates nearby declarations or logic blocks.
  **L398 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L399 EN**: Blank line separates nearby declarations or logic blocks.
  **L399 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L400 EN**: Defines the script entry point used for direct execution.
  **L400 CN**: 定义脚本被直接执行时使用的入口点。
- **L401 EN**: Executes Python statement `main()`.
  **L401 CN**: 执行 Python 语句 `main()`。

## Key Concepts / 关键概念

- EN: lit test runner infrastructure
  - CN: lit 测试运行基础设施
- EN: parallel task coordination
  - CN: 并行任务协调
- EN: concurrency control
  - CN: 并发控制
- EN: subprocess management
  - CN: 子进程管理
- EN: command-line parsing
  - CN: 命令行解析
- EN: worker execution model
  - CN: 工作线程/进程执行模型
- EN: Python utility scripting
  - CN: Python 工具脚本

## Dependencies / 依赖关系

- EN: `argparse` supplies command-line argument parsing.
  - CN: `argparse` 提供了命令行参数解析。
- EN: `logging` supplies supporting Python helpers.
  - CN: `logging` 提供了辅助性的 Python 模块。
- EN: `os` supplies operating-system services.
  - CN: `os` 提供了操作系统服务。
- EN: `random` supplies supporting Python helpers.
  - CN: `random` 提供了辅助性的 Python 模块。
- EN: `subprocess` supplies subprocess spawning and control.
  - CN: `subprocess` 提供了子进程创建与控制。
- EN: `concurrent.futures` supplies supporting Python helpers.
  - CN: `concurrent.futures` 提供了辅助性的 Python 模块。
- EN: `pathlib` supplies path manipulation utilities.
  - CN: `pathlib` 提供了路径处理工具。
- EN: `typing` supplies type-annotation support.
  - CN: `typing` 提供了类型注解支持。
