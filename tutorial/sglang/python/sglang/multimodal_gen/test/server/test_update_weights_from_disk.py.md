# test_update_weights_from_disk.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/test/server/test_update_weights_from_disk.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates update weights from disk with focused assertions and fixtures. Key symbols include `_resolve_active_model_pairs`, `_compute_checksum_from_disk`, `_clone_model_with_modified_module`. / 该测试模块通过有针对性的断言与夹具，验证 update weights from disk 的实现。 关键符号包括 `_resolve_active_model_pairs`, `_compute_checksum_from_disk`, `_clone_model_with_modified_module`。

## Line-by-Line Analysis / 逐行分析
### Lines 1-196: Imports and module setup / 导入与模块初始化
```python
"""Tests for diffusion `update_weights_from_disk`.

This module verifies the ability to update model weights in place without restarting
the server, which is critical for RL workflows and iterative fine-tuning scenarios.

Author:

Menyang Liu, https://github.com/dreamyang-liu
Chenyang Zhao, https://github.com/zhaochenyang20

We use two model pairs for testing (base model / instruct model pairs):

- FLUX.2-klein-base-4B / FLUX.2-klein-4B
- Qwen/Qwen-Image / Qwen/Qwen-Image-2512
# ...
]


_CI_MODEL_PAIR_ENV = "SGLANG_MMGEN_UPDATE_WEIGHTS_PAIR"
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 199-215: Function `_resolve_active_model_pairs` / 函数 `_resolve_active_model_pairs`
```python
def _resolve_active_model_pairs() -> list[tuple[str, str]]:
    if not is_in_ci():
        return _ALL_MODEL_PAIRS

    pair_by_id = {pair[0].split("/")[-1]: pair for pair in _ALL_MODEL_PAIRS}
    selected_pair_id = os.environ.get(_CI_MODEL_PAIR_ENV)
    if selected_pair_id is None:
        return [random.choice(_ALL_MODEL_PAIRS)]

    selected_pair = pair_by_id.get(selected_pair_id)
    if selected_pair is None:
        valid_ids = ", ".join(sorted(pair_by_id))
        raise ValueError(
            f"Invalid {_CI_MODEL_PAIR_ENV}={selected_pair_id!r}. "
            f"Expected one of: {valid_ids}."
        )
    return [selected_pair]
