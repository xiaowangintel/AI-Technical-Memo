# pytorch_jni_lite.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/pytorch_android/src/main/cpp/pytorch_jni_lite.cpp`
- Repository: `pytorch`
- Purpose (EN): Supports Android/mobile runtime integration in the PyTorch repository.
- 用途 (CN): 为 PyTorch 仓库中的 Android/移动运行时集成提供支持。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
````cpp
 1: #include <cassert>
 2: #include <iostream>
 3: #include <memory>
 4: #include <string>
 5: 
 6: #include <fbjni/ByteBuffer.h>
 7: #include <fbjni/fbjni.h>
 8: 
 9: #include <c10/util/irange.h>
10: #include <torch/csrc/jit/mobile/import.h>
11: #include <torch/csrc/jit/mobile/module.h>
12: #include <torch/script.h>
13: #include "caffe2/serialize/read_adapter_interface.h"
14: 
15: #include "pytorch_jni_common.h"
16: 
17: #ifdef __ANDROID__
18: #include <android/asset_manager.h>
19: #include <android/asset_manager_jni.h>
20: #include <android/log.h>
21: #endif
22: 
23: namespace pytorch_jni {
24: 
25: namespace {
26: 
27: struct LiteJITCallGuard {
28:   // VariableType dispatch is not included in default mobile build. We need set
29:   // this guard globally to avoid dispatch error (only for dynamic dispatch).
30:   // Thanks to the unification of Variable class and Tensor class it's no longer
31:   // required to toggle the NonVariableTypeMode per op - so it doesn't hurt to
32:   // always set NonVariableTypeMode for inference only use case.
33:   // TODO: Ideally AutoNonVariableTypeMode in this file should be changed to
34:   // InferenceMode but it's blocked due to typeahead application on Oculus
35:   // (D27943428). To unblock, we need to find out which op is making inplace
36:   // update to an inference tensor outside InferenceMode and properly guard it.
37:   torch::AutoNonVariableTypeMode non_var_guard;
38: };
39: 
40: } // namespace
````
- EN: Pulls in native headers such as `cassert`, `iostream`, `memory`, `string`.
- CN: 引入原生头文件，例如 `cassert`, `iostream`, `memory`, `string`。
- EN: Declares or extends types including `LiteJITCallGuard`, `and`, `it`.
- CN: 声明或扩展类型，包括 `LiteJITCallGuard`, `and`, `it`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41: 
42: class PytorchJni : public facebook::jni::HybridClass<PytorchJni> {
43:  private:
44:   friend HybridBase;
45:   torch::jit::mobile::Module module_;
46:   c10::DeviceType deviceType_;
47: 
48:  public:
49:   constexpr static auto kJavaDescriptor = "Lorg/pytorch/LiteNativePeer;";
50: 
51:   static facebook::jni::local_ref<jhybriddata> initHybrid(
52:       facebook::jni::alias_ref<jclass>,
53:       facebook::jni::alias_ref<jstring> modelPath,
54:       facebook::jni::alias_ref<
55:           facebook::jni::JMap<facebook::jni::JString, facebook::jni::JString>>
56:           extraFiles,
57:       jint device) {
58:     return makeCxxInstance(modelPath, extraFiles, device);
59:   }
60: 
61: #ifdef __ANDROID__
62:   static facebook::jni::local_ref<jhybriddata> initHybridAndroidAsset(
63:       facebook::jni::alias_ref<jclass>,
64:       facebook::jni::alias_ref<jstring> assetName,
65:       facebook::jni::alias_ref<jobject> assetManager,
66:       jint device) {
67:     return makeCxxInstance(assetName, assetManager, device);
68:   }
69: #endif
70: 
71:   PytorchJni(
72:       facebook::jni::alias_ref<jstring> modelPath,
73:       facebook::jni::alias_ref<
74:           facebook::jni::JMap<facebook::jni::JString, facebook::jni::JString>>
75:           extraFiles,
76:       jint device) {
77:     LiteJITCallGuard guard;
78:     std::unordered_map<std::string, std::string> extra_files;
79:     const auto has_extra = extraFiles && extraFiles->size() > 0;
80:     if (has_extra) {
````
- EN: Declares or extends types including `PytorchJni`.
- CN: 声明或扩展类型，包括 `PytorchJni`。
- EN: Implements callable logic such as `initHybrid`, `initHybridAndroidAsset`, `PytorchJni`.
- CN: 实现可调用逻辑，例如 `initHybrid`, `initHybridAndroidAsset`, `PytorchJni`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81:       for (const auto& e : *extraFiles) {
 82:         extra_files[e.first->toStdString()] = "";
 83:       }
 84:     }
 85:     deviceType_ = deviceJniCodeToDeviceType(device);
 86:     module_ = torch::jit::_load_for_mobile(
 87:         std::move(modelPath->toStdString()), std::nullopt, extra_files);
 88:     torch::jit::_load_extra_only_for_mobile(
 89:         std::move(modelPath->toStdString()), std::nullopt, extra_files);
 90:     if (has_extra) {
 91:       static auto putMethod =
 92:           facebook::jni::JMap<facebook::jni::JString, facebook::jni::JString>::
 93:               javaClassStatic()
 94:                   ->template getMethod<facebook::jni::alias_ref<jobject>(
 95:                       facebook::jni::alias_ref<jobject>,
 96:                       facebook::jni::alias_ref<jobject>)>("put");
 97:       for (const auto& ef : extra_files) {
 98:         putMethod(
 99:             extraFiles,
100:             facebook::jni::make_jstring(ef.first),
101:             facebook::jni::make_jstring(ef.second));
102:       }
103:     }
104:   }
105: 
106: #ifdef __ANDROID__
107:   PytorchJni(
108:       facebook::jni::alias_ref<jstring> assetName,
109:       facebook::jni::alias_ref<jobject> assetManager,
110:       jint device) {
111:     JNIEnv* env = facebook::jni::Environment::current();
112:     AAssetManager* mgr = AAssetManager_fromJava(env, assetManager.get());
113:     if (!mgr) {
114:       facebook::jni::throwNewJavaException(
115:           facebook::jni::gJavaLangIllegalArgumentException,
116:           "Unable to get asset manager");
117:     }
118:     AAsset* asset = AAssetManager_open(
119:         mgr, assetName->toStdString().c_str(), AASSET_MODE_BUFFER);
120:     if (!asset) {
````
- EN: Implements callable logic such as `PytorchJni`.
- CN: 实现可调用逻辑，例如 `PytorchJni`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-160
````cpp
121:       facebook::jni::throwNewJavaException(
122:           facebook::jni::gJavaLangIllegalArgumentException,
123:           "Failed to open asset '%s'",
124:           assetName->toStdString().c_str());
125:     }
126:     auto assetBuffer = AAsset_getBuffer(asset);
127:     if (!assetBuffer) {
128:       facebook::jni::throwNewJavaException(
129:           facebook::jni::gJavaLangIllegalArgumentException,
130:           "Could not get buffer for asset '%s'",
131:           assetName->toStdString().c_str());
132:     }
133:     LiteJITCallGuard guard;
134:     module_ =
135:         torch::jit::_load_for_mobile(std::make_unique<MemoryReadAdapter>(
136:             assetBuffer, AAsset_getLength(asset)));
137:     AAsset_close(asset);
138:     deviceType_ = deviceJniCodeToDeviceType(device);
139:   }
140: #endif
141: 
142:   static void registerNatives() {
143:     registerHybrid({
144:         makeNativeMethod("initHybrid", PytorchJni::initHybrid),
145: #ifdef __ANDROID__
146:         makeNativeMethod(
147:             "initHybridAndroidAsset", PytorchJni::initHybridAndroidAsset),
148: #endif
149:         makeNativeMethod("forward", PytorchJni::forward),
150:         makeNativeMethod("runMethod", PytorchJni::runMethod),
151:     });
152:   }
153: 
154:   facebook::jni::local_ref<JIValue> forward(
155:       facebook::jni::alias_ref<
156:           facebook::jni::JArrayClass<JIValue::javaobject>::javaobject>
157:           jinputs) {
158:     std::vector<at::IValue> inputs{};
159:     size_t n = jinputs->size();
160:     inputs.reserve(n);
````
- EN: Implements callable logic such as `registerNatives`, `forward`.
- CN: 实现可调用逻辑，例如 `registerNatives`, `forward`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 161-200
````cpp
161:     for (const auto i : c10::irange(n)) {
162:       at::IValue atIValue = JIValue::JIValueToAtIValue(jinputs->getElement(i));
163:       inputs.push_back(std::move(atIValue));
164:     }
165: 
166:     auto output = [&]() {
167:       LiteJITCallGuard guard;
168:       return module_.forward(inputs);
169:     }();
170:     return JIValue::newJIValueFromAtIValue(output);
171:   }
172: 
173:   facebook::jni::local_ref<JIValue> runMethod(
174:       facebook::jni::alias_ref<facebook::jni::JString::javaobject> jmethodName,
175:       facebook::jni::alias_ref<
176:           facebook::jni::JArrayClass<JIValue::javaobject>::javaobject>
177:           jinputs) {
178:     std::string methodName = jmethodName->toStdString();
179: 
180:     std::vector<at::IValue> inputs{};
181:     size_t n = jinputs->size();
182:     inputs.reserve(n);
183:     for (const auto i : c10::irange(n)) {
184:       at::IValue atIValue = JIValue::JIValueToAtIValue(jinputs->getElement(i));
185:       inputs.push_back(std::move(atIValue));
186:     }
187:     if (auto method = module_.find_method(methodName)) {
188:       auto output = [&]() {
189:         LiteJITCallGuard guard;
190:         return module_.get_method(methodName)(inputs);
191:       }();
192:       return JIValue::newJIValueFromAtIValue(output);
193:     }
194: 
195:     facebook::jni::throwNewJavaException(
196:         facebook::jni::gJavaLangIllegalArgumentException,
197:         "Undefined method %s",
198:         methodName.c_str());
199:   }
200: };
````
- EN: Implements callable logic such as `runMethod`.
- CN: 实现可调用逻辑，例如 `runMethod`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 201-209
````cpp
201: 
202: } // namespace pytorch_jni
203: 
204: JNIEXPORT jint JNICALL JNI_OnLoad(JavaVM* vm, void*) {
205:   return facebook::jni::initialize(vm, [] {
206:     pytorch_jni::common_registerNatives();
207:     pytorch_jni::PytorchJni::registerNatives();
208:   });
209: }
````
- EN: Implements callable logic such as `JNI_OnLoad`.
- CN: 实现可调用逻辑，例如 `JNI_OnLoad`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- Symbol `LiteJITCallGuard` / 符号 `LiteJITCallGuard`
- Symbol `and` / 符号 `and`
- Symbol `it` / 符号 `it`
- Symbol `PytorchJni` / 符号 `PytorchJni`

## Dependencies / 依赖关系
- C/C++ includes: `cassert`, `iostream`, `memory`, `string`, `fbjni/ByteBuffer.h`, `fbjni/fbjni.h`, `c10/util/irange.h`, `torch/csrc/jit/mobile/import.h`, `torch/csrc/jit/mobile/module.h`, `torch/script.h`
- C/C++ 头文件: `cassert`, `iostream`, `memory`, `string`, `fbjni/ByteBuffer.h`, `fbjni/fbjni.h`, `c10/util/irange.h`, `torch/csrc/jit/mobile/import.h`, `torch/csrc/jit/mobile/module.h`, `torch/script.h`
