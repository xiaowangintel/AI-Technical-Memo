# fused_kernel.cpp — Code Analysis / 代码分析

## Source / 来源

- **File / 文件**: `torch/csrc/jit/codegen/fuser/cpu/fused_kernel.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Contains code-generation logic or generated declarations that synthesize JIT-related bindings, schemas, or helper code. This specific file centers on `fused_kernel.cpp`. Generation of declarations, registrations, or helper code is part of the responsibility.
- **Purpose (CN)**: 包含代码生成逻辑或生成后的声明，用于产出 JIT 相关绑定、模式或辅助代码。 该文件具体围绕 `fused_kernel.cpp` 展开。 其职责包含声明、注册或辅助代码的生成。

## Line-by-Line Analysis / 逐行分析

### Lines 1-28 / 第 1-28 行

```cpp
#include <torch/csrc/jit/codegen/fuser/cpu/fused_kernel.h>

#include <ATen/DynamicLibrary.h>
#include <ATen/code_template.h>
#include <c10/util/Exception.h>
#include <c10/util/env.h>
#include <torch/csrc/jit/codegen/fuser/compiler.h>
#include <torch/csrc/jit/codegen/fuser/cpu/temp_file.h>
#include <optional>

#include <cstdlib>
#include <iostream>
#include <string>

