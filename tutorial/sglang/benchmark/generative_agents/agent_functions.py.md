# agent_functions.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `benchmark/generative_agents/agent_functions.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This Python module focuses on generative agents agent functions. It primarily processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. / 该 Python 模块聚焦于 generative agents agent functions 相关流程。它主要用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Imports and setup / 导入与初始化
```python
import sglang as sgl
```
**EN:** This block imports required libraries and establishes module-level constants or helpers used later in the file.
**CN:** 该代码块导入所需库，并建立后续逻辑会使用到的模块级常量或辅助配置。

### Lines 8-15: Function `poignancy_event` / 函数 `poignancy_event`
```python
def poignancy_event(s, persona_name, persona_iss, event):
    s += "Here is a brief description of " + persona_name + ".\n"
    s += persona_iss + "\n"
    s += "On the scale of 1 to 10, where 1 is purely mundane (e.g., brushing teeth, making bed) and 10 is extremely poignant (e.g., a break up, college acceptance), rate the likely poignancy of the following event for"
    s += persona_name + ".\n\n"
    s += "Event: " + event
    s += "Rate (return a number between 1 to 10):"
    s += sgl.gen(name="Rate", max_tokens=2)
```
**EN:** `poignancy_event` is a function that processes tokenized prompts or decoded outputs. Notable calls include `sgl.gen`.
**CN:** `poignancy_event` 是一个函数，用于处理分词后的提示词或解码后的输出。其中较关键的调用包括 `sgl.gen`。

### Lines 18-27: Function `poignancy_event_prompt` / 函数 `poignancy_event_prompt`
```python
def poignancy_event_prompt(persona_name, persona_iss, event):
    # return prompt and max_tokens
    s = ""
    s += "Here is a brief description of " + persona_name + ".\n"
    s += persona_iss + "\n"
    s += "On the scale of 1 to 10, where 1 is purely mundane (e.g., brushing teeth, making bed) and 10 is extremely poignant (e.g., a break up, college acceptance), rate the likely poignancy of the following event for"
    s += persona_name + ".\n\n"
    s += "Event: " + event
    s += "Rate (return a number between 1 to 10):"
    return {"prompt": s, "max_tokens": 2, "stop": None}
```
**EN:** `poignancy_event_prompt` is a function that processes tokenized prompts or decoded outputs. It returns `{'prompt': s, 'max_tokens': 2, 'stop': None}` to the caller.
**CN:** `poignancy_event_prompt` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `{'prompt': s, 'max_tokens': 2, 'stop': None}`。

### Lines 31-53: Function `generate_event_triple` / 函数 `generate_event_triple`
```python
def generate_event_triple(s, persona_name, action):
    s += """Task: Turn the input into (subject, predicate, object).
Input: Sam Johnson is eating breakfast.
Output: (Dolores Murphy, eat, breakfast)
---
Input: Joon Park is brewing coffee.
Output: (Joon Park, brew, coffee)
---
Input: Jane Cook is sleeping.
Output: (Jane Cook, is, sleep)
---
Input: Michael Bernstein is writing email on a computer.
Output: (Michael Bernstein, write, email)
---
Input: Percy Liang is teaching students in a classroom.
Output: (Percy Liang, teach, students)
---
Input: Merrie Morris is running on a treadmill.
Output: (Merrie Morris, run, treadmill)
---"""
    s += persona_name + "is" + action + ".\n"
    s += "(" + persona_name + ","
    s += sgl.gen(name="Triple", max_tokens=20, stop=")")
```
**EN:** `generate_event_triple` is a function that processes tokenized prompts or decoded outputs. Notable calls include `sgl.gen`.
**CN:** `generate_event_triple` 是一个函数，用于处理分词后的提示词或解码后的输出。其中较关键的调用包括 `sgl.gen`。

