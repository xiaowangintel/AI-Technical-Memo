# SparseTensorMath.metal — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/sparse/mps/kernels/SparseTensorMath.metal`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): Defines GPU shader logic for MPS sparse backend support, centered on Sparse Tensor Math with emphasis on sparse tensor processing.
- 用途（中文）: 定义 GPU 着色器逻辑，属于MPS 稀疏后端支持，核心主题是Sparse Tensor Math，重点关注稀疏张量处理。

## Line-by-Line Analysis / 逐行分析

- Note (EN): The file is annotated in contiguous line ranges; each numbered line receives a short bilingual explanation.
- 说明（中文）: 为兼顾可读性，以下按连续行区间展示代码，并对每个编号行给出简短的中英双语说明。

### Lines 1-30

```cpp
   1: #include <c10/metal/indexing.h>
   2: #include <c10/metal/utils.h>
   3: using namespace c10::metal;
   4: using namespace metal;
   5: 
   6: inline uint lower_bound_i64(device const long* arr, uint lo, uint hi, long key) {
   7:   uint l = lo, r = hi;
   8:   while (l < r) {
   9:     uint m = (l + r) >> 1;
  10:     long v = arr[m];
  11:     if (v < key) {
  12:       l = m + 1;
  13:     } else {
  14:       r = m;
  15:     }
  16:   }
  17:   return l;
  18: }
  19: 
  20: inline uint upper_bound_i64(device const long* arr, uint lo, uint hi, long key) {
  21:   uint l = lo, r = hi;
  22:   while (l < r) {
  23:     uint m = (l + r) >> 1;
  24:     long v = arr[m];
  25:     if (v <= key) {
  26:       l = m + 1;
  27:     } else {
  28:       r = m;
  29:     }
  30:   }
```
- L1: Includes `c10/metal/indexing.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/metal/indexing.h`，用于 c10 核心运行时、工具或分发元数据。
- L2: Includes `c10/metal/utils.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/metal/utils.h`，用于 c10 核心运行时、工具或分发元数据。
- L3: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L4: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L6: Marks the entity inline so it can be defined in headers safely. / 将实体标记为 inline，使其可安全地在头文件中定义。
- L7: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L8: Starts a loop that repeats while the controlling condition remains true. / 开始一个在控制条件保持为真时重复执行的循环。
- L9: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L10: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L11: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L12: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L13: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L14: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L15: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L16: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L17: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L18: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L20: Marks the entity inline so it can be defined in headers safely. / 将实体标记为 inline，使其可安全地在头文件中定义。
- L21: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L22: Starts a loop that repeats while the controlling condition remains true. / 开始一个在控制条件保持为真时重复执行的循环。
- L23: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L24: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L25: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L26: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L27: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L28: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L29: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L30: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 31-60

```cpp
  31:   return l;
  32: }
  33: 
  34: kernel void build_row_ptr_from_sorted_rows_by_batch(
  35:     device const long* rows        [[buffer(0)]],
  36:     device const long* batch_ptr   [[buffer(1)]],
  37:     device long*       row_ptr     [[buffer(2)]],
  38:     constant uint2&    dims        [[buffer(3)]],
  39:     uint3              tid         [[thread_position_in_grid]])
  40: {
  41:   const uint I = dims.x;
  42:   const uint B = dims.y;
  43: 
  44:   const uint i = tid.x;
  45:   const uint b = tid.y;
  46: 
  47:   if (b >= B || i > I) return;
  48: 
  49:   const uint base = (uint)batch_ptr[b];
  50:   const uint lim  = (uint)batch_ptr[b + 1];
  51: 
  52:   const ulong out_base = (ulong)b * (ulong)(I + 1);
  53: 
  54:   if (i == I) {
  55:     row_ptr[out_base + (ulong)I] = (long)lim;
  56:   } else {
  57:     const long key = (long)i;
  58:     const uint pos = lower_bound_i64(rows, base, lim, key);
  59:     row_ptr[out_base + (ulong)i] = (long)pos;
  60:   }
```
- L31: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L32: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L34: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L35: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L36: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L37: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L38: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L39: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L40: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L41: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L42: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L44: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L45: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L47: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L49: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L50: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L52: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L54: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L55: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L56: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L57: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L58: Declares function `lower_bound_i64` as part of this file's callable surface. / 声明函数 `lower_bound_i64`，作为本文件可调用接口的一部分。
- L59: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L60: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 61-90

