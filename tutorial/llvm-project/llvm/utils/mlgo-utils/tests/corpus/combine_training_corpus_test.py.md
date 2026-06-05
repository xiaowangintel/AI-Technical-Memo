# combine_training_corpus_test.py — Code Analysis / 代码分析

## Source / 来源

| Item | English | 中文 |
| --- | --- | --- |
| File | `llvm/utils/mlgo-utils/tests/corpus/combine_training_corpus_test.py` | `llvm/utils/mlgo-utils/tests/corpus/combine_training_corpus_test.py` |
| Repository | `llvm-project` | `llvm-project` |
| Purpose | # Test the functionality of combine_training_corpus_lib. | 实现 MLGO 工具脚本，用于语料生成、提取或面向模型的流程。 |

## Line-by-Line Analysis / 逐行分析

### Lines 1-8

````python
# REQUIRES: system-linux

## Test the functionality of combine_training_corpus_lib

import json
import os
import sys

````
- **L1 EN**: Comment documents nearby script behavior: `REQUIRES: system-linux`.
  **L1 CN**: 注释说明了附近脚本逻辑：`REQUIRES: system-linux`。
- **L2 EN**: Blank line separates nearby declarations or logic blocks.
  **L2 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L3 EN**: Comment documents nearby script behavior: `# Test the functionality of combine_training_corpus_lib`.
  **L3 CN**: 注释说明了附近脚本逻辑：`# Test the functionality of combine_training_corpus_lib`。
- **L4 EN**: Blank line separates nearby declarations or logic blocks.
  **L4 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L5 EN**: Imports Python module(s) `json` for supporting functionality.
  **L5 CN**: 导入 Python 模块 `json` 以提供辅助功能。
- **L6 EN**: Imports Python module(s) `os` for supporting functionality.
  **L6 CN**: 导入 Python 模块 `os` 以提供辅助功能。
- **L7 EN**: Imports Python module(s) `sys` for supporting functionality.
  **L7 CN**: 导入 Python 模块 `sys` 以提供辅助功能。
- **L8 EN**: Blank line separates nearby declarations or logic blocks.
  **L8 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 9-17

````python
from mlgo.corpus import combine_training_corpus_lib

## Test that combining two training corpora works as expected

# RUN: rm -rf %t.dir && mkdir %t.dir
# RUN: mkdir %t.dir/subcorpus1
# RUN: mkdir %t.dir/subcorpus2
# RUN: %python %s test_combine_corpus %t.dir | FileCheck %s --check-prefix CHECK-COMBINE-CORPUS

````
- **L9 EN**: Imports `combine_training_corpus_lib` from module `mlgo.corpus`.
  **L9 CN**: 从模块 `mlgo.corpus` 导入 `combine_training_corpus_lib`。
- **L10 EN**: Blank line separates nearby declarations or logic blocks.
  **L10 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L11 EN**: Comment documents nearby script behavior: `# Test that combining two training corpora works as expected`.
  **L11 CN**: 注释说明了附近脚本逻辑：`# Test that combining two training corpora works as expected`。
- **L12 EN**: Blank line separates nearby declarations or logic blocks.
  **L12 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L13 EN**: Comment documents nearby script behavior: `RUN: rm -rf %t.dir && mkdir %t.dir`.
  **L13 CN**: 注释说明了附近脚本逻辑：`RUN: rm -rf %t.dir && mkdir %t.dir`。
- **L14 EN**: Comment documents nearby script behavior: `RUN: mkdir %t.dir/subcorpus1`.
  **L14 CN**: 注释说明了附近脚本逻辑：`RUN: mkdir %t.dir/subcorpus1`。
- **L15 EN**: Comment documents nearby script behavior: `RUN: mkdir %t.dir/subcorpus2`.
  **L15 CN**: 注释说明了附近脚本逻辑：`RUN: mkdir %t.dir/subcorpus2`。
