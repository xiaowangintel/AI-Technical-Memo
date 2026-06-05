# openai_speculative.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/openai_speculative.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example demonstrates openai speculative within the SGLang repository. / 该示例展示了在 SGLang 仓库中如何完成 openai speculative。

## Line-by-Line Analysis / 逐行分析
### Lines 1-4: Module overview and usage
````python
"""
Usage:
python3 openai_speculative.py
"""
````
**EN:** The opening docstring documents the expected inputs, environment variables, or command line used to run the example.
**CN:** 开头的文档字符串说明了运行该示例所需的输入、环境变量或命令行参数。

### Lines 6-6: Import dependencies and runtime symbols
````python
from sglang import OpenAI, function, gen, set_default_backend
````
**EN:** This import section brings in the standard helpers and external packages used by the rest of the example.
**CN:** 这一导入部分引入了后续示例会使用到的标准工具和外部依赖。

### Lines 9-15: Generate character spec
````python
@function(num_api_spec_tokens=64)
def gen_character_spec(s):
    s += "Construct a character within the following format:\n"
    s += "Name: Steve Jobs.\nBirthday: February 24, 1955.\nJob: Apple CEO.\n"
    s += "\nPlease generate new Name, Birthday and Job.\n"
    s += "Name:" + gen("name", stop="\n") + "\nBirthday:" + gen("birthday", stop="\n")
    s += "\nJob:" + gen("job", stop="\n") + "\n"
````
**EN:** This function encapsulates the “Generate character spec” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Generate character spec”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 18-24: Define gen_character_no_spec prompt program
````python
@function
def gen_character_no_spec(s):
    s += "Construct a character within the following format:\n"
    s += "Name: Steve Jobs.\nBirthday: February 24, 1955.\nJob: Apple CEO.\n"
    s += "\nPlease generate new Name, Birthday and Job.\n"
    s += "Name:" + gen("name", stop="\n") + "\nBirthday:" + gen("birthday", stop="\n")
    s += "\nJob:" + gen("job", stop="\n") + "\n"
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 27-32: Generate character spec no few shot
````python
@function(num_api_spec_tokens=64)
def gen_character_spec_no_few_shot(s):
    # s += "Construct a character with name, birthday, and job:\n"
    s += "Construct a character:\n"
    s += "Name:" + gen("name", stop="\n") + "\nBirthday:" + gen("birthday", stop="\n")
    s += "\nJob:" + gen("job", stop="\n") + "\n"
````
**EN:** This function encapsulates the “Generate character spec no few shot” step so the overall example remains modular and easier to follow.
**CN:** 该函数把“Generate character spec no few shot”这一步封装成独立逻辑，从而让整个示例更模块化、更易于理解。

### Lines 35-54: Program entry point
````python
if __name__ == "__main__":
    backend = OpenAI("gpt-3.5-turbo-instruct")
    set_default_backend(backend)

    for function in [
        gen_character_spec,
        gen_character_no_spec,
        gen_character_spec_no_few_shot,
    ]:
        backend.token_usage.reset()

        print(f"function: {function.func.__name__}")

        state = function.run()

        print("...name:", state["name"])
        print("...birthday:", state["birthday"])
        print("...job:", state["job"])
        print(backend.token_usage)
        print()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Engine lifecycle / 引擎生命周期**: The example creates, uses, and tears down an SGLang engine. / 该示例展示了 SGLang 引擎的创建、使用和关闭。

## Dependencies / 依赖关系
- **Project-specific / 项目相关**: sglang.OpenAI, sglang.function, sglang.gen, sglang.set_default_backend
