# Utils.cpp — Code Analysis / 代码分析
## Source / 来源
- File: `aten/src/ATen/cpu/Utils.cpp`
- Repository: `pytorch/pytorch`
- Purpose (EN): This file implements CPU backend support, with primary focus on `cpuinfo_cache`, `XFEATURE_XTILECFG`, `XFEATURE_XTILEDATA`.
- 用途（中文）: 该文件实现CPU 后端支持，核心关注对象是 `cpuinfo_cache`, `XFEATURE_XTILECFG`, `XFEATURE_XTILEDATA`。

## Line-by-Line Analysis / 逐行分析

### Lines 1-11
```cpp
#include <ATen/cpu/Utils.h>
#if !defined(__s390x__ ) && !defined(__powerpc__)
#include <cpuinfo.h>
#endif
#if defined(__linux__)
#include <sys/syscall.h>
#include <unistd.h>
#endif

namespace at::cpu {

```
- EN: Focus symbols: `at::cpu`. This block sets up header dependencies and the initial compilation context. The file brings required declarations into scope before entering the main ATen logic.
- CN: 关注符号：`at::cpu`。该代码块设置头文件依赖与初始编译上下文。文件在进入主要 ATen 逻辑前先引入所需声明。

### Lines 12-21
```cpp
static constexpr const char* get_cpu_architecture() {
#if defined(__x86_64__) || defined(_M_X64)
  return "x86_64";
#elif defined(__aarch64__) || defined(_M_ARM64)
  return "arm64";
#elif defined(__powerpc64__) || defined(__PPC64__)
  return "ppc64";
#elif defined(__s390x__)
  return "s390x";
#elif defined(__riscv) && (__riscv_xlen == 64)
```
- EN: Focus symbols: `get_cpu_architecture`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`get_cpu_architecture`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 22-32
```cpp
  return "riscv64";
#else
  return "unknown";
#endif
}

std::unordered_map<std::string, c10::IValue> get_cpu_capabilities() {
  std::unordered_map<std::string, c10::IValue> capabilities;

  capabilities["architecture"] = std::string(get_cpu_architecture());

```
- EN: Focus symbols: `get_cpu_capabilities`, `string`, `get_cpu_architecture`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`get_cpu_capabilities`, `string`, `get_cpu_architecture`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 33-42
```cpp
#if !defined(__s390x__) && !defined(__powerpc__)
  if (!cpuinfo_initialize()) {
    return capabilities;
  }

  auto get_cache_size = [](int level) -> int64_t {
    const auto processors = cpuinfo_get_processors();
    if (!processors) {
      return 0;
    }
```
- EN: Focus symbols: `cpuinfo_initialize`, `cpuinfo_get_processors`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cpuinfo_initialize`, `cpuinfo_get_processors`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 43-56
```cpp
    const struct cpuinfo_cache* cache = nullptr;
    switch (level) {
      case 1:
        cache = processors[0].cache.l1d;
        break;
      case 2:
        cache = processors[0].cache.l2;
        break;
      default:
        TORCH_CHECK(false, "Unsupported cache level");
    }
    return cache ? static_cast<int64_t>(cache->size) : 0;
  };

```
- EN: Focus symbols: `cpuinfo_cache`, `TORCH_CHECK`. This block declares the main types, aliases, or compile-time descriptors used by this file. These declarations define the shape of the API and the data carried through later code.
- CN: 关注符号：`cpuinfo_cache`, `TORCH_CHECK`。该代码块声明此文件使用的核心类型、别名或编译期描述对象。这些声明决定了 API 的形状以及后续代码中流动的数据结构。

### Lines 57-68
```cpp
  const auto packages = cpuinfo_get_packages();
  if (packages && cpuinfo_get_packages_count() > 0) {
    capabilities["cpu_name"] = std::string(packages[0].name);
  }

  capabilities["num_sockets"] =
      static_cast<int64_t>(cpuinfo_get_packages_count());
  capabilities["num_physical_cores"] =
      static_cast<int64_t>(cpuinfo_get_cores_count());
  capabilities["num_logical_cores"] =
      static_cast<int64_t>(cpuinfo_get_processors_count());