- **L16 EN**: Comment documents nearby script behavior: `RUN: %python %s test_combine_corpus %t.dir | FileCheck %s --check-prefix CHECK-COMBINE-...`.
  **L16 CN**: 注释说明了附近脚本逻辑：`RUN: %python %s test_combine_corpus %t.dir | FileCheck %s --check-prefix CHECK-COMBINE-...`。
- **L17 EN**: Blank line separates nearby declarations or logic blocks.
  **L17 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 18-33

````python

def test_combine_corpus(corpus_dir):
    subcorpus1_dir = os.path.join(corpus_dir, "subcorpus1")
    subcorpus2_dir = os.path.join(corpus_dir, "subcorpus2")
    subcorpus1_description = {
        "has_thinlto": False,
        "modules": ["test1.o", "test2.o"],
    }
    subcorpus2_description = {
        "has_thinlto": False,
        "modules": ["test3.o", "test4.o"],
    }
    with open(
        os.path.join(subcorpus1_dir, "corpus_description.json"), "w"
    ) as corpus1_description_handle:
        json.dump(subcorpus1_description, corpus1_description_handle)
````
- **L18 EN**: Blank line separates nearby declarations or logic blocks.
  **L18 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L19 EN**: Declares function `test_combine_corpus`.
  **L19 CN**: 声明函数 `test_combine_corpus`。
- **L20 EN**: Assigns or updates `subcorpus1_dir`.
  **L20 CN**: 对 `subcorpus1_dir` 进行赋值或更新。
- **L21 EN**: Assigns or updates `subcorpus2_dir`.
  **L21 CN**: 对 `subcorpus2_dir` 进行赋值或更新。
- **L22 EN**: Assigns or updates `subcorpus1_description`.
  **L22 CN**: 对 `subcorpus1_description` 进行赋值或更新。
- **L23 EN**: Executes Python statement `"has_thinlto": False,`.
  **L23 CN**: 执行 Python 语句 `"has_thinlto": False,`。
- **L24 EN**: Executes Python statement `"modules": ["test1.o", "test2.o"],`.
  **L24 CN**: 执行 Python 语句 `"modules": ["test1.o", "test2.o"],`。
- **L25 EN**: Executes Python statement `}`.
  **L25 CN**: 执行 Python 语句 `}`。
- **L26 EN**: Assigns or updates `subcorpus2_description`.
  **L26 CN**: 对 `subcorpus2_description` 进行赋值或更新。
- **L27 EN**: Executes Python statement `"has_thinlto": False,`.
  **L27 CN**: 执行 Python 语句 `"has_thinlto": False,`。
- **L28 EN**: Executes Python statement `"modules": ["test3.o", "test4.o"],`.
  **L28 CN**: 执行 Python 语句 `"modules": ["test3.o", "test4.o"],`。
- **L29 EN**: Executes Python statement `}`.
  **L29 CN**: 执行 Python 语句 `}`。
- **L30 EN**: Controls Python flow with `with` logic.
  **L30 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L31 EN**: Executes Python statement `os.path.join(subcorpus1_dir, "corpus_description.json"), "w"`.
  **L31 CN**: 执行 Python 语句 `os.path.join(subcorpus1_dir, "corpus_description.json"), "w"`。
- **L32 EN**: Executes Python statement `) as corpus1_description_handle:`.
  **L32 CN**: 执行 Python 语句 `) as corpus1_description_handle:`。
- **L33 EN**: Executes Python statement `json.dump(subcorpus1_description, corpus1_description_handle)`.
  **L33 CN**: 执行 Python 语句 `json.dump(subcorpus1_description, corpus1_description_handle)`。

### Lines 34-49

