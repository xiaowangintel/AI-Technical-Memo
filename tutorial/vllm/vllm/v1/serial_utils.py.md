# serial_utils.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/v1/serial_utils.py`
- **Repository**: vllm-project/vllm
- **Purpose**: Implements `OOBTensorConsumer`, `_log_insecure_serialization_warning`, `_typestr` for the V1 `v1` subsystem. / 为 V1 的 `v1` 子系统实现 `OOBTensorConsumer`, `_log_insecure_serialization_warning`, `_typestr`。

## Line-by-Line Analysis / 逐行分析
### Module setup / 模块初始化
```python
import dataclasses
import importlib
import pickle
from abc import ABC, abstractmethod
from collections.abc import Callable, Sequence
from functools import partial
from inspect import isclass
from types import FunctionType
from typing import Any, ClassVar, TypeAlias, cast, get_type_hints

import cloudpickle
import msgspec
import numpy as np
import torch
import zmq
from msgspec import msgpack
from pydantic import GetCoreSchemaHandler
from pydantic_core import core_schema

from vllm import envs
from vllm.logger import init_logger
from vllm.multimodal.inputs import (
    BaseMultiModalField,
    MultiModalBatchedField,
    MultiModalFieldConfig,
    MultiModalFieldElem,
    MultiModalFlatField,
    MultiModalKwargsItem,
    MultiModalKwargsItems,
    MultiModalSharedField,
    NestedTensors,
)
from vllm.utils.platform_utils import is_pin_memory_available
from vllm.v1.utils import tensor_data

logger = init_logger(__name__)

CUSTOM_TYPE_PICKLE = 1
CUSTOM_TYPE_CLOUDPICKLE = 2
CUSTOM_TYPE_RAW_VIEW = 3

# MultiModalField class serialization type map.
# These need to list all possible field types and match them
# to factory methods in `MultiModalFieldConfig`.
MMF_CLASS_TO_FACTORY: dict[type[BaseMultiModalField], str] = {
    MultiModalFlatField: "flat",
    MultiModalSharedField: "shared",
    MultiModalBatchedField: "batched",
}

bytestr: TypeAlias = bytes | bytearray | memoryview | zmq.Frame
```
**EN:** Combines imports, aliases, and module-level setup needed before the main definitions become active. Representative names include `logger`, `CUSTOM_TYPE_PICKLE`, `CUSTOM_TYPE_CLOUDPICKLE`, `CUSTOM_TYPE_RAW_VIEW`, `MMF_CLASS_TO_FACTORY`, `bytestr`.
**CN:** 该代码块组合了主定义生效前所需的导入、别名和模块级初始化逻辑。 代表性名称包括 `logger`, `CUSTOM_TYPE_PICKLE`, `CUSTOM_TYPE_CLOUDPICKLE`, `CUSTOM_TYPE_RAW_VIEW`, `MMF_CLASS_TO_FACTORY`, `bytestr`。

### `OOBTensorConsumer` class / `OOBTensorConsumer` 类
```python
class OOBTensorConsumer(ABC):
```
**EN:** Declares the `OOBTensorConsumer` interface. Downstream implementations are expected to provide methods such as `__call__`, `new_message`.
**CN:** `OOBTensorConsumer` 声明了一组接口约定。下游实现需要提供 `__call__`, `new_message` 等方法。

### `OOBTensorConsumer.__call__` method / `OOBTensorConsumer.__call__` 方法
```python
    @abstractmethod
    def __call__(self, tensor: torch.Tensor) -> dict | None:
        """
        Called with tensors for the current message.
        Returns None to reject the tensor (falls back to regular serialization),
        otherwise a dict with arbitrary placeholder data to be included
        in the serialized message.
        """
        return None
```
**EN:** This method implements `__call__` within `OOBTensorConsumer`. The docstring frames it as: Called with tensors for the current message.
**CN:** 该方法会实现 `__call__`，其作用域位于`OOBTensorConsumer`。

