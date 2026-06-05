# LinearLayout.h — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `./include/triton/Tools/LinearLayout.h`
- **EN:** Declares reusable tool-layer utilities centered on `LinearLayout`.
- **CN:** 声明围绕 `LinearLayout` 的可复用工具层设施。

## Line-by-Line Analysis / 逐行分析

### Lines 1-2
```cpp
   1: #ifndef TRITON_TOOLS_LINEARLAYOUT_H
   2: #define TRITON_TOOLS_LINEARLAYOUT_H
```
**EN:** This block establishes the header guard so the declarations in this file are only processed once per translation unit.
**CN:** 该代码块建立头文件保护宏，避免同一翻译单元重复包含本文件时出现重复声明。

### Lines 4-9
```cpp
   4: #include <cstdint>
   5: #include <numeric>
   6: #include <ostream>
   7: #include <string>
   8: #include <utility>
   9: #include <vector>
```
**EN:** This block imports the direct dependencies needed here, including <cstdint>, <numeric>, <ostream>, <string>, <utility>, and <vector>.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 <cstdint>, <numeric>, <ostream>, <string>, <utility>, and <vector>。

### Lines 11-16
```cpp
  11: #include "mlir/IR/BuiltinAttributes.h"
  12: #include "mlir/IR/ValueRange.h"
  13: #include "llvm/ADT/Hashing.h"
  14: #include "llvm/ADT/MapVector.h"
  15: #include "llvm/ADT/STLExtras.h"
  16: #include "llvm/ADT/SetVector.h"
```
**EN:** This block imports the direct dependencies needed here, including mlir/IR/BuiltinAttributes.h, mlir/IR/ValueRange.h, llvm/ADT/Hashing.h, llvm/ADT/MapVector.h, llvm/ADT/STLExtras.h, and llvm/ADT/SetVector.h.
**CN:** 该代码块引入后续声明直接依赖的头文件，例如 mlir/IR/BuiltinAttributes.h, mlir/IR/ValueRange.h, llvm/ADT/Hashing.h, llvm/ADT/MapVector.h, llvm/ADT/STLExtras.h, and llvm/ADT/SetVector.h。

### Lines 18-18
```cpp
  18: namespace mlir::triton {
```
**EN:** This block opens or closes the namespace scope used to organize APIs under mlir::triton.
**CN:** 该代码块打开或关闭命名空间作用域，用来把相关 API 组织在 mlir::triton 下。