```cpp
  61: }
  62: 
  63: template <typename T>
  64: kernel void spmm_bmm_coo_rows_grouped(
  65:     device const long*   cols      [[buffer(1)]],
  66:     device const T*      vals      [[buffer(2)]],
  67:     device const T*      dense     [[buffer(3)]],
  68:     device T*            out       [[buffer(4)]],
  69:     device const long*   row_ptr   [[buffer(5)]],
  70:     constant uint4&      dims      [[buffer(6)]],
  71:     uint3                tid       [[thread_position_in_grid]],
  72:     uint3                ltid      [[thread_position_in_threadgroup]],
  73:     uint3                tptg      [[threads_per_threadgroup]])
  74: {
  75:   const uint I = dims.y;
  76:   const uint J = dims.z;
  77:   const uint K = dims.w;
  78: 
  79:   const uint b = tid.z;
  80:   const uint i = tid.y;
  81:   const uint lane = ltid.x;
  82:   const uint tgW  = tptg.x;
  83: 
  84:   const ulong rp_base = (ulong)b * (ulong)(I + 1);
  85:   const uint start = (uint)row_ptr[rp_base + (ulong)i];
  86:   const uint end   = (uint)row_ptr[rp_base + (ulong)i + 1];
  87: 
  88:   for (uint k = lane; k < K; k += tgW) {
  89:     auto acc = static_cast<accum_t<T>>(T(0));
  90:     for (uint p = start; p < end; ++p) {
```
- L61: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L63: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L64: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L65: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L66: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L67: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L68: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L69: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L70: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L71: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L72: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L73: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L74: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L75: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L76: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L77: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L79: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L80: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L81: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L82: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L84: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L85: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L86: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L88: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L89: Declares function `T` as part of this file's callable surface. / 声明函数 `T`，作为本文件可调用接口的一部分。
- L90: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。

### Lines 91-120

```cpp
  91:       const uint c = (uint)cols[p];
  92:       const auto v = static_cast<accum_t<T>>(vals[p]);
  93:       const uint d_off = ((b * J) + c) * K + k;
  94:       const auto d = static_cast<accum_t<T>>(dense[d_off]);
  95:       acc += mul(v, d);
  96:     }
  97:     const uint y_off = ((b * I) + i) * K + k;
  98:     out[y_off] = static_cast<T>(acc);
  99:   }
 100: }
 101: 
 102: template <typename T>
 103: kernel void dense_sparse_mul_kernel(
 104:     device const T* dense         [[buffer(0)]],
 105:     device const T* values        [[buffer(1)]],
 106:     device T* out_values          [[buffer(2)]],
 107:     device const long* indices    [[buffer(3)]],
 108:     device const long* sizes      [[buffer(4)]],
 109:     constant uint3& sparse_params [[buffer(5)]],
 110:     uint3 gid                     [[thread_position_in_grid]])
 111: {
 112:   uint col = gid.x;
 113:   uint i = gid.z;
 114:   uint nnz = sparse_params.x;
 115:   uint ndim_i = sparse_params.y;
 116:   uint view_cols = sparse_params.z;
 117: 
 118:   long key = 0;
 119:   for (uint d = 0; d < ndim_i; ++d) {
 120:     long idx_d = indices[(ulong)d * (ulong)nnz + (ulong)i];
```
- L91: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L92: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L93: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L94: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L95: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L96: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L97: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L98: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L99: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L100: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L102: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L103: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L104: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L105: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L106: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L107: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L108: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L109: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L110: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L111: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L112: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L113: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L114: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L115: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L116: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L118: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L119: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L120: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 121-150

```cpp
 121:     const auto sz_d  = sizes[d];
 122:     key = key * sz_d + idx_d;
 123:   }
 124: 
 125:   ulong dense_idx = (ulong)key * (ulong)view_cols + (ulong)col;
 126:   ulong val_idx = (ulong)i * (ulong)view_cols + (ulong)col;
 127: 
 128:   const auto a = static_cast<accum_t<T>>(values[val_idx]);
 129:   const auto b = static_cast<accum_t<T>>(dense[dense_idx]);
 130:   out_values[val_idx] = static_cast<T>(mul(a, b));
 131: }
 132: 
 133: kernel void intersect_binary_search(
 134:     device const long*  keysA        [[buffer(0)]],
 135:     device const long*  keysB        [[buffer(1)]],
 136:     device long*        outA_idx     [[buffer(2)]],
 137:     device long*        outB_idx     [[buffer(3)]],
 138:     device atomic_uint* counter      [[buffer(4)]],
 139:     constant uint&      lenB         [[buffer(5)]],
 140:     constant bool&      A_is_lhs     [[buffer(6)]],
 141:     uint3               tid_in_grid  [[thread_position_in_grid]])
 142: {
 143:   uint gid = tid_in_grid.x;
 144: 
 145:   long key = keysA[gid];
 146: 
 147:   // lower_bound in B
 148:   uint lo = 0;
 149:   uint hi = lenB;
 150:   while (lo < hi) {
```
- L121: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L122: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L123: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L125: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L126: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L128: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L129: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L130: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L131: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L133: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L134: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L135: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L136: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L137: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L138: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L139: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L140: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L141: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L142: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L143: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L145: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L147: Documents the nearby logic: lower_bound in B / 说明附近逻辑的作用：lower_bound in B
- L148: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L149: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L150: Starts a loop that repeats while the controlling condition remains true. / 开始一个在控制条件保持为真时重复执行的循环。

