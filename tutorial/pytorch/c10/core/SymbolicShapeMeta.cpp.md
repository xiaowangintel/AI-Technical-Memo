# SymbolicShapeMeta.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `c10/core/SymbolicShapeMeta.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Defines c10 core abstractions such as tensor metadata, dispatch keys, devices, storage, and symbolic values.
- **Purpose (CN)**: 定义 c10 的核心抽象，如张量元数据、分发键、设备、存储以及符号值。

## Line-by-Line Analysis / 逐行分析
### Lines 1-18
```cpp
#include <c10/core/Contiguity.h>
#include <c10/core/MemoryFormat.h>
#include <c10/core/SymInt.h>
#include <c10/core/SymIntArrayRef.h>
#include <c10/core/SymbolicShapeMeta.h>

namespace c10 {

SymbolicShapeMeta::SymbolicShapeMeta(const SymbolicShapeMeta& other)
    // Non-mutables can be accessed outside the mutex
    : sizes_(other.sizes_),
      strides_(other.strides_),
      storage_offset_(other.storage_offset_),
      strides_valid_(other.strides_valid_) {
  std::scoped_lock lock(other.mutables_);
  // These must be copied under lock, so ignore clang-tidy here!
  // NOLINTBEGIN(cppcoreguidelines-prefer-member-initializer)
  numel_ = other.numel_;
```
- **EN**: This block assembles the compilation dependencies, pulling in local PyTorch/c10 headers such as c10/core/Contiguity.h, c10/core/MemoryFormat.h, c10/core/SymInt.h, and 2 more. The namespace declarations place the code inside c10, matching the surrounding subsystem. This chunk defines `lock`, which implements a focused piece of c10 core logic. Synchronization primitives protect shared state and make concurrent updates deterministic enough for runtime use. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently.
- **CN**: 这一段组织编译依赖，引入了本地 PyTorch/c10 头文件，如 c10/core/Contiguity.h、c10/core/MemoryFormat.h、c10/core/SymInt.h 等共 5 项。 命名空间声明把代码放入 c10 中，与周边子系统保持一致。 这一段定义了 `lock`，其作用是实现一段聚焦的 c10 核心逻辑。 同步原语用于保护共享状态，并让并发更新在运行时场景下保持足够确定。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。

### Lines 19-36
```cpp
  is_contiguous_ = other.is_contiguous_;
  is_channels_last_contiguous_ = other.is_channels_last_contiguous_;
  is_channels_last_3d_contiguous_ = other.is_channels_last_3d_contiguous_;
  is_channels_last_ = other.is_channels_last_;
  is_channels_last_3d_ = other.is_channels_last_3d_;
  is_non_overlapping_and_dense_ = other.is_non_overlapping_and_dense_;
  available_.store(other.available_.load());
  // NOLINTEND(cppcoreguidelines-prefer-member-initializer)
}

// base, sizes, strides
static std::optional<
    std::tuple<SymNode, std::vector<SymNode>, std::vector<SymNode>>>
normalize_sym_sizes_strides(SymIntArrayRef sizes, SymIntArrayRef strides) {
  // Look for a SymNode to dispatch on
  SymNode base;
  bool all_hinted = true;
  // NB: sizes/strides guaranteed to be positive, so only need
```
- **EN**: This chunk defines `normalize_sym_sizes_strides`, which implements a focused piece of c10 core logic. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically.
- **CN**: 这一段定义了 `normalize_sym_sizes_strides`，其作用是实现一段聚焦的 c10 核心逻辑。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。

