# Tunable.h — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/cuda/tunable/Tunable.h`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-side ATen helpers, kernels, runtime wrappers, or performance utilities. This specific file centers on `Tunable.h`. The file header highlights: "Original TunableOp is from onnxruntime. Adapting TunableOp into PyTorch." Autotuning, benchmark selection, or runtime choice of fast kernels is part of the responsibility.
- **Purpose (CN)**: 实现 CUDA 侧的 ATen 辅助逻辑、内核、运行时包装器或性能工具。 该文件具体围绕 `Tunable.h` 展开。 文件头部注释也概括了其核心职责。 该文件还负责自动调优、基准测试后的最优实现选择，或运行时快速内核的挑选。

## Line-by-Line Analysis / 逐行分析
### Lines 1-15 / 第 1-15 行

```cpp
0001: // Original TunableOp is from onnxruntime.
0002: // https://github.com/microsoft/onnxruntime/blob/main/onnxruntime/core/framework/tunable.h
0003: // https://github.com/microsoft/onnxruntime/tree/main/onnxruntime/core/providers/rocm/tunable
0004: // Copyright (c) Microsoft Corporation.
0005: // Licensed under the MIT license.
0006: //
0007: // Adapting TunableOp into PyTorch
0008: // Copyright (c) Advanced Micro Devices, Inc.
0009: //
0010: #pragma once
0011: 
0012: #include <c10/util/CallOnce.h>
0013: #include <c10/util/StringUtil.h>
0014: #include <c10/util/env.h>
0015: 
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端。关键符号：无明显局部符号。

### Lines 16-30 / 第 16-30 行

```cpp
0016: #include <fstream>
0017: #include <functional>
0018: #include <iostream>
0019: #include <memory>
0020: #include <mutex>
0021: #include <string>
0022: #include <unordered_map>
0023: #include <unordered_set>
0024: #include <utility>
0025: 
0026: #define TUNABLE_LOGV(LEVEL, ...) getTuningContext()->Log(LEVEL, __VA_ARGS__)
0027: #define TUNABLE_LOG1(...) TUNABLE_LOGV(1, __VA_ARGS__)
0028: #define TUNABLE_LOG2(...) TUNABLE_LOGV(2, __VA_ARGS__)
0029: #define TUNABLE_LOG3(...) TUNABLE_LOGV(3, __VA_ARGS__)
0030: 
```

- **EN:** Configures compile-time behavior with preprocessor directives and feature gates. Key symbols: no prominent local symbols.
- **CN:** 通过预处理指令和特性开关配置编译期行为。关键符号：无明显局部符号。

### Lines 31-50 / 第 31-50 行

```cpp
0031: namespace at::cuda::tunable {
0032: 
0033: enum TORCH_CUDA_CPP_API TuningStatus {
0034:   OK = 0,
0035:   FAIL = 1,
0036:   UNSUPPORTED = 2,
0037: };
0038: 
0039: // Mapping from params signature to kernel id
0040: class TORCH_CUDA_CPP_API ResultEntry {
0041:   public:
0042:     explicit ResultEntry(std::string  key, double time) : key_(std::move(key)), time_(time) {}
0043:     explicit ResultEntry(std::string  key, double time, std::string blas_sig ) : key_(std::move(key)), time_(time), blas_sig_(std::move(blas_sig)) {}
0044:     bool operator==(const ResultEntry& other) const { return key_ == other.key_; }
0045:     bool operator!=(const ResultEntry& other) const { return key_ != other.key_; }
0046:     operator std::string () { return key_; }
0047:     std::string GetKey() const { return key_; }
0048:     double GetTime() const { return time_; }
0049:     friend std::ostream& operator<<(std::ostream& stream, const ResultEntry& entry);
0050:     static ResultEntry Null() { return ResultEntry("Null", 0.0); }
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `ResultEntry`, `TORCH_CUDA_CPP_API`, `string`, `GetKey`, `GetTime`, `Null`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`ResultEntry`, `TORCH_CUDA_CPP_API`, `string`, `GetKey`, `GetTime`, `Null`。

### Lines 51-66 / 第 51-66 行

```cpp
0051:     static ResultEntry Default() { return ResultEntry("Default", 0.0); }
0052: 
0053:   private:
0054:     std::string key_;
0055:     double time_;
0056:     std::string blas_sig_;
0057: };
0058: 
0059: typedef std::unordered_map<std::string, ResultEntry> KernelMap;
0060: typedef std::unordered_map<std::string, KernelMap> ResultsMap;
0061: typedef std::unordered_map<std::string, std::unordered_set<std::string>> UntunedMap;
0062: 
0063: struct TORCH_CUDA_CPP_API TuningResults {
0064:   // Validates if these results are compatible with the libraries
0065:   std::unordered_map<std::string, std::string> validators;
0066: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `TuningResults`, `Default`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`TuningResults`, `Default`。

### Lines 67-84 / 第 67-84 行

```cpp
0067:   // Mapping from Callable signature to Callable's tuning result
0068:   ResultsMap results;
0069: };
0070: 
0071: class TORCH_CUDA_CPP_API TuningResultsManager {
0072:   public:
0073:     TuningResultsManager() = default;
0074:     ~TuningResultsManager() = default;
0075: 
0076:     KernelMap Lookup(const std::string& op_signature);
0077: 
0078:     ResultEntry Lookup(const std::string& op_signature, const std::string& params_signature);
0079: 
0080:     void AddImpl(const std::string& op_signature,
0081:         const std::string& params_signature,
0082:         ResultEntry best,
0083:         KernelMap& kernel_map);
0084: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `TuningResultsManager`, `Lookup`, `AddImpl`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`TuningResultsManager`, `Lookup`, `AddImpl`。

### Lines 85-99 / 第 85-99 行

```cpp
0085:     void Add(const std::string& op_signature,
0086:         const std::string& params_signature,
0087:         ResultEntry best);
0088: 
0089:     void Delete(const std::string& op_signature, const std::string& params_signature);
0090: 
0091:     void DisjointMergeImpl(
0092:         const std::string& op_signature,
0093:         const KernelMap& kernel_map,
0094:         /*out*/ ResultsMap& results);
0095: 
0096:     void Load(const ResultsMap& results_to_load);
0097: 
0098:     ResultsMap Dump();
0099: 
```

- **EN:** This block implements local helper logic for `Tunable`. Key symbols: `Add`, `Delete`, `DisjointMergeImpl`, `Load`, `Dump`.
- **CN:** 该代码块实现与 `Tunable` 相关的局部辅助逻辑。关键符号：`Add`, `Delete`, `DisjointMergeImpl`, `Load`, `Dump`。

### Lines 100-114 / 第 100-114 行

```cpp
0100:     void DisjointMerge(const std::string& op_signature, const KernelMap& kernel_map);
0101: 
0102:     size_t GetSize();
0103: 
0104:     void RecordUntuned( std::ofstream& untuned_file, const std::string& op_signature,
0105:       const std::string& params_signature, const std::string& blas_signature);
0106: 
0107:     void InitRealtimeAppend(
0108:         const std::string& filename,
0109:         const std::unordered_map<std::string, std::string>& validators);
0110: 
0111:     void AppendResultLine(const std::string& op_sig,
0112:                          const std::string& param_sig,
0113:                          const ResultEntry& result);
0114: 
```

- **EN:** This block interacts with accelerator runtime state or GPU execution details. Key symbols: `DisjointMerge`, `GetSize`, `RecordUntuned`, `InitRealtimeAppend`, `AppendResultLine`.
- **CN:** 该代码块与加速器运行时状态或 GPU 执行细节交互。关键符号：`DisjointMerge`, `GetSize`, `RecordUntuned`, `InitRealtimeAppend`, `AppendResultLine`。

### Lines 115-132 / 第 115-132 行

```cpp
0115:     void CloseRealtimeAppend();  // For clean shutdown
0116:   private:
0117:     std::mutex lock_;
0118:     std::mutex realtime_file_mutex_;
0119:     std::unique_ptr<std::ofstream> realtime_out_;
0120:     std::string realtime_filename_;
0121:     ResultsMap results_;
0122:     UntunedMap untuned_results_;
0123:     bool validators_written_ = false;
0124: 
0125: };
0126: 
0127: class TORCH_CUDA_CPP_API TuningResultsValidator {
0128:   public:
0129:     using GetFunc = std::function<std::string()>;
0130:     using ValidateFunc = std::function<TuningStatus(const std::string&)>;
0131:     using GetValidateFuncs = std::unordered_map<std::string, std::pair<GetFunc, ValidateFunc>>;
0132: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `TuningResultsValidator`, `GetFunc`, `ValidateFunc`, `GetValidateFuncs`, `CloseRealtimeAppend`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`TuningResultsValidator`, `GetFunc`, `ValidateFunc`, `GetValidateFuncs`, `CloseRealtimeAppend`。

### Lines 133-146 / 第 133-146 行

```cpp
0133:     TuningResultsValidator();
0134:     ~TuningResultsValidator() = default;
0135: 
0136:     std::unordered_map<std::string, std::string> GetAllValidators() const;
0137:     TuningStatus ValidateAll(const std::unordered_map<std::string, std::string>& to_validate) const;
0138:     void RegisterValidator(const std::string& key, const GetFunc& gf, const ValidateFunc& vf);
0139: 
0140:   protected:
0141:     static std::string GetPyTorchVersion() ;
0142:     TuningStatus ValidatePyTorchVersion(const std::string& value) const;
0143: 
0144:   public:
0145:     static constexpr const std::array mandatory_keys{"PT_VERSION"};
0146: 
```

- **EN:** This block implements local helper logic for `Tunable`. Key symbols: `TuningResultsValidator`, `GetAllValidators`, `ValidateAll`, `RegisterValidator`, `GetPyTorchVersion`, `ValidatePyTorchVersion`.
- **CN:** 该代码块实现与 `Tunable` 相关的局部辅助逻辑。关键符号：`TuningResultsValidator`, `GetAllValidators`, `ValidateAll`, `RegisterValidator`, `GetPyTorchVersion`, `ValidatePyTorchVersion`。

### Lines 147-160 / 第 147-160 行

```cpp
0147:   private:
0148:     GetValidateFuncs validators_;
0149: };
0150: 
0151: struct NumericalCheckConfig {
0152:   bool   enabled{false};
0153:   double atol{1e-5};
0154:   double rtol{1e-5};
0155: 
0156:   NumericalCheckConfig() = default;
0157:   NumericalCheckConfig(bool e, double a, double r) : enabled(e), atol(a), rtol(r) {}
0158: };
0159: 
0160: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `NumericalCheckConfig`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`NumericalCheckConfig`。

### Lines 161-175 / 第 161-175 行

```cpp
0161: class TORCH_CUDA_CPP_API TuningContext {
0162:   public:
0163:     TuningContext();
0164:     ~TuningContext();
0165:     TuningContext(TuningContext &) = delete;
0166:     TuningContext(TuningContext &&) = delete;
0167:     TuningContext &operator=(TuningContext &) = delete;
0168:     TuningContext &operator=(TuningContext &&) = delete;
0169: 
0170:     void EnableTunableOp(bool value);
0171:     bool IsTunableOpEnabled() const;
0172: 
0173:     void EnableTuning(bool value);
0174:     bool IsTuningEnabled() const;
0175: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `TuningContext`, `~TuningContext`, `EnableTunableOp`, `IsTunableOpEnabled`, `EnableTuning`, `IsTuningEnabled`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`TuningContext`, `~TuningContext`, `EnableTunableOp`, `IsTunableOpEnabled`, `EnableTuning`, `IsTuningEnabled`。

### Lines 176-190 / 第 176-190 行

```cpp
0176:     void EnableRecordUntuned(bool value);
0177:     bool IsRecordUntunedEnabled() const;
0178:     std::ofstream& GetUntunedFile();
0179: 
0180:     void EnableNumericsCheck(bool value);
0181:     bool IsNumericsCheckEnabled() const;
0182:     void SetNumericalCheckConfig(bool enabled, double atol, double rtol);
0183:     NumericalCheckConfig GetNumericalCheckConfig() const;
0184: 
0185:     void SetMaxTuningDurationMs(int max_duration_ms);
0186:     int GetMaxTuningDurationMs() const;
0187: 
0188:     void SetMaxTuningIterations(int max_iter);
0189:     int GetMaxTuningIterations() const;
0190: 
```

- **EN:** This block interacts with accelerator runtime state or GPU execution details. Key symbols: `EnableRecordUntuned`, `IsRecordUntunedEnabled`, `GetUntunedFile`, `EnableNumericsCheck`, `IsNumericsCheckEnabled`, `SetNumericalCheckConfig`, `GetNumericalCheckConfig`, `SetMaxTuningDurationMs`, `...`.
- **CN:** 该代码块与加速器运行时状态或 GPU 执行细节交互。关键符号：`EnableRecordUntuned`, `IsRecordUntunedEnabled`, `GetUntunedFile`, `EnableNumericsCheck`, `IsNumericsCheckEnabled`, `SetNumericalCheckConfig`, `GetNumericalCheckConfig`, `SetMaxTuningDurationMs`, `...`。

### Lines 191-204 / 第 191-204 行

```cpp
0191:     void SetMaxWarmupDurationMs(int max_duration_ms);
0192:     int GetMaxWarmupDurationMs() const;
0193: 
0194:     void SetMaxWarmupIterations(int max_iter);
0195:     int GetMaxWarmupIterations() const;
0196: 
0197:     void EnableICacheFlush(bool value);
0198:     bool IsICacheFlushEnabled() const;
0199: 
0200:     void SetRotatingBufferSize(int size);
0201:     int GetRotatingBufferSize() const;
0202: 
0203:     TuningResultsManager& GetTuningResultsManager();
0204: 
```

- **EN:** This block manages memory allocation, buffers, or ownership boundaries. Key symbols: `SetMaxWarmupDurationMs`, `GetMaxWarmupDurationMs`, `SetMaxWarmupIterations`, `GetMaxWarmupIterations`, `EnableICacheFlush`, `IsICacheFlushEnabled`, `SetRotatingBufferSize`, `GetRotatingBufferSize`, `...`.
- **CN:** 该代码块管理内存分配、缓冲区或所有权边界。关键符号：`SetMaxWarmupDurationMs`, `GetMaxWarmupDurationMs`, `SetMaxWarmupIterations`, `GetMaxWarmupIterations`, `EnableICacheFlush`, `IsICacheFlushEnabled`, `SetRotatingBufferSize`, `GetRotatingBufferSize`, `...`。

### Lines 205-218 / 第 205-218 行

```cpp
0205:     TuningResultsValidator& GetTuningResultsValidator();
0206: 
0207:     TuningResults GetTuningResults();
0208: 
0209:     TuningStatus LoadTuningResults(const TuningResults& tr);
0210: 
0211:     void SetFilename(const std::string& filename, bool insert_device_ordinal=false);
0212:     std::string GetFilename() const;
0213: 
0214:     bool ReadFile(const std::string& filename={});
0215: 
0216:     template<class... Types>
0217:     void Log(int level, Types... args) {
0218:       if (GetLogOkay() && GetLogLevel() >= level) {
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `GetTuningResultsValidator`, `GetTuningResults`, `LoadTuningResults`, `SetFilename`, `GetFilename`, `Log`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`GetTuningResultsValidator`, `GetTuningResults`, `LoadTuningResults`, `SetFilename`, `GetFilename`, `Log`。

### Lines 219-246 / 第 219-246 行

```cpp
0219:         GetLog() << c10::str(args...) << std::endl;
0220:       }
0221:     }
0222: 
0223:   private:
0224:     std::string GetLogFilename() const;
0225:     int GetLogLevel() const;
0226:     bool GetLogOkay() const;
0227:     std::ostream& GetLog() const;
0228: 
0229:     bool enable_;
0230:     bool tuning_enable_;
0231:     bool record_untuned_enable_;
0232:     bool manager_initialized_;
0233:     bool numerics_check_enable_;
0234:     int max_tuning_duration_ms_;
0235:     int max_tuning_iterations_;
0236:     int max_warmup_duration_ms_;
0237:     int max_warmup_iterations_;
0238:     bool icache_flush_;
0239:     int rotating_buffer_size_;
0240:     mutable TuningResultsManager manager_;
0241:     mutable c10::once_flag manager_init_once_;
0242:     TuningResultsValidator validator_;
0243:     std::string filename_;
0244:     std::ofstream untuned_file_;
0245:     size_t results_count_from_input_file_;
0246:     bool is_shutting_down_;
```

- **EN:** This block interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries. Key symbols: `GetLogFilename`, `GetLogLevel`, `GetLogOkay`, `GetLog`.
- **CN:** 该代码块与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界。关键符号：`GetLogFilename`, `GetLogLevel`, `GetLogOkay`, `GetLog`。

### Lines 247-260 / 第 247-260 行

```cpp
0247: 
0248:     NumericalCheckConfig numerics_cfg_;
0249: };
0250: 
0251: TORCH_CUDA_CPP_API TuningContext* getTuningContext();
0252: 
0253: class ITimer {
0254:   public:
0255:     ITimer() = default;
0256:     virtual ~ITimer() = default;
0257: 
0258:     virtual void Start() = 0;
0259:     virtual void End() = 0;
0260: 
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `ITimer`, `getTuningContext`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`ITimer`, `getTuningContext`。

### Lines 261-265 / 第 261-265 行

```cpp
0261:     /// Computes the elapsed time in milliseconds between Start() and End()
0262:     virtual float Duration() = 0;
0263: };
0264: 
0265: } // namespace at::cuda::tunable
```

- **EN:** This block interacts with accelerator runtime state or GPU execution details. Key symbols: no prominent local symbols.
- **CN:** 该代码块与加速器运行时状态或 GPU 执行细节交互。关键符号：无明显局部符号。


## Key Concepts / 关键概念
- **CUDA backend plumbing** — CUDA 后端基础设施
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Linear algebra backend integration** — 线性代数后端集成
- **Device and stream coordination** — 设备与流协调
- **Runtime validation and invariants** — 运行时校验与不变量
- **Core symbols: ResultEntry, TuningResults, TuningResultsManager, TuningResultsValidator, NumericalCheckConfig, TuningContext, ITimer, TORCH_CUDA_CPP_API** — 核心符号：ResultEntry、TuningResults、TuningResultsManager、TuningResultsValidator、NumericalCheckConfig、TuningContext、ITimer、TORCH_CUDA_CPP_API

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `c10/util/CallOnce.h`, `c10/util/StringUtil.h`, `c10/util/env.h`
- **External includes / 外部头文件**: `fstream`, `functional`, `iostream`, `memory`, `mutex`, `string`, `unordered_map`, `unordered_set`, `utility`
- **Namespaces / 命名空间**: `at::cuda::tunable`
- **Representative symbols / 代表性符号**: `ResultEntry`, `TuningResults`, `TuningResultsManager`, `TuningResultsValidator`, `NumericalCheckConfig`, `TuningContext`, `ITimer`, `TORCH_CUDA_CPP_API`, `GetFunc`, `ValidateFunc`, `GetValidateFuncs`, `string`, `...`
