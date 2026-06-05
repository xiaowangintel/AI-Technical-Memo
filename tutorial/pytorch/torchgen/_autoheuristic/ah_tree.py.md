# ah_tree.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torchgen/_autoheuristic/ah_tree.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Collects data, trains heuristics, and evaluates learned rules that influence matmul-related generation or tuning decisions.
- **Purpose (CN)**: 收集数据、训练启发式模型，并评估影响矩阵乘相关生成或调优决策的规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-18
```python
from typing import Any, Optional

import numpy as np
from sklearn.tree import _tree  # type: ignore[import-untyped]


class DecisionTreeNode:
    def __init__(
        self,
        feature: str | None = None,
        threshold: float | None = None,
        left: Optional["DecisionTreeNode"] = None,
        right: Optional["DecisionTreeNode"] = None,
        class_probs: Any = None,
        num_samples: int = 0,
        node_id: int = 0,
    ) -> None:
        self.feature = feature
```
- **EN**: The import section wires together third-party modules such as numpy, sklearn.tree; standard-library modules such as typing for the logic below. It introduces or extends DecisionTreeNode, which hold the primary data model or public surface for this slice of the file. This chunk defines `__init__`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把第三方模块，如 numpy、sklearn.tree；标准库模块，如 typing组织在一起，供下方逻辑使用。 它引入或扩展了 DecisionTreeNode，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `__init__`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 19-35
```python
        self.threshold = threshold
        self.left = left
        self.right = right
        self.class_probs = class_probs
        self.num_samples = num_samples
        self.id = node_id

    def is_leaf(self) -> bool:
        return self.left is None or self.right is None


class DecisionTree:
    """
    Custom decision tree implementation that mimics some of the sklearn API.
    The purpose of this class it to be able to perform transformations, such as custom pruning, which
    does not seem to be easy with sklearn.
    """
```
- **EN**: It introduces or extends DecisionTree, it, which hold the primary data model or public surface for this slice of the file. This chunk defines `is_leaf`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 它引入或扩展了 DecisionTree、it，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `is_leaf`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 37-54
```python
    def __init__(self, sklearn_tree: Any, feature_names: list[str]) -> None:
        self.feature_names = feature_names
        self.root = self._convert_sklearn_tree(sklearn_tree.tree_)
        self.classes_: list[str] = sklearn_tree.classes_

    def _convert_sklearn_tree(
        self, sklearn_tree: Any, node_id: int = 0
    ) -> DecisionTreeNode:
        class_probs = sklearn_tree.value[node_id][0]
        num_samples = sklearn_tree.n_node_samples[node_id]
        if sklearn_tree.feature[node_id] != _tree.TREE_UNDEFINED:
            feature_index = sklearn_tree.feature[node_id]
            feature = self.feature_names[feature_index]
            left = self._convert_sklearn_tree(
                sklearn_tree, sklearn_tree.children_left[node_id]
            )
            right = self._convert_sklearn_tree(
                sklearn_tree, sklearn_tree.children_right[node_id]
```
- **EN**: This chunk defines `_convert_sklearn_tree`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `_convert_sklearn_tree`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 55-71
```python
            )
            return DecisionTreeNode(
                feature=feature,
                threshold=sklearn_tree.threshold[node_id],
                left=left,
                right=right,
                class_probs=class_probs,
                num_samples=num_samples,
                node_id=node_id,
            )
        else:
            return DecisionTreeNode(
                class_probs=class_probs, num_samples=num_samples, node_id=node_id
            )

    def prune(self, df: Any, target_col: str, k: int) -> None:
        self.root = self._prune_tree(self.root, df, target_col, k)
```
- **EN**: This chunk defines `prune`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `prune`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 73-89
```python
    def _prune_tree(
        self, node: DecisionTreeNode, df: Any, target_col: str, k: int
    ) -> DecisionTreeNode:
        if node.is_leaf():
            return node

        left_df = df[df[node.feature] <= node.threshold]
        right_df = df[df[node.feature] > node.threshold]

        # number of unique classes in the left and right subtrees
        left_counts = left_df[target_col].nunique()
        right_counts = right_df[target_col].nunique()

        # for ranking, we want to ensure that we return at least k classes, so if we have less than k classes in the
        # left or right subtree, we remove the split and make this node a leaf node
        if left_counts < k or right_counts < k:
            return DecisionTreeNode(class_probs=node.class_probs)
```
- **EN**: This chunk defines `_prune_tree`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `_prune_tree`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 91-106
```python
        if node.left is None:
            raise AssertionError("expected left child to exist")
        node.left = self._prune_tree(node.left, left_df, target_col, k)
        if node.right is None:
            raise AssertionError("expected right child to exist")
        node.right = self._prune_tree(node.right, right_df, target_col, k)

        return node

    def to_dot(self) -> str:
        dot = "digraph DecisionTree {\n"
        dot += '    node [fontname="helvetica"];\n'
        dot += '    edge [fontname="helvetica"];\n'
        dot += self._node_to_dot(self.root)
        dot += "}"
        return dot
```
- **EN**: This chunk defines `to_dot`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `to_dot`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 108-119
```python
    def _node_to_dot(
        self, node: DecisionTreeNode, parent_id: int = 0, edge_label: str = ""
    ) -> str:
        if node is None:
            return ""

        node_id = id(node)

        # Format class_probs array with line breaks
        class_probs_str = self._format_class_probs_array(
            node.class_probs, node.num_samples
        )
```
- **EN**: This chunk defines `_node_to_dot`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `_node_to_dot`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 121-132
```python
        if node.is_leaf():
            label = class_probs_str
            shape = "box"
        else:
            feature_name = f"{node.feature}"
            label = f"{feature_name} <= {node.threshold:.2f}\\n{class_probs_str}"
            shape = "oval"

        dot = f'    {node_id} [label="{label}", shape={shape}];\n'

        if parent_id != 0:
            dot += f'    {parent_id} -> {node_id} [label="{edge_label}"];\n'
```
- **EN**: This chunk continues `_node_to_dot` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `_node_to_dot`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 134-147
```python
        if not node.is_leaf():
            if node.left is None:
                raise AssertionError("expected left child to exist")
            dot += self._node_to_dot(node.left, node_id, "<=")
            if node.right is None:
                raise AssertionError("expected right child to exist")
            dot += self._node_to_dot(node.right, node_id, ">")

        return dot

    def _format_class_prob(self, num: float) -> str:
        if num == 0:
            return "0"
        return f"{num:.2f}"
```
- **EN**: This chunk defines `_format_class_prob`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `_format_class_prob`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 149-165
```python
    def _format_class_probs_array(
        self, class_probs: Any, num_samples: int, max_per_line: int = 5
    ) -> str:
        # add line breaks to avoid very long lines
        flat_class_probs = class_probs.flatten()
        formatted = [self._format_class_prob(v) for v in flat_class_probs]
        lines = [
            formatted[i : i + max_per_line]
            for i in range(0, len(formatted), max_per_line)
        ]
        return f"num_samples={num_samples}\\n" + "\\n".join(
            [", ".join(line) for line in lines]
        )

    def predict(self, X: Any) -> Any:
        predictions = [self._predict_single(x) for _, x in X.iterrows()]
        return np.array(predictions)
```
- **EN**: This chunk defines `predict`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `predict`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 167-181
```python
    def predict_proba(self, X: Any) -> Any:
        return np.array([self._predict_proba_single(x) for _, x in X.iterrows()])

    def _get_leaf(self, X: Any) -> DecisionTreeNode:
        node = self.root
        while not node.is_leaf():
            if X[node.feature] <= node.threshold:
                if node.left is None:
                    raise AssertionError("expected left child to exist")
                node = node.left
            else:
                if node.right is None:
                    raise AssertionError("expected right child to exist")
                node = node.right
        return node
```
- **EN**: This chunk defines `_get_leaf`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `_get_leaf`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 183-198
```python
    def _predict_single(self, x: Any) -> str:
        node = self._get_leaf(x)
        # map index to class name
        return self.classes_[np.argmax(node.class_probs)]

    def _predict_proba_single(self, x: Any) -> Any:
        node = self._get_leaf(x)
        return node.class_probs

    def apply(self, X: Any) -> Any:
        ids = [self._apply_single(x) for _, x in X.iterrows()]
        return np.array(ids)

    def _apply_single(self, x: Any) -> int:
        node = self._get_leaf(x)
        return node.id
```
- **EN**: It introduces or extends name, which hold the primary data model or public surface for this slice of the file. This chunk defines `_apply_single`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 它引入或扩展了 name，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `_apply_single`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 200-217
```python
    def codegen(
        self,
        dummy_col_2_col_val: dict[str, tuple[str, Any]],
        lines: list[str],
        unsafe_leaves: list[int],
    ) -> None:
        # generates python code for the decision tree
        def codegen_node(node: DecisionTreeNode, depth: int) -> None:
            indent = "    " * (depth + 1)
            if node.is_leaf():
                lines.append(handle_leaf(node, indent, unsafe_leaves))
            else:
                name = node.feature
                threshold = node.threshold
                if name in dummy_col_2_col_val:
                    (orig_name, value) = dummy_col_2_col_val[name]
                    predicate = f"{indent}if str(context.get_value('{orig_name}')) != '{value}':"
                    if threshold != 0.5:
```
- **EN**: This chunk defines `codegen_node`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `codegen_node`，其作用是根据高层元数据生成派生代码、表或分析产物。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 218-232
```python
                        raise AssertionError(
                            f"expected threshold to be 0.5 but is {threshold}"
                        )
                else:
                    predicate = (
                        f"{indent}if context.get_value('{name}') <= {threshold}:"
                    )
                lines.append(predicate)
                if node.left is None:
                    raise AssertionError("expected left child to exist")
                codegen_node(node.left, depth + 1)
                lines.append(f"{indent}else:")
                if node.right is None:
                    raise AssertionError("expected right child to exist")
                codegen_node(node.right, depth + 1)
```
- **EN**: This chunk continues `codegen_node` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `codegen_node`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 234-244
```python
        def handle_leaf(
            node: DecisionTreeNode, indent: str, unsafe_leaves: list[int]
        ) -> str:
            """
            This generates the code for a leaf node in the decision tree. If the leaf is unsafe, the learned heuristic
            will return "unsure" (i.e. None).
            """
            if node.id in unsafe_leaves:
                return f"{indent}return None"
            class_probas = node.class_probs
            return f"{indent}return {best_probas_and_indices(class_probas)}"
```
- **EN**: This chunk defines `handle_leaf`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `handle_leaf`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 246-263
```python
        def best_probas_and_indices(class_probas: Any) -> str:
            """
            Given a list of tuples (proba, idx), this function returns a string in which the tuples are
            sorted by proba in descending order. E.g.:
            Given class_probas=[(0.3, 0), (0.5, 1), (0.2, 2)]
            this function returns
            "[(0.5, 1), (0.3, 0), (0.2, 2)]"
            """
            # we generate a list of tuples (proba, idx) sorted by proba in descending order
            # idx is the index of a choice
            # we only generate a tuple if proba > 0
            probas_indices_sorted = sorted(
                [
                    (proba, index)
                    for index, proba in enumerate(class_probas)
                    if proba > 0
                ],
                key=lambda x: x[0],
```
- **EN**: This chunk defines `best_probas_and_indices`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `best_probas_and_indices`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 264-271
```python
                reverse=True,
            )
            probas_indices_sorted_str = ", ".join(
                f"({value:.3f}, {index})" for value, index in probas_indices_sorted
            )
            return f"[{probas_indices_sorted_str}]"

        codegen_node(self.root, 1)
```
- **EN**: This chunk continues `best_probas_and_indices` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `best_probas_and_indices`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

## Key Concepts / 关键概念

- **Torch operator code generation**
  - EN: Builds derived code artifacts from declarative operator metadata.
  - CN: 从声明式算子元数据构建派生代码产物。
- **Learned heuristics**
  - EN: Collects data or trains rules that guide performance-sensitive decisions.
  - CN: 收集数据或训练规则，以指导性能敏感的决策。
- **DecisionTreeNode**
  - EN: `DecisionTreeNode` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `DecisionTreeNode` 是本文件声明、导出或驱动的显著符号之一。
- **__init__**
  - EN: `__init__` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `__init__` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Third-party modules / 第三方模块**: `numpy`, `sklearn.tree`
- **Standard library / 标准库**: `typing`
- **Primary symbols / 核心符号**: `DecisionTreeNode`, `__init__`, `is_leaf`, `DecisionTree`, `_convert_sklearn_tree`, `prune`, `_prune_tree`, `to_dot`, `_node_to_dot`, `_format_class_prob`
