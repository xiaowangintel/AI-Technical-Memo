# common.hpp — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `benchmarks/common.hpp`
- **EN:** Shared benchmark infrastructure for the repository: cache-size query, benchmark registry, common CLI options, and config-line dispatch.
- **CN:** 仓库共享的基准基础设施：查询缓存大小、维护基准注册表、解析通用命令行参数，并分发配置文件中的一行。

## Line-by-Line Analysis / 逐行分析

### Lines 1-39 — File prologue
```cpp
   1 | /***************************************************************************************************
   2 |  * Copyright (c) 2024 - 2025 Codeplay Software Ltd. All rights reserved.
   3 |  * Copyright (C) 2025 - 2026 Intel Corporation, All rights reserved.
   4 |  * SPDX-License-Identifier: BSD-3-Clause
   5 |  *
   6 |  * Redistribution and use in source and binary forms, with or without
   7 |  * modification, are permitted provided that the following conditions are met:
   8 |  *
   9 |  * 1. Redistributions of source code must retain the above copyright notice, this
  10 |  * list of conditions and the following disclaimer.
  11 |  *
  12 |  * 2. Redistributions in binary form must reproduce the above copyright notice,
  13 |  * this list of conditions and the following disclaimer in the documentation
  14 |  * and/or other materials provided with the distribution.
  15 |  *
  16 |  * 3. Neither the name of the copyright holder nor the names of its
  17 |  * contributors may be used to endorse or promote products derived from
  18 |  * this software without specific prior written permission.
  19 |  *
  20 |  * THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
  21 |  * AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
  22 |  * IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
  23 |  * DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
  24 |  * FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
  25 |  * DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
  26 |  * SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
  27 |  * CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
  28 |  * OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
  29 |  * OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
  30 |  *
  31 |  **************************************************************************************************/
  32 | 
  33 | #include <benchmark/benchmark.h>
  34 | #include <iostream>
  35 | #include <sstream>
  36 | #include <fstream>
  37 | 
  38 | ///////////////////////////////////////////////////////////////////////////////////////////////////
  39 | 
```
**EN:** License comments, include directives, and namespace/prologue code establish the compilation context for the definitions that follow.
**CN:** 许可证注释、include 指令以及命名空间/前导代码为后续定义建立编译上下文。

### Lines 40-52 — Cache-size helper
```cpp
  40 | namespace cutlass {
  41 |   static inline std::size_t get_llc_size() {
  42 |     #if defined(CUTLASS_ENABLE_SYCL)
  43 |       return compat::get_default_queue().get_device().get_info<sycl::info::device::global_mem_cache_size>();   
  44 |     #else
  45 |       cudaDeviceProp prop_struct;
  46 |       auto result = cudaGetDeviceProperties(&prop_struct, 0);
  47 |       if (result != cudaSuccess) {
  48 |         throw std::runtime_error(cudaGetErrorString(result));
  49 |       }
  50 |       return static_cast<std::size_t>(prop_struct.l2CacheSize);
  51 |     #endif
  52 |   }
```
**EN:** Queries the last-level cache size so benchmark runners can size their working sets against the device cache hierarchy.
**CN:** 查询最后一级缓存大小，使基准运行器能够按设备缓存层级估算工作集。

### Lines 55-90 — Benchmark registry
```cpp
  55 | namespace benchmark {
  56 | 
  57 |   template<typename Options>
  58 |   class BenchmarkRegistry {
  59 |     using BM_Lambda = std::function<void(::benchmark::State& state, Options const&, cutlass::KernelHardwareInfo const &)>;
  60 |     std::map<const std::string, BM_Lambda> benchmarks;
  61 | 
  62 |     static BenchmarkRegistry& get_instance() {
  63 |       static BenchmarkRegistry runner;
  64 |       return runner;
  65 |     }
  66 | 
  67 |     BenchmarkRegistry() = default;
  68 |   public:
  69 |     BenchmarkRegistry(BenchmarkRegistry const&) = delete;
  70 |     void operator=(BenchmarkRegistry const&) = delete;
  71 | 
  72 |     static auto const& get_benchmark(std::string const& name) {
  73 |       auto& benchs = get_instance().benchmarks;
  74 |       auto it = benchs.find(name);
  75 |       if (it == benchs.end()) {
  76 |         throw std::runtime_error("Benchmark not found");
  77 |       }
  78 |       return it->second;
  79 |     }
  80 | 
  81 |     static void Register(std::string const& key, BM_Lambda const func) {
  82 |       auto& benchs = get_instance().benchmarks;
  83 |       if (benchs.find(key) == benchs.end()) {
  84 |         benchs.insert(std::make_pair(key, func));
  85 |       } else {
  86 |         std::cerr << "Benchmark " << key << " duplicated." << std::endl;
  87 |       }
  88 |     }
  89 |   };
  90 | } // namespace benchmark
```
**EN:** Implements a singleton registry that stores benchmark callbacks by name, supports lookup, and prevents accidental duplicate registration.
**CN:** 实现按名称保存基准回调的单例注册表，支持查找，并避免重复注册。

