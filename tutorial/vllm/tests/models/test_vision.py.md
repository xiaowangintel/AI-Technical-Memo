# test_vision.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `tests/models/test_vision.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Covers vision or image inputs and model-facing behavior. The file defines 7 test(s), 0 fixture(s), and 10 helper/class block(s) to validate this area. / [CN] 该文件覆盖视觉或图像输入与面向模型的行为。它定义了 7 个测试、0 个 fixture，以及 10 个辅助函数/类块，用于验证这一领域。

## Line-by-Line Analysis / 逐行分析
### Imports and setup / 导入与初始化: module imports (L3-L24)
```python
import math

import pytest
import torch
import torch.multiprocessing as mp

from tests.utils import ensure_current_vllm_config, multi_gpu_test
from vllm.distributed import get_tensor_model_parallel_world_size
from vllm.distributed.parallel_state import (
    init_distributed_environment,
    initialize_model_parallel,
)
from vllm.model_executor.models.vision import (
    get_load_balance_assignment,
    resolve_visual_encoder_outputs,
    run_dp_sharded_mrope_vision_model,
    run_dp_sharded_vision_model,
)
from vllm.platforms import current_platform
from vllm.utils.network_utils import get_open_port
from vllm.utils.system_utils import update_environment_variables
from vllm.utils.torch_utils import set_random_seed
```
**EN:** Imports standard-library modules such as `math`, third-party packages like `pytest`, `torch`, `torch.multiprocessing`, project helpers such as `tests.utils.ensure_current_vllm_config`, `tests.utils.multi_gpu_test`, `vllm.distributed.get_tensor_model_parallel_world_size`.
**CN:** 导入标准库模块（如 `math`）、第三方包（如 `pytest`、`torch`、`torch.multiprocessing`）、项目内辅助模块（如 `tests.utils.ensure_current_vllm_config`、`tests.utils.multi_gpu_test`、`vllm.distributed.get_tensor_model_parallel_world_size`）。

### Module setup / 模块级配置: pytestmark (L26-L26)
```python
pytestmark = pytest.mark.cpu_test
```
**EN:** This block defines shared constants, registries, or module-level configuration such as `pytestmark`. Later helpers and tests reuse these values to keep scenarios concise. It also applies module-wide pytest markers.
**CN:** 该代码块定义共享常量、注册表或模块级配置，例如 `pytestmark`。后续辅助函数和测试会复用这些值，以减少重复。 它还会应用模块级的 pytest 标记。

### Test / 测试: test_resolve_visual_encoder_outputs (L29-L53)
```python
@pytest.mark.parametrize(
    ("select_layers", "num_layers_loaded", "max_possible_layers", "expected_features"),
    [
        # All layers loaded
        ([1, 10], 10, 10, [1, 10]),
        ([-10, -1], 10, 10, [1, 10]),
        # Some layers not loaded
        ([1, 10], 10, 20, [1, 10]),
        ([-20, -11], 10, 20, [1, 10]),
    ],
)
def test_resolve_visual_encoder_outputs(
    select_layers, num_layers_loaded, max_possible_layers, expected_features
):
    """
    Test that offsets are correctly handled for vision feature layers.
    """
    encoder_outputs = [torch.tensor([idx]) for idx in range(num_layers_loaded + 1)]
    output_tensor = resolve_visual_encoder_outputs(
        encoder_outputs=encoder_outputs,
        post_layer_norm=None,
        select_layers=select_layers,
        max_possible_layers=max_possible_layers,
    )
    assert torch.equal(torch.tensor(expected_features), output_tensor)
```
**EN:** This test validates `test_resolve_visual_encoder_outputs`. It uses parameterization over `select_layers`, `num_layers_loaded`, `max_possible_layers`, `expected_features`. Key inputs are `select_layers`, `num_layers_loaded`, `max_possible_layers`, `expected_features`. The main assertion is `torch.equal(torch.tensor(expected_features), output_tensor)`.
**CN:** 这个测试验证 `test_resolve_visual_encoder_outputs`。 它通过参数化组合 `select_layers`、`num_layers_loaded`、`max_possible_layers`、`expected_features`。 关键输入包括 `select_layers`、`num_layers_loaded`、`max_possible_layers`、`expected_features`。 核心断言是 `torch.equal(torch.tensor(expected_features), output_tensor)`。

