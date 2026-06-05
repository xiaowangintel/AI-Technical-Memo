# sampling_params.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `python/sglang/multimodal_gen/configs/sample/sampling_params.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This file belongs to the configuration layer. It centers on `DataType`, `SamplingParams`, and `CacheParams`, organizing the main control flow, data structures, or helper routines for this module. / 该文件属于配置层。它围绕 `DataType`、`SamplingParams` 和 `CacheParams` 展开，组织了本模块的主控制流程、数据结构或辅助逻辑。

## Line-by-Line Analysis / 逐行分析
### Lines 4-21: module setup and imports / 模块初始化与导入
```python
import argparse
import dataclasses
import hashlib
import json
import math
import os
import os.path
import re
import time
import unicodedata
import uuid
from dataclasses import dataclass, field
from enum import Enum, auto
from typing import TYPE_CHECKING, Any, ClassVar

from sglang.multimodal_gen.configs.post_training import RLRolloutArgs
from sglang.multimodal_gen.runtime.utils.logging_utils import init_logger
from sglang.multimodal_gen.utils import StoreBoolean, expand_path_fields
```
**EN:** This block establishes the module context and imports `argparse`, `dataclasses`, `hashlib`, `json`, `math`, and `os`. These dependencies provide the symbols needed by the rest of the file.
**CN:** 该代码块建立模块上下文，并导入 `argparse`、`dataclasses`、`hashlib`、`json`、`math` 和 `os`。这些依赖为后续实现提供所需符号。

### Lines 23-26: supporting statements / 辅助语句
```python
logger = init_logger(__name__)

if TYPE_CHECKING:
    from sglang.multimodal_gen.runtime.server_args import ServerArgs
```
**EN:** This block gathers supporting statements at module scope. It updates names such as `logger`. The code collaborates with `init_logger`.
**CN:** 该代码块汇集了位于模块作用域的辅助语句。 它会更新 `logger` 等名称。 代码会与 `init_logger` 协同工作。

### Lines 29-47: `_json_safe` implementation / `_json_safe` 实现
```python
def _json_safe(obj: Any):
    """
    Recursively convert objects to JSON-serializable forms.
    - Enums -> their name
    - Callables -> stable module-qualified name
    - Sets/Tuples -> lists
    - Dicts/Lists -> recursively processed
    """
    if isinstance(obj, Enum):
        return obj.name
    if callable(obj):
        module = getattr(obj, "__module__", None)
        qualname = getattr(obj, "__qualname__", getattr(obj, "__name__", repr(obj)))
        return f"{module}.{qualname}" if module else qualname
    if isinstance(obj, dict):
        return {k: _json_safe(v) for k, v in obj.items()}
    if isinstance(obj, (list, tuple, set)):
        return [_json_safe(v) for v in obj]
    return obj
```
**EN:** This block defines function `_json_safe`. Recursively convert objects to JSON-serializable forms. - Enums -> their name - Callables -> stable module-qualified name - Sets/Tuples -> lists - Dicts/Lists -> recursively processed Key calls include `isinstance`, `callable`, `getattr`, `_json_safe`, and `repr`. The implementation branches on conditions. Parameters such as `obj` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_json_safe`。 它用于处理 json safe 相关逻辑。 关键调用包括 `isinstance`、`callable`、`getattr`、`_json_safe` 和 `repr`。 实现中包含条件分支。 本段逻辑主要由 `obj` 等参数驱动。

### Lines 50-51: `generate_request_id` implementation / `generate_request_id` 实现
```python
def generate_request_id() -> str:
    return str(uuid.uuid4())
```
**EN:** This block defines function `generate_request_id`. It generates request id. Key calls include `str`, and `uuid.uuid4`.
**CN:** 该代码块定义了函数 `generate_request_id`。 它用于生成request id。 关键调用包括 `str` 和 `uuid.uuid4`。

### Lines 54-72: `_sanitize_filename` implementation / `_sanitize_filename` 实现
```python
def _sanitize_filename(name: str, replacement: str = "_", max_length: int = 150) -> str:
    """Create a filesystem- and ffmpeg-friendly filename.

    - Normalize to ASCII (drop accents and unsupported chars)
    - Replace spaces with underscores
    - Replace any char not in [A-Za-z0-9_.-] with replacement
    - Collapse multiple underscores
    - Trim leading/trailing dots/underscores and limit length
    """
    normalized = unicodedata.normalize("NFKD", name)
    ascii_name = normalized.encode("ascii", "ignore").decode("ascii")
    ascii_name = ascii_name.replace(" ", "_")
    ascii_name = re.sub(r"[^A-Za-z0-9._-]", replacement, ascii_name)
    ascii_name = re.sub(r"_+", "_", ascii_name).strip("._")
    if not ascii_name:
        ascii_name = "output"
    if max_length and len(ascii_name) > max_length:
        ascii_name = ascii_name[:max_length]
    return ascii_name
```
**EN:** This block defines function `_sanitize_filename`. Create a filesystem- and ffmpeg-friendly filename. - Normalize to ASCII (drop accents and unsupported chars) - Replace spaces with underscores - Replace any char not in [A-Za-z0-9_.-] with replacement - Collapse multiple underscores - Trim leading/trailing dots/underscores and limit length Key calls include `unicodedata.normalize`, `normalized.encode.decode`, `ascii_name.replace`, `re.sub`, and `re.sub.strip`. The implementation branches on conditions. Parameters such as `name`, `replacement`, and `max_length` drive the behavior in this section.
**CN:** 该代码块定义了函数 `_sanitize_filename`。 它用于处理 sanitize filename 相关逻辑。 关键调用包括 `unicodedata.normalize`、`normalized.encode.decode`、`ascii_name.replace`、`re.sub` 和 `re.sub.strip`。 实现中包含条件分支。 本段逻辑主要由 `name`、`replacement` 和 `max_length` 等参数驱动。

### Lines 75-75: `DataType` class overview / `DataType` 类概览
```python
class DataType(Enum):
```
**EN:** This block defines class `DataType`. It encapsulates data type behavior. It inherits from `Enum`.
**CN:** 该代码块定义了类 `DataType`。 它用于封装 data type 相关行为。 它继承自 `Enum`。

### Lines 76-78: supporting statements / 辅助语句
```python
    IMAGE = auto()
    VIDEO = auto()
    MESH = auto()