### `OOBTensorConsumer.new_message` method / `OOBTensorConsumer.new_message` 方法
```python
    @abstractmethod
    def new_message(self) -> None:
        """Called at the start of each new encoded message."""
        pass
```
**EN:** This method implements `new_message` within `OOBTensorConsumer`. The docstring frames it as: Called at the start of each new encoded message.
**CN:** 该方法会实现 `new_message`，其作用域位于`OOBTensorConsumer`。

### Module constants / 模块常量
```python
OOBTensorProvider = Callable[[str, tuple[int, ...], dict], torch.Tensor]
```
**EN:** Defines module-level constants or aliases such as `OOBTensorProvider`, which are reused by later definitions.
**CN:** 定义 `OOBTensorProvider` 等模块级常量或别名，供后续定义复用。

### `_log_insecure_serialization_warning` function / `_log_insecure_serialization_warning` 函数
```python
def _log_insecure_serialization_warning():
    logger.warning_once(
        "Allowing insecure serialization using pickle due to "
        "VLLM_ALLOW_INSECURE_SERIALIZATION=1"
    )
```
**EN:** This function implements `_log_insecure_serialization_warning` within the module. Key calls include `warning_once`.
**CN:** 该函数会实现 `_log_insecure_serialization_warning`，其作用域位于the module。 关键调用包括 `warning_once`。

### `_typestr` function / `_typestr` 函数
```python
def _typestr(val: Any) -> tuple[str, str] | None:
    if val is None:
        return None
    t = type(val)
    return t.__module__, t.__qualname__
```
**EN:** This function implements `_typestr` within the module. Key calls include `type`. The control flow contains 1 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_typestr`，其作用域位于the module。 关键调用包括 `type`。 控制流包含 1 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_encode_type_info_recursive` function / `_encode_type_info_recursive` 函数
```python
def _encode_type_info_recursive(obj: Any) -> Any:
    """Recursively encode type information for nested structures of
    lists/dicts."""
    if obj is None:
        return None
    if type(obj) is list:
        return [_encode_type_info_recursive(item) for item in obj]
    if type(obj) is dict:
        return {k: _encode_type_info_recursive(v) for k, v in obj.items()}
    return _typestr(obj)
```
**EN:** This function implements `_encode_type_info_recursive` within the module. The docstring frames it as: Recursively encode type information for nested structures of lists/dicts. Key calls include `_typestr`, `type`, `_encode_type_info_recursive`, `items`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_encode_type_info_recursive`，其作用域位于the module。 关键调用包括 `_typestr`, `type`, `_encode_type_info_recursive`, `items`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `_decode_type_info_recursive` function / `_decode_type_info_recursive` 函数
```python
def _decode_type_info_recursive(
    type_info: Any, data: Any, convert_fn: Callable[[Sequence[str], Any], Any]
) -> Any:
    """Recursively decode type information for nested structures of
    lists/dicts."""
    if type_info is None:
        return data
    if isinstance(type_info, dict):
        assert isinstance(data, dict)
        return {
            k: _decode_type_info_recursive(type_info[k], data[k], convert_fn)
            for k in type_info
        }
    if isinstance(type_info, list) and (
        # Exclude serialized tensors/numpy arrays.
        len(type_info) != 2 or not isinstance(type_info[0], str)
    ):
        assert isinstance(data, list)
        return [
            _decode_type_info_recursive(ti, d, convert_fn)
            for ti, d in zip(type_info, data)
        ]
    return convert_fn(type_info, data)
```
**EN:** This function implements `_decode_type_info_recursive` within the module. The docstring frames it as: Recursively decode type information for nested structures of lists/dicts. Key calls include `isinstance`, `convert_fn`, `_decode_type_info_recursive`, `len`, `zip`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会实现 `_decode_type_info_recursive`，其作用域位于the module。 关键调用包括 `isinstance`, `convert_fn`, `_decode_type_info_recursive`, `len`, `zip`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `UtilityResult` class / `UtilityResult` 类
```python
class UtilityResult:
    """Wrapper for special handling when serializing/deserializing."""
```
**EN:** Introduces the `UtilityResult` class. Core methods include `__init__`. Docstring signal: Wrapper for special handling when serializing/deserializing.
**CN:** 这里定义 `UtilityResult` 类。核心方法包括 `__init__`。

### `UtilityResult.__init__` method / `UtilityResult.__init__` 方法
```python
    def __init__(self, r: Any = None):
        self.result = r
