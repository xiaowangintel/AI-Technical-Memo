# json_decode.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/json_decode.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example demonstrates json decode within the SGLang repository. / 该示例展示了在 SGLang 仓库中如何完成 json decode。

## Line-by-Line Analysis / 逐行分析
### Lines 1-5: Module overview and usage
````python
"""
Usage:
python -m sglang.launch_server --model-path meta-llama/Llama-2-7b-chat-hf --port 30000
python json_decode.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 7-12: Import dependencies and runtime symbols
````python
from enum import Enum

from pydantic import BaseModel

import sglang as sgl
from sglang.srt.constrained.outlines_backend import build_regex_from_object
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 14-29: Set top-level configuration
````python
character_regex = (
    r"""\{\n"""
    + r"""    "name": "[\w\d\s]{1,16}",\n"""
    + r"""    "house": "(Gryffindor|Slytherin|Ravenclaw|Hufflepuff)",\n"""
    + r"""    "blood status": "(Pure-blood|Half-blood|Muggle-born)",\n"""
    + r"""    "occupation": "(student|teacher|auror|ministry of magic|death eater|order of the phoenix)",\n"""
    + r"""    "wand": \{\n"""
    + r"""        "wood": "[\w\d\s]{1,16}",\n"""
    + r"""        "core": "[\w\d\s]{1,16}",\n"""
    + r"""        "length": [0-9]{1,2}\.[0-9]{0,2}\n"""
    + r"""    \},\n"""
    + r"""    "alive": "(Alive|Deceased)",\n"""
    + r"""    "patronus": "[\w\d\s]{1,16}",\n"""
    + r"""    "bogart": "[\w\d\s]{1,16}"\n"""
    + r"""\}"""
)
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 32-41: Define character_gen prompt program
````python
@sgl.function
def character_gen(s, name):
    s += (
        name
        + " is a character in Harry Potter. Please fill in the following information about this character.\n"
    )
    s += "The constrained regex is:\n"
    s += character_regex + "\n"
    s += "The JSON output is:\n"
    s += sgl.gen("json_output", max_tokens=256, regex=character_regex)
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 44-46: Handle character gen
````python
def driver_character_gen():
    state = character_gen.run(name="Hermione Granger")
    print(state.text())
````
**EN:** This function encapsulates the “Handle character gen” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle character gen”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 49-55: Define Weapon class
````python
class Weapon(str, Enum):
    sword = "sword"
    axe = "axe"
    mace = "mace"
    spear = "spear"
    bow = "bow"
    crossbow = "crossbow"
````
**EN:** This class collects the state and helper methods required by this example.
**CN:** 该类汇集了此示例所需的状态以及辅助方法。

### Lines 58-61: Define Wizard schema
````python
class Wizard(BaseModel):
    name: str
    age: int
    weapon: Weapon
````
**EN:** This class groups typed fields into a schema that constrains or validates model output.
**CN:** 该类把带类型的字段组织成一个模式，用于约束或校验模型输出。

### Lines 64-72: Define pydantic_wizard_gen prompt program
````python
@sgl.function
def pydantic_wizard_gen(s):
    s += "Give me a description about a wizard in the JSON format.\n"
    s += sgl.gen(
        "character",
        max_tokens=128,
        temperature=0,
        regex=build_regex_from_object(Wizard),  # Requires pydantic >= 2.0
    )
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 75-77: Handle pydantic wizard gen
````python
def driver_pydantic_wizard_gen():
    state = pydantic_wizard_gen.run()
    print(state.text())
````
**EN:** This function encapsulates the “Handle pydantic wizard gen” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Handle pydantic wizard gen”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 80-82: Program entry point
````python
if __name__ == "__main__":
    sgl.set_default_backend(sgl.RuntimeEndpoint("http://localhost:30000"))
    driver_character_gen()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。
- **Structured generation / 结构化生成**: Schemas or constrained decoding keep outputs machine-readable. / 模式约束或受限解码让输出保持机器可读。

## Dependencies / 依赖关系
- **Standard library / 标准库**: enum.Enum
- **Third-party / 第三方**: pydantic.BaseModel
- **Project-specific / 项目相关**: sglang, sglang.srt.constrained.outlines_backend.build_regex_from_object