### Lines 56-79: Function `generate_event_triple_prompt` / 函数 `generate_event_triple_prompt`
```python
def generate_event_triple_prompt(persona_name, action):
    s = ""
    s += """Task: Turn the input into (subject, predicate, object).
Input: Sam Johnson is eating breakfast.
Output: (Dolores Murphy, eat, breakfast)
---
Input: Joon Park is brewing coffee.
Output: (Joon Park, brew, coffee)
---
Input: Jane Cook is sleeping.
Output: (Jane Cook, is, sleep)
---
Input: Michael Bernstein is writing email on a computer.
Output: (Michael Bernstein, write, email)
---
Input: Percy Liang is teaching students in a classroom.
Output: (Percy Liang, teach, students)
---
Input: Merrie Morris is running on a treadmill.
Output: (Merrie Morris, run, treadmill)
---"""
    s += persona_name + "is" + action + ".\n"
    s += "(" + persona_name + ","
    return {"prompt": s, "max_tokens": 20, "stop": ")"}
```
**EN:** `generate_event_triple_prompt` is a function that processes tokenized prompts or decoded outputs. It returns `{'prompt': s, 'max_tokens': 20, 'stop': ')'}` to the caller.
**CN:** `generate_event_triple_prompt` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `{'prompt': s, 'max_tokens': 20, 'stop': ')'}`。

### Lines 83-86: Function `generate_pronunciatio` / 函数 `generate_pronunciatio`
```python
def generate_pronunciatio(s, action):
    s += "Convert an action description to an emoji (important: use two or less emojis).\n"
    s += "Action description: " + action + ".\n"
    s += "Emoji:" + sgl.gen(name="Emoji", max_tokens=6)
```
**EN:** `generate_pronunciatio` is a function that processes tokenized prompts or decoded outputs. Notable calls include `sgl.gen`.
**CN:** `generate_pronunciatio` 是一个函数，用于处理分词后的提示词或解码后的输出。其中较关键的调用包括 `sgl.gen`。

### Lines 89-94: Function `generate_pronunciatio_prompt` / 函数 `generate_pronunciatio_prompt`
```python
def generate_pronunciatio_prompt(action):
    s = ""
    s += "Convert an action description to an emoji (important: use two or less emojis).\n"
    s += "Action description: " + action + ".\n"
    s += "Emoji:"
    return {"prompt": s, "max_tokens": 6, "stop": None}
```
**EN:** `generate_pronunciatio_prompt` is a function that processes tokenized prompts or decoded outputs. It returns `{'prompt': s, 'max_tokens': 6, 'stop': None}` to the caller.
**CN:** `generate_pronunciatio_prompt` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `{'prompt': s, 'max_tokens': 6, 'stop': None}`。

### Lines 98-155: Function `action_location_sector` / 函数 `action_location_sector`
```python
def action_location_sector(
    s,
    persona_name,
    living_sector,
    living_sector_areas,
    current_sector,
    current_sector_areas,
    daily_plan,
    sector_options,
    current_action,
    next_action,
):
    s += """Task -- choose an appropriate area  from the area options for a task at hand.
Sam Kim lives in {Sam Kim's house} that has Sam Kim's room, bathroom, kitchen.
Sam Kim is currently in {Sam Kim's house} that has Sam Kim's room, bathroom, kitchen.
Area options: {Sam Kim's house, The Rose and Crown Pub, Hobbs Cafe, Oak Hill College, Johnson Park, Harvey Oak Supply Store, The Willows Market and Pharmacy}.
* Stay in the current area if the activity can be done there. Only go out if the activity needs to take place in another place.
* Must be one of the "Area options," verbatim.
For taking a walk, Sam Kim should go to the following area: {Johnson Park}
---
Jane Anderson lives in {Oak Hill College Student Dormatory} that has Jane Anderson's room.
Jane Anderson is currently in {Oak Hill College} that has a classroom, library
Area options: {Oak Hill College Student Dormatory, The Rose and Crown Pub, Hobbs Cafe, Oak Hill College, Johnson Park, Harvey Oak Supply Store, The Willows Market and Pharmacy}.
* Stay in the current area if the activity can be done there. Only go out if the activity needs to take place in another place.
* Must be one of the "Area options," verbatim.
For eating dinner, Jane Anderson should go to the following area: {Hobbs Cafe}
---"""
    s += (
        persona_name
        + " lives in "
        + living_sector
        + " that has "
        + living_sector_areas
        + ".\n"
    )
    s += (
        persona_name
        + " is currently in "
        + current_sector
        + " that has "
        + current_sector_areas
        + ".\n"
    )
    s += daily_plan + ".\n"
    s += "Area options: " + sector_options + ".\n"
    s += """* Stay in the current area if the activity can be done there. Only go out if the activity needs to take place in another place.
* Must be one of the "Area options," verbatim.\n"""
    s += (
        persona_name
        + " is "
        + current_action
        + ". For "
        + next_action
        + ", "
        + persona_name
        + " should go to the following area: {"
    )
    s += sgl.gen(name="Location", max_tokens=10, stop="}")
```
**EN:** `action_location_sector` is a function that processes tokenized prompts or decoded outputs. Notable calls include `sgl.gen`.
**CN:** `action_location_sector` 是一个函数，用于处理分词后的提示词或解码后的输出。其中较关键的调用包括 `sgl.gen`。

