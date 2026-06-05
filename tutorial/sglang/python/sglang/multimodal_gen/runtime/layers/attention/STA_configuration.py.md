# STA_configuration.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/runtime/layers/attention/STA_configuration.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the runtime layer implementation. It centers on `configure_sta`, `read_specific_json_files`, and `average_head_losses`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于运行时算子层。它围绕 `configure_sta`、`read_specific_json_files` 和 `average_head_losses` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-11: module setup and imports / 模块初始化与导入
```python
import json
import os
from collections import defaultdict
from typing import Any

import numpy as np

from sglang.multimodal_gen.utils import dict_to_3d_list
```
**EN:** This block establishes the module context and imports `json`, `os`, `collections`, `typing`, `numpy`, and `sglang.multimodal_gen.utils`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `json`、`os`、`collections`、`typing`、`numpy` 和 `sglang.multimodal_gen.utils`。这些依赖为后续实现提供所需符号。

### Lines 14-250: `configure_sta` implementation / `configure_sta` 实现
```python
def configure_sta(
    mode: str = "STA_searching",
    layer_num: int = 40,
    time_step_num: int = 50,
    head_num: int = 40,
    **kwargs,
) -> list[list[list[Any]]]:
    """
    Configure Sliding Tile Attention (STA) parameters based on the specified mode.

    Parameters:
    ----------
    mode : str
        The STA mode to use. Options are:
        - 'STA_searching': Generate a set of mask candidates for initial search
        - 'STA_tuning': Select best mask strategy based on previously saved results
        - 'STA_inference': Load and use a previously tuned mask strategy
    layer_num: int, number of layers
    time_step_num: int, number of timesteps
    head_num: int, number of heads

    **kwargs : dict
        Mode-specific parameters:

        For 'STA_searching':
        - mask_candidates: list of str, optional, mask candidates to use
        - mask_selected: list of int, optional, indices of selected masks

        For 'STA_tuning':
        - mask_search_files_path: str, required, path to mask search results
        - mask_candidates: list of str, optional, mask candidates to use
        - mask_selected: list of int, optional, indices of selected masks
        - skip_time_steps: int, optional, number of time steps to use full attention (default 12)
        - save_dir: str, optional, directory to save mask strategy (default "mask_candidates")

        For 'STA_inference':
        - load_path: str, optional, path to load mask strategy (default "mask_candidates/mask_strategy.json")
    """
    valid_modes = ["STA_searching", "STA_tuning", "STA_inference", "STA_tuning_cfg"]
    if mode not in valid_modes:
        raise ValueError(f"Mode must be one of {valid_modes}, got {mode}")

    if mode == "STA_searching":
        # Get parameters with defaults
        mask_candidates: list[str] | None = kwargs.get("mask_candidates")
        if mask_candidates is None:
            raise ValueError("mask_candidates is required for STA_searching mode")
        mask_selected: list[int] = kwargs.get(
            "mask_selected", list(range(len(mask_candidates)))
        )

        # Parse selected masks
        selected_masks: list[list[int]] = []
        for index in mask_selected:
            mask = mask_candidates[index]
            masks_list = [int(x) for x in mask.split(",")]
            selected_masks.append(masks_list)

        # Create 3D mask structure with fixed dimensions (t=50, l=60)
        masks_3d: list[list[list[list[int]]]] = []
        for i in range(time_step_num):  # Fixed t dimension = 50
            row = []
            for j in range(layer_num):  # Fixed l dimension = 60
                row.append(selected_masks)  # Add all masks at each position
            masks_3d.append(row)

        return masks_3d

    elif mode == "STA_tuning":
        # Get required parameters
        mask_search_files_path: str | None = kwargs.get("mask_search_files_path")
        if not mask_search_files_path:
            raise ValueError("mask_search_files_path is required for STA_tuning mode")

        # Get optional parameters with defaults
        mask_candidates_tuning: list[str] | None = kwargs.get("mask_candidates")
        if mask_candidates_tuning is None:
            raise ValueError("mask_candidates is required for STA_tuning mode")
        mask_selected_tuning: list[int] = kwargs.get(
            "mask_selected", list(range(len(mask_candidates_tuning)))
        )
        skip_time_steps_tuning: int | None = kwargs.get("skip_time_steps")
        save_dir_tuning: str | None = kwargs.get("save_dir", "mask_candidates")

        # Parse selected masks
        selected_masks_tuning: list[list[int]] = []
        for index in mask_selected_tuning:
            mask = mask_candidates_tuning[index]
            masks_list = [int(x) for x in mask.split(",")]
            selected_masks_tuning.append(masks_list)

        # Read JSON results
        results = read_specific_json_files(mask_search_files_path)
        averaged_results = average_head_losses(results, selected_masks_tuning)

        # Add full attention mask for specific cases
        full_attention_mask_tuning: list[int] | None = kwargs.get("full_attention_mask")
        if full_attention_mask_tuning is not None:
            selected_masks_tuning.append(full_attention_mask_tuning)

        # Select best mask strategy
        timesteps_tuning: int = kwargs.get("timesteps", time_step_num)
        if skip_time_steps_tuning is None:
            skip_time_steps_tuning = 12
        mask_strategy, sparsity, strategy_counts = select_best_mask_strategy(
            averaged_results,
            selected_masks_tuning,
            skip_time_steps_tuning,
            timesteps_tuning,
            head_num,
        )

        # Save mask strategy
        if save_dir_tuning is not None:
            os.makedirs(save_dir_tuning, exist_ok=True)
            file_path = os.path.join(
                save_dir_tuning, f"mask_strategy_s{skip_time_steps_tuning}.json"
            )
            with open(file_path, "w") as f:
                json.dump(mask_strategy, f, indent=4)
            print(f"Successfully saved mask_strategy to {file_path}")

        # Print sparsity and strategy counts for information
        print(f"Overall sparsity: {sparsity:.4f}")
        print("\nStrategy usage counts:")
        total_heads = time_step_num * layer_num * head_num  # Fixed dimensions
        for strategy, count in strategy_counts.items():
            print(f"Strategy {strategy}: {count} heads ({count/total_heads*100:.2f}%)")

        # Convert dictionary to 3D list with fixed dimensions
        mask_strategy_3d = dict_to_3d_list(
            mask_strategy, t_max=time_step_num, l_max=layer_num, h_max=head_num
        )

        return mask_strategy_3d
    elif mode == "STA_tuning_cfg":
        # Get required parameters for both positive and negative paths
        mask_search_files_path_pos: str | None = kwargs.get(
            "mask_search_files_path_pos"
        )
        mask_search_files_path_neg: str | None = kwargs.get(
            "mask_search_files_path_neg"
        )
        save_dir_cfg: str | None = kwargs.get("save_dir")

        if (
            not mask_search_files_path_pos
            or not mask_search_files_path_neg
            or not save_dir_cfg
        ):
            raise ValueError(
                "mask_search_files_path_pos, mask_search_files_path_neg, and save_dir are required for STA_tuning_cfg mode"
            )

        # Get optional parameters with defaults
        mask_candidates_cfg: list[str] | None = kwargs.get("mask_candidates")
        if mask_candidates_cfg is None:
            raise ValueError("mask_candidates is required for STA_tuning_cfg mode")
        mask_selected_cfg: list[int] = kwargs.get(
            "mask_selected", list(range(len(mask_candidates_cfg)))
        )
        skip_time_steps_cfg: int | None = kwargs.get("skip_time_steps")

        # Parse selected masks
        selected_masks_cfg: list[list[int]] = []
        for index in mask_selected_cfg:
            mask = mask_candidates_cfg[index]
            masks_list = [int(x) for x in mask.split(",")]
            selected_masks_cfg.append(masks_list)

        # Read JSON results for both positive and negative paths
        pos_results = read_specific_json_files(mask_search_files_path_pos)
        neg_results = read_specific_json_files(mask_search_files_path_neg)
        # Combine positive and negative results into one list
        combined_results = pos_results + neg_results

        # Average the combined results
        averaged_results = average_head_losses(combined_results, selected_masks_cfg)

        # Add full attention mask for specific cases
        full_attention_mask_cfg: list[int] | None = kwargs.get("full_attention_mask")
        if full_attention_mask_cfg is not None:
            selected_masks_cfg.append(full_attention_mask_cfg)

        timesteps_cfg: int = kwargs.get("timesteps", time_step_num)
        if skip_time_steps_cfg is None:
            skip_time_steps_cfg = 12
        # Select best mask strategy using combined results
        mask_strategy, sparsity, strategy_counts = select_best_mask_strategy(
            averaged_results,
            selected_masks_cfg,
            skip_time_steps_cfg,
            timesteps_cfg,
            head_num,
        )

        # Save mask strategy
        os.makedirs(save_dir_cfg, exist_ok=True)
        file_path = os.path.join(
            save_dir_cfg, f"mask_strategy_s{skip_time_steps_cfg}.json"
        )
        with open(file_path, "w") as f:
            json.dump(mask_strategy, f, indent=4)
        print(f"Successfully saved mask_strategy to {file_path}")

        # Print sparsity and strategy counts for information
        print(f"Overall sparsity: {sparsity:.4f}")
        print("\nStrategy usage counts:")
        total_heads = time_step_num * layer_num * head_num  # Fixed dimensions
        for strategy, count in strategy_counts.items():
            print(f"Strategy {strategy}: {count} heads ({count/total_heads*100:.2f}%)")

        # Convert dictionary to 3D list with fixed dimensions
        mask_strategy_3d = dict_to_3d_list(
            mask_strategy, t_max=time_step_num, l_max=layer_num, h_max=head_num
        )

        return mask_strategy_3d

    else:  # STA_inference
        # Get parameters with defaults
        load_path: str | None = kwargs.get(
            "load_path", "mask_candidates/mask_strategy.json"
        )
        if load_path is None:
            raise ValueError("load_path is required for STA_inference mode")

        # Load previously saved mask strategy
        with open(load_path) as f:
            mask_strategy = json.load(f)

        # Convert dictionary to 3D list with fixed dimensions
        mask_strategy_3d = dict_to_3d_list(
            mask_strategy, t_max=time_step_num, l_max=layer_num, h_max=head_num
        )

        return mask_strategy_3d
```
**EN:** This block defines function `configure_sta`. Configure Sliding Tile Attention (STA) parameters based on the specified mode. Parameters: ---------- mode : str The STA mode to use. Key calls include `ValueError`, `kwargs.get`, `range`, `list`, and `selected_masks.append`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `mode`, `layer_num`, `time_step_num`, and `head_num` drive the behavior in this section.
**CN:** 该代码块定义了函数 `configure_sta`。 它用于处理 configure sta 相关逻辑。 关键调用包括 `ValueError`、`kwargs.get`、`range`、`list` 和 `selected_masks.append`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `mode`、`layer_num`、`time_step_num` 和 `head_num` 等参数驱动。