### Lines 151-180

```cpp
 151:     uint mid = (lo + hi) >> 1;
 152:     long v = keysB[mid];
 153:     if (v < key) lo = mid + 1;
 154:     else         hi = mid;
 155:   }
 156: 
 157:   if (lo < lenB && keysB[lo] == key) {
 158:     uint pos = atomic_fetch_add_explicit(counter, 1u, memory_order_relaxed);
 159:     if (A_is_lhs) {
 160:       outA_idx[pos] = (long)gid;
 161:       outB_idx[pos] = (long)lo;
 162:     } else {
 163:       outA_idx[pos] = (long)lo;
 164:       outB_idx[pos] = (long)gid;
 165:     }
 166:   }
 167: }
 168: 
 169: 
 170: template <typename T>
 171: kernel void fused_gather_mul_kernel(
 172:     device const T*    lhs_vals      [[buffer(0)]],
 173:     device const T*    rhs_vals      [[buffer(1)]],
 174:     device const long* lhs_sel       [[buffer(2)]],
 175:     device const long* rhs_sel       [[buffer(3)]],
 176:     device const long* lhs_indices   [[buffer(4)]],
 177:     device long*       out_indices   [[buffer(5)]],
 178:     device T*          out_vals      [[buffer(6)]],
 179:     constant uint2&    dims_input    [[buffer(7)]],
 180:     constant uint2&    dims_output   [[buffer(8)]],
```
- L151: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L152: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L153: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L154: Provides the fallback branch when earlier conditions are not satisfied. / 在前序条件不满足时提供兜底分支。
- L155: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L157: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L158: Declares function `atomic_fetch_add_explicit` as part of this file's callable surface. / 声明函数 `atomic_fetch_add_explicit`，作为本文件可调用接口的一部分。
- L159: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L160: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L161: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L162: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L163: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L164: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L165: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L166: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L167: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L170: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L171: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L172: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L173: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L174: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L175: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L176: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L177: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L178: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L179: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L180: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 181-210

```cpp
 181:     uint3              gid           [[thread_position_in_grid]])
 182: {
 183:   const uint col = gid.x;
 184:   const uint k = gid.z;
 185:   const uint n_dim_i = dims_input.x;
 186:   const uint L = dims_input.y;
 187:   const uint M = dims_output.x;
 188:   const uint view_cols = dims_output.y;
 189: 
 190:   const long iL = lhs_sel[k];
 191:   const long iR = rhs_sel[k];
 192: 
 193:   if (col < view_cols) {
 194:     const ulong offL = (ulong)iL * (ulong)view_cols + (ulong)col;
 195:     const ulong offR = (ulong)iR * (ulong)view_cols + (ulong)col;
 196:     const ulong offO = (ulong)k  * (ulong)view_cols + (ulong)col;
 197: 
 198:     const auto a = static_cast<accum_t<T>>(lhs_vals[offL]);
 199:     const auto b = static_cast<accum_t<T>>(rhs_vals[offR]);
 200:     out_vals[offO] = static_cast<T>(mul(a, b));
 201:   }
 202: 
 203:   // One thread per match copies the indices column
 204:   if (col == 0) {
 205:     const ulong uL = (ulong)L;
 206:     const ulong uM = (ulong)M;
 207:     const ulong src_col = (ulong)iL; // gather from lhs
 208:     for (uint d = 0; d < n_dim_i; ++d) {
 209:       const long v = lhs_indices[(ulong)d * uL + src_col];
 210:       out_indices[(ulong)d * uM + (ulong)k] = v;
```
- L181: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L182: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L183: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L184: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L185: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L186: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L187: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L188: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L190: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L191: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L193: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L194: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L195: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L196: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L198: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L199: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L200: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L201: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L203: Documents the nearby logic: One thread per match copies the indices column / 说明附近逻辑的作用：One thread per match copies the indices column
- L204: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L205: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L206: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L207: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L208: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L209: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L210: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 211-240