```
- EN: Focus symbols: `cpuinfo_get_packages`, `cpuinfo_get_packages_count`, `string`, `cpuinfo_get_cores_count`, `cpuinfo_get_processors_count`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cpuinfo_get_packages`, `cpuinfo_get_packages_count`, `string`, `cpuinfo_get_cores_count`, `cpuinfo_get_processors_count`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 69-81
```cpp
  capabilities["l1d_cache_size"] = get_cache_size(1);
  capabilities["l2_cache_size"] = get_cache_size(2);

#if defined(__x86_64__) || defined(_M_X64)
  // SSE family
  capabilities["sse"] = cpuinfo_has_x86_sse();
  capabilities["sse2"] = cpuinfo_has_x86_sse2();
  capabilities["sse3"] = cpuinfo_has_x86_sse3();
  capabilities["ssse3"] = cpuinfo_has_x86_ssse3();
  capabilities["sse4_1"] = cpuinfo_has_x86_sse4_1();
  capabilities["sse4_2"] = cpuinfo_has_x86_sse4_2();
  capabilities["sse4a"] = cpuinfo_has_x86_sse4a();

```
- EN: Focus symbols: `get_cache_size`, `cpuinfo_has_x86_sse`, `cpuinfo_has_x86_sse2`, `cpuinfo_has_x86_sse3`, `cpuinfo_has_x86_ssse3`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`get_cache_size`, `cpuinfo_has_x86_sse`, `cpuinfo_has_x86_sse2`, `cpuinfo_has_x86_sse3`, `cpuinfo_has_x86_ssse3`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 82-91
```cpp
  // AVX family
  capabilities["avx"] = cpuinfo_has_x86_avx();
  capabilities["avx2"] = cpuinfo_has_x86_avx2();
  capabilities["avx_vnni"] = cpuinfo_has_x86_avxvnni();

  // AVX-512 family
  capabilities["avx512_f"] = cpuinfo_has_x86_avx512f();
  capabilities["avx512_cd"] = cpuinfo_has_x86_avx512cd();
  capabilities["avx512_dq"] = cpuinfo_has_x86_avx512dq();
  capabilities["avx512_bw"] = cpuinfo_has_x86_avx512bw();
```
- EN: Focus symbols: `cpuinfo_has_x86_avx`, `cpuinfo_has_x86_avx2`, `cpuinfo_has_x86_avxvnni`, `cpuinfo_has_x86_avx512f`, `cpuinfo_has_x86_avx512cd`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cpuinfo_has_x86_avx`, `cpuinfo_has_x86_avx2`, `cpuinfo_has_x86_avxvnni`, `cpuinfo_has_x86_avx512f`, `cpuinfo_has_x86_avx512cd`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 92-104
```cpp
  capabilities["avx512_vl"] = cpuinfo_has_x86_avx512vl();
  capabilities["avx512_ifma"] = cpuinfo_has_x86_avx512ifma();
  capabilities["avx512_vbmi"] = cpuinfo_has_x86_avx512vbmi();
  capabilities["avx512_vbmi2"] = cpuinfo_has_x86_avx512vbmi2();
  capabilities["avx512_bitalg"] = cpuinfo_has_x86_avx512bitalg();
  capabilities["avx512_vpopcntdq"] = cpuinfo_has_x86_avx512vpopcntdq();
  capabilities["avx512_vnni"] = cpuinfo_has_x86_avx512vnni();
  capabilities["avx512_bf16"] = cpuinfo_has_x86_avx512bf16();
  capabilities["avx512_fp16"] = cpuinfo_has_x86_avx512fp16();
  capabilities["avx512_vp2intersect"] = cpuinfo_has_x86_avx512vp2intersect();
  capabilities["avx512_4vnniw"] = cpuinfo_has_x86_avx512_4vnniw();
  capabilities["avx512_4fmaps"] = cpuinfo_has_x86_avx512_4fmaps();