````python
    with open(
        os.path.join(subcorpus2_dir, "corpus_description.json"), "w"
    ) as corpus2_description_handle:
        json.dump(subcorpus2_description, corpus2_description_handle)
    combine_training_corpus_lib.combine_corpus(corpus_dir)
    with open(
        os.path.join(corpus_dir, "corpus_description.json"), encoding="utf-8"
    ) as combined_corpus_description_file:
        combined_corpus_description = json.load(combined_corpus_description_file)
    print(combined_corpus_description["has_thinlto"])
    # CHECK-COMBINE-CORPUS: False
    for module in sorted(combined_corpus_description["modules"]):
        print(module)
    # CHECK-COMBINE-CORPUS: subcorpus1/test1.o
    # CHECK-COMBINE-CORPUS: subcorpus1/test2.o
    # CHECK-COMBINE-CORPUS: subcorpus2/test3.o
````
- **L34 EN**: Controls Python flow with `with` logic.
  **L34 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L35 EN**: Executes Python statement `os.path.join(subcorpus2_dir, "corpus_description.json"), "w"`.
  **L35 CN**: 执行 Python 语句 `os.path.join(subcorpus2_dir, "corpus_description.json"), "w"`。
- **L36 EN**: Executes Python statement `) as corpus2_description_handle:`.
  **L36 CN**: 执行 Python 语句 `) as corpus2_description_handle:`。
- **L37 EN**: Executes Python statement `json.dump(subcorpus2_description, corpus2_description_handle)`.
  **L37 CN**: 执行 Python 语句 `json.dump(subcorpus2_description, corpus2_description_handle)`。
- **L38 EN**: Executes Python statement `combine_training_corpus_lib.combine_corpus(corpus_dir)`.
  **L38 CN**: 执行 Python 语句 `combine_training_corpus_lib.combine_corpus(corpus_dir)`。
- **L39 EN**: Controls Python flow with `with` logic.
  **L39 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L40 EN**: Assigns or updates `os.path.join(corpus_dir, "corpus_description.json"), enco...`.
  **L40 CN**: 对 `os.path.join(corpus_dir, "corpus_description.json"), enco...` 进行赋值或更新。
- **L41 EN**: Executes Python statement `) as combined_corpus_description_file:`.
  **L41 CN**: 执行 Python 语句 `) as combined_corpus_description_file:`。
- **L42 EN**: Assigns or updates `combined_corpus_description`.
  **L42 CN**: 对 `combined_corpus_description` 进行赋值或更新。
- **L43 EN**: Executes Python statement `print(combined_corpus_description["has_thinlto"])`.
  **L43 CN**: 执行 Python 语句 `print(combined_corpus_description["has_thinlto"])`。
- **L44 EN**: Comment documents nearby script behavior: `CHECK-COMBINE-CORPUS: False`.
  **L44 CN**: 注释说明了附近脚本逻辑：`CHECK-COMBINE-CORPUS: False`。
- **L45 EN**: Controls Python flow with `for` logic.
  **L45 CN**: 使用 `for` 逻辑控制 Python 执行流程。
- **L46 EN**: Executes Python statement `print(module)`.
  **L46 CN**: 执行 Python 语句 `print(module)`。
- **L47 EN**: Comment documents nearby script behavior: `CHECK-COMBINE-CORPUS: subcorpus1/test1.o`.
  **L47 CN**: 注释说明了附近脚本逻辑：`CHECK-COMBINE-CORPUS: subcorpus1/test1.o`。
- **L48 EN**: Comment documents nearby script behavior: `CHECK-COMBINE-CORPUS: subcorpus1/test2.o`.
  **L48 CN**: 注释说明了附近脚本逻辑：`CHECK-COMBINE-CORPUS: subcorpus1/test2.o`。
- **L49 EN**: Comment documents nearby script behavior: `CHECK-COMBINE-CORPUS: subcorpus2/test3.o`.
  **L49 CN**: 注释说明了附近脚本逻辑：`CHECK-COMBINE-CORPUS: subcorpus2/test3.o`。

### Lines 50-59

````python
    # CHECK-COMBINE-CORPUS: subcorpus2/test4.o


## Test that we handle the empty folder case gracefully

