# scalar_type.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `vllm/scalar_type.py`
- **Repository**: `vllm-project/vllm`
- **Purpose**: EN: Models custom integer and floating-point scalar formats, including sub-byte and biased quantized types, in sync with the C++ runtime. / CN: 建模与 C++ 运行时保持同步的自定义整数/浮点标量格式，覆盖子字节类型和带偏置的量化类型。

## Line-by-Line Analysis / 逐行分析

### NaN representation enum and ScalarType fields
```python
# Mirrors enum in `core/scalar_type.hpp`
class NanRepr(Enum):
    NONE = 0  # nans are not supported
    IEEE_754 = 1  # nans are: Exp all 1s, mantissa not all 0s
    EXTD_RANGE_MAX_MIN = 2  # nans are: Exp all 1s, mantissa all 1s


# This ScalarType class is a parallel implementation of the C++ ScalarType
# class found in csrc/core/scalar_type.hpp.  These two classes should be kept
# in sync until the inductor fully supports custom C++ classes.
@dataclass(frozen=True)
class ScalarType:
    """
    ScalarType can represent a wide range of floating point and integer
    types, in particular it can be used to represent sub-byte data types
    (something that torch.dtype currently does not support). It is also
    capable of  representing types with a bias, i.e.:
      `stored_value = value + bias`,
    this is useful for quantized types (e.g. standard GPTQ 4bit uses a bias
    of 8). The implementation for this class can be found in
    csrc/core/scalar_type.hpp, these type signatures should be kept in sync
    with that file.
    """

    exponent: int
    """
    Number of bits in the exponent if this is a floating point type
    (zero if this an integer type)
    """

    mantissa: int
    """
    Number of bits in the mantissa if this is a floating point type,
    or the number bits representing an integer excluding the sign bit if
    this an integer type.
    """

    signed: bool
    "If the type is signed (i.e. has a sign bit)"

    bias: int
    """
    bias used to encode the values in this scalar type
    (value = stored_value - bias, default 0) for example if we store the
    type as an unsigned integer with a bias of 128 then the value 0 will be
    stored as 128 and -1 will be stored as 127 and 1 will be stored as 129.
    """

    _finite_values_only: bool = False
    """
    Private: if infs are supported, used `has_infs()` instead.
    """

    nan_repr: NanRepr = NanRepr.IEEE_754
    """
    How NaNs are represent in this scalar type, returns NanRepr value.
    (not applicable for integer types)
    """
```
**EN:** `NanRepr` mirrors the C++ enum that describes how NaN values are encoded. `ScalarType` itself stores the bit layout of a custom scalar format—exponent bits, mantissa bits, sign, optional integer bias, finite-only flag, and NaN convention—so Python can reason about formats that `torch.dtype` cannot represent directly.
**CN:** `NanRepr` 对应 C++ 侧描述 NaN 编码方式的枚举。`ScalarType` 本身则保存自定义标量格式的位级布局：指数位、尾数位、符号位、可选的整数偏置、仅有限值标志以及 NaN 约定，从而让 Python 侧能够表达 `torch.dtype` 无法直接覆盖的格式。

