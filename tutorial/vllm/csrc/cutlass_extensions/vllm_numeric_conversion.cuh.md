# vllm_numeric_conversion.cuh — Code Analysis / 代码分析
## Source / 来源
- **File**: `csrc/cutlass_extensions/vllm_numeric_conversion.cuh`
- **Repository**: vllm-project/vllm
- **Purpose**: [EN] Extends CUTLASS numeric conversion with vLLM quantized types and several register-level fast paths for packed, interleaved, FP16, BF16, FP8, and INT8 conversions. / [CN] 在 CUTLASS 数值转换框架上扩展 vLLM 量化类型，并实现多种寄存器级快速路径，覆盖打包/交错数据以及 FP16、BF16、FP8、INT8 等转换。

## Line-by-Line Analysis / 逐行分析
### Interleaved converter framework / 交错转换框架
```cpp
// InterleavedNumericArrayConverter is like NumericArrayConverter but also
// deinterleaves converted elements based on IlvBlkLayout, interleaving can
// make subbyte converts more efficient by allowing for efficient extraction
// of subbyte elements from a 32bit register.
template <typename IlvBlkLayout, typename T, typename S, int N,
          FloatRoundStyle Round = FloatRoundStyle::round_to_nearest,
          class Enable = void>
struct InterleavedNumericArrayConverter {
  using Converter = NumericArrayConverter<T, S, N, Round>;

  using result_type = typename Converter::result_type;
  using source_type = typename Converter::source_type;

  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    if (cute::elect_one_sync()) {
      if constexpr (std::is_same_v<IlvBlkLayout, void>) {
        printf(
            "Convert %s <= %s (N = %d, IlvBlkLayout = void), not implemented\n",
            nameof_v<T>, nameof_v<S>, N);
      } else {
        printf(
            "Convert %s <= %s (N = %d, size(IlvBlkLayout{}) = %d), not "
            "implemented\n",
            nameof_v<T>, nameof_v<S>, N, size(IlvBlkLayout{}));
      }
      __brkpt();
    }
    return {};
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};

template <typename IlvBlkLayout, typename T, typename S, int N,
          FloatRoundStyle Round>
struct InterleavedNumericArrayConverter<
    IlvBlkLayout, T, S, N, Round,
    std::enable_if_t<is_identity_layout<IlvBlkLayout>()>> {
  using Converter = NumericArrayConverter<T, S, N, Round>;

  using result_type = typename Converter::result_type;
  using source_type = typename Converter::source_type;

  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return Converter::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};
```
**EN:** `InterleavedNumericArrayConverter` generalizes `NumericArrayConverter` with an additional interleaving layout parameter. The primary template is a debug stub: unsupported conversions print the source/target type names and break. A specialization short-circuits to the normal CUTLASS converter when the layout is effectively identity, so deinterleaving work is skipped when not needed.
**CN:** `InterleavedNumericArrayConverter` 在 `NumericArrayConverter` 基础上加入了交错布局参数。主模板本身是调试兜底：遇到不支持的转换时，会打印源/目标类型名并触发断点。若布局本质上是恒等布局，则特化版本会直接退回普通 CUTLASS 转换器，从而避免不必要的反交错开销。

### 32-bit packed conversion scaffold / 32 位打包转换骨架
```cpp
template <typename RegConvert32bit, typename T, typename S, int N>
struct ArrayConverterPacked32Bit {
  using result_type = Array<T, N>;
  using source_type = Array<S, N>;

  using result_packed_8_t = Array<T, 8>;
  using result_packed_4_t = Array<T, 4>;
  using result_packed_2_t = Array<T, 2>;
  using src_packed_8_t = Array<S, 8>;
  using src_packed_4_t = Array<S, 4>;
  using src_packed_2_t = Array<S, 2>;

  static_assert(N % 2 == 0, "N must be a multiple of 2");
  static_assert(cutlass::sizeof_bits_v<S> >= 4);  // TODO: add 16 packed sources
  static_assert(32 % cutlass::sizeof_bits_v<S> == 0);
  static constexpr auto src_elems_per_32bit_reg =
      32 / cutlass::sizeof_bits_v<S>;

  // Maybe not Valid. ScalarConverter will not actually work unless
  // NumericConverter<T, S, Round> is implemented. However it won't be used
  // anyways since we assert N % 2 == 0, just here for compliance with
  // VectorizedConverter.
  using ScalarConverter = NumericConverter<T, S>;

  template <typename PackedSrc>
  CUTLASS_DEVICE static auto to_regs(PackedSrc const& src) {
    if constexpr (sizeof(PackedSrc) == 1) {
      return Array<uint32_t, 1>{reinterpret_cast<uint8_t const&>(src)};
    } else if constexpr (sizeof(PackedSrc) == 2) {
      return Array<uint32_t, 1>{reinterpret_cast<uint16_t const&>(src)};
    } else if constexpr (sizeof(PackedSrc) == 4) {
      return Array<uint32_t, 1>{reinterpret_cast<uint32_t const&>(src)};
    } else {
      static_assert(sizeof(PackedSrc) == 8);
      return reinterpret_cast<Array<uint32_t, 2> const&>(src);
    }
  }

  // The core converter uses bit tricks to construct a known FP16 number, then
  // does a subtraction in FP16 for the final result.
  template <typename PackedResultType, typename PackedSrcType>
  CUTLASS_DEVICE static PackedResultType packed_convert(
      PackedSrcType const& source) {
    static_assert(PackedSrcType::kElements == PackedResultType::kElements);
    static_assert(PackedResultType::kElements == 2 ||
                      PackedResultType::kElements == 4 ||
                      PackedResultType::kElements == 8,
                  "Invalid PackedResultType must be 2, 4 or 8.");
    static_assert(std::is_same_v<typename PackedSrcType::Element, S>);
    static_assert(std::is_same_v<typename PackedResultType::Element, T>);

    return RegConvert32bit::template convert<PackedResultType>(to_regs(source));
  }

  friend class detail::VectorizedConverter;

 public:
  CUTLASS_DEVICE static result_type convert(source_type const& source) {
    result_type result;
    using ConverterType =
        ArrayConverterPacked32Bit<RegConvert32bit,
                                  typename result_type::Element,
                                  typename source_type::Element, N>;

    if constexpr (src_elems_per_32bit_reg >= 8) {
      detail::VectorizedConverter::convert<
          ConverterType, result_packed_8_t, src_packed_8_t, result_packed_4_t,
          src_packed_4_t, result_packed_2_t, src_packed_2_t>(result, source);
    } else if constexpr (src_elems_per_32bit_reg >= 4) {
      detail::VectorizedConverter::convert<ConverterType, result_packed_4_t,
                                           src_packed_4_t, result_packed_2_t,
                                           src_packed_2_t>(result, source);
    } else {
      detail::VectorizedConverter::convert<ConverterType, result_packed_2_t,
                                           src_packed_2_t>(result, source);
    }

    return result;
  }
};
```
**EN:** `ArrayConverterPacked32Bit` is the shared engine behind most specializations. It reinterprets small packed arrays as one or two 32-bit registers, exposes a `packed_convert` hook implemented by a register-level converter, and then lets CUTLASS `detail::VectorizedConverter` tile the work over vectors of 8, 4, or 2 elements depending on how many source elements fit in one register.
**CN:** `ArrayConverterPacked32Bit` 是大多数特化共享的底层骨架。它把小型打包数组重解释成一个或两个 32 位寄存器，暴露由寄存器级转换器实现的 `packed_convert` 钩子，再借助 CUTLASS 的 `detail::VectorizedConverter` 按 8/4/2 元素向量块来展开工作，具体取决于单个寄存器可容纳多少源元素。