# RUN: rm -rf %t.dir && mkdir %t.dir
# RUN: mkdir %t.dir/subcorpus1
# RUN: mkdir %t.dir/empty_dir
# RUN: %python %s test_empty_folder %t.dir | FileCheck %s --check-prefix CHECK-EMPTY-DIR

````
- **L50 EN**: Comment documents nearby script behavior: `CHECK-COMBINE-CORPUS: subcorpus2/test4.o`.
  **L50 CN**: 注释说明了附近脚本逻辑：`CHECK-COMBINE-CORPUS: subcorpus2/test4.o`。
- **L51 EN**: Blank line separates nearby declarations or logic blocks.
  **L51 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L52 EN**: Blank line separates nearby declarations or logic blocks.
  **L52 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L53 EN**: Comment documents nearby script behavior: `# Test that we handle the empty folder case gracefully`.
  **L53 CN**: 注释说明了附近脚本逻辑：`# Test that we handle the empty folder case gracefully`。
- **L54 EN**: Blank line separates nearby declarations or logic blocks.
  **L54 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L55 EN**: Comment documents nearby script behavior: `RUN: rm -rf %t.dir && mkdir %t.dir`.
  **L55 CN**: 注释说明了附近脚本逻辑：`RUN: rm -rf %t.dir && mkdir %t.dir`。
- **L56 EN**: Comment documents nearby script behavior: `RUN: mkdir %t.dir/subcorpus1`.
  **L56 CN**: 注释说明了附近脚本逻辑：`RUN: mkdir %t.dir/subcorpus1`。
- **L57 EN**: Comment documents nearby script behavior: `RUN: mkdir %t.dir/empty_dir`.
  **L57 CN**: 注释说明了附近脚本逻辑：`RUN: mkdir %t.dir/empty_dir`。
- **L58 EN**: Comment documents nearby script behavior: `RUN: %python %s test_empty_folder %t.dir | FileCheck %s --check-prefix CHECK-EMPTY-DIR`.
  **L58 CN**: 注释说明了附近脚本逻辑：`RUN: %python %s test_empty_folder %t.dir | FileCheck %s --check-prefix CHECK-EMPTY-DIR`。
- **L59 EN**: Blank line separates nearby declarations or logic blocks.
  **L59 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 60-75

````python

def test_empty_folder(corpus_dir):
    subcorpus1_dir = os.path.join(corpus_dir, "subcorpus1")
    subcorpus1_description = {"modules": ["test1.o", "test2.o"]}
    with open(
        os.path.join(subcorpus1_dir, "corpus_description.json"), "w"
    ) as subcorpus1_description_handle:
        json.dump(subcorpus1_description, subcorpus1_description_handle)
    combine_training_corpus_lib.combine_corpus(corpus_dir)
    with open(
        os.path.join(corpus_dir, "corpus_description.json"), encoding="utf-8"
    ) as combined_corpus_description_file:
        combined_corpus_description = json.load(combined_corpus_description_file)
    print(len(combined_corpus_description["modules"]))
    # CHECK-EMPTY-DIR: 2

````
- **L60 EN**: Blank line separates nearby declarations or logic blocks.
  **L60 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L61 EN**: Declares function `test_empty_folder`.
  **L61 CN**: 声明函数 `test_empty_folder`。
- **L62 EN**: Assigns or updates `subcorpus1_dir`.
  **L62 CN**: 对 `subcorpus1_dir` 进行赋值或更新。
- **L63 EN**: Assigns or updates `subcorpus1_description`.
  **L63 CN**: 对 `subcorpus1_description` 进行赋值或更新。
- **L64 EN**: Controls Python flow with `with` logic.
  **L64 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L65 EN**: Executes Python statement `os.path.join(subcorpus1_dir, "corpus_description.json"), "w"`.
  **L65 CN**: 执行 Python 语句 `os.path.join(subcorpus1_dir, "corpus_description.json"), "w"`。