### Lines 37-53
```cpp
  // is_heap_allocated
  for (const auto& s : sizes) {
    if (all_hinted && !s.has_hint()) {
      all_hinted = false;
    }
    if (!base && s.is_heap_allocated()) {
      base = s.toSymNode();
    }
  }
  for (const auto& s : strides) {
    if (all_hinted && !s.has_hint()) {
      all_hinted = false;
    }
    if (!base && s.is_heap_allocated()) {
      base = s.toSymNode();
    }
  }
```
- **EN**: This chunk defines `toSymNode`, which converts one representation into another form used by nearby runtime code. Memory-management logic tracks ownership, requested sizes, reuse opportunities, or allocator configuration. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants.
- **CN**: 这一段定义了 `toSymNode`，其作用是把一种表示转换为附近运行时代码使用的另一种形式。 内存管理逻辑会跟踪所有权、申请尺寸、复用机会或分配器配置。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。

### Lines 54-70
```cpp
  if (!base || all_hinted) {
    // Couldn't find.  Tell the caller to do the normal computation
    // Alternately, if everything is hinted, we want the normal computation
    // too
    return std::nullopt;
  }
  // Populate the SymNode array
  std::vector<SymNode> size_nodes;
  std::vector<SymNode> stride_nodes;
  size_nodes.reserve(sizes.size());
  stride_nodes.reserve(strides.size());
  for (const auto& s : sizes) {
    size_nodes.emplace_back(s.wrap_node(base));
  }
  for (const auto& s : strides) {
    stride_nodes.emplace_back(s.wrap_node(base));
  }
```
- **EN**: This chunk defines `emplace_back`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `emplace_back`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 71-83
```cpp
  return std::tuple<SymNode, std::vector<SymNode>, std::vector<SymNode>>(
      std::move(base), std::move(size_nodes), std::move(stride_nodes));
}
namespace {
bool all_hinted(
    const c10::SymIntArrayRef& sizes,
    const c10::SymIntArrayRef& strides) {
  auto all_hinted = true;
  for (const auto& s : sizes) {
    if (!s.has_hint()) {
      return false;
    }
  }
```
- **EN**: This chunk defines `all_hinted`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `all_hinted`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 85-100
```cpp
  if (all_hinted) {
    for (const auto& s : strides) {
      if (!s.has_hint()) {
        return false;
      }
    }
  }
  return all_hinted;
}
} // namespace

// Special treatment because of numel
SymBool SymbolicShapeMeta::compute_contiguous() const {
  if (!strides_valid_) {
    return false;
  }
```
- **EN**: This chunk defines `compute_contiguous`, which implements a focused piece of c10 core logic. Looping logic walks collections, device tables, or metadata arrays so the same rule is applied systematically. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `compute_contiguous`，其作用是实现一段聚焦的 c10 核心逻辑。 循环逻辑会遍历集合、设备表或元数据数组，从而把同一规则系统地应用到每个元素。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 101-117
```cpp
  c10::SymIntArrayRef sizes(sizes_);
  c10::SymIntArrayRef strides(strides_);

  auto result = _compute_contiguous_sym(sizes, strides, numel());

  // If the result is already determined without guarding, just return it.
  auto maybe_as_bool = result.maybe_as_bool();
  if (maybe_as_bool.has_value()) {
    return maybe_as_bool.value();
  }

  if (all_hinted(sizes, strides)) {
    // We avoid going through the slow path if everything is hinted,
    // because evaluating a large SymPy expression can be expensive.
    // TODO exclude backed_size_oblivious from this path.
    return _compute_contiguous<SymInt>(sizes_, strides_, numel());
  }
```
- **EN**: This chunk defines `_compute_contiguous<SymInt>`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `_compute_contiguous<SymInt>`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 119-135
```cpp
  return result;
}

SymBool SymbolicShapeMeta::compute_channels_last_contiguous_2d() const {
  if (!strides_valid_) {
    return false;
  }
  c10::SymIntArrayRef sizes(sizes_);
  c10::SymIntArrayRef strides(strides_);

  auto result = _compute_channels_last_contiguous_2d_sym(sizes, strides);

  // If the result is already determined without guarding, just return it.
  auto maybe_as_bool = result.maybe_as_bool();
  if (maybe_as_bool.has_value()) {
    return maybe_as_bool.value();
  }
```
- **EN**: This chunk defines `value`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `value`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 137-152
```cpp
  if (all_hinted(sizes, strides)) {
    // We avoid going through the slow path if everything is hinted,
    // because evaluating a large SymPy expression can be expensive.
    // TODO exclude backed_size_oblivious from this path.
    return _compute_channels_last_contiguous_2d<SymInt>(sizes_, strides_);
  }

  return result;
}

SymBool SymbolicShapeMeta::compute_channels_last_contiguous_3d() const {
  if (!strides_valid_) {
    return false;
  }
  c10::SymIntArrayRef sizes(sizes_);
  c10::SymIntArrayRef strides(strides_);
```
- **EN**: This chunk defines `strides`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `strides`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 154-170
```cpp
  auto result = _compute_channels_last_contiguous_3d_sym(sizes, strides);

  // If the result is already determined without guarding, just return it.
  auto maybe_as_bool = result.maybe_as_bool();
  if (maybe_as_bool.has_value()) {
    return maybe_as_bool.value();
  }

  if (all_hinted(sizes, strides)) {
    // We avoid going through the slow path if everything is hinted,
    // because evaluating a large SymPy expression can be expensive.
    // TODO exclude backed_size_oblivious from this path.
    return _compute_channels_last_contiguous_3d<SymInt>(sizes_, strides_);
  }

  return result;
}
```
- **EN**: This chunk defines `_compute_channels_last_contiguous_3d<SymInt>`, which implements a focused piece of c10 core logic. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `_compute_channels_last_contiguous_3d<SymInt>`，其作用是实现一段聚焦的 c10 核心逻辑。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 172-189
```cpp
// The rest of them
#define DEFINE_EAGER_SYMBOOL_COMPUTE(name, fallback) \
  SymBool SymbolicShapeMeta::name() const {          \
    if (!strides_valid_) {                           \
      return false;                                  \
    }                                                \
    c10::SymIntArrayRef sizes(sizes_);               \
    c10::SymIntArrayRef strides(strides_);           \
    return fallback(sizes, strides);                 \
  }

#define DEFINE_SYMBOOL_COMPUTE(name, nodeimpl, fallback)        \
  SymBool SymbolicShapeMeta::name() const {                     \
    if (!strides_valid_) {                                      \
      return false;                                             \
    }                                                           \
    auto n = normalize_sym_sizes_strides(sizes_, strides_);     \
    if (n.has_value()) {                                        \
```
- **EN**: The preprocessor guard keeps declarations single-instanced when this header is included transitively. This chunk defines `normalize_sym_sizes_strides`, which implements a focused piece of c10 core logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 预处理器保护用于避免头文件在传递包含时被重复展开。 这一段定义了 `normalize_sym_sizes_strides`，其作用是实现一段聚焦的 c10 核心逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 190-205
```cpp
      auto [base, size_nodes, stride_nodes] = *n;               \
      return SymBool(base->nodeimpl(size_nodes, stride_nodes)); \
    } else {                                                    \
      c10::SymIntArrayRef sizes(sizes_);                        \
      c10::SymIntArrayRef strides(strides_);                    \
      return fallback(sizes, strides);                          \
    }                                                           \
  }

// clang-format off
DEFINE_EAGER_SYMBOOL_COMPUTE(compute_strides_like_channels_last_2d, is_channels_last_strides_2d)
DEFINE_EAGER_SYMBOOL_COMPUTE(compute_strides_like_channels_last_3d, is_channels_last_strides_3d)

DEFINE_SYMBOOL_COMPUTE(compute_non_overlapping_and_dense, is_non_overlapping_and_dense, _compute_non_overlapping_and_dense)

// clang-format on
```
- **EN**: This chunk defines `fallback`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `fallback`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 207-224
```cpp
#undef DEFINE_SYMBOOL_COMPUTE

// Glue compute
// NB: this logic very intentionally short circuits if possible.  Without
// short circuiting, it causes
// python test/functorch/test_aotdispatch.py -k
// test_aot_autograd_symbolic_exhaustive_nn_functional_unfold_cpu_float32 to run
// very slowly.

SymBool SymbolicShapeMeta::compute_is_non_overlapping_and_dense_dim4() const {
  init_is_contiguous();
  if (guard_or_false(is_contiguous(), __FILE__, __LINE__)) {
    return true;
  }
  init_is_channels_last_contiguous();
  if (guard_or_false(is_channels_last_contiguous(), __FILE__, __LINE__)) {
    return true;
  }
```
- **EN**: This chunk defines `init_is_channels_last_contiguous`, which implements a focused piece of c10 core logic. Preprocessor and exported-macro usage adapts the code to compiler, platform, and visibility constraints. Dispatch-oriented logic computes or queries backend/functionality state so later calls reach the correct kernel path. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `init_is_channels_last_contiguous`，其作用是实现一段聚焦的 c10 核心逻辑。 预处理器与导出宏的使用让代码适配编译器、平台以及可见性约束。 面向分发的逻辑会计算或查询后端/功能状态，从而让后续调用进入正确的内核路径。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 225-241
```cpp
  return is_contiguous() | is_channels_last_contiguous() |
      compute_non_overlapping_and_dense();
}

SymBool SymbolicShapeMeta::compute_channels_last_contiguous_3d_dim5() const {
  init_is_channels_last_contiguous();
  if (guard_or_false(is_channels_last_contiguous(), __FILE__, __LINE__)) {
    return false;
  }
  return ~is_channels_last_contiguous() & compute_channels_last_contiguous_3d();
}

SymBool SymbolicShapeMeta::compute_channels_last_2d_dim5() const {
  init_is_channels_last_3d_contiguous();
  if (guard_or_false(is_channels_last_3d_contiguous(), __FILE__, __LINE__)) {
    return false;
  }
```
- **EN**: This chunk defines `init_is_channels_last_3d_contiguous`, which implements a focused piece of c10 core logic. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `init_is_channels_last_3d_contiguous`，其作用是实现一段聚焦的 c10 核心逻辑。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 242-259
```cpp
  return ~is_channels_last_3d_contiguous() &
      compute_strides_like_channels_last_2d();
}

SymBool SymbolicShapeMeta::compute_channels_last_3d_dim5() const {
  if (guard_or_false(is_channels_last(), __FILE__, __LINE__)) {
    return false;
  }
  return ~is_channels_last() & compute_strides_like_channels_last_3d();
}

SymBool SymbolicShapeMeta::compute_is_non_overlapping_and_dense_dim5() const {
  if (guard_or_false(is_contiguous(), __FILE__, __LINE__)) {
    return true;
  }
  if (guard_or_false(is_channels_last_contiguous(), __FILE__, __LINE__)) {
    return true;
  }
```
- **EN**: This chunk defines `compute_is_non_overlapping_and_dense_dim5`, which implements a focused piece of c10 core logic. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `compute_is_non_overlapping_and_dense_dim5`，其作用是实现一段聚焦的 c10 核心逻辑。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 260-272
```cpp
  if (guard_or_false(is_channels_last_3d_contiguous(), __FILE__, __LINE__)) {
    return true;
  }
  return is_contiguous() | is_channels_last_contiguous() |
      is_channels_last_3d_contiguous() | compute_non_overlapping_and_dense();
}

SymBool SymbolicShapeMeta::compute_is_non_overlapping_and_dense_anydim() const {
  if (guard_or_false(is_contiguous(), __FILE__, __LINE__)) {
    return true;
  }
  return is_contiguous() | compute_non_overlapping_and_dense();
}
```
- **EN**: This chunk defines `compute_is_non_overlapping_and_dense_anydim`, which implements a focused piece of c10 core logic. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `compute_is_non_overlapping_and_dense_anydim`，其作用是实现一段聚焦的 c10 核心逻辑。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 274-290
```cpp
void SymbolicShapeMeta::set_numel(SymInt val) const {
  std::scoped_lock lock(mutables_);
  if (has_numel()) {
    return;
  }
  numel_ = std::move(val);
  available_.fetch_or(numel_avail);
}

void SymbolicShapeMeta::set_is_contiguous(SymBool val) const {
  std::scoped_lock lock(mutables_);
  if (has_is_contiguous()) {
    return;
  }
  is_contiguous_ = std::move(val);
  available_.fetch_or(is_contiguous_avail);
}
```
- **EN**: This chunk defines `set_is_contiguous`, which maintains lookup structures and hashing behavior for fast metadata access. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `set_is_contiguous`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 292-308
```cpp
void SymbolicShapeMeta::set_is_channels_last_contiguous(SymBool val) const {
  std::scoped_lock lock(mutables_);
  if (has_is_channels_last_contiguous()) {
    return;
  }
  is_channels_last_contiguous_ = std::move(val);
  available_.fetch_or(is_channels_last_contiguous_avail);
}

void SymbolicShapeMeta::set_is_channels_last_3d_contiguous(SymBool val) const {
  std::scoped_lock lock(mutables_);
  if (has_is_channels_last_3d_contiguous()) {
    return;
  }
  is_channels_last_3d_contiguous_ = std::move(val);
  available_.fetch_or(is_channels_last_3d_contiguous_avail);
}
```
- **EN**: This chunk defines `set_is_channels_last_3d_contiguous`, which maintains lookup structures and hashing behavior for fast metadata access. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `set_is_channels_last_3d_contiguous`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 310-326
```cpp
void SymbolicShapeMeta::set_is_channels_last(SymBool val) const {
  std::scoped_lock lock(mutables_);
  if (has_is_channels_last()) {
    return;
  }
  is_channels_last_ = std::move(val);
  available_.fetch_or(is_channels_last_avail);
}

void SymbolicShapeMeta::set_is_channels_last_3d(SymBool val) const {
  std::scoped_lock lock(mutables_);
  if (has_is_channels_last_3d()) {
    return;
  }
  is_channels_last_3d_ = std::move(val);
  available_.fetch_or(is_channels_last_3d_avail);
}
```
- **EN**: This chunk defines `set_is_channels_last_3d`, which maintains lookup structures and hashing behavior for fast metadata access. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `set_is_channels_last_3d`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 328-343
```cpp
void SymbolicShapeMeta::set_is_non_overlapping_and_dense(SymBool val) const {
  std::scoped_lock lock(mutables_);
  if (has_is_non_overlapping_and_dense()) {
    return;
  }
  is_non_overlapping_and_dense_ = std::move(val);
  available_.fetch_or(is_non_overlapping_and_dense_avail);
}

void SymbolicShapeMeta::init_numel() const {
  set_numel(multiply_integers(sizes_));
}

void SymbolicShapeMeta::init_is_contiguous() const {
  set_is_contiguous(compute_contiguous());
}
```
- **EN**: This chunk defines `set_is_contiguous`, which maintains lookup structures and hashing behavior for fast metadata access. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Conditional branches split fast paths, error cases, and special-case invariants. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `set_is_contiguous`，其作用是维护查找结构与哈希行为，以便快速访问元数据。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 条件分支用于区分快速路径、错误场景以及特殊情况不变量。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 345-356
```cpp
void SymbolicShapeMeta::init_is_channels_last_contiguous() const {
  set_is_channels_last_contiguous([&] {
    switch (dim()) {
      case 5:
      case 4: {
        return compute_channels_last_contiguous_2d();
      }
      default:
        return SymBool{false};
    }
  }());
}
```
- **EN**: This chunk defines `compute_channels_last_contiguous_2d`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `compute_channels_last_contiguous_2d`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 358-375
```cpp
void SymbolicShapeMeta::init_is_channels_last_3d_contiguous() const {
  set_is_channels_last_3d_contiguous([&] {
    switch (dim()) {
      case 5:
        return compute_channels_last_contiguous_3d_dim5();
      default:
        return SymBool{false};
    }
  }());
}

void SymbolicShapeMeta::init_is_channels_last() const {
  set_is_channels_last([&] {
    switch (dim()) {
      case 5:
        return compute_channels_last_2d_dim5();
      case 4:
        return compute_strides_like_channels_last_2d();
```
- **EN**: This chunk defines `compute_strides_like_channels_last_2d`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `compute_strides_like_channels_last_2d`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 376-391
```cpp
      default:
        return SymBool{false};
    }
  }());
}

void SymbolicShapeMeta::init_is_channels_last_3d() const {
  set_is_channels_last_3d([&] {
    switch (dim()) {
      case 5:
        return compute_channels_last_3d_dim5();
      default:
        return SymBool{false};
    }
  }());
}
```
- **EN**: This chunk defines `compute_channels_last_3d_dim5`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `compute_channels_last_3d_dim5`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。

