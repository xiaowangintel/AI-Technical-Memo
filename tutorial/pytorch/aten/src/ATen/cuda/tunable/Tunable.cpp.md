# Tunable.cpp — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `aten/src/ATen/cuda/tunable/Tunable.cpp`
- **Repository / 仓库**: `pytorch` (`/root/xw/pytorch`)
- **Purpose (EN)**: Implements CUDA-side ATen helpers, kernels, runtime wrappers, or performance utilities. This specific file centers on `Tunable.cpp`. The file header highlights: "Original TunableOp is from onnxruntime. Adapting TunableOp into PyTorch." Autotuning, benchmark selection, or runtime choice of fast kernels is part of the responsibility. Descriptor/handle lifecycle management is important here.
- **Purpose (CN)**: 实现 CUDA 侧的 ATen 辅助逻辑、内核、运行时包装器或性能工具。 该文件具体围绕 `Tunable.cpp` 展开。 文件头部注释也概括了其核心职责。 该文件还负责自动调优、基准测试后的最优实现选择，或运行时快速内核的挑选。 描述符/句柄的生命周期管理是这里的重要内容。

## Line-by-Line Analysis / 逐行分析
### Lines 1-33 / 第 1-33 行

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
0010: 
0011: #include <ATen/cuda/CUDAContextLight.h>
0012: #include <ATen/cuda/tunable/Tunable.h>
0013: #include <c10/util/Exception.h>
0014: #include <c10/util/StringUtil.h>
0015: #include <c10/util/env.h>
0016: #include <torch/version.h>
0017: 
0018: 
0019: #include <fstream>
0020: #include <sstream>
0021: #include <string>
0022: #include <unordered_map>
0023: #include <unordered_set>
0024: #include <utility>
0025: #include <vector>
0026: 
0027: // for validators
0028: #ifdef USE_ROCM
0029: #ifdef _WIN32
0030: #include <hip/hip_version.h>
0031: #else
0032: #include <rocm-core/rocm_version.h>
0033: #endif
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; interacts with accelerator runtime state or GPU execution details. Key symbols: no prominent local symbols.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；与加速器运行时状态或 GPU 执行细节交互。关键符号：无明显局部符号。

### Lines 34-63 / 第 34-63 行

