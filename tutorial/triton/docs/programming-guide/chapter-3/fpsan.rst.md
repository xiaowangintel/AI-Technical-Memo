# fpsan.rst — Documentation Analysis / 文档分析

## Source / 来源
- **Path:** `/root/xw/triton/docs/programming-guide/chapter-3/fpsan.rst`
- **EN:** Programming-guide chapter page for fpsan.rst.
- **CN:** fpsan.rst 对应的编程指南章节页面。

## Content Analysis / 内容分析
### Floating-Point Sanitizer (FpSan)
**EN:** This section introduces FpSan as a compiler instrumentation mode for structure-preserving kernel checking. Key listed points include maps floating-point bit-patterns into an integer payload domain, replaces supported floating-point ops with integer-domain rewrites chosen to, and maps the resulting payload back into a floating-point bit-pattern so the rest. Notable prose emphasis: FpSan is a compiler instrumentation mode that rewrites selected floating-point Triton IR operations into deterministic "payload algebra" over integer bit-patterns. Its goal is not to approximate IEEE floating-point arithmetic.
**CN:** 本节将 FpSan 介绍为一种用于结构保持型内核检查的编译器插桩模式。 列出的重点包括 maps floating-point bit-patterns into an integer payload domain、replaces supported floating-point ops with integer-domain rewrites chosen to、maps the resulting payload back into a floating-point bit-pattern so the rest。 其中反复出现的技术关键词包括 floating-point、FpSan、rewrites、selected、payload、wrong。

### Enabling FpSan
**EN:** This section explains how to turn on FpSan from Python or the shell and clarifies current support limits. Key listed points include FpSan is a compiler feature, so it does not apply in interpreter mode. and On AMD, the backend currently enables FpSan only for gfx942, gfx950,. Notable prose emphasis: Enable FpSan before the compile or run you want to instrument. From Python import triton triton.knobs.compilation.instrumentation_mode = "fpsan" # compile and run kernels here triton.knobs.compilation.instrumentation_mode = "" From the shell TRITON_INSTRUMENTATION_MODE=fpsan python your_script.py Notes and gfx1250.
**CN:** 本节说明如何从 Python 或 shell 启用 FpSan，并解释当前支持范围。 列出的重点包括 FpSan is a compiler feature, so it does not apply in interpreter mode.、On AMD, the backend currently enables FpSan only for gfx942, gfx950,。 其中反复出现的技术关键词包括 gfx942、gfx950、gfx1250、FpSan、compile、run。

### How to Use It
**EN:** This section frames FpSan as a comparison tool for checking whether two kernels preserve the same symbolic computation. Key listed points include comparing an optimized kernel against a simple reference kernel, comparing a fused kernel against an unfused composition, comparing two schedule variants that should be mathematically equivalent, and checking that accumulator selection, predication, or TMEM pipelines preserve. Notable prose emphasis: The most effective way to use FpSan is to compare two kernels, or two versions of one kernel, under the same FpSan mode. Typical uses include the intended payload flow FpSan results should only be compared against other FpSan results, not against ordinary floating-point outputs.
**CN:** 本节将 FpSan 定位为比较工具，用于检查两个内核是否保持相同的符号计算。 列出的重点包括 comparing an optimized kernel against a simple reference kernel、comparing a fused kernel against an unfused composition、comparing two schedule variants that should be mathematically equivalent、checking that accumulator selection, predication, or TMEM pipelines preserve。 其中反复出现的技术关键词包括 FpSan、kernel、against、two、comparing、should。

### Payload Model
**EN:** This section defines the integer payload domain that FpSan uses to reinterpret floating-point values. Key listed points include embed(x) maps a float bit-pattern to an integer payload, unembed(u) maps an integer payload back to a float bit-pattern, sanitized float ops are implemented as unembed(F(embed(...))), embed(+0.0) = 0, embed(+1.0) = 1, and embed(-1.0) = all-ones. Notable prose emphasis: For each floating-point width w, FpSan defines a bijection between floating-point bit-patterns and a w-bit integer payload; arithmetic wraps modulo 2^w. Conceptually The embedding is deliberately chosen so that a few important constants are stable Those fixed points are the reason identities such as x + 0 = x and x 1 = x behave naturally under FpSan.
**CN:** 本节定义了 FpSan 用于重新解释浮点值的整数载荷域。 列出的重点包括 embed(x) maps a float bit-pattern to an integer payload、unembed(u) maps an integer payload back to a float bit-pattern、sanitized float ops are implemented as unembed(F(embed(...)))、embed(+0.0) = 0、embed(+1.0) = 1、embed(-1.0) = all-ones。 其中反复出现的技术关键词包括 w、embed(x)、unembed(u)、unembed(F(embed(...)))、embed(+0.0) = 0、embed(+1.0) = 1。