```
**EN:** This method initializes the object state within `UtilityResult`. It touches state such as `result`.
**CN:** 该方法会初始化对象状态，其作用域位于`UtilityResult`。 它会读写 `result` 等状态。

### `MsgpackEncoder` class / `MsgpackEncoder` 类
```python
class MsgpackEncoder:
    """Encoder with custom torch tensor and numpy array serialization.

    Note that unlike vanilla `msgspec` Encoders, this interface is generally
    not thread-safe when encoding tensors / numpy arrays.

    By default, arrays below 256B are serialized inline Larger will get sent
    via dedicated messages. Note that this is a per-tensor limit.

    When a ``oob_tensor_consumer`` is provided, tensors (CUDA and CPU) will be
    offered to it for out-of-band handling.
    """
```
**EN:** Introduces the `MsgpackEncoder` class. Core methods include `__init__`, `encode`, `encode_into`, `enc_hook`, `_encode_ndarray`, `_encode_tensor`. Docstring signal: Encoder with custom torch tensor and numpy array serialization.
**CN:** 这里定义 `MsgpackEncoder` 类。核心方法包括 `__init__`, `encode`, `encode_into`, `enc_hook`, `_encode_ndarray`, `_encode_tensor`。

### `MsgpackEncoder.__init__` method / `MsgpackEncoder.__init__` 方法
```python
    def __init__(
        self,
        size_threshold: int | None = None,
        oob_tensor_consumer: OOBTensorConsumer | None = None,
    ):
        if size_threshold is None:
            size_threshold = envs.VLLM_MSGPACK_ZERO_COPY_THRESHOLD
        self.encoder = msgpack.Encoder(enc_hook=self.enc_hook)
        # This is used as a local stash of buffers that we can then access from
        # our custom `msgspec` hook, `enc_hook`. We don't have a way to
        # pass custom data to the hook otherwise.
        self.aux_buffers: list[bytestr] | None = None
        self.size_threshold = size_threshold
        self.oob_tensor_consumer = oob_tensor_consumer
        if envs.VLLM_ALLOW_INSECURE_SERIALIZATION:
            _log_insecure_serialization_warning()
```
**EN:** This method initializes the object state within `MsgpackEncoder`. Key calls include `Encoder`, `_log_insecure_serialization_warning`. It touches state such as `encoder`, `aux_buffers`, `size_threshold`, `oob_tensor_consumer`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`MsgpackEncoder`。 关键调用包括 `Encoder`, `_log_insecure_serialization_warning`。 它会读写 `encoder`, `aux_buffers`, `size_threshold`, `oob_tensor_consumer` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MsgpackEncoder.encode` method / `MsgpackEncoder.encode` 方法
```python
    def encode(self, obj: Any) -> Sequence[bytestr]:
        try:
            if self.oob_tensor_consumer is not None:
                self.oob_tensor_consumer.new_message()
            self.aux_buffers = bufs = [b""]
            bufs[0] = self.encoder.encode(obj)
            # This `bufs` list allows us to collect direct pointers to backing
            # buffers of tensors and np arrays, and return them along with the
            # top-level encoded buffer instead of copying their data into the
            # new buffer.
            return bufs
        finally:
            self.aux_buffers = None
```
**EN:** This method implements `encode` within `MsgpackEncoder`. Key calls include `encode`, `new_message`. It touches state such as `aux_buffers`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `encode`，其作用域位于`MsgpackEncoder`。 关键调用包括 `encode`, `new_message`。 它会读写 `aux_buffers` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MsgpackEncoder.enc_hook` method / `MsgpackEncoder.enc_hook` 方法
```python
    def enc_hook(self, obj: Any) -> Any:
        if isinstance(obj, torch.Tensor):
            return self._encode_tensor(obj)

        # Fall back to pickle for object or void kind ndarrays.
        if isinstance(obj, np.ndarray) and obj.dtype.kind not in ("O", "V"):
            return self._encode_ndarray(obj)

        if isinstance(obj, slice):
            # We are assuming only int-based values will be used here.
            return tuple(
                int(v) if v is not None else None
                for v in (obj.start, obj.stop, obj.step)
            )

        if isinstance(obj, MultiModalKwargsItem):
            return self._encode_mm_item(obj)

        if isinstance(obj, MultiModalKwargsItems):
            return self._encode_mm_items(obj)

        if isinstance(obj, UtilityResult):
            result = obj.result
            if not envs.VLLM_ALLOW_INSECURE_SERIALIZATION:
                return None, result
            # Since utility results are not strongly typed, we recursively
            # encode type information for nested structures of lists/dicts
            # to help with correct msgspec deserialization.
            return _encode_type_info_recursive(result), result

        if not envs.VLLM_ALLOW_INSECURE_SERIALIZATION:
            raise TypeError(
                f"Object of type {type(obj)} is not serializable"
                "Set VLLM_ALLOW_INSECURE_SERIALIZATION=1 to allow "
                "fallback to pickle-based serialization."
            )

        if isinstance(obj, FunctionType):
            # `pickle` is generally faster than cloudpickle, but can have
            # problems serializing methods.
            return msgpack.Ext(CUSTOM_TYPE_CLOUDPICKLE, cloudpickle.dumps(obj))

        return msgpack.Ext(
            CUSTOM_TYPE_PICKLE, pickle.dumps(obj, protocol=pickle.HIGHEST_PROTOCOL)
        )