### Lines 256-272: `read_specific_json_files` implementation / `read_specific_json_files` 实现
```python
def read_specific_json_files(folder_path: str) -> list[dict[str, Any]]:
    """Read and parse JSON files containing mask search results."""
    json_contents: list[dict[str, Any]] = []

    # List files only in the current directory (no walk)
    files = os.listdir(folder_path)
    # Filter files
    matching_files = [f for f in files if "mask" in f and f.endswith(".json")]
    print(f"Found {len(matching_files)} matching files: {matching_files}")

    for file_name in matching_files:
        file_path = os.path.join(folder_path, file_name)
        with open(file_path) as file:
            data = json.load(file)
            json_contents.append(data)

    return json_contents
```
**EN:** This block defines function `read_specific_json_files`. Read and parse JSON files containing mask search results. Key calls include `os.listdir`, `print`, `os.path.join`, `open`, and `json.load`. The implementation iterates over collections or steps, uses context-managed resources. Parameters such as `folder_path` drive the behavior in this section.
**CN:** 该代码块定义了函数 `read_specific_json_files`。 它用于处理 read specific json files 相关逻辑。 关键调用包括 `os.listdir`、`print`、`os.path.join`、`open` 和 `json.load`。 实现中会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `folder_path` 等参数驱动。

### Lines 275-298: `average_head_losses` implementation / `average_head_losses` 实现
```python
def average_head_losses(
    results: list[dict[str, Any]], selected_masks: list[list[int]]
) -> dict[str, dict[str, np.ndarray]]:
    """Average losses across all prompts for each mask strategy."""
    # Initialize a dictionary to store the averaged results
    averaged_losses: dict[str, dict[str, np.ndarray]] = {}
    loss_type = "L2_loss"
    # Get all loss types (e.g., 'L2_loss')
    averaged_losses[loss_type] = {}

    for mask in selected_masks:
        mask_str = str(mask)
        data_shape = np.array(results[0][loss_type][mask_str]).shape
        accumulated_data = np.zeros(data_shape)

        # Sum across all prompts
        for prompt_result in results:
            accumulated_data += np.array(prompt_result[loss_type][mask_str])

        # Average by dividing by number of prompts
        averaged_data = accumulated_data / len(results)
        averaged_losses[loss_type][mask_str] = averaged_data

    return averaged_losses
```
**EN:** This block defines function `average_head_losses`. Average losses across all prompts for each mask strategy. Key calls include `str`, `np.zeros`, `np.array`, and `len`. The implementation iterates over collections or steps. Parameters such as `results`, and `selected_masks` drive the behavior in this section.
**CN:** 该代码块定义了函数 `average_head_losses`。 它用于处理 average head losses 相关逻辑。 关键调用包括 `str`、`np.zeros`、`np.array` 和 `len`。 实现中会遍历集合或步骤。 本段逻辑主要由 `results` 和 `selected_masks` 等参数驱动。

