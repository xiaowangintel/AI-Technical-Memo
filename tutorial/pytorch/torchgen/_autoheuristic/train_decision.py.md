# train_decision.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `torchgen/_autoheuristic/train_decision.py`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Collects data, trains heuristics, and evaluates learned rules that influence matmul-related generation or tuning decisions.
- **Purpose (CN)**: 收集数据、训练启发式模型，并评估影响矩阵乘相关生成或调优决策的规则。
## Line-by-Line Analysis / 逐行分析

### Lines 1-38
```python
# mypy: ignore-errors

import itertools
import json
import logging
import math
import warnings


warnings.filterwarnings(
    "ignore",
    message="The behavior of DataFrame concatenation with empty or all-NA entries is deprecated",
)

from dataclasses import dataclass

import numpy as np
import pandas as pd  # type: ignore[import-untyped]
from ah_tree import DecisionTree
from scipy.stats import gmean
from sklearn.model_selection import train_test_split
from sklearn.tree import DecisionTreeClassifier
from train import AHTrain


log = logging.getLogger(__name__)
DEBUG = True
if DEBUG:
    ch = logging.StreamHandler()
    ch.setLevel(logging.DEBUG)
    formatter = logging.Formatter(
        "%(asctime)s - %(message)s", datefmt="%Y-%m-%d %H:%M:%S"
    )
    ch.setFormatter(formatter)
    log.addHandler(ch)


class AHTrainDecisionTree(AHTrain):
```
- **EN**: The import section wires together third-party modules such as numpy, pandas, ah_tree, and 4 more; standard-library modules such as itertools, json, logging, math, and 2 more for the logic below. It introduces or extends AHTrainDecisionTree, which hold the primary data model or public surface for this slice of the file. This chunk continues `AHTrainDecisionTree` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 导入区把第三方模块，如 numpy、pandas、ah_tree 等共 7 项；标准库模块，如 itertools、json、logging、math 等共 6 项组织在一起，供下方逻辑使用。 它引入或扩展了 AHTrainDecisionTree，这些类型承载了本段的核心数据模型或对外接口。 这一段延续了 `AHTrainDecisionTree`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 39-77
```python
    def __init__(self):
        super().__init__()

    def debug_time(self, row, top_k_choices):
        choices_feedback = json.loads(row["choice2time"])
        timings = sorted(choices_feedback.items(), key=lambda x: x[1])
        for choice, time in timings:
            result = f"{choice} {time}"
            if choice in top_k_choices:
                result += " TOPK"
            print(result)

    def is_unsafe_leaf(self, row, predicted_config, choice2time):
        """
        Can be overridden by subclasses to define their own logic for deciding when a leaf is unsafe. Returns a sample
        that landed in the leaf, the choice predicted by the tree, and a dictionary that maps each choice to the
        execution time. One can for example decide to mark a leaf as unsafe if the predicted choice is 2x slower
        than the fastest choice.
        If a leaf is unsafe, the learned heuristic will always return 'unsure' if an input lands in that leaf.
        """

        return False

    def get_unsafe_leaves(self, model, df, feature_columns):
        """
        Given a trained decision tree, and a dataframe containing the training data, returns a list of unsafe leaves.
        """
        X = df[feature_columns]
        leaf_ids = model.apply(X)
        unique_leaves = np.unique(leaf_ids)

        unsafe_leaves = []
        # Iterate over each leaf
        for leaf in unique_leaves:
            leaf_mask = leaf_ids == leaf
            # Get samples that land in this leaf
            leaf_X = X[leaf_mask]

            predicted_config = model.predict(leaf_X.iloc[[0]])[0]
```
- **EN**: This chunk defines `get_unsafe_leaves`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `get_unsafe_leaves`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 79-112
```python
            # For each sample, check if we should mark the leaf as unsafe
            for idx, row in leaf_X.iterrows():
                choice2time = json.loads(df.loc[idx, "choice2time"])
                if self.is_unsafe_leaf(row, predicted_config, choice2time):
                    unsafe_leaves.append(leaf)
                    break
        return unsafe_leaves

    def get_allowed_wrong_prediction_pct(self):
        """
        This is used to determine a threshold for when a learned heuristic returns 'unsure'.
        If this function returns 0.01, we will set the probability required for the decision tree to return a decision
        such that at most 1% of the predictions will be wrong on the validation set.
        """
        return 0.01

    def get_grid_search_values(self):
        """
        Standard values for grid search. Can be overridden.
        """
        return {
            "max_depth": [5, 6, 7],
            "min_samples_leaf": [1, 5, 10, 0.01, 0.05, 0.02],
            "criterion": ["gini", "entropy"],
        }

    def predict(self, model, df, feature_columns):
        """
        Returns the predictions, probabilities, and leaf ids for a given dataframe.
        """
        predictions = model.predict(df[feature_columns])
        proba = model.predict_proba(df[feature_columns])
        leaf_ids = model.apply(df[feature_columns])
        return predictions, proba, leaf_ids
```
- **EN**: This chunk defines `predict`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `predict`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 114-153
```python
    def ranking_num_choices(self):
        # if the heuristic is used for ranking, this function returns the number
        # of choices that the heuristic will return
        if self.args.ranking is None:
            return 5
        return self.args.ranking

    def train_and_evaluate_models(
        self,
        datasets,
        max_depths,
        min_samples_leafs,
        criterion_list,
        feature_columns,
        ranking=False,
    ):
        """
        Does a grid search over max_depths, min_samples_leafs, and criterion_list and returns the best model.
        """

        results = []
        best_model = None
        best_model_safe_proba = 0
        best_model_num_correct = 0
        best_model_unsafe_leaves = []
        columns = ["set", "crit", "max_depth", "min_samples_leaf"]
        metrics_columns = []
        for max_depth, min_samples_leaf, criterion in itertools.product(
            max_depths, min_samples_leafs, criterion_list
        ):
            print(
                f"max_depth={max_depth} min_samples_leaf={min_samples_leaf} criterion={criterion}"
            )
            model = DecisionTreeClassifier(
                max_depth=max_depth,
                min_samples_leaf=min_samples_leaf,
                criterion=criterion,
                random_state=42,
            )
            df_train = datasets["train"]
```
- **EN**: This chunk defines `train_and_evaluate_models`, which runs a training procedure that fits a heuristic or predictive model. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `train_and_evaluate_models`，其作用是执行训练过程以拟合启发式或预测模型。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 154-186
```python
            df_val = datasets["val"]
            if ranking:
                model.fit(
                    df_train[feature_columns],
                    df_train["winner"],
                    sample_weight=df_train["relative_performance"],
                )
            else:
                model.fit(df_train[feature_columns], df_train["winner"])

            model = DecisionTree(model, feature_columns)

            if ranking:
                model.prune(df_train, "winner", k=self.ranking_num_choices())

            unsafe_leaves = self.get_unsafe_leaves(model, df_train, feature_columns)
            predictions, proba, leaf_ids = self.predict(model, df_val, feature_columns)

            wrong_pct = self.get_allowed_wrong_prediction_pct()
            evaluator = DecisionEvaluator(
                self,
                model,
                predictions,
                df_val,
                proba,
                wrong_pct=wrong_pct,
                unsafe_leaves=unsafe_leaves,
                leaf_ids=leaf_ids,
                k=self.ranking_num_choices(),
                ranking=ranking,
            )
            safe_proba = evaluator.get_safe_proba()
            print(f"safe_proba={safe_proba}")
```
- **EN**: This chunk continues `train_and_evaluate_models` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `train_and_evaluate_models`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 188-225
```python
            def eval(name, df):
                if ranking:
                    # when ranking is enabled, we duplicate each input for each choice that
                    # is almost as good as the best choice
                    # we do not want to evaluate the same input multiple times, so we remove duplicates here
                    df = df[df["winner"] == df["actual_winner"]]
                predictions, proba, leaf_ids = self.predict(model, df, feature_columns)
                evaluator = DecisionEvaluator(
                    self,
                    model,
                    predictions,
                    df,
                    proba,
                    wrong_pct=wrong_pct,
                    threshold=safe_proba,
                    unsafe_leaves=unsafe_leaves,
                    leaf_ids=leaf_ids,
                    k=self.ranking_num_choices(),
                    ranking=ranking,
                )
                return evaluator.get_results()

            for dataset_name, dataset in datasets.items():
                eval_result: EvalResults = eval(dataset_name, dataset)
                eval_result_metrics = eval_result.to_map()
                if dataset_name == "val":
                    num_correct = eval_result.accuracy.num_correct
                    num_wrong = eval_result.accuracy.num_wrong
                    num_total = eval_result.accuracy.total
                    if num_wrong <= num_total * wrong_pct:
                        if num_correct > best_model_num_correct:
                            print(
                                f"new best model with {num_correct} correct and {num_wrong} wrong"
                            )
                            best_model = model
                            best_model_num_correct = num_correct
                            best_model_safe_proba = safe_proba
                            best_model_unsafe_leaves = unsafe_leaves
```
- **EN**: This chunk defines `eval`, which measures behavior so implementations or heuristics can be compared. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `eval`，其作用是测量行为，以便比较不同实现或启发式规则。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 227-265
```python
                result = (dataset_name, criterion, max_depth, min_samples_leaf)
                result += tuple(eval_result_metrics.values())
                results.append(result)
                if len(metrics_columns) == 0:
                    metrics_columns = list(eval_result_metrics.keys())
                    columns += metrics_columns

        return (
            pd.DataFrame(results, columns=columns),
            best_model,
            best_model_safe_proba,
            best_model_unsafe_leaves,
        )

    def get_test_and_val_size(self):
        """
        Returns the size of the test and validation sets.
        """
        return (0.15, 0.15)

    def prepare_datasets(self, df, other_datasets, cat_feature2cats, ranking=False):
        """
        Splits the dataframe into train, val, and test sets.
        Also adds other datasets, specified by the user, to the train set.
        """
        test_size, val_size = self.get_test_and_val_size()
        # Split into train+val and test
        df_train_val, df_test = train_test_split(
            df, test_size=test_size, random_state=42
        )

        # Split train+val inputs into train and val
        train_val_size = 1 - test_size
        df_train, df_val = train_test_split(
            df_train_val, test_size=val_size / train_val_size, random_state=42
        )
        datasets = {"train": df_train, "val": df_val, "test": df_test}
        self.add_real_datasets(datasets, other_datasets, cat_feature2cats, ranking)
        return datasets
```
- **EN**: This chunk defines `prepare_datasets`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `prepare_datasets`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 267-294
```python
    def export_to_dot(self, best_model, df, feature_columns):
        """
        Export a learned decision tree to a dot file.
        """
        dot_str = best_model.to_dot()
        with open("best_model.dot", "w") as f:
            f.write(dot_str)

    def get_feature_columns(self, df):
        """
        The dataframe contains columns that are not features, such as 'winner', 'speedup' that are only used for
        debugging purposes. This function returns the columns that are actually features.
        """
        exclude_columns = [
            "speedup",
            "winner",
            "target",
            "avail_choices",
            "choice2time",
            "index",
            "actual_winner",
            "relative_performance",
        ]
        feature_columns = [col for col in df.columns if col not in exclude_columns]
        return feature_columns

    def add_training_data(self, df_train, datasets):
        return datasets["train"]
```
- **EN**: This chunk defines `add_training_data`, which runs a training procedure that fits a heuristic or predictive model. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `add_training_data`，其作用是执行训练过程以拟合启发式或预测模型。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 296-335
```python
    def main(
        self,
        log_path,
        other_datasets,
        nrows,
        heuristic_name,
        save_dot=False,
        ranking=False,
    ):
        """
        Main function that trains a decision tree and generates a heuristic.
        """
        # TODO: Enable apply_filters
        (df, choices, cat_feature2cats, dummy_col_2_col_val, metadata) = self.get_df(
            log_path, nrows=nrows, apply_filters=False, add_near_best=ranking
        )
        self.dummy_col_2_col_val = dummy_col_2_col_val
        datasets = self.prepare_datasets(df, other_datasets, cat_feature2cats, ranking)
        df_train = self.add_training_data(datasets["train"], datasets)
        datasets["train"] = df_train
        print(datasets["train"]["winner"].value_counts().to_string())

        feature_columns = self.get_feature_columns(df)
        grid_search_values = self.get_grid_search_values()
        max_depths = grid_search_values["max_depth"]
        min_samples_leafs = grid_search_values["min_samples_leaf"]
        criterion_list = grid_search_values["criterion"]
        (
            results_df,
            best_model,
            best_model_safe_proba,
            unsafe_leaves,
        ) = self.train_and_evaluate_models(
            datasets,
            max_depths,
            min_samples_leafs,
            criterion_list,
            feature_columns,
            ranking=ranking,
        )
```
- **EN**: This chunk defines `main`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `main`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 337-375
```python
        if ranking:
            columns_to_keep = [
                "set",
                "crit",
                "max_depth",
                "min_samples_leaf",
                "total",
                "top_k_correct",
                "top_k_wrong",
                "top_k_unsure",
                "wrong_max_speedup_k",
                "wrong_gmean_speedup_k",
            ]
            results_df = results_df[columns_to_keep]
        # prints results for all models and datasets
        print(results_df.to_string())

        sort_metric = "top_k_correct" if ranking else "correct"
        # prints results grouped by dataset
        for set_name in results_df["set"].unique():
            dataset_results = results_df[results_df["set"] == set_name]
            dataset_results = dataset_results.sort_values(by=sort_metric)
            print(dataset_results.to_string() + "\n")

        if best_model is not None:
            if save_dot:
                self.export_to_dot(best_model, df, feature_columns)
            self.codegen(
                best_model,
                metadata,
                heuristic_name,
                best_model_safe_proba,
                dummy_col_2_col_val,
                unsafe_leaves,
            )
        else:
            print(
                "All learned models have too many wrong predictions, so no heuristic was generated"
            )
```
- **EN**: This chunk continues `main` and expands its control flow, data preparation, or emitted structure. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段延续了 `main`，继续展开其控制流、数据准备或生成结构。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 377-413
```python
    def get_df(
        self,
        log_path,
        cat_feature2cats=None,
        nrows=None,
        apply_filters=False,
        add_near_best=False,
    ):
        """
        Parses the log file and processes the data into a dataframe that can be used for training.
        """
        (df, metadata, features, categorical_features, choices) = self.parse_log(
            log_path, nrows
        )

        def calculate_stats(group):
            count = len(group)
            has_inf = np.isinf(group["feedback"]).any()
            if has_inf:
                relative_std = np.inf
                median = np.inf
            else:
                mean = group["feedback"].mean()
                std = group["feedback"].std()
                relative_std = (std / mean) * 100 if mean != 0 else np.inf
                median = group["feedback"].median()
            if relative_std > 5:
                times = group["feedback"].tolist()
                times_str = ", ".join([f"{t:.3f}" for t in sorted(times)])
                log.debug("High relative std: %f. times=%s", relative_std, times_str)
            return pd.Series(
                {
                    "count": count,
                    "relative_std": relative_std,
                    "median_execution_time": median,
                }
            )
```
- **EN**: This chunk defines `calculate_stats`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `calculate_stats`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 415-448
```python
        feature_columns = features
        stats = (
            df.groupby(feature_columns + ["choice"], as_index=False)
            .apply(calculate_stats, include_groups=False)
            .reset_index()
        )

        # TODO: We have to be careful with removing certain choices, because if we e.g. remove the winner, the
        # heuristic will end up learning wrong things. But, execution times with high variance are also bad
        if apply_filters:
            # Filter out inputs with less than 3 measurements or high relative std
            valid_stats = stats[(stats["count"] >= 3) & (stats["relative_std"] <= 5)]
            # Group by input features and count how many valid choices we have for each input
            valid_inputs = valid_stats.groupby(feature_columns).filter(
                lambda x: len(x) >= 2
            )
        else:
            valid_inputs = stats

        # Compute the winner and speedup for each valid input
        def get_winner_and_speedup(group):
            if len(group) < 2:
                raise AssertionError("Need at least 2 choices")

            sorted_group = group.sort_values("median_execution_time")
            winner = sorted_group.iloc[0]["choice"]
            winning_time = sorted_group.iloc[0]["median_execution_time"]
            second_best_time = sorted_group.iloc[1]["median_execution_time"]
            speedup = second_best_time / winning_time
            unique_choices = group["choice"].unique()

            choice2time = {}
            for row in group.itertuples():
                choice2time[row.choice] = row.median_execution_time
```
- **EN**: This chunk defines `get_winner_and_speedup`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `get_winner_and_speedup`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 450-478
```python
            if len(unique_choices) != len(group):
                raise AssertionError(
                    f"len(unique_choices) != len(group): "
                    f"{len(unique_choices)} != {len(group)}"
                )

            return pd.Series(
                {
                    "winner": winner,
                    "speedup": speedup,
                    "avail_choices": unique_choices,
                    "choice2time": json.dumps(choice2time),
                }
            )

        results = (
            valid_inputs.groupby(feature_columns, as_index=False)
            .filter(lambda x: len(x) >= 2)
            .groupby(feature_columns, as_index=False)
            .apply(get_winner_and_speedup, include_groups=False)
            .reset_index()
        )

        def add_near_best_configs(df):
            new_rows = []

            for index, row in df.iterrows():
                dictionary = json.loads(row["choice2time"])
                min_value = min(dictionary.values())
```
- **EN**: This chunk defines `add_near_best_configs`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `add_near_best_configs`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 480-508
```python
                for key, value in dictionary.items():
                    new_row = row.copy()
                    relative_performance = min_value / value
                    new_row["relative_performance"] = relative_performance
                    if relative_performance is None or relative_performance is np.inf:
                        breakpoint()
                    new_row["actual_winner"] = row["winner"]
                    new_row["winner"] = key
                    if relative_performance >= 0.98:
                        new_rows.append(new_row)

            return pd.DataFrame(new_rows).reset_index(drop=True)

        if add_near_best:
            results = add_near_best_configs(results)
        (results, added_categorical_features) = self.add_new_features(results)
        categorical_features += added_categorical_features

        (
            results,
            cat_feature2cats,
            dummy_col_2_col_val,
        ) = self.handle_categorical_features(
            cat_feature2cats, categorical_features, results
        )
        return (results, choices, cat_feature2cats, dummy_col_2_col_val, metadata)

    def ranking_always_included_choices(self):
        return []
```
- **EN**: This chunk defines `ranking_always_included_choices`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `ranking_always_included_choices`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 510-548
```python
    def gen_classes(self, classes, num_spaces):
        """
        If classes=['choice1', 'choice2', 'choice3'], then this function returns
        the following string:
        self.choices.append('choice1')
        self.choices.append('choice2')
        self.choices.append('choice3')
        Used in the generated heuristic to map the index of a choice to its name.
        """
        indent = " " * num_spaces
        return "\n".join([f"{indent}self.choices.append('{c}')" for c in classes])

    def get_default_config(self, row):
        """
        Returns the default config for a given sample. The default config could for example be the config that is
        the chosen by a current handwritten heuristic. This can for example be used in get_unsafe_leaf to
        compare the predicted config with the default config.
        """
        return None

    def gen_predict_fn_def(self):
        """
        Generates the definition of the predict function.
        """
        return "def get_best_choices(self, context: AHContext) -> Optional[list[tuple[float, int]]]:"

    def codegen_boilerplate(
        self, heuristic_name, opt_name, threshold, shared_memory, device_capa, classes
    ):
        """
        Generates the boilerplate code for the generated heuristic. This includes things like imports, class definition,
        etc.
        """

        boiler_plate = f"""# flake8: noqa: B950
# fmt: off
# This file was generated by AutoHeuristic. Do not modify it manually!
# To regenerate this file, take a look at the steps in the README.md file inside torchgen/_autoheuristic/{opt_name}/
from typing import Optional
```
- **EN**: The import section wires together standard-library modules such as typing for the logic below. It introduces or extends definition, which hold the primary data model or public surface for this slice of the file. This chunk defines `codegen_boilerplate`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把标准库模块，如 typing组织在一起，供下方逻辑使用。 它引入或扩展了 definition，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `codegen_boilerplate`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 550-581
```python
from torch._inductor.autoheuristic.autoheuristic_utils import (
    AHContext,
    AHMetadata,
    Choice,
)
from torch._inductor.autoheuristic.learnedheuristic_interface import (
    LearnedHeuristicDecision,
)


class {heuristic_name}(LearnedHeuristicDecision):

    def __init__(self) -> None:
        self.choices: list[Choice] = []
        self.fill_choices()

{self.gen_precondition(opt_name, shared_memory, device_capa)}

    def get_confidence_threshold(self) -> float:
        return {threshold}

    def get_choice(self, idx: int) -> Optional[str]:
        if idx < len(self.choices):
            return self.choices[idx]
        return None

    def fill_choices(self) -> None:
{self.gen_classes(classes, num_spaces=8)}

    def get_name(self) -> str:
        return '{opt_name}'"""
        return boiler_plate
```
- **EN**: The import section wires together PyTorch-local modules such as torch._inductor.autoheuristic.autoheuristic_utils, torch._inductor.autoheuristic.learnedheuristic_interface for the logic below. This chunk defines `get_name`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 导入区把PyTorch 本地模块，如 torch._inductor.autoheuristic.autoheuristic_utils、torch._inductor.autoheuristic.learnedheuristic_interface组织在一起，供下方逻辑使用。 这一段定义了 `get_name`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 583-622
```python
    def add_real_datasets(
        self, datasets, other_datasets, cat_feature2cats, ranking=False
    ):
        """
        Adds datasets specified by the user to the datasets dictionary.
        """
        if other_datasets:
            for name, path in other_datasets:
                (df_other, choices, _, _, _) = self.get_df(
                    path,
                    cat_feature2cats=cat_feature2cats,
                    apply_filters=False,
                    add_near_best=ranking,
                )
                datasets[name] = df_other

    def codegen(
        self,
        tree,
        metadata,
        heuristic_name,
        threshold,
        dummy_col_2_col_val,
        unsafe_leaves,
    ):
        lines = []
        device_capa = metadata["device_capa"]
        device_capa_str = f"({device_capa[0]}, {device_capa[1]})"
        opt_name = metadata["name"]
        lines.append(
            self.codegen_boilerplate(
                heuristic_name,
                opt_name,
                threshold,
                metadata["shared_memory"],
                device_capa_str,
                tree.classes_,
            )
        )
        fn_def = f"\n    {self.gen_predict_fn_def()}"
```
- **EN**: This chunk defines `codegen`, which generates derived code, tables, or analysis artifacts from higher-level metadata. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `codegen`，其作用是根据高层元数据生成派生代码、表或分析产物。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 623-659
```python
        lines.append(fn_def)
        tree.codegen(dummy_col_2_col_val, lines, unsafe_leaves)
        self.write_heuristic_to_file(lines, heuristic_name)


@dataclass
class AccuracyMetrics:
    # Number of correct predictions
    num_correct: int
    # Number of wrong predictions
    num_wrong: int
    # Number of predictions where model is unsure
    num_unsure: int
    # Total number of predictions
    total: int

    def to_map(self):
        return {
            "correct": self.num_correct,
            "wrong": self.num_wrong,
            "unsure": self.num_unsure,
            "total": self.total,
        }


@dataclass
class WrongSpeedupMetrics:
    # If the model predicted the wrong choice, this is the maximum speedup of the best choice over the predicted choice
    max_speedup: float
    # For all wrong predictions, this is the geometric mean of the speedups of the best choices over the predicted choices
    gmean_speedup: float

    def to_map(self):
        return {
            "wrong_max_speedup": self.max_speedup,
            "wrong_gmean_speedup": self.gmean_speedup,
        }
```
- **EN**: Decorators such as @dataclass, @dataclass modify how the following Python definitions are exported, wrapped, or dataclass-managed. It introduces or extends AccuracyMetrics, WrongSpeedupMetrics, which hold the primary data model or public surface for this slice of the file. This chunk defines `to_map`, which implements one step in the operator code-generation pipeline. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @dataclass、@dataclass 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 它引入或扩展了 AccuracyMetrics、WrongSpeedupMetrics，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `to_map`，其作用是实现算子代码生成流水线中的一个步骤。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 662-697
```python
@dataclass
class RankingMetrics:
    # Number of predictions where best choice is in top k choices
    num_correct: int
    # Number of predictions where best choice is not in top k choices
    num_wrong: int
    # Maximum speedup of best choice over best choice in top k (this tells us how much better the best choice, which
    # is not in top k, is over the best choice in top k)
    max_speedup: float
    # Geometric mean of speedups of best choice over best choice in top k
    gmean_speedup: float
    # Number of predictions where model is unsure
    unsure: int

    def to_map(self):
        return {
            "top_k_correct": self.num_correct,
            "top_k_wrong": self.num_wrong,
            "wrong_max_speedup_k": self.max_speedup,
            "wrong_gmean_speedup_k": self.gmean_speedup,
            "top_k_unsure": self.unsure,
        }


@dataclass
class DefaultComparisonMetrics:
    # Maximum speedup of predicted choice over default choice
    max_speedup: float
    # Geometric mean of speedups of predicted choices over default choices
    gmean_speedup: float
    # Maximum speedup of default choice over predicted choice
    max_slowdown: float
    # Number of predictions where the predicted choice is not the default choice
    non_default_predictions: int
    # Number of predictions where the default choice is better than the predicted choice
    default_better: bool
```
- **EN**: Decorators such as @dataclass, @dataclass modify how the following Python definitions are exported, wrapped, or dataclass-managed. It introduces or extends RankingMetrics, DefaultComparisonMetrics, which hold the primary data model or public surface for this slice of the file. This chunk defines `to_map`, which implements one step in the operator code-generation pipeline. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @dataclass、@dataclass 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 它引入或扩展了 RankingMetrics、DefaultComparisonMetrics，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `to_map`，其作用是实现算子代码生成流水线中的一个步骤。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 699-725
```python
    def to_map(self):
        return {
            "max_speedup_over_default": self.max_speedup,
            "gmean_speedup_over_default": self.gmean_speedup,
            "max_speedup_default_over_heuristic": self.max_slowdown,
            "non_default_predictions": self.non_default_predictions,
            "default_better": self.default_better,
        }


@dataclass
class EvalResults:
    accuracy: AccuracyMetrics
    speedup: WrongSpeedupMetrics
    ranking: RankingMetrics
    default_comparison: DefaultComparisonMetrics

    def to_map(self):
        return {
            **self.accuracy.to_map(),
            **self.speedup.to_map(),
            **self.ranking.to_map(),
            **self.default_comparison.to_map(),
        }


class DecisionEvaluator:
```
- **EN**: Decorators such as @dataclass modify how the following Python definitions are exported, wrapped, or dataclass-managed. It introduces or extends EvalResults, DecisionEvaluator, which hold the primary data model or public surface for this slice of the file. This chunk defines `to_map`, which implements one step in the operator code-generation pipeline. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 像 @dataclass 这样的装饰器会调整后续 Python 定义的导出、包装或 dataclass 行为。 它引入或扩展了 EvalResults、DecisionEvaluator，这些类型承载了本段的核心数据模型或对外接口。 这一段定义了 `to_map`，其作用是实现算子代码生成流水线中的一个步骤。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 726-763
```python
    def __init__(
        self,
        train,
        model,
        predictions,
        df,
        probas,
        wrong_pct=0.01,
        threshold=0.0,
        k=10,
        unsafe_leaves=None,
        leaf_ids=None,
        ranking=False,
    ) -> None:
        self.train = train
        self.model = model
        self.predictions = predictions
        self.df = df
        self.probas = probas
        self.wrong_pct = wrong_pct
        self.threshold = threshold
        self.k = k
        self.unsafe_leaves = unsafe_leaves
        self.leaf_ids = leaf_ids
        self.ranking = ranking

        self.num_correct = 0
        self.num_wrong = 0
        self.num_unsure = 0
        self.wrong_probas = []
        self.speedups_wrong = []
        self.num_correct_top_k = 0
        self.num_wrong_top_k = 0
        self.wrong_speedups_top_k = []
        self.top_k_unsure = 0
        self.num_non_default_predictions = 0
        self.speedups_over_default = []
        self.num_default_better = 0
```
- **EN**: This chunk defines `__init__`, which implements one step in the operator code-generation pipeline. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `__init__`，其作用是实现算子代码生成流水线中的一个步骤。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 765-801
```python
    def compute_speedup_over_default(self, default_config, pred, i, predicted_time):
        if default_config is not None:
            if pred != default_config:
                self.num_non_default_predictions += 1
            default_time = self.get_time(self.df.iloc[i], default_config)
            # TODO: We should keep track of how often this happens
            if default_time is not None and not math.isinf(default_time):
                speedup_over_default = default_time / predicted_time
                if speedup_over_default < 1:
                    self.num_default_better += 1
                self.speedups_over_default.append(speedup_over_default)
            else:
                log.debug(
                    "cannot compute speedup over default because default_time=%d",
                    default_time,
                )

    def get_time(self, row, choice):
        choices_feedback = json.loads(row["choice2time"])
        return choices_feedback.get(choice, None)

    def top_k_classes(self, model, probas, k, avail_choices):
        # Get classes and their corresponding probabilities
        classes = model.classes_

        # Sort by probability (descending) and filter out zero probabilities
        sorted_classes = [
            c
            for c, p in sorted(zip(classes, probas), key=lambda x: x[1], reverse=True)
            if p > 0 and c in avail_choices
        ]

        # Return top k choices
        top_k_choices = sorted_classes[:k]
        top_k_choices += self.train.ranking_always_included_choices()
        top_k_choices = list(dict.fromkeys(top_k_choices))
        return top_k_choices
```
- **EN**: This chunk defines `top_k_classes`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `top_k_classes`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 803-842
```python
    def eval_prediction(
        self, avail_choices, leaf_id, pred, true, prob, threshold, default_config, i
    ):
        predicted_time = self.get_time(self.df.iloc[i], pred)
        max_prob = max(prob)
        if (
            leaf_id in self.unsafe_leaves
            or pred not in avail_choices
            or (max_prob != 1.0 and max_prob <= threshold)
        ):
            self.num_unsure += 1
            self.speedups_over_default.append(1.0)
        elif pred == true:
            self.compute_speedup_over_default(default_config, pred, i, predicted_time)
            self.num_correct += 1
        else:
            self.compute_speedup_over_default(default_config, pred, i, predicted_time)
            self.num_wrong += 1
            self.wrong_probas.append(max_prob)
            best_time = self.get_time(self.df.iloc[i], true)
            wrong_speedup = predicted_time / best_time
            self.speedups_wrong.append(wrong_speedup)

    def eval_ranking_prediction(self, true, top_k_choices, i):
        if true in top_k_choices:
            self.num_correct_top_k += 1
        else:
            top_k_choices_times = []
            for choice in top_k_choices:
                time = self.get_time(self.df.iloc[i], choice)
                if time is not None:
                    top_k_choices_times.append(time)
            best_time = self.get_time(self.df.iloc[i], true)
            min_time = min(top_k_choices_times, default=None)
            if min_time is not None:
                speedup = min_time / best_time
                self.wrong_speedups_top_k.append(speedup)
                self.num_wrong_top_k += 1
            else:
                self.top_k_unsure += 1
```
- **EN**: This chunk defines `eval_ranking_prediction`, which measures behavior so implementations or heuristics can be compared. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state.
- **CN**: 这一段定义了 `eval_ranking_prediction`，其作用是测量行为，以便比较不同实现或启发式规则。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。