```
**EN:** This method implements `enc_hook` within `MsgpackEncoder`. Key calls include `isinstance`, `Ext`, `_encode_tensor`, `_encode_ndarray`, `tuple`, `_encode_mm_item`. The control flow contains 10 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `enc_hook`，其作用域位于`MsgpackEncoder`。 关键调用包括 `isinstance`, `Ext`, `_encode_tensor`, `_encode_ndarray`, `tuple`, `_encode_mm_item`。 控制流包含 10 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MsgpackDecoder` class / `MsgpackDecoder` 类
```python
class MsgpackDecoder:
    """Decoder with custom torch tensor and numpy array serialization.

    Note that unlike vanilla `msgspec` Decoders, this interface is generally
    not thread-safe when encoding tensors / numpy arrays.

    ``oob_tensor_provider`` must be used when an OOBTensorConsumer is used on the
    encoder side.
    """
```
**EN:** Introduces the `MsgpackDecoder` class. Core methods include `__init__`, `decode`, `dec_hook`, `_decode_utility_result`, `_convert_result`, `_decode_ndarray`. Docstring signal: Decoder with custom torch tensor and numpy array serialization.
**CN:** 这里定义 `MsgpackDecoder` 类。核心方法包括 `__init__`, `decode`, `dec_hook`, `_decode_utility_result`, `_convert_result`, `_decode_ndarray`。

### `MsgpackDecoder.__init__` method / `MsgpackDecoder.__init__` 方法
```python
    def __init__(
        self,
        t: Any | None = None,
        share_mem: bool = True,
        oob_tensor_provider: OOBTensorProvider | None = None,
    ):
        self.share_mem = share_mem
        self.pin_tensors = is_pin_memory_available()
        args = () if t is None else (t,)
        self.decoder = msgpack.Decoder(
            *args, ext_hook=self.ext_hook, dec_hook=self.dec_hook
        )
        self.aux_buffers: Sequence[bytestr] = ()
        self.oob_tensor_provider = oob_tensor_provider
        if envs.VLLM_ALLOW_INSECURE_SERIALIZATION:
            _log_insecure_serialization_warning()
```
**EN:** This method initializes the object state within `MsgpackDecoder`. Key calls include `is_pin_memory_available`, `Decoder`, `_log_insecure_serialization_warning`. It touches state such as `share_mem`, `pin_tensors`, `decoder`, `aux_buffers`, `oob_tensor_provider`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会初始化对象状态，其作用域位于`MsgpackDecoder`。 关键调用包括 `is_pin_memory_available`, `Decoder`, `_log_insecure_serialization_warning`。 它会读写 `share_mem`, `pin_tensors`, `decoder`, `aux_buffers`, `oob_tensor_provider` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MsgpackDecoder.decode` method / `MsgpackDecoder.decode` 方法
```python
    def decode(self, bufs: bytestr | Sequence[bytestr]) -> Any:
        if isinstance(bufs, bytestr):  # type: ignore
            return self.decoder.decode(bufs)

        self.aux_buffers = bufs
        try:
            return self.decoder.decode(bufs[0])
        finally:
            self.aux_buffers = ()