```
**EN:** This function drives `_resolve_active_model_pairs`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_resolve_active_model_pairs`。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 216-219: Top-level configuration / 顶层配置
```python


_ACTIVE_MODEL_PAIRS = _resolve_active_model_pairs()
_PAIR_IDS = [p[0].split("/")[-1] for p in _ACTIVE_MODEL_PAIRS]
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

### Lines 222-241: Function `_compute_checksum_from_disk` / 函数 `_compute_checksum_from_disk`
```python
@functools.lru_cache(maxsize=None)
def _compute_checksum_from_disk(model_path: str, module_name: str) -> str:
    """Compute SHA-256 checksum from safetensors files on disk.

    Uses the same compute_weights_checksum function as the server,
    so the checksums are directly comparable.

    Results are cached (keyed on model_path and module_name) because the
    same disk checksum is requested multiple times across tests.
    """
    local_path = maybe_download_model(model_path)
    weights_dir = os.path.join(local_path, module_name)
    assert os.path.exists(
        weights_dir
# ...
    safetensors_files = _list_safetensors_files(weights_dir)
    assert safetensors_files, f"No safetensors files in {weights_dir}"

    return compute_weights_checksum(safetensors_weights_iterator(safetensors_files))
```
**EN:** This function drives `_compute_checksum_from_disk` with inputs such as `model_path`, `module_name`. Compute SHA-256 checksum from safetensors files on disk.
**CN:** 这个函数负责 `_compute_checksum_from_disk`，主要处理 `model_path`, `module_name` 等输入。 文档字符串说明：Compute SHA-256 checksum from safetensors files on disk.

### Lines 244-282: Function `_clone_model_with_modified_module` / 函数 `_clone_model_with_modified_module`
```python
def _clone_model_with_modified_module(
    src_model: str,
    dst_model: str,
    target_module: str,
    transform_safetensor: Callable[[str, str], None],
) -> None:
    # Symlink root-level files (model_index.json, etc.).
    for fname in os.listdir(src_model):
        src_path = os.path.join(src_model, fname)
        dst_path = os.path.join(dst_model, fname)
        if os.path.isfile(src_path) and not os.path.exists(dst_path):
            os.symlink(src_path, dst_path)

    for module_dir in sorted(os.listdir(src_model)):
# ...
                continue

            transform_safetensor(src_file, dst_file)
            transformed = True
```
**EN:** This function drives `_clone_model_with_modified_module` with inputs such as `src_model`, `dst_model`, `target_module`, `transform_safetensor`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_clone_model_with_modified_module`，主要处理 `src_model`, `dst_model`, `target_module`, `transform_safetensor` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 285-295: Function `_truncate_safetensor` / 函数 `_truncate_safetensor`
```python
def _truncate_safetensor(src_file: str, dst_file: str) -> None:
    shutil.copy2(src_file, dst_file)
    size = os.path.getsize(dst_file)
    with open(dst_file, "r+b") as f:
        f.truncate(size - 2)
    logger.info(
        "Created corrupted safetensors: %s (%d -> %d bytes)",
        dst_file,
        size,
        size - 2,
    )
```
**EN:** This function drives `_truncate_safetensor` with inputs such as `src_file`, `dst_file`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_truncate_safetensor`，主要处理 `src_file`, `dst_file` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 298-305: Function `_perturb_safetensor` / 函数 `_perturb_safetensor`
```python
def _perturb_safetensor(src_file: str, dst_file: str) -> None:

    tensors = load_file(src_file)
    perturbed = {
        k: (t + 0.01 if t.is_floating_point() else t) for k, t in tensors.items()
    }
    save_file(perturbed, dst_file)
    logger.info("Created perturbed safetensors: %s", dst_file)
```
**EN:** This function drives `_perturb_safetensor` with inputs such as `src_file`, `dst_file`. The body in this range contains the main control flow, validation, and data movement for this operation.
**CN:** 这个函数负责 `_perturb_safetensor`，主要处理 `src_file`, `dst_file` 等输入。 这一段实现了该操作的主要控制流、校验和数据传递。

### Lines 308-360: Class `_UpdateWeightsApiMixin` / 类 `_UpdateWeightsApiMixin`
```python
class _UpdateWeightsApiMixin:
    def _update_weights(
        self,
        base_url: str,
        model_path: str,
        flush_cache: bool = True,
        target_modules: list[str] | None = None,
        timeout: int = 300,
    ) -> tuple[dict, int]:
        payload = {"model_path": model_path, "flush_cache": flush_cache}
        if target_modules is not None:
            payload["target_modules"] = target_modules
        response = requests.post(
            f"{base_url}/update_weights_from_disk",
# ...
            f"Checksum mismatch on '{_TRANSFORMER_MODULE}'\n"
            f"  expected({expected_model}): {expected_cs}\n"
            f"  server: {server_cs}"
        )
```
**EN:** This class models `_UpdateWeightsApiMixin`. Important methods include `_update_weights`, `_get_weights_checksum`, `_assert_server_matches_model`.
**CN:** 该类实现 `_UpdateWeightsApiMixin`。 其中较重要的方法包括 `_update_weights`, `_get_weights_checksum`, `_assert_server_matches_model`。

### Lines 363-618: Class `TestUpdateWeightsFromDisk` / 类 `TestUpdateWeightsFromDisk`
```python
class TestUpdateWeightsFromDisk(_UpdateWeightsApiMixin):

    @pytest.fixture(
        scope="class",
        params=_ACTIVE_MODEL_PAIRS,
        ids=_PAIR_IDS,
    )
    def diffusion_server_no_offload(self, request):
        default_model, source_model = request.param
        port = get_dynamic_server_port()
        wait_deadline = float(os.environ.get("SGLANG_TEST_WAIT_SECS", "600"))

        manager = ServerManager(
            model=default_model,
# ...
            ), f"Text encoder module '{name}' should stay equal to perturbed"
            assert rolled_back_checksums.get(name) == base_checksums.get(
                name
            ), f"Text encoder module '{name}' should stay equal to base"
```
**EN:** This class models `TestUpdateWeightsFromDisk` as a specialization of `_UpdateWeightsApiMixin`. Important methods include `diffusion_server_no_offload`, `test_update_weights_from_disk_default`, `test_update_weights_specific_modules`, `test_update_weights_nonexistent_model`.
**CN:** 该类实现 `TestUpdateWeightsFromDisk`，并继承/扩展 `_UpdateWeightsApiMixin`。 其中较重要的方法包括 `diffusion_server_no_offload`, `test_update_weights_from_disk_default`, `test_update_weights_specific_modules`, `test_update_weights_nonexistent_model`。

### Lines 621-671: Class `TestUpdateWeightsFromDiskWithOffload` / 类 `TestUpdateWeightsFromDiskWithOffload`
```python
class TestUpdateWeightsFromDiskWithOffload(_UpdateWeightsApiMixin):
    """Test update_weights_from_disk with layerwise offload enabled."""

    @pytest.fixture(scope="class", params=_ACTIVE_MODEL_PAIRS, ids=_PAIR_IDS)
    def diffusion_server_with_offload(self, request):
        default_model, source_model = request.param
        port = get_dynamic_server_port()
        wait_deadline = float(os.environ.get("SGLANG_TEST_WAIT_SECS", "600"))

        local_source = maybe_download_model(source_model)
        perturbed_vae_model_dir = tempfile.mkdtemp(prefix="sglang_perturbed_vae_")

        clone_thread = threading.Thread(
            target=_clone_model_with_modified_module,
# ...
        message = result.get("message", "")
        assert "Shape mismatch" not in message, f"Shape mismatch detected: {message}"

        self._assert_server_matches_model(base_url, perturbed_model_dir)
```
**EN:** This class models `TestUpdateWeightsFromDiskWithOffload` as a specialization of `_UpdateWeightsApiMixin`. Test update_weights_from_disk with layerwise offload enabled. Important methods include `diffusion_server_with_offload`, `test_update_weights_with_offload_enabled`.
**CN:** 该类实现 `TestUpdateWeightsFromDiskWithOffload`，并继承/扩展 `_UpdateWeightsApiMixin`。 文档字符串指出：Test update_weights_from_disk with layerwise offload enabled. 其中较重要的方法包括 `diffusion_server_with_offload`, `test_update_weights_with_offload_enabled`。

### Lines 672-675: Top-level configuration / 顶层配置
```python


if __name__ == "__main__":
    sys.exit(pytest.main([__file__, "-v", "-s"]))
```
**EN:** Imports, fixtures, and helper statements prepare the surrounding test scenarios and shared state.
**CN:** 导入、夹具与辅助语句为后续测试场景和共享状态做准备。

## Key Concepts / 关键概念
- PyTorch tensor computation / PyTorch 张量计算
- Model/component loading / 模型/组件加载
- Pipeline orchestration / 流水线编排
- Caching strategy / 缓存策略
- Automated verification / 自动化验证

## Dependencies / 依赖关系
- **Internal / 内部**: `sglang.multimodal_gen.runtime.loader.utils`, `sglang.multimodal_gen.runtime.loader.weight_utils`, `sglang.multimodal_gen.runtime.utils.hf_diffusers_utils`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.test.server.test_server_utils`, `sglang.multimodal_gen.test.test_utils`
- **External / 外部**: `__future__`, `pytest`, `requests`, `safetensors.torch`
- **Stdlib / 标准库**: `functools`, `os`, `random`, `shutil`, `sys`, `tempfile`, `threading`, `collections.abc`