- **L66 EN**: Executes Python statement `) as subcorpus1_description_handle:`.
  **L66 CN**: 执行 Python 语句 `) as subcorpus1_description_handle:`。
- **L67 EN**: Executes Python statement `json.dump(subcorpus1_description, subcorpus1_description_handle)`.
  **L67 CN**: 执行 Python 语句 `json.dump(subcorpus1_description, subcorpus1_description_handle)`。
- **L68 EN**: Executes Python statement `combine_training_corpus_lib.combine_corpus(corpus_dir)`.
  **L68 CN**: 执行 Python 语句 `combine_training_corpus_lib.combine_corpus(corpus_dir)`。
- **L69 EN**: Controls Python flow with `with` logic.
  **L69 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L70 EN**: Assigns or updates `os.path.join(corpus_dir, "corpus_description.json"), enco...`.
  **L70 CN**: 对 `os.path.join(corpus_dir, "corpus_description.json"), enco...` 进行赋值或更新。
- **L71 EN**: Executes Python statement `) as combined_corpus_description_file:`.
  **L71 CN**: 执行 Python 语句 `) as combined_corpus_description_file:`。
- **L72 EN**: Assigns or updates `combined_corpus_description`.
  **L72 CN**: 对 `combined_corpus_description` 进行赋值或更新。
- **L73 EN**: Executes Python statement `print(len(combined_corpus_description["modules"]))`.
  **L73 CN**: 执行 Python 语句 `print(len(combined_corpus_description["modules"]))`。
- **L74 EN**: Comment documents nearby script behavior: `CHECK-EMPTY-DIR: 2`.
  **L74 CN**: 注释说明了附近脚本逻辑：`CHECK-EMPTY-DIR: 2`。
- **L75 EN**: Blank line separates nearby declarations or logic blocks.
  **L75 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 76-84

````python

## Test that we ignore extra files that will not end up contributing to the
## corpus.

# RUN: rm -rf %t.dir && mkdir %t.dir
# RUN: mkdir %t.dir/subcorpus1
# RUN: touch %t.dir/empty.log
# RUN: %python %s test_ignore_extra_file %t.dir | FileCheck %s --check-prefix CHECK-IGNORE-EXTRA-FILE

````
- **L76 EN**: Blank line separates nearby declarations or logic blocks.
  **L76 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L77 EN**: Comment documents nearby script behavior: `# Test that we ignore extra files that will not end up contributing to the`.
  **L77 CN**: 注释说明了附近脚本逻辑：`# Test that we ignore extra files that will not end up contributing to the`。
- **L78 EN**: Comment documents nearby script behavior: `# corpus.`.
  **L78 CN**: 注释说明了附近脚本逻辑：`# corpus.`。
- **L79 EN**: Blank line separates nearby declarations or logic blocks.
  **L79 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L80 EN**: Comment documents nearby script behavior: `RUN: rm -rf %t.dir && mkdir %t.dir`.
  **L80 CN**: 注释说明了附近脚本逻辑：`RUN: rm -rf %t.dir && mkdir %t.dir`。
- **L81 EN**: Comment documents nearby script behavior: `RUN: mkdir %t.dir/subcorpus1`.
  **L81 CN**: 注释说明了附近脚本逻辑：`RUN: mkdir %t.dir/subcorpus1`。
- **L82 EN**: Comment documents nearby script behavior: `RUN: touch %t.dir/empty.log`.
  **L82 CN**: 注释说明了附近脚本逻辑：`RUN: touch %t.dir/empty.log`。
- **L83 EN**: Comment documents nearby script behavior: `RUN: %python %s test_ignore_extra_file %t.dir | FileCheck %s --check-prefix CHECK-IGNOR...`.
  **L83 CN**: 注释说明了附近脚本逻辑：`RUN: %python %s test_ignore_extra_file %t.dir | FileCheck %s --check-prefix CHECK-IGNOR...`。
- **L84 EN**: Blank line separates nearby declarations or logic blocks.
  **L84 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 85-100

