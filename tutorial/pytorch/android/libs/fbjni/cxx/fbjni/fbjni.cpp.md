# fbjni.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/fbjni.cpp`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-60
````cpp
 1: /*
 2:  * Copyright (c) Facebook, Inc. and its affiliates.
 3:  *
 4:  * Licensed under the Apache License, Version 2.0 (the "License");
 5:  * you may not use this file except in compliance with the License.
 6:  * You may obtain a copy of the License at
 7:  *
 8:  *     http://www.apache.org/licenses/LICENSE-2.0
 9:  *
10:  * Unless required by applicable law or agreed to in writing, software
11:  * distributed under the License is distributed on an "AS IS" BASIS,
12:  * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
13:  * See the License for the specific language governing permissions and
14:  * limitations under the License.
15:  */
16: 
17: #include <fbjni/fbjni.h>
18: 
19: #include <mutex>
20: #include <vector>
21: 
22: #include <fbjni/detail/utf8.h>
23: 
24: namespace facebook {
25: namespace jni {
26: 
27: jint initialize(JavaVM* vm, std::function<void()>&& init_fn) noexcept {
28:   // TODO (t7832883): DTRT when we have exception pointers
29:   static auto error_msg = std::string{"Failed to initialize fbjni"};
30:   static bool error_occured = [vm] {
31:       bool retVal = false;
32:       try {
33:         Environment::initialize(vm);
34:       } catch (std::exception& ex) {
35:         retVal = true;
36:         try {
37:           error_msg = std::string{"Failed to initialize fbjni: "} + ex.what();
38:         } catch (...) {
39:           // Ignore, we already have a fall back message
40:         }
41:       } catch (...) {
42:         retVal = true;
43:       }
44:       return retVal;
45:     }();
46: 
47:   try {
48:     if (error_occured) {
49:       throw std::runtime_error(error_msg);
50:     }
51: 
52:     init_fn();
53:   } catch (const std::exception& e) {
54:     FBJNI_LOGE("error %s", e.what());
55:     translatePendingCppExceptionToJavaException();
56:   } catch (...) {
57:     translatePendingCppExceptionToJavaException();
58:     // So Java will handle the translated exception, fall through and
59:     // return a good version number.
60:   }
````
- EN: Pulls in native headers such as `fbjni/fbjni.h`, `mutex`, `vector`, `fbjni/detail/utf8.h`.
- CN: 引入原生头文件，例如 `fbjni/fbjni.h`, `mutex`, `vector`, `fbjni/detail/utf8.h`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-120
````cpp
 61:   return JNI_VERSION_1_6;
 62: }
 63: 
 64: namespace detail {
 65: 
 66: jclass findClass(JNIEnv* env, const char* name) {
 67:   if (!env) {
 68:     throw std::runtime_error("Unable to retrieve JNIEnv*.");
 69:   }
 70:   jclass cls = env->FindClass(name);
 71:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!cls);
 72:   return cls;
 73: }
 74: 
 75: }
 76: 
 77: local_ref<JClass> findClassLocal(const char* name) {
 78:   return adopt_local(detail::findClass(detail::currentOrNull(), name));
 79: }
 80: 
 81: alias_ref<JClass> findClassStatic(const char* name) {
 82:   JNIEnv* env = detail::currentOrNull();
 83:   auto cls = adopt_local(detail::findClass(env, name));
 84:   auto leaking_ref = (jclass)env->NewGlobalRef(cls.get());
 85:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!leaking_ref);
 86:   return wrap_alias(leaking_ref);
 87: }
 88: 
 89: 
 90: // jstring /////////////////////////////////////////////////////////////////////////////////////////
 91: 
 92: std::string JString::toStdString() const {
 93:   const auto env = Environment::current();
 94:   auto utf16String = JStringUtf16Extractor(env, self());
 95:   return detail::utf16toUTF8(utf16String.chars(), utf16String.length());
 96: }
 97: 
 98: std::u16string JString::toU16String() const {
 99:   const auto env = Environment::current();
100:   auto utf16String = JStringUtf16Extractor(env, self());
101:   if (!utf16String.chars() || utf16String.length() == 0) {
102:     return {};
103:   }
104:   return std::u16string(reinterpret_cast<const char16_t*>(utf16String.chars()), utf16String.length());
105: }
106: 
107: local_ref<JString> make_jstring(const char* utf8) {
108:   if (!utf8) {
109:     return {};
110:   }
111:   const auto env = Environment::current();
112:   size_t len;
113:   size_t modlen = detail::modifiedLength(reinterpret_cast<const uint8_t*>(utf8), &len);
114:   jstring result;
115:   if (modlen == len) {
116:     // The only difference between utf8 and modifiedUTF8 is in encoding 4-byte UTF8 chars
117:     // and '\0' that is encoded on 2 bytes.
118:     //
119:     // Since modifiedUTF8-encoded string can be no shorter than it's UTF8 conterpart we
120:     // know that if those two strings are of the same length we don't need to do any
````
- EN: Implements callable logic such as `findClass`, `findClassLocal`, `findClassStatic`, `JString::toStdString`.
- CN: 实现可调用逻辑，例如 `findClass`, `findClassLocal`, `findClassStatic`, `JString::toStdString`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:     // conversion -> no 4-byte chars nor '\0'.
122:     result = env->NewStringUTF(utf8);
123:   } else {
124:     auto modified = std::vector<char>(modlen + 1); // allocate extra byte for \0
125:     detail::utf8ToModifiedUTF8(
126:       reinterpret_cast<const uint8_t*>(utf8), len,
127:       reinterpret_cast<uint8_t*>(modified.data()), modified.size());
128:     result = env->NewStringUTF(modified.data());
129:   }
130:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
131:   return adopt_local(result);
132: }
133: 
134: local_ref<JString> make_jstring(const std::u16string& utf16) {
135:   if (utf16.empty()) {
136:     return {};
137:   }
138:   const auto env = Environment::current();
139:   static_assert(
140:       sizeof(jchar) == sizeof(std::u16string::value_type),
141:       "Expecting jchar to be the same size as std::u16string::CharT");
142:   jstring result = env->NewString(reinterpret_cast<const jchar*>(utf16.c_str()), utf16.size());
143:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
144:   return adopt_local(result);
145: }
146: 
147: // JniPrimitiveArrayFunctions //////////////////////////////////////////////////////////////////////
148: 
149: #pragma push_macro("DEFINE_PRIMITIVE_METHODS")
150: #undef DEFINE_PRIMITIVE_METHODS
151: #define DEFINE_PRIMITIVE_METHODS(TYPE, NAME, SMALLNAME)                        \
152:                                                                                \
153: template<>                                                                     \
154: TYPE* JPrimitiveArray<TYPE ## Array>::getElements(jboolean* isCopy) {          \
155:   auto env = Environment::current();                                           \
156:   TYPE* res =  env->Get ## NAME ## ArrayElements(self(), isCopy);              \
157:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();                                      \
158:   return res;                                                                  \
159: }                                                                              \
160:                                                                                \
161: template<>                                                                     \
162: void JPrimitiveArray<TYPE ## Array>::releaseElements(                          \
163:     TYPE* elements, jint mode) {                                               \
164:   auto env = Environment::current();                                           \
165:   env->Release ## NAME ## ArrayElements(self(), elements, mode);               \
166:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();                                      \
167: }                                                                              \
168:                                                                                \
169: template<>                                                                     \
170: void JPrimitiveArray<TYPE ## Array>::getRegion(                                \
171:     jsize start, jsize length, TYPE* buf) {                                    \
172:   auto env = Environment::current();                                           \
173:   env->Get ## NAME ## ArrayRegion(self(), start, length, buf);                 \
174:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();                                      \
175: }                                                                              \
176:                                                                                \
177: template<>                                                                     \
178: void JPrimitiveArray<TYPE ## Array>::setRegion(                                \
179:     jsize start, jsize length, const TYPE* elements) {                         \
180:   auto env = Environment::current();                                           \
````
- EN: Implements callable logic such as `make_jstring`, `push_macro`, `releaseElements`, `getRegion`.
- CN: 实现可调用逻辑，例如 `make_jstring`, `push_macro`, `releaseElements`, `getRegion`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181:   env->Set ## NAME ## ArrayRegion(self(), start, length, elements);            \
182:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();                                      \
183: }                                                                              \
184:                                                                                \
185: local_ref<TYPE ## Array> make_ ## SMALLNAME ## _array(jsize size) {            \
186:   auto array = Environment::current()->New ## NAME ## Array(size);             \
187:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!array);                                     \
188:   return adopt_local(array);                                                   \
189: }                                                                              \
190:                                                                                \
191: template<>                                                                     \
192: local_ref<TYPE ## Array> JArray ## NAME::newArray(size_t count) {              \
193:   return make_ ## SMALLNAME ## _array(count);                                  \
194: }                                                                              \
195:                                                                                \
196: 
197: DEFINE_PRIMITIVE_METHODS(jboolean, Boolean, boolean)
198: DEFINE_PRIMITIVE_METHODS(jbyte, Byte, byte)
199: DEFINE_PRIMITIVE_METHODS(jchar, Char, char)
200: DEFINE_PRIMITIVE_METHODS(jshort, Short, short)
201: DEFINE_PRIMITIVE_METHODS(jint, Int, int)
202: DEFINE_PRIMITIVE_METHODS(jlong, Long, long)
203: DEFINE_PRIMITIVE_METHODS(jfloat, Float, float)
204: DEFINE_PRIMITIVE_METHODS(jdouble, Double, double)
205: #pragma pop_macro("DEFINE_PRIMITIVE_METHODS")
206: 
207: namespace detail {
208: 
209: detail::BaseHybridClass* HybridDestructor::getNativePointer() const {
210:   static auto pointerField = javaClassStatic()->getField<jlong>("mNativePointer");
211:   auto* value = reinterpret_cast<detail::BaseHybridClass*>(getFieldValue(pointerField));
212:   if (!value) {
213:     throwNewJavaException("java/lang/NullPointerException", "java.lang.NullPointerException");
214:   }
215:   return value;
216: }
217: 
218: void HybridDestructor::setNativePointer(
219:     std::unique_ptr<detail::BaseHybridClass> new_value) {
220:   static auto pointerField = javaClassStatic()->getField<jlong>("mNativePointer");
221:   auto old_value = std::unique_ptr<detail::BaseHybridClass>(
222:     reinterpret_cast<detail::BaseHybridClass*>(getFieldValue(pointerField)));
223:   if (new_value && old_value) {
224:     FBJNI_LOGF("Attempt to set C++ native pointer twice");
225:   }
226:   setFieldValue(pointerField, reinterpret_cast<jlong>(new_value.release()));
227: }
228: 
229: }
230: 
231: // Internal debug /////////////////////////////////////////////////////////////////////////////////
232: 
233: namespace internal {
234: 
235: ReferenceStats g_reference_stats;
236: 
237: void facebook::jni::internal::ReferenceStats::reset() noexcept {
238:   locals_deleted = globals_deleted = weaks_deleted = 0;
239: }
240: 
````
- EN: Implements callable logic such as `_array`, `NAME::newArray`, `HybridDestructor::getNativePointer`, `HybridDestructor::setNativePointer`.
- CN: 实现可调用逻辑，例如 `_array`, `NAME::newArray`, `HybridDestructor::getNativePointer`, `HybridDestructor::setNativePointer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-243
````cpp
241: }
242: 
243: }}
````
- EN: This range contributes implementation details for the file goal: Implements JNI bridge abstractions used by Android-facing native code.
- CN: 该范围为文件目标提供实现细节：实现供 Android 原生代码使用的 JNI 桥接抽象。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `findClass` / 符号 `findClass`
- Symbol `findClassLocal` / 符号 `findClassLocal`
- Symbol `findClassStatic` / 符号 `findClassStatic`
- Symbol `JString::toStdString` / 符号 `JString::toStdString`

## Dependencies / 依赖关系
- C/C++ includes: `fbjni/fbjni.h`, `mutex`, `vector`, `fbjni/detail/utf8.h`
- C/C++ 头文件: `fbjni/fbjni.h`, `mutex`, `vector`, `fbjni/detail/utf8.h`