```
**EN:** This block gathers supporting statements inside `DataType`. It updates names such as `IMAGE`, `VIDEO`, and `MESH`. The code collaborates with `auto`.
**CN:** 该代码块汇集了位于 `DataType` 内部的辅助语句。 它会更新 `IMAGE`、`VIDEO` 和 `MESH` 等名称。 代码会与 `auto` 协同工作。

### Lines 80-85: `get_default_extension` implementation / `get_default_extension` 实现
```python
    def get_default_extension(self) -> str:
        if self == DataType.IMAGE:
            return "png"
        if self == DataType.VIDEO:
            return "mp4"
        return "glb"
```
**EN:** This block defines method `get_default_extension` on `DataType`. It retrieves default extension. The implementation branches on conditions.
**CN:** 该代码块定义了 `DataType` 的方法 `get_default_extension`。 它用于获取default extension。 实现中包含条件分支。

### Lines 89-96: `SamplingParams` class overview / `SamplingParams` 类概览
```python
class SamplingParams:
    """
    Sampling parameters for generation.

    Dynamic batching compares these fields for compatibility, except fields
    marked with `batch_sig_exclude`.
    """
```
**EN:** This block defines class `SamplingParams`. Sampling parameters for generation. Dynamic batching compares these fields for compatibility, except fields marked with `batch_sig_exclude`.
**CN:** 该代码块定义了类 `SamplingParams`。 它用于封装 sampling params 相关行为。

### Lines 97-224: supporting statements / 辅助语句
```python
    data_type: DataType = DataType.VIDEO

    request_id: str | None = field(default=None, metadata={"batch_sig_exclude": True})

    # All fields below are copied from ForwardBatch

    # Image inputs
    image_path: str | list[str] | None = None

    # Text inputs
    prompt: str | list[str] | None = field(
        default=None, metadata={"batch_sig_exclude": True}
    )
    negative_prompt: str = (
        "Bright tones, overexposed, static, blurred details, subtitles, style, works, paintings, images, static, overall gray, worst quality, low quality, JPEG compression residue, ugly, incomplete, extra fingers, poorly drawn hands, poorly drawn faces, deformed, disfigured, misshapen limbs, fused fingers, still picture, messy background, three legs, many people in the background, walking backwards"
    )
    prompt_path: str | None = field(default=None, metadata={"batch_sig_exclude": True})
    output_path: str | None = field(default=None, metadata={"batch_sig_exclude": True})
    output_file_name: str | None = field(
        default=None, metadata={"batch_sig_exclude": True}
    )
    output_quality: str | None = "default"
    output_compression: int | None = None

    # Frame interpolation
    enable_frame_interpolation: bool = False
    frame_interpolation_exp: int = 1  # 1=2x, 2=4x
    frame_interpolation_scale: float = 1.0  # RIFE inference scale (0.5 for high-res)
    frame_interpolation_model_path: str | None = (
        None  # local dir or HF repo ID with flownet.pkl (default: elfgum/RIFE-4.22.lite)
    )

    # Upscaling
    enable_upscaling: bool = False
    upscaling_model_path: str | None = (
        None  # local .pth, HF repo ID, or repo_id:filename (default: ai-forever/Real-ESRGAN)
    )
    upscaling_scale: int = 4

    # Batch info
    num_outputs_per_prompt: int = 1
    seed: int | list[int] = field(default=42, metadata={"batch_sig_exclude": True})
    generator_device: str | None = None  # None means use the pipeline/model default

    # Original dimensions (before VAE scaling)
    num_frames: int = 1  # Default for image models
    num_frames_round_down: bool = (
        False  # Whether to round down num_frames if it's not divisible by num_gpus
    )

    # Subclasses can set these to provide model-specific default resolutions.
    # The base __post_init__ will apply them when height/width are not provided.
    _default_height: ClassVar[int | None] = None
    _default_width: ClassVar[int | None] = None

    height: int | None = None
    width: int | None = None
    fps: int = 24

    # Resolution validation
    supported_resolutions: list[tuple[int, int]] | None = field(
        default=None, metadata={"batch_sig_exclude": True}
    )  # None means all resolutions allowed

    # Denoising parameters
    num_inference_steps: int = None
    guidance_scale: float = 1.0
    guidance_scale_2: float = None
    true_cfg_scale: float = None  # for CFG vs guidance distillation (e.g., QwenImage)
    guidance_rescale: float = 0.0
    cfg_normalization: float | bool = 0.0
    boundary_ratio: float | None = None

    # TeaCache parameters
    enable_teacache: bool = False
    teacache_params: Any = (
        None  # TeaCacheParams or WanTeaCacheParams, set by model-specific subclass
    )

    # Profiling
    profile: bool = field(default=False, metadata={"batch_sig_exclude": True})
    num_profiled_timesteps: int = field(default=5, metadata={"batch_sig_exclude": True})
    profile_all_stages: bool = field(
        default=False, metadata={"batch_sig_exclude": True}
    )

    # Debugging
    debug: bool = field(default=False, metadata={"batch_sig_exclude": True})
    perf_dump_path: str | None = field(
        default=None, metadata={"batch_sig_exclude": True}
    )

    # Misc
    save_output: bool = True
    return_frames: bool = field(default=False, metadata={"batch_sig_exclude": True})
    rollout: bool = False
    rollout_sde_type: str = "sde"
    rollout_noise_level: float = 0.7
    rollout_log_prob_no_const: bool = False  # exclude constants in rollout logprob
    rollout_debug_mode: bool = (
        False  # return rollout debug tensors (intermediate states)
    )
    return_trajectory_latents: bool = False  # returns all latents for each timestep
    return_trajectory_decoded: bool = False  # returns decoded latents for each timestep
    rollout_return_denoising_env: bool = (
        False  # populate ``denoising_env`` (image/pos/neg kwargs, guidance) for RL replay
    )
    rollout_return_dit_trajectory: bool = (
        False  # per-step noisy latents + final latent + timesteps (RolloutDitTrajectory)
    )
    # 0-indexed denoising-loop step filters; None = all steps.
    rollout_sde_step_indices: list[int] | None = None
    rollout_return_step_indices: list[int] | None = None
    # if True, disallow user params to override subclass-defined protected fields
    no_override_protected_fields: bool = field(
        default=False, metadata={"batch_sig_exclude": True}
    )
    # whether to adjust num_frames for multi-GPU friendly splitting (default: True)
    adjust_frames: bool = True
    # if True, suppress verbose logging for this request
    suppress_logs: bool = field(default=False, metadata={"batch_sig_exclude": True})

    return_file_paths_only: bool = True
    enable_sequence_shard: bool | None = None
    diffusers_kwargs: dict | None = None

    # Prompt enhancement (ErnieImage)
    use_pe: bool | None = None
