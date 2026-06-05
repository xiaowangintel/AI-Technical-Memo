# processor.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/transformers_utils/processor.py`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Provides the central processor loading and adaptation helpers that connect Hugging Face processors to vLLM multimodal inputs. / [CN] 提供核心处理器加载与适配逻辑，用于把 Hugging Face 处理器接入 vLLM 的多模态输入流程。

## Line-by-Line Analysis / 逐行分析

### Lines 4-31: Imports
```python
import importlib
import inspect
from functools import lru_cache
from typing import TYPE_CHECKING, Any, cast, get_args, get_type_hints

from transformers import (
    AutoFeatureExtractor,
    AutoImageProcessor,
    AutoProcessor,
    AutoVideoProcessor,
    BatchFeature,
    processing_utils,
)
from transformers.audio_utils import AudioInput
from transformers.feature_extraction_utils import FeatureExtractionMixin
from transformers.image_processing_utils import BaseImageProcessor
from transformers.image_utils import ImageInput
from transformers.processing_utils import ProcessorMixin
from transformers.video_processing_utils import BaseVideoProcessor
from transformers.video_utils import VideoInput
from typing_extensions import TypeVar

from vllm.logger import init_logger
from vllm.transformers_utils import processors
from vllm.transformers_utils.gguf_utils import is_gguf
from vllm.transformers_utils.repo_utils import get_hf_file_to_dict
from vllm.transformers_utils.utils import convert_model_repo_to_path
from vllm.utils.func_utils import get_allowed_kwarg_only_overrides
```
**EN:** This import block assembles the module's dependencies: standard-library helpers such as `importlib`, `inspect`, `functools`, `typing`, external APIs such as `transformers`, `transformers.audio_utils`, `transformers.feature_extraction_utils`, `transformers.image_processing_utils`, `transformers.image_utils`, `transformers.processing_utils`, `transformers.video_processing_utils`, `transformers.video_utils`, ... (+1 more), and internal vLLM modules such as `vllm.logger`, `vllm.transformers_utils`, `vllm.transformers_utils.gguf_utils`, `vllm.transformers_utils.repo_utils`, `vllm.transformers_utils.utils`, `vllm.utils.func_utils`. Together they define the services the rest of the file builds on.
**CN:** 该导入块汇集了模块依赖：标准库工具如 `importlib`, `inspect`, `functools`, `typing`，外部 API 如 `transformers`, `transformers.audio_utils`, `transformers.feature_extraction_utils`, `transformers.image_processing_utils`, `transformers.image_utils`, `transformers.processing_utils`, `transformers.video_processing_utils`, `transformers.video_utils`, ... (+1 more)，以及 vLLM 内部模块如 `vllm.logger`, `vllm.transformers_utils`, `vllm.transformers_utils.gguf_utils`, `vllm.transformers_utils.repo_utils`, `vllm.transformers_utils.utils`, `vllm.utils.func_utils`。这些依赖共同构成了后续实现所依赖的基础能力。

### Lines 33-33: Module state and constants
```python
logger = init_logger(__name__)
```
**EN:** This block initializes module-level state such as `logger`. The assigned names act as shared context for the rest of the file.
**CN:** 该代码块初始化了模块级状态，例如 `logger`。这些名称为文件其余部分提供共享上下文。

### Lines 35-36: Conditional block
```python
if TYPE_CHECKING:
    from vllm.config import ModelConfig
```
**EN:** This conditional block restricts imports or declarations to static type checking, avoiding unnecessary runtime dependencies.
**CN:** 该条件分支仅在静态类型检查时启用导入或声明，从而避免额外的运行时依赖。