```
**EN:** This method handles decoding logic within `MsgpackDecoder`. Key calls include `isinstance`, `decode`. It touches state such as `aux_buffers`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会处理解码逻辑，其作用域位于`MsgpackDecoder`。 关键调用包括 `isinstance`, `decode`。 它会读写 `aux_buffers` 等状态。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `MsgpackDecoder._decode_tensor` method / `MsgpackDecoder._decode_tensor` 方法
```python
    def _decode_tensor(self, arr: Any) -> torch.Tensor:
        dtype, shape, data = arr
        if isinstance(data, dict):
            assert self.oob_tensor_provider, (
                "Received OOB tensor but tensor provider is not set"
            )
            return self.oob_tensor_provider(dtype, shape, data)

        is_aux = isinstance(data, int)
        buffer = self.aux_buffers[data] if is_aux else data
        buffer = buffer if isinstance(buffer, memoryview) else memoryview(buffer)
        torch_dtype = getattr(torch, dtype)
        assert isinstance(torch_dtype, torch.dtype)
        if not buffer.nbytes:  # torch.frombuffer doesn't like empty buffers
            assert 0 in shape
            return torch.empty(shape, dtype=torch_dtype)
        # Create uint8 array
        arr = torch.frombuffer(buffer, dtype=torch.uint8)
        # Clone ensures tensor is backed by pytorch-owned memory for safe
        # future async CPU->GPU transfer.
        # Pin larger tensors for more efficient CPU->GPU transfer.
        if not is_aux:
            arr = arr.clone()
        elif not self.share_mem:
            arr = arr.pin_memory() if self.pin_tensors else arr.clone()
        # Convert back to proper shape & type
        return arr.view(torch_dtype).view(shape)
```
**EN:** This method implements `_decode_tensor` within `MsgpackDecoder`. Key calls include `isinstance`, `getattr`, `frombuffer`, `view`, `oob_tensor_provider`, `memoryview`. The control flow contains 7 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_decode_tensor`，其作用域位于`MsgpackDecoder`。 关键调用包括 `isinstance`, `getattr`, `frombuffer`, `view`, `oob_tensor_provider`, `memoryview`。 控制流包含 7 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `run_method` function / `run_method` 函数
```python
def run_method(
    obj: Any,
    method: str | bytes | Callable,
    args: tuple[Any, ...],
    kwargs: dict[str, Any],
) -> Any:
    """
    Run a method of an object with the given arguments and keyword arguments.
    If the method is string, it will be converted to a method using getattr.
    If the method is serialized bytes and will be deserialized using
    cloudpickle.
    If the method is a callable, it will be called directly.
    """
    if isinstance(method, bytes):
        func = partial(cloudpickle.loads(method), obj)
    elif isinstance(method, str):
        try:
            func = getattr(obj, method)
        except AttributeError:
            raise NotImplementedError(
                f"Method {method!r} is not implemented."
            ) from None
    else:
        func = partial(method, obj)  # type: ignore
    return func(*args, **kwargs)
```
**EN:** This function runs the main execution path within the module. The docstring frames it as: Run a method of an object with the given arguments and keyword arguments. Key calls include `isinstance`, `func`, `partial`, `loads`, `getattr`, `NotImplementedError`. The control flow contains 3 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该函数会执行主要运行路径，其作用域位于the module。 关键调用包括 `isinstance`, `func`, `partial`, `loads`, `getattr`, `NotImplementedError`。 控制流包含 3 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PydanticMsgspecMixin` class / `PydanticMsgspecMixin` 类
```python
class PydanticMsgspecMixin:
    """Make a ``msgspec.Struct`` compatible with Pydantic for both
    **validation** (JSON/dict -> Struct) and **serialization**
    (Struct -> JSON-safe dict).

    Subclasses may set ``__pydantic_msgspec_exclude__`` (a ``set[str]``)
    to list non-underscore field names that should also be stripped from
    serialized output.  Fields whose names start with ``_`` are always
    excluded automatically.
    """

    # Subclasses can override to exclude additional public-but-internal keys.
    __pydantic_msgspec_exclude__: ClassVar[set[str]] = set()