### Representable range helpers
```python
    def _floating_point_max_int(self) -> int:
        assert self.mantissa <= 52 and self.exponent <= 11, (
            f"Cannot represent max/min as a double for type {self.__str__()}"
        )

        max_mantissa = (1 << self.mantissa) - 1
        if self.nan_repr == NanRepr.EXTD_RANGE_MAX_MIN:
            max_mantissa = max_mantissa - 1

        max_exponent = (1 << self.exponent) - 2
        if self.nan_repr == NanRepr.EXTD_RANGE_MAX_MIN or self.nan_repr == NanRepr.NONE:
            assert self.exponent < 11, (
                f"Cannot represent max/min as a double for type {self.__str__()}"
            )
            max_exponent = max_exponent + 1

        # adjust the exponent to match that of a double
        # for now we assume the exponent bias is the standard 2^(e-1) -1, (where
        # e is the exponent bits), there is some precedent for non-standard
        # biases, example `float8_e4m3b11fnuz` here:
        # https://github.com/jax-ml/ml_dtypes but to avoid premature over
        # complication we are just assuming the standard exponent bias until
        # there is a need to support non-standard biases
        exponent_bias = (1 << (self.exponent - 1)) - 1
        exponent_bias_double = (1 << 10) - 1  # double e = 11

        max_exponent_double = max_exponent - exponent_bias + exponent_bias_double

        # shift the mantissa and exponent into the proper positions for an
        # IEEE double and bitwise-or them together.
        return (max_mantissa << (52 - self.mantissa)) | (max_exponent_double << 52)

    def _floating_point_max(self) -> float:
        double_raw = self._floating_point_max_int()
        return struct.unpack("!d", struct.pack("!Q", double_raw))[0]

    def _raw_max(self) -> int | float:
        if self.is_floating_point():
            return self._floating_point_max()
        else:
            assert self.size_bits < 64 or self.size_bits == 64 and self.is_signed(), (
                "Cannot represent max as an int"
            )
            return (1 << self.mantissa) - 1

    def _raw_min(self) -> int | float:
        if self.is_floating_point():
            assert self.is_signed(), (
                "We currently assume all floating point types are signed"
            )
            sign_bit_double = 1 << 63

            max_raw = self._floating_point_max_int()
            min_raw = max_raw | sign_bit_double
            return struct.unpack("!d", struct.pack("!Q", min_raw))[0]
        else:
            assert not self.is_signed() or self.size_bits <= 64, (
                "Cannot represent min as a int64_t"
            )

            if self.is_signed():
                return -(1 << (self.size_bits - 1))
            else:
                return 0
```
**EN:** The floating-point helper path reconstructs the maximum finite value of the custom format by building an IEEE double bit pattern with matching exponent and mantissa semantics. `_raw_max()` and `_raw_min()` then branch between floating-point and integer logic to compute extrema before any quantization bias is subtracted.
**CN:** 浮点辅助路径会通过构造一个等价的 IEEE double 位模式，重建该自定义格式能够表示的最大有限值，并尽量保持指数与尾数语义一致。随后 `_raw_max()` 与 `_raw_min()` 会在浮点和整数逻辑之间分支，先计算原始极值，再在上层扣除量化偏置。

### Packed IDs and capability queries
```python
    @functools.cached_property
    def id(self) -> int:
        """
        Convert the ScalarType to an int which can be passed to pytorch custom
        ops. This layout of the int must be kept in sync with the C++
        ScalarType's from_id method.
        """
        val = 0
        offset = 0

        def or_and_advance(member, bit_width):
            nonlocal val
            nonlocal offset
            bit_mask = (1 << bit_width) - 1
            val = val | (int(member) & bit_mask) << offset
            offset = offset + bit_width

        or_and_advance(self.exponent, 8)
        or_and_advance(self.mantissa, 8)
        or_and_advance(self.signed, 1)
        or_and_advance(self.bias, 32)
        or_and_advance(self._finite_values_only, 1)
        or_and_advance(self.nan_repr.value, 8)

        assert offset <= 64, f"ScalarType fields too big {offset} to fit into an int64"

        _SCALAR_TYPES_ID_MAP[val] = self

        return val

    @property
    def size_bits(self) -> int:
        return self.exponent + self.mantissa + int(self.signed)

    def min(self) -> int | float:
        """
        Min representable value for this scalar type.
        (accounting for bias if there is one)
        """
        return self._raw_min() - self.bias

    def max(self) -> int | float:
        """
        Max representable value for this scalar type.
        (accounting for bias if there is one)
        """
        return self._raw_max() - self.bias

    def is_signed(self) -> bool:
        """
        If the type is signed (i.e. has a sign bit), same as `signed`
        added for consistency with:
        https://pytorch.org/docs/stable/generated/torch.Tensor.is_signed.html
        """
        return self.signed

    def is_floating_point(self) -> bool:
        "If the type is a floating point type"
        return self.exponent != 0

    def is_integer(self) -> bool:
        "If the type is an integer type"
        return self.exponent == 0

    def has_bias(self) -> bool:
        "If the type has a non-zero bias"
        return self.bias != 0

    def has_infs(self) -> bool:
        "If the type is floating point and supports infinity"
        return not self._finite_values_only

    def has_nans(self) -> bool:
        return self.nan_repr != NanRepr.NONE.value

    def is_ieee_754(self) -> bool:
        """
        If the type is a floating point type that follows IEEE 754
        conventions
        """
        return self.nan_repr == NanRepr.IEEE_754.value and not self._finite_values_only
```
**EN:** `id` packs the scalar layout into a 64-bit integer compatible with the C++ `ScalarType::from_id` layout, caching the Python object in `_SCALAR_TYPES_ID_MAP` for reverse lookup. The remaining methods expose size, signedness, min/max after bias correction, and convenient predicates such as “is floating point” or “has infinities”.
**CN:** `id` 会把标量格式打包成一个与 C++ `ScalarType::from_id` 布局兼容的 64 位整数，并把 Python 对象缓存到 `_SCALAR_TYPES_ID_MAP` 中，以支持反向查找。其余方法则暴露位宽、有符号性、扣除偏置后的最小/最大值，以及“是否浮点”“是否支持无穷大”等便捷能力查询。