### Lines 53-81: Function `_transformers_v4_compatibility_init`
```python
def _transformers_v4_compatibility_init() -> Any:
    """Some remote code processors may define `optional_attributes` in their
    `ProcessorMixin` subclass, and then pass these arbitrary attributes directly to
    `ProcessorMixin.__init__`, which is no longer allowed in Transformers v5. For
    backward compatibility, we intercept these optional attributes and set them on the
    processor instance before calling the original `ProcessorMixin.__init__`.

    This can be removed if `Molmo2ForConditionalGeneration` is upstreamed to
    Transformers."""
    # Transformers v4
    if hasattr(ProcessorMixin, "optional_attributes"):
        return
    # Transformers v5
    if hasattr(ProcessorMixin.__init__, "_vllm_patched"):
        return

    original_init = ProcessorMixin.__init__

    def __init__(self, *args, **kwargs):
        for optional_attribute in getattr(self, "optional_attributes", []):
            if optional_attribute in kwargs:
                setattr(self, optional_attribute, kwargs.pop(optional_attribute))

        original_init(self, *args, **kwargs)

    # Only patch if ProcessorMixin is not mocked (for docs builds)
    if not hasattr(ProcessorMixin, "_mock_name"):
        __init__._vllm_patched = True  # type: ignore[attr-defined]
        ProcessorMixin.__init__ = __init__
```
**EN:** This private function implements `_transformers_v4_compatibility_init`. The docstring states that Some remote code processors may define `optional_attributes` in their
**CN:** 该私有函数实现 `_transformers_v4_compatibility_init` 相关逻辑。 文档字符串进一步说明了它的输入与行为。

### Lines 87-88: Module state and constants
```python
_P = TypeVar("_P", bound=ProcessorMixin, default=ProcessorMixin)
_V = TypeVar("_V", bound=BaseVideoProcessor, default=BaseVideoProcessor)
```
**EN:** This block defines module-level constants/defaults such as `_P`, `_V`. They centralize shared parameters for later helpers and classes.
**CN:** 该代码块定义了模块级常量或默认值，例如 `_P`, `_V`。它们把后续辅助函数和类会复用的参数集中到一起。

### Lines 91-99: Class `HashableDict`
```python
class HashableDict(dict):
    """
    A dictionary that can be hashed by lru_cache.
    """

    # NOTE: pythonic dict is not hashable,
    # we override on it directly for simplicity
    def __hash__(self) -> int:  # type: ignore[override]
        return hash(frozenset(self.items()))
```
**EN:** Defines class `HashableDict` derived from `dict`. The class docstring highlights that A dictionary that can be hashed by lru_cache. Key methods include `__hash__`.
**CN:** 定义类 `HashableDict`，继承自 `dict`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__hash__`。

### Lines 102-108: Class `HashableList`
```python
class HashableList(list):
    """
    A list that can be hashed by lru_cache.
    """

    def __hash__(self) -> int:  # type: ignore[override]
        return hash(tuple(self))
```
**EN:** Defines class `HashableList` derived from `list`. The class docstring highlights that A list that can be hashed by lru_cache. Key methods include `__hash__`.
**CN:** 定义类 `HashableList`，继承自 `list`。 类文档字符串进一步说明了它的职责。 关键方法包括 `__hash__`。

### Lines 148-161: Function `get_processor_cls_name_from_config`
```python
def get_processor_cls_name_from_config(
    processor_name: str,
    revision: str | None = "main",
) -> str | None:
    config_file = [
        "processor_config.json",
        "preprocessor_config.json",
        "tokenizer_config.json",
    ]
    for file in config_file:
        config = get_hf_file_to_dict(file, processor_name, revision=revision)
        if config and "processor_class" in config:
            return config["processor_class"]
    return None