```
**EN:** Introduces the `PydanticMsgspecMixin` class. Core methods include `__get_pydantic_core_schema__`, `_validate_msgspec`, `_serialize_msgspec`. Docstring signal: Make a ``msgspec.Struct`` compatible with Pydantic for both **validation** (JSON/dict -> Struct) and **serialization** (Struct -> JSON-safe dict).
**CN:** 这里定义 `PydanticMsgspecMixin` 类。核心方法包括 `__get_pydantic_core_schema__`, `_validate_msgspec`, `_serialize_msgspec`。

### `PydanticMsgspecMixin.__get_pydantic_core_schema__` method / `PydanticMsgspecMixin.__get_pydantic_core_schema__` 方法
```python
    @classmethod
    def __get_pydantic_core_schema__(
        cls, source_type: Any, handler: GetCoreSchemaHandler
    ) -> core_schema.CoreSchema:
        """
        Make msgspec.Struct compatible with Pydantic, respecting defaults.
        Handle JSON=>msgspec.Struct. Used when exposing msgspec.Struct to the
        API as input or in `/docs`. Note this is cached by Pydantic and not
        called on every validation.
        """
        msgspec_fields = {f.name: f for f in msgspec.structs.fields(source_type)}
        type_hints = get_type_hints(source_type)

        # Build the Pydantic typed_dict_field for each msgspec field
        fields = {}
        for name, hint in type_hints.items():
            if name not in msgspec_fields:
                # Skip ClassVar and other non-struct annotations.
                continue
            # Skip private fields — they are excluded from serialization
            # and should not appear in the generated JSON/OpenAPI schema.
            if name.startswith("_"):
                continue
            msgspec_field = msgspec_fields[name]

            # typed_dict_field using the handler to get the schema
            field_schema = handler(hint)

            # Add default value to the schema.
            # Mark fields with defaults as not required so the generated
            # JSON Schema stays consistent with ``omit_defaults=True``
            # serialization (fields at their default value may be absent).
            if msgspec_field.default_factory is not msgspec.NODEFAULT:
                wrapped_schema = core_schema.with_default_schema(
                    schema=field_schema,
                    default_factory=msgspec_field.default_factory,
                )
                fields[name] = core_schema.typed_dict_field(
                    wrapped_schema, required=False
                )
            elif msgspec_field.default is not msgspec.NODEFAULT:
                wrapped_schema = core_schema.with_default_schema(
                    schema=field_schema,
                    default=msgspec_field.default,
                )
                fields[name] = core_schema.typed_dict_field(
                    wrapped_schema, required=False
                )
            else:
                # No default, so Pydantic will treat it as required
                fields[name] = core_schema.typed_dict_field(field_schema)
        typed_dict_then_convert = core_schema.no_info_after_validator_function(
            cls._validate_msgspec,
            core_schema.typed_dict_schema(fields),
        )

        # Build a serializer that strips private / excluded fields.
        serializer = core_schema.plain_serializer_function_ser_schema(
            cls._serialize_msgspec,
            info_arg=False,
        )

        # Accept either an already-constructed msgspec.Struct instance or a
        # JSON/dict-like payload.
        return core_schema.union_schema(
            [
                core_schema.is_instance_schema(source_type),
                typed_dict_then_convert,
            ],
            serialization=serializer,
        )
```
**EN:** This method implements `__get_pydantic_core_schema__` within `PydanticMsgspecMixin`. The docstring frames it as: Make msgspec.Struct compatible with Pydantic, respecting defaults. Key calls include `get_type_hints`, `items`, `no_info_after_validator_function`, `plain_serializer_function_ser_schema`, `union_schema`, `startswith`. The control flow contains 4 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `__get_pydantic_core_schema__`，其作用域位于`PydanticMsgspecMixin`。 关键调用包括 `get_type_hints`, `items`, `no_info_after_validator_function`, `plain_serializer_function_ser_schema`, `union_schema`, `startswith`。 控制流包含 4 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

### `PydanticMsgspecMixin._validate_msgspec` method / `PydanticMsgspecMixin._validate_msgspec` 方法
```python
    @classmethod
    def _validate_msgspec(cls, value: Any) -> Any:
        """Validate and convert input to msgspec.Struct instance."""
        if isinstance(value, cls):
            return value
        if isinstance(value, dict):
            return cls(**value)
        return msgspec.convert(value, type=cls)