### Lines 158-215: Function `action_location_sector_prompt` / 函数 `action_location_sector_prompt`
```python
def action_location_sector_prompt(
    persona_name,
    living_sector,
    living_sector_areas,
    current_sector,
    current_sector_areas,
    daily_plan,
    sector_options,
    current_action,
    next_action,
):
    s = ""
    s += """Task -- choose an appropriate area  from the area options for a task at hand.
Sam Kim lives in {Sam Kim's house} that has Sam Kim's room, bathroom, kitchen.
Sam Kim is currently in {Sam Kim's house} that has Sam Kim's room, bathroom, kitchen.
Area options: {Sam Kim's house, The Rose and Crown Pub, Hobbs Cafe, Oak Hill College, Johnson Park, Harvey Oak Supply Store, The Willows Market and Pharmacy}.
* Stay in the current area if the activity can be done there. Only go out if the activity needs to take place in another place.
* Must be one of the "Area options," verbatim.
For taking a walk, Sam Kim should go to the following area: {Johnson Park}
---
Jane Anderson lives in {Oak Hill College Student Dormatory} that has Jane Anderson's room.
Jane Anderson is currently in {Oak Hill College} that has a classroom, library
Area options: {Oak Hill College Student Dormatory, The Rose and Crown Pub, Hobbs Cafe, Oak Hill College, Johnson Park, Harvey Oak Supply Store, The Willows Market and Pharmacy}.
* Stay in the current area if the activity can be done there. Only go out if the activity needs to take place in another place.
* Must be one of the "Area options," verbatim.
For eating dinner, Jane Anderson should go to the following area: {Hobbs Cafe}
---"""
    s += (
        persona_name
        + " lives in "
        + living_sector
        + " that has "
        + living_sector_areas
        + ".\n"
    )
    s += (
        persona_name
        + " is currently in "
        + current_sector
        + " that has "
        + current_sector_areas
        + ".\n"
    )
    s += daily_plan + ".\n"
    s += "Area options: " + sector_options + ".\n"
    s += """* Stay in the current area if the activity can be done there. Only go out if the activity needs to take place in another place.
* Must be one of the "Area options," verbatim.\n"""
    s += (
        persona_name
        + " is "
        + current_action
        + ". For "
        + next_action
        + ", "
        + persona_name
        + " should go to the following area: {"
    )
    return {"prompt": s, "max_tokens": 10, "stop": "}"}
```
**EN:** `action_location_sector_prompt` is a function that processes tokenized prompts or decoded outputs. It returns `{'prompt': s, 'max_tokens': 10, 'stop': '}'}` to the caller.
**CN:** `action_location_sector_prompt` 是一个函数，用于处理分词后的提示词或解码后的输出。它会向调用方返回 `{'prompt': s, 'max_tokens': 10, 'stop': '}'}`。