### What FpSan Preserves
**EN:** This section lists the algebraic identities and dataflow properties that FpSan intentionally preserves. Key listed points include ring identities for add, subtract, unary negation, multiply, FMA, and, selected exponential identities for exp and exp2 (see below for details), trigonometric identities for sin and cos, payload equality through casts, loads, stores, and copies, and deterministic op-distinguishing tags for unary functions that do not yet have. Notable prose emphasis: FpSan preserves exact identities in the payload algebra selected by each rewrite. The most important ones are dot-like accumulation a richer algebraic model This is what makes FpSan valuable for kernel checks: if two kernels should be the same symbolic computation under the preserved properties, they should produce the same payloads.
**CN:** 本节列出 FpSan 有意保留的代数恒等式和数据流性质。 列出的重点包括 ring identities for add, subtract, unary negation, multiply, FMA, and、selected exponential identities for exp and exp2 (see below for details)、trigonometric identities for sin and cos、payload equality through casts, loads, stores, and copies、deterministic op-distinguishing tags for unary functions that do not yet have。 其中反复出现的技术关键词包括 exp、exp2、sin、cos、FpSan、identities。

### What FpSan Does Not Preserve
**EN:** This section explicitly bounds FpSan by listing behaviors it does not model faithfully. Key listed points include real floating-point ordering, rounding, NaN propagation, infinities,, real transcendental semantics for log, sqrt, erf, floor,, and expected floating-point bit patterns (i.e. for kernels that bitcast. Notable prose emphasis: FpSan is not an IEEE simulator. In particular, do not rely on it for subnormals, or exceptions ceil, rsqrt, and similar tagged unary ops between floats and integers) When a property matters for your check, the right question is "is this property preserved by the payload rewrite for this specific op family?"
**CN:** 本节通过列举未被忠实建模的行为，明确界定 FpSan 的边界。 列出的重点包括 real floating-point ordering, rounding, NaN propagation, infinities,、real transcendental semantics for log, sqrt, erf, floor,、expected floating-point bit patterns (i.e. for kernels that bitcast。 其中反复出现的技术关键词包括 log、sqrt、erf、floor、ceil、rsqrt。

### Common Arithmetic Ops
**EN:** This section introduces the arithmetic operator families that receive sanitizer-specific rewrites.
**CN:** 本节介绍接受清理器专用重写的算术操作族。

### Add, Sub, Neg, Mul
**EN:** This section documents payload-space rewrites for basic ring-style arithmetic. Key listed points include x + y, x - y, x, x y, add, subtract, negate, or multiply the embedded payloads, then unembed the, and x + 0 = x. Notable prose emphasis: Supported operations Rewrite result Exact preserved properties Important caveat
**CN:** 本节记录基本环式算术在载荷空间中的重写规则。 列出的重点包括 x + y、x - y、x、x y、add, subtract, negate, or multiply the embedded payloads, then unembed the、x + 0 = x。 其中反复出现的技术关键词包括 x + y、x - y、-x、x * y、x + 0 = x、x - 0 = x。

### Min and Max
**EN:** This section explains how minimum and maximum are interpreted over payloads instead of IEEE float ordering. Key listed points include tl.minimum(x, y), tl.maximum(x, y), min(x, y) and max(x, y) in Triton code, signed integer min or max on payloads, idempotence: min(x, x) = x and max(x, x) = x, and commutativity. Notable prose emphasis: Supported operations Rewrite Exact preserved properties Important caveats
**CN:** 本节说明最小值和最大值如何基于载荷顺序而非 IEEE 浮点顺序来解释。 列出的重点包括 tl.minimum(x, y)、tl.maximum(x, y)、min(x, y) and max(x, y) in Triton code、signed integer min or max on payloads、idempotence: min(x, x) = x and max(x, x) = x、commutativity。 其中反复出现的技术关键词包括 tl.minimum(x, y)、tl.maximum(x, y)、min(x, y)、max(x, y)、min、max。

### Division
**EN:** This section describes the sanitizer's algebraic division rewrite and its deliberate caveats. Key listed points include x / y, x / y becomes embed(x) inv(embed(y)), then unembed, the true modular inverse for odd payloads, a parity-preserving involution for even payloads, x / 1 = x, and 1 / (1 / x) = x. Notable prose emphasis: Supported operation Rewrite Here inv is Exact preserved properties Important caveats
**CN:** 本节说明清理器对除法的代数重写及其刻意保留的注意事项。 列出的重点包括 x / y、x / y becomes embed(x) inv(embed(y)), then unembed、the true modular inverse for odd payloads、a parity-preserving involution for even payloads、x / 1 = x、1 / (1 / x) = x。 其中反复出现的技术关键词包括 x / y、embed(x) * inv(embed(y))、unembed、inv、x / 1 = x、1 / (1 / x) = x。

### Remainder
**EN:** This section describes the sanitizer's payload-space remainder rule. Key listed points include x % y, signed integer remainder on payloads after forcing the denominator odd with, same inputs produce the same sanitized remainder payload, Real floating-point remainder semantics are not modeled., and Zero denominators are intentionally mapped to a safe odd payload instead of. Notable prose emphasis: Supported operation Rewrite den | 1 Exact preserved properties Important caveats trapping.
**CN:** 本节说明清理器在载荷空间中的取余规则。 列出的重点包括 x % y、signed integer remainder on payloads after forcing the denominator odd with、same inputs produce the same sanitized remainder payload、Real floating-point remainder semantics are not modeled.、Zero denominators are intentionally mapped to a safe odd payload instead of。 其中反复出现的技术关键词包括 x % y、den | 1、remainder、odd、same、payload。

### FMA
**EN:** This section explains how fused multiply-add is represented under the sanitizer. Key listed points include tl.fma(a, b, c), a b + c in payload arithmetic, exact agreement with the sanitized expansion mul followed by add, fma(a, b, c) = ab + c in the payload ring, and There is no special fused-rounding behavior.. Notable prose emphasis: Supported operation Rewrite Exact preserved properties Important caveat
**CN:** 本节说明融合乘加在清理器下如何表示。 列出的重点包括 tl.fma(a, b, c)、a b + c in payload arithmetic、exact agreement with the sanitized expansion mul followed by add、fma(a, b, c) = ab + c in the payload ring、There is no special fused-rounding behavior.。 其中反复出现的技术关键词包括 tl.fma(a, b, c)、a * b + c、mul、add、fma(a, b, c) = a*b + c、payload。

### Unary Math Ops
**EN:** This section groups unary transcendental and tagged operations under the sanitizer model.
**CN:** 本节将一元超越函数和带标签操作纳入清理器模型统一说明。

### ``exp2``
**EN:** This section discusses ``exp2`` in the context of the source document. Key listed points include tl.exp2(x), modular exponentiation by a fixed odd generator in payload space, exp2(x + y) = exp2(x) exp2(y), exp2(0) = 1, and exp2(-x) = 1.0 / exp2(x). Notable prose emphasis: Supported operation Rewrite Exact preserved properties
**CN:** 本节围绕 ``exp2`` 讨论该文档中的相关内容。 列出的重点包括 tl.exp2(x)、modular exponentiation by a fixed odd generator in payload space、exp2(x + y) = exp2(x) exp2(y)、exp2(0) = 1、exp2(-x) = 1.0 / exp2(x)。 其中反复出现的技术关键词包括 tl.exp2(x)、exp2(x + y) = exp2(x) * exp2(y)、exp2(0) = 1、exp2(-x) = 1.0 / exp2(x)、exp2、Supported。

### ``exp``
**EN:** This section discusses ``exp`` in the context of the source document. Key listed points include tl.exp(x), exp(x) is implemented as exp2(x rcp_log2) in payload space, and exp uses the same payload-space construction as exp2 after scaling. Notable prose emphasis: Supported operation Rewrite Exact preserved properties the input by a fixed internal payload constant
**CN:** 本节围绕 ``exp`` 讨论该文档中的相关内容。 列出的重点包括 tl.exp(x)、exp(x) is implemented as exp2(x rcp_log2) in payload space、exp uses the same payload-space construction as exp2 after scaling。 其中反复出现的技术关键词包括 tl.exp(x)、exp(x)、exp2(x * rcp_log2)、exp、exp2、payload。

### ``sin`` and ``cos``
**EN:** This section discusses ``sin`` and ``cos`` in the context of the source document. Key listed points include tl.sin(x), tl.cos(x), a deterministic payload-space rewrite chosen to preserve the identities below, sin(x + y) = sin(x) cos(y) + cos(x) sin(y), sin(x - y) = sin(x) cos(y) - cos(x) sin(y), and cos(x + y) = cos(x) cos(y) - sin(x) sin(y). Notable prose emphasis: Supported operations Rewrite Exact preserved properties Important caveat the angle identities above.
**CN:** 本节围绕 ``sin`` and ``cos`` 讨论该文档中的相关内容。 列出的重点包括 tl.sin(x)、tl.cos(x)、a deterministic payload-space rewrite chosen to preserve the identities below、sin(x + y) = sin(x) cos(y) + cos(x) sin(y)、sin(x - y) = sin(x) cos(y) - cos(x) sin(y)、cos(x + y) = cos(x) cos(y) - sin(x) sin(y)。 其中反复出现的技术关键词包括 tl.sin(x)、tl.cos(x)、cos(x)^2 + sin(x)^2 = 1、sin、cos、chosen。

### Tagged Unary Ops
**EN:** This section explains the hash/tag strategy used for unary operations without a richer algebraic model. Key listed points include tl.log(x), tl.log2(x), tl.sqrt(x), tl.rsqrt(x), tl.erf(x), and tl.floor(x). Notable prose emphasis: Supported operations Rewrite op-specific hash, then multiply again Exact preserved properties space, then op(x) == op(y) Important caveats such as sqrt(x)^2 = x or log(xy) = log(x) + log(y).
**CN:** 本节说明对于缺少更丰富代数模型的一元操作，系统采用的哈希/标签策略。 列出的重点包括 tl.log(x)、tl.log2(x)、tl.sqrt(x)、tl.rsqrt(x)、tl.erf(x)、tl.floor(x)。 其中反复出现的技术关键词包括 tl.log(x)、tl.log2(x)、tl.sqrt(x)、tl.rsqrt(x)、tl.erf(x)、tl.floor(x)。

### Casts and Format Conversions
**EN:** This section explains how FpSan treats type conversions while preserving payload structure.
**CN:** 本节说明 FpSan 如何在保留载荷结构的同时处理类型转换。

### Float-to-Float Conversions
**EN:** This section covers widening and narrowing conversions between floating-point payloads. Key listed points include converting a tensor between floating-point types with x.to(dtype), implicit float widening and narrowing conversions, signed integer extension or truncation in payload space, followed by, 0, +1, and -1 remain stable across the conversion, sign-extension behavior in the payload domain, and truncation drops high payload bits. Notable prose emphasis: Supported operations Rewrite unembed Exact preserved properties Important caveat precision or range, so for example under fpsan fn(a.to(tl.float16)).to(tl.bfloat16) == fn(a) (for any bfloat16 a).
**CN:** 本节涵盖浮点载荷之间的扩宽与缩窄转换。 列出的重点包括 converting a tensor between floating-point types with x.to(dtype)、implicit float widening and narrowing conversions、signed integer extension or truncation in payload space, followed by、0, +1, and -1 remain stable across the conversion、sign-extension behavior in the payload domain、truncation drops high payload bits。 其中反复出现的技术关键词包括 x.to(dtype)、unembed、0、+1、-1、a。

### Packed fp4 conversion
**EN:** This section describes how packed fp4 storage is unpacked into payload values. Key listed points include unpack low and high nibbles from the source byte tensor, reshape and reorder them, interpret each unpacked nibble directly as a payload in the destination float, deterministic unpacking of packed fp4 storage, exact preservation of the unpacked nibble payloads, and This is not real fp4 numeric decoding.. Notable prose emphasis: Rewrite width Exact preserved properties Important caveat e2m1.
**CN:** 本节说明打包 fp4 存储如何解包为载荷值。 列出的重点包括 unpack low and high nibbles from the source byte tensor、reshape and reorder them、interpret each unpacked nibble directly as a payload in the destination float、deterministic unpacking of packed fp4 storage、exact preservation of the unpacked nibble payloads、This is not real fp4 numeric decoding.。 其中反复出现的技术关键词包括 e2m1、unpacked、nibble、fp4、Rewrite、unpack。

### Pure Extern Elementwise Ops
**EN:** This section explains the deterministic structural tagging applied to pure external elementwise functions. Key listed points include tl.extern_elementwise when all of the following hold, the op is pure, the result type is float-like, there is at least one operand, every operand is numeric, and rotate each operand payload by its argument index. Notable prose emphasis: Supported operation Rewrite Exact preserved properties integer operands are used directly after signed casting to the result width Important caveat
**CN:** 本节说明纯外部逐元素函数所采用的确定性结构标签方案。 列出的重点包括 tl.extern_elementwise when all of the following hold、the op is pure、the result type is float-like、there is at least one operand、every operand is numeric、rotate each operand payload by its argument index。 其中反复出现的技术关键词包括 tl.extern_elementwise、pure、operand、operands、result、all。

### Gluon MMA and Tensor Memory
**EN:** This section extends FpSan to Gluon-specific matrix-multiply and tensor-memory primitives. Key listed points include mma_v2, warpgroup_mma and warpgroup_mma_wait, tcgen05_mma and tcgen05_mma_scaled, tcgen05_copy and tcgen05_commit, allocate_tensor_memory, and tensor-memory descriptor methods such as load, load_min,. Notable prose emphasis: Supported Gluon operations include load_max, store, slice, index, and _reinterpret scaled_upcast Rewrite Exact preserved properties Important caveats operands and scales, not exact hardware-format numeric decoding. substitute for race or synchronization checking.
**CN:** 本节将 FpSan 扩展到 Gluon 专用的矩阵乘与张量内存原语。 列出的重点包括 mma_v2、warpgroup_mma and warpgroup_mma_wait、tcgen05_mma and tcgen05_mma_scaled、tcgen05_copy and tcgen05_commit、allocate_tensor_memory、tensor-memory descriptor methods such as load, load_min,。 其中反复出现的技术关键词包括 mma_v2、warpgroup_mma、warpgroup_mma_wait、tcgen05_mma、tcgen05_mma_scaled、tcgen05_copy。

### Practical Guidance for Checks
**EN:** This section summarizes when FpSan is an effective validation tool and when ordinary numerical testing is still required. Key listed points include that two kernels implement the same preserved algebra, that a fused kernel keeps the intended dataflow, that predication or accumulator-selection logic is wired correctly, that a tensor-memory or warp-specialized pipeline preserves payload flow, IEEE edge cases, and real transcendental accuracy. Notable prose emphasis: FpSan is a good fit when you want to check FpSan is a poor fit when you want to check In short, rely on FpSan for structure-preserving kernel validation, and rely on ordinary numerical tests for IEEE behavior.
**CN:** 本节总结何时适合使用 FpSan 进行验证，以及何时仍需普通数值测试。 列出的重点包括 that two kernels implement the same preserved algebra、that a fused kernel keeps the intended dataflow、that predication or accumulator-selection logic is wired correctly、that a tensor-memory or warp-specialized pipeline preserves payload flow、IEEE edge cases、real transcendental accuracy。 其中反复出现的技术关键词包括 FpSan、fit、you、want、check、kernel。

## Key Concepts / 关键概念
- **EN:** Floating-Point Sanitizer (FpSan)  **CN:** 浮点清理器（FpSan）
- **EN:** Enabling FpSan  **CN:** 启用 FpSan
- **EN:** How to Use It  **CN:** 使用方式
- **EN:** Payload Model  **CN:** 载荷模型
- **EN:** What FpSan Preserves  **CN:** FpSan 保留的性质
- **EN:** What FpSan Does Not Preserve  **CN:** FpSan 不保留的性质

## Related Files / 相关文件