````python

def test_ignore_extra_file(corpus_dir):
    subcorpus1_dir = os.path.join(corpus_dir, "subcorpus1")
    subcorpus1_description = {"modules": ["test1.o", "test2.o"]}
    with open(
        os.path.join(subcorpus1_dir, "corpus_description.json"), "w"
    ) as subcorpus1_description_handle:
        json.dump(subcorpus1_description, subcorpus1_description_handle)
    combine_training_corpus_lib.combine_corpus(corpus_dir)
    with open(
        os.path.join(corpus_dir, "corpus_description.json"), encoding="utf-8"
    ) as combined_corpus_description_file:
        combined_corpus_description = json.load(combined_corpus_description_file)
    print(len(combined_corpus_description["modules"]))
    # CHECK-IGNORE-EXTRA-FILE: 2

````
- **L85 EN**: Blank line separates nearby declarations or logic blocks.
  **L85 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L86 EN**: Declares function `test_ignore_extra_file`.
  **L86 CN**: 声明函数 `test_ignore_extra_file`。
- **L87 EN**: Assigns or updates `subcorpus1_dir`.
  **L87 CN**: 对 `subcorpus1_dir` 进行赋值或更新。
- **L88 EN**: Assigns or updates `subcorpus1_description`.
  **L88 CN**: 对 `subcorpus1_description` 进行赋值或更新。
- **L89 EN**: Controls Python flow with `with` logic.
  **L89 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L90 EN**: Executes Python statement `os.path.join(subcorpus1_dir, "corpus_description.json"), "w"`.
  **L90 CN**: 执行 Python 语句 `os.path.join(subcorpus1_dir, "corpus_description.json"), "w"`。
- **L91 EN**: Executes Python statement `) as subcorpus1_description_handle:`.
  **L91 CN**: 执行 Python 语句 `) as subcorpus1_description_handle:`。
- **L92 EN**: Executes Python statement `json.dump(subcorpus1_description, subcorpus1_description_handle)`.
  **L92 CN**: 执行 Python 语句 `json.dump(subcorpus1_description, subcorpus1_description_handle)`。
- **L93 EN**: Executes Python statement `combine_training_corpus_lib.combine_corpus(corpus_dir)`.
  **L93 CN**: 执行 Python 语句 `combine_training_corpus_lib.combine_corpus(corpus_dir)`。
- **L94 EN**: Controls Python flow with `with` logic.
  **L94 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L95 EN**: Assigns or updates `os.path.join(corpus_dir, "corpus_description.json"), enco...`.
  **L95 CN**: 对 `os.path.join(corpus_dir, "corpus_description.json"), enco...` 进行赋值或更新。
- **L96 EN**: Executes Python statement `) as combined_corpus_description_file:`.
  **L96 CN**: 执行 Python 语句 `) as combined_corpus_description_file:`。
- **L97 EN**: Assigns or updates `combined_corpus_description`.
  **L97 CN**: 对 `combined_corpus_description` 进行赋值或更新。
- **L98 EN**: Executes Python statement `print(len(combined_corpus_description["modules"]))`.
  **L98 CN**: 执行 Python 语句 `print(len(combined_corpus_description["modules"]))`。
- **L99 EN**: Comment documents nearby script behavior: `CHECK-IGNORE-EXTRA-FILE: 2`.
  **L99 CN**: 注释说明了附近脚本逻辑：`CHECK-IGNORE-EXTRA-FILE: 2`。
- **L100 EN**: Blank line separates nearby declarations or logic blocks.
  **L100 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 101-109

````python

## Test that we raise an error in the case where the corpora differ in a
## substantial way.

# RUN: rm -rf  %t.dir && mkdir %t.dir
# RUN: mkdir %t.dir/subcorpus1
# RUN: mkdir %t.dir/subcorpus2
# RUN: %python %s test_different_corpora %t.dir | FileCheck %s --check-prefix CHECK-DIFFERENT-CORPORA