### 4-bit LUT expansion / 4bit LUT 展开
```cpp
// Convert 8 4bit values packed into a 32bit register to 8 8bit values packed
// into 2 32bit register.
template <uint8_t LUT0, uint8_t LUT1, uint8_t LUT2, uint8_t LUT3,    //
          uint8_t LUT4, uint8_t LUT5, uint8_t LUT6, uint8_t LUT7,    //
          uint8_t LUT8, uint8_t LUT9, uint8_t LUT10, uint8_t LUT11,  //
          uint8_t LUT12, uint8_t LUT13, uint8_t LUT14, uint8_t LUT15>
CUTLASS_DEVICE cutlass::AlignedArray<uint32_t, 2> lut_4bit_to_8bit_convert(
    uint32_t src) {
  cutlass::AlignedArray<uint32_t, 2> r;
  // Determines if the value is in the top half of the LUT if set or
  //  (i.e. LUT[8:15]) in the bottom half (i.e. LUT[0:7]) if not set. Then move
  //  into bit position 0x4 of each nibble so when or'd with final_prmt_base it
  //  selects the correct candidate. When elements in final_prmt_base
  //  are >= 0x4, the high candidate is selected (i.e. LUT[8:15]), when elements
  //  are  < 0x4, the low candidate is selected (i.e. LUT[0:7])
  uint32_t high_bit = (src & 0x88888888) >> 1;

  // `high_bit` is OR'd with 0x31203120 to find the correct value in the LUT
  // (selects correct high or low candidate)
  const uint32_t final_prmt_base = 0x32103210;

  // Ignore the high bit when indexing into LUT, for each 4bit value
  //  we index into both the high and low candidates then use
  //  high_bit | final_prmt_base to select the correct candidate
  uint32_t lut_idx = (src & 0x77777777);

  auto pack = [](uint8_t a, uint8_t b, uint8_t c, uint8_t d) {
    return uint32_t(a) | (uint32_t(b) << 8) | (uint32_t(c) << 16) |
           (uint32_t(d) << 24);
  };

  static constexpr uint32_t LOW_0 = pack(LUT0, LUT1, LUT2, LUT3);
  static constexpr uint32_t LOW_1 = pack(LUT4, LUT5, LUT6, LUT7);
  static constexpr uint32_t HIGH_0 = pack(LUT8, LUT9, LUT10, LUT11);
  static constexpr uint32_t HIGH_1 = pack(LUT12, LUT13, LUT14, LUT15);

  CUTLASS_PRAGMA_UNROLL
  for (int ii = 0; ii < 2; ++ii, lut_idx >>= 16, high_bit >>= 16) {
    uint32_t final_prmt_idx = final_prmt_base | high_bit;

    // This uses a look up table to convert packed int4s to packed int8s,
    // using the int4 value as the index to prmt. It first select both the
    // high and low candidates, then uses the high bit (i.e. `high_bit`) to
    // select the correct candidate.
    asm volatile(
        "{\n"
        "  .reg .b32 low, high;\n"
        "  prmt.b32 low, %1, %2, %5;\n"
        "  prmt.b32 high, %3, %4, %5;\n"
        "  prmt.b32 %0, low, high, %6;\n"
        "}\n"
        : "=r"(r[ii])
        : "n"(LOW_0), "n"(LOW_1), "n"(HIGH_0), "n"(HIGH_1), "r"(lut_idx),
          "r"(final_prmt_idx));
  }

  return r;
};
```
**EN:** `lut_4bit_to_8bit_convert` is a low-level helper that expands eight 4-bit values packed in one register into eight 8-bit values packed in two registers. It uses `prmt.b32` twice: first to gather low/high lookup candidates from compile-time LUT constants, then to select the correct candidate based on each nibble’s high bit.
**CN:** `lut_4bit_to_8bit_convert` 是一个底层工具，用来把单个寄存器里打包的 8 个 4bit 值展开成两个寄存器中的 8 个 8bit 值。它通过两次 `prmt.b32` 实现：先从编译期 LUT 常量中取出低半区/高半区候选值，再根据每个 nibble 的高位选择正确结果。