### Lines 393-406
```cpp
void SymbolicShapeMeta::init_is_non_overlapping_and_dense() const {
  set_is_non_overlapping_and_dense([&] {
    switch (dim()) {
      case 5:
        return compute_is_non_overlapping_and_dense_dim5();
      case 4:
        return compute_is_non_overlapping_and_dense_dim4();
      default:
        return compute_is_non_overlapping_and_dense_anydim();
    }
  }());
}

} // namespace c10
```
- **EN**: This chunk defines `compute_is_non_overlapping_and_dense_anydim`, which implements a focused piece of c10 core logic. Container logic organizes metadata, caches, or lookup state so surrounding code can access it efficiently. Switch-based dispatch selects specialized behavior for enum variants, backend kinds, or opcode-like cases. The tail returns accumulated results or hands the updated state back to the caller.
- **CN**: 这一段定义了 `compute_is_non_overlapping_and_dense_anydim`，其作用是实现一段聚焦的 c10 核心逻辑。 容器逻辑用于组织元数据、缓存或查找状态，从而让周边代码能够高效访问。 基于 switch 的分发会为枚举分支、后端类型或类似 opcode 的场景选择专门行为。 末尾会返回累积结果，或把更新后的状态交还给调用方。


## Key Concepts / 关键概念
- **c10 core abstractions**
  - EN: Defines c10 core abstractions such as tensor metadata, dispatch keys, devices, storage, and symbolic values.
  - CN: 定义 c10 的核心抽象，如张量元数据、分发键、设备、存储以及符号值。