````
- **L101 EN**: Blank line separates nearby declarations or logic blocks.
  **L101 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L102 EN**: Comment documents nearby script behavior: `# Test that we raise an error in the case where the corpora differ in a`.
  **L102 CN**: 注释说明了附近脚本逻辑：`# Test that we raise an error in the case where the corpora differ in a`。
- **L103 EN**: Comment documents nearby script behavior: `# substantial way.`.
  **L103 CN**: 注释说明了附近脚本逻辑：`# substantial way.`。
- **L104 EN**: Blank line separates nearby declarations or logic blocks.
  **L104 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L105 EN**: Comment documents nearby script behavior: `RUN: rm -rf %t.dir && mkdir %t.dir`.
  **L105 CN**: 注释说明了附近脚本逻辑：`RUN: rm -rf %t.dir && mkdir %t.dir`。
- **L106 EN**: Comment documents nearby script behavior: `RUN: mkdir %t.dir/subcorpus1`.
  **L106 CN**: 注释说明了附近脚本逻辑：`RUN: mkdir %t.dir/subcorpus1`。
- **L107 EN**: Comment documents nearby script behavior: `RUN: mkdir %t.dir/subcorpus2`.
  **L107 CN**: 注释说明了附近脚本逻辑：`RUN: mkdir %t.dir/subcorpus2`。
- **L108 EN**: Comment documents nearby script behavior: `RUN: %python %s test_different_corpora %t.dir | FileCheck %s --check-prefix CHECK-DIFFE...`.
  **L108 CN**: 注释说明了附近脚本逻辑：`RUN: %python %s test_different_corpora %t.dir | FileCheck %s --check-prefix CHECK-DIFFE...`。
- **L109 EN**: Blank line separates nearby declarations or logic blocks.
  **L109 CN**: 空行，用于分隔相邻的声明或逻辑块。

### Lines 110-125

````python

def test_different_corpora(corpus_dir):
    subcorpus1_dir = os.path.join(corpus_dir, "subcorpus1")
    subcorpus2_dir = os.path.join(corpus_dir, "subcorpus2")
    subcorpus1_description = {"has_thinlto": False, "modules": ["test1.o"]}
    subcorpus2_description = {"has_thinlto": True, "modules": ["test2.o"]}
    with open(
        os.path.join(subcorpus1_dir, "corpus_description.json"), "w"
    ) as subcorpus1_description_handle:
        json.dump(subcorpus1_description, subcorpus1_description_handle)
    with open(
        os.path.join(subcorpus2_dir, "corpus_description.json"), "w"
    ) as subcorpus2_description_handle:
        json.dump(subcorpus2_description, subcorpus2_description_handle)
    try:
        combine_training_corpus_lib.combine_corpus(corpus_dir)
````
- **L110 EN**: Blank line separates nearby declarations or logic blocks.
  **L110 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L111 EN**: Declares function `test_different_corpora`.
  **L111 CN**: 声明函数 `test_different_corpora`。
- **L112 EN**: Assigns or updates `subcorpus1_dir`.
  **L112 CN**: 对 `subcorpus1_dir` 进行赋值或更新。
- **L113 EN**: Assigns or updates `subcorpus2_dir`.
  **L113 CN**: 对 `subcorpus2_dir` 进行赋值或更新。
- **L114 EN**: Assigns or updates `subcorpus1_description`.
  **L114 CN**: 对 `subcorpus1_description` 进行赋值或更新。
- **L115 EN**: Assigns or updates `subcorpus2_description`.
  **L115 CN**: 对 `subcorpus2_description` 进行赋值或更新。
- **L116 EN**: Controls Python flow with `with` logic.
  **L116 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L117 EN**: Executes Python statement `os.path.join(subcorpus1_dir, "corpus_description.json"), "w"`.
  **L117 CN**: 执行 Python 语句 `os.path.join(subcorpus1_dir, "corpus_description.json"), "w"`。