### u4b8 to INT8 and FP8 / u4b8 到 INT8 与 FP8
```cpp
template <FloatRoundStyle Round, int N>
struct NumericArrayConverter<int8_t, vllm_uint4b8_t, N, Round> {
  using result_type = Array<int8_t, N>;
  using source_type = Array<vllm_uint4b8_t, N>;

  static FloatRoundStyle const round_style = Round;

 private:
  struct RegConvert {
    template <typename PackedResultType>
    CUTLASS_DEVICE static PackedResultType convert(Array<uint32_t, 1> src_) {
      // [-8, -7, -6, -5, -4, -3, -2, -1, 0, 1, 2, 3, 4, 5, 6, 7] as int8s
      auto r = lut_4bit_to_8bit_convert<0xF8, 0xF9, 0xFA, 0xFB,  //
                                        0xFC, 0xFD, 0xFE, 0xFF,  //
                                        0x00, 0x01, 0x02, 0x03,  //
                                        0x04, 0x05, 0x06, 0x07>(src_[0]);
      return reinterpret_cast<PackedResultType&>(r);
    };
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};

// for Array<cutlass::float_e4m3_t, N> <= Array<vllm_uint4b8_t, N>
template <FloatRoundStyle Round, int N>
struct NumericArrayConverter<cutlass::float_e4m3_t, vllm_uint4b8_t, N, Round> {
  using result_type = Array<cutlass::float_e4m3_t, N>;
  using source_type = Array<vllm_uint4b8_t, N>;

  static FloatRoundStyle const round_style = Round;

 private:
  struct RegConvert {
    template <typename PackedResultType>
    CUTLASS_DEVICE static PackedResultType convert(Array<uint32_t, 1> src_) {
      // [-8, -7, -6, -5, -4, -3, -2, -1, 0, 1, 2, 3, 4, 5, 6, 7] as fp8s
      auto r = lut_4bit_to_8bit_convert<0xD0, 0xCE, 0xCC, 0xCA,  //
                                        0xC8, 0xC4, 0xC0, 0xB8,  //
                                        0x00, 0x38, 0x40, 0x44,  //
                                        0x48, 0x4A, 0x4C, 0x4E>(src_[0]);
      return reinterpret_cast<PackedResultType&>(r);
    };
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};
```
**EN:** These specializations convert biased 4-bit values (`x + 8` storage) into signed int8 or FP8 E4M3. The core idea is to reuse the LUT expander with different compile-time tables: one table maps to integer values `[-8, 7]`, the other maps directly to the byte encodings of the corresponding FP8 numbers.
**CN:** 这些特化负责把带偏置的 4bit 值（存储的是 `x + 8`）转换成有符号 int8 或 FP8 E4M3。核心思路都是复用前面的 LUT 展开器，只是使用不同的编译期映射表：一张表映射到整数 `[-8, 7]`，另一张表则直接映射到对应 FP8 数值的字节编码。

### u4b8 to FP16 via bit tricks / u4b8 到 FP16 的位运算快路径
```cpp
// for Array<cutlass::half_t, N> <= Array<vllm_uint4b8_t, N>
template <FloatRoundStyle Round, int N>
struct NumericArrayConverter<cutlass::half_t, vllm_uint4b8_t, N, Round> {
  using result_type = Array<cutlass::half_t, N>;
  using source_type = Array<vllm_uint4b8_t, N>;

  struct RegConvert {
    template <typename PackedResultType>
    CUTLASS_DEVICE static PackedResultType convert(Array<uint32_t, 1> src_) {
      uint32_t src = src_[0];
      using RegArray =
          cutlass::AlignedArray<uint32_t, PackedResultType::kElements / 2,
                                sizeof(PackedResultType)>;
      RegArray r;

      // Below constructs the following temporary:
      // fp16s_01 = {0x00, i4_01, 0x00, i4_01}
      // fp16s_23 = {0x00, i4_23, 0x00, i4_23}
      // fp16s_45 = {0x00, i4_45, 0x00, i4_45}
      // fp16s_67 = {0x00, i4_67, 0x00, i4_67}
      // We use inline asm instead of __byte_perm intrinsic since we don't want
      // the documented (& 0x7) on the index. NVCC might be able to optimize it
      // out since the index is a constexpr, but we choose to be safe about it
      // here.
      uint32_t prmt_indices[4] = {0x4040, 0x4141, 0x4242, 0x4343};
      static_assert(RegArray::kElements <= 4,
                    "Too many inputs for F16 -> I4 vector converter");
      CUTLASS_PRAGMA_UNROLL
      for (int ii = 0; ii < RegArray::kElements; ++ii) {
        asm volatile(
            "{\n"
            "  prmt.b32 %0, %1, %2, %3;\n"
            "}\n"
            : "=r"(r[ii])
            : "r"(src), "n"(0), "r"(prmt_indices[ii]));
      }

      // Since the stored 4bit values are biased by 8 we get stored_val = (x+8)
      //  we are trying to construct x and a fp16 value
      // The below XOR does the following:
      //  1) Sets the exponent bits of the FP16 to the correct value for the
      //  FP16 magic_num. We will be constructing {1024+16*(x1+8), 1024+(x0+8)},
      //  where x1 in the high nibble and x0 is the low nibble then using hfma
      //  to subtract 1032 from that
      // The AND does the following:
      //  1) Clear the set bits for the int4 we will ignore.
      // We use lop3 so that we can use 1 instruction for AND and XOR.
      static constexpr uint32_t xor_mask = 0x64006400;
      static constexpr uint32_t and_mask = 0xFFF0FF0F;
      static constexpr uint32_t immLut = (0xf0 & 0xcc) ^ 0xaa;

      // For each operand, computes:
      // r[i] = (r[i] & and_mask) ^ xor_mask
      CUTLASS_PRAGMA_UNROLL
      for (int ii = 0; ii < RegArray::kElements; ++ii) {
        asm volatile(
            "{\n"
            "  lop3.b32 %0, %0, %1, %2, %3;\n"
            "}\n"
            : "+r"(r[ii])
            : "n"(and_mask), "n"(xor_mask), "n"(immLut));
      }

      // We will issue 2 hfmas that do the following:
      // {x1, x0} = {1024+16*(x1+8), 1024+(x0+8)} * {1/16, 1} - {72, 1032}
      //          = {x1 + 1152, x0 + 1032} * {1/16, 1} - {72, 1032}
      static constexpr uint32_t hfma_bias_rep = 0xD480E408;   // {72, 1032}
      static constexpr uint32_t hfma_scale_rep = 0x2C003C00;  // {1 / 16, 1}

      const half2& hfma_bias = reinterpret_cast<const half2&>(hfma_bias_rep);
      const half2& hfma_scale = reinterpret_cast<const half2&>(hfma_scale_rep);
      CUTLASS_PRAGMA_UNROLL
      for (int ii = 0; ii < RegArray::kElements; ++ii) {
        half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii]);
        fp16x2_val = __hfma2(hfma_scale, fp16x2_val, hfma_bias);
      }

      return reinterpret_cast<PackedResultType&>(r);
    };
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};
```
**EN:** The FP16 path constructs temporary half-precision bit patterns with `prmt`, then uses `lop3` to inject the correct exponent bits and keep only the desired nibbles. Finally it applies `__hfma2` with carefully chosen scale and bias constants to turn those temporary values into real signed FP16 numbers. This avoids scalar unpack-and-convert loops.
**CN:** FP16 路径先用 `prmt` 构造临时半精度位模式，再用 `lop3` 写入正确的指数位并屏蔽无关 nibble，最后借助精心设计的缩放与偏置常量，通过 `__hfma2` 一次把这些临时值变成真正的有符号 FP16。这样就避免了逐元素拆包和标量转换循环。

