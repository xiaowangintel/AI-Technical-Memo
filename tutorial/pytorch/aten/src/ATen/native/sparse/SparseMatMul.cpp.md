# SparseMatMul.cpp — Code Analysis / 代码分析

## Source / 来源

- File: `aten/src/ATen/native/sparse/SparseMatMul.cpp`
- Repository: `pytorch` (`/root/xw/pytorch`)
- Purpose (EN): Implements executable backend logic for Sparse tensor math and layout utilities, centered on Sparse Mat Mul with emphasis on sparse tensor processing.
- 用途（中文）: 实现可执行的后端逻辑，属于稀疏张量数学与布局工具，核心主题是Sparse Mat Mul，重点关注稀疏张量处理。

## Line-by-Line Analysis / 逐行分析

- Note (EN): The file is annotated in contiguous line ranges; each numbered line receives a short bilingual explanation.
- 说明（中文）: 为兼顾可读性，以下按连续行区间展示代码，并对每个编号行给出简短的中英双语说明。

### Lines 1-20

```cpp
   1: #define TORCH_ASSERT_ONLY_METHOD_OPERATORS
   2: #include <ATen/core/Tensor.h>
   3: #include <ATen/Config.h>
   4: #include <ATen/Dispatch.h>
   5: #include <ATen/NamedTensorUtils.h>
   6: #include <ATen/SparseTensorImpl.h>
   7: #include <ATen/native/SparseTensorUtils.h>
   8: #include <ATen/native/Resize.h>
   9: #include <ATen/native/StridedRandomAccessor.h>
  10: #include <ATen/native/CompositeRandomAccessor.h>
  11: #include <c10/util/irange.h>
  12: #include <unordered_map>
  13: 
  14: #ifndef AT_PER_OPERATOR_HEADERS
  15: #include <ATen/Functions.h>
  16: #include <ATen/NativeFunctions.h>
  17: #else
  18: #include <ATen/ops/_sparse_sparse_matmul_native.h>
  19: #include <ATen/ops/empty.h>
  20: #include <ATen/ops/empty_like_native.h>
```
- L1: Defines a macro or placeholder used by the remaining source. / 定义一个供后续源码使用的宏或占位符。
- L2: Includes `ATen/core/Tensor.h` for ATen tensor/operator infrastructure. / 引入 `ATen/core/Tensor.h`，为 ATen 的张量/算子基础设施提供支持。
- L3: Includes `ATen/Config.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Config.h`，为 ATen 的张量/算子基础设施提供支持。
- L4: Includes `ATen/Dispatch.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Dispatch.h`，为 ATen 的张量/算子基础设施提供支持。
- L5: Includes `ATen/NamedTensorUtils.h` for ATen tensor/operator infrastructure. / 引入 `ATen/NamedTensorUtils.h`，为 ATen 的张量/算子基础设施提供支持。
- L6: Includes `ATen/SparseTensorImpl.h` for ATen tensor/operator infrastructure. / 引入 `ATen/SparseTensorImpl.h`，为 ATen 的张量/算子基础设施提供支持。
- L7: Includes `ATen/native/SparseTensorUtils.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/SparseTensorUtils.h`，为 ATen 的张量/算子基础设施提供支持。
- L8: Includes `ATen/native/Resize.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/Resize.h`，为 ATen 的张量/算子基础设施提供支持。
- L9: Includes `ATen/native/StridedRandomAccessor.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/StridedRandomAccessor.h`，为 ATen 的张量/算子基础设施提供支持。
- L10: Includes `ATen/native/CompositeRandomAccessor.h` for ATen tensor/operator infrastructure. / 引入 `ATen/native/CompositeRandomAccessor.h`，为 ATen 的张量/算子基础设施提供支持。
- L11: Includes `c10/util/irange.h` for c10 core runtime, utilities, or dispatch metadata. / 引入 `c10/util/irange.h`，用于 c10 核心运行时、工具或分发元数据。
- L12: Includes `unordered_map` for standard-library or external support. / 引入 `unordered_map`，用于标准库或外部支持。
- L14: Starts a conditional-compilation branch guarded by build-time macros. / 开始一个受构建期宏控制的条件编译分支。
- L15: Includes `ATen/Functions.h` for ATen tensor/operator infrastructure. / 引入 `ATen/Functions.h`，为 ATen 的张量/算子基础设施提供支持。
- L16: Includes `ATen/NativeFunctions.h` for ATen tensor/operator infrastructure. / 引入 `ATen/NativeFunctions.h`，为 ATen 的张量/算子基础设施提供支持。
- L17: Switches to an alternate conditional-compilation branch. / 切换到条件编译的备用分支。
- L18: Includes `ATen/ops/_sparse_sparse_matmul_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/_sparse_sparse_matmul_native.h`，为 ATen 的张量/算子基础设施提供支持。
- L19: Includes `ATen/ops/empty.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/empty.h`，为 ATen 的张量/算子基础设施提供支持。
- L20: Includes `ATen/ops/empty_like_native.h` for ATen tensor/operator infrastructure. / 引入 `ATen/ops/empty_like_native.h`，为 ATen 的张量/算子基础设施提供支持。