- **SymbolicShapeMeta**
  - EN: `SymbolicShapeMeta` is one of the dominant symbols declared or implemented in this file.
  - CN: `SymbolicShapeMeta` 是本文件声明或实现的关键符号之一。
- **lock**
  - EN: `lock` is one of the dominant symbols declared or implemented in this file.
  - CN: `lock` 是本文件声明或实现的关键符号之一。
- **Storage ownership**
  - EN: Separates raw memory ownership from higher-level tensor metadata.
  - CN: 把原始内存所有权与更高层的张量元数据解耦。

## Dependencies / 依赖关系
- **Internal includes / 内部依赖**: `c10/core/Contiguity.h`、`c10/core/MemoryFormat.h`、`c10/core/SymInt.h`、`c10/core/SymIntArrayRef.h`、`c10/core/SymbolicShapeMeta.h`
- **Third-party includes / 第三方依赖**: 无
- **Standard includes / 标准库依赖**: 无
- **System includes / 系统依赖**: 无
- **Namespaces / 命名空间**: `c10`
- **Representative symbols / 代表性符号**: `SymbolicShapeMeta`、`lock`、`store`、`normalize_sym_sizes_strides`、`toSymNode`、`reserve`、`emplace_back`、`vector<SymNode>>`、`all_hinted`、`compute_contiguous`