### Interleaved FP16 for biased u4 / 交错布局下的 biased u4 到 FP16
```cpp
template <FloatRoundStyle Round, int N>
struct InterleavedNumericArrayConverter<Layout<Shape<_2, _4>, Stride<_4, _1>>,
                                        cutlass::half_t, vllm_uint4b8_t, N,
                                        Round, void> {
  using IlvdLayout = Layout<Shape<_2, _4>, Stride<_4, _1>>;
  static_assert(N % size(IlvdLayout{}) == 0);

  using result_type = Array<cutlass::half_t, N>;
  using source_type = Array<vllm_uint4b8_t, N>;

  static FloatRoundStyle const round_style = Round;

 private:
  struct RegConvert {
    template <typename PackedResultType>
    CUTLASS_DEVICE static PackedResultType convert(Array<uint32_t, 1> src_) {
      uint32_t src = src_[0];
      using RegArray =
          cutlass::AlignedArray<uint32_t, PackedResultType::kElements / 2,
                                sizeof(PackedResultType)>;
      RegArray r;

      static_assert(PackedResultType::kElements <= size(IlvdLayout{}));
      static constexpr uint32_t xor_mask = 0x64006400;

      for (int ii = 0; ii < RegArray::kElements; ii += 2) {
        auto src_ = src >> (4 * (ii));
        r[ii + 0] = src_;
        r[ii + 1] = src_;

        static constexpr uint32_t and_xor_imm_lut = (0xf0 & 0xcc) ^ 0xaa;

        static constexpr uint32_t low_nib_mask = 0x000F000F;
        static constexpr uint32_t high_nib_mask = 0x00F000F0;

        asm volatile(
            "{\n"
            "  lop3.b32 %0, %0, %1, %2, %3;\n"
            "}\n"
            : "+r"(r[ii + 0])
            : "n"(low_nib_mask), "n"(xor_mask), "n"(and_xor_imm_lut));

        asm volatile(
            "{\n"
            "  lop3.b32 %0, %0, %1, %2, %3;\n"
            "}\n"
            : "+r"(r[ii + 1])
            : "n"(high_nib_mask), "n"(xor_mask), "n"(and_xor_imm_lut));

        // For low nibble:
        //  {x1, x0} = {1024+(x1+8), 1024+(x0+8)} * {1, 1} - {1032, 1032}
        // For high nibble:
        //  {x1, x0} = {1024+16*(x1+8), 1024+16*(x0+8)} * {1/16, 1/16}
        //             - {72, 72}
        static constexpr uint32_t low_nib_bias = 0x64086408;    // {1032, 1032}
        static constexpr uint32_t high_nib_scale = 0x2C002C00;  // {1/16, 1/16}
        static constexpr uint32_t high_nib_bias = 0xD480D480;   // {-72, -72}

        {
          half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii + 0]);
          fp16x2_val =
              __hsub2(fp16x2_val, reinterpret_cast<const half2&>(low_nib_bias));
        }

        {
          half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii + 1]);
          fp16x2_val = __hfma2(fp16x2_val,
                               reinterpret_cast<const half2&>(high_nib_scale),
                               reinterpret_cast<const half2&>(high_nib_bias));
        }
      }

      return reinterpret_cast<PackedResultType&>(r);
    };
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};
```
**EN:** This specialization handles the CuTe layout `(2,4):(4,1)` and assumes the source is interleaved. Instead of unpacking the whole register in logical order, it processes low and high nibbles separately so the output FP16 values already appear in the required deinterleaved order. Bias handling differs between low and high nibbles, so the code uses separate masks and arithmetic for each half.
**CN:** 这个特化针对 CuTe 布局 `(2,4):(4,1)`，并假设源数据本身是交错存储的。它不会先完整按逻辑顺序解包寄存器，而是分别处理低 nibble 和高 nibble，使输出 FP16 在生成时就已经处于所需的反交错顺序。由于低/高 nibble 的偏置处理方式不同，代码分别使用了不同的掩码和算术常量。