- **L118 EN**: Executes Python statement `) as subcorpus1_description_handle:`.
  **L118 CN**: 执行 Python 语句 `) as subcorpus1_description_handle:`。
- **L119 EN**: Executes Python statement `json.dump(subcorpus1_description, subcorpus1_description_handle)`.
  **L119 CN**: 执行 Python 语句 `json.dump(subcorpus1_description, subcorpus1_description_handle)`。
- **L120 EN**: Controls Python flow with `with` logic.
  **L120 CN**: 使用 `with` 逻辑控制 Python 执行流程。
- **L121 EN**: Executes Python statement `os.path.join(subcorpus2_dir, "corpus_description.json"), "w"`.
  **L121 CN**: 执行 Python 语句 `os.path.join(subcorpus2_dir, "corpus_description.json"), "w"`。
- **L122 EN**: Executes Python statement `) as subcorpus2_description_handle:`.
  **L122 CN**: 执行 Python 语句 `) as subcorpus2_description_handle:`。
- **L123 EN**: Executes Python statement `json.dump(subcorpus2_description, subcorpus2_description_handle)`.
  **L123 CN**: 执行 Python 语句 `json.dump(subcorpus2_description, subcorpus2_description_handle)`。
- **L124 EN**: Controls Python flow with `try` logic.
  **L124 CN**: 使用 `try` 逻辑控制 Python 执行流程。
- **L125 EN**: Executes Python statement `combine_training_corpus_lib.combine_corpus(corpus_dir)`.
  **L125 CN**: 执行 Python 语句 `combine_training_corpus_lib.combine_corpus(corpus_dir)`。

### Lines 126-132

````python
    except ValueError:
        print("ValueError")
        # CHECK-DIFFERENT-CORPORA: ValueError


if __name__ == "__main__":
    globals()[sys.argv[1]](*sys.argv[2:])
````
- **L126 EN**: Controls Python flow with `except` logic.
  **L126 CN**: 使用 `except` 逻辑控制 Python 执行流程。
- **L127 EN**: Executes Python statement `print("ValueError")`.
  **L127 CN**: 执行 Python 语句 `print("ValueError")`。
- **L128 EN**: Comment documents nearby script behavior: `CHECK-DIFFERENT-CORPORA: ValueError`.
  **L128 CN**: 注释说明了附近脚本逻辑：`CHECK-DIFFERENT-CORPORA: ValueError`。
- **L129 EN**: Blank line separates nearby declarations or logic blocks.
  **L129 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L130 EN**: Blank line separates nearby declarations or logic blocks.
  **L130 CN**: 空行，用于分隔相邻的声明或逻辑块。
- **L131 EN**: Defines the script entry point used for direct execution.
  **L131 CN**: 定义脚本被直接执行时使用的入口点。
- **L132 EN**: Executes Python statement `globals()[sys.argv[1]](*sys.argv[2:])`.
  **L132 CN**: 执行 Python 语句 `globals()[sys.argv[1]](*sys.argv[2:])`。

## Key Concepts / 关键概念

- EN: lit test runner infrastructure
  - CN: lit 测试运行基础设施
- EN: pattern-based output verification
  - CN: 基于模式的输出校验
- EN: test harness behavior
  - CN: 测试框架行为
- EN: training corpus preparation
  - CN: 训练语料准备
- EN: Python utility scripting
  - CN: Python 工具脚本

## Dependencies / 依赖关系

- EN: `json` supplies JSON serialization helpers.
  - CN: `json` 提供了JSON 序列化辅助工具。
- EN: `os` supplies operating-system services.
  - CN: `os` 提供了操作系统服务。
- EN: `sys` supplies Python runtime state and argv access.
  - CN: `sys` 提供了Python 运行时状态与 argv 访问。
- EN: `mlgo.corpus` supplies supporting Python helpers.
  - CN: `mlgo.corpus` 提供了辅助性的 Python 模块。