```cpp
 211:     }
 212:   }
 213: }
 214: 
 215: 
 216: kernel void build_batch_ptr_from_sorted_batches(
 217:     device const long* batches       [[buffer(0)]],
 218:     device long*       batch_ptr     [[buffer(1)]],
 219:     constant uint2&    nnz_B         [[buffer(2)]],
 220:     uint3              tid           [[thread_position_in_grid]])
 221: {
 222:   uint b = tid.x;
 223:   uint nnz = nnz_B.x;
 224:   uint batch = nnz_B.y;
 225: 
 226:   if (b == batch) {
 227:     batch_ptr[b] = (long)nnz;
 228:     return;
 229:   }
 230: 
 231:   uint lo = 0;
 232:   uint hi = nnz;
 233:   long key = (long)b;
 234:   while (lo < hi) {
 235:     uint mid = (lo + hi) >> 1;
 236:     long v = batches[mid];
 237:     if (v < key) lo = mid + 1;
 238:     else         hi = mid;
 239:   }
 240:   batch_ptr[b] = (long)lo;
```
- L211: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L212: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L213: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L216: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L217: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L218: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L219: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L220: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L221: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L222: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L223: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L224: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L226: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L227: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L228: Returns from the current function without producing a value. / 从当前函数直接返回，不产生返回值。
- L229: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L231: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L232: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L233: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L234: Starts a loop that repeats while the controlling condition remains true. / 开始一个在控制条件保持为真时重复执行的循环。
- L235: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L236: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L237: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L238: Provides the fallback branch when earlier conditions are not satisfied. / 在前序条件不满足时提供兜底分支。
- L239: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L240: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 241-270

```cpp
 241: }
 242: 
 243: template <typename T>
 244: kernel void spmm_addmm_coo(
 245:     device const long*   indices2d   [[buffer(0)]],
 246:     device const T*      vals        [[buffer(1)]],
 247:     device const T*      dense       [[buffer(2)]],
 248:     device const T*      t_in        [[buffer(3)]],
 249:     device T*            out         [[buffer(4)]],
 250:     constant uint3&      dims        [[buffer(5)]],
 251:     constant float2&     alpha_beta  [[buffer(6)]],
 252:     constant uint&       nnz         [[buffer(7)]],
 253:     uint3                tid         [[thread_position_in_grid]])
 254: {
 255:   const uint K = dims.z;
 256:   const uint k = tid.x;
 257:   const uint i = tid.z;
 258:   const float alpha = alpha_beta.x;
 259:   const float beta = alpha_beta.y;
 260: 
 261:   device const long* rows = indices2d;
 262:   device const long* cols = indices2d + nnz;
 263: 
 264:   const uint start = lower_bound_i64(rows, 0u, nnz, (long)i);
 265:   const uint end = upper_bound_i64(rows, 0u, nnz, (long)i);
 266: 
 267:   // accumulator is float for scalar/half/bfloat and float2 for float2
 268:   auto acc = static_cast<accum_t<T>>(T(0));
 269: 
 270:   for (uint p = start; p < end; ++p) {
```
- L241: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L243: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L244: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L245: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L246: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L247: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L248: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L249: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L250: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L251: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L252: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L253: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L254: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L255: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L256: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L257: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L258: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L259: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L261: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L262: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L264: Declares function `lower_bound_i64` as part of this file's callable surface. / 声明函数 `lower_bound_i64`，作为本文件可调用接口的一部分。
- L265: Declares function `upper_bound_i64` as part of this file's callable surface. / 声明函数 `upper_bound_i64`，作为本文件可调用接口的一部分。
- L267: Documents the nearby logic: accumulator is float for scalar/half/bfloat and float2 for float2 / 说明附近逻辑的作用：accumulator is float for scalar/half/bfloat and float2 for float2
- L268: Declares function `T` as part of this file's callable surface. / 声明函数 `T`，作为本文件可调用接口的一部分。
- L270: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。

### Lines 271-300

```cpp
 271:     const uint c = (uint)cols[p];
 272:     const auto v = static_cast<accum_t<T>>(vals[p]);
 273:     const uint dense_off = c * K + k;
 274:     const auto d = static_cast<accum_t<T>>(dense[dense_off]);
 275:     acc += mul(v, d);
 276:   }
 277: 
 278:   const uint off = i * K + k;
 279:   const auto base = (beta != 0.0f) ? (static_cast<accum_t<T>>(t_in[off]) * beta) : static_cast<accum_t<T>>(T(0));
 280:   const auto y = base + alpha * acc;
 281:   out[off] = static_cast<T>(y);
 282: }
 283: 
 284: 
 285: kernel void mark_segments(
 286:     device const int64_t* indices [[buffer(0)]],
 287:     device int*           mask    [[buffer(1)]],
 288:     uint                  tid     [[thread_position_in_grid]])
 289: {
 290:     mask[tid] = (tid == 0 || indices[tid] != indices[tid - 1]) ? 1 : 0;
 291: }
 292: 
 293: kernel void compute_offsets_and_counts(
 294:     device const int* scan           [[buffer(0)]],
 295:     device int*       offsets        [[buffer(1)]],
 296:     device int*       counts         [[buffer(2)]],
 297:     constant uint&    total_elements [[buffer(3)]],
 298:     uint              tid            [[thread_position_in_grid]])
 299: {
 300:     int num_pools = scan[total_elements - 1];
```
- L271: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L272: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L273: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L274: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L275: Declares function `mul` as part of this file's callable surface. / 声明函数 `mul`，作为本文件可调用接口的一部分。
- L276: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L278: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L279: Declares function `T` as part of this file's callable surface. / 声明函数 `T`，作为本文件可调用接口的一部分。
- L280: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L281: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L282: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L285: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L286: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L287: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L288: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L289: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L290: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L291: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L293: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L294: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L295: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L296: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L297: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L298: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L299: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L300: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 301-330