### Lines 20-76
```cpp
  20: // # High-level overview of linear layouts
  21: //
  22: // The idea for linear layouts is due to Adam P. Goucher.
  23: //
  24: // In Triton, a linear layout (LL) is a function that maps from a "hardware
  25: // location" to a "logical tensor index".
  26: //
  27: // For example, suppose we have a 2D tensor T stored in GPU registers.  T's
  28: // layout (i.e., L) is the function that, given a "hardware location" tuple of
  29: // (thread-id, warp-id), returns an index (x,y) into T.  In other words, if
  30: // L(t,w) = (x,y) is our linear layout func, then a register in thread t in warp
  31: // w contains the value T[x,y].
  32: //
  33: // The key fact about LLs is, the mapping from (t,w) to (x,y) is not arbitrary.
  34: // We only need to specify the value of L(t,w) at certain special points
  35: // (namely, the values L(t,0) and L(0,w) where t and w are powers of 2), and
  36: // from those we can compute all the other values of L.
  37: //
  38: // Here's an example LL where we have 4 warps and 4 threads per warp, and the
  39: // tensor T has shape 4x4.  We define the function L by choosing the values of
  40: // L(0,1), L(0,2), L(1,0), and L(2,0).  Our choices are shown below.
  41: //
  42: //               t/w    0     1     2    3
  43: //               0      ? (0,1) (0,2)    ?
  44: //    L(t,w) =   1  (1,1)     ?     ?    ?
  45: //               2  (2,2)     ?     ?    ?
  46: //               3      ?     ?     ?    ?
  47: //
  48: // You only need to specify these four values to define the whole linear layout.
  49: // These special values are called the "basis vectors" or "bases" of the layout.
  50: // We complete the table by xor'ing together the bases, according to the
  51: // following rule.  (I write "⊕" for xor.)
  52: //
  53: //    L(t1 ⊕ t2, w1 ⊕ w2) = L(t1, w1) ⊕ L(t2, w2)  (linearity rule).
  54: //
  55: // The linearity rule plus our four choices allows us to fill in the whole
  56: // table.  Here's how we might compute some of the values.
  57: //
  58: //    L(0,0) = L(1 ⊕ 1, 0 ⊕ 0) = L(1,0) ⊕ L(1,0) = (1,1) ⊕ (1,1) = (0,0)
  59: //    L(0,3) = L(0 ⊕ 0, 2 ⊕ 1) = L(0,2) ⊕ L(0,1) = (0,2) ⊕ (0,1) = (0,3)
  60: //    L(3,0) = L(2 ⊕ 1, 0 ⊕ 0) = L(2,0) ⊕ L(1,0) = (2,2) ⊕ (1,1) = (3,3)
  61: //    L(3,3) = L(3 ⊕ 0, 0 ⊕ 3) = L(3,0) ⊕ L(0,3) = (3,3) ⊕ (0,3) = (3,0).
  62: //
  63: // (Notice it's a consequence of the linearity rule that L(0,0) = (0,0), no
  64: // matter what values we chose for the table.)
  65: //
  66: // The whole table looks like this.
  67: //
  68: //              t/w   0     1     2     3
  69: //              0  (0,0) (0,1) (0,2) (0,3)
  70: //    L(t,w) =  1  (1,1) (1,0) (1,3) (1,2)
  71: //              2  (2,2) (2,3) (2,0) (2,1)
  72: //              3  (3,3) (3,2) (3,1) (3,0).
  73: //
  74: // Careful readers will recognize this as a classic "swizzled" layout where
  75: // (t, w) -> (t, w ⊕ t).  To go from this formula to an LL, you only need to
  76: // compute the results at input points (0,1), (0,2), (1,0), and (2,0).
```
**EN:** This comment block records the design rationale of the surrounding code: # High-level overview of linear layouts The idea for linear layouts is due to Adam P. Goucher. In Triton, a linear layout (LL) is a function that maps from a "hardware location"....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 78-325
```cpp
  78: // Indeed the whole point of LLs is that they allow us to specify transposed and
  79: // swizzled layouts as a "general case".  Instead of a layout class for
  80: // registers in a thread, and another layout for registers in a thread but in
  81: // MMAv2 order, and so on, all of these can be represented by different LLs.
  82: // This gets rid of special cases and lets us write more general code.
  83: //
  84: // In this example, L was a 2D -> 2D function, but LLs are general MD -> ND
  85: // functions.  In practice, a GPU register layout usually has input dims (reg,
  86: // thread-id, warp-id, block-id), where reg represents the fact that one thread
  87: // may store values for the tensor in multiple registers.
  88: //
  89: // To summarize, a linear layout is a function from tuples of integers to tuples
  90: // of integers.  We specify some key values of the function, and then we can
  91: // compute all the other values using the linearity rule.
  92: //
  93: // Here are the key things you can do with linear layout objects.
  94: //
  95: //  1. Given an LL, construct a new LL by modifying it or combining it with
  96: //     another LL.
  97: //
  98: //  2. "Apply" an LL, i.e. use it to map an input index to an output index.
  99: //     A function for this that uses LLVM-dialect MLIR as its input and output
 100: //     lives in TritonGPUToLLVM.h.
 101: //
 102: //  3. Convert an existing Triton layout (e.g. BlockedLayoutAttr) to an LL.
 103: //     These functions live in TritonGPU/LinearLayoutConversions.h.  During
 104: //     TTGIR -> LLVM codegen, we convert Triton layouts to linear layouts and
 105: //     then apply them.  In the future, we intend to remove the Triton layouts
 106: //     entirely.
 107: //
 108: // # Examples of linear layouts
 109: //
 110: // 1. The 1D identity layout.  This maps L(x) = x.
 111: //
 112: //    Recall that our bases are the values of L(x) where x is a power of two.
 113: //    So for e.g. an 8-element layout, we have L(1) = 1, L(2) = 2, L(4) = 4, and
 114: //    therefore our bases are [1, 2, 4].
 115: //
 116: // 2. The 1D zeros layout.  This maps L(x) = 0.
 117: //
 118: //    For an 8-element layout, we have L(1) = L(2) = L(4) = 0, so our bases are
 119: //    [0, 0, 0].
 120: //
 121: // 3. A 2D -> 2D identity layout.  Our basis vectors are the values of L(x,0)
 122: //    and L(0,y) where x and y are powers of two.  The bases are
 123: //
 124: //    - L(0,1) = (0,1)
 125: //    - L(0,2) = (0,2)
 126: //    - L(1,0) = (1,0)
 127: //    - L(2,0) = (2,0).
 128: //
 129: // 4. A 2D -> 2D transpose layout.  For a 4x4 layout, we have:
 130: //
 131: //    - L(0,1) = (1,0)
 132: //    - L(0,2) = (2,0)
 133: //    - L(1,0) = (0,1)
 134: //    - L(2,0) = (0,2).
 135: //
 136: // 5. A 1D -> 1D "transpose" layout.  Consider the 16-element layout that maps
 137: //
 138: //    x    = 0 1 2 3 4 5 6 7 8 9 A B C D E F
 139: //    L(x) = 0 4 8 C 1 5 9 D 2 6 A E 3 7 B F.
 140: //
 141: //    The bases are [L(1), L(2), L(4), L(8)] = [4, 8, 1, 2].  You can also think
 142: //    of this as a rearrangement of the 1D identity layout [1, 2, 4, 8].
 143: //
 144: // 6. A 2D -> 1D broadcasted layout.  L(x,y) = x.  For a 4x4 -> 4 layout, our
 145: //    bases are
 146: //
 147: //    - L(0,1) = 0
 148: //    - L(0,2) = 0
 149: //    - L(1,0) = 1
 150: //    - L(2,0) = 2.
 151: //
 152: // # Implementation notes
 153: //
 154: // ## Dimension order
 155: //
 156: // An LL's input and output dimensions have an order.  This order only affects
 157: // the reshapeIns/Outs and similar operations, where the layout is logically
 158: // flattened according to the dimension order and then chopped up again.
 159: //
 160: // ## Surjectivity and injectivity
 161: //
 162: // Most LLs are surjective, i.e. all output values are covered by some input
 163: // value.  But occasionally you might create a non-surjective layout, usually
 164: // via invertAndCompose.  We aggressively assert that LLs are surjective unless
 165: // you explicitly create one that's not.
 166: //
 167: // LLs are not, in general, injective.  There might exist multiple input values
 168: // that map to the same output value.  This represents the idea that the same
 169: // logical tensor elements can be stored in multiple places in the hardware.
 170: //
 171: // ## Why map hardware loc -> tensor index and not the other way around?
 172: //
 173: // In Triton, a linear layout usually tells us which logical tensor value is
 174: // stored at a particular place in the hardware.  For example, an LL might map
 175: // the tuple (thread-id, warp-id, block-id) to a 2D index into a tensor, (x,y),
 176: // meaning that the register at (t,w,b) has value tensor[x,y].  Or it might map
 177: // from a shared memory (offset, block) to a tensor index.
 178: //
 179: // It might seem more natural to go the other way around, from tensor index to
 180: // place in the hardware.  But a particular tensor[x,y] value might be stored in
 181: // more than one place in the hardware, so if we went in this direction, the
 182: // layout would no longer be a proper function.  This would complicate
 183: // everything else.
 184: //
 185: // # Optional mathematical background: Linear functions over GF(2)
 186: //
 187: // (You shouldn't need to understand this math to use linear layouts, but it
 188: // helps with the implementation.)
 189: //
 190: // One way to define a linear function is to say it's any function F that can be
 191: // written as
 192: //
 193: //    L(a) = a1 * B1 + a2 * B2 + ... + aM * BM,
 194: //
 195: // where
 196: //
 197: //   - a is a vector [a1...aM], and ai is a scalar in some field 𝔽 (for
 198: //     example, ai might be a real number), and
 199: //   - each Bj is a vector [b1j, b1j, ..., bNj] of N scalars in 𝔽.
 200: //
 201: // We can also write this as a matrix-vector product Ba, where
 202: //
 203: //    - a is the column vector [a1, ..., aM] and
 204: //
 205: //    - B is the matrix formed by concatenating the column vectors B1, ..., BM:
 206: //
 207: //           | ↑    ↑         ↑ |
 208: //       B = | B1,  B2, ...,  BM|
 209: //           | ↓    ↓         ↓ |
 210: //
 211: //           |b11, b12, ..., b1M|
 212: //           |b21, b22, ..., b2M|
 213: //         = | ↓    ↓         ↓ |
 214: //           |bN1, bN2, ..., bNM|.
 215: //
 216: // Usually when we do linear algebra, the field 𝔽 from which `ai` and `bij` are
 217: // drawn is the real or complex numbers.  But in linear layouts, we let	𝔽 be a
 218: // different field: GF(2).
 219: //
 220: // GF(2) is the two-element field of bits.  To define a field, I need to give
 221: // you the set of elements and also addition and multiplication operations.  For
 222: // GF(2) the elements are simply {0,1}.  We define addition as xor, and
 223: // multiplication as binary `and`.
 224: //
 225: // Here's an example of a 4x4 matrix-vector multiply where the elements are in
 226: // GF(2).  I'm using ⊕ to represent GF(2)'s addition operation (i.e xor) and ×
 227: // to represent multiplication (i.e. binary `and`).
 228: //
 229: //    | 1 0 0 0 | | 0 |     | 1 |         | 0 |         | 0 |         | 0 |
 230: //    | 0 1 1 0 | | 1 |  =  | 0 | × 0  ⊕  | 1 | × 1  ⊕  | 1 | × 1  ⊕  | 0 | × 0
 231: //    | 0 0 1 1 | | 1 |     | 0 |         | 0 |         | 1 |         | 1 |
 232: //    | 0 0 1 1 | | 0 |     | 0 |         | 0 |         | 1 |         | 1 |
 233: //
 234: //                                        | 0 |         | 0 |
 235: //                       =                | 1 |    ⊕    | 1 |
 236: //                                        | 0 |         | 1 |
 237: //                                        | 0 |         | 1 |
 238: //
 239: //                          | 0 |
 240: //                       =  | 0 |.
 241: //                          | 1 |
 242: //                          | 1 |
 243: //
 244: // This works, but it's cumbersome.  It's more compact to think of the vector
 245: // `a` as an M-bit integer, and each column Bi of the matrix B as an N-bit
 246: // integer.  Here's the same matrix-vector product written this way.
 247: //
 248: //   = | 1 2 14 12 | × 6
 249: //   = | 1 2 14 12 | × 0b0110
 250: //   = (1 × 0) ⊕ (2 × 1) ⊕ (14 × 1) ⊕ (12 × 0)
 251: //   = 2 ⊕ 14
 252: //   = 12.
 253: //
 254: // And we confirm that our answer of 12 is equal to the binary value 0b1100 we
 255: // got before.
 256: //
 257: // Notice that the function F(a) is fully specified by the matrix B, and that
 258: // the four columns of B tell us the values of F at power-of-two values for `a`,
 259: // namely F(1), F(2), F(4), and F(8).  In other words, we specify four results
 260: // of F(x) (we call these the function's "basis vectors" or its "bases") and we
 261: // can then compute any other value by xor'ing together subsets of the bases.
 262: //
 263: // In the case of a 1D -> 1D layout, the implementation of an LL is
 264: // straightforward from the mathematical description.  If the LL is
 265: // higher-dimensional, we can "stack" the bit vectors to create 1D vectors.
 266: // For example, if we have a 2D LL and we're given input tuple (0b0011, 0b1100),
 267: // we can treat this like a 1D input 0b0011'1100 and then do the regular 1D LL
 268: // computation.  Similarly we can "unstack" the output from 1D to ND.
 269: //
 270: // The linearity rule presented earlier is perhaps misleading at this point.  In
 271: // the 1D view of things, we really only need
 272: //
 273: //    L(x ⊕ y) = L(x) ⊕ L(y)  (1D linearity rule),
 274: //
 275: // which is part of the definition of L being a linear function.  The new 1D
 276: // linearity rule plus stacking/unstacking is equivalent to the earlier
 277: // N-dimensional linearity rule.
 278: //
 279: // That's all we need in order to define linear layouts mathematically!
 280: //
 281: // # Comparison to Nvidia CuTe
 282: //
 283: // (Note, I'm not an expert on CuTe; this is my best understanding.)
 284: //
 285: // CuTe is a programmatic layout system that's part of Nvidia CUTLASS; see
 286: // https://github.com/NVIDIA/cutlass/blob/629f465/media/docs/cute/00_quickstart.md
 287: //
 288: // LLs and CuTe solve similar problems.  Before CuTe, CUTLASS v2 had many
 289: // handcrafted layouts, "RowMajor", "VoltaTensorOpMultiplicandCongruous", etc,
 290: // see https://www.youtube.com/watch?v=QLdUML5MCfE&t=574s.  Each of these was a
 291: // special case.  CUTLASS v3 introduced CuTe layouts, which are programmable and
 292: // subsume all of these special cases.  The CUTLASS folks say this simplified
 293: // CUTLASS, in the same way that we hope LLs will simplify Triton.
 294: //
 295: // Like CuTe layouts, LLs are also programmable and composable.  But there are
 296: // also some differences.
 297: //
 298: //  - Dimensions in LLs are named; CuTe dimensions are numbered.
 299: //  - CuTe layouts can be nested; LLs cannot be.  (Nesting doesn't give CuTe
 300: //    layouts additional power; any nested layout can be flattened.)
 301: //  - CuTe layouts support non-power-of-two shapes; LLs do not.  In particular
 302: //    this means that LLs cannot represent padded layouts.
 303: //  - In CuTe, swizzling is a separate step applied after specifying a layout.
 304: //    In LLs, swizzling is part of the layout itself.
 305: //  - The structure of LLs allows us to programmatically search for layouts that
 306: //    satisfy certain requirements, for example a shared layout that doesn't
 307: //    have bank conflicts when read into a particular register layout.  CuTe
 308: //    expects a human to choose the layout using their brain.
 309: //  - CuTe emits code that is in the critical path of your CPU and GPU programs,
 310: //    therefore it needs to be fast.  It uses C++ template magic to specialize
 311: //    on known-sized dimensions, and so on.  LLs themselves do not need to be
 312: //    fast; only the emitted `apply` code is on the critical path.
 313: //  - CuTe requires a CUDA compiler such as nvcc; LLs do not.
 314: //
 315: class LinearLayout {
 316: private:
 317:   // bases[inDim][i] = L(0, ..., inDim=2^i, ..., 0).  All other values of L are
 318:   // computed by xor'ing bases together, using the linearity rule.  In addition:
 319:   //
 320:   // - Each inDim has the same set of outDims, in the same order.
 321:   // - The order of dims is minor-to-major, although this only affects reshape.
 322:   llvm::MapVector<StringAttr /*inDim*/,
 323:                   std::vector<std::vector<int32_t> /*size=getNumOutDims()*/>
 324:                   /*size=getInDimSizeLog2(inDim)*/>
 325:       bases;
```
**EN:** This block introduces `LinearLayout`, the main class/struct defined here. Within the declaration, methods such as getNumOutDims expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `LinearLayout`。 其中 getNumOutDims 等方法构成了它的主要接口。