```
**EN:** This block gathers supporting statements inside `SamplingParams`. It updates names such as `data_type`, `request_id`, `image_path`, `prompt`, `negative_prompt`, and `prompt_path`. The code collaborates with `field`.
**CN:** 该代码块汇集了位于 `SamplingParams` 内部的辅助语句。 它会更新 `data_type`、`request_id`、`image_path`、`prompt`、`negative_prompt` 和 `prompt_path` 等名称。 代码会与 `field` 协同工作。

### Lines 226-234: `_set_output_file_ext` implementation / `_set_output_file_ext` 实现
```python
    def _set_output_file_ext(self):
        # add extension if needed
        if not any(
            self.output_file_name.endswith(ext)
            for ext in [".mp4", ".jpg", ".png", ".webp", ".obj", ".glb"]
        ):
            self.output_file_name = (
                f"{self.output_file_name}.{self.data_type.get_default_extension()}"
            )
```
**EN:** This block defines method `_set_output_file_ext` on `SamplingParams`. It configures output file ext. Key calls include `any`, `self.output_file_name.endswith`, and `self.data_type.get_default_extension`. The implementation branches on conditions.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `_set_output_file_ext`。 它用于设置output file ext。 关键调用包括 `any`、`self.output_file_name.endswith` 和 `self.data_type.get_default_extension`。 实现中包含条件分支。

### Lines 236-267: `_set_output_file_name` implementation / `_set_output_file_name` 实现
```python
    def _set_output_file_name(self):
        # settle output_file_name
        if (
            self.output_file_name is None
            and self.prompt
            and isinstance(self.prompt, str)
        ):
            # generate a random filename
            # get a hash of current params
            params_dict = dataclasses.asdict(self)
            # Avoid recursion
            params_dict["output_file_name"] = ""

            # Convert to a stable JSON string
            params_str = json.dumps(_json_safe(params_dict), sort_keys=True)
            # Create a hash
            hasher = hashlib.sha256()
            hasher.update(params_str.encode("utf-8"))
            param_hash = hasher.hexdigest()[:8]

            timestamp = time.strftime("%Y%m%d-%H%M%S")
            base = f"{self.prompt[:100]}_{timestamp}_{param_hash}"
            self.output_file_name = base

        if self.output_file_name is None:
            timestamp = time.strftime("%Y%m%d-%H%M%S")
            self.output_file_name = f"output_{timestamp}"

        self.output_file_name = _sanitize_filename(self.output_file_name)

        # Ensure a proper extension is present
        self._set_output_file_ext()
```
**EN:** This block defines method `_set_output_file_name` on `SamplingParams`. It configures output file name. Key calls include `_sanitize_filename`, `self._set_output_file_ext`, `isinstance`, `dataclasses.asdict`, and `json.dumps`. The implementation branches on conditions.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `_set_output_file_name`。 它用于设置output file name。 关键调用包括 `_sanitize_filename`、`self._set_output_file_ext`、`isinstance`、`dataclasses.asdict` 和 `json.dumps`。 实现中包含条件分支。

### Lines 269-288: `__post_init__` implementation / `__post_init__` 实现
```python
    def __post_init__(self) -> None:
        assert self.num_frames >= 1

        if self.width is None and self._default_width is not None:
            self.width = self._default_width
        if self.height is None and self._default_height is not None:
            self.height = self._default_height

        # Handle output_quality to output_compression conversion
        if self.output_compression is None and self.output_quality is not None:
            self.output_compression = self._adjust_output_quality(
                self.output_quality, self.data_type
            )

        self._validate()

        # Allow env var to override num_inference_steps (for faster CI testing on AMD)
        env_steps = os.environ.get("SGLANG_TEST_NUM_INFERENCE_STEPS")
        if env_steps is not None and self.num_inference_steps is not None:
            self.num_inference_steps = int(env_steps)
```
**EN:** This block defines method `__post_init__` on `SamplingParams`. It post-processes init. Key calls include `self._validate`, `os.environ.get`, `self._adjust_output_quality`, and `int`. The implementation branches on conditions.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `__post_init__`。 它用于后处理init。 关键调用包括 `self._validate`、`os.environ.get`、`self._adjust_output_quality` 和 `int`。 实现中包含条件分支。

### Lines 290-299: `build_request_extra` implementation / `build_request_extra` 实现
```python
    def build_request_extra(self) -> dict[str, Any]:
        """Return optional request-scoped extras for downstream pipeline stages."""
        extra = {}
        diffusers_kwargs = getattr(self, "diffusers_kwargs", None)
        if diffusers_kwargs:
            extra["diffusers_kwargs"] = diffusers_kwargs
        explicit_fields = getattr(self, "_explicit_fields", None)
        if explicit_fields is not None:
            extra["explicit_fields"] = sorted(explicit_fields)
        return extra
```
**EN:** This block defines method `build_request_extra` on `SamplingParams`. Return optional request-scoped extras for downstream pipeline stages. Key calls include `getattr`, and `sorted`. The implementation branches on conditions.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `build_request_extra`。 它用于构建request extra。 关键调用包括 `getattr` 和 `sorted`。 实现中包含条件分支。

### Lines 301-303: `apply_request_extra` implementation / `apply_request_extra` 实现
```python
    def apply_request_extra(self, req: Any) -> None:
        """Merge request extras (model specific, e.g., LTX2.3) into an already-created pipeline request."""
        req.extra.update(self.build_request_extra())
```
**EN:** This block defines method `apply_request_extra` on `SamplingParams`. Merge request extras (model specific, e.g., LTX2.3) into an already-created pipeline request. Key calls include `req.extra.update`, and `self.build_request_extra`. Parameters such as `req` drive the behavior in this section.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `apply_request_extra`。 它用于应用request extra。 关键调用包括 `req.extra.update` 和 `self.build_request_extra`。 本段逻辑主要由 `req` 等参数驱动。

### Lines 305-310: `_adjust_output_quality` implementation / `_adjust_output_quality` 实现
```python
    def _adjust_output_quality(self, output_quality: str, data_type: DataType) -> int:
        """Convert output_quality string to compression level."""
        output_quality_mapper = {"maximum": 100, "high": 90, "medium": 55, "low": 35}
        if output_quality == "default":
            return 50 if data_type == DataType.VIDEO else 75
        return output_quality_mapper.get(output_quality)