### Lines 301-357: `select_best_mask_strategy` implementation / `select_best_mask_strategy` 实现
```python
def select_best_mask_strategy(
    averaged_results: dict[str, dict[str, np.ndarray]],
    selected_masks: list[list[int]],
    skip_time_steps: int = 12,
    timesteps: int = 50,
    head_num: int = 40,
) -> tuple[dict[str, list[int]], float, dict[str, int]]:
    """Select the best mask strategy for each head based on loss minimization."""
    best_mask_strategy: dict[str, list[int]] = {}
    loss_type = "L2_loss"
    # Get the shape of time steps and layers
    layers = len(averaged_results[loss_type][str(selected_masks[0])][0])

    # Counter for sparsity calculation
    total_tokens = 0  # total number of masked tokens
    total_length = 0  # total sequence length

    strategy_counts: dict[str, int] = {str(strategy): 0 for strategy in selected_masks}
    full_attn_strategy = selected_masks[-1]  # Last strategy is full attention
    print(f"Strategy {full_attn_strategy}, skip first {skip_time_steps} steps ")

    for t in range(timesteps):
        for layer_idx in range(layers):
            for h in range(head_num):
                if t < skip_time_steps:  # First steps use full attention
                    strategy = full_attn_strategy
                else:
                    # Get losses for this head across all strategies
                    head_losses = []
                    for strategy in selected_masks[:-1]:  # Exclude full attention
                        head_losses.append(
                            averaged_results[loss_type][str(strategy)][t][layer_idx][h]
                        )

                    # Find which strategy gives minimum loss
                    best_strategy_idx = np.argmin(head_losses)
                    strategy = selected_masks[best_strategy_idx]

                best_mask_strategy[f"{t}_{layer_idx}_{h}"] = strategy

                # Calculate sparsity
                nums = strategy  # strategy is already a list of numbers
                total_tokens += (
                    nums[0] * nums[1] * nums[2]
                )  # masked tokens for chosen strategy
                total_length += (
                    full_attn_strategy[0]
                    * full_attn_strategy[1]
                    * full_attn_strategy[2]
                )

                # Count strategy usage
                strategy_counts[str(strategy)] += 1

    overall_sparsity = 1 - total_tokens / total_length

    return best_mask_strategy, overall_sparsity, strategy_counts
```
**EN:** This block defines function `select_best_mask_strategy`. Select the best mask strategy for each head based on loss minimization. Key calls include `len`, `print`, `range`, `str`, and `np.argmin`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `averaged_results`, `selected_masks`, `skip_time_steps`, `timesteps`, and `head_num` drive the behavior in this section.
**CN:** 该代码块定义了函数 `select_best_mask_strategy`。 它用于选择best mask strategy。 关键调用包括 `len`、`print`、`range`、`str` 和 `np.argmin`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `averaged_results`、`selected_masks`、`skip_time_steps`、`timesteps` 和 `head_num` 等参数驱动。