### Lines 97-130 — Common benchmark options
```cpp
  97 | // Command line options parsing
  98 | struct BenchmarkOptions {
  99 | 
 100 |   bool help;
 101 |   bool error;
 102 |   std::string config_file;
 103 | 
 104 |   BenchmarkOptions():
 105 |           help(false),
 106 |           error(false)
 107 |   { }
 108 | 
 109 |   // Parses the command line
 110 |   void parse(int argc, char const **args) {
 111 |     cutlass::CommandLine cmd(argc, args);
 112 | 
 113 |     if (cmd.check_cmd_line_flag("help")) {
 114 |       help = true;
 115 |       return;
 116 |     }
 117 | 
 118 |     cmd.get_cmd_line_argument("config_file", config_file);
 119 |   }
 120 | 
 121 |   /// Prints the usage statement.
 122 |   std::ostream & print_usage(std::ostream &out) const {
 123 | 
 124 |     out << "Benchmark\n\n"
 125 |         << "Options:\n\n"
 126 |         << "  --config_file=/path/to/config_file.in\n\n";
 127 | 
 128 |     return out;
 129 |   }
 130 | };
```
**EN:** Defines help/config-file parsing and usage printing shared by all benchmark executables.
**CN:** 定义所有基准可执行文件共用的帮助、配置文件解析和用法输出逻辑。

### Lines 134-154 — Config-line to benchmark entry
```cpp
 134 | template <typename BenchOptions>
 135 | auto benchmark_main(int argc, const char **argv) -> int {
 136 |   BenchOptions options;
 137 | 
 138 |   options.parse(argc, argv);
 139 | 
 140 |   if (options.error) {
 141 |     std::cerr << "Aborting execution." << std::endl;
 142 |     return -1;
 143 |   }
 144 | 
 145 |   cutlass::KernelHardwareInfo hw_info;
 146 |   hw_info.sm_count = cutlass::KernelHardwareInfo::query_device_multiprocessor_count(hw_info.device_id);
 147 |   const auto benchmark_config = argv[0];
 148 |   auto runner = cutlass::benchmark::BenchmarkRegistry<BenchOptions>::get_benchmark(benchmark_config);
 149 | 
 150 |   std::stringstream benchmark_name;
 151 |   benchmark_name << benchmark_config << "/" << options.benchmark_name();
 152 |   ::benchmark::RegisterBenchmark(benchmark_name.str(), runner, options, hw_info)->UseManualTime();
 153 |   return 0;
 154 | }
```
**EN:** Builds a benchmark entry from one tokenized config line, parses benchmark-specific options, and registers the resulting function with Google Benchmark.
**CN:** 把一条分词后的配置行转换成基准条目，解析具体基准选项，并把函数注册到 Google Benchmark。

### Lines 157-178 — Line tokenizer and dispatcher
```cpp
 157 | template <typename BenchOptions>
 158 | void register_benchmarks(std::string line) {
 159 |   // Split the line into arguments
 160 |   std::istringstream iss(line);
 161 |   std::vector<std::string> args;
 162 |   std::string arg;
 163 | 
 164 |   while (iss >> arg) {
 165 |     args.push_back(arg);
 166 |   }
 167 | 
 168 |   // Prepare argc and argv for secondary_main
 169 |   int line_argc = static_cast<int>(args.size());
 170 |   std::vector<const char*> line_argv(line_argc);
 171 | 
 172 |   for (int i = 0; i < line_argc; ++i) {
 173 |     line_argv[i] = args[i].c_str();
 174 |   }
 175 | 
 176 |   // Call the secondary main function with the parsed arguments
 177 |   benchmark_main<BenchOptions>(line_argc, line_argv.data());
 178 | }
```
**EN:** Splits a raw config line into argv-style tokens and forwards them to the generic registration helper.
**CN:** 把原始配置行拆成类似 argv 的 token，并转发给通用注册辅助函数。

## Key Concepts / 关键概念

- Device cache awareness / 设备缓存感知
- Registry-based benchmark lookup / 基于注册表的基准查找
- Config-file-driven execution / 配置文件驱动的执行

## Dependencies / 依赖关系

- `benchmark/benchmark.h` — Google Benchmark runtime / Google Benchmark 运行时
- `cutlass::CommandLine` — option parser / 选项解析器
- `cutlass::KernelHardwareInfo` — hardware description / 硬件描述
- `BenchmarkRegistry` — string-to-runner registry / 字符串到运行器的注册表