```
- EN: Focus symbols: `cpuinfo_has_x86_avx512vl`, `cpuinfo_has_x86_avx512ifma`, `cpuinfo_has_x86_avx512vbmi`, `cpuinfo_has_x86_avx512vbmi2`, `cpuinfo_has_x86_avx512bitalg`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cpuinfo_has_x86_avx512vl`, `cpuinfo_has_x86_avx512ifma`, `cpuinfo_has_x86_avx512vbmi`, `cpuinfo_has_x86_avx512vbmi2`, `cpuinfo_has_x86_avx512bitalg`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 105-114
```cpp
  // AVX10 family
  capabilities["avx10_1"] = cpuinfo_has_x86_avx10_1();
  capabilities["avx10_2"] = cpuinfo_has_x86_avx10_2();

  // AVX-VNNI-INT variants
  capabilities["avx_vnni_int8"] = cpuinfo_has_x86_avx_vnni_int8();
  capabilities["avx_vnni_int16"] = cpuinfo_has_x86_avx_vnni_int16();
  capabilities["avx_ne_convert"] = cpuinfo_has_x86_avx_ne_convert();

  // AMX (Advanced Matrix Extensions)
```
- EN: Focus symbols: `cpuinfo_has_x86_avx10_1`, `cpuinfo_has_x86_avx10_2`, `cpuinfo_has_x86_avx_vnni_int8`, `cpuinfo_has_x86_avx_vnni_int16`, `cpuinfo_has_x86_avx_ne_convert`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cpuinfo_has_x86_avx10_1`, `cpuinfo_has_x86_avx10_2`, `cpuinfo_has_x86_avx_vnni_int8`, `cpuinfo_has_x86_avx_vnni_int16`, `cpuinfo_has_x86_avx_ne_convert`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 115-124
```cpp
  capabilities["amx_bf16"] = cpuinfo_has_x86_amx_bf16();
  capabilities["amx_tile"] = cpuinfo_has_x86_amx_tile();
  capabilities["amx_int8"] = cpuinfo_has_x86_amx_int8();
  capabilities["amx_fp16"] = cpuinfo_has_x86_amx_fp16();

  // FMA
  capabilities["fma3"] = cpuinfo_has_x86_fma3();
  capabilities["fma4"] = cpuinfo_has_x86_fma4();

  // Other useful capabilities
```
- EN: Focus symbols: `cpuinfo_has_x86_amx_bf16`, `cpuinfo_has_x86_amx_tile`, `cpuinfo_has_x86_amx_int8`, `cpuinfo_has_x86_amx_fp16`, `cpuinfo_has_x86_fma3`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cpuinfo_has_x86_amx_bf16`, `cpuinfo_has_x86_amx_tile`, `cpuinfo_has_x86_amx_int8`, `cpuinfo_has_x86_amx_fp16`, `cpuinfo_has_x86_fma3`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 125-136
```cpp
  capabilities["f16c"] = cpuinfo_has_x86_f16c();
  capabilities["bmi"] = cpuinfo_has_x86_bmi();
  capabilities["bmi2"] = cpuinfo_has_x86_bmi2();
  capabilities["popcnt"] = cpuinfo_has_x86_popcnt();
  capabilities["lzcnt"] = cpuinfo_has_x86_lzcnt();
  capabilities["aes"] = cpuinfo_has_x86_aes();
  capabilities["sha"] = cpuinfo_has_x86_sha();
  capabilities["clflush"] = cpuinfo_isa.clflush;
  capabilities["clflushopt"] = cpuinfo_isa.clflushopt;
  capabilities["clwb"] = cpuinfo_has_x86_clwb();
#endif

```
- EN: Focus symbols: `cpuinfo_has_x86_f16c`, `cpuinfo_has_x86_bmi`, `cpuinfo_has_x86_bmi2`, `cpuinfo_has_x86_popcnt`, `cpuinfo_has_x86_lzcnt`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cpuinfo_has_x86_f16c`, `cpuinfo_has_x86_bmi`, `cpuinfo_has_x86_bmi2`, `cpuinfo_has_x86_popcnt`, `cpuinfo_has_x86_lzcnt`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 137-146
```cpp
  // ARM64 specific capabilities