namespace torch::jit::fuser::cpu {

#ifdef _MSC_VER
static const std::string getTempPath() {
  wchar_t lpTempPathBuffer[MAX_PATH];

  DWORD dwRetVal = GetTempPathW(
      MAX_PATH, // length of the buffer
      lpTempPathBuffer); // buffer for path

  TORCH_CHECK(dwRetVal < MAX_PATH && dwRetVal != 0, "GetTempPath failed.");

  return std::string(c10::u16u8(lpTempPathBuffer));
}
```

- **EN:** This range collects the headers or imports needed by the following declarations and implementation blocks.
- **CN:** 这一段集中引入后续声明与实现所需的头文件或模块。
- **EN:** It enters or references namespace scopes such as torch::jit::fuser::cpu, which positions the code inside the JIT subsystem hierarchy.
- **CN:** 这里进入或引用了 torch::jit::fuser::cpu 等命名空间作用域，用于把代码放入 JIT 子系统层级中。
- **EN:** Important callable entry points in this range include getTempPath, TORCH_CHECK, string.
- **CN:** 这一段的重要可调用入口包括 getTempPath, TORCH_CHECK, string。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 29-56 / 第 29-56 行

```cpp
static const std::string temp_dir = getTempPath();
static const std::string so_template = temp_dir + "pytorch_fuserXXXXXX.dll";
static const std::string cpp_template = temp_dir + "pytorch_fuserXXXXXX.cpp";
static const std::string check_exists_string = "where ${program} > nul 2> nul";
static std::vector<std::wstring> env_list;
constexpr int so_suffix_len = 4;
constexpr int cpp_suffix_len = 4;
#else
static const std::string so_template = "/tmp/pytorch_fuserXXXXXX.so";
static const std::string cpp_template = "/tmp/pytorch_fuserXXXXXX.cpp";
static const std::string check_exists_string = "which ${program} > /dev/null";
constexpr int so_suffix_len = 3;
constexpr int cpp_suffix_len = 4;
#endif

#ifdef _MSC_VER
static std::optional<std::wstring> exec(const std::wstring& cmd) {
  std::array<wchar_t, 128> buffer;
  std::wstring result;
  std::unique_ptr<FILE, decltype(&_pclose)> pipe(
      _wpopen(cmd.c_str(), L"r"), _pclose);
  if (!pipe) {
    return std::nullopt;
  }
  while (fgetws(buffer.data(), static_cast<int>(buffer.size()), pipe.get()) !=
         nullptr) {
    result += buffer.data();
  }
```

- **EN:** Important callable entry points in this range include exec, decltype.
- **CN:** 这一段的重要可调用入口包括 exec, decltype。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 57-84 / 第 57-84 行

```cpp
  return result;
}

inline std::wstring& rtrim(std::wstring& s, const wchar_t* t = L" \t\n\r\f\v") {
  s.erase(s.find_last_not_of(t) + 1);
  return s;
}

static void activate() {
  wchar_t* root = nullptr;
  std::wstring cmd;
  std::optional<std::wstring> exec_out;
  std::wstring path;
  std::wstring vcruntime_plat;
  std::wstring envvars;

  // Checking whether the environment is already activated
  if (_wgetenv(L"VSCMD_ARG_TGT_ARCH")) {
    return;
  }

  // Getting `ProgramFiles` through environment variable queries
  root = _wgetenv(L"ProgramFiles(x86)");
  if (!root) {
    root = _wgetenv(L"ProgramFiles");
  }
  if (!root) {
    return;
```

- **EN:** Important callable entry points in this range include rtrim, activate.
- **CN:** 这一段的重要可调用入口包括 rtrim, activate。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 85-112 / 第 85-112 行

```cpp
  }

  // Getting VS 2017 installation path through `vswhere`
  cmd = L"\"" + std::wstring(root) +
      L"\\Microsoft Visual Studio\\Installer\\vswhere.exe\""
      L" -latest -prerelease -requires Microsoft.VisualStudio.Component.VC.Tools.x86.x64 -property installationPath";
  exec_out = exec(cmd);
  if (!exec_out) {
    return;
  }
  path = *exec_out;
  rtrim(path);

  // Checking whether the activation script `vcvarsall.bat` exists
  path += L"\\VC\\Auxiliary\\Build";
  struct _stati64 st;
  if (_wstati64(path.c_str(), &st) == -1 || !(st.st_mode & _S_IFDIR)) {
    return;
  }
  path += L"\\vcvarsall.bat";
  if (_waccess(path.c_str(), 0) == -1) {
    return;
  }

  // Determining current platform
  if (sizeof(void*) == 8) {
    vcruntime_plat = L"x64";
  } else {
```

- **EN:** The block declares or refines core types including _stati64.
- **CN:** 该代码块声明或细化了 _stati64 等核心类型。
- **EN:** Important callable entry points in this range include rtrim.
- **CN:** 这一段的重要可调用入口包括 rtrim。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Operator schema / 算子模式, Type definition / 类型定义, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Operator schema / 算子模式, Type definition / 类型定义, Branching logic / 分支逻辑。

### Lines 113-140 / 第 113-140 行

```cpp
    vcruntime_plat = L"x86";
  }

  // Getting environment variables after activating VS development shell
  cmd = L"\"" + path + L"\" " + vcruntime_plat + L">NUL && set";
  exec_out = exec(cmd);
  if (!exec_out) {
    return;
  }
  envvars = *exec_out;

  // Setting environment variables to the current environment
  std::wistringstream f(envvars);
  std::wstring envvar;
  while (getline(f, envvar, L'\n')) {
    env_list.push_back(envvar);
  }
}

static intptr_t run(const std::string& cmd) {
  // Getting the path of `cmd.exe`
  const wchar_t* comspec = _wgetenv(L"COMSPEC");
  if (!comspec) {
    comspec = L"C:\\Windows\\System32\\cmd.exe";
  }
  // Constructing the command line
  auto wCmd = c10::u8u16(cmd);
  const wchar_t* a[] = {L"/c", wCmd.c_str(), nullptr};
```

- **EN:** Important callable entry points in this range include f, run.
- **CN:** 这一段的重要可调用入口包括 f, run。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 141-168 / 第 141-168 行

```cpp
  // Constructing the env array
  // If `env_list` is not empty, then add char pointers ending with nullptr.
  // Otherwise, it will be nullptr, which implies the default env.
  std::vector<const wchar_t*> e;
  if (!env_list.empty()) {
    for (auto& s : env_list) {
      e.push_back(s.c_str());
    }
    e.push_back(nullptr);
  }
  // Running the command
  intptr_t r = _wspawnve(_P_WAIT, comspec, a, e.data());
  return r;
}
#endif

static bool programExists(const std::string& program) {
  std::stringstream ss;
  c10::printQuotedString(ss, program);
  at::jit::TemplateEnv env;
  env.s("program", ss.str());
  std::string cmd = format(check_exists_string, env);
#ifdef _MSC_VER
  return (run(cmd.c_str()) == 0);
#else
  return (system(cmd.c_str()) == 0);
#endif
}
```

- **EN:** Important callable entry points in this range include programExists, printQuotedString.
- **CN:** 这一段的重要可调用入口包括 programExists, printQuotedString。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Loop constructs iterate over IR elements, schema entries, or serialized records in this range.
- **CN:** 这一段中的循环用于遍历 IR 元素、模式条目或序列化记录。

### Lines 169-196 / 第 169-196 行

```cpp

// A single compiler config is accessed through getConfig() (below)
// Controls compilation options and may be updated based on the result
// of compilation attempts.
struct CompilerConfig {
  CompilerConfig() {
    const auto cxx_env = c10::utils::get_env("CXX");
    if (cxx_env) {
      cxx = cxx_env.value();
    }

#ifdef _MSC_VER
    activate();
#endif

    if (!programExists(cxx)) {
      TORCH_WARN("Compiler passed via CXX envvar does not exist!");
      cxx = "";
    }
  }