### Interleaved FP16 for unbiased u4 / 交错布局下的无偏置 u4 到 FP16
```cpp
template <FloatRoundStyle Round, int N>
struct InterleavedNumericArrayConverter<Layout<Shape<_2, _4>, Stride<_4, _1>>,
                                        cutlass::half_t, uint4_t, N, Round,
                                        void> {
  using IlvdLayout = Layout<Shape<_2, _4>, Stride<_4, _1>>;
  static_assert(N % size(IlvdLayout{}) == 0);

  using result_type = Array<cutlass::half_t, N>;
  using source_type = Array<uint4_t, N>;

  static FloatRoundStyle const round_style = Round;

 private:
  struct RegConvert {
    template <typename PackedResultType>
    CUTLASS_DEVICE static PackedResultType convert(Array<uint32_t, 1> src_) {
      uint32_t src = src_[0];
      using RegArray =
          cutlass::AlignedArray<uint32_t, PackedResultType::kElements / 2,
                                sizeof(PackedResultType)>;
      RegArray r;

      static_assert(PackedResultType::kElements <= size(IlvdLayout{}));
      static constexpr uint32_t xor_mask = 0x64006400;

      for (int ii = 0; ii < RegArray::kElements; ii += 2) {
        auto src_ = src >> (4 * (ii));
        r[ii + 0] = src_;
        r[ii + 1] = src_;

        static constexpr uint32_t and_xor_imm_lut = (0xf0 & 0xcc) ^ 0xaa;

        static constexpr uint32_t low_nib_mask = 0x000F000F;
        static constexpr uint32_t high_nib_mask = 0x00F000F0;

        asm volatile(
            "{\n"
            "  lop3.b32 %0, %0, %1, %2, %3;\n"
            "}\n"
            : "+r"(r[ii + 0])
            : "n"(low_nib_mask), "n"(xor_mask), "n"(and_xor_imm_lut));

        asm volatile(
            "{\n"
            "  lop3.b32 %0, %0, %1, %2, %3;\n"
            "}\n"
            : "+r"(r[ii + 1])
            : "n"(high_nib_mask), "n"(xor_mask), "n"(and_xor_imm_lut));

        // For low nibble:
        //  {x1, x0} = {1024+x1, 1024+x0} - {1024, 1024}
        // For high nibble:
        //  {x1, x0} = {1024+16*x1, 1024+16*x0} * {1/16, 1/16} - {64, 64}
        static constexpr uint32_t low_nib_bias = 0x64006400;    // {1024, 1024}
        static constexpr uint32_t high_nib_scale = 0x2C002C00;  // {1/16, 1/16}
        static constexpr uint32_t high_nib_bias = 0xD400D400;   // {-64, -64}

        {
          half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii + 0]);
          fp16x2_val =
              __hsub2(fp16x2_val, reinterpret_cast<const half2&>(low_nib_bias));
        }

        {
          half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii + 1]);
          fp16x2_val = __hfma2(fp16x2_val,
                               reinterpret_cast<const half2&>(high_nib_scale),
                               reinterpret_cast<const half2&>(high_nib_bias));
        }
      }

      return reinterpret_cast<PackedResultType&>(r);
    };
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};
```
**EN:** The `uint4_t` version mirrors the previous specialization but removes the `+8` storage bias. The constants therefore change from the biased case: low nibbles subtract 1024, while high nibbles use a different negative bias after scaling by `1/16`.
**CN:** `uint4_t` 版本与前一个特化结构相同，但去掉了 `+8` 的存储偏置，因此常量也随之变化：低 nibble 直接减去 1024，高 nibble 则在乘以 `1/16` 后使用另一组负偏置常量。

### u8b128 to FP16 and FP32 / u8b128 到 FP16 与 FP32
```cpp
template <FloatRoundStyle Round, int N>
struct NumericArrayConverter<cutlass::half_t, vllm_uint8b128_t, N, Round> {
  using result_type = Array<cutlass::half_t, N>;
  using source_type = Array<vllm_uint8b128_t, N>;

  struct RegConvert {
    template <typename PackedResultType>
    CUTLASS_DEVICE static PackedResultType convert(Array<uint32_t, 1> src_) {
      uint32_t src = src_[0];
      // Hold output FP16s in reg. We need 1 reg for every 2 elements
      using RegArray =
          cutlass::AlignedArray<uint32_t, PackedResultType::kElements / 2,
                                sizeof(PackedResultType)>;
      RegArray r;

      uint32_t const prmt_indices[2] = {0x5150, 0x5352};
      static constexpr uint32_t start_byte_for_fp16 = 0x64646464;

      for (int ii = 0; ii < RegArray::kElements; ++ii) {
        asm volatile("prmt.b32 %0,%1,%2,%3;\n"
                     : "=r"(r[ii])
                     : "r"(src), "n"(start_byte_for_fp16),
                       "r"(prmt_indices[ii]));
      }

      // -128 is folded into bias subtraction, i.e. the 0x80 in the low bytes
      static constexpr uint32_t bias_rep = 0x64806480;
      const half2& bias = reinterpret_cast<const half2&>(bias_rep);
      CUTLASS_PRAGMA_UNROLL
      for (int ii = 0; ii < RegArray::kElements; ++ii) {
        half2& fp16x2_val = reinterpret_cast<__half2&>(r[ii]);
        fp16x2_val = __hsub2(fp16x2_val, bias);
      }

      return reinterpret_cast<PackedResultType&>(r);
    };
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};

// for Array<cutlass::float, N> <= Array<vllm_uint8b128_t, N>
template <FloatRoundStyle Round, int N>
struct NumericArrayConverter<float, vllm_uint8b128_t, N, Round> {
  using result_type = Array<float, N>;
  using source_type = Array<vllm_uint8b128_t, N>;
  static FloatRoundStyle const round_style = Round;

 private:
  struct RegConvert {
    template <typename PackedResultType>
    CUTLASS_DEVICE static PackedResultType convert(Array<uint32_t, 1> src_) {
      uint32_t src = src_[0];
      PackedResultType r;

      // __byte_perm simulates the add.u32 0x4B000000 to every u8 element of
      // u8x4 source and stores the result in r (without introducing extra
      // cvt.u32.u8 instruction)
      uint32_t const prmt_indices[4] = {0x7650, 0x7651, 0x7652, 0x7653};
      uint32_t* result_as_int = reinterpret_cast<uint32_t*>(&r);
      for (int ii = 0; ii < PackedResultType::kElements; ++ii) {
        result_as_int[ii] = __byte_perm(src, 0x4B000000, prmt_indices[ii]);
        // Subtract the magic number 0x4B000000 from tmp in floating-point
        // arithmetic to obtain final result
        r[ii] -= (8388608.f + 128.f);  // fold in -128 bias
      }

      return r;
    };
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};
```
**EN:** For biased unsigned 8-bit values, the half-precision path uses `prmt` to place bytes into FP16 mantissa positions, then subtracts a folded `-128` bias in half2 arithmetic. The float path uses `__byte_perm` to synthesize IEEE float bit patterns around the magic constant `0x4B000000`, then subtracts `8388608 + 128` in floating-point to recover the final value.
**CN:** 对于带偏置的无符号 8bit 值，半精度路径使用 `prmt` 把字节放入 FP16 mantissa 位置，再通过 half2 算术减去折叠进来的 `-128` 偏置。单精度路径则利用 `__byte_perm` 围绕魔数 `0x4B000000` 合成 IEEE float 位模式，然后在浮点运算中减去 `8388608 + 128`，还原最终数值。