```
**EN:** This method implements `_validate_msgspec` within `PydanticMsgspecMixin`. The docstring frames it as: Validate and convert input to msgspec.Struct instance. Key calls include `isinstance`, `convert`, `cls`. The control flow contains 2 branch(es) and 0 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_validate_msgspec`，其作用域位于`PydanticMsgspecMixin`。 关键调用包括 `isinstance`, `convert`, `cls`。 控制流包含 2 个分支和 0 个循环，说明这里承担了较强的协调逻辑。

### `PydanticMsgspecMixin._serialize_msgspec` method / `PydanticMsgspecMixin._serialize_msgspec` 方法
```python
    @staticmethod
    def _serialize_msgspec(value: Any) -> Any:
        """Serialize a msgspec.Struct to a JSON-compatible dict, stripping
        private (``_``-prefixed) and explicitly excluded fields.

        Uses ``msgspec.to_builtins`` which respects ``omit_defaults=True``,
        so only fields that differ from their declared defaults are included.
        """
        raw = msgspec.to_builtins(value)
        if not isinstance(raw, dict):
            return raw

        exclude: set[str] = cast(
            set[str],
            getattr(type(value), "__pydantic_msgspec_exclude__", set()),
        )
        for key in list(raw):
            if key.startswith("_") or key in exclude:
                del raw[key]

        return raw
```
**EN:** This method implements `_serialize_msgspec` within `PydanticMsgspecMixin`. The docstring frames it as: Serialize a msgspec.Struct to a JSON-compatible dict, stripping private (``_``-prefixed) and explicitly excluded fields. Key calls include `to_builtins`, `cast`, `list`, `isinstance`, `getattr`, `type`. The control flow contains 2 branch(es) and 1 loop(s), indicating non-trivial coordination logic.
**CN:** 该方法会实现 `_serialize_msgspec`，其作用域位于`PydanticMsgspecMixin`。 关键调用包括 `to_builtins`, `cast`, `list`, `isinstance`, `getattr`, `type`。 控制流包含 2 个分支和 1 个循环，说明这里承担了较强的协调逻辑。

## Key Concepts / 关键概念
- `OOBTensorConsumer`: central class or interface in this module. / `OOBTensorConsumer`：本模块中的核心类或接口。
- `_log_insecure_serialization_warning`: top-level helper or orchestration entry point. / `_log_insecure_serialization_warning`：顶层辅助函数或编排入口。
- `_typestr`: top-level helper or orchestration entry point. / `_typestr`：顶层辅助函数或编排入口。
- `_encode_type_info_recursive`: top-level helper or orchestration entry point. / `_encode_type_info_recursive`：顶层辅助函数或编排入口。
- `_decode_type_info_recursive`: top-level helper or orchestration entry point. / `_decode_type_info_recursive`：顶层辅助函数或编排入口。
- `UtilityResult`: central class or interface in this module. / `UtilityResult`：本模块中的核心类或接口。
- `MsgpackEncoder`: central class or interface in this module. / `MsgpackEncoder`：本模块中的核心类或接口。
- `MsgpackDecoder`: central class or interface in this module. / `MsgpackDecoder`：本模块中的核心类或接口。

## Dependencies / 依赖关系
- Standard library / 标准库: `dataclasses`, `importlib`, `pickle`, `abc`, `collections`, `functools`, `inspect`, `types`, `typing`
- External / 外部依赖: `cloudpickle`, `msgspec`, `numpy`, `torch`, `zmq`, `pydantic`, `pydantic_core`
- Internal vLLM / 内部依赖: `vllm`, `vllm.logger`, `vllm.multimodal.inputs`, `vllm.utils.platform_utils`, `vllm.v1.utils`
