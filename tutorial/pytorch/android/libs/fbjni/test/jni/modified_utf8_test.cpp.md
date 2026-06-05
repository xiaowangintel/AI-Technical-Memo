# modified_utf8_test.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/test/jni/modified_utf8_test.cpp`
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
17: #include <gtest/gtest.h>
18: 
19: #include <fbjni/detail/utf8.h>
20: 
21: #include <vector>
22: 
23: using namespace std;
24: using namespace facebook::jni;
25: 
26: void testPair(const vector<uint8_t>& utf8, const vector<uint8_t>& modified, bool test_out_eq) {
27:   // utf8 -> modified utf8
28: 
29:   string utf8str(reinterpret_cast<const char*>(utf8.data()), utf8.size());
30: 
31:   size_t modlen = detail::modifiedLength(utf8str);
32:   EXPECT_EQ(modlen, modified.size());
33:   vector<uint8_t> out(modlen + 1);
34:   detail::utf8ToModifiedUTF8(utf8.data(), utf8.size(), out.data(), out.size());
35:   // we expect utf8ToModified to return null terminated string, but vector in modified
36:   // will not have \0 at the end, therefore we crop out buffer before comparing
37:   EXPECT_EQ('\0', out[modlen]);
38:   out.resize(modlen);
39:   if (test_out_eq) {
40:     EXPECT_EQ(out, modified);
````
- EN: Pulls in native headers such as `gtest/gtest.h`, `fbjni/detail/utf8.h`, `vector`.
- CN: 引入原生头文件，例如 `gtest/gtest.h`, `fbjni/detail/utf8.h`, `vector`。
- EN: Implements callable logic such as `testPair`.
- CN: 实现可调用逻辑，例如 `testPair`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 41-80
````cpp
41:   }
42: 
43:   // modified utf8 -> utf8
44: 
45:   string s = detail::modifiedUTF8ToUTF8(modified.data(), modified.size());
46:   if (test_out_eq) {
47:     EXPECT_EQ(s, utf8str);
48:   }
49: }
50: 
51: void testFailForTooShortBuffer(const vector<uint8_t>& utf8, int out_len) {
52:   const char* message =
53: #ifdef __ANDROID__
54:     "output buffer is too short";
55: #else
56:     "";
57: #endif
58: 
59:   ASSERT_DEATH({
60:       vector<uint8_t> out(out_len);
61:       detail::utf8ToModifiedUTF8(utf8.data(), utf8.size(), out.data(), out.size());
62:     }, message);
63: }
64: 
65: void vector_append(vector<uint8_t>& target, const vector<uint8_t>& source) {
66:   target.insert(target.end(), source.begin(), source.end());
67: }
68: 
69: TEST(ModifiedUTF8Test, pairs) {
70:   vector<uint8_t> zero = { 0 }; // U+0000
71:   vector<uint8_t> zero_modified = { 0xc0, 0x80 };
72:   vector<uint8_t> one_byte = { 'a' };  // U+0041
73:   vector<uint8_t> two_byte = { 0xd8, 0xa1 };  // U+00E1  small a with acute
74:   vector<uint8_t> three_byte = { 0xe4, 0xba, 0xba };  // U+4EBA  unihan ren
75:   vector<uint8_t> four_byte =
76:     { 0xf0, 0x9f, 0x98, 0xb8 };  // U+1F638  grinning cat face with smiling eyes
77:   vector<uint8_t> four_byte_modified = { 0xed, 0xa0, 0xbd, 0xed, 0xb8, 0xb8 };
78:   vector<uint8_t> four_byte_truncated(four_byte.begin(),
79:                                       four_byte.begin() + (four_byte.size() - 1));
80:   vector<uint8_t> modified_truncated(
````
- EN: Implements callable logic such as `testFailForTooShortBuffer`, `vector_append`, `TEST`.
- CN: 实现可调用逻辑，例如 `testFailForTooShortBuffer`, `vector_append`, `TEST`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 81-120
````cpp
 81:     four_byte_modified.begin(),
 82:     four_byte_modified.begin() + (four_byte_modified.size() - 1));
 83: 
 84:   testPair(zero, zero_modified, true);
 85:   testPair(one_byte, one_byte, true);
 86:   testPair(two_byte, two_byte, true);
 87:   testPair(three_byte, three_byte, true);
 88:   testPair(four_byte, four_byte_modified, true);
 89: 
 90:   // The output is not strictly defined, but should not crash.
 91:   testPair(four_byte_truncated, four_byte_truncated, false);
 92:   testPair(modified_truncated, modified_truncated, false);
 93: 
 94:   vector<uint8_t> all;
 95:   vector_append(all, zero);
 96:   vector_append(all, one_byte);
 97:   vector_append(all, two_byte);
 98:   vector_append(all, three_byte);
 99:   vector_append(all, four_byte);
100:   vector_append(all, zero);
101:   vector_append(all, one_byte);
102:   vector_append(all, two_byte);
103:   vector_append(all, three_byte);
104:   vector_append(all, four_byte);
105: 
106:   vector<uint8_t> all_modified;
107:   vector_append(all_modified, zero_modified);
108:   vector_append(all_modified, one_byte);
109:   vector_append(all_modified, two_byte);
110:   vector_append(all_modified, three_byte);
111:   vector_append(all_modified, four_byte_modified);
112:   vector_append(all_modified, zero_modified);
113:   vector_append(all_modified, one_byte);
114:   vector_append(all_modified, two_byte);
115:   vector_append(all_modified, three_byte);
116:   vector_append(all_modified, four_byte_modified);
117: 
118:   testPair(all, all_modified, true);
119: }
120: 
````
- EN: This range contributes implementation details for the file goal: Implements JNI bridge abstractions used by Android-facing native code.
- CN: 该范围为文件目标提供实现细节：实现供 Android 原生代码使用的 JNI 桥接抽象。

### Lines 121-137
````cpp
121: TEST(ModifiedUTF8Test, conversionFailForTooShortBuffer) {
122:   vector<uint8_t> zero = { 0 };
123:   vector<uint8_t> one_byte = { 'a' };
124:   vector<uint8_t> four_byte = { 0xf0, 0x9f, 0x98, 0xb8 };
125: 
126:   testFailForTooShortBuffer(zero, 1);
127:   testFailForTooShortBuffer(zero, 2);
128:   testFailForTooShortBuffer(one_byte, 1);
129:   testFailForTooShortBuffer(four_byte, 2);
130:   testFailForTooShortBuffer(four_byte, 4);
131: }
132: 
133: int main(int argc, char **argv) {
134:   testing::InitGoogleTest(&argc, argv);
135: 
136:   return RUN_ALL_TESTS();
137: }
````
- EN: Implements callable logic such as `TEST`, `main`.
- CN: 实现可调用逻辑，例如 `TEST`, `main`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Validation and test coverage / 校验与测试覆盖
- Symbol `testPair` / 符号 `testPair`
- Symbol `testFailForTooShortBuffer` / 符号 `testFailForTooShortBuffer`
- Symbol `vector_append` / 符号 `vector_append`
- Symbol `TEST` / 符号 `TEST`

## Dependencies / 依赖关系
- C/C++ includes: `gtest/gtest.h`, `fbjni/detail/utf8.h`, `vector`
- C/C++ 头文件: `gtest/gtest.h`, `fbjni/detail/utf8.h`, `vector`