```cpp
 301: 
 302:     int target = int(tid) + 1;
 303:     int lo = 0;
 304:     int hi = int(total_elements);
 305:     while (lo < hi) {
 306:         int mid = lo + (hi - lo) / 2;
 307:         if (scan[mid] < target) {
 308:             lo = mid + 1;
 309:         } else {
 310:             hi = mid;
 311:         }
 312:     }
 313:     int start = lo;
 314:     offsets[tid] = start;
 315: 
 316:     int end;
 317:     if (int(tid) == num_pools - 1) {
 318:         end = int(total_elements);
 319:     } else {
 320:         target = int(tid) + 2;
 321:         lo = start + 1;
 322:         hi = int(total_elements);
 323:         while (lo < hi) {
 324:             int mid = lo + (hi - lo) / 2;
 325:             if (scan[mid] < target) {
 326:                 lo = mid + 1;
 327:             } else {
 328:                 hi = mid;
 329:             }
 330:         }
```
- L302: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L303: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L304: Declares function `int` as part of this file's callable surface. / 声明函数 `int`，作为本文件可调用接口的一部分。
- L305: Starts a loop that repeats while the controlling condition remains true. / 开始一个在控制条件保持为真时重复执行的循环。
- L306: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L307: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L308: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L309: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L310: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L311: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L312: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L313: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L314: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L316: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L317: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L318: Declares function `int` as part of this file's callable surface. / 声明函数 `int`，作为本文件可调用接口的一部分。
- L319: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L320: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L321: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L322: Declares function `int` as part of this file's callable surface. / 声明函数 `int`，作为本文件可调用接口的一部分。
- L323: Starts a loop that repeats while the controlling condition remains true. / 开始一个在控制条件保持为真时重复执行的循环。
- L324: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L325: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L326: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L327: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L328: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L329: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L330: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 331-360

```cpp
 331:         end = lo;
 332:     }
 333: 
 334:     counts[tid] = end - start;
 335: }
 336: 
 337: template <typename T>
 338: kernel void softmax_sparse_forward(
 339:     device const T*    values       [[buffer(0)]],
 340:     device T*          output       [[buffer(1)]],
 341:     device const int*  pool_offsets [[buffer(2)]],
 342:     device const int*  pool_sizes   [[buffer(3)]],
 343:     device const int*  scan         [[buffer(4)]],
 344:     constant uint2&    nnz_nvalues  [[buffer(5)]],
 345:     constant bool&     is_log       [[buffer(6)]],
 346:     uint               tid          [[thread_position_in_grid]])
 347: {
 348:     uint nnz = nnz_nvalues.x;
 349:     uint nvalues = nnz_nvalues.y;
 350:     int num_pools = scan[nnz - 1];
 351:     if (tid >= uint(num_pools)) return;
 352: 
 353:     int start = pool_offsets[tid];
 354:     int count = pool_sizes[tid];
 355: 
 356:     for (uint j = 0; j < nvalues; ++j) {
 357:         float max_val = -INFINITY;
 358:         for (int i = 0; i < count; ++i) {
 359:             float val = static_cast<float>(values[(start + i) * nvalues + j]);
 360:             if (val > max_val) max_val = val;
```
- L331: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L332: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L334: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L335: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L337: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L338: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L339: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L340: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L341: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L342: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L343: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L344: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L345: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L346: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L347: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L348: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L349: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L350: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L351: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L353: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L354: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L356: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L357: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L358: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L359: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L360: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。

### Lines 361-390