### Lines 327-328
```cpp
 327:   llvm::MapVector<StringAttr, int32_t /*size*/> outDims;
 328:   int32_t rank = 0;
```
**EN:** This block stores supporting state such as outDims, which other APIs in the file consume.
**CN:** 该代码块声明了 outDims 等支撑状态，供本文件中的其他 API 使用。

### Lines 330-331
```cpp
 330: public:
 331:   using BasesT = decltype(bases);
```
**EN:** This block carries supporting declarations or implementation details that complete the file-level API.
**CN:** 该代码块承载了补全文件级 API 所需的辅助声明或实现细节。

### Lines 333-333
```cpp
 333:   LinearLayout() = default;
```
**EN:** This block declares or defines callable APIs such as LinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 LinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 335-341
```cpp
 335:   // The 0-dimensional layout that maps everything to 0.  This is useful as a
 336:   // starting point when doing something like
 337:   //
 338:   //   LinearLayout ret = LinearLayout::empty();
 339:   //   for (...) ret *= ...;
 340:   //   return ret;
 341:   static LinearLayout empty() { return {}; }
```
**EN:** This block declares or defines callable APIs such as empty, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 empty 等可调用 API，用来封装这里提供的核心行为。

### Lines 343-346
```cpp
 343:   // Creates a 1D -> 1D layout that's the function L(x) = stride * x
 344:   // for x in [0, size).
 345:   static LinearLayout strided1D(int32_t size, int32_t stride, StringAttr inDim,
 346:                                 StringAttr outDim);
```
**EN:** This block declares or defines callable APIs such as strided1D, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 strided1D 等可调用 API，用来封装这里提供的核心行为。

### Lines 348-353
```cpp
 348:   // Creates a 1D -> 1D layout that's the identity function, i.e. L(x) = x
 349:   // for x in [0, size).
 350:   static LinearLayout identity1D(int32_t size, StringAttr inDim,
 351:                                  StringAttr outDim) {
 352:     return strided1D(size, /*stride=*/1, inDim, outDim);
 353:   }
```
**EN:** This block declares or defines callable APIs such as identity1D and strided1D, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 identity1D and strided1D 等可调用 API，用来封装这里提供的核心行为。

### Lines 355-361
```cpp
 355:   // Creates a 1D -> 1D layout that maps every input value to 0, i.e. L(x) = 0
 356:   // for x in [0, size). By default this creates a surjective layout where
 357:   // `outDim` has size 1 (the only element is 0). If `outDimSize` is specified
 358:   // to be greater than 1, then this creates a non-surjective layout with a
 359:   // specific size for `outDim`.
 360:   static LinearLayout zeros1D(int32_t size, StringAttr inDim, StringAttr outDim,
 361:                               int32_t outDimSize = 1);
```
**EN:** This block declares or defines callable APIs such as zeros1D, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 zeros1D 等可调用 API，用来封装这里提供的核心行为。