#if defined(__aarch64__) || defined(_M_ARM64)
  capabilities["neon"] = cpuinfo_has_arm_neon();
  capabilities["fp16_arith"] = cpuinfo_has_arm_fp16_arith();
  capabilities["bf16"] = cpuinfo_has_arm_bf16();
  capabilities["i8mm"] = cpuinfo_has_arm_i8mm();
  capabilities["dot"] = cpuinfo_has_arm_neon_dot();
  capabilities["sve"] = cpuinfo_has_arm_sve();
  capabilities["sve2"] = cpuinfo_has_arm_sve2();
  capabilities["sve_bf16"] = cpuinfo_has_arm_sve_bf16();
```
- EN: Focus symbols: `cpuinfo_has_arm_neon`, `cpuinfo_has_arm_fp16_arith`, `cpuinfo_has_arm_bf16`, `cpuinfo_has_arm_i8mm`, `cpuinfo_has_arm_neon_dot`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cpuinfo_has_arm_neon`, `cpuinfo_has_arm_fp16_arith`, `cpuinfo_has_arm_bf16`, `cpuinfo_has_arm_i8mm`, `cpuinfo_has_arm_neon_dot`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 147-156
```cpp
  capabilities["sme"] = cpuinfo_has_arm_sme();
  capabilities["sme2"] = cpuinfo_has_arm_sme2();
  capabilities["atomics"] = cpuinfo_has_arm_atomics();
  capabilities["fhm"] = cpuinfo_has_arm_fhm();
  capabilities["rdm"] = cpuinfo_has_arm_neon_rdm();
  capabilities["crc32"] = cpuinfo_has_arm_crc32();
  capabilities["aes"] = cpuinfo_has_arm_aes();
  capabilities["sha1"] = cpuinfo_has_arm_sha1();
  capabilities["sha2"] = cpuinfo_has_arm_sha2();
  capabilities["pmull"] = cpuinfo_has_arm_pmull();
```
- EN: Focus symbols: `cpuinfo_has_arm_sme`, `cpuinfo_has_arm_sme2`, `cpuinfo_has_arm_atomics`, `cpuinfo_has_arm_fhm`, `cpuinfo_has_arm_neon_rdm`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cpuinfo_has_arm_sme`, `cpuinfo_has_arm_sme2`, `cpuinfo_has_arm_atomics`, `cpuinfo_has_arm_fhm`, `cpuinfo_has_arm_neon_rdm`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 157-168
```cpp
  if (cpuinfo_has_arm_sve()) {
    capabilities["sve_max_length"] =
        static_cast<int64_t>(cpuinfo_get_max_arm_sve_length());
  }
  if (cpuinfo_has_arm_sme()) {
    capabilities["sme_max_length"] =
        static_cast<int64_t>(cpuinfo_get_max_arm_sme_length());
  }
#endif

#endif // !defined(__s390x__) && !defined(__powerpc__)

```
- EN: Focus symbols: `cpuinfo_has_arm_sve`, `cpuinfo_get_max_arm_sve_length`, `cpuinfo_has_arm_sme`, `cpuinfo_get_max_arm_sme_length`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`cpuinfo_has_arm_sve`, `cpuinfo_get_max_arm_sve_length`, `cpuinfo_has_arm_sme`, `cpuinfo_get_max_arm_sme_length`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 169-179
```cpp
  return capabilities;
}

bool is_avx512_vnni_supported() {
#if !defined(__s390x__) && !defined(__powerpc__)
  return cpuinfo_initialize() && cpuinfo_has_x86_avx512vnni();
#else
  return false;
#endif
}

```
- EN: Focus symbols: `is_avx512_vnni_supported`, `cpuinfo_initialize`, `cpuinfo_has_x86_avx512vnni`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`is_avx512_vnni_supported`, `cpuinfo_initialize`, `cpuinfo_has_x86_avx512vnni`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 180-192
```cpp
static bool is_amx_tile_supported() {
#if !defined(__s390x__) && !defined(__powerpc__)
  return cpuinfo_initialize() && cpuinfo_has_x86_amx_tile();
#else
  return false;
#endif
}