### Lines 843-882
```python
                # TODO (AlnisM): print more info (input and choices)
                log.debug(
                    "All top k choices have no time which means all top k are unavailable"
                )

    def get_safe_proba(self):
        return self.get_results(return_safe_proba=True)

    def compute_safe_proba(self, num_predictions, wrong_probas, wrong_pct):
        wrong_probas.sort()
        num_wrong = len(wrong_probas)
        allowed_wrong = int(num_predictions * wrong_pct)
        if allowed_wrong >= num_wrong:
            return 0.0
        too_many_wrong = num_wrong - allowed_wrong
        idx = min(too_many_wrong, len(wrong_probas) - 1)
        return wrong_probas[idx]

    def get_results(self, return_safe_proba=False) -> EvalResults:
        """
        Custom evaluation function that evaluates a learned decision tree.
        """

        y_true = self.df["actual_winner"] if self.ranking else self.df["winner"]
        for i, (pred, true, prob, leaf_id) in enumerate(
            zip(self.predictions, y_true, self.probas, self.leaf_ids)
        ):
            avail_choices = self.df["avail_choices"].iloc[i]
            top_k_choices = self.top_k_classes(
                self.model, prob, k=self.k, avail_choices=avail_choices
            )
            if true not in avail_choices:
                raise AssertionError(
                    f"Best choice {true} not in available choices {avail_choices}"
                )
            default_config = self.train.get_default_config(self.df.iloc[i])
            self.eval_prediction(
                avail_choices,
                leaf_id,
                pred,
```
- **EN**: This chunk defines `get_results`, which implements one step in the operator code-generation pipeline. Looping logic applies the same rule across collections, operators, dimensions, workers, or table entries. Conditional branches preserve special cases and keep invariants explicit. Assertions or exception paths ensure invalid states fail early instead of leaking downstream. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `get_results`，其作用是实现算子代码生成流水线中的一个步骤。 循环逻辑会把同一规则应用到集合、算子、维度、工作线程或表项上。 条件分支用于保留特殊情况，并显式维护不变量。 断言或异常路径可让非法状态尽早失败，而不是把问题泄漏到下游。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 883-922
```python
                true,
                prob,
                self.threshold,
                default_config,
                i,
            )
            self.eval_ranking_prediction(true, top_k_choices, i)

        total = len(self.predictions)
        if return_safe_proba:
            return self.compute_safe_proba(total, self.wrong_probas, self.wrong_pct)

        def safe_gmean(x):
            return gmean(x) if x else 0

        max_speedup = max(self.speedups_wrong, default=0)
        gmean_speedup = safe_gmean(self.speedups_wrong)
        max_speedup_top_k = max(self.wrong_speedups_top_k, default=0)
        gmean_speedup_top_k = safe_gmean(self.wrong_speedups_top_k)
        max_speedup_over_default = max(self.speedups_over_default, default=0)
        gmean_speedup_over_default = safe_gmean(self.speedups_over_default)
        max_slowdown_over_default = min(self.speedups_over_default, default=0)

        accuracyMetrics = AccuracyMetrics(
            self.num_correct, self.num_wrong, self.num_unsure, total
        )
        wrongSpeedupMetrics = WrongSpeedupMetrics(max_speedup, gmean_speedup)
        rankingMetrics = RankingMetrics(
            self.num_correct_top_k,
            self.num_wrong_top_k,
            max_speedup_top_k,
            gmean_speedup_top_k,
            self.top_k_unsure,
        )
        defaultComparisonMetrics = DefaultComparisonMetrics(
            max_speedup_over_default,
            gmean_speedup_over_default,
            max_slowdown_over_default,
            self.num_non_default_predictions,
            self.num_default_better,
```
- **EN**: This chunk defines `safe_gmean`, which implements one step in the operator code-generation pipeline. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段定义了 `safe_gmean`，其作用是实现算子代码生成流水线中的一个步骤。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

