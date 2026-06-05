# gpu_allocator.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `sgl-model-gateway/e2e_test/infra/gpu_allocator.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module exercises gpu allocator behavior in the end-to-end infrastructure layer. It encodes scenarios, assertions, and shared setup used to verify correctness and regressions. / 该测试模块覆盖 端到端测试基础设施 中与 gpu allocator 相关的行为，包含用于验证正确性与回归问题的场景、断言以及共享初始化逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Module docstring
```python
"""GPU detection and slot allocation for parallel test execution."""
```
**EN:** This string literal documents the file-level intent and provides context before the executable code begins.
**CN:** 这个字符串字面量说明了文件级意图，在可执行代码开始前为读者提供上下文。

### Lines 3-11: Imports and dependencies
```python
from __future__ import annotations

import logging
import os
import socket
import threading
import time
from contextlib import contextmanager
from dataclasses import dataclass
```
**EN:** This block imports `__future__`, `logging`, `os`, `socket`, and 4 more, supplying the dependencies required by the rest of the file.
**CN:** 该代码块导入后续逻辑所需的模块，包括标准库工具、第三方包以及项目内部代码。

### Lines 13-13: Module constants and configuration
```python
logger = logging.getLogger(__name__)
```
**EN:** This section defines module-level names such as `logger`, centralizing configuration and shared state.
**CN:** 该部分定义了模块级常量或共享状态，为文件中的其余逻辑提供统一配置。

### Lines 14-22: Guarded setup and fallback handling
```python

# Try to import nvidia-ml-py for GPU detection
try:
    import pynvml

    NVML_AVAILABLE = True
except ImportError:
    NVML_AVAILABLE = False
    logger.debug("nvidia-ml-py not available, GPU detection will be limited")
```
**EN:** This block wraps initialization in exception handling so optional functionality can fail gracefully or expose clearer diagnostics.
**CN:** 该代码块使用异常处理包裹初始化流程，使可选功能在失败时能够优雅回退或提供更清晰的诊断信息。

### Lines 23-42: Helper function `nvml_context`
```python


@contextmanager
def nvml_context():
    """Context manager for NVML initialization/shutdown.

    Usage:
        with nvml_context():
            handle = pynvml.nvmlDeviceGetHandleByIndex(0)
            ...
    """
    if not NVML_AVAILABLE:
        yield
        return

    try:
        pynvml.nvmlInit()
        yield
    finally:
        pynvml.nvmlShutdown()
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 43-55: Class `GPUInfo`
```python


@dataclass
class GPUInfo:
    """Information about a single GPU."""

    id: int
    name: str
    memory_mb: int

    @property
    def memory_gb(self) -> float:
        return self.memory_mb / 1024
```
**EN:** Class `GPUInfo` groups related state and behavior. It exposes 1 method(s) that implement the module's primary abstraction.
**CN:** 类 `GPUInfo` 将相关状态与行为封装在一起，提供 1 个方法来实现本模块的核心抽象。

### Lines 56-73: Class `GPUSlot`
```python


@dataclass
class GPUSlot:
    """A slot representing one or more GPUs allocated for a model."""

    gpu_ids: list[int]
    total_memory_mb: int
    assigned_model: str | None = None
    port: int | None = None

    @property
    def total_memory_gb(self) -> float:
        return self.total_memory_mb / 1024

    def cuda_visible_devices(self) -> str:
        """Return CUDA_VISIBLE_DEVICES string for this slot."""
        return ",".join(str(g) for g in self.gpu_ids)