```cpp
 361:         }
 362: 
 363:         float sum_exp = 0.0f;
 364:         for (int i = 0; i < count; ++i) {
 365:             float val = static_cast<float>(values[(start + i) * nvalues + j]);
 366:             sum_exp += exp(val - max_val);
 367:         }
 368: 
 369:         float log_sum = is_log ? log(sum_exp) : 0.0f;
 370:         float inv_sum = is_log ? 0.0f : (1.0f / sum_exp);
 371: 
 372:         for (int i = 0; i < count; ++i) {
 373:             uint idx = (start + i) * nvalues + j;
 374:             float val = static_cast<float>(values[idx]);
 375: 
 376:             if (is_log) {
 377:                 output[idx] = static_cast<T>(val - max_val - log_sum);
 378:             } else {
 379:                 output[idx] = static_cast<T>(exp(val - max_val) * inv_sum);
 380:             }
 381:         }
 382:     }
 383: }
 384: 
 385: template <typename T>
 386: kernel void softmax_sparse_backward(
 387:     device const T*    grad_output  [[buffer(0)]],
 388:     device const T*    output       [[buffer(1)]],
 389:     device T*          grad_input   [[buffer(2)]],
 390:     device const int*  offsets      [[buffer(3)]],
```
- L361: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L363: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L364: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L365: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L366: Declares function `exp` as part of this file's callable surface. / 声明函数 `exp`，作为本文件可调用接口的一部分。
- L367: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L369: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L370: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L372: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L373: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L374: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L376: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L377: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L378: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L379: Declares function `exp` as part of this file's callable surface. / 声明函数 `exp`，作为本文件可调用接口的一部分。
- L380: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L381: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L382: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L383: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L385: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L386: Declares an entry-point GPU kernel for Metal execution. / 声明一个面向 Metal 执行的 GPU 入口内核。
- L387: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L388: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L389: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L390: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 391-420

```cpp
 391:     device const int*  counts       [[buffer(4)]],
 392:     device const int*  scan         [[buffer(5)]],
 393:     constant uint2&    nnz_nvalues  [[buffer(6)]],
 394:     constant bool&     is_log       [[buffer(7)]],
 395:     uint               tid          [[thread_position_in_grid]])
 396: {
 397:     uint nnz = nnz_nvalues.x;
 398:     uint nvalues = nnz_nvalues.y;
 399:     int num_pools = scan[nnz - 1];
 400:     if (tid >= uint(num_pools)) return;
 401: 
 402:     int start = offsets[tid];
 403:     int count = counts[tid];
 404: 
 405:     for (uint j = 0; j < nvalues; ++j) {
 406:         float sum_val = 0.0f;
 407: 
 408:         for (int i = 0; i < count; ++i) {
 409:             uint idx = (start + i) * nvalues + j;
 410:             float g = static_cast<float>(grad_output[idx]);
 411:             if (is_log) {
 412:                 sum_val += g;
 413:             } else {
 414:                 float y = static_cast<float>(output[idx]);
 415:                 sum_val += g * y;
 416:             }
 417:         }
 418: 
 419:         for (int i = 0; i < count; ++i) {
 420:             uint idx = (start + i) * nvalues + j;
```
- L391: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L392: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L393: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L394: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L395: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L396: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L397: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L398: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L399: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L400: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L402: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L403: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L405: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L406: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L408: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L409: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L410: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L411: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L412: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L413: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L414: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L415: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L416: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L417: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L419: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L420: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 421-450

```cpp
 421:             float g = static_cast<float>(grad_output[idx]);
 422:             float y = static_cast<float>(output[idx]);
 423:             float res;
 424: 
 425:             if (is_log) {
 426:                 res = g - exp(y) * sum_val;
 427:             } else {
 428:                 res = y * (g - sum_val);
 429:             }
 430:             grad_input[idx] = static_cast<T>(res);
 431:         }
 432:     }
 433: }
 434: 
 435: #define INSTANTIATE_SOFTMAX_SPARSE_FORWARD(DTYPE)                           \
 436:   template [[host_name("softmax_sparse_forward_" #DTYPE)]] kernel void      \
 437:   softmax_sparse_forward<DTYPE>(                                            \
 438:       device const DTYPE* values       [[buffer(0)]],                       \
 439:       device DTYPE*       output       [[buffer(1)]],                       \
 440:       device const int*   pool_offsets [[buffer(2)]],                       \
 441:       device const int*   pool_sizes   [[buffer(3)]],                       \
 442:       device const int*   scan         [[buffer(4)]],                       \
 443:       constant uint2&     nnz_nvalues  [[buffer(5)]],                       \
 444:       constant bool&      is_log       [[buffer(6)]],                       \
 445:       uint                tid          [[thread_position_in_grid]]);
 446: 
 447: #define INSTANTIATE_SOFTMAX_SPARSE_BACKWARD(DTYPE)                          \
 448:   template [[host_name("softmax_sparse_backward_" #DTYPE)]] kernel void     \
 449:   softmax_sparse_backward<DTYPE>(                                           \
 450:       device const DTYPE* grad         [[buffer(0)]],                       \
```
- L421: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L422: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L423: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L425: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L426: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L427: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L428: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L429: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L430: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L431: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L432: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L433: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L435: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L436: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L437: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L438: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L439: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L440: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L441: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L442: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L443: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L444: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L445: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L447: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L448: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L449: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L450: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 451-480