### Class / 类: SimpleLinearModel (L56-L67)
```python
class SimpleLinearModel(torch.nn.Module):
    """A simple linear vision model for testing."""

    def __init__(self, input_dim: int = 3 * 224 * 224, output_dim: int = 32):
        super().__init__()
        self.flatten = torch.nn.Flatten()
        self.linear = torch.nn.Linear(input_dim, output_dim)

    def forward(self, x: torch.Tensor):
        # Flatten the input and apply linear transformation
        x = self.flatten(x)
        return self.linear(x)
```
**EN:** This class groups related scenarios in `SimpleLinearModel`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `forward`.
**CN:** 该类将与 `SimpleLinearModel` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`forward`。

### Helper method / 辅助方法: SimpleLinearModel.__init__ (L59-L62)
```python
    def __init__(self, input_dim: int = 3 * 224 * 224, output_dim: int = 32):
        super().__init__()
        self.flatten = torch.nn.Flatten()
        self.linear = torch.nn.Linear(input_dim, output_dim)
```
**EN:** This helper encapsulates reusable logic in `SimpleLinearModel.__init__`. Key inputs are `input_dim`, `output_dim`.
**CN:** 这个辅助函数将可复用逻辑封装在 `SimpleLinearModel.__init__` 中。 关键输入包括 `input_dim`、`output_dim`。

### Helper method / 辅助方法: SimpleLinearModel.forward (L64-L67)
```python
    def forward(self, x: torch.Tensor):
        # Flatten the input and apply linear transformation
        x = self.flatten(x)
        return self.linear(x)
```
**EN:** This helper encapsulates reusable logic in `SimpleLinearModel.forward`. Key inputs are `x`. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `SimpleLinearModel.forward` 中。 关键输入包括 `x`。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_run_dp_sharded_vision_model (L70-L90)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "batch_size",
    [
        1,  # Single image
        4,  # Small batch
        5,  # Odd batch size (for testing padding)
    ],
)
def test_run_dp_sharded_vision_model(batch_size: int):
    world_size = 2
    # Launch processes
    mp.spawn(
        run_dp_sharded_vision_model_vs_direct,
        args=(
            world_size,
            batch_size,
            get_open_port(),
        ),
        nprocs=world_size,
    )