### Lines 363-380
```cpp
 363:   // Creates a LinearLayout from a list of bases.  These are interpreted
 364:   // according to the rules written for the member variable `bases`.
 365:   //
 366:   // Calculates the out-dim sizes according to the bases.  Consider the
 367:   // following example.
 368:   //
 369:   //   L(in1=1) = (out1=1, out2=0)
 370:   //   L(in1=2) = (out1=5, out2=1)
 371:   //   L(in1=4) = (out1=2, out2=2)
 372:   //
 373:   // To calculate the out-dim sizes, we first find the largest values for out1
 374:   // and out2, namely 5 and 2, then round these up to the next power of 2,
 375:   // namely 8 and 4.  These are the out-dim sizes.
 376:   //
 377:   // Assert-fails if the layout is not surjective given these out-dim sizes.
 378:   // That is, every possible out-dim in range [0, size) must be produced by
 379:   // xor'ing some combination of bases.
 380:   explicit LinearLayout(BasesT bases, ArrayRef<StringAttr> outDimNames);
```
**EN:** This block declares or defines callable APIs such as LinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 LinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 382-397
```cpp
 382:   // Creates a LinearLayout given a list of bases and the explicit out-dimension
 383:   // sizes.  Allows the layout to be non-surjective.
 384:   //
 385:   // To see why we need to explicitly pass out-dim sizes when creating a
 386:   // non-surjective layout, consider the following example.
 387:   //
 388:   //   L(in1=1) = 1
 389:   //   L(in1=2) = 4
 390:   //
 391:   // If we naively infer the out-dim sizes from these bases, we'd infer a size
 392:   // of nextPow2(4) = 8.  But given that the layout is non-surjective, who is to
 393:   // say that the codomain is not (say) [0,32)?  We can't tell, thus we need to
 394:   // be explicit about the sizes.
 395:   explicit LinearLayout(BasesT bases,
 396:                         ArrayRef<std::pair<StringAttr, int32_t>> outDims,
 397:                         bool requireSurjective);
```
**EN:** This block declares or defines callable APIs such as LinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 LinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 399-426
```cpp
 399:   // Construct a LinearLayout from an explicit list of bases.  (This constructor
 400:   // is needed because llvm::MapVector does not have a constructor that accepts
 401:   // an initializer_list.)
 402:   //
 403:   // For example, given these bases
 404:   //
 405:   //   L(in1=1, in2=0) = (out1=0, out2=1)
 406:   //   L(in1=2, in2=0) = (out1=0, out2=2)
 407:   //   L(in1=0, in2=1) = (out1=0, out2=4)
 408:   //   L(in1=0, in2=2) = (out1=0, out2=8)
 409:   //   L(in1=0, in2=4) = (out1=1, out2=1)
 410:   //
 411:   // we can use this constructor to build an equivalent LL:
 412:   //
 413:   // LinearLayout({
 414:   //     {"in1", {/*L(in1=1)=*/{0,1}, /*L(in1=2)=*/{0,2}}},
 415:   //     {"in2", {/*L(in2=1)=*/{0,4}, /*L(in2=2)=*/{0,8}, /*L(in2=4)=*/{1,1}}},
 416:   //   },
 417:   //   {"out1", "out2"})
 418:   //
 419:   // The overload that infers out-dim sizes assert-fails if the layout is not
 420:   // surjective.
 421:   explicit LinearLayout(
 422:       ArrayRef<std::pair<StringAttr, std::vector<std::vector<int32_t>>>> bases,
 423:       ArrayRef<StringAttr> outDimNames);
 424:   explicit LinearLayout(
 425:       ArrayRef<std::pair<StringAttr, std::vector<std::vector<int32_t>>>> bases,
 426:       ArrayRef<std::pair<StringAttr, int32_t>> outDims, bool requireSurjective);
```
**EN:** This block declares or defines callable APIs such as LinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 LinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 428-429
```cpp
 428:   bool isSurjective() const { return rank == getTotalOutDimSizeLog2(); }
 429:   bool isInjective() const { return rank == getTotalInDimSizeLog2(); }
```
**EN:** This block declares or defines callable APIs such as isSurjective, getTotalOutDimSizeLog2, isInjective, and getTotalInDimSizeLog2, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isSurjective, getTotalOutDimSizeLog2, isInjective, and getTotalInDimSizeLog2 等可调用 API，用来封装这里提供的核心行为。

### Lines 431-433
```cpp
 431:   bool isInvertible() const {
 432:     return isSurjective() && getTotalInDimSize() == getTotalOutDimSize();
 433:   }
```
**EN:** This block declares or defines callable APIs such as isInvertible, isSurjective, getTotalInDimSize, and getTotalOutDimSize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isInvertible, isSurjective, getTotalInDimSize, and getTotalOutDimSize 等可调用 API，用来封装这里提供的核心行为。

### Lines 435-437
```cpp
 435:   // Remove a dimension of size 1 from the layout.
 436:   [[nodiscard]] LinearLayout squeezeIns(StringAttr dim) const;
 437:   [[nodiscard]] LinearLayout squeezeOuts(StringAttr dim) const;
```
**EN:** This block declares or defines callable APIs such as squeezeIns and squeezeOuts, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 squeezeIns and squeezeOuts 等可调用 API，用来封装这里提供的核心行为。

### Lines 439-439
```cpp
 439:   const BasesT &getBases() const { return bases; }
```
**EN:** This block declares or defines callable APIs such as getBases, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBases 等可调用 API，用来封装这里提供的核心行为。

### Lines 441-449
```cpp
 441:   // Get the pos'th basis vector for the inDim -> outDim mapping.
 442:   // getBasis(inDim, pos) = L(0, ..., inDim = 2^pos, ..., 0).
 443:   ArrayRef<int32_t> getBasis(StringAttr inDim, int32_t pos) const {
 444:     auto it = bases.find(inDim);
 445:     assert(it != bases.end());
 446:     assert(pos >= 0);
 447:     assert(static_cast<size_t>(pos) < it->second.size());
 448:     return it->second[pos];
 449:   }
```
**EN:** This block declares or defines callable APIs such as getBasis, find, end, and size, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBasis, find, end, and size 等可调用 API，用来封装这里提供的核心行为。

### Lines 451-453
```cpp
 451:   int32_t getBasis(StringAttr inDim, int32_t pos, StringAttr outDim) const {
 452:     return getBasis(inDim, pos)[getOutDimIndex(outDim)];
 453:   }
```
**EN:** This block declares or defines callable APIs such as getBasis and getOutDimIndex, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getBasis and getOutDimIndex 等可调用 API，用来封装这里提供的核心行为。

### Lines 455-459
```cpp
 455:   // These are in minor-to-major order, although if you don't flatten the dims
 456:   // (e.g. by reshaping) then the order doesn't really affect anything.
 457:   auto getInDimNames() const { return llvm::make_first_range(bases); }
 458:   auto getOutDimNames() const { return llvm::make_first_range(outDims); }
 459:   auto getOutDimSizes() const { return llvm::make_second_range(outDims); }
```
**EN:** This block declares or defines callable APIs such as getInDimNames, make_first_range, getOutDimNames, getOutDimSizes, and make_second_range, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getInDimNames, make_first_range, getOutDimNames, getOutDimSizes, and make_second_range 等可调用 API，用来封装这里提供的核心行为。

### Lines 461-461
```cpp
 461:   // Relevant for reshaping
```
**EN:** This comment block records the intent and constraints of the surrounding code: Relevant for reshaping.
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 463-473
```cpp
 463:   SmallVector<std::pair<StringAttr, int32_t>> getInDims() const {
 464:     SmallVector<std::pair<StringAttr, int32_t>> inDims;
 465:     inDims.reserve(bases.size());
 466:     for (auto [inDim, inDimBases] : bases) {
 467:       inDims.push_back({inDim, getInDimSize(inDim)});
 468:     }
 469:     return inDims;
 470:   }
 471:   SmallVector<std::pair<StringAttr, int32_t>> getOutDims() const {
 472:     return to_vector(outDims);
 473:   }
```
**EN:** This block declares or defines callable APIs such as getInDims, reserve, size, push_back, getInDimSize, getOutDims, and to_vector, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getInDims, reserve, size, push_back, getInDimSize, getOutDims, and to_vector 等可调用 API，用来封装这里提供的核心行为。

### Lines 475-477
```cpp
 475:   // Gets the position that this outDim occupies in getOutDimNames().  Asserts
 476:   // if the dim is not present.
 477:   int32_t getOutDimIndex(StringAttr outDim) const;
```
**EN:** This block declares or defines callable APIs such as getOutDimIndex, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOutDimIndex 等可调用 API，用来封装这里提供的核心行为。

### Lines 479-480
```cpp
 479:   bool hasInDim(StringAttr inDim) const { return bases.contains(inDim); }
 480:   bool hasOutDim(StringAttr outDim) const { return outDims.contains(outDim); }
```
**EN:** This block declares or defines callable APIs such as hasInDim, contains, and hasOutDim, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 hasInDim, contains, and hasOutDim 等可调用 API，用来封装这里提供的核心行为。

### Lines 482-483
```cpp
 482:   int32_t getNumInDims() const { return bases.size(); }
 483:   int32_t getNumOutDims() const { return outDims.size(); }
```
**EN:** This block declares or defines callable APIs such as getNumInDims, size, and getNumOutDims, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumInDims, size, and getNumOutDims 等可调用 API，用来封装这里提供的核心行为。