```
**EN:** This block defines method `_adjust_output_quality` on `SamplingParams`. Convert output_quality string to compression level. Key calls include `output_quality_mapper.get`. The implementation branches on conditions. Parameters such as `output_quality`, and `data_type` drive the behavior in this section.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `_adjust_output_quality`。 它用于处理 adjust output quality 相关逻辑。 关键调用包括 `output_quality_mapper.get`。 实现中包含条件分支。 本段逻辑主要由 `output_quality` 和 `data_type` 等参数驱动。

### Lines 312-415: `_validate` implementation / `_validate` 实现
```python
    def _validate(self):
        """
        check if the sampling params is correct by itself
        """
        if self.prompt_path and not self.prompt_path.endswith(".txt"):
            raise ValueError(
                f"prompt_path must be a txt file, got {self.prompt_path!r}"
            )

        # These are always required to be sane regardless of pipeline.
        if (
            not isinstance(self.num_outputs_per_prompt, int)
            or self.num_outputs_per_prompt <= 0
        ):
            raise ValueError(
                f"num_outputs_per_prompt must be a positive int, got {self.num_outputs_per_prompt!r}"
            )

        if isinstance(self.seed, list):
            if not self.seed:
                raise ValueError("seed list must not be empty")
            for seed in self.seed:
                if isinstance(seed, bool) or not isinstance(seed, int) or seed < 0:
                    raise ValueError(
                        f"seed list must contain non-negative ints, got {self.seed!r}"
                    )
        elif (
            isinstance(self.seed, bool)
            or not isinstance(self.seed, int)
            or self.seed < 0
        ):
            raise ValueError(
                "seed must be a non-negative int or list of ints, " f"got {self.seed!r}"
            )

        # Used by seconds() and video writer; fps <= 0 is always invalid.
        if not isinstance(self.fps, int) or self.fps <= 0:
            raise ValueError(f"fps must be a positive int, got {self.fps!r}")

        # num_frames is already asserted in __post_init__, but keep a friendly error here too
        # (e.g., when validation is triggered from other code paths).
        if not isinstance(self.num_frames, int) or self.num_frames <= 0:
            raise ValueError(
                f"num_frames must be a positive int, got {self.num_frames!r}"
            )

        if self.num_inference_steps is not None:
            if (
                not isinstance(self.num_inference_steps, int)
                or self.num_inference_steps <= 0
            ):
                raise ValueError(
                    f"num_inference_steps must be a positive int, got {self.num_inference_steps!r}"
                )

        # Numeric hyperparams should not be NaN/Inf and should be within basic ranges.
        # Note: bool is a subclass of int; reject it explicitly to avoid silent surprises.
        def _finite_non_negative_float(
            name: str, value: Any, allow_none: bool = True
        ) -> None:
            if value is None and allow_none:
                return
            if isinstance(value, bool) or not isinstance(value, (int, float)):
                raise ValueError(f"{name} must be a number, got {value!r}")
            if not math.isfinite(float(value)):
                raise ValueError(f"{name} must be finite, got {value!r}")
            if float(value) < 0.0:
                raise ValueError(f"{name} must be non-negative, got {value!r}")

        _finite_non_negative_float(
            "guidance_scale", self.guidance_scale, allow_none=True
        )
        _finite_non_negative_float(
            "guidance_scale_2", self.guidance_scale_2, allow_none=True
        )
        _finite_non_negative_float(
            "true_cfg_scale", self.true_cfg_scale, allow_none=True
        )
        _finite_non_negative_float(
            "guidance_rescale", self.guidance_rescale, allow_none=False
        )

        if self.cfg_normalization is None:
            self.cfg_normalization = 0.0
        elif isinstance(self.cfg_normalization, bool):
            self.cfg_normalization = 1.0 if self.cfg_normalization else 0.0

        if self.boundary_ratio is not None:
            if isinstance(self.boundary_ratio, bool) or not isinstance(
                self.boundary_ratio, (int, float)
            ):
                raise ValueError(
                    f"boundary_ratio must be a number, got {self.boundary_ratio!r}"
                )
            if not math.isfinite(float(self.boundary_ratio)):
                raise ValueError(
                    f"boundary_ratio must be finite, got {self.boundary_ratio!r}"
                )
            if not (0.0 <= float(self.boundary_ratio) <= 1.0):
                raise ValueError(
                    f"boundary_ratio must be within [0, 1], got {self.boundary_ratio!r}"
                )

        RLRolloutArgs.validate_sampling_params(self)
```
**EN:** This block defines method `_validate` on `SamplingParams`. check if the sampling params is correct by itself Key calls include `isinstance`, `_finite_non_negative_float`, `RLRolloutArgs.validate_sampling_params`, `ValueError`, and `self.prompt_path.endswith`. The implementation branches on conditions, iterates over collections or steps.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `_validate`。 它用于校验函数。 关键调用包括 `isinstance`、`_finite_non_negative_float`、`RLRolloutArgs.validate_sampling_params`、`ValueError` 和 `self.prompt_path.endswith`。 实现中包含条件分支，会遍历集合或步骤。

### Lines 417-419: `check_sampling_param` implementation / `check_sampling_param` 实现
```python
    def check_sampling_param(self):
        # Keep backward-compatibility for old call sites.
        self._validate()
```
**EN:** This block defines method `check_sampling_param` on `SamplingParams`. It checks sampling param. Key calls include `self._validate`.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `check_sampling_param`。 它用于检查sampling param。 关键调用包括 `self._validate`。

### Lines 421-437: `_validate_with_pipeline_config` implementation / `_validate_with_pipeline_config` 实现
```python
    def _validate_with_pipeline_config(self, pipeline_config):
        """
        check if the sampling params is compatible and valid with server_args
        """
        if pipeline_config.task_type.requires_image_input():
            # requires image input
            if self.image_path is None:
                raise ValueError(
                    f"Served model with task type '{pipeline_config.task_type.name}' requires an 'image_path' input, but none was provided"
                )

        if not pipeline_config.task_type.accepts_image_input():
            # does not support image input
            if self.image_path is not None:
                raise ValueError(
                    f"input_reference is not supported for {pipeline_config.task_type.name} models."
                )