```cpp
 451:       device const DTYPE* output       [[buffer(1)]],                       \
 452:       device DTYPE*       grad_input   [[buffer(2)]],                       \
 453:       device const int*   pool_offsets [[buffer(3)]],                       \
 454:       device const int*   pool_sizes   [[buffer(4)]],                       \
 455:       device const int*   scan         [[buffer(5)]],                       \
 456:       constant uint2&     nnz_nvalues  [[buffer(6)]],                       \
 457:       constant bool&      is_log       [[buffer(7)]],                       \
 458:       uint                tid          [[thread_position_in_grid]]);
 459: 
 460: 
 461: INSTANTIATE_FOR_FLOAT_TYPES(INSTANTIATE_SOFTMAX_SPARSE_FORWARD);
 462: INSTANTIATE_FOR_FLOAT_TYPES(INSTANTIATE_SOFTMAX_SPARSE_BACKWARD);
 463: 
 464: 
 465: #define INSTANTIATE_DENSE_SPARSE_MUL(DTYPE)                                 \
 466:   template [[host_name("dense_sparse_mul_kernel_" #DTYPE)]] kernel void     \
 467:   dense_sparse_mul_kernel<DTYPE>(                                           \
 468:       device const DTYPE* dense         [[buffer(0)]],                      \
 469:       device const DTYPE* values        [[buffer(1)]],                      \
 470:       device DTYPE* out_values          [[buffer(2)]],                      \
 471:       device const long* indices        [[buffer(3)]],                      \
 472:       device const long* sizes          [[buffer(4)]],                      \
 473:       constant uint3& sparse_params     [[buffer(5)]],                      \
 474:       uint3 gid                         [[thread_position_in_grid]]);
 475: 
 476: INSTANTIATE_DENSE_SPARSE_MUL(float);
 477: INSTANTIATE_DENSE_SPARSE_MUL(half);
 478: INSTANTIATE_DENSE_SPARSE_MUL(bfloat);
 479: INSTANTIATE_DENSE_SPARSE_MUL(long);
 480: INSTANTIATE_DENSE_SPARSE_MUL(float2);
```
- L451: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L452: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L453: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L454: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L455: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L456: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L457: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L458: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L461: Declares function `INSTANTIATE_FOR_FLOAT_TYPES` as part of this file's callable surface. / 声明函数 `INSTANTIATE_FOR_FLOAT_TYPES`，作为本文件可调用接口的一部分。
- L462: Declares function `INSTANTIATE_FOR_FLOAT_TYPES` as part of this file's callable surface. / 声明函数 `INSTANTIATE_FOR_FLOAT_TYPES`，作为本文件可调用接口的一部分。
- L465: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L466: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L467: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L468: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L469: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L470: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L471: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L472: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L473: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L474: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L476: Declares function `INSTANTIATE_DENSE_SPARSE_MUL` as part of this file's callable surface. / 声明函数 `INSTANTIATE_DENSE_SPARSE_MUL`，作为本文件可调用接口的一部分。
- L477: Declares function `INSTANTIATE_DENSE_SPARSE_MUL` as part of this file's callable surface. / 声明函数 `INSTANTIATE_DENSE_SPARSE_MUL`，作为本文件可调用接口的一部分。
- L478: Declares function `INSTANTIATE_DENSE_SPARSE_MUL` as part of this file's callable surface. / 声明函数 `INSTANTIATE_DENSE_SPARSE_MUL`，作为本文件可调用接口的一部分。
- L479: Declares function `INSTANTIATE_DENSE_SPARSE_MUL` as part of this file's callable surface. / 声明函数 `INSTANTIATE_DENSE_SPARSE_MUL`，作为本文件可调用接口的一部分。
- L480: Declares function `INSTANTIATE_DENSE_SPARSE_MUL` as part of this file's callable surface. / 声明函数 `INSTANTIATE_DENSE_SPARSE_MUL`，作为本文件可调用接口的一部分。

### Lines 481-510