### Lines 485-489
```cpp
 485:   // Asserts if the dimension is not present.
 486:   int32_t getInDimSizeLog2(StringAttr inDim) const;
 487:   int32_t getInDimSize(StringAttr inDim) const {
 488:     return 1 << getInDimSizeLog2(inDim);
 489:   }
```
**EN:** This block declares or defines callable APIs such as getInDimSizeLog2 and getInDimSize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getInDimSizeLog2 and getInDimSize 等可调用 API，用来封装这里提供的核心行为。

### Lines 491-492
```cpp
 491:   int32_t getTotalInDimSizeLog2() const;
 492:   int32_t getTotalInDimSize() const { return 1 << getTotalInDimSizeLog2(); }
```
**EN:** This block declares or defines callable APIs such as getTotalInDimSizeLog2 and getTotalInDimSize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getTotalInDimSizeLog2 and getTotalInDimSize 等可调用 API，用来封装这里提供的核心行为。

### Lines 494-512
```cpp
 494:   // getOutDimSize(dim) == s means that there exists an input value that will
 495:   // produce each output value in [0,s) (if the layout is surjective).
 496:   //
 497:   // For example, if our bases are
 498:   //
 499:   //   L(in0=1) = 1
 500:   //   L(in0=2) = 4
 501:   //   L(in1=1) = 2
 502:   //   L(in1=2) = 8
 503:   //
 504:   // then the largest value we can produce is L(3,3) = 1 ⊕ 4 ⊕ 2 ⊕ 8 = 15 (and
 505:   // indeed we can produce all values in [0,16) by xor'ing subsets of the bases
 506:   // 1,2,4,8), so getOutDimSize(out_dim0) == 16.
 507:   //
 508:   // Asserts if the dimension is not present.
 509:   int32_t getOutDimSizeLog2(StringAttr outDim) const;
 510:   int32_t getOutDimSize(StringAttr outDim) const {
 511:     return 1 << getOutDimSizeLog2(outDim);
 512:   }
```
**EN:** This block declares or defines callable APIs such as getOutDimSizeLog2 and getOutDimSize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getOutDimSizeLog2 and getOutDimSize 等可调用 API，用来封装这里提供的核心行为。

### Lines 514-515
```cpp
 514:   int32_t getTotalOutDimSizeLog2() const;
 515:   int32_t getTotalOutDimSize() const { return 1 << getTotalOutDimSizeLog2(); }
```
**EN:** This block declares or defines callable APIs such as getTotalOutDimSizeLog2 and getTotalOutDimSize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getTotalOutDimSizeLog2 and getTotalOutDimSize 等可调用 API，用来封装这里提供的核心行为。

### Lines 517-536
```cpp
 517:   // Finds the number of consecutive input elements in the first input dimension
 518:   // that map to consecutive output elements in the first output dimension.
 519:   //
 520:   // Mathematically, finds the maximum value V such that for any a, b, c, and
 521:   // for all v in [0,V),
 522:   //
 523:   //   L(a*V + v, b, c, ...) = L(a*V, b, c, ...) + (v, 0, ..., 0)
 524:   //
 525:   // Note that's +, not ⊕, in the RHS.  (Equivalently, we could use binary-or
 526:   // instead of +.  In other words, we require that L(a*V, b, c, ...) have no
 527:   // bits that overlap with v.)
 528:   //
 529:   // For example, if L maps (register, lane) to (dim1, dim0), then this tells
 530:   // you how many consecutive registers map to consecutive elements of dim1.
 531:   //
 532:   // This only works across the first (i.e. the most-minor) dimension of in/out.
 533:   // If you want it to work across more dimensions, flatten the layout.
 534:   //
 535:   // TODO(jlebar): Replace with divideLeft.
 536:   int32_t getNumConsecutiveInOut() const;
```
**EN:** This block declares or defines callable APIs such as getNumConsecutiveInOut, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getNumConsecutiveInOut 等可调用 API，用来封装这里提供的核心行为。

### Lines 538-544
```cpp
 538:   // Reorders the in/out dimensions of the layout.  This is mostly cosmetic
 539:   // (affecting e.g. the order of getIn/OutDimNames), but it also affects the
 540:   // behavior of reshape.
 541:   [[nodiscard]] LinearLayout
 542:   transposeIns(ArrayRef<StringAttr> newInDimOrder) const;
 543:   [[nodiscard]] LinearLayout
 544:   transposeOuts(ArrayRef<StringAttr> newOutDimOrder) const;
```
**EN:** This block declares or defines callable APIs such as transposeIns and transposeOuts, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 transposeIns and transposeOuts 等可调用 API，用来封装这里提供的核心行为。

### Lines 546-548
```cpp
 546:   [[nodiscard]] LinearLayout reshapeIns(
 547:       ArrayRef<std::pair<StringAttr /*inDimName*/, int32_t /*size*/>> newInDims)
 548:       const;
```
**EN:** This block declares or defines callable APIs such as reshapeIns, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 reshapeIns 等可调用 API，用来封装这里提供的核心行为。

### Lines 550-556
```cpp
 550:   // Reshapes to a single input dim (named whatever our first in-dim is named).
 551:   [[nodiscard]] LinearLayout flattenIns() const {
 552:     if (getNumInDims() == 0) {
 553:       return reshapeIns({});
 554:     }
 555:     return reshapeIns({{*getInDimNames().begin(), getTotalInDimSize()}});
 556:   }
```
**EN:** This block declares or defines callable APIs such as flattenIns, getNumInDims, reshapeIns, getInDimNames, begin, and getTotalInDimSize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 flattenIns, getNumInDims, reshapeIns, getInDimNames, begin, and getTotalInDimSize 等可调用 API，用来封装这里提供的核心行为。

### Lines 558-560
```cpp
 558:   [[nodiscard]] LinearLayout
 559:   reshapeOuts(ArrayRef<std::pair<StringAttr /*outDimName*/, int32_t /*size*/>>
 560:                   newOutDims) const;
```
**EN:** This block declares or defines callable APIs such as reshapeOuts, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 reshapeOuts 等可调用 API，用来封装这里提供的核心行为。

### Lines 562-568
```cpp
 562:   // Reshapes to a single out dim (named whatever our first out-dim is named).
 563:   [[nodiscard]] LinearLayout flattenOuts() const {
 564:     if (getNumOutDims() == 0) {
 565:       return reshapeOuts({});
 566:     }
 567:     return reshapeOuts({{*getOutDimNames().begin(), getTotalOutDimSize()}});
 568:   }
```
**EN:** This block declares or defines callable APIs such as flattenOuts, getNumOutDims, reshapeOuts, getOutDimNames, begin, and getTotalOutDimSize, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 flattenOuts, getNumOutDims, reshapeOuts, getOutDimNames, begin, and getTotalOutDimSize 等可调用 API，用来封装这里提供的核心行为。

### Lines 570-575
```cpp
 570:   // Resizes the dimension to one that is smallre or equal to the given size.
 571:   // These operations are similar to `sublayout` but at a dimension level.
 572:   [[nodiscard]] LinearLayout resizeInDim(StringAttr inDim,
 573:                                          int32_t newSize) const;
 574:   [[nodiscard]] LinearLayout resizeOutDim(StringAttr outDim,
 575:                                           int32_t newSize) const;
```
**EN:** This block declares or defines callable APIs such as resizeInDim and resizeOutDim, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 resizeInDim and resizeOutDim 等可调用 API，用来封装这里提供的核心行为。