### Naming and constructors
```python
    def __str__(self) -> str:
        """
        naming generally follows: https://github.com/jax-ml/ml_dtypes
        for floating point types (leading f) the scheme is:
        `float<size_bits>_e<exponent_bits>m<mantissa_bits>[flags]`
        flags:
          - no-flags: means it follows IEEE 754 conventions
          - f: means finite values only (no infinities)
          - n: means nans are supported (non-standard encoding)
        for integer types the scheme is:
          `[u]int<size_bits>[b<bias>]`
          - if bias is not present it means its zero
        """
        if self.is_floating_point():
            ret = (
                "float"
                + str(self.size_bits)
                + "_e"
                + str(self.exponent)
                + "m"
                + str(self.mantissa)
            )

            if not self.is_ieee_754():
                if self._finite_values_only:
                    ret = ret + "f"
                if self.nan_repr != NanRepr.NONE:
                    ret = ret + "n"

            return ret
        else:
            ret = ("int" if self.is_signed() else "uint") + str(self.size_bits)
            if self.has_bias():
                ret = ret + "b" + str(self.bias)
            return ret

    def __repr__(self) -> str:
        return "ScalarType." + self.__str__()

    # __len__ needs to be defined (and has to throw TypeError) for pytorch's
    # opcheck to work.
    def __len__(self) -> int:
        raise TypeError

    #
    # Convenience Constructors
    #

    @classmethod
    def int_(cls, size_bits: int, bias: int | None) -> "ScalarType":
        "Create a signed integer scalar type (size_bits includes sign-bit)."
        ret = cls(0, size_bits - 1, True, bias if bias else 0)
        ret.id  # noqa B018: make sure the id is cached
        return ret

    @classmethod
    def uint(cls, size_bits: int, bias: int | None) -> "ScalarType":
        """Create an unsigned integer scalar type."""
        ret = cls(0, size_bits, False, bias if bias else 0)
        ret.id  # noqa B018: make sure the id is cached
        return ret

    @classmethod
    def float_IEEE754(cls, exponent: int, mantissa: int) -> "ScalarType":
        """
        Create a standard floating point type
        (i.e. follows IEEE 754 conventions).
        """
        assert mantissa > 0 and exponent > 0
        ret = cls(exponent, mantissa, True, 0)
        ret.id  # noqa B018: make sure the id is cached
        return ret

    @classmethod
    def float_(
        cls, exponent: int, mantissa: int, finite_values_only: bool, nan_repr: NanRepr
    ) -> "ScalarType":
        """
        Create a non-standard floating point type
        (i.e. does not follow IEEE 754 conventions).
        """
        assert mantissa > 0 and exponent > 0
        assert nan_repr != NanRepr.IEEE_754, (
            "use `float_IEEE754` constructor for floating point types that "
            "follow IEEE 754 conventions"
        )
        ret = cls(exponent, mantissa, True, 0, finite_values_only, nan_repr)
        ret.id  # noqa B018: make sure the id is cached
        return ret

    @classmethod
    def from_id(cls, scalar_type_id: int):
        if scalar_type_id not in _SCALAR_TYPES_ID_MAP:
            raise ValueError(f"scalar_type_id {scalar_type_id} doesn't exists.")
        return _SCALAR_TYPES_ID_MAP[scalar_type_id]
```
**EN:** `__str__()` follows the `ml_dtypes` naming style, producing strings like `float8_e4m3fn` or `uint4b8` that encode both layout and special behavior. The convenience constructors build common integer, IEEE-float, and non-standard float formats, precomputing their packed ids so later interop with custom ops is cheap.
**CN:** `__str__()` 采用类似 `ml_dtypes` 的命名风格，生成如 `float8_e4m3fn`、`uint4b8` 这样的名字，用于同时编码位布局与特殊行为。后面的便捷构造器可以快速构造常见整数格式、IEEE 浮点格式和非标准浮点格式，并提前计算好 packed id，从而降低后续与自定义算子交互的成本。