### Lines 360-414: `save_mask_search_results` implementation / `save_mask_search_results` 实现
```python
def save_mask_search_results(
    mask_search_final_result: list[dict[str, list[float]]],
    prompt: str,
    mask_strategies: list[str],
    output_dir: str = "output/mask_search_result/",
) -> str | None:
    if not mask_search_final_result:
        print("No mask search results to save")
        return None

    # Create result dictionary with defaultdict for nested lists
    mask_search_dict: dict[str, dict[str, list[list[float]]]] = {
        "L2_loss": defaultdict(list),
        "L1_loss": defaultdict(list),
    }

    mask_selected = list(range(len(mask_strategies)))
    selected_masks: list[list[int]] = []
    for index in mask_selected:
        mask = mask_strategies[index]
        masks_list = [int(x) for x in mask.split(",")]
        selected_masks.append(masks_list)

    # Process each mask strategy
    for i, mask_strategy in enumerate(selected_masks):
        mask_strategy_str = str(mask_strategy)
        # Process L2 loss
        step_results: list[list[float]] = []
        for step_data in mask_search_final_result:
            if isinstance(step_data, dict) and "L2_loss" in step_data:
                layer_losses = [float(loss) for loss in step_data["L2_loss"]]
                step_results.append(layer_losses)
        mask_search_dict["L2_loss"][mask_strategy_str] = step_results

        step_results = []
        for step_data in mask_search_final_result:
            if isinstance(step_data, dict) and "L1_loss" in step_data:
                layer_losses = [float(loss) for loss in step_data["L1_loss"]]
                step_results.append(layer_losses)
        mask_search_dict["L1_loss"][mask_strategy_str] = step_results

    # Create the output directory if it doesn't exist
    os.makedirs(output_dir, exist_ok=True)

    # Create a filename based on the first 20 characters of the prompt
    filename = prompt[:50].replace(" ", "_")
    filepath = os.path.join(output_dir, f"mask_search_{filename}.json")

    # Save the results to a JSON file
    with open(filepath, "w") as f:
        json.dump(mask_search_dict, f, indent=4)

    print(f"Successfully saved mask research results to {filepath}")

    return filepath
```
**EN:** This block defines function `save_mask_search_results`. It saves mask search results. Key calls include `list`, `enumerate`, `os.makedirs`, `prompt.replace`, and `os.path.join`. The implementation branches on conditions, iterates over collections or steps, uses context-managed resources. Parameters such as `mask_search_final_result`, `prompt`, `mask_strategies`, and `output_dir` drive the behavior in this section.
**CN:** 该代码块定义了函数 `save_mask_search_results`。 它用于保存mask search results。 关键调用包括 `list`、`enumerate`、`os.makedirs`、`prompt.replace` 和 `os.path.join`。 实现中包含条件分支，会遍历集合或步骤，使用上下文管理资源。 本段逻辑主要由 `mask_search_final_result`、`prompt`、`mask_strategies` 和 `output_dir` 等参数驱动。

## Key Concepts / 关键概念
- `configure_sta`: Configure Sliding Tile Attention (STA) parameters based on the specified mode. / 顶层函数，用于处理 configure sta 相关逻辑。
- `read_specific_json_files`: Read and parse JSON files containing mask search results. / 顶层函数，用于处理 read specific json files 相关逻辑。
- `average_head_losses`: Average losses across all prompts for each mask strategy. / 顶层函数，用于处理 average head losses 相关逻辑。
- `select_best_mask_strategy`: Select the best mask strategy for each head based on loss minimization. / 顶层函数，用于选择best mask strategy。
- `save_mask_search_results`: Top-level function that saves mask search results. / 顶层函数，用于保存mask search results。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `collections`, `typing`
- **Third-party / 第三方依赖**: `numpy`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.utils`

- **Total lines / 总行数**: 414