```
**EN:** Class `GPUSlot` groups related state and behavior. It exposes 2 method(s) that implement the module's primary abstraction.
**CN:** 类 `GPUSlot` 将相关状态与行为封装在一起，提供 2 个方法来实现本模块的核心抽象。

### Lines 76-95: Helper function `get_open_port`
```python
def get_open_port() -> int:
    """Get an available port by binding to port 0 and reading the assigned port.

    Capped below 55536 so sglang's `--grpc-mode` default
    `grpc_port = port + 10000` (in srt/server_args.py) cannot overflow
    16-bit port range. The kernel's ephemeral range is typically
    32768-60999, so a cap > 32768 keeps reasonable headroom while
    avoiding the rare overflow that crashes worker startup.
    """
    for _ in range(20):
        with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
            s.bind(("", 0))
            s.listen(1)
            port = s.getsockname()[1]
        if port < 55536:
            return port
    raise RuntimeError(
        "Failed to allocate an open port below 55536 after 20 attempts; "
        "kernel ephemeral range may be unusually high"
    )
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 98-106: Helper function `get_physical_device_indices`
```python
def get_physical_device_indices(devices: list[int]) -> list[int]:
    """Map logical device indices to physical indices based on CUDA_VISIBLE_DEVICES."""
    visible_devices = os.environ.get("CUDA_VISIBLE_DEVICES")
    if visible_devices is None:
        return devices

    visible_indices = [int(x) for x in visible_devices.split(",")]
    index_mapping = {i: physical for i, physical in enumerate(visible_indices)}
    return [index_mapping[i] for i in devices if i in index_mapping]
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 109-124: Helper function `get_gpu_memory_usage`
```python
def get_gpu_memory_usage(device_id: int) -> tuple[float, float]:
    """Get GPU memory usage in GB (used, total).

    Args:
        device_id: Physical GPU device ID

    Returns:
        Tuple of (used_gb, total_gb)
    """
    if not NVML_AVAILABLE:
        return (0.0, 0.0)

    with nvml_context():
        handle = pynvml.nvmlDeviceGetHandleByIndex(device_id)
        mem_info = pynvml.nvmlDeviceGetMemoryInfo(handle)
        return (mem_info.used / (1024**3), mem_info.total / (1024**3))
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 127-202: Helper function `wait_for_gpu_memory_to_clear`
```python
def wait_for_gpu_memory_to_clear(
    *,
    devices: list[int],
    threshold_bytes: int | None = None,
    threshold_ratio: float | None = None,
    timeout_s: float = 120,
) -> None:
    """Wait for GPU memory to be freed below a threshold.

    Args:
        devices: List of logical GPU device IDs to check
        threshold_bytes: Memory threshold in bytes (used <= threshold)
        threshold_ratio: Memory threshold as ratio (used/total <= ratio)
        timeout_s: Timeout in seconds

    Raises:
        ValueError: If memory doesn't clear within timeout
    """
    if not NVML_AVAILABLE:
        logger.warning("nvidia-ml-py not available, skipping memory wait")
        return

    if threshold_bytes is None and threshold_ratio is None:
        raise ValueError("Must specify threshold_bytes or threshold_ratio")

    physical_devices = get_physical_device_indices(devices)
    start_time = time.time()

    with nvml_context():
        while True:
            output: dict[int, str] = {}
            output_raw: dict[int, tuple[float, float]] = {}

            for device in physical_devices:
                handle = pynvml.nvmlDeviceGetHandleByIndex(device)
                mem_info = pynvml.nvmlDeviceGetMemoryInfo(handle)
                gb_used = mem_info.used / (1024**3)
                gb_total = mem_info.total / (1024**3)
                output_raw[device] = (gb_used, gb_total)
                output[device] = f"{gb_used:.02f}/{gb_total:.02f}"

            logger.debug(
                "GPU memory used/total (GiB): %s",
                " ".join(f"{k}={v}" for k, v in output.items()),
            )

            if threshold_bytes is not None:

                def is_free(used: float, total: float) -> bool:
                    return used <= threshold_bytes / (1024**3)

                threshold_desc = f"{threshold_bytes / (1024**3):.1f} GiB"
            else:

                def is_free(used: float, total: float) -> bool:
                    return used / total <= threshold_ratio  # type: ignore[operator]

                threshold_desc = f"{threshold_ratio:.2%}"  # type: ignore[str-format]

            dur_s = time.time() - start_time
            if all(is_free(used, total) for used, total in output_raw.values()):
                logger.info(
                    "GPU memory cleared on devices %s (threshold=%s) in %.1fs",
                    devices,
                    threshold_desc,
                    dur_s,
                )
                return

            if dur_s >= timeout_s:
                raise ValueError(
                    f"GPU memory on devices {devices} not freed after {dur_s:.1f}s "
                    f"(threshold={threshold_desc})"
                )

            time.sleep(5)
```
**EN:** This helper function supports the surrounding tests by centralizing reusable setup, data shaping, or assertions.
**CN:** 该辅助函数通过集中可复用的初始化、数据整理或断言逻辑，为周围测试提供支撑。