### SM80 BF16 conversions / SM80 上的 BF16 转换
```cpp
template <FloatRoundStyle Round, int N>
struct NumericArrayConverter<cutlass::bfloat16_t, vllm_uint4b8_t, N, Round> {
  using result_type = Array<cutlass::bfloat16_t, N>;
  using source_type = Array<vllm_uint4b8_t, N>;

  static FloatRoundStyle const round_style = Round;

 private:
  struct RegConvert {
    template <typename PackedResultType>
    CUTLASS_DEVICE static PackedResultType convert(Array<uint32_t, 1> src_) {
      uint32_t src_reg = src_[0];
      // Hold output BF16s in reg. We need 1 reg for every 2 elements
      using RegArray =
          cutlass::AlignedArray<uint32_t, PackedResultType::kElements / 2,
                                sizeof(PackedResultType)>;
      RegArray r;
      uint32_t src_reg_shifted = src_reg >> 4;

      // Below constructs the following temporary:
      uint32_t const prmt_indices[4] = {0xF4F0, 0xF5F1, 0xF6F2, 0xF7F3};
      static_assert(RegArray::kElements <= 4,
                    "Too many inputs for uint4b8_t -> BF16 vector converter");
      CUTLASS_PRAGMA_UNROLL
      for (int ii = 0; ii < RegArray::kElements; ++ii) {
        asm volatile(
            "{\n"
            "  prmt.b32 %0, %1, %2, %3;\n"
            "}\n"
            : "=r"(r[ii])
            : "r"(src_reg), "r"(src_reg_shifted), "r"(prmt_indices[ii]));
      }

      // Since the stored 4bit values are biased by 8 we get stored_val = (x+8)
      //  we are trying to construct x and a BF16 value
      // The below XOR does the following:
      //  1) Sets the exponent bits of the BF16 to the correct value for the
      //  BF16 magic_num. We will be constructing {128 + (x1+8), 128 + (x0+8)}
      //  and subtracting 136 to get {x1, x0}
      static constexpr uint32_t xor_mask = 0x43004300;
      static constexpr uint32_t and_mask = 0x000F000F;
      static constexpr uint32_t immLut = (0xf0 & 0xcc) ^ 0xaa;

      // For each operand, computes:
      // r[i] = (r[i] & and_mask) ^ xor_mask
      CUTLASS_PRAGMA_UNROLL
      for (int ii = 0; ii < RegArray::kElements; ++ii) {
        asm volatile(
            "{\n"
            "  lop3.b32 %0, %0, %1, %2, %3;\n"
            "}\n"
            : "+r"(r[ii])
            : "n"(and_mask), "n"(xor_mask), "n"(immLut));
      }

      // We will issue 2 bfmas that do the following:
      // high BF16:
      // hi_bf16 - 136, lo_bf16 - 136

      // This is the BF16 {136, 136} represented as an integer.
      static constexpr uint32_t bias_rep = 0x43084308;
      const __nv_bfloat162& bias =
          reinterpret_cast<const __nv_bfloat162&>(bias_rep);

      CUTLASS_PRAGMA_UNROLL
      for (int ii = 0; ii < RegArray::kElements; ++ii) {
        __nv_bfloat162& bf16x2_val = reinterpret_cast<__nv_bfloat162&>(r[ii]);
        bf16x2_val = __hsub2(bf16x2_val, bias);
      }

      return reinterpret_cast<PackedResultType&>(r);
    }
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};
```
**EN:** BF16 support is guarded by `__CUDA_ARCH__ >= 800` because it relies on Ampere-or-newer instructions and BF16 vector types. The direct `vllm_uint4b8_t -> bfloat16_t` path is analogous to the FP16 version, but it uses BF16-specific bit layouts, `__nv_bfloat162`, and a simpler “subtract 136” bias correction.
**CN:** BF16 支持被 `__CUDA_ARCH__ >= 800` 宏保护，因为它依赖 Ampere 及更新架构上的指令和 BF16 向量类型。直接的 `vllm_uint4b8_t -> bfloat16_t` 路径与 FP16 版本思路相近，但使用 BF16 专属位布局、`__nv_bfloat162`，并采用更简单的“减去 136”偏置修正。