### Lines 219-257: Function `action_location_object` / 函数 `action_location_object`
```python
def action_location_object(
    s, persona_name, target_sector, target_sector_areas, current_action, next_action
):
    s += """
Jane Anderson is in kitchen in Jane Anderson's house.
Jane Anderson is going to Jane Anderson's house that has the following areas: {kitchen,  bedroom, bathroom}
Stay in the current area if the activity can be done there. Never go into other people's rooms unless necessary.
For cooking, Jane Anderson should go to the following area in Jane Anderson's house:
Answer: {kitchen}
---
Tom Watson is in common room in Tom Watson's apartment.
Tom Watson is going to Hobbs Cafe that has the following areas: {cafe}
Stay in the current area if the activity can be done there. Never go into other people's rooms unless necessary.
For getting coffee, Tom Watson should go to the following area in Hobbs Cafe:
Answer: {cafe}
---"""
    s += (
        persona_name
        + " is going to "
        + target_sector
        + " that has the following areas: {"
        + target_sector_areas
        + "}\n"
    )
    s += """* Stay in the current area if the activity can be done there.
* NEVER go into other people's rooms unless necessary."""
    s += (
        persona_name
        + " is "
        + current_action
        + ". For "
        + next_action
        + ", "
        + persona_name
        + "should go to the following area in "
        + target_sector
    )
    s += " (MUST pick one of {" + target_sector_areas + "}):\n"
    s += "Answer: {" + sgl.gen(name="Area", max_tokens=5, stop="}")
```
**EN:** `action_location_object` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. Notable calls include `sgl.gen`.
**CN:** `action_location_object` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。其中较关键的调用包括 `sgl.gen`。

### Lines 260-300: Function `action_location_object_prompt` / 函数 `action_location_object_prompt`
```python
def action_location_object_prompt(
    persona_name, target_sector, target_sector_areas, current_action, next_action
):
    s = ""
    s += """
Jane Anderson is in kitchen in Jane Anderson's house.
Jane Anderson is going to Jane Anderson's house that has the following areas: {kitchen,  bedroom, bathroom}
Stay in the current area if the activity can be done there. Never go into other people's rooms unless necessary.
For cooking, Jane Anderson should go to the following area in Jane Anderson's house:
Answer: {kitchen}
---
Tom Watson is in common room in Tom Watson's apartment.
Tom Watson is going to Hobbs Cafe that has the following areas: {cafe}
Stay in the current area if the activity can be done there. Never go into other people's rooms unless necessary.
For getting coffee, Tom Watson should go to the following area in Hobbs Cafe:
Answer: {cafe}
---"""
    s += (
        persona_name
        + " is going to "
        + target_sector
        + " that has the following areas: {"
        + target_sector_areas
        + "}\n"
    )
    s += """* Stay in the current area if the activity can be done there.
* NEVER go into other people's rooms unless necessary."""
    s += (
        persona_name
        + " is "
        + current_action
        + ". For "
        + next_action
        + ", "
        + persona_name
        + "should go to the following area in "
        + target_sector
    )
    s += " (MUST pick one of {" + target_sector_areas + "}):\n"
    s += "Answer: {"
    return {"prompt": s, "max_tokens": 5, "stop": "}"}
```
**EN:** `action_location_object_prompt` is a function that processes tokenized prompts or decoded outputs and computes evaluation scores and aggregate statistics. It returns `{'prompt': s, 'max_tokens': 5, 'stop': '}'}` to the caller.
**CN:** `action_location_object_prompt` 是一个函数，用于处理分词后的提示词或解码后的输出、计算评测分数与聚合统计结果。它会向调用方返回 `{'prompt': s, 'max_tokens': 5, 'stop': '}'}`。

## Key Concepts / 关键概念
- **Benchmarking / 基准测试**: Measures performance-related signals such as latency, throughput, or speedup. / 衡量延迟、吞吐或加速比等性能信号。
- **Tokenization / 分词**: Counts or transforms prompt/output tokens for fair comparisons. / 对提示词与输出 token 进行统计或转换，以便公平比较。
- **Metrics / 指标**: Aggregates quality signals in addition to raw runtime. / 除运行时之外还汇总质量指标。
- **Orchestration / 编排**: Launches tools, captures logs, or stitches multi-step experiments together. / 启动工具、抓取日志或拼接多步实验流程。

## Dependencies / 依赖关系
- **Internal / 项目内部依赖**: `sglang`
