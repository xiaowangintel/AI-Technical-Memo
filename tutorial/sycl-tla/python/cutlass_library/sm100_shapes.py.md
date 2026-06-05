# sm100_shapes.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/sm100_shapes.py`
- **EN:** Stores architecture-specific SM100 shape catalogs used by higher-level generator utilities.
- **CN:** 存储供上层生成工具使用的 SM100 架构专用 shape 目录。

## Line-by-Line Analysis / 逐行分析

### L1-L32 — Header comments

```python
  1: #################################################################################################
  2: #
  3: # Copyright (c) 2025 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
  4: # SPDX-License-Identifier: BSD-3-Clause
  5: #
  6: # Redistribution and use in source and binary forms, with or without
  7: # modification, are permitted provided that the following conditions are met:
  8: #
  9: # 1. Redistributions of source code must retain the above copyright notice, this
 10: # list of conditions and the following disclaimer.
 11: #
 12: # 2. Redistributions in binary form must reproduce the above copyright notice,
 13: # this list of conditions and the following disclaimer in the documentation
 14: # and/or other materials provided with the distribution.
 15: #
 16: # 3. Neither the name of the copyright holder nor the names of its
 17: # contributors may be used to endorse or promote products derived from
 18: # this software without specific prior written permission.
 19: #
 20: # THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
 21: # AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
 22: # IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
 23: # DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
 24: # FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
 25: # DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
 26: # SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
 27: # CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
 28: # OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
 29: # OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
 30: #
 31: #################################################################################################
 32: 
```
**EN:** Contains the license banner and file-level header comments.

**CN:** 包含许可证横幅以及文件级头部注释。

### L33-L342 — Import fallback