```
**EN:** This block defines method `_validate_with_pipeline_config` on `SamplingParams`. check if the sampling params is compatible and valid with server_args Key calls include `pipeline_config.task_type.requires_image_input`, `pipeline_config.task_type.accepts_image_input`, and `ValueError`. The implementation branches on conditions. Parameters such as `pipeline_config` drive the behavior in this section.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `_validate_with_pipeline_config`。 它用于校验with pipeline config。 关键调用包括 `pipeline_config.task_type.requires_image_input`、`pipeline_config.task_type.accepts_image_input` 和 `ValueError`。 实现中包含条件分支。 本段逻辑主要由 `pipeline_config` 等参数驱动。

### Lines 439-594: `_adjust` implementation / `_adjust` 实现
```python
    def _adjust(
        self,
        server_args,
    ):
        """
        final adjustment, called after merged with user params
        """
        expand_path_fields(self)

        # TODO: SamplingParams should not rely on ServerArgs
        pipeline_config = server_args.pipeline_config

        if self.guidance_scale is None:
            try:
                from sglang.multimodal_gen.configs.pipeline_configs.hunyuan3d import (
                    Hunyuan3D2PipelineConfig,
                )

                if isinstance(pipeline_config, Hunyuan3D2PipelineConfig):
                    self.guidance_scale = pipeline_config.guidance_scale
                else:
                    self.guidance_scale = 1.0
            except ImportError:
                self.guidance_scale = 1.0

        self.data_type = server_args.pipeline_config.task_type.data_type()

        if self.output_path is None:
            if server_args.output_path is not None:
                self.output_path = server_args.output_path
                logger.debug(
                    f"Overriding output_path with server configuration: {self.output_path}"
                )
            else:
                self.save_output = False

        # Process negative prompt
        if self.negative_prompt is not None and not self.negative_prompt.isspace():
            # avoid stripping default negative prompt: ' ' for qwen-image
            self.negative_prompt = self.negative_prompt.strip()

        # Validate dimensions
        if self.num_frames <= 0:
            raise ValueError(
                f"height, width, and num_frames must be positive integers, got "
                f"height={self.height}, width={self.width}, "
                f"num_frames={self.num_frames}"
            )

        # Validate resolution against pipeline-specific supported resolutions
        if self.height is None and self.width is None:
            if self.supported_resolutions is not None:
                self.width, self.height = self.supported_resolutions[0]
                logger.info(
                    f"Resolution unspecified, using default: {self.supported_resolutions[0]}"
                )

        if self.height is not None and self.width is not None:
            if self.supported_resolutions is not None:
                if (self.width, self.height) not in self.supported_resolutions:
                    supported_str = ", ".join(
                        [f"{w}x{h}" for w, h in self.supported_resolutions]
                    )
                    error_msg = (
                        f"Unsupported resolution: {self.width}x{self.height}, output quality may suffer. "
                        f"Supported resolutions: {supported_str}"
                    )
                    logger.warning(error_msg)

        pipeline_name_lower = server_args.pipeline_config.__class__.__name__.lower()

        if (
            "wan" in pipeline_name_lower
            or "helios" in pipeline_name_lower
            or "joy" in pipeline_name_lower
        ) and (self.enable_sequence_shard is None or self.enable_sequence_shard):
            self.enable_sequence_shard = True
            logger.debug("Automatically enabled enable_sequence_shard")
        else:
            self.enable_sequence_shard = False

        if self.enable_sequence_shard:
            self.adjust_frames = False
            logger.info(
                f"Sequence dimension shard is enabled, disabling frame adjustment for better performance"
            )

        if pipeline_config.task_type.is_image_gen():
            # settle num_frames
            if not server_args.pipeline_config.allow_set_num_frames():
                logger.debug(f"Setting `num_frames` to 1 for image generation model")
                self.num_frames = 1

        else:
            # mandatory frame adjusting logic, mod
            # NOTE: We must apply adjust_num_frames BEFORE the SP alignment logic below.
            # If we apply it after, adjust_num_frames might modify the frame count
            # and break the divisibility constraint (alignment) required by num_gpus.
            original_num_frames = self.num_frames
            self.num_frames = server_args.pipeline_config.adjust_num_frames(
                original_num_frames
            )
            logger.info(
                "Adjusting number of frames from %s to %s based on model",
                original_num_frames,
                self.num_frames,
            )

            if self.adjust_frames:
                # Adjust number of frames based on number of GPUs for video task
                use_temporal_scaling_frames = (
                    pipeline_config.vae_config.use_temporal_scaling_frames
                )
                num_frames = self.num_frames
                num_gpus = server_args.num_gpus
                temporal_scale_factor = (
                    pipeline_config.vae_config.arch_config.temporal_compression_ratio
                )

                if use_temporal_scaling_frames:
                    orig_latent_num_frames = (
                        num_frames - 1
                    ) // temporal_scale_factor + 1
                else:
                    orig_latent_num_frames = num_frames

                if orig_latent_num_frames % server_args.num_gpus != 0:
                    # Adjust latent frames to be divisible by number of GPUs
                    if self.num_frames_round_down:
                        # Ensure we have at least 1 batch per GPU
                        new_latent_num_frames = (
                            max(1, (orig_latent_num_frames // num_gpus)) * num_gpus
                        )
                    else:
                        new_latent_num_frames = (
                            math.ceil(orig_latent_num_frames / num_gpus) * num_gpus
                        )

                    if use_temporal_scaling_frames:
                        # Convert back to number of frames, ensuring num_frames-1 is a multiple of temporal_scale_factor
                        new_num_frames = (
                            new_latent_num_frames - 1
                        ) * temporal_scale_factor + 1
                    else:
                        new_num_frames = new_latent_num_frames

                    logger.info(
                        "Adjusting number of frames from %s to %s based on number of GPUs (%s)",
                        self.num_frames,
                        new_num_frames,
                        server_args.num_gpus,
                    )
                    self.num_frames = new_num_frames

        if not server_args.comfyui_mode:
            self._set_output_file_name()
```
**EN:** This block defines method `_adjust` on `SamplingParams`. final adjustment, called after merged with user params Key calls include `expand_path_fields`, `server_args.pipeline_config.task_type.data_type`, `server_args.pipeline_config.__class__.__name__.lower`, `pipeline_config.task_type.is_image_gen`, and `self.negative_prompt.strip`. The implementation branches on conditions, handles exceptional paths. Parameters such as `server_args` drive the behavior in this section.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `_adjust`。 它用于处理 adjust 相关逻辑。 关键调用包括 `expand_path_fields`、`server_args.pipeline_config.task_type.data_type`、`server_args.pipeline_config.__class__.__name__.lower`、`pipeline_config.task_type.is_image_gen` 和 `self.negative_prompt.strip`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `server_args` 等参数驱动。

### Lines 596-604: `from_pretrained` implementation / `from_pretrained` 实现
```python
    @classmethod
    def from_pretrained(cls, model_path: str, **kwargs) -> "SamplingParams":
        from sglang.multimodal_gen.registry import get_model_info

        backend = kwargs.pop("backend", None)
        model_id = kwargs.pop("model_id", None)
        model_info = get_model_info(model_path, backend=backend, model_id=model_id)
        sampling_params: SamplingParams = model_info.sampling_param_cls(**kwargs)
        return sampling_params
```
**EN:** This block defines method `from_pretrained` on `SamplingParams`. It constructs from pretrained. Key calls include `kwargs.pop`, `get_model_info`, and `model_info.sampling_param_cls`. Parameters such as `model_path` drive the behavior in this section.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `from_pretrained`。 它用于从…构造pretrained。 关键调用包括 `kwargs.pop`、`get_model_info` 和 `model_info.sampling_param_cls`。 本段逻辑主要由 `model_path` 等参数驱动。

### Lines 606-675: `from_user_sampling_params_args` implementation / `from_user_sampling_params_args` 实现
```python
    @staticmethod
    def from_user_sampling_params_args(
        model_path: str, server_args: "ServerArgs", *args, **kwargs
    ):
        pipeline_class_name = getattr(server_args, "pipeline_class_name", None)
        try:
            sampling_params = None
            if pipeline_class_name:
                from sglang.multimodal_gen.registry import get_pipeline_config_classes

                config_classes = get_pipeline_config_classes(pipeline_class_name)
                if config_classes is not None:
                    _, sampling_params_cls = config_classes
                    sampling_params = sampling_params_cls()

            if sampling_params is None:
                sampling_params = SamplingParams.from_pretrained(
                    model_path,
                    backend=server_args.backend,
                    model_id=server_args.model_id,
                )
        except (AttributeError, ValueError):
            # Handle safetensors files or other cases where model_index.json is not available
            # Use appropriate SamplingParams based on pipeline_class_name from registry
            if os.path.isfile(model_path) and model_path.endswith(".safetensors"):
                # Determine which sampling params to use based on pipeline_class_name
                from sglang.multimodal_gen.registry import get_pipeline_config_classes

                config_classes = (
                    get_pipeline_config_classes(pipeline_class_name)
                    if pipeline_class_name
                    else None
                )

                if config_classes is not None:
                    _, sampling_params_cls = config_classes
                    try:
                        sampling_params = sampling_params_cls()
                        logger.info(
                            f"Using {sampling_params_cls.__name__} for {pipeline_class_name} safetensors file (no model_index.json): %s",
                            model_path,
                        )
                    except Exception as import_error:
                        logger.warning(
                            f"Failed to instantiate {sampling_params_cls.__name__}: {import_error}. "
                            "Using default SamplingParams"
                        )
                        sampling_params = SamplingParams()
                else:
                    raise ValueError(
                        f"Could not get pipeline config classes for {pipeline_class_name}"
                    )
            else:
                # Re-raise if it's not a safetensors file issue
                raise

        user_kwargs = dict(kwargs)
        user_kwargs.pop("diffusers_kwargs", None)

        user_sampling_params = type(sampling_params)(*args, **user_kwargs)
        # TODO: refactor
        sampling_params._merge_with_user_params(
            user_sampling_params, explicit_fields=set(user_kwargs.keys())
        )
        sampling_params._explicit_fields = set(user_kwargs.keys())
        sampling_params._adjust(server_args)

        sampling_params._validate_with_pipeline_config(server_args.pipeline_config)

        return sampling_params
```
**EN:** This block defines method `from_user_sampling_params_args` on `SamplingParams`. It constructs from user sampling params args. Key calls include `getattr`, `dict`, `user_kwargs.pop`, `type`, and `sampling_params._merge_with_user_params`. The implementation branches on conditions, handles exceptional paths. Parameters such as `model_path`, and `server_args` drive the behavior in this section.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `from_user_sampling_params_args`。 它用于从…构造user sampling params args。 关键调用包括 `getattr`、`dict`、`user_kwargs.pop`、`type` 和 `sampling_params._merge_with_user_params`。 实现中包含条件分支，处理异常路径。 本段逻辑主要由 `model_path` 和 `server_args` 等参数驱动。

### Lines 677-678: `output_size_str` implementation / `output_size_str` 实现
```python
    def output_size_str(self) -> str:
        return f"{self.width}x{self.height}"
```
**EN:** This block defines method `output_size_str` on `SamplingParams`. It handles output size str logic.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `output_size_str`。 它用于处理 output size str 相关逻辑。

### Lines 680-681: `seconds` implementation / `seconds` 实现
```python
    def seconds(self) -> float:
        return self.num_frames / self.fps
```
**EN:** This block defines method `seconds` on `SamplingParams`. It handles seconds logic.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `seconds`。 它用于处理 seconds 相关逻辑。

### Lines 683-977: `add_cli_args` implementation / `add_cli_args` 实现
```python
    @staticmethod
    def add_cli_args(parser: Any) -> Any:
        """Add CLI arguments for SamplingParam fields"""

        def add_argument(*name_or_flags, **kwargs):
            kwargs.setdefault("default", argparse.SUPPRESS)
            return parser.add_argument(*name_or_flags, **kwargs)

        add_argument("--data-type", type=str, nargs="+")
        add_argument(
            "--num-frames-round-down",
            action="store_true",
        )
        add_argument(
            "--enable-teacache",
            action="store_true",
        )

        # profiling
        add_argument(
            "--profile",
            action="store_true",
            help="Enable torch profiler for denoising stage",
        )
        add_argument(
            "--num-profiled-timesteps",
            type=int,
            help="Number of timesteps to profile after warmup",
        )
        add_argument(
            "--profile-all-stages",
            action="store_true",
            dest="profile_all_stages",
            help="Used with --profile, profile all pipeline stages",
        )

        add_argument(
            "--debug",
            action="store_true",
            help="",
        )

        add_argument(
            "--prompt",
            type=str,
            nargs="+",
            help="Text prompt(s) for generation. Use space-separated values for multiple prompts, e.g., --prompt 'prompt 1' 'prompt 2'",
        )
        add_argument(
            "--negative-prompt",
            type=str,
            help="Negative text prompt for generation",
        )
        add_argument(
            "--prompt-path",
            type=str,
            help="Path to a text file containing prompts (one per line)",
        )
        add_argument(
            "--output-file-name",
            type=str,
            help="Name of the output file",
        )
        add_argument(
            "--output-quality",
            type=str,
            help="Output quality setting (default, low, medium, high, maximum)",
        )
        add_argument(
            "--output-compression",
            type=int,
            help="Output compression level (0-100, higher means better quality but larger file size)",
        )
        add_argument(
            "--num-outputs-per-prompt",
            type=int,
            help="Number of outputs to generate per prompt",
        )
        add_argument(
            "--seed",
            type=int,
            nargs="+",
            help="Random seed for generation",
        )
        add_argument(
            "--generator-device",
            type=str,
            choices=["cuda", "musa", "cpu"],
            help="Device for random generator (cuda, musa or cpu). Default: use the model-specific setting.",
        )
        add_argument(
            "--num-frames",
            type=int,
            help="Number of frames to generate",
        )
        add_argument(
            "--height",
            type=int,
            help="Height of generated output",
        )
        add_argument(
            "--width",
            type=int,
            help="Width of generated output",
        )
        # resolution shortcuts
        add_argument(
            "--4k",
            action="store_true",
            dest="resolution_4k",
            help="Set resolution to 4K (3840x2160)",
        )
        add_argument(
            "--2k",
            action="store_true",
            dest="resolution_2k",
            help="Set resolution to 2K (2560x1440)",
        )
        add_argument(
            "--1080p",
            action="store_true",
            dest="resolution_1080p",
            help="Set resolution to 1080p (1920x1080)",
        )
        add_argument(
            "--720p",
            action="store_true",
            dest="resolution_720p",
            help="Set resolution to 720p (1280x720)",
        )

        add_argument(
            "--fps",
            type=int,
            help="Frames per second for saved output",
        )
        add_argument(
            "--num-inference-steps",
            type=int,
            help="Number of denoising steps",
        )
        add_argument(
            "--guidance-scale",
            type=float,
            help="Classifier-free guidance scale",
        )
        add_argument(
            "--guidance-scale-2",
            type=float,
            dest="guidance_scale_2",
            help="Secondary guidance scale for dual-guidance models (e.g., Wan low-noise expert)",
        )
        add_argument(
            "--true-cfg-scale",
            type=float,
            dest="true_cfg_scale",
            help="True CFG scale for models that distinguish distilled guidance from standard CFG (e.g., Qwen-Image)",
        )
        add_argument(
            "--guidance-rescale",
            type=float,
            help="Guidance rescale factor",
        )
        add_argument(
            "--cfg-normalization",
            type=float,
            dest="cfg_normalization",
            help="CFG renormalization factor (for Z-Image). ",
        )
        add_argument(
            "--boundary-ratio",
            type=float,
            help="Boundary timestep ratio",
        )
        add_argument(
            "--save-output",
            action="store_true",
            help="Whether to save the output to disk",
        )
        add_argument(
            "--no-save-output",
            action="store_false",
            dest="save_output",
            help="Don't save the output to disk",
        )
        add_argument(
            "--return-frames",
            action="store_true",
            help="Whether to return the raw frames",
        )
        add_argument(
            "--image-path",
            type=str,
            nargs="+",
            help=(
                "Path(s) to input image(s) for image-to-image / image-to-video "
                "generation. For multiple images, pass them as space-separated "
                "values, e.g.: "
                '--image-path "img1.png" "img2.png"'
            ),
        )
        add_argument(
            "--moba-config-path",
            type=str,
            help="Path to a JSON file containing V-MoBA specific configurations.",
        )
        add_argument(
            "--return-trajectory-latents",
            action="store_true",
            help="Whether to return the trajectory",
        )

        # Rollout arguments
        RLRolloutArgs.add_cli_args(parser, add_argument=add_argument)

        add_argument(
            "--return-trajectory-decoded",
            action="store_true",
            help="Whether to return the decoded trajectory",
        )
        add_argument(
            "--diffusers-kwargs",
            type=str,
            help="JSON string of extra kwargs to pass to diffusers pipeline. "
            'Example: \'{"output_type": "latent", "clip_skip": 2}\'',
        )
        add_argument(
            "--no-override-protected-fields",
            action="store_true",
            help=(
                "If set, disallow user params to override fields defined in subclasses."
            ),
        )
        add_argument(
            "--adjust-frames",
            action=StoreBoolean,
            help=(
                "Enable/disable adjusting num_frames to evenly split latent frames across GPUs "
                "and satisfy model temporal constraints. If disabled, tokens might be padded for SP."
                "Default: true. Examples: --adjust-frames, --adjust-frames true, --adjust-frames false."
            ),
        )
        add_argument(
            "--return-file-paths-only",
            action=StoreBoolean,
            help="If set, output file will be saved early to get a performance boost, while output tensors will not be returned.",
        )
        add_argument(
            "--enable-sequence-shard",
            action=StoreBoolean,
            help="Enable sequence dimension shard with sequence parallelism.",
        )
        add_argument(
            "--enable-frame-interpolation",
            action="store_true",
            help="Enable post-generation frame interpolation using RIFE 4.22.lite.",
        )
        add_argument(
            "--frame-interpolation-exp",
            type=int,
            help="Frame interpolation exponent: 1=2x, 2=4x (default: 1).",
        )
        add_argument(
            "--frame-interpolation-scale",
            type=float,
            help="RIFE inference scale factor (default: 1.0; use 0.5 for high-res).",
        )
        add_argument(
            "--frame-interpolation-model-path",
            type=str,
            help="Local directory or HuggingFace repo ID containing RIFE flownet.pkl weights "
            "(default: elfgum/RIFE-4.22.lite, downloaded automatically). "
            "Only RIFE 4.22.lite architecture is supported; other RIFE versions or "
            "frame interpolation models are not compatible.",
        )
        add_argument(
            "--enable-upscaling",
            action="store_true",
            help="Enable post-generation upscaling using Real-ESRGAN.",
        )
        add_argument(
            "--upscaling-model-path",
            type=str,
            help="Local .pth file, HuggingFace repo ID, or repo_id:filename for Real-ESRGAN weights "
            "(default: ai-forever/Real-ESRGAN with RealESRGAN_x4.pth). "
            "Only RRDBNet (e.g. RealESRGAN_x4plus) and SRVGGNetCompact (e.g. realesr-animevideov3) "
            "architectures are supported; other super-resolution models are not compatible. "
            "Use 'repo_id:filename' to specify a custom weight file from a HF repo.",
        )
        add_argument(
            "--upscaling-scale",
            type=int,
            help="Upscaling factor (default: 4).",
        )
        return parser
```
**EN:** This block defines method `add_cli_args` on `SamplingParams`. Add CLI arguments for SamplingParam fields Key calls include `add_argument`, `RLRolloutArgs.add_cli_args`, `kwargs.setdefault`, and `parser.add_argument`. Parameters such as `parser` drive the behavior in this section.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `add_cli_args`。 它用于处理 add cli args 相关逻辑。 关键调用包括 `add_argument`、`RLRolloutArgs.add_cli_args`、`kwargs.setdefault` 和 `parser.add_argument`。 本段逻辑主要由 `parser` 等参数驱动。

### Lines 979-1005: `get_cli_args` implementation / `get_cli_args` 实现
```python
    @classmethod
    def get_cli_args(cls, args: argparse.Namespace):
        # handle resolution shortcuts
        if hasattr(args, "resolution_4k") and args.resolution_4k:
            args.width = 3840
            args.height = 2160
        elif hasattr(args, "resolution_2k") and args.resolution_2k:
            args.width = 2560
            args.height = 1440
        elif hasattr(args, "resolution_1080p") and args.resolution_1080p:
            args.width = 1920
            args.height = 1080
        elif hasattr(args, "resolution_720p") and args.resolution_720p:
            args.width = 1280
            args.height = 720

        sampling_params_fields = {attr.name for attr in dataclasses.fields(cls)}
        args_attrs = set(vars(args).keys())
        attrs = sampling_params_fields & args_attrs
        cli_args = {
            attr: getattr(args, attr)
            for attr in attrs
            if hasattr(args, attr) and getattr(args, attr) is not None
        }
        if isinstance(cli_args.get("seed"), list) and len(cli_args["seed"]) == 1:
            cli_args["seed"] = cli_args["seed"][0]
        return cli_args
```
**EN:** This block defines method `get_cli_args` on `SamplingParams`. It retrieves cli args. Key calls include `set`, `hasattr`, `vars.keys`, `getattr`, and `isinstance`. The implementation branches on conditions. Parameters such as `args` drive the behavior in this section.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `get_cli_args`。 它用于获取cli args。 关键调用包括 `set`、`hasattr`、`vars.keys`、`getattr` 和 `isinstance`。 实现中包含条件分支。 本段逻辑主要由 `args` 等参数驱动。

### Lines 1007-1010: `output_file_path` implementation / `output_file_path` 实现
```python
    def output_file_path(self):
        if self.output_path is None:
            return None
        return os.path.join(self.output_path, self.output_file_name)
```
**EN:** This block defines method `output_file_path` on `SamplingParams`. It handles output file path logic. Key calls include `os.path.join`. The implementation branches on conditions.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `output_file_path`。 它用于处理 output file path 相关逻辑。 关键调用包括 `os.path.join`。 实现中包含条件分支。

### Lines 1012-1052: `_merge_with_user_params` implementation / `_merge_with_user_params` 实现
```python
    def _merge_with_user_params(
        self,
        user_params: "SamplingParams",
        explicit_fields: set[str] | None = None,
    ):
        """
        Merges parameters from a user-provided SamplingParams object.

        Args:
            explicit_fields: field names explicitly set by the user (e.g. from
                CLI kwargs). These are always treated as user-modified even when
                their value matches the base-class default.
        """
        if user_params is None:
            return

        predefined_fields = set(type(self).__annotations__.keys())

        # global switch: if True, allow overriding protected fields
        allow_override_protected = not user_params.no_override_protected_fields
        for field in dataclasses.fields(user_params):
            field_name = field.name
            user_value = getattr(user_params, field_name)
            if hasattr(SamplingParams, field_name):
                default_class_value = getattr(SamplingParams, field_name)
            elif field.default is not dataclasses.MISSING:
                default_class_value = field.default
            elif field.default_factory is not dataclasses.MISSING:
                default_class_value = field.default_factory()
            else:
                default_class_value = dataclasses.MISSING

            is_user_modified = user_value != default_class_value or (
                explicit_fields is not None and field_name in explicit_fields
            )
            is_protected_field = field_name in predefined_fields
            if is_user_modified and (
                allow_override_protected or not is_protected_field
            ):
                setattr(self, field_name, user_value)
        self.__post_init__()
```
**EN:** This block defines method `_merge_with_user_params` on `SamplingParams`. Merges parameters from a user-provided SamplingParams object. Args: explicit_fields: field names explicitly set by the user (e.g. Key calls include `set`, `dataclasses.fields`, `self.__post_init__`, `type.__annotations__.keys`, and `getattr`. The implementation branches on conditions, iterates over collections or steps. Parameters such as `user_params`, and `explicit_fields` drive the behavior in this section.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `_merge_with_user_params`。 它用于合并with user params。 关键调用包括 `set`、`dataclasses.fields`、`self.__post_init__`、`type.__annotations__.keys` 和 `getattr`。 实现中包含条件分支，会遍历集合或步骤。 本段逻辑主要由 `user_params` 和 `explicit_fields` 等参数驱动。

### Lines 1054-1066: `n_tokens` implementation / `n_tokens` 实现
```python
    @property
    def n_tokens(self) -> int:
        # Calculate latent sizes
        if self.height and self.width:
            latents_size = [
                (self.num_frames - 1) // 4 + 1,
                self.height // 8,
                self.width // 8,
            ]
            n_tokens = latents_size[0] * latents_size[1] * latents_size[2]
        else:
            n_tokens = -1
        return n_tokens
```
**EN:** This block defines method `n_tokens` on `SamplingParams`. It handles n tokens logic. The implementation branches on conditions.
**CN:** 该代码块定义了 `SamplingParams` 的方法 `n_tokens`。 它用于处理 n tokens 相关逻辑。 实现中包含条件分支。

### Lines 1070-1070: `CacheParams` class overview / `CacheParams` 类概览
```python
class CacheParams:
```
**EN:** This block defines class `CacheParams`. It encapsulates cache params behavior.
**CN:** 该代码块定义了类 `CacheParams`。 它用于封装 cache params 相关行为。

### Lines 1071-1071: supporting statements / 辅助语句
```python
    cache_type: str = "none"
```
**EN:** This block gathers supporting statements inside `CacheParams`. It updates names such as `cache_type`.
**CN:** 该代码块汇集了位于 `CacheParams` 内部的辅助语句。 它会更新 `cache_type` 等名称。

## Key Concepts / 关键概念
- `_json_safe`: Recursively convert objects to JSON-serializable forms. / 顶层函数，用于处理 json safe 相关逻辑。
- `generate_request_id`: Top-level function that generates request id. / 顶层函数，用于生成request id。
- `_sanitize_filename`: Create a filesystem- and ffmpeg-friendly filename. / 顶层函数，用于处理 sanitize filename 相关逻辑。
- `DataType`: Primary class that encapsulates data type behavior. / 核心类，用于封装 data type 相关行为。
- `SamplingParams`: Sampling parameters for generation. / 核心类，用于封装 sampling params 相关行为。
- `CacheParams`: Primary class that encapsulates cache params behavior. / 核心类，用于封装 cache params 相关行为。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `argparse`, `dataclasses`, `hashlib`, `json`, `math`, `os`, `os.path`, `re`, `time`, `unicodedata`
- **Internal modules / 内部模块**: `sglang.multimodal_gen.configs.post_training`, `sglang.multimodal_gen.runtime.utils.logging_utils`, `sglang.multimodal_gen.utils`, `sglang.multimodal_gen.runtime.server_args`, `sglang.multimodal_gen.registry`, `sglang.multimodal_gen.configs.pipeline_configs.hunyuan3d`

- **Total lines / 总行数**: 1071