### Interleaved BF16 paths / 交错布局下的 BF16 路径
```cpp
template <FloatRoundStyle Round, int N>
struct InterleavedNumericArrayConverter<Layout<Shape<_2, _4>, Stride<_4, _1>>,
                                        cutlass::bfloat16_t, vllm_uint4b8_t, N,
                                        Round, void> {
  using IlvdLayout = Layout<Shape<_2, _4>, Stride<_4, _1>>;
  static_assert(N % size(IlvdLayout{}) == 0);

  using result_type = Array<cutlass::bfloat16_t, N>;
  using source_type = Array<vllm_uint4b8_t, N>;

 private:
  struct RegConvert {
    template <typename PackedResultType>
    CUTLASS_DEVICE static PackedResultType convert(Array<uint32_t, 1> src_) {
      uint32_t src = src_[0];
      using RegArray =
          cutlass::AlignedArray<uint32_t, PackedResultType::kElements / 2,
                                sizeof(PackedResultType)>;
      RegArray r;

      static_assert(PackedResultType::kElements <= size(IlvdLayout{}));
      static constexpr uint32_t or_mask = 0x43004300;

      // Unlike float16 where the mantissa is large enough to contain 2
      // nibbles, bfloat16 can only fit one, so we can only convert one
      // nibble at a time
      for (int ii = 0; ii < RegArray::kElements; ++ii) {
        r[ii] = src >> (4 * ii);

        static constexpr uint32_t and_or_imm_lut = (0xf0 & 0xcc) | 0xaa;
        static constexpr uint32_t low_nib_mask = 0x000F000F;

        asm volatile(
            "{\n"
            "  lop3.b32 %0, %0, %1, %2, %3;\n"
            "}\n"
            : "+r"(r[ii + 0])
            : "n"(low_nib_mask), "n"(or_mask), "n"(and_or_imm_lut));

        // For low nibble:
        //  {x1, x0} = {128+(x1+8), 128+(x0+8)} * {1, 1} - {136, 136}
        static constexpr uint32_t low_nib_bias = 0x43084308;  // {136, 136}

        {
          __nv_bfloat162& fp16x2_val = reinterpret_cast<__nv_bfloat162&>(r[ii]);
          fp16x2_val =
              __hsub2(fp16x2_val,
                      reinterpret_cast<const __nv_bfloat162&>(low_nib_bias));
        }
      }

      return reinterpret_cast<PackedResultType&>(r);
    };
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};

// for Array<cutlass::bfloat16_t, N> <= Array<uint4_t, N>
//   for IlvdLayout: (2, 4):(4, 1)
template <FloatRoundStyle Round, int N>
struct InterleavedNumericArrayConverter<Layout<Shape<_2, _4>, Stride<_4, _1>>,
                                        cutlass::bfloat16_t, uint4_t, N, Round,
                                        void> {
  using IlvdLayout = Layout<Shape<_2, _4>, Stride<_4, _1>>;
  static_assert(N % size(IlvdLayout{}) == 0);

  using result_type = Array<cutlass::bfloat16_t, N>;
  using source_type = Array<uint4_t, N>;

 private:
  struct RegConvert {
    template <typename PackedResultType>
    CUTLASS_DEVICE static PackedResultType convert(Array<uint32_t, 1> src_) {
      uint32_t src = src_[0];
      using RegArray =
          cutlass::AlignedArray<uint32_t, PackedResultType::kElements / 2,
                                sizeof(PackedResultType)>;
      RegArray r;

      static_assert(PackedResultType::kElements <= size(IlvdLayout{}));
      static constexpr uint32_t or_mask = 0x43004300;

      // Unlike float16 where the mantissa is large enough to contain 2
      // nibbles, bfloat16 can only fit one, so we can only convert one
      // nibble at a time
      for (int ii = 0; ii < RegArray::kElements; ++ii) {
        r[ii] = src >> (4 * ii);

        static constexpr uint32_t and_or_imm_lut = (0xf0 & 0xcc) | 0xaa;
        static constexpr uint32_t low_nib_mask = 0x000F000F;

        asm volatile(
            "{\n"
            "  lop3.b32 %0, %0, %1, %2, %3;\n"
            "}\n"
            : "+r"(r[ii])
            : "n"(low_nib_mask), "n"(or_mask), "n"(and_or_imm_lut));

        // For low nibble:
        //  {x1, x0} = {128 + x1, 128 + x0} * {1, 1} - {128, 128}
        static constexpr uint32_t low_nib_bias = 0x43004300;  // {128, 128}

        {
          __nv_bfloat162& fp16x2_val = reinterpret_cast<__nv_bfloat162&>(r[ii]);
          fp16x2_val =
              __hsub2(fp16x2_val,
                      reinterpret_cast<const __nv_bfloat162&>(low_nib_bias));
        }
      }

      return reinterpret_cast<PackedResultType&>(r);
    };
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};
```
**EN:** The interleaved BF16 specializations exist for both biased `vllm_uint4b8_t` and unbiased `uint4_t`. Because BF16 has a much smaller mantissa than FP16, the converter cannot pack two nibbles into one BF16 construction step; it shifts the source and converts one nibble pair at a time, then subtracts either 136 or 128 depending on whether the source was biased.
**CN:** 交错 BF16 特化同时覆盖带偏置的 `vllm_uint4b8_t` 和无偏置的 `uint4_t`。由于 BF16 的 mantissa 比 FP16 更小，转换器无法像 FP16 那样在一次构造中处理两个 nibble，因此它通过右移源寄存器逐次处理 nibble 对，并根据源类型是否带偏置分别减去 136 或 128。

### u8b128 to BF16 via float / u8b128 先转 float 再转 BF16
```cpp
// for Array<cutlass::bfloat16_t, N> <= Array<vllm_uint8b128_t, N>
template <FloatRoundStyle Round, int N>
struct NumericArrayConverter<cutlass::bfloat16_t, vllm_uint8b128_t, N, Round> {
  using result_type = Array<cutlass::bfloat16_t, N>;
  using source_type = Array<vllm_uint8b128_t, N>;
  static FloatRoundStyle const round_style = Round;

 private:
  using result_packed_4_t = Array<cutlass::bfloat16_t, 4>;
  using result_packed_2_t = Array<cutlass::bfloat16_t, 2>;
  using src_packed_4_t = Array<vllm_uint8b128_t, 4>;
  using src_packed_2_t = Array<vllm_uint8b128_t, 2>;

  // Not Valid, not supported, only here to satisfy the interface and to avoid
  //  a compile error. ScalarConverter will not actually work until
  //  NumericConverter<cutlass::bfloat16_t, vllm_uint8b128_t, Round> is
  //  implemented
  using ScalarConverter =
      NumericConverter<cutlass::bfloat16_t, vllm_uint8b128_t, Round>;

  template <typename PackedResultType, typename PackedSrcType>
  CUTLASS_DEVICE static PackedResultType packed_convert(
      PackedSrcType const& source) {
    static_assert(
        (platform::is_same<PackedSrcType, src_packed_2_t>::value &&
         platform::is_same<PackedResultType, result_packed_2_t>::value) ||
            (platform::is_same<PackedSrcType, src_packed_4_t>::value &&
             platform::is_same<PackedResultType, result_packed_4_t>::value),
        "Invalid PackedSrcType/PackedResultType must be 2 or 4 to use private "
        "convert dispatch.");

    NumericArrayConverter<float, vllm_uint8b128_t, PackedResultType::kElements,
                          Round>
        convert_uint8_to_f32;
    Array<float, PackedResultType::kElements> tmp =
        convert_uint8_to_f32(source);
    NumericArrayConverter<cutlass::bfloat16_t, float,
                          PackedResultType::kElements, Round>
        convert_f32_to_bf16_;
    return convert_f32_to_bf16_(tmp);
  }

  friend class detail::VectorizedConverter;

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    result_type result;
    using ConverterType =
        NumericArrayConverter<typename result_type::Element,
                              typename source_type::Element, N, Round>;
    detail::VectorizedConverter::convert<ConverterType, result_packed_4_t,
                                         src_packed_4_t, result_packed_2_t,
                                         src_packed_2_t>(result, source);

    return result;
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};
```
**EN:** Rather than duplicating another bit-manipulation path, the BF16 converter for `vllm_uint8b128_t` composes two existing converters: `uint8b128 -> float`, then `float -> bfloat16`. The specialization only needs to implement packed dispatch for 2- and 4-element vectors and let CUTLASS vectorization utilities handle the outer loop.
**CN:** 对 `vllm_uint8b128_t` 的 BF16 转换没有再复制一套新的位操作快路径，而是直接组合两个现成转换器：先 `uint8b128 -> float`，再 `float -> bfloat16`。因此该特化只需实现 2 元素和 4 元素打包向量的分发，再让 CUTLASS 的向量化工具处理外层循环即可。