```
**EN:** This test validates `test_run_dp_sharded_vision_model`. It uses parameterization over `batch_size`. Key inputs are `batch_size`.
**CN:** 这个测试验证 `test_run_dp_sharded_vision_model`。 它通过参数化组合 `batch_size`。 关键输入包括 `batch_size`。

### Helper / 辅助函数: run_dp_sharded_vision_model_vs_direct (L93-L144)
```python
def run_dp_sharded_vision_model_vs_direct(
    local_rank: int, world_size: int, batch_size: int, master_port: int
):
    """
    Test that run_dp_sharded_vision_model produces the same results as
    calling the model directly.
    """

    # Set random seed for reproducibility
    set_random_seed(0)

    device = f"{current_platform.device_name}:{local_rank}"
    torch.accelerator.set_device_index(device)
    torch.set_default_device(device)

    update_environment_variables(
        {
            "RANK": str(local_rank),
# ... 26 lines omitted for brevity ...
    # Check that the world size is set up correctly
    assert get_tensor_model_parallel_world_size() == world_size

    # Check that the outputs have the same shape
    assert direct_output.shape == sharded_output.shape

    # Check that the outputs are close (they should be identical)
    assert torch.allclose(direct_output, sharded_output, rtol=1e-5, atol=1e-5)
```
**EN:** This helper encapsulates reusable logic in `run_dp_sharded_vision_model_vs_direct`. Key inputs are `local_rank`, `world_size`, `batch_size`, `master_port`. The main assertion is `get_tensor_model_parallel_world_size() == world_size` and `direct_output.shape == sharded_output.shape`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_dp_sharded_vision_model_vs_direct` 中。 关键输入包括 `local_rank`、`world_size`、`batch_size`、`master_port`。 核心断言是 `get_tensor_model_parallel_world_size() == world_size` and `direct_output.shape == sharded_output.shape`。

### Test / 测试: test_get_load_balance_assignment_cases (L147-L205)
```python
@pytest.mark.parametrize(
    "sizes,num_gpus,expected_shuffle_indices,expected_gpu_sample_counts,"
    "expected_grouped_sizes_per_gpu,test_description",
    [
        # Empty input
        ([], 2, [], [0, 0], [0, 0], "empty input"),
        # Fewer samples than GPUs
        (
            [100, 200],
            4,
            [1, 0],
            [1, 1, 0, 0],
            [200, 100, 0, 0],
            "fewer samples than GPUs",
        ),
        # Single GPU
        ([100, 200, 300], 1, [2, 1, 0], [3], [600], "single GPU"),
        # Balanced assignment
# ... 33 lines omitted for brevity ...
    assert len(gpu_sample_counts) == num_gpus
    assert len(grouped_sizes_per_gpu) == num_gpus
    assert sum(gpu_sample_counts) == len(sizes)

    assert shuffle_indices == expected_shuffle_indices

    assert gpu_sample_counts == expected_gpu_sample_counts
    assert grouped_sizes_per_gpu == expected_grouped_sizes_per_gpu
```
**EN:** This test validates `test_get_load_balance_assignment_cases`. It uses parameterization over `sizes`, `num_gpus`, `expected_shuffle_indices`, `expected_gpu_sample_counts`, `expected_grouped_sizes_per_gpu`, `test_description`. Key inputs are `sizes`, `num_gpus`, `expected_shuffle_indices`, `expected_gpu_sample_counts`, `expected_grouped_sizes_per_gpu`, `test_description`. The main assertion is `len(shuffle_indices) == len(sizes)` and `len(gpu_sample_counts) == num_gpus`.
**CN:** 这个测试验证 `test_get_load_balance_assignment_cases`。 它通过参数化组合 `sizes`、`num_gpus`、`expected_shuffle_indices`、`expected_gpu_sample_counts`、`expected_grouped_sizes_per_gpu`、`test_description`。 关键输入包括 `sizes`、`num_gpus`、`expected_shuffle_indices`、`expected_gpu_sample_counts`、`expected_grouped_sizes_per_gpu`、`test_description`。 核心断言是 `len(shuffle_indices) == len(sizes)` and `len(gpu_sample_counts) == num_gpus`。

### Class / 类: SimpleMRopeVisionModel (L208-L255)
```python
class SimpleMRopeVisionModel(torch.nn.Module):
    """A simple vision model for testing mrope functionality."""

    def __init__(self, spatial_merge_size: int = 2, out_hidden_size: int = 64):
        super().__init__()
        self.spatial_merge_size = spatial_merge_size
        self.out_hidden_size = out_hidden_size
        self.linear = torch.nn.Linear(768, out_hidden_size)

    def forward(self, pixel_values: torch.Tensor, grid_thw_list: list[list[int]]):
        """Simple forward pass that simulates spatial merging."""
        # Apply linear transformation
        embeddings = self.linear(pixel_values)

# ... 26 lines omitted for brevity ...
        if merged_embeddings:
            return torch.cat(merged_embeddings, dim=0)
        else:
            return torch.empty(
                (0, self.out_hidden_size),
                device=pixel_values.device,
                dtype=pixel_values.dtype,
            )
```
**EN:** This class groups related scenarios in `SimpleMRopeVisionModel`. It contains 0 test method(s) and 2 supporting method(s). Representative methods include `__init__`, `forward`.
**CN:** 该类将与 `SimpleMRopeVisionModel` 相关的场景组织在一起。 它包含 0 个测试方法和 2 个辅助方法。 代表性方法包括 `__init__`、`forward`。

### Helper method / 辅助方法: SimpleMRopeVisionModel.__init__ (L211-L215)
```python
    def __init__(self, spatial_merge_size: int = 2, out_hidden_size: int = 64):
        super().__init__()
        self.spatial_merge_size = spatial_merge_size
        self.out_hidden_size = out_hidden_size
        self.linear = torch.nn.Linear(768, out_hidden_size)
```
**EN:** This helper encapsulates reusable logic in `SimpleMRopeVisionModel.__init__`. Key inputs are `spatial_merge_size`, `out_hidden_size`.
**CN:** 这个辅助函数将可复用逻辑封装在 `SimpleMRopeVisionModel.__init__` 中。 关键输入包括 `spatial_merge_size`、`out_hidden_size`。

### Helper method / 辅助方法: SimpleMRopeVisionModel.forward (L217-L255)
```python
    def forward(self, pixel_values: torch.Tensor, grid_thw_list: list[list[int]]):
        """Simple forward pass that simulates spatial merging."""
        # Apply linear transformation
        embeddings = self.linear(pixel_values)

        # Simulate spatial merging by reducing the number of patches
        merge_factor = self.spatial_merge_size * self.spatial_merge_size

        # Group patches and merge spatially
        merged_embeddings = []
        start_idx = 0

        for grid_thw in grid_thw_list:
            num_patches = math.prod(grid_thw)
            end_idx = start_idx + num_patches

            # Get patches for this image
            image_patches = embeddings[start_idx:end_idx]
# ... 13 lines omitted for brevity ...
        if merged_embeddings:
            return torch.cat(merged_embeddings, dim=0)
        else:
            return torch.empty(
                (0, self.out_hidden_size),
                device=pixel_values.device,
                dtype=pixel_values.dtype,
            )
```
**EN:** This helper encapsulates reusable logic in `SimpleMRopeVisionModel.forward`. Key inputs are `pixel_values`, `grid_thw_list`. Patching or monkeypatching is used to isolate external state and heavyweight dependencies. It returns computed state or helper objects back to the caller.
**CN:** 这个辅助函数将可复用逻辑封装在 `SimpleMRopeVisionModel.forward` 中。 关键输入包括 `pixel_values`、`grid_thw_list`。 这里使用 patch 或 monkeypatch 来隔离外部状态和重量级依赖。 它把计算得到的状态或辅助对象返回给调用方。

### Test / 测试: test_run_dp_sharded_mrope_vision_model (L258-L278)
```python
@multi_gpu_test(num_gpus=2)
@pytest.mark.parametrize(
    "batch_size",
    [
        1,  # Single image
        3,  # Small batch
        5,  # Odd batch size (for testing padding)
    ],
)
def test_run_dp_sharded_mrope_vision_model(batch_size: int):
    world_size = 2
    # Launch processes
    mp.spawn(
        run_dp_sharded_mrope_vision_model_vs_direct,
        args=(
            world_size,
            batch_size,
            get_open_port(),
        ),
        nprocs=world_size,
    )
```
**EN:** This test validates `test_run_dp_sharded_mrope_vision_model`. It uses parameterization over `batch_size`. Key inputs are `batch_size`.
**CN:** 这个测试验证 `test_run_dp_sharded_mrope_vision_model`。 它通过参数化组合 `batch_size`。 关键输入包括 `batch_size`。

### Helper / 辅助函数: run_dp_sharded_mrope_vision_model_vs_direct (L281-L349)
```python
def run_dp_sharded_mrope_vision_model_vs_direct(
    local_rank: int, world_size: int, batch_size: int, master_port: int
):
    """
    Test that run_dp_sharded_mrope_vision_model produces the same results as
    calling the model directly.
    """
    # Set random seed for reproducibility
    set_random_seed(0)
    device = f"{current_platform.device_name}:{local_rank}"
    torch.accelerator.set_device_index(device)
    torch.set_default_device(device)

    update_environment_variables(
        {
            "RANK": str(local_rank),
            "LOCAL_RANK": str(local_rank),
            "WORLD_SIZE": str(world_size),
# ... 43 lines omitted for brevity ...
    assert get_tensor_model_parallel_world_size() == world_size

    # Compare outputs (only on rank 0)
    if local_rank == 0:
        # Check that the outputs have the same shape
        assert direct_output.shape == sharded_output.shape
        # Check that the outputs are close (they should be identical)
        assert torch.allclose(direct_output, sharded_output, rtol=1e-5, atol=1e-5)
```
**EN:** This helper encapsulates reusable logic in `run_dp_sharded_mrope_vision_model_vs_direct`. Key inputs are `local_rank`, `world_size`, `batch_size`, `master_port`. The main assertion is `get_tensor_model_parallel_world_size() == world_size` and `direct_output.shape == sharded_output.shape`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_dp_sharded_mrope_vision_model_vs_direct` 中。 关键输入包括 `local_rank`、`world_size`、`batch_size`、`master_port`。 核心断言是 `get_tensor_model_parallel_world_size() == world_size` and `direct_output.shape == sharded_output.shape`。

### Test / 测试: test_run_dp_sharded_mrope_vision_model_empty_input (L352-L359)
```python
@multi_gpu_test(num_gpus=2)
def test_run_dp_sharded_mrope_vision_model_empty_input():
    world_size = 2
    mp.spawn(
        run_dp_sharded_mrope_vision_model_empty_input_worker,
        args=(world_size, get_open_port()),
        nprocs=world_size,
    )
```
**EN:** This test validates `test_run_dp_sharded_mrope_vision_model_empty_input`.
**CN:** 这个测试验证 `test_run_dp_sharded_mrope_vision_model_empty_input`。

### Helper / 辅助函数: run_dp_sharded_mrope_vision_model_empty_input_worker (L362-L397)
```python
def run_dp_sharded_mrope_vision_model_empty_input_worker(
    local_rank: int, world_size: int, master_port: int
):
    """Test run_dp_sharded_mrope_vision_model with empty input."""
    # Set up distributed environment
    device = f"{current_platform.device_name}:{local_rank}"
    torch.accelerator.set_device_index(device)
    torch.set_default_device(device)

    update_environment_variables(
        {
            "RANK": str(local_rank),
            "LOCAL_RANK": str(local_rank),
            "WORLD_SIZE": str(world_size),
            "MASTER_ADDR": "localhost",
            "MASTER_PORT": str(master_port),
        }
    )
# ... 10 lines omitted for brevity ...

    # Should handle empty input gracefully
    with torch.inference_mode():
        output = run_dp_sharded_mrope_vision_model(
            vision_model, pixel_values, grid_thw_list, rope_type="rope_3d"
        )

    assert len(output) == 0
```
**EN:** This helper encapsulates reusable logic in `run_dp_sharded_mrope_vision_model_empty_input_worker`. Key inputs are `local_rank`, `world_size`, `master_port`. The main assertion is `len(output) == 0`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_dp_sharded_mrope_vision_model_empty_input_worker` 中。 关键输入包括 `local_rank`、`world_size`、`master_port`。 核心断言是 `len(output) == 0`。

### Test / 测试: test_run_dp_sharded_mrope_vision_model_uneven_load (L400-L407)
```python
@multi_gpu_test(num_gpus=4)
def test_run_dp_sharded_mrope_vision_model_uneven_load():
    world_size = 4
    mp.spawn(
        run_dp_sharded_mrope_vision_model_uneven_load_worker,
        args=(world_size, get_open_port()),
        nprocs=world_size,
    )
```
**EN:** This test validates `test_run_dp_sharded_mrope_vision_model_uneven_load`.
**CN:** 这个测试验证 `test_run_dp_sharded_mrope_vision_model_uneven_load`。

### Helper / 辅助函数: run_dp_sharded_mrope_vision_model_uneven_load_worker (L410-L464)
```python
def run_dp_sharded_mrope_vision_model_uneven_load_worker(
    local_rank: int, world_size: int, master_port: int
):
    """Test run_dp_sharded_mrope_vision_model with uneven load distribution."""
    # Set up distributed environment
    set_random_seed(123)
    device = f"{current_platform.device_name}:{local_rank}"
    torch.accelerator.set_device_index(device)
    torch.set_default_device(device)

    update_environment_variables(
        {
            "RANK": str(local_rank),
            "LOCAL_RANK": str(local_rank),
            "WORLD_SIZE": str(world_size),
            "MASTER_ADDR": "localhost",
            "MASTER_PORT": str(master_port),
        }
# ... 29 lines omitted for brevity ...
    merge_factor = vision_model.spatial_merge_size**2
    expected_output_patches = list(
        math.prod(grid_thw) // merge_factor for grid_thw in grid_thw_list
    )

    for i, output in enumerate(output_tuple):
        assert output.shape[0] == expected_output_patches[i]
        assert output.shape[1] == vision_model.out_hidden_size
```
**EN:** This helper encapsulates reusable logic in `run_dp_sharded_mrope_vision_model_uneven_load_worker`. Key inputs are `local_rank`, `world_size`, `master_port`. The main assertion is `output.shape[0] == expected_output_patches[i]` and `output.shape[1] == vision_model.out_hidden_size`.
**CN:** 这个辅助函数将可复用逻辑封装在 `run_dp_sharded_mrope_vision_model_uneven_load_worker` 中。 关键输入包括 `local_rank`、`world_size`、`master_port`。 核心断言是 `output.shape[0] == expected_output_patches[i]` and `output.shape[1] == vision_model.out_hidden_size`。

### Test / 测试: test_simple_mrope_vision_model_spatial_merge (L467-L494)
```python
@pytest.mark.parametrize("spatial_merge_size", [2, 4])
def test_simple_mrope_vision_model_spatial_merge(spatial_merge_size: int):
    """Test SimpleMRopeVisionModel with different spatial merge sizes."""
    device = current_platform.device_type

    grid_thw_list = [[1, 4, 4], [1, 6, 6]]  # Two images
    pixel_values_list = []

    for grid_thw in grid_thw_list:
        num_patches = math.prod(grid_thw)
        image_pixels = torch.randn(num_patches, 768, device=device)
        pixel_values_list.append(image_pixels)

    pixel_values = torch.cat(pixel_values_list, dim=0)
    vision_model = SimpleMRopeVisionModel(spatial_merge_size=spatial_merge_size).to(
        device
    )

    with torch.inference_mode():
        output = vision_model(pixel_values, grid_thw_list)

    # Verify output dimensions based on spatial merging
    total_patches = sum(math.prod(grid_thw) for grid_thw in grid_thw_list)
    merge_factor = spatial_merge_size**2
    expected_output_patches = total_patches // merge_factor

    assert output.shape[0] == expected_output_patches
    assert output.shape[1] == vision_model.out_hidden_size
```
**EN:** This test validates `test_simple_mrope_vision_model_spatial_merge`. It uses parameterization over `spatial_merge_size`. Key inputs are `spatial_merge_size`. The main assertion is `output.shape[0] == expected_output_patches` and `output.shape[1] == vision_model.out_hidden_size`.
**CN:** 这个测试验证 `test_simple_mrope_vision_model_spatial_merge`。 它通过参数化组合 `spatial_merge_size`。 关键输入包括 `spatial_merge_size`。 核心断言是 `output.shape[0] == expected_output_patches` and `output.shape[1] == vision_model.out_hidden_size`。

## Key Concepts / 关键概念
- **EN:** Parametrization expands one definition into a scenario matrix.
  **CN:** 参数化把一个测试定义扩展为场景矩阵。
- **EN:** Patching/mocking isolates heavy dependencies and environment-sensitive branches.
  **CN:** Patching/mocking 用于隔离重量级依赖和环境敏感分支。
- **EN:** Multimodal inputs are validated through image, audio, or mixed-modal paths.
  **CN:** 通过图像、音频或混合模态路径验证多模态输入。
- **EN:** The checks cover model loading, execution, registry data, or backend compatibility.
  **CN:** 这些检查覆盖模型加载、执行、注册表数据或后端兼容性。
- **EN:** Assertions encode the acceptance criteria for each scenario.
  **CN:** 断言定义了每个场景的验收标准。

## Dependencies / 依赖关系
- **Stdlib / 标准库**: `math`
- **Third-party / 第三方**: `pytest`, `torch`, `torch.multiprocessing`
- **Project / 项目内**: `tests.utils.ensure_current_vllm_config`, `tests.utils.multi_gpu_test`, `vllm.distributed.get_tensor_model_parallel_world_size`, `vllm.distributed.parallel_state.init_distributed_environment`, `vllm.distributed.parallel_state.initialize_model_parallel`, `vllm.model_executor.models.vision.get_load_balance_assignment`, `vllm.model_executor.models.vision.resolve_visual_encoder_outputs`, `vllm.model_executor.models.vision.run_dp_sharded_mrope_vision_model`, `vllm.model_executor.models.vision.run_dp_sharded_vision_model`, `vllm.platforms.current_platform`, `vllm.utils.network_utils.get_open_port`, `vllm.utils.system_utils.update_environment_variables`, `vllm.utils.torch_utils.set_random_seed`
