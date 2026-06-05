# utf8.cpp — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/utf8.cpp`
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
17: #include "utf8.h"
18: 
19: #include "Log.h"
20: 
21: namespace facebook {
22: namespace jni {
23: 
24: namespace {
25: 
26: const uint16_t kUtf8OneByteBoundary       = 0x80;
27: const uint16_t kUtf8TwoBytesBoundary      = 0x800;
28: const uint16_t kUtf16HighSubLowBoundary   = 0xD800;
29: const uint16_t kUtf16HighSubHighBoundary  = 0xDC00;
30: const uint16_t kUtf16LowSubHighBoundary   = 0xE000;
31: 
32: inline void encode3ByteUTF8(char32_t code, uint8_t* out) {
33:   if ((code & 0xffff0000) != 0) {
34:     FBJNI_LOGF("3 byte utf-8 encodings only valid for up to 16 bits");
35:   }
36: 
37:   out[0] = 0xE0 | (code >> 12);
38:   out[1] = 0x80 | ((code >> 6) & 0x3F);
39:   out[2] = 0x80 | (code & 0x3F);
40: }
41: 
42: inline char32_t decode3ByteUTF8(const uint8_t* in) {
43:   return (((in[0] & 0x0f) << 12) |
44:           ((in[1] & 0x3f) << 6) |
45:           ( in[2] & 0x3f));
46: }
47: 
48: inline void encode4ByteUTF8(char32_t code, std::string& out, size_t offset) {
49:   if ((code & 0xfff80000) != 0) {
50:     FBJNI_LOGF("4 byte utf-8 encodings only valid for up to 21 bits");
51:   }
52: 
53:   out[offset] =     (char) (0xF0 | (code >> 18));
54:   out[offset + 1] = (char) (0x80 | ((code >> 12) & 0x3F));
55:   out[offset + 2] = (char) (0x80 | ((code >> 6) & 0x3F));
56:   out[offset + 3] = (char) (0x80 | (code & 0x3F));
57: }
58: 
59: template <typename T>
60: inline bool isFourByteUTF8Encoding(const T* utf8) {
````
- EN: Pulls in native headers such as `utf8.h`, `Log.h`.
- CN: 引入原生头文件，例如 `utf8.h`, `Log.h`。
- EN: Implements callable logic such as `encode3ByteUTF8`, `decode3ByteUTF8`, `encode4ByteUTF8`, `isFourByteUTF8Encoding`.
- CN: 实现可调用逻辑，例如 `encode3ByteUTF8`, `decode3ByteUTF8`, `encode4ByteUTF8`, `isFourByteUTF8Encoding`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 61-120
````cpp
 61:   return ((*utf8 & 0xF8) == 0xF0);
 62: }
 63: 
 64: }
 65: 
 66: namespace detail {
 67: 
 68: size_t modifiedLength(const std::string& str) {
 69:   // Scan for supplementary characters
 70:   size_t j = 0;
 71:   for (size_t i = 0; i < str.size(); ) {
 72:     if (str[i] == 0) {
 73:       i += 1;
 74:       j += 2;
 75:     } else if (i + 4 > str.size() ||
 76:                !isFourByteUTF8Encoding(&(str[i]))) {
 77:       // See the code in utf8ToModifiedUTF8 for what's happening here.
 78:       i += 1;
 79:       j += 1;
 80:     } else {
 81:       i += 4;
 82:       j += 6;
 83:     }
 84:   }
 85: 
 86:   return j;
 87: }
 88: 
 89: // returns modified utf8 length; *length is set to strlen(str)
 90: size_t modifiedLength(const uint8_t* str, size_t* length) {
 91:   // NUL-terminated: Scan for length and supplementary characters
 92:   size_t i = 0;
 93:   size_t j = 0;
 94:   while (str[i] != 0) {
 95:     if (str[i + 1] == 0 ||
 96:         str[i + 2] == 0 ||
 97:         str[i + 3] == 0 ||
 98:         !isFourByteUTF8Encoding(&(str[i]))) {
 99:       i += 1;
100:       j += 1;
101:     } else {
102:       i += 4;
103:       j += 6;
104:     }
105:   }
106: 
107:   *length = i;
108:   return j;
109: }
110: 
111: void utf8ToModifiedUTF8(const uint8_t* utf8, size_t len, uint8_t* modified, size_t modifiedBufLen)
112: {
113:   size_t j = 0;
114:   for (size_t i = 0; i < len; ) {
115:     if (j >= modifiedBufLen) {
116:       FBJNI_LOGF("output buffer is too short");
117:     }
118:     if (utf8[i] == 0) {
119:       if (j + 1 >= modifiedBufLen) {
120:         FBJNI_LOGF("output buffer is too short");
````
- EN: Implements callable logic such as `modifiedLength`, `strlen`, `utf8ToModifiedUTF8`.
- CN: 实现可调用逻辑，例如 `modifiedLength`, `strlen`, `utf8ToModifiedUTF8`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 121-180
````cpp
121:       }
122:       modified[j] = 0xc0;
123:       modified[j + 1] = 0x80;
124:       i += 1;
125:       j += 2;
126:       continue;
127:     }
128: 
129:     if (i + 4 > len ||
130:         !isFourByteUTF8Encoding(utf8 + i)) {
131:       // If the input is too short for this to be a four-byte
132:       // encoding, or it isn't one for real, just copy it on through.
133:       modified[j] = utf8[i];
134:       i++;
135:       j++;
136:       continue;
137:     }
138: 
139:     // Convert 4 bytes of input to 2 * 3 bytes of output
140:     char32_t code = (((utf8[i]     & 0x07) << 18) |
141:                      ((utf8[i + 1] & 0x3f) << 12) |
142:                      ((utf8[i + 2] & 0x3f) << 6) |
143:                      ( utf8[i + 3] & 0x3f));
144:     char32_t first;
145:     char32_t second;
146: 
147:     if (code > 0x10ffff) {
148:       // These could be valid utf-8, but cannot be represented as modified UTF-8, due to the 20-bit
149:       // limit on that representation.  Encode two replacement characters, so the expected output
150:       // length lines up.
151:       const char32_t kUnicodeReplacementChar = 0xfffd;
152:       first = kUnicodeReplacementChar;
153:       second = kUnicodeReplacementChar;
154:     } else {
155:       // split into surrogate pair
156:       first = ((code - 0x010000) >> 10) | 0xd800;
157:       second = ((code - 0x010000) & 0x3ff) | 0xdc00;
158:     }
159: 
160:     // encode each as a 3 byte surrogate value
161:     if (j + 5 >= modifiedBufLen) {
162:       FBJNI_LOGF("output buffer is too short");
163:     }
164:     encode3ByteUTF8(first, modified + j);
165:     encode3ByteUTF8(second, modified + j + 3);
166:     i += 4;
167:     j += 6;
168:   }
169: 
170:   if (j >= modifiedBufLen) {
171:     FBJNI_LOGF("output buffer is too short");
172:   }
173:   modified[j++] = '\0';
174: }
175: 
176: std::string modifiedUTF8ToUTF8(const uint8_t* modified, size_t len) noexcept {
177:   // Converting from modified utf8 to utf8 will always shrink, so this will always be sufficient
178:   std::string utf8(len, 0);
179:   size_t j = 0;
180:   for (size_t i = 0; i < len; ) {
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 181-240
````cpp
181:     // surrogate pair: 1101 10xx  xxxx xxxx  1101 11xx  xxxx xxxx
182:     // encoded pair: 1110 1101  1010 xxxx  10xx xxxx  1110 1101  1011 xxxx  10xx xxxx
183: 
184:     if (len >= i + 6 &&
185:         modified[i] == 0xed &&
186:         (modified[i + 1] & 0xf0) == 0xa0 &&
187:         modified[i + 3] == 0xed &&
188:         (modified[i + 4] & 0xf0) == 0xb0) {
189:       // Valid surrogate pair
190:       char32_t pair1 = decode3ByteUTF8(modified + i);
191:       char32_t pair2 = decode3ByteUTF8(modified + i + 3);
192:       char32_t ch = 0x10000 + (((pair1 & 0x3ff) << 10) |
193:                                ( pair2 & 0x3ff));
194:       encode4ByteUTF8(ch, utf8, j);
195:       i += 6;
196:       j += 4;
197:       continue;
198:     } else if (len >= i + 2 &&
199:                modified[i] == 0xc0 &&
200:                modified[i + 1] == 0x80) {
201:       utf8[j] = 0;
202:       i += 2;
203:       j += 1;
204:       continue;
205:     }
206: 
207:     // copy one byte.  This might be a one, two, or three-byte encoding.  It might be an invalid
208:     // encoding of some sort, but garbage in garbage out is ok.
209: 
210:     utf8[j] = (char) modified[i];
211:     i++;
212:     j++;
213:   }
214: 
215:   utf8.resize(j);
216: 
217:   return utf8;
218: }
219: 
220: // Calculate how many bytes are needed to convert an UTF16 string into UTF8
221: // UTF16 string
222: size_t utf16toUTF8Length(const uint16_t* utf16String, size_t utf16StringLen) {
223:   if (!utf16String || utf16StringLen == 0) {
224:     return 0;
225:   }
226: 
227:   uint32_t utf8StringLen = 0;
228:   auto utf16StringEnd = utf16String + utf16StringLen;
229:   auto idx16 = utf16String;
230:   while (idx16 < utf16StringEnd) {
231:     auto ch = *idx16++;
232:     if (ch < kUtf8OneByteBoundary) {
233:       utf8StringLen++;
234:     } else if (ch < kUtf8TwoBytesBoundary) {
235:       utf8StringLen += 2;
236:     } else if (
237:         (ch >= kUtf16HighSubLowBoundary) && (ch < kUtf16HighSubHighBoundary) &&
238:         (idx16 < utf16StringEnd) &&
239:         (*idx16 >= kUtf16HighSubHighBoundary) && (*idx16 < kUtf16LowSubHighBoundary)) {
240:       utf8StringLen += 4;
````
- EN: Implements callable logic such as `utf16toUTF8Length`.
- CN: 实现可调用逻辑，例如 `utf16toUTF8Length`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 241-288
````cpp
241:       idx16++;
242:     } else {
243:       utf8StringLen += 3;
244:     }
245:   }
246: 
247:   return utf8StringLen;
248: }
249: 
250: std::string utf16toUTF8(const uint16_t* utf16String, size_t utf16StringLen) noexcept {
251:   if (!utf16String || utf16StringLen <= 0) {
252:     return "";
253:   }
254: 
255:   std::string utf8String(utf16toUTF8Length(utf16String, utf16StringLen), '\0');
256:   auto idx8 = utf8String.begin();
257:   auto idx16 = utf16String;
258:   auto utf16StringEnd = utf16String + utf16StringLen;
259:   while (idx16 < utf16StringEnd) {
260:     auto ch = *idx16++;
261:     if (ch < kUtf8OneByteBoundary) {
262:       *idx8++ = (ch & 0x7F);
263:     } else if (ch < kUtf8TwoBytesBoundary) {
264:       *idx8++ = 0b11000000 | (ch >> 6);
265:       *idx8++ = 0b10000000 | (ch & 0x3F);
266:     } else if (
267:         (ch >= kUtf16HighSubLowBoundary) && (ch < kUtf16HighSubHighBoundary) &&
268:         (idx16 < utf16StringEnd) &&
269:         (*idx16 >= kUtf16HighSubHighBoundary) && (*idx16 < kUtf16LowSubHighBoundary)) {
270:       auto ch2 = *idx16++;
271:       uint8_t trunc_byte = (((ch >> 6) & 0x0F) + 1);
272:       *idx8++ = 0b11110000 | (trunc_byte >> 2);
273:       *idx8++ = 0b10000000 | ((trunc_byte & 0x03) << 4) | ((ch >> 2) & 0x0F);
274:       *idx8++ = 0b10000000 | ((ch & 0x03) << 4) | ((ch2 >> 6) & 0x0F);
275:       *idx8++ = 0b10000000 | (ch2 & 0x3F);
276:     } else {
277:       *idx8++ = 0b11100000 | (ch >> 12);
278:       *idx8++ = 0b10000000 | ((ch >> 6) & 0x3F);
279:       *idx8++ = 0b10000000 | (ch & 0x3F);
280:     }
281:   }
282: 
283:   return utf8String;
284: }
285: 
286: }
287: }
288: }
````
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `encode3ByteUTF8` / 符号 `encode3ByteUTF8`
- Symbol `decode3ByteUTF8` / 符号 `decode3ByteUTF8`
- Symbol `encode4ByteUTF8` / 符号 `encode4ByteUTF8`
- Symbol `isFourByteUTF8Encoding` / 符号 `isFourByteUTF8Encoding`

## Dependencies / 依赖关系
- C/C++ includes: `utf8.h`, `Log.h`
- C/C++ 头文件: `utf8.h`, `Log.h`