### Fast FP16 to INT8 / 快速 FP16 到 INT8
```cpp
template <FloatRoundStyle Round, int N>
struct NumericArrayConverter<int8_t, cutlass::half_t, N, Round> {
  using result_type = Array<int8_t, N>;
  using source_type = Array<cutlass::half_t, N>;

  struct RegConvert {
    // FastFP16toINT8 from https://arxiv.org/pdf/2406.09904
    template <typename PackedResultType, int src_regs>
    CUTLASS_DEVICE static PackedResultType convert(
        Array<uint32_t, src_regs> src) {
      // Hold output int8s in reg. We need 1 reg for every 4 elements
      using RegArray = cutlass::AlignedArray<
          uint32_t, std::max(PackedResultType::kElements / 4, size_t(1))>;
      RegArray r;

      static constexpr uint32_t MAGIC_BIAS_ = 0x64806480;
      auto MAGIC_BIAS = *reinterpret_cast<const half2*>(&MAGIC_BIAS_);

      *reinterpret_cast<half2*>(&src[0]) =
          __hadd2(*reinterpret_cast<half2*>(&src[0]), MAGIC_BIAS);

      if constexpr (src_regs > 1) {
        *reinterpret_cast<half2*>(&src[1]) =
            __hadd2(*reinterpret_cast<half2*>(&src[1]), MAGIC_BIAS);
      }

      static_assert(PackedResultType::kElements <= 4);
      uint32_t uint8s;
      static constexpr uint32_t MASK_0246 = 0x6420;
      static constexpr uint32_t UINT8s_TO_INT8s_MASK = 0x80808080;
      asm volatile("prmt.b32 %0,%1,%2,%3;\n"
                   : "=r"(uint8s)
                   : "r"(src[0]), "r"((src_regs > 1) ? src[1] : src[0]),
                     "n"(MASK_0246));

      uint32_t int8s = (uint8s ^ UINT8s_TO_INT8s_MASK);

      return reinterpret_cast<PackedResultType&>(int8s);
    };
  };

 public:
  CUTLASS_DEVICE
  static result_type convert(source_type const& source) {
    return ArrayConverterPacked32Bit<RegConvert, typename result_type::Element,
                                     typename source_type::Element,
                                     N>::convert(source);
  }

  CUTLASS_DEVICE
  result_type operator()(source_type const& s) const { return convert(s); }
};
```
**EN:** The last specialization implements the FastFP16toINT8 idea cited from the paper. It adds a magic half2 bias, uses `prmt` to gather the converted bytes, and then flips the sign bit with `0x80808080` to obtain signed INT8 values. This is a compact, register-only path for dequantization output requantization.
**CN:** 最后的特化实现了论文中提到的 FastFP16toINT8 思路：先对 half2 加上魔术偏置，再用 `prmt` 收集转换后的字节，最后通过 `0x80808080` 翻转符号位得到有符号 INT8。对于反量化结果再量化，这是一条紧凑的纯寄存器快路径。

## Key Concepts / 关键概念
- Register-level packed conversion instead of scalar unpack loops / 用寄存器级打包转换替代逐元素拆包循环
- Custom converters for biased 4-bit and 8-bit quantized storage / 面向带偏置 4bit 与 8bit 量化存储的自定义转换器
- Interleaving-aware specializations keyed by CuTe layouts / 基于 CuTe 布局的交错感知特化
- FP16/BF16/FP8/INT8 fast paths using `prmt`, `lop3`, `half2`, and `bfloat162` / 使用 `prmt`、`lop3`、`half2`、`bfloat162` 的 FP16/BF16/FP8/INT8 快路径

## Dependencies / 依赖关系
- `cutlass/numeric_conversion.h` provides the base `NumericArrayConverter`, `NumericConverter`, and vectorized helpers / `cutlass/numeric_conversion.h` 提供基础 `NumericArrayConverter`、`NumericConverter` 和向量化辅助设施
- `vllm_custom_types.cuh` defines `vllm_uint4b8_t` and `vllm_uint8b128_t` / `vllm_custom_types.cuh` 定义 `vllm_uint4b8_t` 与 `vllm_uint8b128_t`
- `cute_utils.cuh` contributes identity-layout detection used by interleaved specializations / `cute_utils.cuh` 提供交错特化所用的恒等布局判断
- `vllm_type_utils.cuh` supplies type-name strings for fallback diagnostics / `vllm_type_utils.cuh` 为兜底诊断提供类型名字字符串