```python
 33: """
 34: Valid tcgen05 shapes and cluster sizes for SM100, associated with levels.
 35: These shape and level pairs are defined as dicts, where keys are shapes and values are their
 36: associated levels. If the user input level for that category (tcgen05 shape, cluster
 37: size) is smaller than a shape's associated level, it will be excluded, and otherwise, included.
 38: Higher levels are therefore less likely emitted, but lower levels are more emitted more frequently.
 39: Level 0 is always emitted. 
 40: """
 41: 
 42: try:
 43:     from .library import DynamicClusterShape
 44: except:
 45:     from library import DynamicClusterShape
 46: 
 47: SM100_CLUSTER_SHAPES_1SM = {
 48:     tuple(DynamicClusterShape) : 0,
 49:     # size 1 cluster
 50:     (1, 1, 1): 1,
 51:     # size 2 cluster
 52:     (1, 2, 1): 2,
 53:     (2, 1, 1): 5,
 54:     # size 4 clusters
 55:     (2, 2, 1): 6,
 56:     (1, 4, 1): 3,
 57:     (4, 1, 1): 6,
 58:     # size 8 clusters
 59:     (2, 4, 1): 7,
 60:     (4, 2, 1): 7,
 61:     (1, 8, 1): 8,
 62:     (8, 1, 1): 8,
 63:     # size 16 cluster
 64:     (4, 4, 1): 4,
 65: }
 66: 
 67: SM100_CLUSTER_SHAPES_2SM = {
 68:     tuple(DynamicClusterShape) : 0,
 69:     # size 2 cluster
 70:     (2, 1, 1): 1,
 71:     # size 4 clusters
 72:     (2, 2, 1): 2,
 73:     (4, 1, 1): 2,
 74:     # size 8 clusters
 75:     (2, 4, 1): 3,
 76:     (4, 2, 1): 3,
 77:     (8, 1, 1): 6,
 78:     # size 16 cluster
 79:     (4, 4, 1): 4,
 80: }
 81: 
 82: # MMA shapes
 83: 
 84: # 16b Dense
 85: 
 86: SM100_MMA_SHAPES_16b_DENSE_1SM = {
 87:     (64,   8, 16): 5,
 88:     (64,  16, 16): 2,
 89:     (64,  24, 16): 5,
 90:     (64,  32, 16): 2,
 91:     (64,  40, 16): 5,
 92:     (64,  48, 16): 5,
 93:     (64,  56, 16): 5,
 94:     (64,  64, 16): 2,
 95:     (64,  72, 16): 5,
 96:     (64,  80, 16): 5,
 97:     (64,  88, 16): 5,
 98:     (64,  96, 16): 5,
 99:     (64, 104, 16): 5,
100:     (64, 112, 16): 5,
101:     (64, 120, 16): 5,
102:     (64, 128, 16): 0,
103:     (64, 136, 16): 5,
104:     (64, 144, 16): 5,
105:     (64, 152, 16): 5,
106:     (64, 160, 16): 5,
107:     (64, 168, 16): 5,
108:     (64, 176, 16): 5,
109:     (64, 184, 16): 5,
110:     (64, 192, 16): 3,
111:     (64, 200, 16): 5,
112:     (64, 208, 16): 5,
113:     (64, 216, 16): 5,
114:     (64, 224, 16): 5,
115:     (64, 232, 16): 5,
116:     (64, 240, 16): 5,
117:     (64, 248, 16): 5,
118:     (64, 256, 16): 3,
119: 
120:     (128,  16, 16): 2,
121:     (128,  32, 16): 2,
122:     (128,  48, 16): 5,
123:     (128,  64, 16): 2,
124:     (128,  80, 16): 5,
125:     (128,  96, 16): 5,
126:     (128, 112, 16): 5,
127:     (128, 128, 16): 0,
128:     (128, 144, 16): 5,
129:     (128, 160, 16): 5,
130:     (128, 176, 16): 5,
131:     (128, 192, 16): 3,
132:     (128, 208, 16): 5,
133:     (128, 224, 16): 5,
134:     (128, 240, 16): 5,
135:     (128, 256, 16): 0,
136: 
137: }
138: 
139: 
140: SM100_MMA_SHAPES_16b_DENSE_2SM = {
141:     (128,  32, 16): 2,
142:     (128,  64, 16): 2,
143:     (128,  96, 16): 5,
144:     (128, 128, 16): 0,
145:     (128, 160, 16): 5,
146:     (128, 192, 16): 5,
147:     (128, 224, 16): 5,
148:     (128, 256, 16): 0,
149: 
150:     (256,  32, 16): 2,
151:     (256,  64, 16): 2,
152:     (256,  96, 16): 5,
153:     (256, 128, 16): 0,
154:     (256, 160, 16): 5,
155:     (256, 192, 16): 3,
156:     (256, 224, 16): 5,
157:     (256, 256, 16): 0,
158: }
159: 
160: # TF32 Dense
161: 
162: SM100_MMA_SHAPES_TF32_DENSE_1SM = {
163:     (64,   8, 8): 5,
164:     (64,  16, 8): 2,
165:     (64,  24, 8): 5,
166:     (64,  32, 8): 2,
167:     (64,  40, 8): 5,
168:     (64,  48, 8): 5,
169:     (64,  56, 8): 5,
170:     (64,  64, 8): 1,
171:     (64,  72, 8): 5,
172:     (64,  80, 8): 5,
173:     (64,  88, 8): 5,
174:     (64,  96, 8): 5,
175:     (64, 104, 8): 5,
176:     (64, 112, 8): 5,
177:     (64, 120, 8): 5,
178:     (64, 128, 8): 0,
179:     (64, 136, 8): 5,
180:     (64, 144, 8): 5,
181:     (64, 152, 8): 5,
182:     (64, 160, 8): 5,
183:     (64, 168, 8): 5,
184:     (64, 176, 8): 5,
185:     (64, 184, 8): 5,
186:     (64, 192, 8): 3,
187:     (64, 200, 8): 5,
188:     (64, 208, 8): 5,
189:     (64, 216, 8): 5,
190:     (64, 224, 8): 5,
191:     (64, 232, 8): 5,
192:     (64, 240, 8): 5,
193:     (64, 248, 8): 5,
194:     (64, 256, 8): 3,
195: 
196:     (128,  16, 8): 2,
197:     (128,  32, 8): 2,
198:     (128,  48, 8): 5,
199:     (128,  64, 8): 2,
200:     (128,  80, 8): 5,
201:     (128,  96, 8): 5,
202:     (128, 112, 8): 5,
203:     (128, 128, 8): 0,
204:     (128, 144, 8): 5,
205:     (128, 160, 8): 5,
206:     (128, 176, 8): 5,
207:     (128, 192, 8): 3,
208:     (128, 208, 8): 5,
209:     (128, 224, 8): 5,
210:     (128, 240, 8): 5,
211:     (128, 256, 8): 0,
212: 
213: }
214: 
215: SM100_MMA_SHAPES_TF32_DENSE_2SM = {
216:     (128,  32, 8): 2,
217:     (128,  64, 8): 1,
218:     (128,  96, 8): 5,
219:     (128, 128, 8): 0,
220:     (128, 160, 8): 5,
221:     (128, 192, 8): 5,
222:     (128, 224, 8): 5,
223:     (128, 256, 8): 0,
224: 
225:     (256,  32, 8): 2,
226:     (256,  64, 8): 1,
227:     (256,  96, 8): 5,
228:     (256, 128, 8): 0,
229:     (256, 160, 8): 5,
230:     (256, 192, 8): 5,
231:     (256, 224, 8): 5,
232:     (256, 256, 8): 0,
233: }
234: 
235: # F8F6F4
236: SM100_MMA_SHAPES_F8F6F4_DENSE_1SM = {
237:     (64,   8, 32): 4,
238:     (64,  16, 32): 4,
239:     (64,  24, 32): 5,
240:     (64,  32, 32): 3,
241:     (64,  40, 32): 5,
242:     (64,  48, 32): 5,
243:     (64,  56, 32): 5,
244:     (64,  64, 32): 2,
245:     (64,  72, 32): 5,
246:     (64,  80, 32): 5,
247:     (64,  88, 32): 5,
248:     (64,  96, 32): 5,
249:     (64, 104, 32): 5,
250:     (64, 112, 32): 5,
251:     (64, 120, 32): 5,
252:     (64, 128, 32): 0,
253:     (64, 136, 32): 5,
254:     (64, 144, 32): 5,
255:     (64, 152, 32): 5,
256:     (64, 160, 32): 5,
257:     (64, 168, 32): 5,
258:     (64, 176, 32): 5,
259:     (64, 184, 32): 5,
260:     (64, 192, 32): 5,
261:     (64, 200, 32): 5,
262:     (64, 208, 32): 5,
263:     (64, 216, 32): 5,
264:     (64, 224, 32): 5,
265:     (64, 232, 32): 5,
266:     (64, 240, 32): 5,
267:     (64, 248, 32): 5,
268:     (64, 256, 32): 0,
269: 
270:     (128,  16, 32): 4,
271:     (128,  32, 32): 3,
272:     (128,  48, 32): 5,
273:     (128,  64, 32): 2,
274:     (128,  80, 32): 5,
275:     (128,  96, 32): 5,
276:     (128, 112, 32): 5,
277:     (128, 128, 32): 0,
278:     (128, 144, 32): 5,
279:     (128, 160, 32): 5,
280:     (128, 176, 32): 5,
281:     (128, 192, 32): 5,
282:     (128, 208, 32): 5,
283:     (128, 224, 32): 5,
284:     (128, 240, 32): 5,
285:     (128, 256, 32): 0,
286: 
287: }
288: 
289: SM100_MMA_SHAPES_F8F6F4_DENSE_2SM = {
290:     (128,  32, 32): 3,
291:     (128,  64, 32): 2,
292:     (128,  96, 32): 5,
293:     (128, 128, 32): 1,
294:     (128, 160, 32): 5,
295:     (128, 192, 32): 5,
296:     (128, 224, 32): 5,
297:     (128, 256, 32): 1,
298: 
299:     (256,  32, 32): 2,
300:     (256,  64, 32): 2,
301:     (256,  96, 32): 5,
302:     (256, 128, 32): 0,
303:     (256, 160, 32): 5,
304:     (256, 192, 32): 5,
305:     (256, 224, 32): 5,
306:     (256, 256, 32): 0,
307: }
308: 
309: # MXF8F6F4
310: SM100_MMA_SHAPES_MXF8F6F4_DENSE_1SM = {
311:     (128,  64, 32): 1,
312:     (128, 128, 32): 0,
313:     (128, 192, 32): 1,
314:     (128, 256, 32): 0,
315: }
316: 
317: 
318: SM100_MMA_SHAPES_MXF8F6F4_DENSE_2SM = {
319:     (256,  64, 32): 1,
320:     (256, 128, 32): 0,
321:     (256, 192, 32): 1,
322:     (256, 256, 32): 0,
323: 
324: 
325: }
326: 
327: # MXF4NVF4
328: SM100_MMA_SHAPES_MXF4NVF4_DENSE_1SM = {
329:     (128,  64, 64): 1,
330:     (128, 128, 64): 0,
331:     (128, 192, 64): 1,
332:     (128, 256, 64): 0,
333: }
334: 
335: SM100_MMA_SHAPES_MXF4NVF4_DENSE_2SM = {
336:     # Multiples of 16 for N
337:     (256,  64, 64): 1,
338:     (256, 128, 64): 0,
339:     (256, 192, 64): 1,
340:     (256, 256, 64): 0,
341: 
342: }
```
**EN:** Uses `try`/`except` to prefer package imports and fall back to local imports, keeping both installed-package and script execution modes working.

