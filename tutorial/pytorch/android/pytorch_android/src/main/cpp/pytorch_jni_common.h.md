# pytorch_jni_common.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android/src/main/cpp/pytorch_jni_common.h`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````cpp
 1: #pragma once
 2: 
 3: #include <c10/util/FunctionRef.h>
 4: #include <fbjni/fbjni.h>
 5: #include <torch/csrc/api/include/torch/types.h>
 6: #include "caffe2/serialize/read_adapter_interface.h"
 7: 
 8: #include "cmake_macros.h"
 9: 
10: #ifdef __ANDROID__
11: #include <android/log.h>
12: #define ALOGI(...) \
13:   __android_log_print(ANDROID_LOG_INFO, "pytorch-jni", __VA_ARGS__)
14: #define ALOGE(...) \
15:   __android_log_print(ANDROID_LOG_ERROR, "pytorch-jni", __VA_ARGS__)
16: #endif
17: 
18: #if defined(TRACE_ENABLED) && defined(__ANDROID__)
19: #include <android/trace.h>
20: #include <dlfcn.h>
21: #endif
22: 
23: namespace pytorch_jni {
24: 
25: constexpr static int kDeviceCPU = 1;
26: constexpr static int kDeviceVulkan = 2;
27: 
28: c10::DeviceType deviceJniCodeToDeviceType(jint deviceJniCode);
29: 
30: class Trace {
31:  public:
32: #if defined(TRACE_ENABLED) && defined(__ANDROID__)
33:   typedef void* (*fp_ATrace_beginSection)(const char* sectionName);
34:   typedef void* (*fp_ATrace_endSection)(void);
35: 
36:   static fp_ATrace_beginSection ATrace_beginSection;
37:   static fp_ATrace_endSection ATrace_endSection;
38: #endif
39: 
40:   static void ensureInit() {
````
- EN: Pulls in native headers such as `c10/util/FunctionRef.h`, `fbjni/fbjni.h`, `torch/csrc/api/include/torch/types.h`, `caffe2/serialize/read_adapter_interface.h`.
- CN: 引入原生头文件，例如 `c10/util/FunctionRef.h`, `fbjni/fbjni.h`, `torch/csrc/api/include/torch/types.h`, `caffe2/serialize/read_adapter_interface.h`。
- EN: Declares or extends types including `Trace`.
- CN: 声明或扩展类型，包括 `Trace`。
- EN: Implements callable logic such as `ensureInit`.
- CN: 实现可调用逻辑，例如 `ensureInit`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41:     if (!Trace::is_initialized_) {
42:       init();
43:       Trace::is_initialized_ = true;
44:     }
45:   }
46: 
47:   static void beginSection(const char* name) {
48:     Trace::ensureInit();
49: #if defined(TRACE_ENABLED) && defined(__ANDROID__)
50:     ATrace_beginSection(name);
51: #endif
52:   }
53: 
54:   static void endSection() {
55: #if defined(TRACE_ENABLED) && defined(__ANDROID__)
56:     ATrace_endSection();
57: #endif
58:   }
59: 
60:   Trace(const char* name) {
61:     ensureInit();
62:     beginSection(name);
63:   }
64: 
65:   ~Trace() {
66:     endSection();
67:   }
68: 
69:  private:
70:   static void init();
71:   static bool is_initialized_;
72: };
73: 
74: class MemoryReadAdapter final : public caffe2::serialize::ReadAdapterInterface {
75:  public:
76:   explicit MemoryReadAdapter(const void* data, off_t size)
77:       : data_(data), size_(size){};
78: 
79:   size_t size() const override {
80:     return size_;
````
- EN: Declares or extends types including `MemoryReadAdapter`.
- CN: 声明或扩展类型，包括 `MemoryReadAdapter`。
- EN: Implements callable logic such as `beginSection`, `endSection`, `Trace`, `~Trace`.
- CN: 实现可调用逻辑，例如 `beginSection`, `endSection`, `Trace`, `~Trace`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81:   }
 82: 
 83:   size_t read(uint64_t pos, void* buf, size_t n, const char* what = "")
 84:       const override {
 85:     memcpy(buf, (int8_t*)(data_) + pos, n);
 86:     return n;
 87:   }
 88: 
 89:   ~MemoryReadAdapter() {}
 90: 
 91:  private:
 92:   const void* data_;
 93:   off_t size_;
 94: };
 95: 
 96: class JIValue : public facebook::jni::JavaClass<JIValue> {
 97:   using DictCallback = c10::function_ref<facebook::jni::local_ref<JIValue>(
 98:       c10::Dict<c10::IValue, c10::IValue>)>;
 99: 
100:  public:
101:   constexpr static const char* kJavaDescriptor = "Lorg/pytorch/IValue;";
102: 
103:   constexpr static int kTypeCodeNull = 1;
104: 
105:   constexpr static int kTypeCodeTensor = 2;
106:   constexpr static int kTypeCodeBool = 3;
107:   constexpr static int kTypeCodeLong = 4;
108:   constexpr static int kTypeCodeDouble = 5;
109:   constexpr static int kTypeCodeString = 6;
110: 
111:   constexpr static int kTypeCodeTuple = 7;
112:   constexpr static int kTypeCodeBoolList = 8;
113:   constexpr static int kTypeCodeLongList = 9;
114:   constexpr static int kTypeCodeDoubleList = 10;
115:   constexpr static int kTypeCodeTensorList = 11;
116:   constexpr static int kTypeCodeList = 12;
117: 
118:   constexpr static int kTypeCodeDictStringKey = 13;
119:   constexpr static int kTypeCodeDictLongKey = 14;
120: 
````
- EN: Declares or extends types including `JIValue`.
- CN: 声明或扩展类型，包括 `JIValue`。
- EN: Implements callable logic such as `read`, `~MemoryReadAdapter`.
- CN: 实现可调用逻辑，例如 `read`, `~MemoryReadAdapter`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-137
````cpp
121:   static facebook::jni::local_ref<JIValue> newJIValueFromAtIValue(
122:       const at::IValue& ivalue,
123:       DictCallback stringDictCallback = newJIValueFromStringDict,
124:       DictCallback intDictCallback = newJIValueFromIntDict);
125: 
126:   static at::IValue JIValueToAtIValue(
127:       facebook::jni::alias_ref<JIValue> jivalue);
128: 
129:  private:
130:   static facebook::jni::local_ref<JIValue> newJIValueFromStringDict(
131:       c10::Dict<c10::IValue, c10::IValue>);
132:   static facebook::jni::local_ref<JIValue> newJIValueFromIntDict(
133:       c10::Dict<c10::IValue, c10::IValue>);
134: };
135: 
136: void common_registerNatives();
137: } // namespace pytorch_jni
````
- EN: This range contributes implementation details for the file goal: Supports Android/mobile runtime integration in the PyTorch repository.
- CN: 该范围为文件目标提供实现细节：为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `Trace` / 符号 `Trace`
- Symbol `MemoryReadAdapter` / 符号 `MemoryReadAdapter`
- Symbol `JIValue` / 符号 `JIValue`
- Symbol `ensureInit` / 符号 `ensureInit`

## Dependencies / 依赖关系
- C/C++ includes: `c10/util/FunctionRef.h`, `fbjni/fbjni.h`, `torch/csrc/api/include/torch/types.h`, `caffe2/serialize/read_adapter_interface.h`, `cmake_macros.h`, `android/log.h`, `android/trace.h`, `dlfcn.h`
- C/C++ 头文件: `c10/util/FunctionRef.h`, `fbjni/fbjni.h`, `torch/csrc/api/include/torch/types.h`, `caffe2/serialize/read_adapter_interface.h`, `cmake_macros.h`, `android/log.h`, `android/trace.h`, `dlfcn.h`
