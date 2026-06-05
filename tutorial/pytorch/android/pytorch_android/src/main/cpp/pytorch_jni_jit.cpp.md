# pytorch_jni_jit.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android/src/main/cpp/pytorch_jni_jit.cpp`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````cpp
 1: #include <cassert>
 2: #include <iostream>
 3: #include <memory>
 4: #include <string>
 5: 
 6: #include <fbjni/ByteBuffer.h>
 7: #include <fbjni/fbjni.h>
 8: 
 9: #include <ATen/record_function.h>
10: #include <torch/csrc/jit/runtime/print_handler.h>
11: #include <torch/script.h>
12: #include "caffe2/serialize/read_adapter_interface.h"
13: 
14: #include "pytorch_jni_common.h"
15: 
16: #ifdef __ANDROID__
17: #include <android/asset_manager.h>
18: #include <android/asset_manager_jni.h>
19: #include <android/log.h>
20: #endif
21: 
22: namespace pytorch_jni {
23: 
24: namespace {
25: 
26: struct JITCallGuard {
27:   // Inference only workload.
28:   c10::InferenceMode guard;
29:   // Disable graph optimizer to ensure list of unused ops are not changed for
30:   // custom mobile build.
31:   torch::jit::GraphOptimizerEnabledGuard no_optimizer_guard{false};
32: };
33: 
34: } // namespace
35: 
36: class PytorchJni : public facebook::jni::HybridClass<PytorchJni> {
37:  private:
38:   friend HybridBase;
39:   torch::jit::Module module_;
40:   c10::DeviceType deviceType_;
41: 
42:  public:
43:   constexpr static auto kJavaDescriptor = "Lorg/pytorch/NativePeer;";
44: 
45:   static facebook::jni::local_ref<jhybriddata> initHybrid(
46:       facebook::jni::alias_ref<jclass>,
47:       facebook::jni::alias_ref<jstring> modelPath,
48:       facebook::jni::alias_ref<
49:           facebook::jni::JMap<facebook::jni::JString, facebook::jni::JString>>
50:           extraFiles,
51:       jint device) {
52:     return makeCxxInstance(modelPath, extraFiles, device);
53:   }
54: 
55: #ifdef __ANDROID__
56:   static facebook::jni::local_ref<jhybriddata> initHybridAndroidAsset(
57:       facebook::jni::alias_ref<jclass>,
58:       facebook::jni::alias_ref<jstring> assetName,
59:       facebook::jni::alias_ref<jobject> assetManager,
60:       jint device) {
````
- EN: Pulls in native headers such as `cassert`, `iostream`, `memory`, `string`.
- CN: 引入原生头文件，例如 `cassert`, `iostream`, `memory`, `string`。
- EN: Declares or extends types including `JITCallGuard`, `PytorchJni`.
- CN: 声明或扩展类型，包括 `JITCallGuard`, `PytorchJni`。
- EN: Implements callable logic such as `initHybrid`, `initHybridAndroidAsset`.
- CN: 实现可调用逻辑，例如 `initHybrid`, `initHybridAndroidAsset`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61:     return makeCxxInstance(assetName, assetManager, device);
 62:   }
 63: #endif
 64: 
 65: #ifdef TRACE_ENABLED
 66:   static std::unique_ptr<at::ObserverContext> onFunctionEnter(
 67:       const at::RecordFunction& fn) {
 68:     Trace::beginSection(fn.name().str());
 69:     return nullptr;
 70:   }
 71: 
 72:   static void onFunctionExit(const at::RecordFunction&, at::ObserverContext*) {
 73:     Trace::endSection();
 74:   }
 75: #endif
 76: 
 77:   static void preModuleLoadSetupOnce() {
 78:     auto qengines = at::globalContext().supportedQEngines();
 79:     if (std::find(qengines.begin(), qengines.end(), at::QEngine::QNNPACK) !=
 80:         qengines.end()) {
 81:       at::globalContext().setQEngine(at::QEngine::QNNPACK);
 82:     }
 83: 
 84: #ifdef __ANDROID__
 85:     torch::jit::setPrintHandler([](const std::string& s) {
 86:       __android_log_print(ANDROID_LOG_DEBUG, "pytorch-print", "%s", s.c_str());
 87:     });
 88: #endif
 89: 
 90: #ifdef TRACE_ENABLED
 91:     at::addGlobalCallback(
 92:         at::RecordFunctionCallback(&onFunctionEnter, &onFunctionExit)
 93:             .scopes({RecordScope::FUNCTION, RecordScope::USER_SCOPE}));
 94: #endif
 95:   }
 96: 
 97:   void preModuleLoadSetup() {
 98:     static const int once = []() {
 99:       preModuleLoadSetupOnce();
100:       return 0;
101:     }();
102:     ((void)once);
103:   }
104: 
105:   PytorchJni(
106:       facebook::jni::alias_ref<jstring> modelPath,
107:       facebook::jni::alias_ref<
108:           facebook::jni::JMap<facebook::jni::JString, facebook::jni::JString>>
109:           extraFiles,
110:       jint device) {
111:     preModuleLoadSetup();
112:     JITCallGuard guard;
113:     std::unordered_map<std::string, std::string> extra_files;
114:     const auto has_extra = extraFiles && extraFiles->size() > 0;
115:     if (has_extra) {
116:       for (const auto& e : *extraFiles) {
117:         extra_files[e.first->toStdString()] = "";
118:       }
119:     }
120:     deviceType_ = deviceJniCodeToDeviceType(device);
````
- EN: Implements callable logic such as `onFunctionEnter`, `onFunctionExit`, `preModuleLoadSetupOnce`, `torch::jit::setPrintHandler`.
- CN: 实现可调用逻辑，例如 `onFunctionEnter`, `onFunctionExit`, `preModuleLoadSetupOnce`, `torch::jit::setPrintHandler`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:     module_ = torch::jit::load(
122:         std::move(modelPath->toStdString()), std::nullopt, extra_files);
123:     if (has_extra) {
124:       static auto putMethod =
125:           facebook::jni::JMap<facebook::jni::JString, facebook::jni::JString>::
126:               javaClassStatic()
127:                   ->template getMethod<facebook::jni::alias_ref<jobject>(
128:                       facebook::jni::alias_ref<jobject>,
129:                       facebook::jni::alias_ref<jobject>)>("put");
130:       for (const auto& ef : extra_files) {
131:         putMethod(
132:             extraFiles,
133:             facebook::jni::make_jstring(ef.first),
134:             facebook::jni::make_jstring(ef.second));
135:       }
136:     }
137: 
138:     module_.eval();
139:   }
140: 
141: #ifdef __ANDROID__
142:   PytorchJni(
143:       facebook::jni::alias_ref<jstring> assetName,
144:       facebook::jni::alias_ref<jobject> assetManager,
145:       jint device) {
146:     preModuleLoadSetup();
147:     JNIEnv* env = facebook::jni::Environment::current();
148:     AAssetManager* mgr = AAssetManager_fromJava(env, assetManager.get());
149:     if (!mgr) {
150:       facebook::jni::throwNewJavaException(
151:           facebook::jni::gJavaLangIllegalArgumentException,
152:           "Unable to get asset manager");
153:     }
154:     AAsset* asset = AAssetManager_open(
155:         mgr, assetName->toStdString().c_str(), AASSET_MODE_BUFFER);
156:     if (!asset) {
157:       facebook::jni::throwNewJavaException(
158:           facebook::jni::gJavaLangIllegalArgumentException,
159:           "Failed to open asset '%s'",
160:           assetName->toStdString().c_str());
161:     }
162:     auto assetBuffer = AAsset_getBuffer(asset);
163:     if (!assetBuffer) {
164:       facebook::jni::throwNewJavaException(
165:           facebook::jni::gJavaLangIllegalArgumentException,
166:           "Could not get buffer for asset '%s'",
167:           assetName->toStdString().c_str());
168:     }
169:     JITCallGuard guard;
170:     module_ = torch::jit::load(std::make_unique<MemoryReadAdapter>(
171:         assetBuffer, AAsset_getLength(asset)));
172:     AAsset_close(asset);
173:     module_.eval();
174:     deviceType_ = deviceJniCodeToDeviceType(device);
175:   }
176: #endif
177: 
178:   static void registerNatives() {
179:     registerHybrid({
180:         makeNativeMethod("initHybrid", PytorchJni::initHybrid),
````
- EN: Implements callable logic such as `PytorchJni`, `registerNatives`.
- CN: 实现可调用逻辑，例如 `PytorchJni`, `registerNatives`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 181-240
````cpp
181: #ifdef __ANDROID__
182:         makeNativeMethod(
183:             "initHybridAndroidAsset", PytorchJni::initHybridAndroidAsset),
184: #endif
185:         makeNativeMethod("forward", PytorchJni::forward),
186:         makeNativeMethod("runMethod", PytorchJni::runMethod),
187:     });
188:   }
189: 
190:   facebook::jni::local_ref<JIValue> forward(
191:       facebook::jni::alias_ref<
192:           facebook::jni::JArrayClass<JIValue::javaobject>::javaobject>
193:           jinputs) {
194:     Trace _s{"jni::Module::forward"};
195:     std::vector<at::IValue> inputs{};
196:     size_t n = jinputs->size();
197:     inputs.reserve(n);
198:     for (const auto i : c10::irange(n)) {
199:       at::IValue atIValue = JIValue::JIValueToAtIValue(jinputs->getElement(i));
200:       inputs.push_back(std::move(atIValue));
201:     }
202:     auto output = [&]() {
203:       JITCallGuard guard;
204:       return module_.forward(std::move(inputs));
205:     }();
206:     return JIValue::newJIValueFromAtIValue(output);
207:   }
208: 
209:   facebook::jni::local_ref<JIValue> runMethod(
210:       facebook::jni::alias_ref<facebook::jni::JString::javaobject> jmethodName,
211:       facebook::jni::alias_ref<
212:           facebook::jni::JArrayClass<JIValue::javaobject>::javaobject>
213:           jinputs) {
214:     std::string methodName = jmethodName->toStdString();
215: 
216:     std::vector<at::IValue> inputs{};
217:     size_t n = jinputs->size();
218:     inputs.reserve(n);
219:     for (const auto i : c10::irange(n)) {
220:       at::IValue atIValue = JIValue::JIValueToAtIValue(jinputs->getElement(i));
221:       inputs.push_back(std::move(atIValue));
222:     }
223:     if (auto method = module_.find_method(methodName)) {
224:       auto output = [&]() {
225:         JITCallGuard guard;
226:         return (*method)(std::move(inputs));
227:       }();
228:       return JIValue::newJIValueFromAtIValue(output);
229:     }
230: 
231:     facebook::jni::throwNewJavaException(
232:         facebook::jni::gJavaLangIllegalArgumentException,
233:         "Undefined method %s",
234:         methodName.c_str());
235:   }
236: };
237: 
238: } // namespace pytorch_jni
239: 
240: JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void*) {
````
- EN: Implements callable logic such as `forward`, `runMethod`, `JNI_OnLoad`.
- CN: 实现可调用逻辑，例如 `forward`, `runMethod`, `JNI_OnLoad`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-245
````cpp
241:   return facebook::jni::initialize(vm, [] {
242:     pytorch_jni::common_registerNatives();
243:     pytorch_jni::PytorchJni::registerNatives();
244:   });
245: }
````
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `JITCallGuard` / 符号 `JITCallGuard`
- Symbol `PytorchJni` / 符号 `PytorchJni`
- Symbol `initHybrid` / 符号 `initHybrid`
- Symbol `initHybridAndroidAsset` / 符号 `initHybridAndroidAsset`

## Dependencies / 依赖关系
- C/C++ includes: `cassert`, `iostream`, `memory`, `string`, `fbjni/ByteBuffer.h`, `fbjni/fbjni.h`, `ATen/record_function.h`, `torch/csrc/jit/runtime/print_handler.h`, `torch/script.h`, `caffe2/serialize/read_adapter_interface.h`
- C/C++ 头文件: `cassert`, `iostream`, `memory`, `string`, `fbjni/ByteBuffer.h`, `fbjni/fbjni.h`, `ATen/record_function.h`, `torch/csrc/jit/runtime/print_handler.h`, `torch/script.h`, `caffe2/serialize/read_adapter_interface.h`