### Lines 923-934
```python
        )
        return EvalResults(
            accuracyMetrics,
            wrongSpeedupMetrics,
            rankingMetrics,
            defaultComparisonMetrics,
        )


if __name__ == "__main__":
    train = AHTrainDecisionTree()
    train.generate_heuristic()
```
- **EN**: This chunk continues `safe_gmean` and expands its control flow, data preparation, or emitted structure. Conditional branches preserve special cases and keep invariants explicit. Assignments in this block assemble configuration, exported symbols, or intermediate transform state. The tail hands back the assembled result or the updated state for the next stage.
- **CN**: 这一段延续了 `safe_gmean`，继续展开其控制流、数据准备或生成结构。 条件分支用于保留特殊情况，并显式维护不变量。 本段中的赋值语句用于组织配置、导出符号或中间变换状态。 末尾会把组装好的结果或更新后的状态交给下一阶段。

## Key Concepts / 关键概念

- **Torch operator code generation**
  - EN: Builds derived code artifacts from declarative operator metadata.
  - CN: 从声明式算子元数据构建派生代码产物。
- **Learned heuristics**
  - EN: Collects data or trains rules that guide performance-sensitive decisions.
  - CN: 收集数据或训练规则，以指导性能敏感的决策。
- **Model training**
  - EN: Runs data-driven training loops to fit heuristic or regression models.
  - CN: 运行数据驱动的训练循环，以拟合启发式或回归模型。
