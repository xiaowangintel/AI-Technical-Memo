# References.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/References.cpp`
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
17: #include "References.h"
18: 
19: namespace facebook {
20: namespace jni {
21: 
22: JniLocalScope::JniLocalScope(JNIEnv* env, jint capacity)
23:     : env_(env) {
24:   hasFrame_ = false;
25:   auto pushResult = env->PushLocalFrame(capacity);
26:   FACEBOOK_JNI_THROW_EXCEPTION_IF(pushResult < 0);
27:   hasFrame_ = true;
28: }
29: 
30: JniLocalScope::~JniLocalScope() {
31:   if (hasFrame_) {
32:     env_->PopLocalFrame(nullptr);
33:   }
34: }
35: 
36: namespace {
37: 
38: #ifdef __ANDROID__
39: 
40: int32_t getAndroidApiLevel() {
````
- EN: Pulls in native headers such as `References.h`.
- CN: 引入原生头文件，例如 `References.h`。
- EN: Implements callable logic such as `JniLocalScope::JniLocalScope`, `JniLocalScope::~JniLocalScope`, `getAndroidApiLevel`.
- CN: 实现可调用逻辑，例如 `JniLocalScope::JniLocalScope`, `JniLocalScope::~JniLocalScope`, `getAndroidApiLevel`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41:   // This is called from the static local initializer in
42:   // isObjectRefType(), and creating fbjni references can call
43:   // isObjectRefType().  So, to avoid recursively entering the block
44:   // where the static is initialized (which is undefined behavior), we
45:   // avoid using standard fbjni references here.
46: 
47:   JNIEnv* env = Environment::current();
48:   jclass cls = detail::findClass(env, "android/os/Build$VERSION");
49:   jfieldID field = env->GetStaticFieldID(cls, "SDK_INT",
50:                                          jtype_traits<jint>::kDescriptor.c_str());
51:   if (!field) {
52:     env->DeleteLocalRef(cls);
53:   }
54:   FACEBOOK_JNI_THROW_EXCEPTION_IF(!field);
55:   int32_t ret = env->GetStaticIntField(cls, field);
56:   env->DeleteLocalRef(cls);
57:   return ret;
58: }
59: 
60: bool doesGetObjectRefTypeWork() {
61:   auto level = getAndroidApiLevel();
62:   return level >= 14;
63: }
64: 
65: #else
66: 
67: bool doesGetObjectRefTypeWork() {
68:   auto jni_version = Environment::current()->GetVersion();
69:   return jni_version >= JNI_VERSION_1_6;
70: }
71: 
72: #endif
73: 
74: }
75: 
76: bool isObjectRefType(jobject reference, jobjectRefType refType) {
77:   // null-check first so that we short-circuit during (safe) global
78:   // constructors, where we won't have an Environment::current() yet
79:   if (!reference) {
80:     return true;
````
- EN: Implements callable logic such as `doesGetObjectRefTypeWork`, `isObjectRefType`, `during`.
- CN: 实现可调用逻辑，例如 `doesGetObjectRefTypeWork`, `isObjectRefType`, `during`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-91
````cpp
81:   }
82: 
83:   static bool getObjectRefTypeWorks = doesGetObjectRefTypeWork();
84: 
85:   return
86:     !getObjectRefTypeWorks ||
87:     Environment::current()->GetObjectRefType(reference) == refType;
88: }
89: 
90: }
91: }
````
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `JniLocalScope::JniLocalScope` / 符号 `JniLocalScope::JniLocalScope`
- Symbol `JniLocalScope::~JniLocalScope` / 符号 `JniLocalScope::~JniLocalScope`
- Symbol `getAndroidApiLevel` / 符号 `getAndroidApiLevel`
- Symbol `doesGetObjectRefTypeWork` / 符号 `doesGetObjectRefTypeWork`

## Dependencies / 依赖关系
- C/C++ includes: `References.h`
- C/C++ 头文件: `References.h`