  ~CompilerConfig() = default;

#ifdef _MSC_VER
  std::string cxx = "cl";
  const std::string openmp_flags = "/openmp";
#elif defined(__clang__)
  std::string cxx = "clang++";
```

- **EN:** The block declares or refines core types including CompilerConfig.
- **CN:** 该代码块声明或细化了 CompilerConfig 等核心类型。
- **EN:** Important callable entry points in this range include CompilerConfig, activate, TORCH_WARN.
- **CN:** 这一段的重要可调用入口包括 CompilerConfig, activate, TORCH_WARN。
- **EN:** Graph/IR vocabulary dominates this section, so the code is likely defining or transforming JIT program structure.
- **CN:** 这一段以图/IR 术语为主，因此代码大概率在定义或变换 JIT 程序结构。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。

### Lines 197-224 / 第 197-224 行

```cpp
  const std::string openmp_flags = "-fopenmp";
#else
  std::string cxx = "g++";
  const std::string openmp_flags = "-fopenmp";
#endif
// Set openmp to true only if PyTorch is compiled with OpenMP support
// OpenMP is typically not available on MacOS platform
#if defined(_OPENMP)
  bool openmp = true;
#else
  bool openmp = false;
#endif
};

static CompilerConfig& getConfig() {
  static CompilerConfig config;
  return config;
}

// NB: -march=native not supported on PPC64 g++.  It's a bit annoying
// to do a configure-style test to decide whether or not the g++
// actually supports it or not, so we heuristically use the host
// compiler to predict if the runtime compiler supports the option we
// want.  This probably won't work if you're cross-compiling.
// NB: -march=native is disabled because it has caused problems where
// compiler and assembler do not agree on what native instruction they
// understand for AVX512. When we need better CPU performance this
// optimization can be re-enabled by tracking down the platforms where
```

- **EN:** Important callable entry points in this range include getConfig.
- **CN:** 这一段的重要可调用入口包括 getConfig。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Operator schema / 算子模式, Backend integration / 后端集成, Optimization pass / 优化 pass, Macro control flow / 宏控制流, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Operator schema / 算子模式, Backend integration / 后端集成, Optimization pass / 优化 pass, Macro control flow / 宏控制流, Result propagation / 结果传递。

### Lines 225-252 / 第 225-252 行

```cpp
// this error occurs and only selectively disabling it.
#if (defined(_MSC_VER) && !defined(_M_ARM64))
// According to https://stackoverflow.com/a/29178079, we are able to
// detect which arch level is supported by the vectorizer using
// the macro __isa_available. It is added during runtime.
// The result of __isa_available and the corresponding arch:
//  AVX       4
//  AVX2      5
//  AVX512    6
extern "C" int __isa_available;
static std::string getArchFlags() {
  if (__isa_available >= 6) {
    return "/arch:AVX512";
  } else if (__isa_available >= 5) {
    return "/arch:AVX2";
  } else if (__isa_available >= 4) {
    return "/arch:AVX";
  } else {
    return "";
  }
}
static const std::string arch_flags = getArchFlags();
static const std::string compile_string = "cd /D \"" + temp_dir +
    "\" && "
    "${cxx} /nologo /MD /O2 " +
    arch_flags +
    " /LD /EHsc "
    "${fopenmp} \"${cpp_file}\" /link /out:\"${so_file}\"";
```

- **EN:** Important callable entry points in this range include getArchFlags.
- **CN:** 这一段的重要可调用入口包括 getArchFlags。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。

### Lines 253-280 / 第 253-280 行

```cpp
#else
static const std::string compile_string =
    "\"${cxx}\" -O3 -g "
#ifndef __PPC64__
//  "-march=native "
#endif
    "-std=c++20 -fPIC ${fopenmp} -shared \"${cpp_file}\" -o \"${so_file}\" -lm";
#endif
static void runCompiler(
    const std::string& cpp_file,
    const std::string& so_file) {
  auto& config = getConfig();
  TORCH_CHECK(
      !config.cxx.empty(),
      "Failed to compile a fused CPU kernel: Compiler not found");
  at::jit::TemplateEnv env;
  env.s("cxx", config.cxx);
  env.s("fopenmp", config.openmp ? config.openmp_flags : "");
  env.s("cpp_file", cpp_file);
  env.s("so_file", so_file);
  std::string result = format(compile_string, env);
#ifdef _MSC_VER
  intptr_t r = run(result);
#else
  int r = system(result.c_str());
#endif
  if (config.openmp && r != 0) {
    std::cerr
```

- **EN:** Important callable entry points in this range include runCompiler, TORCH_CHECK.
- **CN:** 这一段的重要可调用入口包括 runCompiler, TORCH_CHECK。
- **EN:** Comments in this range document the intent of the API or the invariants expected by later code.
- **CN:** 这一段中的注释说明了 API 意图，或描述了后续代码依赖的不变式。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Backend integration / 后端集成, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Branching logic / 分支逻辑.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Backend integration / 后端集成, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Branching logic / 分支逻辑。

### Lines 281-308 / 第 281-308 行

```cpp
        << "warning: pytorch jit fuser failed to compile with openmp, trying without it...\n";
    config.openmp = false; // disable for future compiles
    return runCompiler(cpp_file, so_file);
  }
  TORCH_CHECK(r == 0, "Failed to compile a fused CPU kernel");
}

#ifdef _MSC_VER
static const std::string disas_string =
    "dumpbin /DISASM:NOBYTES \"${so_file}\"";
#else
static const std::string disas_string = "objdump -M  intel -d \"${so_file}\"";
#endif
static void disas(const std::string& so_file) {
  at::jit::TemplateEnv env;
  env.s("so_file", so_file);
  std::string cmd = format(disas_string, env);
  int r = system(cmd.c_str());
  AT_ASSERT(r == 0);
}

FusedKernelCPU::FusedKernelCPU(
    std::string name,
    std::string code,
    std::vector<TensorDesc> input_desc,
    std::vector<TensorDesc> output_desc,
    std::vector<PartitionDesc> chunk_desc,
    std::vector<PartitionDesc> concat_desc,
```

- **EN:** Important callable entry points in this range include runCompiler, TORCH_CHECK, disas, AT_ASSERT.
- **CN:** 这一段的重要可调用入口包括 runCompiler, TORCH_CHECK, disas, AT_ASSERT。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Operator schema / 算子模式, Backend integration / 后端集成, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Result propagation / 结果传递.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Operator schema / 算子模式, Backend integration / 后端集成, Declared symbols / 声明的符号, Macro control flow / 宏控制流, Result propagation / 结果传递。

### Lines 309-336 / 第 309-336 行

```cpp
    bool has_random)
    : FusedKernel(
          std::move(name),
          std::move(code),
          std::move(input_desc),
          std::move(output_desc),
          std::move(chunk_desc),
          std::move(concat_desc),
          has_random) {
  TempFile so_file(so_template, so_suffix_len);
  TempFile cpp_file(cpp_template, cpp_suffix_len);
  cpp_file.write(code_);
  cpp_file.sync();
#ifdef _MSC_VER
  so_file.close();
  cpp_file.close();
#endif
  runCompiler(cpp_file.name(), so_file.name());
  if (debugFuser() >= 2)
    disas(so_file.name());
  so_lib = std::make_unique<at::DynamicLibrary>(so_file.name().c_str());
#pragma GCC diagnostic ignored "-Wpedantic"
  kernel =
      reinterpret_cast<void (*)(uint32_t, void**)>(so_lib->sym(name_.c_str()));
#pragma GCC diagnostic pop
}

static std::shared_ptr<FusedKernel> createFusionKernel(
```

- **EN:** Important callable entry points in this range include move, so_file, cpp_file, runCompiler.
- **CN:** 这一段的重要可调用入口包括 move, so_file, cpp_file, runCompiler。
- **EN:** Conditional branches here separate fast paths, special cases, or validation logic.
- **CN:** 这里的条件分支用于区分快路径、特殊情况或校验逻辑。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Backend integration / 后端集成, Optimization pass / 优化 pass, Registration / 注册机制, Alias analysis / 别名分析, Macro control flow / 宏控制流.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Backend integration / 后端集成, Optimization pass / 优化 pass, Registration / 注册机制, Alias analysis / 别名分析, Macro control flow / 宏控制流。

### Lines 337-356 / 第 337-356 行

```cpp
    int16_t device,
    std::string name,
    std::string code,
    std::vector<TensorDesc> input_desc,
    std::vector<TensorDesc> output_desc,
    std::vector<PartitionDesc> chunk_desc,
    std::vector<PartitionDesc> concat_desc,
    bool has_random) {
  return std::make_shared<FusedKernelCPU>(
      std::move(name),
      std::move(code),
      std::move(input_desc),
      std::move(output_desc),
      std::move(chunk_desc),
      std::move(concat_desc),
      has_random);
}

static RegisterFusionBackend reg(DeviceType::CPU, createFusionKernel);
} // namespace torch::jit::fuser::cpu
```

- **EN:** Important callable entry points in this range include move, reg.
- **CN:** 这一段的重要可调用入口包括 move, reg。
- **EN:** Registration-related statements connect the implementation to operators, schemas, or runtime dispatch tables.
- **CN:** 与注册相关的语句把实现连接到算子、模式或运行时分发表。
- **EN:** Return statements in this block finalize the transformed value, queried metadata, or constructed helper object.
- **CN:** 该代码块中的返回语句会给出变换后的值、查询结果或构造好的辅助对象。
- **EN:** Concepts touched here: Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Backend integration / 后端集成, Optimization pass / 优化 pass, Registration / 注册机制.
- **CN:** 这里涉及的概念包括：Nodes and values / 节点与值, Type system / 类型系统, Operator schema / 算子模式, Backend integration / 后端集成, Optimization pass / 优化 pass, Registration / 注册机制。

## Key Concepts / 关键概念

- **Generated code pipeline** — 代码生成流水线
- **Nodes and values** — 节点与值
- **Type system** — 类型系统
- **Operator schema** — 算子模式
- **Backend integration** — 后端集成
- **Code generation** — 代码生成
- **Optimization pass** — 优化 pass
- **Registration** — 注册机制
- **Alias analysis** — 别名分析
- **Core symbols: _stati64, CompilerConfig, getTempPath, TORCH_CHECK, string, exec, decltype, rtrim** — 核心符号：_stati64、CompilerConfig、getTempPath、TORCH_CHECK、string、exec、decltype、rtrim

## Dependencies / 依赖关系

- `torch/csrc/jit/codegen/fuser/cpu/fused_kernel.h`
- `ATen/DynamicLibrary.h`
- `ATen/code_template.h`
- `c10/util/Exception.h`
- `c10/util/env.h`
- `torch/csrc/jit/codegen/fuser/compiler.h`
- `torch/csrc/jit/codegen/fuser/cpu/temp_file.h`
