# utf8.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/utf8.h`
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
19: #include <string>
20: 
21: #include <jni.h>
22: 
23: namespace facebook {
24: namespace jni {
25: 
26: namespace detail {
27: 
28: void utf8ToModifiedUTF8(const uint8_t* bytes, size_t len, uint8_t* modified, size_t modifiedLength);
29: size_t modifiedLength(const std::string& str);
30: size_t modifiedLength(const uint8_t* str, size_t* length);
31: std::string modifiedUTF8ToUTF8(const uint8_t* modified, size_t len) noexcept;
32: std::string utf16toUTF8(const uint16_t* utf16Bytes, size_t len) noexcept;
33: 
34: }
35: 
36: // JNI represents strings encoded with modified version of UTF-8.  The difference between UTF-8 and
37: // Modified UTF-8 is that the latter support only 1-byte, 2-byte, and 3-byte formats. Supplementary
38: // character (4 bytes in unicode) needs to be represented in the form of surrogate pairs. To create
39: // a Modified UTF-8 surrogate pair that Dalvik would understand we take 4-byte unicode character,
40: // encode it with UTF-16 which gives us two 2 byte chars (surrogate pair) and then we encode each
````
- EN: Pulls in native headers such as `string`, `jni.h`.
- CN: 引入原生头文件，例如 `string`, `jni.h`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41: // pair as UTF-8. This result in 2 x 3 byte characters.  To convert modified UTF-8 to standard
42: // UTF-8, this mus tbe reversed.
43: //
44: // The second difference is that Modified UTF-8 is encoding NUL byte in 2-byte format.
45: //
46: // In order to avoid complex error handling, only a minimum of validity checking is done to avoid
47: // crashing.  If the input is invalid, the output may be invalid as well.
48: //
49: // Relevant links:
50: //  - http://docs.oracle.com/javase/7/docs/technotes/guides/jni/spec/functions.html
51: //  - https://docs.oracle.com/javase/6/docs/api/java/io/DataInput.html#modified-utf-8
52: 
53: // JString to UTF16 extractor using RAII idiom. Note that the
54: // ctor/dtor use GetStringCritical/ReleaseStringCritical, so this
55: // class is subject to the restrictions imposed by those functions.
56: class JStringUtf16Extractor {
57: public:
58:   JStringUtf16Extractor(JNIEnv* env, jstring javaString)
59:   : env_(env)
60:   , javaString_(javaString)
61:   , length_(0)
62:   , utf16String_(nullptr) {
63:     if (env_ && javaString_) {
64:       length_ = env_->GetStringLength(javaString_);
65:       utf16String_ = env_->GetStringCritical(javaString_, nullptr);
66:     }
67:   }
68: 
69:   ~JStringUtf16Extractor() {
70:     if (utf16String_) {
71:       env_->ReleaseStringCritical(javaString_, utf16String_);
72:     }
73:   }
74: 
75:   jsize length() const {
76:     return length_;
77:   }
78: 
79:   const jchar* chars() const {
80:     return utf16String_;
````
- EN: Declares or extends types including `is`, `JStringUtf16Extractor`.
- CN: 声明或扩展类型，包括 `is`, `JStringUtf16Extractor`。
- EN: Implements callable logic such as `JStringUtf16Extractor`, `~JStringUtf16Extractor`, `length`, `chars`.
- CN: 实现可调用逻辑，例如 `JStringUtf16Extractor`, `~JStringUtf16Extractor`, `length`, `chars`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 81-91
````cpp
81:   }
82: 
83: private:
84:   JNIEnv* env_;
85:   jstring javaString_;
86:   jsize length_;
87:   const jchar* utf16String_;
88: };
89: 
90: }
91: }
````
- EN: This range contributes implementation details for the file goal: Implements JNI bridge abstractions used by Android-facing native code.
- CN: 该范围为文件目标提供实现细节：实现供 Android 原生代码使用的 JNI 桥接抽象。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `is` / 符号 `is`
- Symbol `JStringUtf16Extractor` / 符号 `JStringUtf16Extractor`
- Symbol `~JStringUtf16Extractor` / 符号 `~JStringUtf16Extractor`
- Symbol `length` / 符号 `length`

## Dependencies / 依赖关系
- C/C++ includes: `string`, `jni.h`
- C/C++ 头文件: `string`, `jni.h`