### Lines 21-40

```cpp
  21: #endif
  22: 
  23: namespace at::native {
  24: 
  25: using namespace at::sparse;
  26: 
  27: /*
  28:     This is an implementation of the SMMP algorithm:
  29:      "Sparse Matrix Multiplication Package (SMMP)"
  30: 
  31:       Randolph E. Bank and Craig C. Douglas
  32:       https://doi.org/10.1007/BF02070824
  33: */
  34: namespace {
  35: // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
  36: void csr_to_coo(const int64_t n_row, const int64_t Ap[], int64_t Bi[]) {
  37:   /*
  38:     Expands a compressed row pointer into a row indices array
  39:     Inputs:
  40:       `n_row` is the number of rows in `Ap`
```
- L21: Ends the current conditional-compilation region. / 结束当前条件编译区域。
- L23: Opens namespace `at::native` to scope the following declarations. / 打开命名空间 `at::native`，为后续声明限定作用域。
- L25: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L27: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L28: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L29: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L31: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L32: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L33: Documents the nearby logic: / / 说明附近逻辑的作用：/
- L34: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L35: Documents the nearby logic: NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays) / 说明附近逻辑的作用：NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
- L36: Defines function `csr_to_coo` and begins its implementation body. / 定义函数 `csr_to_coo`，并开始其实现体。
- L37: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L38: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L39: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L40: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 41-60

```cpp
  41:       `Ap` is the row pointer
  42: 
  43:     Output:
  44:       `Bi` is the row indices
  45:   */
  46:   for (const auto i : c10::irange(n_row)) {
  47:     for (int64_t jj = Ap[i]; jj < Ap[i + 1]; jj++) {
  48:       Bi[jj] = i;
  49:     }
  50:   }
  51: }
  52: 
  53: template<typename index_t_ptr = int64_t*>
  54: int64_t _csr_matmult_maxnnz(
  55:     const int64_t n_row,
  56:     const int64_t n_col,
  57:     const index_t_ptr Ap,
  58:     const index_t_ptr Aj,
  59:     const index_t_ptr Bp,
  60:     const index_t_ptr Bj) {
```
- L41: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L43: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L44: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L45: Documents the nearby logic: / / 说明附近逻辑的作用：/
- L46: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L47: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L48: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L49: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L50: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L51: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L53: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L54: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L55: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L56: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L57: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L58: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L59: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L60: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。

### Lines 61-80

```cpp
  61:   /*
  62:     Compute needed buffer size for matrix `C` in `C = A@B` operation.
  63: 
  64:     The matrices should be in proper CSR structure, and their dimensions
  65:     should be compatible.
  66:   */
  67:   std::vector<int64_t> mask(n_col, -1);
  68:   int64_t nnz = 0;
  69:   for (const auto i : c10::irange(n_row)) {
  70:     int64_t row_nnz = 0;
  71: 
  72:     for (int64_t jj = Ap[i]; jj < Ap[i + 1]; jj++) {
  73:       int64_t j = Aj[jj];
  74:       for (int64_t kk = Bp[j]; kk < Bp[j + 1]; kk++) {
  75:         int64_t k = Bj[kk];
  76:         if (mask[k] != i) {
  77:           mask[k] = i;
  78:           row_nnz++;
  79:         }
  80:       }
```
- L61: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L62: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L64: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L65: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L66: Documents the nearby logic: / / 说明附近逻辑的作用：/
- L67: Declares function `mask` as part of this file's callable surface. / 声明函数 `mask`，作为本文件可调用接口的一部分。
- L68: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L69: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L70: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L72: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L73: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L74: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L75: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L76: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L77: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L78: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L79: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L80: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 81-100