bool init_amx() {
  if (!is_amx_tile_supported()) {
    return false;
  }

```
- EN: Focus symbols: `is_amx_tile_supported`, `cpuinfo_initialize`, `cpuinfo_has_x86_amx_tile`, `init_amx`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`is_amx_tile_supported`, `cpuinfo_initialize`, `cpuinfo_has_x86_amx_tile`, `init_amx`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 193-202
```cpp
#if defined(__linux__) && !defined(__ANDROID__) && defined(__x86_64__)
#define XFEATURE_XTILECFG 17
#define XFEATURE_XTILEDATA 18
#define XFEATURE_MASK_XTILECFG (1 << XFEATURE_XTILECFG)
#define XFEATURE_MASK_XTILEDATA (1 << XFEATURE_XTILEDATA)
#define XFEATURE_MASK_XTILE (XFEATURE_MASK_XTILECFG | XFEATURE_MASK_XTILEDATA)

#define ARCH_GET_XCOMP_PERM 0x1022
#define ARCH_REQ_XCOMP_PERM 0x1023

```
- EN: Focus symbols: `XFEATURE_XTILECFG`, `XFEATURE_XTILEDATA`, `XFEATURE_MASK_XTILECFG`, `XFEATURE_MASK_XTILEDATA`, `XFEATURE_MASK_XTILE`. This block defines preprocessor helpers for repetitive code generation. The macros compress boilerplate and make repeated specializations easier to maintain.
- CN: 关注符号：`XFEATURE_XTILECFG`, `XFEATURE_XTILEDATA`, `XFEATURE_MASK_XTILECFG`, `XFEATURE_MASK_XTILEDATA`, `XFEATURE_MASK_XTILE`。该代码块定义用于重复代码生成的预处理器辅助宏。这些宏压缩样板代码，使重复特化更易维护。

### Lines 203-212
```cpp
  unsigned long bitmask = 0;
  // Request permission to use AMX instructions
  long rc = syscall(SYS_arch_prctl, ARCH_REQ_XCOMP_PERM, XFEATURE_XTILEDATA);
  if (rc) {
      return false;
  }
  // Check if the system supports AMX instructions
  rc = syscall(SYS_arch_prctl, ARCH_GET_XCOMP_PERM, &bitmask);
  if (rc) {
      return false;
```
- EN: Focus symbols: `syscall`. This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 关注符号：`syscall`。该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 213-222
```cpp
  }
  if (bitmask & XFEATURE_MASK_XTILE) {
      return true;
  }
  return false;
#else
  return true;
#endif
}

```
- EN: This block declares or implements callable behavior for this module. Inputs are translated into lower-level operations, state updates, or returned values here.
- CN: 该代码块声明或实现该模块的可调用行为。这里把输入转换为更底层操作、状态更新或返回值。

### Lines 223-223
```cpp
} // namespace at::cpu
```
- EN: Focus symbols: `at::cpu`. This block opens or closes namespaces to keep symbols in the intended API scope. This keeps the implementation aligned with the broader ATen/C10 module hierarchy.
- CN: 关注符号：`at::cpu`。该代码块打开或关闭命名空间以维持符号的预期作用域。这使实现与更大的 ATen/C10 模块层次保持一致。

## Key Concepts / 关键概念
- CPU backend support / CPU 后端支持
- Conditional compilation / 条件编译
- Type-system design / 类型系统设计
- Runtime validation / 运行时校验
- ATen/C10 internal layering / ATen/C10 内部分层

## Dependencies / 依赖关系
- Direct internal includes / 直接内部依赖: `ATen/cpu/Utils.h`
- External/system includes / 外部或系统头: `cpuinfo.h`, `sys/syscall.h`, `unistd.h`
- Companion translation units / 配套实现或声明文件: `aten/src/ATen/cpu/Utils.h`
- Inferred semantic dependencies / 推断出的语义依赖: runtime validation / 运行时校验; namespace scoping / 命名空间作用域