### Lines 577-587
```cpp
 577:   [[nodiscard]] LinearLayout renameInDim(StringAttr oldDim,
 578:                                          StringAttr newDim) const {
 579:     auto bases = getBases();
 580:     auto it = bases.find(oldDim);
 581:     assert(it != bases.end());
 582:     auto value = std::move(it->second);
 583:     bases.erase(it);
 584:     bases.insert({newDim, std::move(value)});
 585:     return LinearLayout(std::move(bases), getOutDims(),
 586:                         /*requireSurjective=*/isSurjective());
 587:   }
```
**EN:** This block declares or defines callable APIs such as renameInDim, getBases, find, end, move, erase, insert, and LinearLayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 renameInDim, getBases, find, end, move, erase, insert, and LinearLayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 589-595
```cpp
 589:   // Concatenates two layouts by their in (resp. out) dimensions. The layouts
 590:   // must have the same output (resp. input) dimensions and sizes and different
 591:   // input (resp. output) dimensions. The input dimensions of this layout are
 592:   // placed before those of 'other'. This can be thought of as the opposite of
 593:   // `sublayout`, which slices a layout from a larger one.
 594:   [[nodiscard]] LinearLayout concatIns(const LinearLayout &other) const;
 595:   [[nodiscard]] LinearLayout concatOuts(const LinearLayout &other) const;
```
**EN:** This block declares or defines callable APIs such as concatIns and concatOuts, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 concatIns and concatOuts 等可调用 API，用来封装这里提供的核心行为。

### Lines 597-631
```cpp
 597:   // Computes the direct sum of two layouts.
 598:   // https://en.wikipedia.org/wiki/Direct_sum#Direct_sum_of_matrices
 599:   //
 600:   // Roughly speaking, the first layout acts on the first part of the input
 601:   // dimensions, and the second layout acts on the second part.
 602:   // In other words, it's the generalisation of concatenation of the inputs
 603:   // to linear maps.
 604:   //
 605:   // Examples:
 606:   //
 607:   //  - empty() is the multiplicative identity:
 608:   //
 609:   //      L * empty() == empty() * L == L.
 610:   //
 611:   //  - Multiplying two identity1D layouts with disjoint in/out dimensions gives
 612:   //    a 2D identity layout:
 613:   //
 614:   //      identity1D(4, "i1", "o1") * identity1D(8, "i2", "o2") =>
 615:   //      L(i1,i2) = (i1,i2),
 616:   //
 617:   //    with in-dims ("i1", "i2") and out-dims ("o1", "o2"), in that order.
 618:   //
 619:   //  - If out-dims overlap, they are combined, as in the following examples.
 620:   //
 621:   //    - identity1D(4, "i", "o") * identity1D(2, "i", "o") ==
 622:   //      identity1D(8, "i", "o")
 623:   //      The output matrix is [[1, 0, 0], [0, 1, 0], [0, 0, 1]]
 624:   //
 625:   //    - identity1D(4, "i", "o") * zeros1D(2, "i", "o") => L(x) = x % 4
 626:   //      for x in [0,8).
 627:   //      The output matrix is [[1, 0, 0], [0, 1, 0]]
 628:   //
 629:   //    - zeros1D(2, "i", "o") * identity1D(4, "i", "o") => L(x) = x / 2
 630:   //      for x in [0,8).
 631:   //      The output matrix is [[0, 1, 0], [0, 0, 1]]
```
**EN:** This comment block records the design rationale of the surrounding code: Computes the direct sum of two layouts. https://en.wikipedia.org/wiki/Direct_sum#Direct_sum_of_matrices Roughly speaking, the first layout acts on the first part of the input di....
**CN:** 该注释块解释了周边代码的设计背景、意图或约束条件，帮助读者理解后续实现。

### Lines 633-654
```cpp
 633:   //    - identity1D(4, "i", "o1") * identity1D(8, "i", "o2") =>
 634:   //      L(x) = (x % 4, x / 4) for x in [0,32).
 635:   //      The output dims are ("o1", "o2") in that order.
 636:   //
 637:   // If the input (or output) dims of the layouts are not the same, we take
 638:   // the supremum of the two ordered lists with the inclusion, respecting the
 639:   // order. If multiple suprema exist, we bias towards the first list.
 640:   // e.g. sup([a, b], [a, c]) = [a, b, c], sup([a, b], [b, c]) = [a, b, c]
 641:   //      sup([a, b], [b, a]) = error! Supremum does not exist.
 642:   //
 643:   // Notice that this operation is not commutative, but it is associative.
 644:   //
 645:   // Requires: Any in/out dimensions which are in both outer and inner appear in
 646:   // the same relative order.
 647:   //
 648:   // Postcondition: If both inner and outer are surjective, the result is
 649:   // surjective.
 650:   friend LinearLayout operator*(LinearLayout inner, LinearLayout outer);
 651:   LinearLayout &operator*=(LinearLayout outer) {
 652:     *this = *this * outer;
 653:     return *this;
 654:   }
```
**EN:** This block stores supporting state such as this, which other APIs in the file consume.
**CN:** 该代码块声明了 this 等支撑状态，供本文件中的其他 API 使用。

### Lines 656-673
```cpp
 656:   // Compute a C such that A = B * C if it exists.
 657:   // In other words, C = B^{-1} * A.
 658:   // For divideRight, we compute A = C * B, that is, C = A * B^{-1}.
 659:   // Note that such a C exists iff (every pair of input/output dim of) A is
 660:   // of the form
 661:   // [[B, 0],
 662:   //  [0, C]]
 663:   // as a matrix, whenever those dimensions are present in B.
 664:   //
 665:   // C will always have the same input/output dimensions as A.
 666:   // When there are dimensions of size 1 there is some ambiguity in the
 667:   // division, as in `operator*` we treat missing dimensions as dimensions
 668:   // of size 1 whenever it makes sense to do so. The rule that C has the
 669:   // same dimensions as A ensures that C is well-defined.
 670:   friend std::optional<LinearLayout> divideLeft(const LinearLayout &A,
 671:                                                 const LinearLayout &B);
 672:   friend std::optional<LinearLayout> divideRight(const LinearLayout &A,
 673:                                                  const LinearLayout &B);
```
**EN:** This block declares or defines callable APIs such as divideLeft and divideRight, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 divideLeft and divideRight 等可调用 API，用来封装这里提供的核心行为。

### Lines 675-678
```cpp
 675:   // Returns true if this layout acts trivially (as the identity) on the given
 676:   // dimensions. This means that it's the identity on those dimensions, and it
 677:   // does not map other dimensions onto those or these onto other dimensions.
 678:   bool isTrivialOver(ArrayRef<StringAttr> dimNames) const;
```
**EN:** This block declares or defines callable APIs such as isTrivialOver, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isTrivialOver 等可调用 API，用来封装这里提供的核心行为。

### Lines 680-688
```cpp
 680:   // For an endomorphism on dimNames (linear map that maps dimNames to dimNames)
 681:   // checks whether it is the identity map on these dimensions (i.e
 682:   // LinearLayouts::isTrivialOver) and if so, returns the sublayout of the
 683:   // remaining dimensions.
 684:   // nb. The isTrivialOver condition is more restrictive than the usual
 685:   //     "leaves the subspace invariant" condition in maths.
 686:   //     We can always relax it if we know how to take advantage of a conversion
 687:   //     layout being block-diagonal in the future.
 688:   std::optional<LinearLayout> quotient(ArrayRef<StringAttr> dimNames) const;
```
**EN:** This block declares or defines callable APIs such as quotient, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 quotient 等可调用 API，用来封装这里提供的核心行为。

### Lines 690-699
```cpp
 690:   // Gets a layout with only these in/out dimensions.
 691:   //
 692:   // In other words, gets a layout where the in-dims not mentioned in inDimNames
 693:   // are set to 0, and the out-dims not mentioned in outDimNames are omitted.
 694:   //
 695:   // The output-dim sizes are unchanged.  The order of the in/out dims in the
 696:   // returned layout matches the order of the original layout, not the order of
 697:   // the arguments.
 698:   LinearLayout sublayout(ArrayRef<StringAttr> inDimNames,
 699:                          ArrayRef<StringAttr> outDimNames) const;
```
**EN:** This block declares or defines callable APIs such as sublayout, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 sublayout 等可调用 API，用来封装这里提供的核心行为。