```cpp
  81:     }
  82:     int64_t next_nnz = nnz + row_nnz;
  83:     nnz = next_nnz;
  84:   }
  85:   return nnz;
  86: }
  87: 
  88: template<typename index_t_ptr, typename scalar_t_ptr>
  89: void _csr_matmult(
  90:     const int64_t n_row,
  91:     const int64_t n_col,
  92:     const index_t_ptr Ap,
  93:     const index_t_ptr Aj,
  94:     const scalar_t_ptr Ax,
  95:     const index_t_ptr Bp,
  96:     const index_t_ptr Bj,
  97:     const scalar_t_ptr Bx,
  98:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
  99:     typename index_t_ptr::value_type Cp[],
 100:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
```
- L81: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L82: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L83: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L84: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L85: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L86: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L88: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L89: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L90: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L91: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L92: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L93: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L94: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L95: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L96: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L97: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L98: Documents the nearby logic: NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays) / 说明附近逻辑的作用：NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
- L99: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L100: Documents the nearby logic: NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays) / 说明附近逻辑的作用：NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)

### Lines 101-120

```cpp
 101:     typename index_t_ptr::value_type Cj[],
 102:     // NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
 103:     typename scalar_t_ptr::value_type Cx[]) {
 104:   /*
 105:     Compute CSR entries for matrix C = A@B.
 106: 
 107:     The matrices `A` and 'B' should be in proper CSR structure, and their dimensions
 108:     should be compatible.
 109: 
 110:     Inputs:
 111:       `n_row`         - number of row in A
 112:       `n_col`         - number of columns in B
 113:       `Ap[n_row+1]`   - row pointer
 114:       `Aj[nnz(A)]`    - column indices
 115:       `Ax[nnz(A)]     - nonzeros
 116:       `Bp[?]`         - row pointer
 117:       `Bj[nnz(B)]`    - column indices
 118:       `Bx[nnz(B)]`    - nonzeros
 119:     Outputs:
 120:       `Cp[n_row+1]` - row pointer
