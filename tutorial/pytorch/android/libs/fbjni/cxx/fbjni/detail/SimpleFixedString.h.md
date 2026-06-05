# SimpleFixedString.h — Code Analysis / 代码分析

## Source / 来源
- File: `android/libs/fbjni/cxx/fbjni/detail/SimpleFixedString.h`
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
17: // Simplified fork of folly::FixedString to avoid folly dependency.
18: 
19: #pragma once
20: 
21: #include <cassert>
22: #include <cstring>
23: #include <stdexcept>
24: #include <string>
25: #include <utility>
26: 
27: #if defined(__has_feature)
28: #define FBJNI_HAS_FEATURE(...) __has_feature(__VA_ARGS__)
29: #else
30: #define FBJNI_HAS_FEATURE(...) 0
31: #endif
32: 
33: namespace facebook {
34: namespace jni {
35: 
36: namespace detail {
37: 
38: template <std::size_t N>
39: class SimpleFixedString;
40: 
41: constexpr size_t constexpr_strlen_internal(const char* s, size_t len) {
42:   // clang-format off
43:   return
44:       *(s + 0) == char(0) ? len + 0 :
45:       *(s + 1) == char(0) ? len + 1 :
46:       *(s + 2) == char(0) ? len + 2 :
47:       *(s + 3) == char(0) ? len + 3 :
48:       *(s + 4) == char(0) ? len + 4 :
49:       *(s + 5) == char(0) ? len + 5 :
50:       *(s + 6) == char(0) ? len + 6 :
51:       *(s + 7) == char(0) ? len + 7 :
52:       constexpr_strlen_internal(s + 8, len + 8);
53:   // clang-format on
54: }
55: static_assert(
56:     constexpr_strlen_internal("123456789", 0) == 9,
57:     "Someone appears to have broken constexpr_strlen...");
58: 
59: constexpr size_t constexpr_strlen(const char* s) {
60: #if FBJNI_HAS_FEATURE(cxx_constexpr_string_builtins)
````
- EN: Pulls in native headers such as `cassert`, `cstring`, `stdexcept`, `string`.
- CN: 引入原生头文件，例如 `cassert`, `cstring`, `stdexcept`, `string`。
- EN: Declares or extends types including `SimpleFixedString`.
- CN: 声明或扩展类型，包括 `SimpleFixedString`。
- EN: Implements callable logic such as `constexpr_strlen_internal`, `constexpr_strlen`.
- CN: 实现可调用逻辑，例如 `constexpr_strlen_internal`, `constexpr_strlen`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 61-120
````cpp
 61:   // clang provides a constexpr builtin
 62:   return __builtin_strlen(s);
 63: #elif defined(__GNUC__) && !defined(__clang__)
 64:   // strlen() happens to already be constexpr under gcc
 65:   return std::strlen(s);
 66: #else
 67:   return detail::constexpr_strlen_internal(s, 0);
 68: #endif
 69: }
 70: 
 71: namespace fixedstring {
 72: 
 73: // Intentionally NOT constexpr. By making this not constexpr, we make
 74: // checkOverflow below ill-formed in a constexpr context when the condition
 75: // it's testing for fails. In this way, precondition violations are reported
 76: // at compile-time instead of at runtime.
 77: [[noreturn]] inline void assertOutOfBounds() {
 78:   assert(!"Array index out of bounds in SimpleFixedString");
 79:   throw std::out_of_range(
 80:       "Array index out of bounds in SimpleFixedString");
 81: }
 82: 
 83: constexpr std::size_t checkOverflow(std::size_t i, std::size_t max) {
 84:   return i <= max ? i : (void(assertOutOfBounds()), max);
 85: }
 86: 
 87: constexpr std::size_t checkOverflowOrNpos(std::size_t i, std::size_t max) {
 88:   return i == static_cast<std::size_t>(-1)
 89:       ? max
 90:       : (i <= max ? i : (void(assertOutOfBounds()), max));
 91: }
 92: 
 93: // Intentionally NOT constexpr. See note above for assertOutOfBounds
 94: [[noreturn]] inline void assertNotNullTerminated() noexcept {
 95:   assert(!"Non-null terminated string used to initialize a SimpleFixedString");
 96:   std::terminate(); // Fail hard, fail fast.
 97: }
 98: 
 99: // Parsing help for human readers: the following is a constexpr noexcept
100: // function that accepts a reference to an array as a parameter and returns
101: // a reference to the same array.
102: template <std::size_t N>
103: constexpr const char (&checkNullTerminated(const char (&a)[N]) noexcept)[N] {
104:   // Strange decltype(a)(a) used to make MSVC happy.
105:   if (a[N - 1u] == '\0'
106: #ifndef NDEBUG
107:           // In Debug mode, guard against embedded nulls:
108:           && N - 1u == detail::constexpr_strlen_internal(a, 0u)
109: #endif
110:       ) {
111:     return decltype(a)(a);
112:   } else {
113:     assertNotNullTerminated();
114:     return decltype(a)(a);
115:   }
116: }
117: 
118: struct Helper {
119:   template <class Left, class Right, std::size_t... Is>
120:   static constexpr SimpleFixedString<sizeof...(Is)> concat_(
````
- EN: Declares or extends types including `Helper`, `Left`, `Right`.
- CN: 声明或扩展类型，包括 `Helper`, `Left`, `Right`。
- EN: Implements callable logic such as `assertOutOfBounds`, `checkOverflow`, `checkOverflowOrNpos`, `decltype`.
- CN: 实现可调用逻辑，例如 `assertOutOfBounds`, `checkOverflow`, `checkOverflowOrNpos`, `decltype`。
- EN: Includes validation, assertions, or defensive checks.
- CN: 包含校验、断言或防御性检查。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 121-180
````cpp
121:       const Left& left,
122:       std::size_t left_count,
123:       const Right& right,
124:       std::size_t right_count,
125:       std::index_sequence<Is...> is) noexcept {
126:     return {left, left_count, right, right_count, is};
127:   }
128: 
129:   template <std::size_t N>
130:   static constexpr const char (
131:       &data_(const SimpleFixedString<N>& that) noexcept)[N + 1u] {
132:     return that.data_;
133:   }
134: };
135: 
136: template <class Left, class Right>
137: constexpr int compare_(
138:     const Left& left,
139:     std::size_t left_pos,
140:     std::size_t left_size,
141:     const Right& right,
142:     std::size_t right_pos,
143:     std::size_t right_size) noexcept {
144:   return left_pos == left_size
145:       ? (right_pos == right_size ? 0 : -1)
146:       : (right_pos == right_size ? 1
147:                                  : (left[left_pos] < right[right_pos]
148:                                         ? -1
149:                                         : (left[left_pos] > right[right_pos]
150:                                                ? 1
151:                                                : fixedstring::compare_(
152:                                                      left,
153:                                                      left_pos + 1u,
154:                                                      left_size,
155:                                                      right,
156:                                                      right_pos + 1u,
157:                                                      right_size))));
158: }
159: 
160: template <class Left, class Right>
161: constexpr bool equal_(
162:     const Left& left,
163:     std::size_t left_size,
164:     const Right& right,
165:     std::size_t right_size) noexcept {
166:   return left_size == right_size &&
167:       0 == compare_(left, 0u, left_size, right, 0u, right_size);
168: }
169: 
170: template <class Left, class Right>
171: constexpr char char_at_(
172:     const Left& left,
173:     std::size_t left_count,
174:     const Right& right,
175:     std::size_t right_count,
176:     std::size_t i) noexcept {
177:   return i < left_count
178:       ? left[i]
179:       : i < (left_count + right_count) ? right[i - left_count] : '\0';
180: }
````
- EN: Declares or extends types including `Left`, `Right`.
- CN: 声明或扩展类型，包括 `Left`, `Right`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 181-240
````cpp
181: 
182: } // namespace fixedstring
183: 
184: template <std::size_t N>
185: class SimpleFixedString {
186:  private:
187:   template <std::size_t>
188:   friend class SimpleFixedString;
189:   friend struct detail::fixedstring::Helper;
190: 
191:   char data_[N + 1u]; // +1 for the null terminator
192:   std::size_t size_; // Nbr of chars, not incl. null teminator. size_ <= N
193: 
194:   using Indices = std::make_index_sequence<N>;
195: 
196:   template <class That, std::size_t... Is>
197:   constexpr SimpleFixedString(
198:     const That& that,
199:     std::size_t size,
200:     std::index_sequence<Is...>,
201:     std::size_t pos = 0,
202:     std::size_t count = static_cast<std::size_t>(-1)) noexcept
203:       : data_{(Is < (size - pos) && Is < count ? that[Is + pos] : '\0')..., '\0'},
204:         size_{(size - pos) < count ? (size - pos) : count} {}
205: 
206:   // Concatenation constructor
207:   template <class Left, class Right, std::size_t... Is>
208:   constexpr SimpleFixedString(
209:       const Left& left,
210:       std::size_t left_size,
211:       const Right& right,
212:       std::size_t right_size,
213:       std::index_sequence<Is...>) noexcept
214:       : data_{detail::fixedstring::char_at_(
215:                   left,
216:                   left_size,
217:                   right,
218:                   right_size,
219:                   Is)...,
220:               '\0'},
221:         size_{left_size + right_size} {}
222: 
223:  public:
224:   constexpr SimpleFixedString() : data_{}, size_{} {}
225: 
226:   constexpr SimpleFixedString(const SimpleFixedString&) = default;
227: 
228:   template <std::size_t M>
229:   constexpr SimpleFixedString(const SimpleFixedString<M>& that) noexcept(M <= N)
230:       : SimpleFixedString{that, 0u, that.size_} {}
231: 
232:   // Support substr.
233:   template <std::size_t M>
234:   constexpr SimpleFixedString(const SimpleFixedString<M>& that, std::size_t pos, std::size_t count) noexcept(false)
235:       : SimpleFixedString{
236:             that.data_,
237:             that.size_,
238:             std::make_index_sequence<(M < N ? M : N)>{},
239:             pos,
240:             detail::fixedstring::checkOverflow(
````
- EN: Declares or extends types including `SimpleFixedString`, `detail`, `That`, `Left`.
- CN: 声明或扩展类型，包括 `SimpleFixedString`, `detail`, `That`, `Left`。
- EN: Uses control flow to branch over configuration, data, or execution cases.
- CN: 使用控制流在配置、数据或执行场景之间分支。

### Lines 241-300
````cpp
241:                 detail::fixedstring::checkOverflowOrNpos(
242:                     count,
243:                     that.size_ -
244:                         detail::fixedstring::checkOverflow(pos, that.size_)),
245:                 N)} {}
246: 
247:   // Construct from literal.
248:   template <std::size_t M, class = typename std::enable_if<(M - 1u <= N)>::type>
249:   constexpr SimpleFixedString(const char (&literal)[M]) noexcept
250:       : SimpleFixedString{detail::fixedstring::checkNullTerminated(literal),
251:                           M - 1u,
252:                           std::make_index_sequence<M - 1u>{}} {}
253: 
254:   constexpr SimpleFixedString(const char *str, std::size_t count) noexcept(false)
255:       : SimpleFixedString{str, detail::fixedstring::checkOverflow(count, N),
256:                           Indices{}} {}
257: 
258:   constexpr const char* c_str() const noexcept {
259:     return data_;
260:   }
261: 
262:   constexpr std::size_t size() const noexcept {
263:     return size_;
264:   }
265: 
266:   constexpr SimpleFixedString substr(std::size_t pos, std::size_t count) const {
267:     return {*this, pos, count};
268:   }
269: 
270:   /** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** **
271:    * Asymmetric relational operators
272:    */
273:   friend constexpr bool operator==(
274:       const char* a,
275:       const SimpleFixedString& b) noexcept {
276:     return detail::fixedstring::equal_(
277:         a, constexpr_strlen(a), b.data_, b.size_);
278:   }
279: 
280:   /**
281:    * \overload
282:    */
283:   friend constexpr bool operator==(
284:       const SimpleFixedString& a,
285:       const char* b) noexcept {
286:     return b == a;
287:   }
288: 
289:   /** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** ** **
290:    * Asymmetric concatenation
291:    */
292:   template <std::size_t M>
293:   friend constexpr SimpleFixedString<N + M - 1u> operator+(
294:       const char (&a)[M],
295:       const SimpleFixedString& b) noexcept {
296:     return detail::fixedstring::Helper::concat_(
297:       detail::fixedstring::checkNullTerminated(a),
298:       M-1u,
299:       b.data_,
300:       b.size_,
````
- EN: Implements callable logic such as `substr`.
- CN: 实现可调用逻辑，例如 `substr`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

### Lines 301-349
````cpp
301:       std::make_index_sequence<N + M - 1u>{});
302:   }
303: 
304:   template <std::size_t M>
305:   friend constexpr SimpleFixedString<N + M - 1u> operator+(
306:       const SimpleFixedString& a,
307:       const char (&b)[M]) noexcept {
308:     return detail::fixedstring::Helper::concat_(
309:       a.data_,
310:       a.size_,
311:       detail::fixedstring::checkNullTerminated(b),
312:       M - 1u,
313:       std::make_index_sequence<N + M - 1u>{});
314:   }
315: 
316:   constexpr const char* begin() const noexcept {
317:     return data_;
318:   }
319: 
320:   constexpr const char* end() const noexcept {
321:     return data_ + size_;
322:   }
323: 
324:   operator std::string() const noexcept(false) {
325:     return std::string(begin(), end());
326:   }
327: };
328: 
329: template <std::size_t N, std::size_t M>
330: constexpr SimpleFixedString<N + M> operator+(
331:     const SimpleFixedString<N>& a,
332:     const SimpleFixedString<M>& b) noexcept {
333:   return detail::fixedstring::Helper::concat_(
334:       detail::fixedstring::Helper::data_(a),
335:       a.size(),
336:       detail::fixedstring::Helper::data_(b),
337:       b.size(),
338:       std::make_index_sequence<N + M>{});
339: }
340: 
341: template <std::size_t N>
342: constexpr SimpleFixedString<N - 1u> makeSimpleFixedString(
343:     const char (&a)[N]) noexcept {
344:   return {a};
345: }
346: 
347: } // namespace detail
348: } // namespace jni
349: } // namespace facebook
````
- EN: Implements callable logic such as `std::string`.
- CN: 实现可调用逻辑，例如 `std::string`。
- EN: Produces values or forwards results to the surrounding call chain.
- CN: 生成返回值，或将结果继续传递给外围调用链。

## Key Concepts / 关键概念
- Android integration / Android 集成
- JNI bridge layer / JNI 桥接层
- Symbol `SimpleFixedString` / 符号 `SimpleFixedString`
- Symbol `Helper` / 符号 `Helper`
- Symbol `Left` / 符号 `Left`
- Symbol `Right` / 符号 `Right`

## Dependencies / 依赖关系
- C/C++ includes: `cassert`, `cstring`, `stdexcept`, `string`, `utility`
- C/C++ 头文件: `cassert`, `cstring`, `stdexcept`, `string`, `utility`
