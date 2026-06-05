# chinese_regex.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `examples/frontend_language/usage/chinese_regex.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This example demonstrates chinese regex within the SGLang repository. / 该示例展示了在 SGLang 仓库中如何完成 chinese regex。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: Import dependencies and runtime symbols
````python
import sglang as sgl
````
**EN:** The file only needs the SGLang frontend/runtime API, which it imports under the `sgl` alias.
**CN:** 该文件只依赖 SGLang 前端/运行时 API，并以 `sgl` 别名导入。

### Lines 3-18: Set top-level configuration
````python
character_regex = (
    r"""\{\n"""
    + r"""    "姓名": "[^"]{1,32}",\n"""
    + r"""    "学院": "(格兰芬多|赫奇帕奇|拉文克劳|斯莱特林)",\n"""
    + r"""    "血型": "(纯血|混血|麻瓜)",\n"""
    + r"""    "职业": "(学生|教师|傲罗|魔法部|食死徒|凤凰社成员)",\n"""
    + r"""    "魔杖": \{\n"""
    + r"""        "材质": "[^"]{1,32}",\n"""
    + r"""        "杖芯": "[^"]{1,32}",\n"""
    + r"""        "长度": [0-9]{1,2}\.[0-9]{0,2}\n"""
    + r"""    \},\n"""
    + r"""    "存活": "(存活|死亡)",\n"""
    + r"""    "守护神": "[^"]{1,32}",\n"""
    + r"""    "博格特": "[^"]{1,32}"\n"""
    + r"""\}"""
)
````
**EN:** This block establishes constants or shared objects that the rest of the file relies on.
**CN:** 该代码块建立了后续逻辑依赖的常量或共享对象。

### Lines 21-42: Define character_gen prompt program
````python
@sgl.function
def character_gen(s, name):
    s += name + " 是一名哈利波特系列小说中的角色。请填写以下关于这个角色的信息。"
    s += """\
这是一个例子
{
    "姓名": "哈利波特",
    "学院": "格兰芬多",
    "血型": "混血",
    "职业": "学生",
    "魔杖": {
        "材质": "冬青木",
        "杖芯": "凤凰尾羽",
        "长度": 11.0
    },
    "存活": "存活",
    "守护神": "麋鹿",
    "博格特": "摄魂怪"
}
"""
    s += f"现在请你填写{name}的信息：\n"
    s += sgl.gen("json_output", max_tokens=256, regex=character_regex)
````
**EN:** This function defines an SGLang prompt program that appends turns to the state object and reserves one or more generation slots for model output.
**CN:** 该函数定义了一个 SGLang 提示程序：它会向状态对象追加对话或指令内容，并预留一个或多个生成槽位来接收模型输出。

### Lines 45-49: Parse arguments and run the workflow
````python
def main():
    backend = sgl.RuntimeEndpoint("http://localhost:30000")
    sgl.set_default_backend(backend)
    ret = character_gen.run(name="赫敏格兰杰", temperature=0)
    print(ret.text())
````
**EN:** This function serves as the entry workflow: it parses user options, prepares runtime objects, and dispatches the requested operation.
**CN:** 该函数充当入口工作流：它会解析用户选项、准备运行时对象，并分发到相应的操作步骤。

### Lines 52-53: Program entry point
````python
if __name__ == "__main__":
    main()
````
**EN:** The `__main__` block wires configuration together and runs the example end to end.
**CN:** `__main__` 代码块会把配置串联起来，并端到端地运行整个示例。

## Key Concepts / 关键概念
- **Prompt programming / 提示程序编排**: The file builds prompts with SGLang state transitions. / 该文件使用 SGLang 状态转换来构造提示程序。

## Dependencies / 依赖关系
- **Project-specific / 项目相关**: sglang
