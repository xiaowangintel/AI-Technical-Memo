# Hybrid.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/Hybrid.h`
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
17: #pragma once
18: 
19: #include <memory>
20: #include <type_traits>
21: 
22: #include <fbjni/detail/SimpleFixedString.h>
23: 
24: #include "CoreClasses.h"
25: 
26: namespace facebook {
27: namespace jni {
28: 
29: namespace detail {
30: 
31: class BaseHybridClass {
32: public:
33:   virtual ~BaseHybridClass() {}
34: };
35: 
36: struct HybridData : public JavaClass<HybridData> {
37:   constexpr static auto kJavaDescriptor = "Lcom/facebook/jni/HybridData;";
38:   static local_ref<HybridData> create();
39: };
40: 
41: class HybridDestructor : public JavaClass<HybridDestructor> {
42:   public:
43:     static auto constexpr kJavaDescriptor = "Lcom/facebook/jni/HybridData$Destructor;";
44: 
45:   detail::BaseHybridClass* getNativePointer() const;
46: 
47:   void setNativePointer(std::unique_ptr<detail::BaseHybridClass> new_value);
48: };
49: 
50: template<typename T>
51: detail::BaseHybridClass* getNativePointer(const T* t) {
52:   return getHolder(t)->getNativePointer();
53: }
54: 
55: template<typename T>
56: void setNativePointer(const T* t, std::unique_ptr<detail::BaseHybridClass> new_value) {
57:   getHolder(t)->setNativePointer(std::move(new_value));
58: }
59: 
60: // Save space: use unified getHolder implementation.
````
- EN: Pulls in native headers such as `memory`, `type_traits`, `fbjni/detail/SimpleFixedString.h`, `CoreClasses.h`.
- CN: 引入原生头文件，例如 `memory`, `type_traits`, `fbjni/detail/SimpleFixedString.h`, `CoreClasses.h`。
- EN: Declares or extends types including `BaseHybridClass`, `HybridData`, `HybridDestructor`.
- CN: 声明或扩展类型，包括 `BaseHybridClass`, `HybridData`, `HybridDestructor`。
- EN: Implements callable logic such as `~BaseHybridClass`, `getNativePointer`, `setNativePointer`.
- CN: 实现可调用逻辑，例如 `~BaseHybridClass`, `getNativePointer`, `setNativePointer`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61: template<typename T, typename Alloc>
 62: void setNativePointer(basic_strong_ref<T, Alloc> t, std::unique_ptr<detail::BaseHybridClass> new_value) {
 63:   getHolder(&*t)->setNativePointer(std::move(new_value));
 64: }
 65: 
 66: // Save space: use unified getHolder implementation.
 67: template<typename T>
 68: void setNativePointer(alias_ref<T> t, std::unique_ptr<detail::BaseHybridClass> new_value) {
 69:   getHolder(&*t)->setNativePointer(std::move(new_value));
 70: }
 71: 
 72: // Inline rather than in cpp file so that consumers can call into
 73: // their own copy directly rather than into the DSO containing fbjni,
 74: // saving space for the symbol table.
 75: inline JField<HybridDestructor::javaobject> getDestructorField(const local_ref<JClass>& c) {
 76:   return c->template getField<HybridDestructor::javaobject>("mDestructor");
 77: }
 78: 
 79: template<typename T>
 80: local_ref<HybridDestructor::javaobject> getHolder(const T* t) {
 81:   static auto holderField = getDestructorField(t->getClass());
 82:   return t->getFieldValue(holderField);
 83: }
 84: 
 85: // JavaClass for HybridClassBase
 86: struct HybridClassBase : public JavaClass<HybridClassBase> {
 87:   constexpr static auto kJavaDescriptor = "Lcom/facebook/jni/HybridClassBase;";
 88: 
 89:   static bool isHybridClassBase(alias_ref<jclass> jclass) {
 90:     return HybridClassBase::javaClassStatic()->isAssignableFrom(jclass);
 91:   }
 92: };
 93: 
 94: template <typename Base, typename Enabled = void>
 95: struct HybridTraits {
 96:   // This static assert should actually always fail if we don't use one of the
 97:   // specializations below.
 98:   static_assert(
 99:       std::is_base_of<JObject, Base>::value ||
100:       std::is_base_of<BaseHybridClass, Base>::value,
101:       "The base of a HybridClass must be either another HybridClass or derived from JObject.");
102: };
103: 
104: template <>
105: struct HybridTraits<BaseHybridClass> {
106:  using CxxBase = BaseHybridClass;
107:  using JavaBase = JObject;
108: };
109: 
110: template <typename Base>
111: struct HybridTraits<
112:     Base,
113:     typename std::enable_if<std::is_base_of<BaseHybridClass, Base>::value>::type> {
114:  using CxxBase = Base;
115:  using JavaBase = typename Base::JavaPart;
116: };
117: 
118: template <typename Base>
119: struct HybridTraits<
120:     Base,
````
- EN: Declares or extends types including `HybridClassBase`, `HybridTraits`.
- CN: 声明或扩展类型，包括 `HybridClassBase`, `HybridTraits`。
- EN: Implements callable logic such as `setNativePointer`, `getDestructorField`, `getHolder`, `isHybridClassBase`.
- CN: 实现可调用逻辑，例如 `setNativePointer`, `getDestructorField`, `getHolder`, `isHybridClassBase`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:     typename std::enable_if<std::is_base_of<JObject, Base>::value>::type> {
122:  using CxxBase = BaseHybridClass;
123:  using JavaBase = Base;
124: };
125: 
126: // convert to HybridClass* from jhybridobject
127: template <typename T>
128: struct Convert<
129:   T, typename std::enable_if<
130:     std::is_base_of<BaseHybridClass, typename std::remove_pointer<T>::type>::value>::type> {
131:   typedef typename std::remove_pointer<T>::type::jhybridobject jniType;
132:   static T fromJni(jniType t) {
133:     if (t == nullptr) {
134:       return nullptr;
135:     }
136:     return wrap_alias(t)->cthis();
137:   }
138:   // There is no automatic return conversion for objects.
139: };
140: 
141: template<typename T>
142: struct RefReprType<T, typename std::enable_if<std::is_base_of<BaseHybridClass, T>::value, void>::type> {
143:   static_assert(std::is_same<T, void>::value,
144:       "HybridFoo (where HybridFoo derives from HybridClass<HybridFoo>) is not supported in this context. "
145:       "For an xxx_ref<HybridFoo>, you may want: xxx_ref<HybridFoo::javaobject> or HybridFoo*.");
146:   using Repr = T;
147: };
148: 
149: 
150: }
151: 
152: template <typename T, typename Base = detail::BaseHybridClass>
153: class HybridClass : public detail::HybridTraits<Base>::CxxBase {
154: public:
155:   struct JavaPart : JavaClass<JavaPart, typename detail::HybridTraits<Base>::JavaBase> {
156:     // At this point, T is incomplete, and so we cannot access
157:     // T::kJavaDescriptor directly. jtype_traits support this escape hatch for
158:     // such a case.
159:     static constexpr const char* kJavaDescriptor = nullptr;
160:     static constexpr auto /* detail::SimpleFixedString<_> */ get_instantiated_java_descriptor();
161:     static constexpr auto /* detail::SimpleFixedString<_> */ get_instantiated_base_name();
162: 
163:     using HybridType = T;
164: 
165:     // This will reach into the java object and extract the C++ instance from
166:     // the mHybridData and return it.
167:     T* cthis() const;
168: 
169:     friend class HybridClass;
170:     friend T;
171:   };
172: 
173:   using jhybridobject = typename JavaPart::javaobject;
174:   using javaobject = typename JavaPart::javaobject;
175:   typedef detail::HybridData::javaobject jhybriddata;
176: 
177:   static alias_ref<JClass> javaClassStatic() {
178:     return JavaPart::javaClassStatic();
179:   }
180: 
````
- EN: Declares or extends types including `Convert`, `RefReprType`, `HybridClass`, `JavaPart`.
- CN: 声明或扩展类型，包括 `Convert`, `RefReprType`, `HybridClass`, `JavaPart`。
- EN: Implements callable logic such as `fromJni`, `javaClassStatic`.
- CN: 实现可调用逻辑，例如 `fromJni`, `javaClassStatic`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181:   static local_ref<JClass> javaClassLocal() {
182:     std::string className(T::kJavaDescriptor + 1, strlen(T::kJavaDescriptor) - 2);
183:     return findClassLocal(className.c_str());
184:   }
185: 
186: protected:
187:   typedef HybridClass HybridBase;
188: 
189:   // This ensures that a C++ hybrid part cannot be created on its own
190:   // by default.  If a hybrid wants to enable this, it can provide its
191:   // own public ctor, or change the accessibility of this to public.
192:   using detail::HybridTraits<Base>::CxxBase::CxxBase;
193: 
194:   static void registerHybrid(std::initializer_list<JNINativeMethod> methods) {
195:     javaClassLocal()->registerNatives(methods);
196:   }
197: 
198:   static local_ref<detail::HybridData> makeHybridData(std::unique_ptr<T> cxxPart) {
199:     auto hybridData = detail::HybridData::create();
200:     setNativePointer(hybridData, std::move(cxxPart));
201:     return hybridData;
202:   }
203: 
204:   template <typename... Args>
205:   static local_ref<detail::HybridData> makeCxxInstance(Args&&... args) {
206:     return makeHybridData(std::unique_ptr<T>(new T(std::forward<Args>(args)...)));
207:   }
208: 
209:   template <typename... Args>
210:   static void setCxxInstance(alias_ref<jhybridobject> o, Args&&... args) {
211:     setNativePointer(o, std::unique_ptr<T>(new T(std::forward<Args>(args)...)));
212:   }
213: 
214: public:
215:   // Factory method for creating a hybrid object where the arguments
216:   // are used to initialize the C++ part directly without passing them
217:   // through java.  This method requires the Java part to have a ctor
218:   // which takes a HybridData, and for the C++ part to have a ctor
219:   // compatible with the arguments passed here.  For safety, the ctor
220:   // can be private, and the hybrid declared a friend of its base, so
221:   // the hybrid can only be created from here.
222:   //
223:   // Exception behavior: This can throw an exception if creating the
224:   // C++ object fails, or any JNI methods throw.
225:   template <typename... Args>
226:   static local_ref<JavaPart> newObjectCxxArgs(Args&&... args) {
227:     static bool isHybrid = detail::HybridClassBase::isHybridClassBase(javaClassStatic());
228:     auto cxxPart = std::unique_ptr<T>(new T(std::forward<Args>(args)...));
229: 
230:     local_ref<JavaPart> result;
231:     if (isHybrid) {
232:       result = JavaPart::newInstance();
233:       setNativePointer(result, std::move(cxxPart));
234:     }
235:     else {
236:       auto hybridData = makeHybridData(std::move(cxxPart));
237:       result = JavaPart::newInstance(hybridData);
238:     }
239: 
240:     return result;
````
- EN: Implements callable logic such as `javaClassLocal`, `registerHybrid`, `makeHybridData`, `makeCxxInstance`.
- CN: 实现可调用逻辑，例如 `javaClassLocal`, `registerHybrid`, `makeHybridData`, `makeCxxInstance`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-300
````cpp
241:   }
242: 
243:  // TODO? Create reusable interface for Allocatable classes and use it to
244:   // strengthen type-checking (and possibly provide a default
245:   // implementation of allocate().)
246:   template <typename... Args>
247:   static local_ref<jhybridobject> allocateWithCxxArgs(Args&&... args) {
248:     auto hybridData = makeCxxInstance(std::forward<Args>(args)...);
249:     static auto allocateMethod =
250:         javaClassStatic()->template getStaticMethod<jhybridobject(jhybriddata)>("allocate");
251:     return allocateMethod(javaClassStatic(), hybridData.get());
252:   }
253: 
254:   // Factory method for creating a hybrid object where the arguments
255:   // are passed to the java ctor.
256:   template <typename... Args>
257:   static local_ref<JavaPart> newObjectJavaArgs(Args&&... args) {
258:     return JavaPart::newInstance(std::move(args)...);
259:   }
260: 
261:   // If a hybrid class throws an exception which derives from
262:   // std::exception, it will be passed to mapException on the hybrid
263:   // class, or nearest ancestor.  This allows boilerplate exception
264:   // translation code (for example, calling throwNewJavaException on a
265:   // particular java class) to be hoisted to a common function.  If
266:   // mapException returns, then the std::exception will be translated
267:   // to Java.
268:   static void mapException(const std::exception& ex) {
269:     (void)ex;
270:   }
271: };
272: 
273: [[noreturn]] inline void throwNPE() {
274:   throwNewJavaException("java/lang/NullPointerException", "java.lang.NullPointerException");
275: }
276: 
277: // Detect whether the given class is a hybrid. If not, grab its
278: // mHybridData field.
279: // Returns a null field if the class is a hybrid, the ID of the mHybridData field if not.
280: template<typename T, typename B>
281: inline JField<detail::HybridData::javaobject> detectHybrid(alias_ref<jclass> jclass) {
282:   bool isHybrid = detail::HybridClassBase::isHybridClassBase(jclass);
283:   if (isHybrid) {
284:     return JField<detail::HybridData::javaobject>(nullptr);
285:   } else {
286:     auto result = HybridClass<T, B>::JavaPart::javaClassStatic()->template getField<detail::HybridData::javaobject>("mHybridData");
287:     if (!result) {
288:       throwNPE();
289:     }
290:     return result;
291:   }
292: }
293: 
294: inline detail::BaseHybridClass* getHybridDataFromField(const JObject* self, const JField<detail::HybridData::javaobject>& field) {
295:   const bool isHybrid = !field;
296:   if (isHybrid) {
297:     return getNativePointer(self);
298:   } else {
299:     auto hybridData = self->getFieldValue(field);
300:     if (!hybridData) {
````
- EN: Declares or extends types including `for`, `throws`, `is`.
- CN: 声明或扩展类型，包括 `for`, `throws`, `is`。
- EN: Implements callable logic such as `checking`, `newObjectJavaArgs`, `code`, `throwNPE`.
- CN: 实现可调用逻辑，例如 `checking`, `newObjectJavaArgs`, `code`, `throwNPE`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-340
````cpp
301:       throwNPE();
302:     }
303:     return getNativePointer(&*hybridData);
304:   }
305: }
306: 
307: template <typename T, typename B>
308: inline T* HybridClass<T, B>::JavaPart::cthis() const {
309:   detail::BaseHybridClass* result = nullptr;
310:   static const auto hybridDataField = detectHybrid<T, B>(this->getClass());
311:   result = getHybridDataFromField(this, hybridDataField);
312: 
313:   // I'd like to use dynamic_cast here, but -fno-rtti is the default.
314:   return static_cast<T*>(result);
315: };
316: 
317: template <typename T, typename B>
318: constexpr auto /* detail::SimpleFixedString<_> */ HybridClass<T, B>::JavaPart::get_instantiated_java_descriptor() {
319:   constexpr auto len = detail::constexpr_strlen(T::kJavaDescriptor);
320:   return detail::SimpleFixedString<len>(T::kJavaDescriptor, len);
321: }
322: 
323: template <typename T, typename B>
324: /* static */ constexpr auto /* detail::SimpleFixedString<_> */ HybridClass<T, B>::JavaPart::get_instantiated_base_name() {
325:   constexpr auto len = detail::constexpr_strlen(T::kJavaDescriptor);
326:   return detail::SimpleFixedString<len>(T::kJavaDescriptor + 1, len - 2);
327: }
328: 
329: // Given a *_ref object which refers to a hybrid class, this will reach inside
330: // of it, find the mHybridData, extract the C++ instance pointer, cast it to
331: // the appropriate type, and return it.
332: template <typename T>
333: inline auto cthis(T jthis) -> decltype(jthis->cthis()) {
334:   return jthis->cthis();
335: }
336: 
337: void HybridDataOnLoad();
338: 
339: }
340: }
````
- EN: Implements callable logic such as `JavaPart::cthis`, `JavaPart::get_instantiated_java_descriptor`, `JavaPart::get_instantiated_base_name`, `cthis`.
- CN: 实现可调用逻辑，例如 `JavaPart::cthis`, `JavaPart::get_instantiated_java_descriptor`, `JavaPart::get_instantiated_base_name`, `cthis`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `BaseHybridClass` / 符号 `BaseHybridClass`
- Symbol `HybridData` / 符号 `HybridData`
- Symbol `HybridDestructor` / 符号 `HybridDestructor`
- Symbol `HybridClassBase` / 符号 `HybridClassBase`

## Dependencies / 依赖关系
- C/C++ includes: `memory`, `type_traits`, `fbjni/detail/SimpleFixedString.h`, `CoreClasses.h`
- C/C++ 头文件: `memory`, `type_traits`, `fbjni/detail/SimpleFixedString.h`, `CoreClasses.h`