### Lines 701-703
```cpp
 701:   // Is the sublayout restricted to inDimNames + outDimNames all zeros?
 702:   bool sublayoutIsZero(ArrayRef<StringAttr> inDimNames,
 703:                        ArrayRef<StringAttr> outDimNames) const;
```
**EN:** This block declares or defines callable APIs such as sublayoutIsZero, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 sublayoutIsZero 等可调用 API，用来封装这里提供的核心行为。

### Lines 705-710
```cpp
 705:   // Computes and returns L(x, y, z).
 706:   //
 707:   // If you want to apply the layout to mlir Values instead of integers, that
 708:   // function lives in TritonGPUToLLVM/Utility.h.
 709:   SmallVector<std::pair<StringAttr, int32_t>>
 710:   apply(ArrayRef<std::pair<StringAttr, int32_t>> ins) const;
```
**EN:** This block declares or defines callable APIs such as apply, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 apply 等可调用 API，用来封装这里提供的核心行为。

### Lines 712-729
```cpp
 712:   // Creates a new layout which is equivalent to running this layout, then
 713:   // running `outer`.  That is,
 714:   //
 715:   //  - let this layout be L(x), and
 716:   //  - let `outer` be O(x).
 717:   //  - Then compose(outer) returns the layout (O∘L)(x), aka O(L(x)).
 718:   //
 719:   // Requires:
 720:   //   - The output dimensions of this layout equal the input dimensions of
 721:   //     outer (order doesn't matter).
 722:   //   - For each output dim d of this layout, this->getOutDimSize(d) <=
 723:   //     outer.getInDimSize(d).
 724:   //
 725:   // Postcondition: The result is surjective iff `this` and `outer` are
 726:   // surjective and this->getOutDimSize(d) == outer.getInDimSize(d) for each of
 727:   // this->getOutDimNames().
 728:   //
 729:   [[nodiscard]] LinearLayout compose(const LinearLayout &outer) const;
```
**EN:** This block declares or defines callable APIs such as compose, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 compose 等可调用 API，用来封装这里提供的核心行为。

### Lines 731-763
```cpp
 731:   // Inverts or pseudo-inverts `outer` and composes it with `this`.
 732:   //
 733:   // Formally, if C = A.invertAndCompose(B), then for all x, C(x) = y implies
 734:   // A(x) = B(y), or in other words A(x) = B(C(x)).  If B is invertible, then
 735:   // C(x) = B^-1(A(x)), which is how this function gets its name.
 736:   //
 737:   // For example, suppose you have the following two LLs.
 738:   //
 739:   //   - R is an LL representing registers, mapping (lane, warp) to a 2D index.
 740:   //   - S is an LL representing shared memory, mapping offset to a 2D index.
 741:   //
 742:   // Suppose you want to store tensor values from registers into shared memory.
 743:   // That is, given a (lane, warp), you want to know the corresponding shared
 744:   // memory offset to store into.
 745:   //
 746:   // This is equivalent to converting a (lane, warp) into a 2D index (i.e.
 747:   // applying R), then converting a 2D index into a shmem offset (i.e. applying
 748:   // the inverse of S).  R.invertAndCompose(S) computes this transformation.
 749:   //
 750:   // Notice the following requirements in order for this to work.
 751:   //
 752:   //   - R and S must have the same output dimension names (different order is
 753:   //     allowed).
 754:   //   - S must be surjective, i.e. there must be some offset for each output
 755:   //     dimension of S.  This way when we compose S^-1 with R, every possible
 756:   //     2D index that we might get from R has some shmem offset.
 757:   //   - The codomain of S must be at least as large as the codomain of R.
 758:   //     Otherwise, R could map some tensor index that is not stored in S.
 759:   //
 760:   // One requirement we *don't* have is that S is injective; we allow two shmem
 761:   // offsets to hold the same 2D index.  If S is not injective,
 762:   // the algorithm chooses the smallest offset for a given (lane, warp).
 763:   [[nodiscard]] LinearLayout invertAndCompose(const LinearLayout &outer) const;
```
**EN:** This block declares or defines callable APIs such as invertAndCompose, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 invertAndCompose 等可调用 API，用来封装这里提供的核心行为。

### Lines 765-770
```cpp
 765:   // Get the layout that is the inverse of this layout.
 766:   [[nodiscard]] LinearLayout invert() const;
 767:   // Compute and return a psueodinverse of this layout. This is a layout such
 768:   // that `B = A.psuedoinvert()` implies that `A(B(x)) = I`. If `A` is
 769:   // invertible, then this returns `A^-1`.
 770:   [[nodiscard]] LinearLayout pseudoinvert() const;
```
**EN:** This block declares or defines callable APIs such as invert and pseudoinvert, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 invert and pseudoinvert 等可调用 API，用来封装这里提供的核心行为。

### Lines 772-778
```cpp
 772:   // For each in-dim, returns a bitmask of the "free variables" in the layout
 773:   // function.
 774:   //
 775:   // These are the bits in the input that can be changed without changing the
 776:   // output.  If all of the free variables are 0, then the layout is injective
 777:   // (i.e. every input bit affects the output).
 778:   llvm::MapVector<StringAttr, int32_t> getFreeVariableMasks() const;
```
**EN:** This block declares or defines callable APIs such as getFreeVariableMasks, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getFreeVariableMasks 等可调用 API，用来封装这里提供的核心行为。

### Lines 780-784
```cpp
 780:   // Take the current linear layout and remove all zero bases for the provided
 781:   // dimension and return the resulting layout. This is useful for deriving a
 782:   // layout that returns just the unique output values when varying a given
 783:   // input dimension that has broadcasting.
 784:   [[nodiscard]] LinearLayout removeZeroBasesAlongDim(StringAttr stripDim) const;
```
**EN:** This block declares or defines callable APIs such as removeZeroBasesAlongDim, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 removeZeroBasesAlongDim 等可调用 API，用来封装这里提供的核心行为。

### Lines 786-786
```cpp
 786:   std::string toString() const;
```
**EN:** This block declares or defines callable APIs such as toString, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toString 等可调用 API，用来封装这里提供的核心行为。

### Lines 788-793
```cpp
 788:   friend bool operator==(const LinearLayout &lhs, const LinearLayout &rhs);
 789:   friend bool operator!=(const LinearLayout &lhs, const LinearLayout &rhs) {
 790:     return !(lhs == rhs);
 791:   }
 792:   bool equalIgnoringOutDimSizes(const LinearLayout &other) const;
 793:   friend size_t hash_value(const LinearLayout &layout);
```
**EN:** This block declares or defines callable APIs such as equalIgnoringOutDimSizes and hash_value, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 equalIgnoringOutDimSizes and hash_value 等可调用 API，用来封装这里提供的核心行为。

### Lines 795-800
```cpp
 795: private:
 796:   // Factory function that gracefully fails rather than asserts if the layout is
 797:   // not well-formed.
 798:   static std::optional<LinearLayout>
 799:   tryCreate(BasesT bases, ArrayRef<std::pair<StringAttr, int32_t>> outDims,
 800:             bool requireSurjective);
```
**EN:** This block declares or defines callable APIs such as tryCreate, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 tryCreate 等可调用 API，用来封装这里提供的核心行为。

### Lines 802-805
```cpp
 802:   // Constructor that does not check invariants.  Used by tryCreate.
 803:   struct NoCheckInvariants {};
 804:   LinearLayout(BasesT bases, ArrayRef<std::pair<StringAttr, int32_t>> outDims,
 805:                NoCheckInvariants);
```
**EN:** This block introduces `NoCheckInvariants`, the main class/struct defined here. Within the declaration, methods such as LinearLayout expose its core API.
**CN:** 该代码块引入了此文件的核心类/结构体 `NoCheckInvariants`。 其中 LinearLayout 等方法构成了它的主要接口。

### Lines 807-809
```cpp
 807:   [[nodiscard]] std::optional<std::string>
 808:   checkInvariants(bool requireSurjective);
 809: };
```
**EN:** This block declares or defines callable APIs such as checkInvariants, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 checkInvariants 等可调用 API，用来封装这里提供的核心行为。