```cpp
0034: #define ROCBLAS_BETA_FEATURES_API
0035: #include <rocblas/rocblas.h>
0036: #include <hipblaslt/hipblaslt.h>
0037: #include <hipblaslt/hipblaslt-ext.hpp>
0038: #endif
0039: 
0040: namespace at::cuda::tunable {
0041: 
0042: TuningContext* getTuningContext() {
0043:   static TuningContext tuning_context;
0044:   return &tuning_context;
0045: }
0046: 
0047: std::ostream& operator<<(std::ostream& stream, const ResultEntry& entry) {
0048:   static const bool blaslog = c10::utils::get_env("PYTORCH_TUNABLEOP_BLAS_LOG") == "1";
0049:   if (!blaslog) {
0050:     return stream << entry.key_ << ',' << entry.time_;
0051:   }
0052:   else {
0053:     return stream << entry.key_ << ',' << entry.time_ << ",BLAS_PARAMS: " << entry.blas_sig_;
0054:   }
0055: }
0056: 
0057: // TuningResultsManager
0058: 
0059: KernelMap TuningResultsManager::Lookup(const std::string& op_signature) {
0060:   std::scoped_lock l{lock_};
0061:   auto it = results_.find(op_signature);
0062:   if (it == results_.cend()) {
0063:     return {};
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details. Key symbols: `getTuningContext`, `Lookup`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`getTuningContext`, `Lookup`。

### Lines 64-99 / 第 64-99 行

```cpp
0064:   }
0065:   return it->second;  // copied
0066: }
0067: 
0068: ResultEntry TuningResultsManager::Lookup(const std::string& op_signature, const std::string& params_signature) {
0069:   std::scoped_lock l{lock_};
0070:   auto kernel_map_it = results_.find(op_signature);
0071:   if (kernel_map_it == results_.cend()) {
0072:     TUNABLE_LOG3("missing op_signature, returning null ResultEntry for ", op_signature, ",", params_signature);
0073:     return ResultEntry::Null();
0074:   }
0075: 
0076:   const auto& km = kernel_map_it->second;
0077:   auto it = km.find(params_signature);
0078:   if (it == km.cend()) {
0079:     TUNABLE_LOG3("missing params_signature, returning null ResultEntry for ", op_signature, ",", params_signature);
0080:     return ResultEntry::Null();
0081:   }
0082:   TUNABLE_LOG3("ResultEntry found for ", op_signature, ",", params_signature);
0083:   return it->second;
0084: }
0085: 
0086: void TuningResultsManager::AddImpl(const std::string& op_signature,
0087:     const std::string& params_signature,
0088:     ResultEntry best,
0089:     KernelMap& kernel_map) {
0090:   auto it = kernel_map.find(params_signature);
0091:   if (it != kernel_map.end()) {
0092:     if (it->second != best) {
0093:       TUNABLE_LOG1(op_signature, "(", params_signature, ") already has a best kernel ",
0094:           "id=", it->second, " selected, want to add a different best kernel ", best,
0095:           ", the new kernel id will be ignored.");
0096:     }
0097:     return;
0098:   }
0099: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `Lookup`, `Null`, `AddImpl`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`Lookup`, `Null`, `AddImpl`。

### Lines 100-130 / 第 100-130 行

```cpp
0100:   TUNABLE_LOG2(op_signature, "(", params_signature, ") -> ", best);
0101:   kernel_map.emplace(params_signature, std::move(best));
0102: }
0103: 
0104: void TuningResultsManager::Add(const std::string& op_signature, const std::string& params_signature, ResultEntry best) {
0105:   bool is_new = false;
0106:   ResultEntry inserted = ResultEntry::Null();
0107: 
0108:   // ---- mutate maps under results lock ----
0109:   {
0110:     std::scoped_lock l{lock_};
0111:     auto& km = results_[op_signature];  // creates if missing
0112:     is_new = (km.find(params_signature) == km.end());
0113:     AddImpl(op_signature, params_signature, std::move(best), km);
0114:     if (is_new) {
0115:       inserted = km.at(params_signature);  // snapshot for I/O after unlocking
0116:     }
0117:   }
0118:    if (!is_new) return;  // only write once per unique (op, params)
0119: 
0120:    TuningContext* ctx = getTuningContext();
0121:   if (ctx->IsTuningEnabled() && !ctx->IsRecordUntunedEnabled()) {
0122:     InitRealtimeAppend(ctx->GetFilename(), ctx->GetTuningResultsValidator().GetAllValidators());
0123: 
0124:     if (is_new && realtime_out_ && realtime_out_->good()) {
0125:       AppendResultLine(op_signature, params_signature, inserted);
0126:     }
0127:   }
0128: 
0129: }
0130: 
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `Add`, `AddImpl`, `InitRealtimeAppend`, `AppendResultLine`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`Add`, `AddImpl`, `InitRealtimeAppend`, `AppendResultLine`。

### Lines 131-163 / 第 131-163 行

```cpp
0131: void TuningResultsManager::RecordUntuned( std::ofstream& untuned_file, const std::string& op_signature,
0132:     const std::string& params_signature, const std::string& blas_signature) {
0133:   std::scoped_lock l{lock_};
0134:   if (!untuned_file.good()) {
0135:     TORCH_WARN_ONCE("failed to open file for writing; untuned gemm will not be saved");
0136:     return;
0137:   } else {
0138:     bool isNew = false;
0139:     auto it = untuned_results_.find(op_signature);
0140:     if (it == untuned_results_.end()) {
0141:       it = untuned_results_.insert({op_signature, {}}).first;
0142:       isNew = true;
0143:     }
0144: 
0145:     auto it_kernel_map = it->second.find(params_signature);
0146:     if (it_kernel_map == it->second.end()) {
0147:       it->second.insert(params_signature);
0148:       isNew = true;
0149:     }
0150: 
0151:     if (isNew) {
0152:       static const bool blaslog = c10::utils::get_env("PYTORCH_TUNABLEOP_BLAS_LOG") == "1";
0153:       if (!blaslog) {
0154:         untuned_file << op_signature << ',' << params_signature << std::endl;
0155:       }
0156:       else {
0157:         untuned_file << op_signature << ',' << params_signature << ",BLAS_PARAMS: " << blas_signature << std::endl;
0158:       }
0159:       TUNABLE_LOG3("Untuned,", op_signature, ",", params_signature);
0160:     }
0161:   }
0162: }
0163: 
```

- **EN:** This block handles conditional branches and special cases; interacts with accelerator runtime state or GPU execution details. Key symbols: `RecordUntuned`.
- **CN:** 该代码块处理条件分支与特殊情况；与加速器运行时状态或 GPU 执行细节交互。关键符号：`RecordUntuned`。

### Lines 164-196 / 第 164-196 行

```cpp
0164: void TuningResultsManager::InitRealtimeAppend(const std::string& filename, const std::unordered_map<std::string, std::string>& validators) {
0165:   std::scoped_lock fl{realtime_file_mutex_};
0166: 
0167:   if (realtime_out_ && realtime_out_->good() && realtime_filename_ == filename) {
0168:     return;
0169:   }
0170: 
0171:   if (realtime_out_ && realtime_filename_ != filename) {
0172:     realtime_out_->flush();
0173:     realtime_out_->close();
0174:     realtime_out_.reset();
0175:     validators_written_ = false;
0176:   }
0177: 
0178:   bool file_exists = false;
0179:   bool file_empty = true;
0180: 
0181:   {
0182:     std::ifstream check_file(filename);
0183:     if (check_file.good()) {
0184:       file_exists = true;
0185:       file_empty = (check_file.peek() == std::ifstream::traits_type::eof());
0186:     }
0187:   }
0188: 
0189:   realtime_out_ = std::make_unique<std::ofstream>(filename, std::ios::out | std::ios::app);
0190: 
0191:   if (!realtime_out_->good()) {
0192:     TORCH_WARN("TunableOp realtime append: failed to open '", filename,"'");
0193:     realtime_out_.reset();
0194:     return;
0195:   }
0196: 
```

- **EN:** This block handles conditional branches and special cases; interacts with accelerator runtime state or GPU execution details. Key symbols: `InitRealtimeAppend`, `check_file`.
- **CN:** 该代码块处理条件分支与特殊情况；与加速器运行时状态或 GPU 执行细节交互。关键符号：`InitRealtimeAppend`, `check_file`。

### Lines 197-226 / 第 197-226 行

```cpp
0197:   if(!file_exists || file_empty) {
0198:     for(const auto& [key, val] : validators) {
0199:       (*realtime_out_) << "Validator," << key << ',' << val << std::endl;
0200:       realtime_out_->flush();
0201:     }
0202:     validators_written_ = true;
0203: 
0204:     TUNABLE_LOG2("Wrote validators to realtime output file");
0205:   }
0206: 
0207:   realtime_filename_ = filename;
0208: }
0209: 
0210: void TuningResultsManager::AppendResultLine(const std::string& op_sig, const std::string& param_sig, const ResultEntry& result) {
0211:   std::scoped_lock fl{realtime_file_mutex_};
0212: 
0213:   if(!realtime_out_ || !realtime_out_->good()) {
0214:     return;
0215:   }
0216: 
0217:   (*realtime_out_) << op_sig << ',' << param_sig << ',' << result << std::endl;
0218:   realtime_out_->flush(); //ensure immediate write to disk
0219: 
0220:   TUNABLE_LOG3("Realtime append: ", op_sig, "(", param_sig, ") -> ", result);
0221: }
0222: 
0223: void TuningResultsManager::CloseRealtimeAppend() {
0224:   std::scoped_lock fl{realtime_file_mutex_};
0225: 
0226: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state. Key symbols: `AppendResultLine`, `CloseRealtimeAppend`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态。关键符号：`AppendResultLine`, `CloseRealtimeAppend`。

### Lines 227-257 / 第 227-257 行

```cpp
0227:   if(realtime_out_) {
0228:     realtime_out_->flush();
0229:     realtime_out_->close();
0230:     realtime_out_.reset();
0231:     TUNABLE_LOG2("Closed realtime output file");
0232:   }
0233: }
0234: 
0235: void TuningResultsManager::Delete(const std::string& op_signature, const std::string& params_signature) {
0236:   std::scoped_lock l{lock_};
0237: 
0238:   auto it = results_.find(op_signature);
0239:   if (it == results_.end()) {
0240:     return;
0241:   }
0242: 
0243:   auto it2 = it->second.find(params_signature);
0244:   if (it2 == it->second.end()) {
0245:     return;
0246:   }
0247: 
0248:   TUNABLE_LOG2(op_signature, "(", params_signature, ")");
0249:   it->second.erase(it2);
0250: }
0251: 
0252: void TuningResultsManager::DisjointMergeImpl(
0253:     const std::string& op_signature,
0254:     const KernelMap& kernel_map,
0255:     /*out*/ std::unordered_map<std::string, KernelMap>& results) {
0256:   auto it = results.find(op_signature);
0257:   if (it == results.end()) {
```

- **EN:** This block handles conditional branches and special cases. Key symbols: `Delete`, `DisjointMergeImpl`.
- **CN:** 该代码块处理条件分支与特殊情况。关键符号：`Delete`, `DisjointMergeImpl`。

### Lines 258-287 / 第 258-287 行

```cpp
0258:     for (const auto& [param_sig, kernel_id] : kernel_map) {
0259:       TUNABLE_LOG2(op_signature, "(", param_sig, ") -> ", kernel_id);
0260:     }
0261:     results[op_signature] = kernel_map;
0262:     return;
0263:   }
0264: 
0265:   for (const auto& [params_signature, best] : kernel_map) {
0266:     AddImpl(op_signature, params_signature, best, it->second);
0267:   }
0268: }
0269: 
0270: void TuningResultsManager::Load(const std::unordered_map<std::string, KernelMap>& results_to_load) {
0271:   TUNABLE_LOG1("Loading results");
0272:   std::scoped_lock l{lock_};
0273:   for (const auto& [op_signature, kernel_map] : results_to_load) {
0274:     DisjointMergeImpl(op_signature, kernel_map, results_);
0275:   }
0276: }
0277: 
0278: ResultsMap TuningResultsManager::Dump() {
0279:   std::scoped_lock l{lock_};
0280:   return results_;
0281: }
0282: 
0283: void TuningResultsManager::DisjointMerge(const std::string& op_signature, const KernelMap& kernel_map) {
0284:   std::scoped_lock l{lock_};
0285:   DisjointMergeImpl(op_signature, kernel_map, results_);
0286: }
0287: 
```

- **EN:** This block iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value. Key symbols: `AddImpl`, `Load`, `DisjointMergeImpl`, `Dump`, `DisjointMerge`.
- **CN:** 该代码块遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值。关键符号：`AddImpl`, `Load`, `DisjointMergeImpl`, `Dump`, `DisjointMerge`。

### Lines 288-326 / 第 288-326 行

```cpp
0288: size_t TuningResultsManager::GetSize() {
0289:   size_t size = 0;
0290:   std::scoped_lock l{lock_};
0291:   for (const auto& [op_signature, kernel_map] : results_) {
0292:     size += kernel_map.size();
0293:   }
0294:   return size;
0295: }
0296: 
0297: // TuningResultsValidator
0298: 
0299: TuningResultsValidator::TuningResultsValidator() {
0300:   RegisterValidator(
0301:       "PT_VERSION",
0302:       []() { return GetPyTorchVersion(); },
0303:       [this](auto&& k) { return ValidatePyTorchVersion(std::forward<decltype(k)>(k)); });
0304: #ifdef USE_ROCM
0305:   // hip
0306:   {
0307:     // HIP version is more accurate than ROCm version.  User's environment could be a stock
0308:     // ROCm install but with a mix of newer components, making ROCm version meaningless.
0309:     std::string hip_version = c10::str(TORCH_HIP_VERSION);
0310:     RegisterValidator(
0311:        "HIP_VERSION",
0312:        [hip_version]() { return hip_version; },
0313:        [hip_version](auto&& k) {
0314:         TUNABLE_LOG1("HIP_VERSION validation: expect ", k, " to match ", hip_version);
0315:         return hip_version == k ? OK : FAIL;
0316:       });
0317:   }
0318:   // gfx arch
0319:   {
0320:     std::string gcn_arch_name = at::cuda::getCurrentDeviceProperties()->gcnArchName;
0321:     RegisterValidator(
0322:         "GCN_ARCH_NAME",
0323:         [gcn_arch_name]() { return gcn_arch_name; },
0324:         [gcn_arch_name](auto&& k) {
0325:           TUNABLE_LOG1("GCN_ARCH_NAME validation: expect ", k, " to match ", gcn_arch_name);
0326:           return gcn_arch_name == k ? OK : FAIL;
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details. Key symbols: `GetSize`, `TuningResultsValidator`, `RegisterValidator`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`GetSize`, `TuningResultsValidator`, `RegisterValidator`。

### Lines 327-357 / 第 327-357 行

```cpp
0327:         });
0328:   }
0329:   // rocblas
0330:   {
0331:     size_t rocblas_version_size;
0332:     rocblas_get_version_string_size(&rocblas_version_size);
0333:     std::string rocblas_version(rocblas_version_size - 1, '\0');
0334:     rocblas_get_version_string(rocblas_version.data(), rocblas_version_size);
0335:     RegisterValidator(
0336:         "ROCBLAS_VERSION",
0337:         [rocblas_version]() { return rocblas_version; },
0338:         [rocblas_version](auto&& k) {
0339:           TUNABLE_LOG1("ROCBLAS_VERSION validation: expect ", k, " to match ", rocblas_version);
0340:           return rocblas_version == k ? OK : FAIL;
0341:         });
0342:   }
0343:   // hipblaslt
0344:   {
0345:     int version;
0346:     std::string revision(128, '\0');
0347:     auto handle = at::cuda::getCurrentCUDABlasLtHandle();
0348:     hipblasLtGetVersion(handle, &version);
0349:     hipblasLtGetGitRevision(handle, revision.data());
0350:     std::string hipblaslt_version =
0351:         c10::str(version, "-", revision.c_str());
0352:     RegisterValidator(
0353:         "HIPBLASLT_VERSION",
0354:         [hipblaslt_version]() { return hipblaslt_version; },
0355:         [hipblaslt_version](auto&& k) {
0356:           TUNABLE_LOG1("HIPBLASLT_VERSION validation: expect ", k, " to match ", hipblaslt_version);
0357:           return hipblaslt_version == k ? OK : FAIL;
```

- **EN:** This block produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; connects ATen logic to backend library/resource abstractions. Key symbols: `rocblas_get_version_string_size`, `rocblas_version`, `rocblas_get_version_string`, `RegisterValidator`, `revision`, `hipblasLtGetVersion`, `hipblasLtGetGitRevision`, `str`.
- **CN:** 该代码块返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；把 ATen 逻辑连接到后端库或资源抽象。关键符号：`rocblas_get_version_string_size`, `rocblas_version`, `rocblas_get_version_string`, `RegisterValidator`, `revision`, `hipblasLtGetVersion`, `hipblasLtGetGitRevision`, `str`。

### Lines 358-387 / 第 358-387 行

```cpp
0358:         });
0359:   }
0360: #endif
0361: }
0362: 
0363: std::unordered_map<std::string, std::string> TuningResultsValidator::GetAllValidators() const {
0364:   std::unordered_map<std::string, std::string> ret;
0365:   for (const auto& [key, get_validate_func_pair] : validators_) {
0366:     const GetFunc& getter = get_validate_func_pair.first;
0367:     ret[key] = getter();
0368:   }
0369:   return ret;
0370: }
0371: 
0372: static bool CheckMandatoryKeys(
0373:     const TuningResultsValidator::GetValidateFuncs& gv_funcs,
0374:     const std::unordered_map<std::string, std::string>& to_check) {
0375:   bool passed = true;
0376:   for (const auto& k : TuningResultsValidator::mandatory_keys) {
0377:     if (gv_funcs.find(k) == gv_funcs.end()) {
0378:       passed = false;
0379:       TUNABLE_LOG1("key=\"", k, "\" is not registered for Get and Validate. ");
0380:     }
0381: 
0382:     if (to_check.find(k) == to_check.end()) {
0383:       passed = false;
0384:       TUNABLE_LOG1("key=\"", k, "\" is not provided for validation. ");
0385:     }
0386:   }
0387:   return passed;
```

- **EN:** This block uses compile-time conditionals to adapt behavior across builds or backends; handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value. Key symbols: `GetAllValidators`, `CheckMandatoryKeys`.
- **CN:** 该代码块使用编译期开关来适配不同构建配置或后端；处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值。关键符号：`GetAllValidators`, `CheckMandatoryKeys`。

### Lines 388-417 / 第 388-417 行

```cpp
0388: }
0389: 
0390: static bool CheckKeysMatching(
0391:     const TuningResultsValidator::GetValidateFuncs& gv_funcs,
0392:     const std::unordered_map<std::string, std::string>& to_check) {
0393:   auto get_keys = [](const auto& it) -> std::string { return it.first; };
0394:   std::vector<std::string> required_keys;
0395:   std::vector<std::string> provided_keys;
0396:   std::transform(gv_funcs.cbegin(), gv_funcs.cend(), std::back_inserter(required_keys), get_keys);
0397:   std::transform(to_check.cbegin(), to_check.cend(), std::back_inserter(provided_keys), get_keys);
0398:   std::sort(required_keys.begin(), required_keys.end());
0399:   std::sort(provided_keys.begin(), provided_keys.end());
0400: 
0401:   std::unordered_set<std::string> intersection;
0402:   std::set_intersection(required_keys.cbegin(), required_keys.cend(),
0403:                         provided_keys.cbegin(), provided_keys.cend(),
0404:                         std::inserter(intersection, intersection.end()));
0405:   bool matched = true;
0406:   if (intersection.size() != required_keys.size()) {
0407:     matched = false;
0408:     for (const auto& k : required_keys) {
0409:       if (intersection.find(k) == intersection.end()) {
0410:         TORCH_WARN("Unmatched validator: \"", k, "\" is required, but the tuning results does not provide it. ");
0411:       }
0412:     }
0413:   }
0414:   if (intersection.size() != provided_keys.size()) {
0415:     matched = false;
0416:     for (const auto& k : provided_keys) {
0417:       if (intersection.find(k) == intersection.end()) {
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value. Key symbols: `CheckKeysMatching`, `transform`, `sort`, `set_intersection`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值。关键符号：`CheckKeysMatching`, `transform`, `sort`, `set_intersection`。

### Lines 418-449 / 第 418-449 行

```cpp
0418:         TORCH_WARN("Unmatched validator: \"", k, "\" is provided, but pytorch is unable to consume it. ");
0419:       }
0420:     }
0421:   }
0422:   return matched;
0423: }
0424: 
0425: TuningStatus TuningResultsValidator::ValidateAll(
0426:         const std::unordered_map<std::string, std::string>& to_validate) const {
0427:   if (!CheckMandatoryKeys(validators_, to_validate)) {
0428:     return FAIL;
0429:   }
0430:   if (!CheckKeysMatching(validators_, to_validate)) {
0431:     return FAIL;
0432:   }
0433: 
0434:   for (const auto& [key, value] : to_validate) {
0435:     const auto& it = validators_.find(key);
0436:     if (it == validators_.cend()) {
0437:       TORCH_WARN("Failed to lookup validator using key ", key);
0438:       for (const auto& [key2, val2] : validators_) {
0439:         TORCH_WARN("available key ", key2);
0440:       }
0441:       return FAIL;
0442:     }
0443:     const ValidateFunc& validator = it->second.second;
0444:     if (validator(value) != OK) {
0445:       TORCH_WARN("Failed validator: ", key);
0446:       return FAIL;
0447:     }
0448:   }
0449: 
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value. Key symbols: `ValidateAll`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值。关键符号：`ValidateAll`。

### Lines 450-492 / 第 450-492 行

```cpp
0450:   return OK;
0451: }
0452: 
0453: void TuningResultsValidator::RegisterValidator(const std::string& key, const GetFunc& gf, const ValidateFunc& vf) {
0454:   if (validators_.find(key) != validators_.end()) {
0455:     TORCH_WARN("Attempting to re-register validator with key ", key);
0456:   }
0457:   else {
0458:     validators_[key] = std::make_pair(gf, vf);
0459:   }
0460: }
0461: 
0462: std::string TuningResultsValidator::GetPyTorchVersion() {
0463:   return TORCH_VERSION;
0464: }
0465: 
0466: TuningStatus TuningResultsValidator::ValidatePyTorchVersion(const std::string& value) const {
0467:   TUNABLE_LOG1("PT_VERSION validation: expect ", value, " to match ", GetPyTorchVersion());
0468:   if (value == GetPyTorchVersion()) {
0469:     return OK;
0470:   }
0471:   return FAIL;
0472: }
0473: 
0474: // TuningContext
0475: 
0476: TuningContext::TuningContext() :
0477:     enable_{false},
0478:     tuning_enable_{true},
0479:     record_untuned_enable_{false},
0480:     manager_initialized_{false},
0481:     numerics_check_enable_{false},
0482:     max_tuning_duration_ms_{30},
0483:     max_tuning_iterations_{100},
0484:     max_warmup_duration_ms_{0},
0485:     max_warmup_iterations_{0},
0486:     icache_flush_{true},
0487:     rotating_buffer_size_{-1},
0488:     results_count_from_input_file_{0},
0489:     is_shutting_down_{false}
0490: {
0491: }
0492: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manages memory allocation, buffers, or ownership boundaries. Key symbols: `RegisterValidator`, `GetPyTorchVersion`, `ValidatePyTorchVersion`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；管理内存分配、缓冲区或所有权边界。关键符号：`RegisterValidator`, `GetPyTorchVersion`, `ValidatePyTorchVersion`。

### Lines 493-522 / 第 493-522 行

```cpp
0493: TuningContext::~TuningContext() {
0494:   is_shutting_down_ = true;
0495:   if (!manager_initialized_) {
0496:     // TuningResultsManager was never initialized, no tuning requested or performed.
0497:     // This can happen in a DDP job where a python process spawns other workers
0498:     // but doesn't do any computation itself.
0499:     return;
0500:   }
0501:   TUNABLE_LOG1("Closing File");
0502:   GetTuningResultsManager().CloseRealtimeAppend(); // Since, we do instant logging by default now.
0503: 
0504:   if (untuned_file_.good()) {
0505:     untuned_file_.close();
0506:   }
0507: }
0508: 
0509: void TuningContext::EnableTunableOp(bool value) {
0510:   enable_ = value;
0511:   if (value) {
0512:     TUNABLE_LOG1("Enable TunableOp");
0513:   }
0514:   else {
0515:     TUNABLE_LOG1("Disable TunableOp");
0516:   }
0517: }
0518: 
0519: bool TuningContext::IsTunableOpEnabled() const {
0520:   static const bool eval = c10::utils::get_env("PYTORCH_TUNABLEOP_ENABLED") == "1";
0521:   if (eval) {
0522:     return true;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `GetTuningResultsManager`, `EnableTunableOp`, `IsTunableOpEnabled`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`GetTuningResultsManager`, `EnableTunableOp`, `IsTunableOpEnabled`。

### Lines 523-553 / 第 523-553 行

```cpp
0523:   }
0524:   return enable_;
0525: }
0526: 
0527: void TuningContext::EnableTuning(bool value) {
0528:   tuning_enable_ = value;
0529:   if (value) {
0530:     TUNABLE_LOG1("Enable Tuning for TunableOp");
0531:   }
0532:   else {
0533:     TUNABLE_LOG1("Disable Tuning for TunableOp");
0534:   }
0535: }
0536: 
0537: void TuningContext::EnableRecordUntuned(bool value) {
0538:   record_untuned_enable_ = value;
0539:   if (value) {
0540:     TUNABLE_LOG1("Enable Record Untuned for TunableOp");
0541:   } else {
0542:     TUNABLE_LOG1("Disable Record Untuned for TunableOp");
0543:     TUNABLE_LOG1("Closing Untuned GEMM Results File");
0544:     untuned_file_.close();
0545:   }
0546: }
0547: 
0548: bool TuningContext::IsTuningEnabled() const {
0549:   static const bool eval = c10::utils::get_env("PYTORCH_TUNABLEOP_TUNING") == "0";
0550:   if (eval) {
0551:     return false;
0552:   }
0553:   return tuning_enable_;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `EnableTuning`, `EnableRecordUntuned`, `IsTuningEnabled`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`EnableTuning`, `EnableRecordUntuned`, `IsTuningEnabled`。

### Lines 554-583 / 第 554-583 行

```cpp
0554: }
0555: 
0556: bool TuningContext::IsRecordUntunedEnabled() const {
0557:   static const bool eval = c10::utils::get_env("PYTORCH_TUNABLEOP_RECORD_UNTUNED") == "1";
0558:   if (eval) {
0559:     return true;
0560:   }
0561:   return record_untuned_enable_;
0562: }
0563: 
0564: std::ofstream& TuningContext::GetUntunedFile(){
0565:   if (!untuned_file_.is_open()) {
0566:     const auto env = c10::utils::get_env("PYTORCH_TUNABLEOP_UNTUNED_FILENAME");
0567:     std::string filename = (!env.has_value()) ? "tunableop_untuned.csv" : env.value();
0568: 
0569:     std::string device = c10::str(int(c10::cuda::current_device()));
0570:     std::size_t found = filename.rfind('.');
0571:     if (found != std::string::npos) {
0572:       filename.insert(found, device);
0573:     } else {
0574:       // all else fails, just append
0575:       filename.append(device);
0576:     }
0577: 
0578:     untuned_file_ = std::ofstream(filename, std::ios::out | std::ios::app);
0579:   }
0580:   return untuned_file_;
0581: }
0582: 
0583: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details. Key symbols: `IsRecordUntunedEnabled`, `GetUntunedFile`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`IsRecordUntunedEnabled`, `GetUntunedFile`。

### Lines 584-616 / 第 584-616 行

```cpp
0584: void TuningContext::EnableNumericsCheck(bool value) {
0585:   numerics_check_enable_ = value;
0586: }
0587: 
0588: NumericalCheckConfig TuningContext::GetNumericalCheckConfig() const {
0589:   const auto env_opt = c10::utils::get_env("PYTORCH_TUNABLEOP_NUMERICAL_CHECK");
0590: 
0591:   if (!env_opt.has_value()) {
0592:     return numerics_cfg_;
0593:   }
0594: 
0595:   const std::string& env = env_opt.value();
0596: 
0597:   if (env == "0") {
0598:     return NumericalCheckConfig(false, 1e-5, 1e-5);
0599:   }
0600: 
0601:   const size_t underscore = env.find('_');
0602: 
0603:   TORCH_CHECK(
0604:       underscore != std::string::npos,
0605:       "Invalid PYTORCH_TUNABLEOP_NUMERICAL_CHECK format. "
0606:       "Expected 'atol_rtol', got: ",
0607:       env);
0608: 
0609:   double atol = 0.0;
0610:   double rtol = 0.0;
0611: 
0612:   try {
0613:     atol = std::stod(env.substr(0, underscore));
0614:     rtol = std::stod(env.substr(underscore + 1));
0615:   } catch (const std::exception& e) {
0616:     TORCH_CHECK(false, "Failed to parse PYTORCH_TUNABLEOP_NUMERICAL_CHECK: ", e.what());
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; validates runtime invariants before continuing. Key symbols: `EnableNumericsCheck`, `GetNumericalCheckConfig`, `NumericalCheckConfig`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；在继续执行前校验运行时不变量。关键符号：`EnableNumericsCheck`, `GetNumericalCheckConfig`, `NumericalCheckConfig`。

### Lines 617-649 / 第 617-649 行

```cpp
0617:   }
0618: 
0619:   TORCH_CHECK( atol > 0.0 && rtol > 0.0, "Tolerance values must be positive. atol=", atol, ", rtol=", rtol);
0620:   return NumericalCheckConfig(true, atol, rtol);
0621: }
0622: 
0623: void TuningContext::SetNumericalCheckConfig(bool enabled, double atol, double rtol) {
0624:   TORCH_CHECK(atol > 0.0 && rtol > 0.0, "Numerical check tolerances must be positive");
0625:   numerics_cfg_ = {enabled, atol, rtol};
0626: }
0627: 
0628: bool TuningContext::IsNumericsCheckEnabled() const {
0629:   const auto cfg = GetNumericalCheckConfig();
0630:   return cfg.enabled || numerics_check_enable_;
0631: }
0632: 
0633: void TuningContext::SetMaxTuningDurationMs(int max_duration_ms) {
0634:   max_tuning_duration_ms_ = max_duration_ms < 0 ? 0 : max_duration_ms;
0635: }
0636: 
0637: int TuningContext::GetMaxTuningDurationMs() const {
0638:   static const auto env = c10::utils::get_env("PYTORCH_TUNABLEOP_MAX_TUNING_DURATION_MS");
0639:   if (env.has_value()) {
0640:     int val = stoi(env.value());
0641:     return val < 0 ? 0 : val;
0642:   }
0643:   return max_tuning_duration_ms_;
0644: }
0645: 
0646: void TuningContext::SetMaxTuningIterations(int max_iter) {
0647:   max_tuning_iterations_ = max_iter < 0 ? 0 : max_iter;
0648: }
0649: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; validates runtime invariants before continuing. Key symbols: `NumericalCheckConfig`, `SetNumericalCheckConfig`, `IsNumericsCheckEnabled`, `SetMaxTuningDurationMs`, `GetMaxTuningDurationMs`, `SetMaxTuningIterations`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；在继续执行前校验运行时不变量。关键符号：`NumericalCheckConfig`, `SetNumericalCheckConfig`, `IsNumericsCheckEnabled`, `SetMaxTuningDurationMs`, `GetMaxTuningDurationMs`, `SetMaxTuningIterations`。

### Lines 650-680 / 第 650-680 行

```cpp
0650: int TuningContext::GetMaxTuningIterations() const {
0651:   static const auto env = c10::utils::get_env("PYTORCH_TUNABLEOP_MAX_TUNING_ITERATIONS");
0652:   if (env.has_value()) {
0653:     int val = stoi(env.value());
0654:     return val < 0 ? 0 : val;
0655:   }
0656:   return max_tuning_iterations_;
0657: }
0658: 
0659: void TuningContext::SetMaxWarmupDurationMs(int max_duration_ms) {
0660:   max_warmup_duration_ms_ = max_duration_ms < 0 ? 0 : max_duration_ms;
0661: }
0662: 
0663: int TuningContext::GetMaxWarmupDurationMs() const {
0664:   static const auto env = c10::utils::get_env("PYTORCH_TUNABLEOP_MAX_WARMUP_DURATION_MS");
0665:   if (env.has_value()) {
0666:     int val = stoi(env.value());
0667:     return val < 0 ? 0 : val;
0668:   }
0669:   return max_warmup_duration_ms_;
0670: }
0671: 
0672: void TuningContext::SetMaxWarmupIterations(int max_iter) {
0673:   max_warmup_iterations_ = max_iter < 0 ? 0 : max_iter;
0674: }
0675: 
0676: int TuningContext::GetMaxWarmupIterations() const {
0677:   static const auto env = c10::utils::get_env("PYTORCH_TUNABLEOP_MAX_WARMUP_ITERATIONS");
0678:   if (env.has_value()) {
0679:     int val = stoi(env.value());
0680:     return val < 0 ? 0 : val;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value. Key symbols: `GetMaxTuningIterations`, `SetMaxWarmupDurationMs`, `GetMaxWarmupDurationMs`, `SetMaxWarmupIterations`, `GetMaxWarmupIterations`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值。关键符号：`GetMaxTuningIterations`, `SetMaxWarmupDurationMs`, `GetMaxWarmupDurationMs`, `SetMaxWarmupIterations`, `GetMaxWarmupIterations`。

### Lines 681-711 / 第 681-711 行

```cpp
0681:   }
0682:   return max_warmup_iterations_;
0683: }
0684: 
0685: void TuningContext::EnableICacheFlush(bool value) {
0686:   icache_flush_ = value;
0687: }
0688: 
0689: bool TuningContext::IsICacheFlushEnabled() const {
0690:   static const auto env = c10::utils::get_env("PYTORCH_TUNABLEOP_ICACHE_FLUSH_ENABLED");
0691:   if (env == "0") {
0692:     return false;
0693:   }
0694:   return icache_flush_;
0695: }
0696: 
0697: void TuningContext::SetRotatingBufferSize(int size) {
0698:   // Any negative rotating buffer size means l2_cache_size
0699:   // see GetRotatingBufferSize
0700:   //
0701:   // size is set in MB like the environment variable
0702:   constexpr int MB = 1024 * 1024;
0703:   rotating_buffer_size_ = size * MB;
0704: }
0705: 
0706: int TuningContext::GetRotatingBufferSize() const {
0707:   // If the environment variable is negative or not set, return the L2 cache size.
0708:   // The default rotating_buffer_size is -1, but this member function will
0709:   // return l2_cache size.
0710:   // This member function will always return a zero or a positive integer.
0711:   static const auto env = c10::utils::get_env("PYTORCH_TUNABLEOP_ROTATING_BUFFER_SIZE");
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; manages memory allocation, buffers, or ownership boundaries. Key symbols: `EnableICacheFlush`, `IsICacheFlushEnabled`, `SetRotatingBufferSize`, `GetRotatingBufferSize`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；管理内存分配、缓冲区或所有权边界。关键符号：`EnableICacheFlush`, `IsICacheFlushEnabled`, `SetRotatingBufferSize`, `GetRotatingBufferSize`。

### Lines 712-742 / 第 712-742 行

```cpp
0712:   int l2_cache_size = at::cuda::getCurrentDeviceProperties()->l2CacheSize;
0713:   if (env.has_value()) {  // env variable is set
0714:     constexpr int MB = 1024 * 1024;
0715:     int val = stoi(env.value());
0716:     return val < 0 ? l2_cache_size : val * MB;  // env var is specified as MB, returned as bytes
0717:   }
0718:   else {  // env variable is not set
0719:     if (rotating_buffer_size_ < 0) {
0720:       return l2_cache_size;
0721:     }
0722:     else {
0723:       return rotating_buffer_size_;
0724:     }
0725:   }
0726: }
0727: 
0728: TuningResultsManager& TuningContext::GetTuningResultsManager() {
0729:   c10::call_once(manager_init_once_, [this]() {
0730:     manager_initialized_ = true;
0731:     if (GetFilename().empty()) {
0732:       // if SetFilename() was not already called, call it now with the default or env var
0733:       const auto env = c10::utils::get_env("PYTORCH_TUNABLEOP_FILENAME");
0734:       std::string filename = (!env.has_value()) ? "tunableop_results.csv" : env.value();
0735:       SetFilename(filename, true);
0736:     }
0737:     auto filename = GetFilename();
0738:     if (!filename.empty() && !IsRecordUntunedEnabled()) {
0739:       ReadFile(filename);
0740:     }
0741:   });
0742:   return manager_;
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; manages memory allocation, buffers, or ownership boundaries. Key symbols: `GetTuningResultsManager`, `call_once`, `SetFilename`, `ReadFile`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；管理内存分配、缓冲区或所有权边界。关键符号：`GetTuningResultsManager`, `call_once`, `SetFilename`, `ReadFile`。

### Lines 743-773 / 第 743-773 行

```cpp
0743: }
0744: 
0745: TuningResultsValidator& TuningContext::GetTuningResultsValidator() {
0746:   return validator_;
0747: }
0748: 
0749: TuningResults TuningContext::GetTuningResults() {
0750:   TuningResults tr;
0751:   tr.validators = GetTuningResultsValidator().GetAllValidators();
0752:   tr.results = GetTuningResultsManager().Dump();
0753:   return tr;
0754: }
0755: 
0756: TuningStatus TuningContext::LoadTuningResults(const TuningResults& tr) {
0757:   TORCH_CHECK(GetTuningResultsValidator().ValidateAll(tr.validators));
0758:   GetTuningResultsManager().Load(tr.results);
0759:   return OK;
0760: }
0761: 
0762: void TuningContext::SetFilename(const std::string& filename, bool insert_device_ordinal) {
0763:   filename_ = filename;
0764: 
0765:   if (filename_.empty()) {
0766:     return;
0767:   }
0768: 
0769:   if (insert_device_ordinal) {
0770:     // differentiate filename based on device ordinal to avoid
0771:     // use case of one process per device writing to same file
0772:     std::string device = c10::str(int(c10::cuda::current_device()));
0773: 
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details; validates runtime invariants before continuing. Key symbols: `GetTuningResultsValidator`, `GetTuningResults`, `LoadTuningResults`, `GetTuningResultsManager`, `SetFilename`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互；在继续执行前校验运行时不变量。关键符号：`GetTuningResultsValidator`, `GetTuningResults`, `LoadTuningResults`, `GetTuningResultsManager`, `SetFilename`。

### Lines 774-805 / 第 774-805 行

```cpp
0774:     // does filename contain %d to insert device ordinal in specific location?
0775:     const std::string TOKEN("%d");
0776:     std::size_t found = filename_.find(TOKEN);
0777:     if (found != std::string::npos) {
0778:       filename_.replace(found, TOKEN.length(), device);
0779:     }
0780:     else {
0781:       // no %d present, so append device ordinal before final '.'
0782:       found = filename_.rfind('.');
0783:       if (found != std::string::npos) {
0784:         filename_.insert(found, device);
0785:       }
0786:       else {
0787:         // all else fails, just append
0788:         filename_.append(device);
0789:       }
0790:     }
0791:   }
0792: }
0793: 
0794: std::string TuningContext::GetFilename() const {
0795:   return filename_;
0796: }
0797: 
0798: bool TuningContext::ReadFile(const std::string& filename_) {
0799:   std::string filename = filename_.empty() ? GetFilename() : filename_;
0800:   TUNABLE_LOG1("reading tuning results from ", filename);
0801:   ResultsMap results;
0802:   std::unordered_map<std::string, std::string> validators;
0803:   std::string line;
0804:   std::ifstream file(filename);
0805:   if (!file) {
```

- **EN:** This block handles conditional branches and special cases; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details. Key symbols: `GetFilename`, `ReadFile`, `file`.
- **CN:** 该代码块处理条件分支与特殊情况；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`GetFilename`, `ReadFile`, `file`。

### Lines 806-840 / 第 806-840 行

```cpp
0806:     TUNABLE_LOG1("could not open ", filename, " for reading tuning results");
0807:     return false;
0808:   }
0809:   while (std::getline(file, line)) {
0810:     if (line.empty()) {
0811:       continue;
0812:     }
0813:     std::string part;
0814:     std::vector<std::string> parts;
0815:     std::stringstream line_as_stream(line);
0816:     while (std::getline(line_as_stream, part, ',')) {
0817:       parts.push_back(part);
0818:     }
0819:     if (parts[0] == "Validator" && parts.size() >= 3) {
0820:       validators[parts[1]] = parts[2];
0821:       TUNABLE_LOG1("Validator ", parts[1], "=", parts[2]);
0822:     }
0823:     else if (parts.size() >= 4) {
0824:       results[parts[0]].emplace(parts[1], ResultEntry(parts[2], atof(parts[3].c_str())));
0825:     }
0826:     else if (parts.size() >= 3) {
0827:       // the timestamp from the file is optional
0828:       results[parts[0]].emplace(parts[1], ResultEntry(parts[2], 0));
0829:     }
0830:     else {
0831:       TUNABLE_LOG1("could not parse line: ", line);
0832:     }
0833:   }
0834:   if (GetTuningResultsValidator().ValidateAll(validators) != FAIL) {
0835:     manager_.Load(results);
0836:     results_count_from_input_file_ = manager_.GetSize();
0837:   }
0838:   else {
0839:     TUNABLE_LOG1("results validator check failed");
0840:     return false;
```

- **EN:** This block handles conditional branches and special cases; iterates over tensors, descriptors, options, or runtime state; produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details. Key symbols: `line_as_stream`.
- **CN:** 该代码块处理条件分支与特殊情况；遍历张量、描述符、配置项或运行时状态；返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`line_as_stream`。

### Lines 841-870 / 第 841-870 行

```cpp
0841:   }
0842:   return true;
0843: }
0844: 
0845: namespace {
0846: 
0847: struct MaybeDelete {
0848:   bool owns_pointer;
0849:   void operator()(std::ostream* os) const { if (owns_pointer) delete os; }
0850: };
0851: 
0852: using OstreamPtr = std::unique_ptr<std::ostream, MaybeDelete>;
0853: 
0854: inline OstreamPtr get_stream(const std::string& filename) {
0855:   if (filename == "out") {
0856:     return OstreamPtr { &std::cout, MaybeDelete {false} };
0857:   }
0858:   else if (filename == "err") {
0859:     return OstreamPtr { &std::cerr, MaybeDelete {false} };
0860:   }
0861:   else {
0862:     return OstreamPtr { new std::ofstream {filename.c_str()}, MaybeDelete {true} };
0863:   }
0864: }
0865: 
0866: } // anonymous namespace
0867: 
0868: std::string TuningContext::GetLogFilename() const {
0869:   static const auto env_file = c10::utils::get_env("PYTORCH_TUNABLEOP_VERBOSE_FILENAME");
0870:   static std::string val_file = env_file.has_value() ? env_file.value() : "err";
```

- **EN:** Declares core types, wrappers, or enums used by this file. Prominent symbols: `MaybeDelete`, `OstreamPtr`, `get_stream`, `GetLogFilename`.
- **CN:** 声明该文件使用的核心类型、包装器或枚举。主要符号：`MaybeDelete`, `OstreamPtr`, `get_stream`, `GetLogFilename`。

### Lines 871-889 / 第 871-889 行

```cpp
0871:   return val_file;
0872: }
0873: 
0874: int TuningContext::GetLogLevel() const {
0875:   static const auto env_verbose = c10::utils::get_env("PYTORCH_TUNABLEOP_VERBOSE");
0876:   static int val_verbose = env_verbose.has_value() ? stoi(env_verbose.value()) : 0;
0877:   return val_verbose;
0878: }
0879: 
0880: bool TuningContext::GetLogOkay() const {
0881:   return !is_shutting_down_;
0882: }
0883: 
0884: std::ostream& TuningContext::GetLog() const {
0885:   static auto streamptr = get_stream(GetLogFilename());
0886:   return *streamptr;
0887: }
0888: 
0889: } // namespace at::cuda::tunable
```

- **EN:** This block produces a result or forwards a computed value; interacts with accelerator runtime state or GPU execution details. Key symbols: `GetLogLevel`, `GetLogOkay`, `GetLog`.
- **CN:** 该代码块返回结果或转发已计算的值；与加速器运行时状态或 GPU 执行细节交互。关键符号：`GetLogLevel`, `GetLogOkay`, `GetLog`。


## Key Concepts / 关键概念
- **CUDA backend plumbing** — CUDA 后端基础设施
- **Dispatch and backend routing** — 调度与后端路由
- **GPU kernel/runtime interaction** — GPU 内核/运行时交互
- **Vendor library descriptor management** — 厂商库描述符管理
- **Linear algebra backend integration** — 线性代数后端集成
- **Device and stream coordination** — 设备与流协调
- **Mixed-precision/autocast logic** — 混合精度/自动混精逻辑
- **Runtime validation and invariants** — 运行时校验与不变量
- **Parallel scan/reduction behavior** — 并行扫描/归约行为
- **Core symbols: MaybeDelete, OstreamPtr, getTuningContext, Lookup, Null, AddImpl, Add, InitRealtimeAppend** — 核心符号：MaybeDelete、OstreamPtr、getTuningContext、Lookup、Null、AddImpl、Add、InitRealtimeAppend

## Dependencies / 依赖关系
- **Internal includes / 内部头文件**: `ATen/cuda/CUDAContextLight.h`, `ATen/cuda/tunable/Tunable.h`, `c10/util/Exception.h`, `c10/util/StringUtil.h`, `c10/util/env.h`, `torch/version.h`
- **External includes / 外部头文件**: `fstream`, `sstream`, `string`, `unordered_map`, `unordered_set`, `utility`, `vector`, `hip/hip_version.h`, `rocm-core/rocm_version.h`, `rocblas/rocblas.h`, `hipblaslt/hipblaslt.h`, `hipblaslt/hipblaslt-ext.hpp`
- **Namespaces / 命名空间**: `at::cuda::tunable`, `std::string`
- **Representative symbols / 代表性符号**: `MaybeDelete`, `OstreamPtr`, `getTuningContext`, `Lookup`, `Null`, `AddImpl`, `Add`, `InitRealtimeAppend`, `AppendResultLine`, `RecordUntuned`, `check_file`, `CloseRealtimeAppend`, `...`