```
**EN:** This function retrieves processor cls name from config. Main inputs include `processor_name`, `revision`.
**CN:** 该函数负责完成 `get_processor_cls_name_from_config` 对应的核心步骤。 主要输入参数包括 `processor_name`, `revision`。

### Lines 164-231: Function `get_processor`
```python
def get_processor(
    processor_name: str,
    *args: Any,
    revision: str | None = None,
    trust_remote_code: bool = False,
    processor_cls: type[_P] | tuple[type[_P], ...] = ProcessorMixin,
    **kwargs: Any,
) -> _P:
    """Load a processor for the given model name via HuggingFace."""
    if revision is None:
        revision = "main"
    try:
        processor_name = convert_model_repo_to_path(processor_name)
        registered_cls_name = get_processor_cls_name_from_config(
            processor_name, revision=revision
        )
        registered_processor_cls = (
            getattr(processors, registered_cls_name, None)
            if registered_cls_name
            else None
        )
        registered_processor_cls = cast(type[_P] | None, registered_processor_cls)
        # Use registered processor class when it's available
        # and explicit processor_cls is not set.
        if isinstance(processor_cls, tuple) or processor_cls == ProcessorMixin:
            _processor_cls = registered_processor_cls or AutoProcessor
            processor = _processor_cls.from_pretrained(
                processor_name,
                *args,
                revision=revision,
                trust_remote_code=trust_remote_code,
                **kwargs,
            )
        elif issubclass(processor_cls, ProcessorMixin):
            processor = processor_cls.from_pretrained(
                processor_name,
                *args,
                revision=revision,
                trust_remote_code=trust_remote_code,
                **kwargs,
# ... omitted for brevity ...
            raise RuntimeError(err_msg) from e
        else:
            raise e

    if not isinstance(processor, processor_cls):
        raise TypeError(
            "Invalid type of HuggingFace processor. "
            f"Expected type: {processor_cls}, but "
            f"found type: {type(processor)}"
        )

    return processor
```
**EN:** This function retrieves processor. The docstring states that Load a processor for the given model name via HuggingFace. Main inputs include `processor_name`, `revision`, `trust_remote_code`, `processor_cls`.
**CN:** 该函数负责完成 `get_processor` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `processor_name`, `revision`, `trust_remote_code`, `processor_cls`。

### Lines 234-234: Module state and constants
```python
cached_get_processor = lru_cache(get_processor)
```
**EN:** This block initializes module-level state such as `cached_get_processor`. The assigned names act as shared context for the rest of the file.
**CN:** 该代码块初始化了模块级状态，例如 `cached_get_processor`。这些名称为文件其余部分提供共享上下文。

### Lines 238-266: Function `get_processor_kwargs_type`
```python
def get_processor_kwargs_type(
    processor: ProcessorMixin,
) -> type[processing_utils.ProcessingKwargs]:
    try:
        # get kwargs annotations in processor
        call_params = inspect.signature(type(processor).__call__).parameters
        call_kwargs = call_params.get("kwargs")
        call_kwargs_annotations = call_kwargs.annotation if call_kwargs else None

        # if the processor has explicit kwargs annotation, use it
        if call_kwargs_annotations not in (None, inspect._empty):  # noqa: SIM102
            # get_type_hints will parse all type annotations at runtime,
            # and if an annotation refers to a type or
            # name that hasn’t been imported or defined, it will raise an error.
            # So we use __annotations__ to get the raw annotations directly.
            if anno_args := get_args(call_kwargs_annotations):
                return anno_args[0]

        # otherwise, try to get from ProcessorKwargs
        module_name = type(processor).__module__
        mod = importlib.import_module(module_name)
        for name, obj in vars(mod).items():
            if name.endswith("ProcessorKwargs"):
                return obj

    except Exception:
        logger.exception("Failed to collect processor kwargs")

    return processing_utils.ProcessingKwargs
```
**EN:** This function retrieves processor kwargs type. Main inputs include `processor`. Decorators such as `lru_cache` modify caching, validation, or dispatch behavior.
**CN:** 该函数负责完成 `get_processor_kwargs_type` 对应的核心步骤。 主要输入参数包括 `processor`。 装饰器如 `lru_cache` 会影响缓存、校验或分发行为。

### Lines 270-300: Function `get_processor_kwargs_keys`
```python
def get_processor_kwargs_keys(
    kwargs_cls: type[processing_utils.ProcessingKwargs],
) -> set[str]:
    dynamic_kwargs: set[str] = set()
    modality_kwargs = {
        "text_kwargs",
        "images_kwargs",
        "videos_kwargs",
        "audio_kwargs",
    }

    try:
        # get kwargs annotations in processor
        # merge text_kwargs / images_kwargs / videos_kwargs / audio_kwargs
        kwargs_type_annotations = get_type_hints(kwargs_cls)
        for kw_type in modality_kwargs:
            if kw_type in kwargs_type_annotations:
                # Use __annotations__ instead of get_type_hints() to avoid
                # NameError from unresolved forward references (e.g.
                # PILImageResampling). We only need key names, not types.
                kw_cls = kwargs_type_annotations[kw_type]
                kw_annotations: dict[str, Any] = {}
                for base in reversed(kw_cls.__mro__):
                    kw_annotations.update(getattr(base, "__annotations__", {}))
                for kw_name in kw_annotations:
                    dynamic_kwargs.add(kw_name)

    except Exception:
        logger.exception("Failed to collect processor kwargs")

    return dynamic_kwargs | modality_kwargs
```
**EN:** This function retrieves processor kwargs keys. Main inputs include `kwargs_cls`. Decorators such as `lru_cache` modify caching, validation, or dispatch behavior.
**CN:** 该函数负责完成 `get_processor_kwargs_keys` 对应的核心步骤。 主要输入参数包括 `kwargs_cls`。 装饰器如 `lru_cache` 会影响缓存、校验或分发行为。

### Lines 303-336: Function `cached_get_processor_without_dynamic_kwargs`
```python
def cached_get_processor_without_dynamic_kwargs(
    processor_name: str,
    *args: Any,
    revision: str | None = None,
    trust_remote_code: bool = False,
    processor_cls: type[_P] | tuple[type[_P], ...] = ProcessorMixin,
    **kwargs: Any,
) -> _P:
    # Step 1: use default kwargs to get a temporary processor instance
    processor = cached_get_processor(
        processor_name,
        revision=revision,
        trust_remote_code=trust_remote_code,
        processor_cls=processor_cls,  # type: ignore[arg-type]
    )

    # Step 2: use temporary processor collect dynamic keys
    dynamic_keys = get_processor_kwargs_keys(
        get_processor_kwargs_type(processor)  # type: ignore[arg-type]
    )

    # Step 3: use dynamic_keys filter kwargs
    filtered_kwargs = {k: v for k, v in kwargs.items() if k not in dynamic_keys}

    # Step 4: use filtered kwargs to get final processor instance
    final_processor = cached_get_processor(
        processor_name,
        revision=revision,
        trust_remote_code=trust_remote_code,
        processor_cls=processor_cls,  # type: ignore[arg-type]
        **filtered_kwargs,
    )

    return final_processor
```
**EN:** This function implements `cached_get_processor_without_dynamic_kwargs`. Main inputs include `processor_name`, `revision`, `trust_remote_code`, `processor_cls`.
**CN:** 该函数实现 `cached_get_processor_without_dynamic_kwargs` 相关逻辑。 主要输入参数包括 `processor_name`, `revision`, `trust_remote_code`, `processor_cls`。

### Lines 339-361: Function `cached_processor_from_config`
```python
def cached_processor_from_config(
    model_config: "ModelConfig",
    processor_cls: type[_P] | tuple[type[_P], ...] = ProcessorMixin,
    **kwargs: Any,
) -> _P:
    if is_gguf(model_config.model):
        assert not is_gguf(model_config.tokenizer), (
            "For multimodal GGUF models, the original tokenizer "
            "should be used to correctly load processor."
        )
        model = model_config.tokenizer
        revision = model_config.tokenizer_revision
    else:
        model = model_config.model
        revision = model_config.revision

    return cached_get_processor_without_dynamic_kwargs(
        model,
        revision=revision,
        trust_remote_code=model_config.trust_remote_code,
        processor_cls=processor_cls,  # type: ignore[arg-type]
        **_merge_mm_kwargs(model_config, processor_cls, **kwargs),
    )
```
**EN:** This function implements `cached_processor_from_config`. Main inputs include `model_config`, `processor_cls`.
**CN:** 该函数实现 `cached_processor_from_config` 相关逻辑。 主要输入参数包括 `model_config`, `processor_cls`。

### Lines 364-397: Function `get_feature_extractor`
```python
def get_feature_extractor(
    processor_name: str,
    *args: Any,
    revision: str | None = None,
    trust_remote_code: bool = False,
    **kwargs: Any,
):
    """Load an audio feature extractor for the given model name
    via HuggingFace."""
    try:
        processor_name = convert_model_repo_to_path(processor_name)
        feature_extractor = AutoFeatureExtractor.from_pretrained(
            processor_name,
            *args,
            revision=revision,
            trust_remote_code=trust_remote_code,
            **kwargs,
        )
    except ValueError as e:
        # If the error pertains to the processor class not existing or not
        # currently being imported, suggest using the --trust-remote-code flag.
        # Unlike AutoTokenizer, AutoImageProcessor does not separate such errors
        if not trust_remote_code:
            err_msg = (
                "Failed to load the feature extractor. If the feature "
                "extractor is a custom extractor not yet available in the "
                "HuggingFace transformers library, consider setting "
                "`trust_remote_code=True` in LLM or using the "
                "`--trust-remote-code` flag in the CLI."
            )
            raise RuntimeError(err_msg) from e
        else:
            raise e
    return cast(FeatureExtractionMixin, feature_extractor)
```
**EN:** This function retrieves feature extractor. The docstring states that Load an audio feature extractor for the given model name Main inputs include `processor_name`, `revision`, `trust_remote_code`.
**CN:** 该函数负责完成 `get_feature_extractor` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `processor_name`, `revision`, `trust_remote_code`。

### Lines 400-400: Module state and constants
```python
cached_get_feature_extractor = lru_cache(get_feature_extractor)
```
**EN:** This block initializes module-level state such as `cached_get_feature_extractor`. The assigned names act as shared context for the rest of the file.
**CN:** 该代码块初始化了模块级状态，例如 `cached_get_feature_extractor`。这些名称为文件其余部分提供共享上下文。

### Lines 403-412: Function `cached_feature_extractor_from_config`
```python
def cached_feature_extractor_from_config(
    model_config: "ModelConfig",
    **kwargs: Any,
):
    return cached_get_feature_extractor(
        model_config.model,
        revision=model_config.revision,
        trust_remote_code=model_config.trust_remote_code,
        **_merge_mm_kwargs(model_config, AutoFeatureExtractor, **kwargs),
    )
```
**EN:** This function implements `cached_feature_extractor_from_config`. Main inputs include `model_config`.
**CN:** 该函数实现 `cached_feature_extractor_from_config` 相关逻辑。 主要输入参数包括 `model_config`。

### Lines 415-448: Function `get_image_processor`
```python
def get_image_processor(
    processor_name: str,
    *args: Any,
    revision: str | None = None,
    trust_remote_code: bool = False,
    **kwargs: Any,
):
    """Load an image processor for the given model name via HuggingFace."""
    try:
        processor_name = convert_model_repo_to_path(processor_name)
        processor = AutoImageProcessor.from_pretrained(
            processor_name,
            *args,
            revision=revision,
            trust_remote_code=trust_remote_code,
            **kwargs,
        )
    except ValueError as e:
        # If the error pertains to the processor class not existing or not
        # currently being imported, suggest using the --trust-remote-code flag.
        # Unlike AutoTokenizer, AutoImageProcessor does not separate such errors
        if not trust_remote_code:
            err_msg = (
                "Failed to load the image processor. If the image processor is "
                "a custom processor not yet available in the HuggingFace "
                "transformers library, consider setting "
                "`trust_remote_code=True` in LLM or using the "
                "`--trust-remote-code` flag in the CLI."
            )
            raise RuntimeError(err_msg) from e
        else:
            raise e

    return cast(BaseImageProcessor, processor)
```
**EN:** This function retrieves image processor. The docstring states that Load an image processor for the given model name via HuggingFace. Main inputs include `processor_name`, `revision`, `trust_remote_code`.
**CN:** 该函数负责完成 `get_image_processor` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `processor_name`, `revision`, `trust_remote_code`。

### Lines 451-451: Module state and constants
```python
cached_get_image_processor = lru_cache(get_image_processor)
```
**EN:** This block initializes module-level state such as `cached_get_image_processor`. The assigned names act as shared context for the rest of the file.
**CN:** 该代码块初始化了模块级状态，例如 `cached_get_image_processor`。这些名称为文件其余部分提供共享上下文。

### Lines 454-473: Function `cached_image_processor_from_config`
```python
def cached_image_processor_from_config(
    model_config: "ModelConfig",
    **kwargs: Any,
):
    if is_gguf(model_config.model):
        assert not is_gguf(model_config.tokenizer), (
            "For multimodal GGUF models, the original tokenizer "
            "should be used to correctly load image processor."
        )
        model = model_config.tokenizer
        revision = model_config.tokenizer_revision
    else:
        model = model_config.model
        revision = model_config.revision
    return cached_get_image_processor(
        model,
        revision=revision,
        trust_remote_code=model_config.trust_remote_code,
        **_merge_mm_kwargs(model_config, AutoImageProcessor, **kwargs),
    )
```
**EN:** This function implements `cached_image_processor_from_config`. Main inputs include `model_config`.
**CN:** 该函数实现 `cached_image_processor_from_config` 相关逻辑。 主要输入参数包括 `model_config`。

### Lines 476-511: Function `get_video_processor`
```python
def get_video_processor(
    processor_name: str,
    *args: Any,
    revision: str | None = None,
    trust_remote_code: bool = False,
    processor_cls_overrides: type[_V] | None = None,
    **kwargs: Any,
):
    """Load a video processor for the given model name via HuggingFace."""
    try:
        processor_name = convert_model_repo_to_path(processor_name)
        processor_cls = processor_cls_overrides or AutoVideoProcessor
        processor = processor_cls.from_pretrained(
            processor_name,
            *args,
            revision=revision,
            trust_remote_code=trust_remote_code,
            **kwargs,
        )
    except ValueError as e:
        # If the error pertains to the processor class not existing or not
        # currently being imported, suggest using the --trust-remote-code flag.
        # Unlike AutoTokenizer, AutoVideoProcessor does not separate such errors
        if not trust_remote_code:
            err_msg = (
                "Failed to load the video processor. If the video processor is "
                "a custom processor not yet available in the HuggingFace "
                "transformers library, consider setting "
                "`trust_remote_code=True` in LLM or using the "
                "`--trust-remote-code` flag in the CLI."
            )
            raise RuntimeError(err_msg) from e
        else:
            raise e

    return cast(BaseVideoProcessor, processor)
```
**EN:** This function retrieves video processor. The docstring states that Load a video processor for the given model name via HuggingFace. Main inputs include `processor_name`, `revision`, `trust_remote_code`, `processor_cls_overrides`.
**CN:** 该函数负责完成 `get_video_processor` 对应的核心步骤。 文档字符串进一步说明了它的输入与行为。 主要输入参数包括 `processor_name`, `revision`, `trust_remote_code`, `processor_cls_overrides`。

### Lines 514-514: Module state and constants
```python
cached_get_video_processor = lru_cache(get_video_processor)
```
**EN:** This block initializes module-level state such as `cached_get_video_processor`. The assigned names act as shared context for the rest of the file.
**CN:** 该代码块初始化了模块级状态，例如 `cached_get_video_processor`。这些名称为文件其余部分提供共享上下文。

### Lines 517-528: Function `cached_video_processor_from_config`
```python
def cached_video_processor_from_config(
    model_config: "ModelConfig",
    processor_cls: type[_V] | None = None,
    **kwargs: Any,
):
    return cached_get_video_processor(
        model_config.model,
        revision=model_config.revision,
        trust_remote_code=model_config.trust_remote_code,
        processor_cls_overrides=processor_cls,  # type: ignore[arg-type]
        **_merge_mm_kwargs(model_config, AutoVideoProcessor, **kwargs),
    )
```
**EN:** This function implements `cached_video_processor_from_config`. Main inputs include `model_config`, `processor_cls`.
**CN:** 该函数实现 `cached_video_processor_from_config` 相关逻辑。 主要输入参数包括 `model_config`, `processor_cls`。

### Lines 531-568: Function `call_hf_processor_mm_only`
```python
def call_hf_processor_mm_only(
    processor: ProcessorMixin,
    images: ImageInput | None = None,
    videos: VideoInput | None = None,
    audio: AudioInput | None = None,
    **kwargs,
) -> BatchFeature:
    output_kwargs = processor._merge_kwargs(
        get_processor_kwargs_type(processor),
        **kwargs,
    )

    if audio is not None and (
        feature_extractor := getattr(processor, "feature_extractor", None)
    ):
        audio_inputs = feature_extractor(audio, **output_kwargs["audio_kwargs"])
        audio_inputs["feature_attention_mask"] = audio_inputs.pop("attention_mask")
    else:
        audio_inputs = {}

    if images is not None and (
        image_processor := getattr(processor, "image_processor", None)
    ):
        images_inputs = image_processor(images=images, **output_kwargs["images_kwargs"])
    else:
        images_inputs = {}

    if videos is not None and (
        video_processor := getattr(processor, "video_processor", None)
    ):
        videos_inputs = video_processor(videos=videos, **output_kwargs["videos_kwargs"])
    else:
        videos_inputs = {}

    return BatchFeature(
        data={**audio_inputs, **images_inputs, **videos_inputs},
        tensor_type=kwargs.get("return_tensors"),
    )
```
**EN:** This function implements `call_hf_processor_mm_only`. Main inputs include `processor`, `images`, `videos`, `audio`.
**CN:** 该函数实现 `call_hf_processor_mm_only` 相关逻辑。 主要输入参数包括 `processor`, `images`, `videos`, `audio`。

## Key Concepts / 关键概念
- **EN:** The module is tightly coupled to Hugging Face Transformers APIs, so version compatibility is often an explicit concern.
- **CN:** 该模块与 Hugging Face Transformers API 紧密耦合，因此版本兼容性通常是显式关注点。
- **EN:** Internal vLLM imports connect the file to runtime registries, logging, artifact lookup, and model-execution utilities.
- **CN:** vLLM 内部导入会把该文件连接到运行时注册表、日志、模型文件查找以及模型执行工具。

## Dependencies / 依赖关系
- **EN:** Standard library modules: `importlib`, `inspect`, `functools`, `typing`.
- **CN:** 标准库模块：`importlib`, `inspect`, `functools`, `typing`。
- **EN:** External packages: `transformers`, `transformers.audio_utils`, `transformers.feature_extraction_utils`, `transformers.image_processing_utils`, `transformers.image_utils`, `transformers.processing_utils`, `transformers.video_processing_utils`, `transformers.video_utils`, `typing_extensions`.
- **CN:** 外部依赖包：`transformers`, `transformers.audio_utils`, `transformers.feature_extraction_utils`, `transformers.image_processing_utils`, `transformers.image_utils`, `transformers.processing_utils`, `transformers.video_processing_utils`, `transformers.video_utils`, `typing_extensions`。
- **EN:** Internal modules: `vllm.logger`, `vllm.transformers_utils`, `vllm.transformers_utils.gguf_utils`, `vllm.transformers_utils.repo_utils`, `vllm.transformers_utils.utils`, `vllm.utils.func_utils`, `vllm.config`.
- **CN:** 内部模块：`vllm.logger`, `vllm.transformers_utils`, `vllm.transformers_utils.gguf_utils`, `vllm.transformers_utils.repo_utils`, `vllm.transformers_utils.utils`, `vllm.utils.func_utils`, `vllm.config`。