```
- L101: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L102: Documents the nearby logic: NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays) / 说明附近逻辑的作用：NOLINTNEXTLINE(modernize-avoid-c-arrays,cppcoreguidelines-avoid-c-arrays)
- L103: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L104: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L105: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L107: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L108: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L110: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L111: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L112: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L113: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L114: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L115: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L116: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L117: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L118: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L119: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L120: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。

### Lines 121-140

```cpp
 121:       `Cj[nnz(C)]`  - column indices
 122:       `Cx[nnz(C)]`  - nonzeros
 123: 
 124:     Note:
 125:       Output arrays Cp, Cj, and Cx must be preallocated
 126:   */
 127:   using index_t = typename index_t_ptr::value_type;
 128:   using scalar_t = typename scalar_t_ptr::value_type;
 129: 
 130:   std::vector<index_t> next(n_col, -1);
 131:   std::vector<scalar_t> sums(n_col, 0);
 132: 
 133:   int64_t nnz = 0;
 134: 
 135:   Cp[0] = 0;
 136: 
 137:   for (const auto i : c10::irange(n_row)) {
 138:     index_t head = -2;
 139:     index_t length = 0;
 140: 
```
- L121: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L122: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L124: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L125: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L126: Documents the nearby logic: / / 说明附近逻辑的作用：/
- L127: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L128: Introduces a type alias or imported symbol for cleaner downstream code. / 引入类型别名或导入符号，便于后续代码书写。
- L130: Declares function `next` as part of this file's callable surface. / 声明函数 `next`，作为本文件可调用接口的一部分。
- L131: Declares function `sums` as part of this file's callable surface. / 声明函数 `sums`，作为本文件可调用接口的一部分。
- L133: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L135: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L137: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L138: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L139: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。

### Lines 141-160

```cpp
 141:     index_t jj_start = Ap[i];
 142:     index_t jj_end = Ap[i + 1];
 143:     for (const auto jj : c10::irange(jj_start, jj_end)) {
 144:       index_t j = Aj[jj];
 145:       scalar_t v = Ax[jj];
 146: 
 147:       index_t kk_start = Bp[j];
 148:       index_t kk_end = Bp[j + 1];
 149:       for (const auto kk : c10::irange(kk_start, kk_end)) {
 150:         index_t k = Bj[kk];
 151: 
 152:         sums[k] += v * Bx[kk];
 153: 
 154:         if (next[k] == -1) {
 155:           next[k] = head;
 156:           head = k;
 157:           length++;
 158:         }
 159:       }
 160:     }
```
- L141: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L142: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L143: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L144: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L145: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L147: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L148: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L149: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L150: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L152: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L154: Starts a conditional branch that guards the following logic. / 开始一个条件分支，用于保护后续逻辑。
- L155: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L156: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L157: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L158: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L159: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L160: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。

### Lines 161-180

```cpp
 161: 
 162:     for ([[maybe_unused]] const auto jj : c10::irange(length)) {
 163:       // NOTE: the linked list that encodes col indices
 164:       // is not guaranteed to be sorted.
 165:       Cj[nnz] = head;
 166:       Cx[nnz] = sums[head];
 167:       nnz++;
 168: 
 169:       index_t temp = head;
 170:       head = next[head];
 171: 
 172:       next[temp] = -1; // clear arrays
 173:       sums[temp] = 0;
 174:     }
 175: 
 176:     // Make sure that col indices are sorted.
 177:     // TODO: a better approach is to implement a CSR @ CSC kernel.
 178:     // NOTE: Cx arrays are expected to be contiguous!
 179:     auto col_indices_accessor = StridedRandomAccessor<int64_t>(Cj + nnz - length, 1);
 180:     auto val_accessor = StridedRandomAccessor<scalar_t>(Cx + nnz - length, 1);
```
- L162: Starts a loop over indices, tensor elements, or kernel work items. / 开始循环，遍历索引、张量元素或内核工作项。
- L163: Documents the nearby logic: NOTE: the linked list that encodes col indices / 说明附近逻辑的作用：NOTE: the linked list that encodes col indices
- L164: Documents the nearby logic: is not guaranteed to be sorted. / 说明附近逻辑的作用：is not guaranteed to be sorted.
- L165: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L166: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L167: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L169: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L170: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L172: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L173: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L174: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L176: Documents the nearby logic: Make sure that col indices are sorted. / 说明附近逻辑的作用：Make sure that col indices are sorted.
- L177: Documents the nearby logic: TODO: a better approach is to implement a CSR @ CSC kernel. / 说明附近逻辑的作用：TODO: a better approach is to implement a CSR @ CSC kernel.
- L178: Documents the nearby logic: NOTE: Cx arrays are expected to be contiguous! / 说明附近逻辑的作用：NOTE: Cx arrays are expected to be contiguous!
- L179: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L180: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。

### Lines 181-200

```cpp
 181:     auto kv_accessor = CompositeRandomAccessorCPU<
 182:       decltype(col_indices_accessor), decltype(val_accessor)
 183:     >(col_indices_accessor, val_accessor);
 184:     std::sort(kv_accessor, kv_accessor + length, [](const auto& lhs, const auto& rhs) -> bool {
 185:         return get<0>(lhs) < get<0>(rhs);
 186:     });
 187: 
 188:     Cp[i + 1] = nnz;
 189:   }
 190: }
 191: 
 192: 
 193: template <typename scalar_t>
 194: void sparse_matmul_kernel(
 195:     Tensor& output,
 196:     const Tensor& mat1,
 197:     const Tensor& mat2) {
 198:   /*
 199:     Computes  the sparse-sparse matrix multiplication between `mat1` and `mat2`, which are sparse tensors in COO format.
 200:   */
```
- L181: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L182: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L183: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L184: Defines function `sort` and begins its implementation body. / 定义函数 `sort`，并开始其实现体。
- L185: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L186: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L188: Initializes or updates state consumed by the surrounding algorithm. / 初始化或更新周围算法所使用的状态。
- L189: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L190: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L193: Begins a template so the following entity can be specialized for multiple types. / 开始模板声明，使后续实体可针对多种类型进行特化。
- L194: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L195: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L196: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L197: Opens a new block for the declaration or control structure above it. / 为上方声明或控制结构打开新的代码块。
- L198: Provides commentary for nearby code. / 为附近代码提供注释说明。
- L199: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L200: Documents the nearby logic: / / 说明附近逻辑的作用：/

### Lines 201-220

```cpp
 201: 
 202:   auto M = mat1.size(0);
 203:   auto N = mat2.size(1);
 204: 
 205:   const auto mat1_csr = mat1.to_sparse_csr();
 206:   const auto mat2_csr = mat2.to_sparse_csr();
 207: 
 208:   auto mat1_crow_indices_ptr = StridedRandomAccessor<int64_t>(
 209:       mat1_csr.crow_indices().data_ptr<int64_t>(),
 210:       mat1_csr.crow_indices().stride(-1));
 211:   auto mat1_col_indices_ptr = StridedRandomAccessor<int64_t>(
 212:       mat1_csr.col_indices().data_ptr<int64_t>(),
 213:       mat1_csr.col_indices().stride(-1));
 214:   auto mat1_values_ptr = StridedRandomAccessor<scalar_t>(
 215:       mat1_csr.values().data_ptr<scalar_t>(),
 216:       mat1_csr.values().stride(-1));
 217:   auto mat2_crow_indices_ptr = StridedRandomAccessor<int64_t>(
 218:       mat2_csr.crow_indices().data_ptr<int64_t>(),
 219:       mat2_csr.crow_indices().stride(-1));
 220:   auto mat2_col_indices_ptr = StridedRandomAccessor<int64_t>(
```
- L202: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L203: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L205: Declares function `to_sparse_csr` as part of this file's callable surface. / 声明函数 `to_sparse_csr`，作为本文件可调用接口的一部分。
- L206: Declares function `to_sparse_csr` as part of this file's callable surface. / 声明函数 `to_sparse_csr`，作为本文件可调用接口的一部分。
- L208: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L209: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L210: Declares function `crow_indices` as part of this file's callable surface. / 声明函数 `crow_indices`，作为本文件可调用接口的一部分。
- L211: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L212: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L213: Declares function `col_indices` as part of this file's callable surface. / 声明函数 `col_indices`，作为本文件可调用接口的一部分。
- L214: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L215: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L216: Declares function `values` as part of this file's callable surface. / 声明函数 `values`，作为本文件可调用接口的一部分。
- L217: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L218: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L219: Declares function `crow_indices` as part of this file's callable surface. / 声明函数 `crow_indices`，作为本文件可调用接口的一部分。
- L220: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。

### Lines 221-240

```cpp
 221:       mat2_csr.col_indices().data_ptr<int64_t>(),
 222:       mat2_csr.col_indices().stride(-1));
 223:   auto mat2_values_ptr = StridedRandomAccessor<scalar_t>(
 224:       mat2_csr.values().data_ptr<scalar_t>(),
 225:       mat2_csr.values().stride(-1));
 226: 
 227:   const auto nnz = _csr_matmult_maxnnz(
 228:       M,
 229:       N,
 230:       mat1_crow_indices_ptr,
 231:       mat1_col_indices_ptr,
 232:       mat2_crow_indices_ptr,
 233:       mat2_col_indices_ptr);
 234: 
 235:   auto output_indices = output._indices();
 236:   auto output_values = output._values();
 237: 
 238:   Tensor output_indptr = at::empty({M + 1}, kLong);
 239:   at::native::resize_output(output_indices, {2, nnz});
 240:   at::native::resize_output(output_values, nnz);
```
- L221: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L222: Declares function `col_indices` as part of this file's callable surface. / 声明函数 `col_indices`，作为本文件可调用接口的一部分。
- L223: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L224: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L225: Declares function `values` as part of this file's callable surface. / 声明函数 `values`，作为本文件可调用接口的一部分。
- L227: Introduces a local variable with a type deduced from its initializer. / 引入一个局部变量，其类型由初始化表达式推导。
- L228: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L229: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L230: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L231: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L232: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L233: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L235: Declares function `_indices` as part of this file's callable surface. / 声明函数 `_indices`，作为本文件可调用接口的一部分。
- L236: Declares function `_values` as part of this file's callable surface. / 声明函数 `_values`，作为本文件可调用接口的一部分。
- L238: Declares function `empty` as part of this file's callable surface. / 声明函数 `empty`，作为本文件可调用接口的一部分。
- L239: Declares function `resize_output` as part of this file's callable surface. / 声明函数 `resize_output`，作为本文件可调用接口的一部分。
- L240: Declares function `resize_output` as part of this file's callable surface. / 声明函数 `resize_output`，作为本文件可调用接口的一部分。

### Lines 241-260

```cpp
 241: 
 242:   Tensor output_row_indices = output_indices.select(0, 0);
 243:   Tensor output_col_indices = output_indices.select(0, 1);
 244: 
 245:   // TODO: replace with a CSR @ CSC kernel for better performance.
 246:   _csr_matmult(
 247:       M,
 248:       N,
 249:       mat1_crow_indices_ptr,
 250:       mat1_col_indices_ptr,
 251:       mat1_values_ptr,
 252:       mat2_crow_indices_ptr,
 253:       mat2_col_indices_ptr,
 254:       mat2_values_ptr,
 255:       output_indptr.data_ptr<int64_t>(),
 256:       output_col_indices.data_ptr<int64_t>(),
 257:       output_values.data_ptr<scalar_t>());
 258: 
 259:   csr_to_coo(M, output_indptr.data_ptr<int64_t>(), output_row_indices.data_ptr<int64_t>());
 260:   output._coalesced_(true);
```
- L242: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L243: Declares function `select` as part of this file's callable surface. / 声明函数 `select`，作为本文件可调用接口的一部分。
- L245: Documents the nearby logic: TODO: replace with a CSR @ CSC kernel for better performance. / 说明附近逻辑的作用：TODO: replace with a CSR @ CSC kernel for better performance.
- L246: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L247: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L248: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L249: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L250: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L251: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L252: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L253: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L254: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L255: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L256: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L257: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L259: Declares function `csr_to_coo` as part of this file's callable surface. / 声明函数 `csr_to_coo`，作为本文件可调用接口的一部分。
- L260: Declares function `_coalesced_` as part of this file's callable surface. / 声明函数 `_coalesced_`，作为本文件可调用接口的一部分。

### Lines 261-280

```cpp
 261: }
 262: 
 263: } // end anonymous namespace
 264: 
 265: Tensor sparse_sparse_matmul_cpu(const Tensor& mat1_, const Tensor& mat2_) {
 266:   TORCH_INTERNAL_ASSERT(mat1_.is_sparse());
 267:   TORCH_INTERNAL_ASSERT(mat2_.is_sparse());
 268:   TORCH_CHECK(mat1_.dim() == 2);
 269:   TORCH_CHECK(mat2_.dim() == 2);
 270:   TORCH_CHECK(mat1_.dense_dim() == 0, "sparse_sparse_matmul_cpu: scalar values expected, got ", mat1_.dense_dim(), "D values");
 271:   TORCH_CHECK(mat2_.dense_dim() == 0, "sparse_sparse_matmul_cpu: scalar values expected, got ", mat2_.dense_dim(), "D values");
 272: 
 273:   TORCH_CHECK(
 274:       mat1_.size(1) == mat2_.size(0), "mat1 and mat2 shapes cannot be multiplied (",
 275:       mat1_.size(0), "x", mat1_.size(1), " and ", mat2_.size(0), "x", mat2_.size(1), ")");
 276: 
 277:   TORCH_CHECK(mat1_.scalar_type() == mat2_.scalar_type(),
 278:            "mat1 dtype ", mat1_.scalar_type(), " does not match mat2 dtype ", mat2_.scalar_type());
 279: 
 280:   auto output = at::native::empty_like(mat1_);
```
- L261: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L263: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L265: Defines function `sparse_sparse_matmul_cpu` and begins its implementation body. / 定义函数 `sparse_sparse_matmul_cpu`，并开始其实现体。
- L266: Checks an internal invariant that should always hold inside PyTorch. / 检查 PyTorch 内部应始终成立的不变量。
- L267: Checks an internal invariant that should always hold inside PyTorch. / 检查 PyTorch 内部应始终成立的不变量。
- L268: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L269: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L270: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L271: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L273: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L274: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L275: Declares function `size` as part of this file's callable surface. / 声明函数 `size`，作为本文件可调用接口的一部分。
- L277: Checks a runtime precondition and throws a descriptive error if it fails. / 检查运行时前置条件；若失败则抛出带说明的信息。
- L278: Declares function `scalar_type` as part of this file's callable surface. / 声明函数 `scalar_type`，作为本文件可调用接口的一部分。
- L280: Declares function `empty_like` as part of this file's callable surface. / 声明函数 `empty_like`，作为本文件可调用接口的一部分。

### Lines 281-290

```cpp
 281:   output.sparse_resize_and_clear_({mat1_.size(0), mat2_.size(1)}, mat1_.sparse_dim(), 0);
 282: 
 283:   AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES(mat1_.scalar_type(), "sparse_matmul", [&] {
 284:     sparse_matmul_kernel<scalar_t>(output, mat1_.coalesce(), mat2_.coalesce());
 285:   });
 286:   return output;
 287: }
 288: 
 289: 
 290: } // namespace at::native