### Lines 205-450: Class `GPUAllocator`
```python
class GPUAllocator:
    """Detects GPUs and assigns them to model slots using bin-packing."""

    def __init__(self, gpus: list[GPUInfo] | None = None):
        """Initialize the allocator.

        Args:
            gpus: Optional list of GPUs. If None, auto-detects via nvidia-ml-py.
        """
        self.gpus = gpus if gpus is not None else self._detect_gpus()
        self.slots: list[GPUSlot] = []
        self._used_gpus: set[int] = set()  # Track GPUs used across all allocations
        self._lock = threading.RLock()  # Protects slots and _used_gpus

    def _detect_gpus(self) -> list[GPUInfo]:
        """Auto-detect available GPUs via nvidia-ml-py (NVML)."""
        if not NVML_AVAILABLE:
            logger.warning("nvidia-ml-py not available - no GPUs detected")
            return []

        # Check for CUDA_VISIBLE_DEVICES restriction
        visible_devices = os.environ.get("CUDA_VISIBLE_DEVICES")
        allowed_ids: set[int] | None = None
        if visible_devices:
            allowed_ids = set(int(x) for x in visible_devices.split(",") if x.strip())

        try:
            with nvml_context():
                device_count = pynvml.nvmlDeviceGetCount()

                gpus = []
                for idx in range(device_count):
                    # Skip GPUs not in CUDA_VISIBLE_DEVICES if set
                    if allowed_ids is not None and idx not in allowed_ids:
                        continue

                    handle = pynvml.nvmlDeviceGetHandleByIndex(idx)
                    name = pynvml.nvmlDeviceGetName(handle)
                    # Handle bytes vs string return type (varies by pynvml version)
                    if isinstance(name, bytes):
                        name = name.decode("utf-8", errors="replace")
                    mem_info = pynvml.nvmlDeviceGetMemoryInfo(handle)
                    # Convert bytes to MB
                    memory_mb = mem_info.total // (1024 * 1024)

                    gpus.append(GPUInfo(idx, name, memory_mb))

            logger.info("Detected %d GPUs: %s", len(gpus), [g.name for g in gpus])
            return gpus

        except pynvml.NVMLError as e:
            logger.warning("NVML error during GPU detection: %s", e)
            return []
        except Exception as e:
            logger.warning("Failed to detect GPUs: %s", e)
            return []

    def allocate_slots(
        self, model_specs: dict[str, dict], preserve_order: bool = False
    ) -> list[GPUSlot]:
        """Allocate GPU slots based on model memory requirements.

        Uses a first-fit decreasing bin-packing algorithm by default:
        1. Sort models by memory requirement (largest first)
        2. For each model, find the first GPU(s) that can fit it
        3. For multi-GPU models, find consecutive GPUs

        When preserve_order=True, processes models in dict insertion order
        (test collection order) instead of sorting by memory. This ensures
        models needed by earlier tests are allocated first.

        Note: This method tracks used GPUs across multiple calls, so subsequent
        allocations will use different GPUs than previous ones.

        Thread-safe: Protected by internal lock.

        Args:
            model_specs: Dict of model_id -> spec dict with 'memory_gb' and 'tp' keys
            preserve_order: If True, allocate in dict order (test order) instead
                           of sorting by memory size. Default False.

        Returns:
            List of GPUSlots with assigned models (only the newly allocated slots)
        """
        with self._lock:
            return self._allocate_slots_unlocked(model_specs, preserve_order)

    def _allocate_slots_unlocked(
        self, model_specs: dict[str, dict], preserve_order: bool = False
    ) -> list[GPUSlot]:
        """Internal allocation logic. Caller must hold _lock."""
        if not self.gpus:
            logger.warning("No GPUs available for allocation")
            return []

        if preserve_order:
            # Process in dict insertion order (test collection order)
            ordered_models = list(model_specs.items())
        else:
            # Sort models by memory requirement (largest first for better packing)
            ordered_models = sorted(
                model_specs.items(),
                key=lambda x: x[1].get("memory_gb", 0),
                reverse=True,
            )

        # Track new slots allocated in this call
        new_slots: list[GPUSlot] = []

        for model_id, spec in ordered_models:
            memory_gb = spec.get("memory_gb", 16)
            tp_size = spec.get("tp", 1)

            # Find available GPUs (not used by any previous allocation)
            available = [g for g in self.gpus if g.id not in self._used_gpus]

            if tp_size == 1:
                # Single GPU - find one with enough memory
                for gpu in available:
                    if gpu.memory_gb >= memory_gb:
                        slot = GPUSlot(
                            gpu_ids=[gpu.id],
                            total_memory_mb=gpu.memory_mb,
                            assigned_model=model_id,
                            port=get_open_port(),
                        )
                        new_slots.append(slot)
                        self._used_gpus.add(gpu.id)
                        logger.info(
                            "Allocated GPU %d (%s, %.1fGB) for %s",
                            gpu.id,
                            gpu.name,
                            gpu.memory_gb,
                            model_id,
                        )
                        break
                else:
                    logger.warning(
                        "No GPU with %.1fGB available for %s (used: %s)",
                        memory_gb,
                        model_id,
                        self._used_gpus,
                    )
            else:
                # Multi-GPU - find consecutive GPUs with enough total memory
                # Sort available by ID for consecutive allocation
                available_sorted = sorted(available, key=lambda g: g.id)

                for i in range(len(available_sorted) - tp_size + 1):
                    candidate_gpus = available_sorted[i : i + tp_size]
                    total_mem = sum(g.memory_mb for g in candidate_gpus)

                    if total_mem >= memory_gb * 1024:
                        gpu_ids = [g.id for g in candidate_gpus]
                        slot = GPUSlot(
                            gpu_ids=gpu_ids,
                            total_memory_mb=total_mem,
                            assigned_model=model_id,
                            port=get_open_port(),
                        )
                        new_slots.append(slot)
                        self._used_gpus.update(gpu_ids)
                        logger.info(
                            "Allocated GPUs %s (%.1fGB total) for %s (tp=%d)",
                            gpu_ids,
                            total_mem / 1024,
                            model_id,
                            tp_size,
                        )
                        break
                else:
                    logger.warning(
                        "No %d consecutive GPUs with %.1fGB available for %s (used: %s)",
                        tp_size,
                        memory_gb,
                        model_id,
                        self._used_gpus,
                    )

        # Add new slots to existing slots list
        self.slots.extend(new_slots)
        return new_slots

    def get_slot_for_model(self, model_id: str) -> GPUSlot | None:
        """Get the slot assigned to a specific model.

        Thread-safe: Protected by internal lock.
        """
        with self._lock:
            for slot in self.slots:
                if slot.assigned_model == model_id:
                    return slot
            return None

    def release_gpus(self, gpu_ids: list[int]) -> None:
        """Release GPUs back to the available pool.

        Thread-safe: Protected by internal lock.

        Args:
            gpu_ids: List of GPU IDs to release.
        """
        with self._lock:
            for gpu_id in gpu_ids:
                self._used_gpus.discard(gpu_id)
            # Remove slots that used these GPUs
            self.slots = [
                s for s in self.slots if not any(g in gpu_ids for g in s.gpu_ids)
            ]
            logger.info("Released GPUs %s, now used: %s", gpu_ids, self._used_gpus)

    def release_slot(self, slot: GPUSlot) -> None:
        """Release a GPU slot back to the available pool.

        Args:
            slot: The GPUSlot to release.
        """
        self.release_gpus(slot.gpu_ids)

    def available_gpus(self) -> list[int]:
        """Get list of available (unused) GPU IDs.

        Thread-safe: Protected by internal lock.

        Returns:
            List of GPU IDs that are not currently allocated.
        """
        with self._lock:
            return [g.id for g in self.gpus if g.id not in self._used_gpus]

    def summary(self) -> str:
        """Return a summary of GPU allocations.

        Thread-safe: Protected by internal lock.
        """
        with self._lock:
            lines = ["GPU Allocation Summary:"]
            lines.append(f"  Total GPUs: {len(self.gpus)}")
            lines.append(f"  Used GPUs: {sorted(self._used_gpus)}")
            lines.append(f"  Allocated Slots: {len(self.slots)}")
            for slot in self.slots:
                lines.append(
                    f"    - {slot.assigned_model}: GPUs {slot.gpu_ids} "
                    f"({slot.total_memory_gb:.1f}GB) port={slot.port}"
                )
            return "\n".join(lines)
```
**EN:** Class `GPUAllocator` groups related state and behavior. It exposes 9 method(s) that implement the module's primary abstraction.
**CN:** 类 `GPUAllocator` 将相关状态与行为封装在一起，提供 9 个方法来实现本模块的核心抽象。

## Key Concepts / 关键概念
- Pytest-driven validation / 基于 Pytest 的验证流程
- Test infrastructure orchestration / 测试基础设施编排
- Primary classes: `GPUInfo`, `GPUSlot`, `GPUAllocator` / 主要类：`GPUInfo`, `GPUSlot`, `GPUAllocator`
- Reusable functions: `nvml_context`, `get_open_port`, `get_physical_device_indices`, `get_gpu_memory_usage`, and 1 more / 可复用函数：`nvml_context`, `get_open_port`, `get_physical_device_indices`, `get_gpu_memory_usage`, and 1 more

## Dependencies / 依赖关系
- **Standard library / 标准库**: `__future__`, `contextlib`, `dataclasses`, `logging`, `os`, `socket`, `threading`, `time`
- **Third-party / 第三方**: `pynvml`
- **Internal / 内部模块**: None explicitly imported / 未显式导入