- **DEBUG**
  - EN: `DEBUG` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `DEBUG` 是本文件声明、导出或驱动的显著符号之一。
- **AHTrainDecisionTree**
  - EN: `AHTrainDecisionTree` is one of the most visible symbols declared, exported, or driven by this file.
  - CN: `AHTrainDecisionTree` 是本文件声明、导出或驱动的显著符号之一。
## Dependencies / 依赖关系

- **Internal modules / 内部模块**: `torch._inductor.autoheuristic.autoheuristic_utils`, `torch._inductor.autoheuristic.learnedheuristic_interface`
- **Third-party modules / 第三方模块**: `numpy`, `pandas`, `ah_tree`, `scipy.stats`, `sklearn.model_selection`, `sklearn.tree`, `train`
- **Standard library / 标准库**: `itertools`, `json`, `logging`, `math`, `warnings`, `dataclasses`, `typing`
- **Primary symbols / 核心符号**: `DEBUG`, `AHTrainDecisionTree`, `__init__`, `debug_time`, `is_unsafe_leaf`, `get_unsafe_leaves`, `get_allowed_wrong_prediction_pct`, `get_grid_search_values`, `predict`, `ranking_num_choices`
- **Note / 说明**: 16 imports were detected; only the first few are listed for readability. / 检测到 16 个导入项，为便于阅读这里只展示前若干项。