**CN:** 通过 `try`/`except` 优先使用包导入，并在失败时回退到本地导入，从而同时兼容已安装包与脚本运行模式。

## Key Concepts / 关键概念

- **EN:** File role: Stores architecture-specific SM100 shape catalogs used by higher-level generator utilities.
- **CN:** 文件角色：存储供上层生成工具使用的 SM100 架构专用 shape 目录。
- **EN:** Important constants/tables: `SM100_CLUSTER_SHAPES_1SM, SM100_CLUSTER_SHAPES_2SM, SM100_MMA_SHAPES_TF32_DENSE_1SM, SM100_MMA_SHAPES_TF32_DENSE_2SM, SM100_MMA_SHAPES_F8F6F4_DENSE_1SM, SM100_MMA_SHAPES_F8F6F4_DENSE_2SM, SM100_MMA_SHAPES_MXF8F6F4_DENSE_1SM, SM100_MMA_SHAPES_MXF8F6F4_DENSE_2SM, SM100_MMA_SHAPES_MXF4NVF4_DENSE_1SM, SM100_MMA_SHAPES_MXF4NVF4_DENSE_2SM`
- **CN:** 重要常量/表：`SM100_CLUSTER_SHAPES_1SM, SM100_CLUSTER_SHAPES_2SM, SM100_MMA_SHAPES_TF32_DENSE_1SM, SM100_MMA_SHAPES_TF32_DENSE_2SM, SM100_MMA_SHAPES_F8F6F4_DENSE_1SM, SM100_MMA_SHAPES_F8F6F4_DENSE_2SM, SM100_MMA_SHAPES_MXF8F6F4_DENSE_1SM, SM100_MMA_SHAPES_MXF8F6F4_DENSE_2SM, SM100_MMA_SHAPES_MXF4NVF4_DENSE_1SM, SM100_MMA_SHAPES_MXF4NVF4_DENSE_2SM`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `None explicitly imported`
- **CN:** 标准库依赖：`未显式导入`
- **EN:** Internal CUTLASS dependencies: `.library, library`
- **CN:** CUTLASS 内部依赖：`.library, library`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