```cpp
 481: 
 482: #define INSTANTIATE_FUSED_GATHER_MUL(DTYPE)                                  \
 483:   template [[host_name("fused_gather_mul_kernel_" #DTYPE)]] kernel void      \
 484:   fused_gather_mul_kernel<DTYPE>(                                            \
 485:       device const DTYPE* lhs_vals      [[buffer(0)]],                       \
 486:       device const DTYPE* rhs_vals      [[buffer(1)]],                       \
 487:       device const long*  lhs_sel       [[buffer(2)]],                       \
 488:       device const long*  rhs_sel       [[buffer(3)]],                       \
 489:       device const long*  lhs_indices   [[buffer(4)]],                       \
 490:       device long*        out_indices   [[buffer(5)]],                       \
 491:       device DTYPE*       out_vals      [[buffer(6)]],                       \
 492:       constant uint2&     dims_input    [[buffer(7)]],                       \
 493:       constant uint2&     dims_output   [[buffer(8)]],                       \
 494:       uint3               gid           [[thread_position_in_grid]]);
 495: 
 496: INSTANTIATE_FOR_ALL_TYPES(INSTANTIATE_FUSED_GATHER_MUL);
 497: 
 498: 
 499: #define INSTANTIATE_SPMM_BMM_COO_ROWS_GROUPED(DTYPE)                         \
 500:   template [[host_name("spmm_bmm_coo_rows_grouped_" #DTYPE)]] kernel void    \
 501:   spmm_bmm_coo_rows_grouped<DTYPE>(                                          \
 502:       device const long*   cols      [[buffer(1)]],                          \
 503:       device const DTYPE*  vals      [[buffer(2)]],                          \
 504:       device const DTYPE*  dense     [[buffer(3)]],                          \
 505:       device DTYPE*        out       [[buffer(4)]],                          \
 506:       device const long*   row_ptr   [[buffer(5)]],                          \
 507:       constant uint4&      dims      [[buffer(6)]],                          \
 508:       uint3                tid       [[thread_position_in_grid]],            \
 509:       uint3                ltid      [[thread_position_in_threadgroup]],     \
 510:       uint3                tptg      [[threads_per_threadgroup]]);
```
- L482: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L483: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L484: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L485: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L486: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L487: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L488: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L489: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L490: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L491: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L492: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L493: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L494: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L496: Declares function `INSTANTIATE_FOR_ALL_TYPES` as part of this file's callable surface. / 声明函数 `INSTANTIATE_FOR_ALL_TYPES`，作为本文件可调用接口的一部分。
- L499: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L500: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L501: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L502: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L503: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L504: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L505: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L506: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L507: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L508: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L509: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L510: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 511-527

```cpp
 511: 
 512: INSTANTIATE_FOR_ALL_TYPES(INSTANTIATE_SPMM_BMM_COO_ROWS_GROUPED);
 513: 
 514: #define INSTANTIATE_SPMM_ADDMM_COO(DTYPE) \
 515:   template [[host_name("spmm_addmm_coo_" #DTYPE)]] kernel void  \
 516:   spmm_addmm_coo<DTYPE>(                                        \
 517:     device const long*   indices2d   [[buffer(0)]],             \
 518:     device const DTYPE*  vals        [[buffer(1)]],             \
 519:     device const DTYPE*  dense       [[buffer(2)]],             \
 520:     device const DTYPE*  t_in        [[buffer(3)]],             \
 521:     device DTYPE*        out         [[buffer(4)]],             \
 522:     constant uint3&      dims        [[buffer(5)]],             \
 523:     constant float2&     alpha_beta  [[buffer(6)]],             \
 524:     constant uint&       nnz         [[buffer(7)]],             \
 525:     uint3                tid         [[thread_position_in_grid]]);
 526: 
 527: INSTANTIATE_FOR_ALL_TYPES(INSTANTIATE_SPMM_ADDMM_COO);
```
- L512: Declares function `INSTANTIATE_FOR_ALL_TYPES` as part of this file's callable surface. / 声明函数 `INSTANTIATE_FOR_ALL_TYPES`，作为本文件可调用接口的一部分。
- L514: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L515: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L516: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L517: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L518: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L519: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L520: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L521: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L522: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L523: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L524: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L525: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L527: Declares function `INSTANTIATE_FOR_ALL_TYPES` as part of this file's callable surface. / 声明函数 `INSTANTIATE_FOR_ALL_TYPES`，作为本文件可调用接口的一部分。

## Key Concepts / 关键概念

- MPS sparse backend support / MPS 稀疏后端支持
- Sparse tensor math and layout utilities / 稀疏张量数学与布局工具
- Softmax normalization and numerical stability / Softmax 归一化与数值稳定性
- Sparse layout semantics and NNZ traversal / 稀疏布局语义与 NNZ 遍历
- COO index/value representation / COO 索引/数值表示
- Exponentiation inside normalization/reduction / 归一化/归约中的指数运算

## Dependencies / 依赖关系

- `c10/metal/indexing.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- `c10/metal/utils.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- Subsystem tie-in: sparse layouts (COO/CSR/CSC/BSR), index transforms, and NNZ-oriented computation. / 子系统关联：稀疏布局（COO/CSR/CSC/BSR）、索引变换以及面向 NNZ 的计算。
- Runtime dependency: descriptor/binding layout must match the host-side kernel launcher. / 运行时依赖：描述符/绑定布局必须与主机侧内核启动器保持一致。
