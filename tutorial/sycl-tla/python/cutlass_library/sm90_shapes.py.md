# sm90_shapes.py — Code Analysis / 代码分析

## Source / 来源
- **Path / 路径:** `python/cutlass_library/sm90_shapes.py`
- **EN:** Stores architecture-specific SM90 shape catalogs used by higher-level generator utilities.
- **CN:** 存储供上层生成工具使用的 SM90 架构专用 shape 目录。

## Line-by-Line Analysis / 逐行分析

### L1-L32 — Header comments

```python
  1: #################################################################################################
  2: #
  3: # Copyright (c) 2024 - 2025 NVIDIA CORPORATION & AFFILIATES. All rights reserved.
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

### L33-L212 — Data definitions

```python
 33: """
 34: Valid WGMMA shapes, MMA multipliers, and cluster sizes for SM90, associated with levels.
 35: These shape and level pairs are defined as dicts, where keys are shapes and values are their
 36: associated levels. If the user input level for that category (MMA multiplier, WGMMA shape, cluster
 37: size) is smaller than a shape's associated level, it will be excluded, and otherwise, included.
 38: Higher levels are therefore less likely emitted, but lower levels are more emitted more frequently.
 39: Level 0 is always emitted. The default behavior in `generator.py` is that level 1 is only emitted
 40: when the `--kernel` argument is non-empty.
 41: """
 42: 
 43: # NOTE: more combinations are possible here.
 44: # Levels [0, 3] exist in order to control exactly what configs are generated in different dtypes.
 45: # The rest are only used in the exhaustive mode (when the corresponding level digit is 9).
 46: # MMA multipliers are multiplied by MMA instruction shapes (WGMMA shapes) to get CTA shapes.
 47: SM90_MMA_MULTIPLIERS = {
 48:     (2, 1, 4): 0,
 49:     (1, 1, 4): 1,
 50:     (4, 1, 4): 2,
 51:     (2, 2, 4): 3,
 52:     (2, 1, 8): 4,
 53:     (4, 1, 8): 4,
 54:     (1, 1, 8): 4,
 55:     (2, 2, 8): 4,
 56:     (2, 1, 16): 5,
 57:     (4, 1, 16): 5,
 58:     (1, 1, 16): 5,
 59:     (2, 2, 16): 5,
 60: }
 61: 
 62: # Level 0: only (1, 2, 1) -- fp8 dense gemms in pruned case
 63: # Level 1: clusters with 2 CTAs -- all but fp8 (s8, u8, f16, b16, f32, tf32) dense gemms in pruned case
 64: # Level 2: clusters with 1 or 2 CTAs
 65: # Level 3: clusters with 1, 2, or 4 CTAs
 66: # Level 4: clusters with 1, 2, 4, or 8 CTAs
 67: # Level 5: clusters with 1, 2, 4, 8, or 16 CTAs
 68: SM90_CLUSTER_SIZES = {
 69:     (1, 2, 1): 0,
 70:     (2, 1, 1): 1,
 71:     (1, 1, 1): 2,
 72:     (2, 2, 1): 3,
 73:     (1, 4, 1): 3,
 74:     (4, 1, 1): 3,
 75:     (2, 4, 1): 4,
 76:     (4, 2, 1): 4,
 77:     (1, 8, 1): 4,
 78:     (8, 1, 1): 4,
 79:     (4, 4, 1): 5,
 80: }
 81: 
 82: 
 83: # WGMMA shapes
 84: # Level 0: "default" shape only,
 85: # Level 1: additional shapes for the unpruned case (tf32 only)
 86: # Level 2: shapes that are all powers of 2
 87: # Level 3: all other shapes
 88: SM90_WGMMA_SHAPES_FP16_BF16_DENSE = {
 89:     (64, 8, 16): 2,
 90:     (64, 16, 16): 2,
 91:     (64, 24, 16): 3,
 92:     (64, 32, 16): 2,
 93:     (64, 40, 16): 3,
 94:     (64, 48, 16): 3,
 95:     (64, 56, 16): 3,
 96:     (64, 64, 16): 2,
 97:     (64, 72, 16): 3,
 98:     (64, 80, 16): 3,
 99:     (64, 88, 16): 3,
100:     (64, 96, 16): 3,
101:     (64, 104, 16): 3,
102:     (64, 112, 16): 3,
103:     (64, 120, 16): 3,
104:     (64, 128, 16): 0,
105:     (64, 136, 16): 3,
106:     (64, 144, 16): 3,
107:     (64, 152, 16): 3,
108:     (64, 160, 16): 3,
109:     (64, 168, 16): 3,
110:     (64, 176, 16): 3,
111:     (64, 184, 16): 3,
112:     (64, 192, 16): 3,
113:     (64, 200, 16): 3,
114:     (64, 208, 16): 3,
115:     (64, 216, 16): 3,
116:     (64, 224, 16): 3,
117:     (64, 232, 16): 3,
118:     (64, 240, 16): 3,
119:     (64, 248, 16): 3,
120:     (64, 256, 16): 1,
121: }
122: 
123: SM90_WGMMA_SHAPES_TF32_DENSE = {
124:     (64, 8, 8): 2,
125:     (64, 16, 8): 2,
126:     (64, 24, 8): 3,
127:     (64, 32, 8): 2,
128:     (64, 40, 8): 3,
129:     (64, 48, 8): 3,
130:     (64, 56, 8): 3,
131:     (64, 64, 8): 2,
132:     (64, 72, 8): 3,
133:     (64, 80, 8): 3,
134:     (64, 88, 8): 3,
135:     (64, 96, 8): 3,
136:     (64, 104, 8): 3,
137:     (64, 112, 8): 3,
138:     (64, 120, 8): 3,
139:     (64, 128, 8): 0,
140:     (64, 136, 8): 3,
141:     (64, 144, 8): 3,
142:     (64, 152, 8): 3,
143:     (64, 160, 8): 3,
144:     (64, 168, 8): 3,
145:     (64, 176, 8): 3,
146:     (64, 184, 8): 3,
147:     (64, 192, 8): 3,
148:     (64, 200, 8): 3,
149:     (64, 208, 8): 3,
150:     (64, 216, 8): 3,
151:     (64, 224, 8): 3,
152:     (64, 232, 8): 3,
153:     (64, 240, 8): 3,
154:     (64, 248, 8): 3,
155:     (64, 256, 8): 1,
156: }
157: 
158: SM90_WGMMA_SHAPES_FP8_DENSE = {
159:     (64, 8, 32): 2,
160:     (64, 16, 32): 2,
161:     (64, 24, 32): 3,
162:     (64, 32, 32): 2,
163:     (64, 40, 32): 3,
164:     (64, 48, 32): 3,
165:     (64, 56, 32): 3,
166:     (64, 64, 32): 2,
167:     (64, 72, 32): 3,
168:     (64, 80, 32): 3,
169:     (64, 88, 32): 3,
170:     (64, 96, 32): 3,
171:     (64, 104, 32): 3,
172:     (64, 112, 32): 3,
173:     (64, 120, 32): 3,
174:     (64, 128, 32): 0,
175:     (64, 136, 32): 3,
176:     (64, 144, 32): 3,
177:     (64, 152, 32): 3,
178:     (64, 160, 32): 3,
179:     (64, 168, 32): 3,
180:     (64, 176, 32): 3,
181:     (64, 184, 32): 3,
182:     (64, 192, 32): 3,
183:     (64, 200, 32): 3,
184:     (64, 208, 32): 3,
185:     (64, 216, 32): 3,
186:     (64, 224, 32): 3,
187:     (64, 232, 32): 3,
188:     (64, 240, 32): 3,
189:     (64, 248, 32): 3,
190:     (64, 256, 32): 1,
191: }
192: 
193: SM90_WGMMA_SHAPES_INT8_DENSE = {
194:     (64, 8, 32): 2,
195:     (64, 16, 32): 2,
196:     (64, 24, 32): 3,
197:     (64, 32, 32): 2,
198:     (64, 48, 32): 3,
199:     (64, 64, 32): 2,
200:     (64, 80, 32): 3,
201:     (64, 96, 32): 3,
202:     (64, 112, 32): 3,
203:     (64, 128, 32): 0,
204:     (64, 144, 32): 3,
205:     (64, 160, 32): 3,
206:     (64, 176, 32): 3,
207:     (64, 192, 32): 3,
208:     (64, 208, 32): 3,
209:     (64, 224, 32): 3,
210:     (64, 240, 32): 3,
211:     (64, 256, 32): 1,
212: }
```
**EN:** Defines or updates module/class-level data such as SM90_MMA_MULTIPLIERS, SM90_CLUSTER_SIZES, SM90_WGMMA_SHAPES_FP16_BF16_DENSE, SM90_WGMMA_SHAPES_TF32_DENSE, SM90_WGMMA_SHAPES_FP8_DENSE, SM90_WGMMA_SHAPES_INT8_DENSE; these values are referenced by later generators and emitters.

**CN:** 定义或更新模块/类级数据，例如 SM90_MMA_MULTIPLIERS, SM90_CLUSTER_SIZES, SM90_WGMMA_SHAPES_FP16_BF16_DENSE, SM90_WGMMA_SHAPES_TF32_DENSE, SM90_WGMMA_SHAPES_FP8_DENSE, SM90_WGMMA_SHAPES_INT8_DENSE；这些值会被后续生成器与 emitter 引用。

## Key Concepts / 关键概念

- **EN:** File role: Stores architecture-specific SM90 shape catalogs used by higher-level generator utilities.
- **CN:** 文件角色：存储供上层生成工具使用的 SM90 架构专用 shape 目录。
- **EN:** Important constants/tables: `SM90_MMA_MULTIPLIERS, SM90_CLUSTER_SIZES, SM90_WGMMA_SHAPES_FP16_BF16_DENSE, SM90_WGMMA_SHAPES_TF32_DENSE, SM90_WGMMA_SHAPES_FP8_DENSE, SM90_WGMMA_SHAPES_INT8_DENSE`
- **CN:** 重要常量/表：`SM90_MMA_MULTIPLIERS, SM90_CLUSTER_SIZES, SM90_WGMMA_SHAPES_FP16_BF16_DENSE, SM90_WGMMA_SHAPES_TF32_DENSE, SM90_WGMMA_SHAPES_FP8_DENSE, SM90_WGMMA_SHAPES_INT8_DENSE`

## Dependencies / 依赖关系

- **EN:** Standard-library dependencies: `None explicitly imported`
- **CN:** 标准库依赖：`未显式导入`
- **EN:** Internal CUTLASS dependencies: `None explicitly imported`
- **CN:** CUTLASS 内部依赖：`未显式导入`
- **EN:** External/non-stdlib dependencies: `None explicitly imported`
- **CN:** 外部/非标准库依赖：`未显式导入`
