# ReferenceAllocators-inl.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/ReferenceAllocators-inl.h`
- Repository: `pytorch`
- Purpose (EN): Implements JNI bridge abstractions used by Android-facing native code.
- 用途 (CN): 实现供 Android 原生代码使用的 JNI 桥接抽象。

## Line-by-Line Analysis / 逐行分析
### Lines 1-40
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
19: #include <cassert>
20: #include <new>
21: #include <atomic>
22: 
23: #include <fbjni/detail/FbjniApi.h>
24: #include "Environment.h"
25: 
26: namespace facebook {
27: namespace jni {
28: 
29: /// @cond INTERNAL
30: namespace internal {
31: 
32: // Statistics mostly provided for test (only updated if FBJNI_DEBUG_REFS is defined)
33: struct ReferenceStats {
34:   std::atomic_uint locals_created, globals_created, weaks_created,
35:                    locals_deleted, globals_deleted, weaks_deleted;
36: 
37:   void reset() noexcept;
38: };
39: 
40: extern FBJNI_API ReferenceStats g_reference_stats;
````
- EN: Pulls in native headers such as `cassert`, `new`, `atomic`, `fbjni/detail/FbjniApi.h`.
- CN: 引入原生头文件，例如 `cassert`, `new`, `atomic`, `fbjni/detail/FbjniApi.h`。
- EN: Declares or extends types including `ReferenceStats`.
- CN: 声明或扩展类型，包括 `ReferenceStats`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41: }
42: /// @endcond
43: 
44: 
45: // LocalReferenceAllocator /////////////////////////////////////////////////////////////////////////
46: 
47: inline jobject LocalReferenceAllocator::newReference(jobject original) const {
48:   internal::dbglog("Local new: %p", original);
49:   #ifdef FBJNI_DEBUG_REFS
50:     ++internal::g_reference_stats.locals_created;
51:   #endif
52:   auto ref = Environment::current()->NewLocalRef(original);
53:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
54:   return ref;
55: }
56: 
57: inline void LocalReferenceAllocator::deleteReference(jobject reference) const noexcept {
58:   internal::dbglog("Local release: %p", reference);
59: 
60:   if (reference) {
61:     #ifdef FBJNI_DEBUG_REFS
62:       ++internal::g_reference_stats.locals_deleted;
63:     #endif
64:     assert(verifyReference(reference));
65:     Environment::current()->DeleteLocalRef(reference);
66:   }
67: }
68: 
69: inline bool LocalReferenceAllocator::verifyReference(jobject reference) const noexcept {
70:   return isObjectRefType(reference, JNILocalRefType);
71: }
72: 
73: 
74: // GlobalReferenceAllocator ////////////////////////////////////////////////////////////////////////
75: 
76: inline jobject GlobalReferenceAllocator::newReference(jobject original) const {
77:   internal::dbglog("Global new: %p", original);
78:   #ifdef FBJNI_DEBUG_REFS
79:     ++internal::g_reference_stats.globals_created;
80:   #endif
````
- EN: Implements callable logic such as `LocalReferenceAllocator::newReference`, `GlobalReferenceAllocator::newReference`.
- CN: 实现可调用逻辑，例如 `LocalReferenceAllocator::newReference`, `GlobalReferenceAllocator::newReference`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-120
````cpp
 81:   auto ref = Environment::current()->NewGlobalRef(original);
 82:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
 83:   return ref;
 84: }
 85: 
 86: inline void GlobalReferenceAllocator::deleteReference(jobject reference) const noexcept {
 87:   internal::dbglog("Global release: %p", reference);
 88: 
 89:   if (reference) {
 90:     #ifdef FBJNI_DEBUG_REFS
 91:       ++internal::g_reference_stats.globals_deleted;
 92:     #endif
 93:     assert(verifyReference(reference));
 94:     Environment::current()->DeleteGlobalRef(reference);
 95:   }
 96: }
 97: 
 98: inline bool GlobalReferenceAllocator::verifyReference(jobject reference) const noexcept {
 99:   return isObjectRefType(reference, JNIGlobalRefType);
100: }
101: 
102: 
103: // WeakGlobalReferenceAllocator ////////////////////////////////////////////////////////////////////
104: 
105: inline jobject WeakGlobalReferenceAllocator::newReference(jobject original) const {
106:   internal::dbglog("Weak global new: %p", original);
107:   #ifdef FBJNI_DEBUG_REFS
108:     ++internal::g_reference_stats.weaks_created;
109:   #endif
110:   auto ref = Environment::current()->NewWeakGlobalRef(original);
111:   FACEBOOK_JNI_THROW_PENDING_EXCEPTION();
112:   return ref;
113: }
114: 
115: inline void WeakGlobalReferenceAllocator::deleteReference(jobject reference) const noexcept {
116:   internal::dbglog("Weak Global release: %p", reference);
117: 
118:   if (reference) {
119:     #ifdef FBJNI_DEBUG_REFS
120:       ++internal::g_reference_stats.weaks_deleted;
````
- EN: Implements callable logic such as `WeakGlobalReferenceAllocator::newReference`.
- CN: 实现可调用逻辑，例如 `WeakGlobalReferenceAllocator::newReference`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-131
````cpp
121:     #endif
122:     assert(verifyReference(reference));
123:     Environment::current()->DeleteWeakGlobalRef(reference);
124:   }
125: }
126: 
127: inline bool WeakGlobalReferenceAllocator::verifyReference(jobject reference) const noexcept {
128:   return isObjectRefType(reference, JNIWeakGlobalRefType);
129: }
130: 
131: }}
````
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `ReferenceStats` / 符号 `ReferenceStats`
- Symbol `LocalReferenceAllocator::newReference` / 符号 `LocalReferenceAllocator::newReference`
- Symbol `GlobalReferenceAllocator::newReference` / 符号 `GlobalReferenceAllocator::newReference`
- Symbol `WeakGlobalReferenceAllocator::newReference` / 符号 `WeakGlobalReferenceAllocator::newReference`

## Dependencies / 依赖关系
- C/C++ includes: `cassert`, `new`, `atomic`, `fbjni/detail/FbjniApi.h`, `Environment.h`
- C/C++ 头文件: `cassert`, `new`, `atomic`, `fbjni/detail/FbjniApi.h`, `Environment.h`