```
- L281: Declares function `sparse_resize_and_clear_` as part of this file's callable surface. / 声明函数 `sparse_resize_and_clear_`，作为本文件可调用接口的一部分。
- L283: Defines function `AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES` and begins its implementation body. / 定义函数 `AT_DISPATCH_FLOATING_AND_COMPLEX_TYPES`，并开始其实现体。
- L284: Declares function `coalesce` as part of this file's callable surface. / 声明函数 `coalesce`，作为本文件可调用接口的一部分。
- L285: Contributes a supporting statement to the surrounding implementation. / 为周围实现补充一条支持性语句。
- L286: Returns a value from the current function to its caller. / 从当前函数返回一个值给调用者。
- L287: Closes the current block, type, or function body. / 结束当前代码块、类型定义或函数体。
- L290: Closes namespace `at::native` and returns to the outer scope. / 关闭命名空间 `at::native`，返回外层作用域。

## Key Concepts / 关键概念

- Sparse tensor math and layout utilities / 稀疏张量数学与布局工具
- Sparse layout semantics and NNZ traversal / 稀疏布局语义与 NNZ 遍历
- CSR compressed sparse representation / CSR 压缩稀疏表示
- COO index/value representation / COO 索引/数值表示
- Tensor shape, stride, and dtype flow / Tensor 形状、步长与数据类型流转
- Dispatch stubs and backend selection / 分发桩与后端选择

## Dependencies / 依赖关系

- `ATen/core/Tensor.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/Config.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/Dispatch.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/NamedTensorUtils.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/SparseTensorImpl.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/SparseTensorUtils.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/Resize.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/StridedRandomAccessor.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/native/CompositeRandomAccessor.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `c10/util/irange.h` — c10 runtime or utility dependency / c10 运行时或工具依赖
- `unordered_map` — standard or external dependency / 标准库或外部依赖
- `ATen/Functions.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/NativeFunctions.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/_sparse_sparse_matmul_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/empty.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- `ATen/ops/empty_like_native.h` — ATen operator/tensor dependency / ATen 算子或张量依赖
- Subsystem tie-in: sparse layouts (COO/CSR/CSC/BSR), index transforms, and NNZ-oriented computation. / 子系统关联：稀疏布局（COO/CSR/CSC/BSR）、索引变换以及面向 NNZ 的计算。