### Lines 811-815
```cpp
 811: inline llvm::raw_ostream &operator<<(llvm::raw_ostream &os,
 812:                                      const LinearLayout &layout) {
 813:   os << layout.toString();
 814:   return os;
 815: }
```
**EN:** This block declares or defines callable APIs such as toString, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toString 等可调用 API，用来封装这里提供的核心行为。

### Lines 817-820
```cpp
 817: inline std::ostream &operator<<(std::ostream &os, const LinearLayout &layout) {
 818:   os << layout.toString();
 819:   return os;
 820: }
```
**EN:** This block declares or defines callable APIs such as toString, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toString 等可调用 API，用来封装这里提供的核心行为。

### Lines 822-839
```cpp
 822: // Defines a map acting on the columns (i.e. bases) a given input dimension of a
 823: // layout as per:
 824: //  action[i] -> i.
 825: // This action can be:
 826: //  - Applied to a layout to get a new layout with the same input dimensions
 827: //    but with the bases permuted (and perhaps some of them dropped).
 828: //  - Applied to a range of Values to apply the same transformation to them
 829: //
 830: // E.g. if action = [2, 0, 1] and basesDim = [1, 2, 4]
 831: //  - action.apply(layout) returns a LL with basesDim = [4, 1, 2]
 832: //  - action.apply(range) with range.size() == 8, returns a range permuted as
 833: //    [x[0], x[4], x[1], x[5], x[2], x[6], x[3], x[7]]
 834: class ColumnAction {
 835: private:
 836:   SmallVector<size_t> action;
 837:   StringAttr inDim;
 838:   size_t inSizeLog2;
 839:   bool m_isIdentity = true;
```
**EN:** This block introduces `ColumnAction`, the main class/struct defined here.
**CN:** 该代码块引入了此文件的核心类/结构体 `ColumnAction`。

### Lines 841-852
```cpp
 841: public:
 842:   ColumnAction() = default;
 843:   ColumnAction(ArrayRef<size_t> action, StringAttr inDim, size_t inSizeLog2)
 844:       : action(action), inDim(inDim), inSizeLog2(inSizeLog2) {
 845:     auto it = llvm::max_element(action);
 846:     // Assert in the constructor... ugh
 847:     assert(it == action.end() || *it < inSizeLog2);
 848:     // In many cases the action will be the identity, so we save that as an
 849:     // early return
 850:     m_isIdentity = action.size() == inSizeLog2 &&
 851:                    llvm::equal(action, llvm::seq<size_t>(action.size()));
 852:   }
```
**EN:** This block declares or defines callable APIs such as ColumnAction, action, inDim, inSizeLog2, max_element, end, size, and equal, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 ColumnAction, action, inDim, inSizeLog2, max_element, end, size, and equal 等可调用 API，用来封装这里提供的核心行为。

### Lines 854-860
```cpp
 854:   // Act on the columns of a layout
 855:   // Examples:
 856:   //  - if action = [2, 0, 1] and layout.getBases()[inDim] = [[1], [2], [4]]
 857:   //    - action.apply(layout) returns a LL with basesDim = [[4], [1], [2]]
 858:   //  - if action = [2, 0] and layout.getBases()[inDim] = [[1], [4], [2]]
 859:   //    - action.apply(layout) returns a LL with bases[inDim] = [[2], [1]]
 860:   LinearLayout apply(const LinearLayout &layout) const;
```
**EN:** This block declares or defines callable APIs such as apply, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 apply 等可调用 API，用来封装这里提供的核心行为。

### Lines 862-866
```cpp
 862:   // Act on a range of values (representing registers)
 863:   // e.g. if action = [2, 0, 1] and inSizeLog2 = 3 and inDim.str() = "register"
 864:   //  - action.apply(range) with range.size() == 8, returns
 865:   //    [x[0], x[4], x[1], x[5], x[2], x[6], x[3], x[7]]
 866:   SmallVector<Value> apply(ValueRange values) const;
```
**EN:** This block declares or defines callable APIs such as apply, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 apply 等可调用 API，用来封装这里提供的核心行为。

### Lines 868-869
```cpp
 868:   // Inverse of the action
 869:   ColumnAction inverse() const;
```
**EN:** This block declares or defines callable APIs such as inverse, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 inverse 等可调用 API，用来封装这里提供的核心行为。

### Lines 871-873
```cpp
 871:   // Given two permutations self, other seen as functions, returns
 872:   // ret(x) = other(self(x))
 873:   ColumnAction leftCompose(const ColumnAction &other) const;
```
**EN:** This block declares or defines callable APIs such as leftCompose, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 leftCompose 等可调用 API，用来封装这里提供的核心行为。

### Lines 875-878
```cpp
 875:   static ColumnAction identity(StringAttr inDim, size_t inSizeLog2) {
 876:     return ColumnAction(llvm::to_vector(llvm::seq<size_t>(inSizeLog2)), inDim,
 877:                         inSizeLog2);
 878:   }
```
**EN:** This block declares or defines callable APIs such as identity, ColumnAction, and to_vector, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 identity, ColumnAction, and to_vector 等可调用 API，用来封装这里提供的核心行为。

### Lines 880-881
```cpp
 880:   // Returns true if the action is the identity
 881:   bool isIdentity() const { return m_isIdentity; }
```
**EN:** This block declares or defines callable APIs such as isIdentity, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 isIdentity 等可调用 API，用来封装这里提供的核心行为。

### Lines 883-884
```cpp
 883:   std::string toString() const;
 884: };
```
**EN:** This block declares or defines callable APIs such as toString, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toString 等可调用 API，用来封装这里提供的核心行为。

### Lines 886-890
```cpp
 886: inline llvm::raw_ostream &operator<<(llvm::raw_ostream &os,
 887:                                      const ColumnAction &action) {
 888:   os << action.toString();
 889:   return os;
 890: }
```
**EN:** This block declares or defines callable APIs such as toString, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toString 等可调用 API，用来封装这里提供的核心行为。

### Lines 892-895
```cpp
 892: inline std::ostream &operator<<(std::ostream &os, const ColumnAction &action) {
 893:   os << action.toString();
 894:   return os;
 895: }
```
**EN:** This block declares or defines callable APIs such as toString, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 toString 等可调用 API，用来封装这里提供的核心行为。

### Lines 897-897
```cpp
 897: std::unique_ptr<uint64_t[]> getMatrix(const LinearLayout &layout);
```
**EN:** This block declares or defines callable APIs such as getMatrix, which package the main behavior offered here.
**CN:** 该代码块声明或定义了 getMatrix 等可调用 API，用来封装这里提供的核心行为。

### Lines 899-899
```cpp
 899: } // namespace mlir::triton
```
**EN:** This block manages the namespace scope for the surrounding declarations.
**CN:** 该代码块负责管理周围声明所属的命名空间作用域。

### Lines 901-901
```cpp
 901: #endif // TRITON_TOOLS_LINEARLAYOUT_H
```
**EN:** This block closes the header guard started at the top of the file.
**CN:** 该代码块结束文件顶部开始的头文件保护宏。

## Key Concepts / 关键概念
- **EN:** layout conversion and reasoning  
  **CN:** 布局转换与推理
- **EN:** linear layout algebra  
  **CN:** 线性布局代数
- **EN:** warp-level execution  
  **CN:** warp 级执行
- **EN:** LLVM lowering  
  **CN:** LLVM 降级
- **EN:** MLIR dialect definition  
  **CN:** MLIR 方言定义
- **EN:** attributes  
  **CN:** 属性
- **EN:** MLIR integration  
  **CN:** MLIR 集成

## Dependencies / 依赖关系
- **Project includes / 项目内依赖:**
  - `mlir/IR/BuiltinAttributes.h`
  - `mlir/IR/ValueRange.h`
  - `llvm/ADT/Hashing.h`
  - `llvm/ADT/MapVector.h`
  - `llvm/ADT/STLExtras.h`
  - `llvm/ADT/SetVector.h`
- **System or external includes / 系统或外部依赖:**
  - `<cstdint>`
  - `<numeric>`
  - `<ostream>`
  - `<string>`
  - `<utility>`
  - `<vector>`