### Predefined scalar catalog
```python
class scalar_types:
    int4 = ScalarType.int_(4, None)
    uint4 = ScalarType.uint(4, None)
    int8 = ScalarType.int_(8, None)
    uint8 = ScalarType.uint(8, None)
    float8_e4m3fn = ScalarType.float_(4, 3, True, NanRepr.EXTD_RANGE_MAX_MIN)
    float8_e5m2 = ScalarType.float_IEEE754(5, 2)
    float8_e8m0fnu = ScalarType(8, 0, False, 0, True, NanRepr.EXTD_RANGE_MAX_MIN)
    float16_e8m7 = ScalarType.float_IEEE754(8, 7)
    float16_e5m10 = ScalarType.float_IEEE754(5, 10)

    # fp6, https://github.com/usyd-fsalab/fp6_llm/tree/main
    # and https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf
    float6_e3m2f = ScalarType.float_(3, 2, True, NanRepr.NONE)

    float6_e2m3f = ScalarType.float_(2, 3, True, NanRepr.NONE)

    # fp4, https://www.opencompute.org/documents/ocp-microscaling-formats-mx-v1-0-spec-final-pdf
    float4_e2m1f = ScalarType.float_(2, 1, True, NanRepr.NONE)

    # "gptq" types
    uint2b2 = ScalarType.uint(2, 2)
    uint3b4 = ScalarType.uint(3, 4)
    uint4b8 = ScalarType.uint(4, 8)
    uint8b128 = ScalarType.uint(8, 128)

    # colloquial names
    bfloat16 = float16_e8m7
    float16 = float16_e5m10
```
**EN:** `scalar_types` is a namespace of ready-made low-bit formats used elsewhere in the project: standard int/uint widths, float8/float16 variants, fp6/fp4 microscaling formats, and GPTQ-style biased unsigned integers. Using pre-instantiated objects keeps Python and C++ code aligned on the same canonical encodings.
**CN:** `scalar_types` 提供了一组项目内可直接复用的低比特格式命名空间：标准 int/uint 位宽、float8/float16 变体、fp6/fp4 microscaling 格式，以及 GPTQ 风格的带偏置无符号整数。使用预先实例化的对象可以让 Python 与 C++ 侧始终对齐到同一套规范编码。

## Key Concepts / 关键概念
- **Custom numeric layouts** — EN: The file describes bit-level formats that are more expressive than built-in `torch.dtype`. / CN: 该文件描述的位级数值格式比内置 `torch.dtype` 更灵活。
- **Python/C++ interop** — EN: Packed 64-bit ids let Python pass scalar-type metadata into custom C++ ops cheaply and deterministically. / CN: 打包后的 64 位 id 让 Python 可以低成本、确定性地把标量类型元数据传给 C++ 自定义算子。
- **Quantization bias** — EN: Integer formats may use a storage bias so raw encoded values map to signed mathematical values. / CN: 整数格式可以带存储偏置，使原始编码值映射为带符号的数学值。

## Dependencies / 依赖关系
- **struct** — EN: Used to reinterpret constructed bit patterns as IEEE double values. / CN: 用于把构造出的位模式重新解释成 IEEE double 浮点数。
- **dataclasses** — EN: Provides the frozen record semantics needed for immutable scalar-type descriptors. / CN: 提供不可变标量类型描述对象所需的 frozen dataclass 语义。
- **csrc/core/scalar_type.hpp** — EN: The Python layout and ids are explicitly documented as needing to stay in sync with the C++ implementation. / CN: 文档明确要求 Python 侧布局与 id 编码必须和 C++ 实现 `csrc/core/scalar_type.hpp` 保持同步。
