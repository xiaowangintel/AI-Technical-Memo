# test_conversation.py — Code Analysis / 代码分析

## Source / 来源
- **File**: `test/registered/unit/parser/test_conversation.py`
- **Repository**: sgl-project/sglang
- **Purpose**: This test module validates conversation behavior in SGLang's unit / parser area. It prepares inputs, exercises runtime paths, and checks expected results or regressions. / 该测试模块验证 SGLang 在 单元 / 解析器 领域中与 conversation 相关的行为。它会准备输入、执行运行路径，并检查预期结果或回归情况。

## Line-by-Line Analysis / 逐行分析
### Lines 1-1: supporting statements / 辅助语句
```python
"""Unit tests for srt/parser/conversation.py"""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 3-33: module imports and dependencies / 模块导入与依赖
```python
import json
import os
import tempfile
import unittest

from sglang.srt.entrypoints.openai.protocol import (
    ChatCompletionMessageContentAudioPart,
    ChatCompletionMessageContentAudioURL,
    ChatCompletionMessageContentImagePart,
    ChatCompletionMessageContentImageURL,
    ChatCompletionMessageContentTextPart,
    ChatCompletionMessageContentVideoPart,
    ChatCompletionMessageContentVideoURL,
    ChatCompletionMessageGenericParam,
    ChatCompletionMessageUserParam,
    ChatCompletionRequest,
)
from sglang.srt.parser.conversation import (
    Conversation,
    SeparatorStyle,
    _get_full_multimodal_text_prompt,
    chat_template_exists,
    chat_templates,
    generate_chat_conv,
    generate_embedding_convs,
    get_conv_template_by_model_path,
    get_model_type,
    register_conv_template,
)
from sglang.test.ci.ci_register import register_cpu_ci
from sglang.test.test_utils import CustomTestCase
```
**EN:** This block imports the modules needed by the rest of the file, including `json`, `os`, `tempfile`, `unittest`.
**CN:** 该代码块导入后续实现所需的模块，其中包括 `json`, `os`, `tempfile`, `unittest`。

### Lines 35-35: CI registration and metadata / CI 注册与元数据
```python
register_cpu_ci(est_time=7, suite="base-a-test-cpu")
```
**EN:** This block registers the test with the continuous-integration harness and records scheduling metadata through register_cpu_ci.
**CN:** 该代码块通过 register_cpu_ci 等调用将测试注册到持续集成框架，并记录调度元数据。

### Lines 38-38: class TestConversationGetPrompt declaration / 类 TestConversationGetPrompt 声明
```python
class TestConversationGetPrompt(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 39-53: test case add colon single / 测试用例 add colon single
```python
    def test_add_colon_single(self):
        """Test prompt generation with ADD_COLON_SINGLE style."""
        conv = Conversation(
            name="test",
            system_message="System msg",
            roles=("User", "Assistant"),
            messages=[["User", "Hello"], ["Assistant", "Hi"], ["User", None]],
            sep_style=SeparatorStyle.ADD_COLON_SINGLE,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("System msg\n", prompt)
        self.assertIn("User: Hello\n", prompt)
        self.assertIn("Assistant: Hi\n", prompt)
        self.assertTrue(prompt.endswith("User:"))
```
**EN:** Test prompt generation with ADD_COLON_SINGLE style. This test exercises `test_add_colon_single` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with ADD_COLON_SINGLE style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_colon_single`。

### Lines 55-69: test case add colon two / 测试用例 add colon two
```python
    def test_add_colon_two(self):
        """Test prompt generation with ADD_COLON_TWO style (alternating separators)."""
        conv = Conversation(
            name="test",
            system_message="Sys",
            roles=("User", "Assistant"),
            messages=[["User", "Q"], ["Assistant", "A"], ["User", None]],
            sep_style=SeparatorStyle.ADD_COLON_TWO,
            sep="<s1>",
            sep2="<s2>",
        )
        prompt = conv.get_prompt()
        self.assertIn("User: Q<s1>", prompt)
        self.assertIn("Assistant: A<s2>", prompt)
        self.assertTrue(prompt.endswith("User:"))
```
**EN:** Test prompt generation with ADD_COLON_TWO style (alternating separators). This test exercises `test_add_colon_two` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with ADD_COLON_TWO style (alternating separators). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_colon_two`。

### Lines 71-87: test case chatml / 测试用例 chatml
```python
    def test_chatml(self):
        """Test prompt generation with CHATML style."""
        conv = Conversation(
            name="test",
            system_message="<|im_start|>system\nYou are helpful",
            roles=("<|im_start|>user", "<|im_start|>assistant"),
            messages=[
                ["<|im_start|>user", "Hello"],
                ["<|im_start|>assistant", None],
            ],
            sep_style=SeparatorStyle.CHATML,
            sep="<|im_end|>",
        )
        prompt = conv.get_prompt()
        self.assertIn("You are helpful<|im_end|>", prompt)
        self.assertIn("<|im_start|>user\nHello<|im_end|>", prompt)
        self.assertTrue(prompt.endswith("<|im_start|>assistant\n"))
```
**EN:** Test prompt generation with CHATML style. This test exercises `test_chatml` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with CHATML style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chatml`。

### Lines 89-105: test case llama3 / 测试用例 llama3
```python
    def test_llama3(self):
        """Test prompt generation with LLAMA3 style."""
        conv = Conversation(
            name="test",
            system_message="<|start_header_id|>system<|end_header_id|>\n\nBe helpful<|eot_id|>",
            roles=("user", "assistant"),
            messages=[["user", "Hi"], ["assistant", None]],
            sep_style=SeparatorStyle.LLAMA3,
        )
        prompt = conv.get_prompt()
        self.assertIn("Be helpful<|eot_id|>", prompt)
        self.assertIn(
            "<|start_header_id|>user<|end_header_id|>\n\nHi<|eot_id|>", prompt
        )
        self.assertTrue(
            prompt.endswith("<|start_header_id|>assistant<|end_header_id|>\n\n")
        )
```
**EN:** Test prompt generation with LLAMA3 style. This test exercises `test_llama3` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with LLAMA3 style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_llama3`。

### Lines 107-119: test case no colon single / 测试用例 no colon single
```python
    def test_no_colon_single(self):
        """Test prompt generation with NO_COLON_SINGLE style."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("[USER]", "[ASST]"),
            messages=[["[USER]", "Hello"], ["[ASST]", None]],
            sep_style=SeparatorStyle.NO_COLON_SINGLE,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("[USER]Hello\n", prompt)
        self.assertTrue(prompt.endswith("[ASST]"))
```
**EN:** Test prompt generation with NO_COLON_SINGLE style. This test exercises `test_no_colon_single` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with NO_COLON_SINGLE style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_colon_single`。

### Lines 121-132: test case none message in prompt / 测试用例 none message in prompt
```python
    def test_none_message_in_prompt(self):
        """Test that None message produces role-only output (no content)."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("User", "Assistant"),
            messages=[["User", "Q"], ["Assistant", None]],
            sep_style=SeparatorStyle.ADD_COLON_SINGLE,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertTrue(prompt.endswith("Assistant:"))
```
**EN:** Test that None message produces role-only output (no content). This test exercises `test_none_message_in_prompt` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that None message produces role-only output (no content). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_message_in_prompt`。

### Lines 134-144: test case empty system message / 测试用例 empty system message
```python
    def test_empty_system_message(self):
        """Test that empty system message produces empty prefix for LLAMA3."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("User", "Assistant"),
            messages=[["User", "Hello"], ["Assistant", None]],
            sep_style=SeparatorStyle.LLAMA3,
        )
        prompt = conv.get_prompt()
        self.assertNotIn("system", prompt.lower())
```
**EN:** Test that empty system message produces empty prefix for LLAMA3. This test exercises `test_empty_system_message` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that empty system message produces empty prefix for LLAMA3. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_empty_system_message`。

### Lines 146-159: test case add colon space single / 测试用例 add colon space single
```python
    def test_add_colon_space_single(self):
        """Test prompt generation with ADD_COLON_SPACE_SINGLE style."""
        conv = Conversation(
            name="test",
            system_message="Sys",
            roles=("User", "Bot"),
            messages=[["User", "Hi"], ["Bot", None]],
            sep_style=SeparatorStyle.ADD_COLON_SPACE_SINGLE,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("User: Hi\n", prompt)
        # None message should end with ": " (space after colon)
        self.assertTrue(prompt.endswith("Bot: "))
```
**EN:** Test prompt generation with ADD_COLON_SPACE_SINGLE style. This test exercises `test_add_colon_space_single` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with ADD_COLON_SPACE_SINGLE style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_colon_space_single`。

### Lines 161-173: test case add new line single / 测试用例 add new line single
```python
    def test_add_new_line_single(self):
        """Test prompt generation with ADD_NEW_LINE_SINGLE style."""
        conv = Conversation(
            name="test",
            system_message="Sys",
            roles=("User", "Bot"),
            messages=[["User", "Hi"], ["Bot", None]],
            sep_style=SeparatorStyle.ADD_NEW_LINE_SINGLE,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("User\nHi\n", prompt)
        self.assertTrue(prompt.endswith("Bot\n"))
```
**EN:** Test prompt generation with ADD_NEW_LINE_SINGLE style. This test exercises `test_add_new_line_single` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with ADD_NEW_LINE_SINGLE style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_new_line_single`。

### Lines 175-189: test case no colon two / 测试用例 no colon two
```python
    def test_no_colon_two(self):
        """Test prompt generation with NO_COLON_TWO style (alternating separators)."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("[U]", "[A]"),
            messages=[["[U]", "Q"], ["[A]", "A"], ["[U]", None]],
            sep_style=SeparatorStyle.NO_COLON_TWO,
            sep="<s1>",
            sep2="<s2>",
        )
        prompt = conv.get_prompt()
        self.assertIn("[U]Q<s1>", prompt)
        self.assertIn("[A]A<s2>", prompt)
        self.assertTrue(prompt.endswith("[U]"))
```
**EN:** Test prompt generation with NO_COLON_TWO style (alternating separators). This test exercises `test_no_colon_two` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with NO_COLON_TWO style (alternating separators). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_colon_two`。

### Lines 191-205: test case llama2 with system / 测试用例 llama2 with system
```python
    def test_llama2_with_system(self):
        """Test LLAMA2 with system message."""
        conv = Conversation(
            name="test",
            system_message="<<SYS>>\nBe helpful\n<</SYS>>\n\n",
            system_template="[INST] {system_message}",
            roles=("[INST]", "[/INST]"),
            messages=[["[INST]", "Hi"], ["[/INST]", None]],
            sep_style=SeparatorStyle.LLAMA2,
            sep=" ",
            sep2=" </s><s>",
        )
        prompt = conv.get_prompt()
        self.assertIn("Be helpful", prompt)
        self.assertIn("Hi ", prompt)
```
**EN:** Test LLAMA2 with system message. This test exercises `test_llama2_with_system` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LLAMA2 with system message. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_llama2_with_system`。

### Lines 207-219: test case llama2 without system / 测试用例 llama2 without system
```python
    def test_llama2_without_system(self):
        """Test LLAMA2 without system message falls back to '[INST] ' prefix."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("[INST]", "[/INST]"),
            messages=[["[INST]", "Hi"], ["[/INST]", None]],
            sep_style=SeparatorStyle.LLAMA2,
            sep=" ",
            sep2=" </s><s>",
        )
        prompt = conv.get_prompt()
        self.assertTrue(prompt.startswith("[INST] Hi"))
```
**EN:** Test LLAMA2 without system message falls back to '[INST] ' prefix. This test exercises `test_llama2_without_system` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LLAMA2 without system message falls back to '[INST] ' prefix. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_llama2_without_system`。

### Lines 221-242: test case llama2 multi turn / 测试用例 llama2 multi turn
```python
    def test_llama2_multi_turn(self):
        """Test LLAMA2 with multi-turn (i>0 uses tag+sep pattern)."""
        conv = Conversation(
            name="test",
            system_message="<<SYS>>\nSys\n<</SYS>>\n\n",
            system_template="[INST] {system_message}",
            roles=("[INST]", "[/INST]"),
            messages=[
                ["[INST]", "Q1"],
                ["[/INST]", "A1"],
                ["[INST]", "Q2"],
                ["[/INST]", None],
            ],
            sep_style=SeparatorStyle.LLAMA2,
            sep=" ",
            sep2=" </s><s>",
        )
        prompt = conv.get_prompt()
        # i=0: message + " " (no tag prefix)
        self.assertIn("Q1 ", prompt)
        # i=1: tag + " " + message + sep2
        self.assertIn("[/INST] A1 </s><s>", prompt)
```
**EN:** Test LLAMA2 with multi-turn (i>0 uses tag+sep pattern). This test exercises `test_llama2_multi_turn` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LLAMA2 with multi-turn (i>0 uses tag+sep pattern). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_llama2_multi_turn`。

### Lines 244-257: test case llama4 / 测试用例 llama4
```python
    def test_llama4(self):
        """Test prompt generation with LLAMA4 style."""
        conv = Conversation(
            name="test",
            system_message="Be helpful",
            system_template="{system_message}",
            roles=("user", "assistant"),
            messages=[["user", "Hello"], ["assistant", None]],
            sep_style=SeparatorStyle.LLAMA4,
        )
        prompt = conv.get_prompt()
        self.assertIn("Be helpful", prompt)
        self.assertIn("<|header_start|>user<|header_end|>", prompt)
        self.assertIn("Hello<|eot|>", prompt)
```
**EN:** Test prompt generation with LLAMA4 style. This test exercises `test_llama4` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with LLAMA4 style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_llama4`。

### Lines 259-269: test case llama4 empty system / 测试用例 llama4 empty system
```python
    def test_llama4_empty_system(self):
        """Test LLAMA4 with empty system message omits system prefix."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("user", "assistant"),
            messages=[["user", "Hello"], ["assistant", None]],
            sep_style=SeparatorStyle.LLAMA4,
        )
        prompt = conv.get_prompt()
        self.assertTrue(prompt.startswith("<|header_start|>user"))
```
**EN:** Test LLAMA4 with empty system message omits system prefix. This test exercises `test_llama4_empty_system` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test LLAMA4 with empty system message omits system prefix. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_llama4_empty_system`。

### Lines 271-283: test case chatglm3 / 测试用例 chatglm3
```python
    def test_chatglm3(self):
        """Test prompt generation with CHATGLM3 style."""
        conv = Conversation(
            name="test",
            system_message="<|system|>\nBe helpful",
            roles=("<|user|>", "<|assistant|>"),
            messages=[["<|user|>", "Hi"], ["<|assistant|>", None]],
            sep_style=SeparatorStyle.CHATGLM3,
        )
        prompt = conv.get_prompt()
        self.assertIn("Be helpful", prompt)
        self.assertIn("<|user|>\nHi", prompt)
        self.assertTrue(prompt.endswith("<|assistant|>"))
```
**EN:** Test prompt generation with CHATGLM3 style. This test exercises `test_chatglm3` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with CHATGLM3 style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chatglm3`。

### Lines 285-299: test case deepseek chat / 测试用例 deepseek chat
```python
    def test_deepseek_chat(self):
        """Test prompt generation with DEEPSEEK_CHAT style."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("User", "Assistant"),
            messages=[["User", "Q"], ["Assistant", "A"], ["User", None]],
            sep_style=SeparatorStyle.DEEPSEEK_CHAT,
            sep="\n\n",
            sep2="<end>",
        )
        prompt = conv.get_prompt()
        self.assertIn("User: Q\n\n", prompt)
        self.assertIn("Assistant: A<end>", prompt)
        self.assertTrue(prompt.endswith("User:"))
```
**EN:** Test prompt generation with DEEPSEEK_CHAT style. This test exercises `test_deepseek_chat` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with DEEPSEEK_CHAT style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_chat`。

### Lines 301-313: test case robin / 测试用例 robin
```python
    def test_robin(self):
        """Test prompt generation with ROBIN style."""
        conv = Conversation(
            name="test",
            system_message="Sys",
            roles=("###Human", "###Assistant"),
            messages=[["###Human", "Hi"], ["###Assistant", None]],
            sep_style=SeparatorStyle.ROBIN,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("###Human:\nHi\n", prompt)
        self.assertTrue(prompt.endswith("###Assistant:\n"))
```
**EN:** Test prompt generation with ROBIN style. This test exercises `test_robin` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with ROBIN style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_robin`。

### Lines 315-328: test case falcon chat / 测试用例 falcon chat
```python
    def test_falcon_chat(self):
        """Test prompt generation with FALCON_CHAT style."""
        conv = Conversation(
            name="test",
            system_message="System prompt.",
            roles=("User", "Falcon"),
            messages=[["User", "Hi"], ["Falcon", None]],
            sep_style=SeparatorStyle.FALCON_CHAT,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("System prompt.\n", prompt)
        self.assertIn("User: Hi\n", prompt)
        self.assertTrue(prompt.endswith("Falcon:"))
```
**EN:** Test prompt generation with FALCON_CHAT style. This test exercises `test_falcon_chat` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with FALCON_CHAT style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_falcon_chat`。

### Lines 330-343: test case metamath / 测试用例 metamath
```python
    def test_metamath(self):
        """Test prompt generation with METAMATH style."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("Query", "Response"),
            messages=[["Query", "2+2?"], ["Response", None]],
            sep_style=SeparatorStyle.METAMATH,
            sep="\n",
            sep2="Let's think step by step.\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("Query:\n2+2?\n", prompt)
        self.assertIn("Response: Let's think step by step.\n", prompt)
```
**EN:** Test prompt generation with METAMATH style. This test exercises `test_metamath` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with METAMATH style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_metamath`。

### Lines 345-357: test case mpt / 测试用例 mpt
```python
    def test_mpt(self):
        """Test prompt generation with MPT style."""
        conv = Conversation(
            name="test",
            system_message="<|system|>",
            roles=("<|user|>", "<|assistant|>"),
            messages=[["<|user|>", "Hi"], ["<|assistant|>", None]],
            sep_style=SeparatorStyle.MPT,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("<|user|>Hi\n", prompt)
        self.assertTrue(prompt.endswith("<|assistant|>"))
```
**EN:** Test prompt generation with MPT style. This test exercises `test_mpt` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with MPT style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mpt`。

### Lines 359-372: test case chatintern / 测试用例 chatintern
```python
    def test_chatintern(self):
        """Test prompt generation with CHATINTERN style."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("HUMAN", "BOT"),
            messages=[["HUMAN", "Hi"], ["BOT", "Hello"], ["HUMAN", None]],
            sep_style=SeparatorStyle.CHATINTERN,
            sep="\n",
            sep2="</s>",
        )
        prompt = conv.get_prompt()
        self.assertIn("<s>HUMAN:Hi\n", prompt)
        self.assertIn("BOT:Hello</s>", prompt)
```
**EN:** Test prompt generation with CHATINTERN style. This test exercises `test_chatintern` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with CHATINTERN style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chatintern`。

### Lines 374-388: test case dolly / 测试用例 dolly
```python
    def test_dolly(self):
        """Test prompt generation with DOLLY style."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("Instruction", "Response"),
            messages=[["Instruction", "Q"], ["Response", "A"], ["Instruction", None]],
            sep_style=SeparatorStyle.DOLLY,
            sep="\n\n",
            sep2="</s>",
        )
        prompt = conv.get_prompt()
        self.assertIn("Instruction:\nQ\n\n", prompt)
        self.assertIn("Response:\nA</s>", prompt)
        self.assertTrue(prompt.endswith("Instruction:\n"))
```
**EN:** Test prompt generation with DOLLY style. This test exercises `test_dolly` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with DOLLY style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dolly`。

### Lines 390-401: test case phoenix / 测试用例 phoenix
```python
    def test_phoenix(self):
        """Test prompt generation with PHOENIX style."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("Human", "Phoenix"),
            messages=[["Human", "Hi"], ["Phoenix", None]],
            sep_style=SeparatorStyle.PHOENIX,
        )
        prompt = conv.get_prompt()
        self.assertIn("Human: <s>Hi</s>", prompt)
        self.assertTrue(prompt.endswith("Phoenix: <s>"))
```
**EN:** Test prompt generation with PHOENIX style. This test exercises `test_phoenix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with PHOENIX style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_phoenix`。

### Lines 403-417: test case deepseek vl2 / 测试用例 deepseek vl2
```python
    def test_deepseek_vl2(self):
        """Test prompt generation with DeepSeekVL2 style."""
        conv = Conversation(
            name="test",
            system_message="Sys",
            roles=("User", "Assistant"),
            messages=[["User", "Q"], ["Assistant", None]],
            sep_style=SeparatorStyle.DeepSeekVL2,
            sep="\n",
            sep2="<end>",
        )
        prompt = conv.get_prompt()
        self.assertIn("Sys\n", prompt)
        self.assertIn("User: Q\n", prompt)
        self.assertTrue(prompt.endswith("Assistant:"))
```
**EN:** Test prompt generation with DeepSeekVL2 style. This test exercises `test_deepseek_vl2` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with DeepSeekVL2 style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_vl2`。

### Lines 419-431: test case deepseek vl2 empty system / 测试用例 deepseek vl2 empty system
```python
    def test_deepseek_vl2_empty_system(self):
        """Test DeepSeekVL2 with empty system message omits system prefix."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("User", "Assistant"),
            messages=[["User", "Q"], ["Assistant", None]],
            sep_style=SeparatorStyle.DeepSeekVL2,
            sep="\n",
            sep2="<end>",
        )
        prompt = conv.get_prompt()
        self.assertTrue(prompt.startswith("User: Q"))
```
**EN:** Test DeepSeekVL2 with empty system message omits system prefix. This test exercises `test_deepseek_vl2_empty_system` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test DeepSeekVL2 with empty system message omits system prefix. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_vl2_empty_system`。

### Lines 433-447: test case gemma3 / 测试用例 gemma3
```python
    def test_gemma3(self):
        """Test prompt generation with GEMMA3 style (first message special)."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("<start>", "<model>"),
            messages=[["<start>", "Hello"], ["<model>", "Hi"], ["<start>", None]],
            sep_style=SeparatorStyle.GEMMA3,
            sep="<end>",
        )
        prompt = conv.get_prompt()
        # First message: no role prefix, just message + sep
        self.assertTrue(prompt.startswith("Hello<end>"))
        # Subsequent: role + message + sep
        self.assertIn("<model>Hi<end>", prompt)
```
**EN:** Test prompt generation with GEMMA3 style (first message special). This test exercises `test_gemma3` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with GEMMA3 style (first message special). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_gemma3`。

### Lines 449-460: test case rwkv / 测试用例 rwkv
```python
    def test_rwkv(self):
        """Test prompt generation with RWKV style (newline replacement)."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("Bob", "Alice"),
            messages=[["Bob", "Hello\n\nWorld"], ["Alice", None]],
            sep_style=SeparatorStyle.RWKV,
        )
        prompt = conv.get_prompt()
        # RWKV replaces \n\n with \n in message
        self.assertIn("Bob: Hello\nWorld\n\n", prompt)
```
**EN:** Test prompt generation with RWKV style (newline replacement). This test exercises `test_rwkv` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with RWKV style (newline replacement). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_rwkv`。

### Lines 462-475: test case qwen2 vl embed / 测试用例 qwen2 vl embed
```python
    def test_qwen2_vl_embed(self):
        """Test prompt generation with QWEN2_VL_EMBED style."""
        conv = Conversation(
            name="test",
            system_message="Sys",
            roles=("user", "assistant"),
            messages=[["user", "Hi"], ["assistant", None]],
            sep_style=SeparatorStyle.QWEN2_VL_EMBED,
            sep="\n",
            stop_str="<|endoftext|>",
        )
        prompt = conv.get_prompt()
        self.assertIn("user\nHi\n", prompt)
        self.assertTrue(prompt.endswith("<|endoftext|>"))
```
**EN:** Test prompt generation with QWEN2_VL_EMBED style. This test exercises `test_qwen2_vl_embed` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with QWEN2_VL_EMBED style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen2_vl_embed`。

### Lines 477-491: test case chatglm / 测试用例 chatglm
```python
    def test_chatglm(self):
        """Test prompt generation with CHATGLM style (round numbering)."""
        conv = Conversation(
            name="chatglm",
            system_message="",
            roles=("问", "答"),
            messages=[["问", "Hello"], ["答", "Hi"], ["问", None]],
            sep_style=SeparatorStyle.CHATGLM,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("[Round 0]\n", prompt)
        self.assertIn("问：Hello\n", prompt)
        self.assertIn("答：Hi\n", prompt)
        self.assertTrue(prompt.endswith("问："))
```
**EN:** Test prompt generation with CHATGLM style (round numbering). This test exercises `test_chatglm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with CHATGLM style (round numbering). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chatglm`。

### Lines 493-504: test case chatglm2 round offset / 测试用例 chatglm2 round offset
```python
    def test_chatglm2_round_offset(self):
        """Test CHATGLM style with chatglm2 name (round starts at 1 instead of 0)."""
        conv = Conversation(
            name="chatglm2",
            system_message="",
            roles=("问", "答"),
            messages=[["问", "Hello"], ["答", None]],
            sep_style=SeparatorStyle.CHATGLM,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("[Round 1]\n", prompt)
```
**EN:** Test CHATGLM style with chatglm2 name (round starts at 1 instead of 0). This test exercises `test_chatglm2_round_offset` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test CHATGLM style with chatglm2 name (round starts at 1 instead of 0). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chatglm2_round_offset`。

### Lines 506-517: test case chatglm with system / 测试用例 chatglm with system
```python
    def test_chatglm_with_system(self):
        """Test CHATGLM with non-empty system message."""
        conv = Conversation(
            name="chatglm",
            system_message="You are helpful",
            roles=("问", "答"),
            messages=[["问", "Hi"], ["答", None]],
            sep_style=SeparatorStyle.CHATGLM,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertTrue(prompt.startswith("You are helpful\n"))
```
**EN:** Test CHATGLM with non-empty system message. This test exercises `test_chatglm_with_system` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test CHATGLM with non-empty system message. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chatglm_with_system`。

### Lines 519-537: test case qwen2 audio / 测试用例 qwen2 audio
```python
    def test_qwen2_audio(self):
        """Test QWEN2_AUDIO style with audio token counter replacement."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("user", "assistant"),
            messages=[
                ["user", "Listen: <audio>{idx}</audio> and <audio>{idx}</audio>"],
                ["assistant", None],
            ],
            sep_style=SeparatorStyle.QWEN2_AUDIO,
            sep="\n",
            audio_token="<audio>{idx}</audio>",
        )
        prompt = conv.get_prompt()
        # Audio tokens should be replaced with counter: idx=1, idx=2
        self.assertIn("<audio>1</audio>", prompt)
        self.assertIn("<audio>2</audio>", prompt)
        self.assertNotIn("{idx}", prompt)
```
**EN:** Test QWEN2_AUDIO style with audio token counter replacement. This test exercises `test_qwen2_audio` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test QWEN2_AUDIO style with audio token counter replacement. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_qwen2_audio`。

### Lines 539-551: test case paddle ocr / 测试用例 paddle ocr
```python
    def test_paddle_ocr(self):
        """Test prompt generation with PADDLE_OCR style."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("USER", "ASSISTANT"),
            messages=[["USER", "Describe image"], ["ASSISTANT", None]],
            sep_style=SeparatorStyle.PADDLE_OCR,
            sep="<eos>",
        )
        prompt = conv.get_prompt()
        self.assertIn("USER: Describe image", prompt)
        self.assertTrue(prompt.endswith("ASSISTANT: "))
```
**EN:** Test prompt generation with PADDLE_OCR style. This test exercises `test_paddle_ocr` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test prompt generation with PADDLE_OCR style. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_paddle_ocr`。

### Lines 553-570: test case paddle ocr with image token / 测试用例 paddle ocr with image token
```python
    def test_paddle_ocr_with_image_token(self):
        """Test PADDLE_OCR strips newline after image token for USER role."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("USER", "ASSISTANT"),
            messages=[
                ["USER", "<image>\nDescribe this"],
                ["ASSISTANT", "It shows a cat"],
            ],
            sep_style=SeparatorStyle.PADDLE_OCR,
            sep="<eos>",
            image_token="<image>",
        )
        prompt = conv.get_prompt()
        # image_token + "\n" should be replaced with just image_token
        self.assertIn("USER: <image>Describe this\n", prompt)
        self.assertIn("ASSISTANT: It shows a cat<eos>", prompt)
```
**EN:** Test PADDLE_OCR strips newline after image token for USER role. This test exercises `test_paddle_ocr_with_image_token` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test PADDLE_OCR strips newline after image token for USER role. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_paddle_ocr_with_image_token`。

### Lines 572-587: test case mpt with tuple message / 测试用例 mpt with tuple message
```python
    def test_mpt_with_tuple_message(self):
        """Test MPT style extracts first element from tuple messages."""
        conv = Conversation(
            name="test",
            system_message="<|system|>",
            roles=("<|user|>", "<|assistant|>"),
            messages=[
                ["<|user|>", ("Hello", "extra1", "extra2")],
                ["<|assistant|>", None],
            ],
            sep_style=SeparatorStyle.MPT,
            sep="\n",
        )
        prompt = conv.get_prompt()
        self.assertIn("<|user|>Hello\n", prompt)
        self.assertNotIn("extra1", prompt)
```
**EN:** Test MPT style extracts first element from tuple messages. This test exercises `test_mpt_with_tuple_message` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test MPT style extracts first element from tuple messages. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_mpt_with_tuple_message`。

### Lines 589-600: test case invalid sep style raises / 测试用例 invalid sep style raises
```python
    def test_invalid_sep_style_raises(self):
        """Test that an invalid SeparatorStyle raises ValueError."""
        conv = Conversation(
            name="test",
            system_message="",
            roles=("A", "B"),
            messages=[["A", "Hi"]],
            sep_style=999,
            sep="\n",
        )
        with self.assertRaises(ValueError):
            conv.get_prompt()
```
**EN:** Test that an invalid SeparatorStyle raises ValueError. This test exercises `test_invalid_sep_style_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that an invalid SeparatorStyle raises ValueError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_sep_style_raises`。

### Lines 603-603: class TestConversationMethods declaration / 类 TestConversationMethods 声明
```python
class TestConversationMethods(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 604-611: method make conv / 方法 make conv
```python
    def _make_conv(self):
        return Conversation(
            name="test",
            roles=("User", "Assistant"),
            messages=[],
            sep_style=SeparatorStyle.ADD_COLON_SINGLE,
            sep="\n",
        )
```
**EN:** This block implements `_make_conv` and captures one focused piece of the module's behavior.
**CN:** 该代码块实现 `_make_conv`，承担模块行为中的一个聚焦逻辑片段。

### Lines 613-619: test case append message / 测试用例 append message
```python
    def test_append_message(self):
        """Test appending messages to conversation."""
        conv = self._make_conv()
        conv.append_message("User", "Hello")
        conv.append_message("Assistant", "Hi")
        self.assertEqual(len(conv.messages), 2)
        self.assertEqual(conv.messages[0], ["User", "Hello"])
```
**EN:** Test appending messages to conversation. This test exercises `test_append_message` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test appending messages to conversation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_append_message`。

### Lines 621-625: test case set system message / 测试用例 set system message
```python
    def test_set_system_message(self):
        """Test setting the system message."""
        conv = self._make_conv()
        conv.set_system_message("Be helpful")
        self.assertEqual(conv.system_message, "Be helpful")
```
**EN:** Test setting the system message. This test exercises `test_set_system_message` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test setting the system message. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_set_system_message`。

### Lines 627-633: test case update last message / 测试用例 update last message
```python
    def test_update_last_message(self):
        """Test updating the last message in-place."""
        conv = self._make_conv()
        conv.append_message("User", "Q")
        conv.append_message("Assistant", None)
        conv.update_last_message("Answer")
        self.assertEqual(conv.messages[-1][1], "Answer")
```
**EN:** Test updating the last message in-place. This test exercises `test_update_last_message` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test updating the last message in-place. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_update_last_message`。

### Lines 635-644: test case to openai api messages with system / 测试用例 to openai api messages with system
```python
    def test_to_openai_api_messages_with_system(self):
        """Test conversion to OpenAI format with system message."""
        conv = self._make_conv()
        conv.system_message = "Be helpful"
        conv.append_message("User", "Hello")
        conv.append_message("Assistant", "Hi")
        result = conv.to_openai_api_messages()
        self.assertEqual(result[0], {"role": "system", "content": "Be helpful"})
        self.assertEqual(result[1], {"role": "user", "content": "Hello"})
        self.assertEqual(result[2], {"role": "assistant", "content": "Hi"})
```
**EN:** Test conversion to OpenAI format with system message. This test exercises `test_to_openai_api_messages_with_system` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test conversion to OpenAI format with system message. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_openai_api_messages_with_system`。

### Lines 646-652: test case to openai api messages without system / 测试用例 to openai api messages without system
```python
    def test_to_openai_api_messages_without_system(self):
        """Test conversion to OpenAI format without system message."""
        conv = self._make_conv()
        conv.append_message("User", "Hello")
        result = conv.to_openai_api_messages()
        self.assertEqual(len(result), 1)
        self.assertEqual(result[0]["role"], "user")
```
**EN:** Test conversion to OpenAI format without system message. This test exercises `test_to_openai_api_messages_without_system` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test conversion to OpenAI format without system message. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_openai_api_messages_without_system`。

### Lines 654-660: test case to openai api messages skips none assistant / 测试用例 to openai api messages skips none assistant
```python
    def test_to_openai_api_messages_skips_none_assistant(self):
        """Test that None assistant message is omitted from OpenAI format."""
        conv = self._make_conv()
        conv.append_message("User", "Hello")
        conv.append_message("Assistant", None)
        result = conv.to_openai_api_messages()
        self.assertEqual(len(result), 1)  # only user message
```
**EN:** Test that None assistant message is omitted from OpenAI format. This test exercises `test_to_openai_api_messages_skips_none_assistant` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that None assistant message is omitted from OpenAI format. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_openai_api_messages_skips_none_assistant`。

### Lines 662-672: test case to gradio chatbot / 测试用例 to gradio chatbot
```python
    def test_to_gradio_chatbot(self):
        """Test conversion to Gradio chatbot format (user/assistant pairs)."""
        conv = self._make_conv()
        conv.append_message("User", "Q1")
        conv.append_message("Assistant", "A1")
        conv.append_message("User", "Q2")
        conv.append_message("Assistant", "A2")
        result = conv.to_gradio_chatbot()
        self.assertEqual(len(result), 2)
        self.assertEqual(result[0], ["Q1", "A1"])
        self.assertEqual(result[1], ["Q2", "A2"])
```
**EN:** Test conversion to Gradio chatbot format (user/assistant pairs). This test exercises `test_to_gradio_chatbot` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test conversion to Gradio chatbot format (user/assistant pairs). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_gradio_chatbot`。

### Lines 674-680: test case to gradio chatbot pending response / 测试用例 to gradio chatbot pending response
```python
    def test_to_gradio_chatbot_pending_response(self):
        """Test Gradio format with pending assistant response (None)."""
        conv = self._make_conv()
        conv.append_message("User", "Q1")
        conv.append_message("Assistant", None)
        result = conv.to_gradio_chatbot()
        self.assertEqual(result, [["Q1", None]])
```
**EN:** Test Gradio format with pending assistant response (None). This test exercises `test_to_gradio_chatbot_pending_response` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test Gradio format with pending assistant response (None). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_to_gradio_chatbot_pending_response`。

### Lines 682-689: test case append image / 测试用例 append image
```python
    def test_append_image(self):
        """Test appending image data to conversation."""
        conv = self._make_conv()
        conv.image_data = []
        conv.append_image("http://example.com/img.jpg", "auto")
        self.assertEqual(len(conv.image_data), 1)
        self.assertEqual(conv.image_data[0].url, "http://example.com/img.jpg")
        self.assertEqual(conv.image_data[0].detail, "auto")
```
**EN:** Test appending image data to conversation. This test exercises `test_append_image` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test appending image data to conversation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_append_image`。

### Lines 691-697: test case append video / 测试用例 append video
```python
    def test_append_video(self):
        """Test appending video data to conversation."""
        conv = self._make_conv()
        conv.video_data = []
        conv.append_video("http://example.com/vid.mp4")
        self.assertEqual(len(conv.video_data), 1)
        self.assertEqual(conv.video_data[0], "http://example.com/vid.mp4")
```
**EN:** Test appending video data to conversation. This test exercises `test_append_video` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test appending video data to conversation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_append_video`。

### Lines 699-705: test case append audio / 测试用例 append audio
```python
    def test_append_audio(self):
        """Test appending audio data to conversation."""
        conv = self._make_conv()
        conv.audio_data = []
        conv.append_audio("http://example.com/audio.wav")
        self.assertEqual(len(conv.audio_data), 1)
        self.assertEqual(conv.audio_data[0], "http://example.com/audio.wav")
```
**EN:** Test appending audio data to conversation. This test exercises `test_append_audio` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test appending audio data to conversation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_append_audio`。

### Lines 707-714: test case copy is independent / 测试用例 copy is independent
```python
    def test_copy_is_independent(self):
        """Test that copy() creates an independent conversation."""
        conv = self._make_conv()
        conv.append_message("User", "Hello")
        copied = conv.copy()
        copied.append_message("Assistant", "Hi")
        self.assertEqual(len(conv.messages), 1)
        self.assertEqual(len(copied.messages), 2)
```
**EN:** Test that copy() creates an independent conversation. This test exercises `test_copy_is_independent` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that copy() creates an independent conversation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_copy_is_independent`。

### Lines 716-723: test case dict serialization / 测试用例 dict serialization
```python
    def test_dict_serialization(self):
        """Test dict() returns expected keys."""
        conv = self._make_conv()
        conv.append_message("User", "Hello")
        d = conv.dict()
        self.assertEqual(d["template_name"], "test")
        self.assertIn("messages", d)
        self.assertIn("roles", d)
```
**EN:** Test dict() returns expected keys. This test exercises `test_dict_serialization` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test dict() returns expected keys. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_dict_serialization`。

### Lines 726-726: class TestTemplateRegistry declaration / 类 TestTemplateRegistry 声明
```python
class TestTemplateRegistry(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 727-730: test case builtin templates exist / 测试用例 builtin templates exist
```python
    def test_builtin_templates_exist(self):
        """Test that common built-in templates are registered."""
        self.assertTrue(chat_template_exists("chatml"))
        self.assertTrue(chat_template_exists("llama-2"))
```
**EN:** Test that common built-in templates are registered. This test exercises `test_builtin_templates_exist` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that common built-in templates are registered. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_builtin_templates_exist`。

### Lines 732-734: test case unregistered template not found / 测试用例 unregistered template not found
```python
    def test_unregistered_template_not_found(self):
        """Test that non-existent template returns False."""
        self.assertFalse(chat_template_exists("_nonexistent_template_xyz"))
```
**EN:** Test that non-existent template returns False. This test exercises `test_unregistered_template_not_found` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that non-existent template returns False. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unregistered_template_not_found`。

### Lines 736-748: test case register and lookup / 测试用例 register and lookup
```python
    def test_register_and_lookup(self):
        """Test registering and looking up a custom template."""
        t = Conversation(
            name="_test_conv_template",
            roles=("A", "B"),
            messages=[],
            sep_style=SeparatorStyle.ADD_COLON_SINGLE,
            sep="\n",
        )
        register_conv_template(t)
        self.assertTrue(chat_template_exists("_test_conv_template"))
        # Cleanup
        del chat_templates["_test_conv_template"]
```
**EN:** Test registering and looking up a custom template. This test exercises `test_register_and_lookup` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test registering and looking up a custom template. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_and_lookup`。

### Lines 750-761: test case register duplicate raises / 测试用例 register duplicate raises
```python
    def test_register_duplicate_raises(self):
        """Test that registering a duplicate name without override raises."""
        with self.assertRaises(AssertionError):
            register_conv_template(
                Conversation(
                    name="chatml",
                    roles=("A", "B"),
                    messages=[],
                    sep_style=SeparatorStyle.CHATML,
                    sep="",
                )
            )
```
**EN:** Test that registering a duplicate name without override raises. This test exercises `test_register_duplicate_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that registering a duplicate name without override raises. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_register_duplicate_raises`。

### Lines 763-766: test case get conv template by model path returns none for unknown / 测试用例 get conv template by model path returns none for unknown
```python
    def test_get_conv_template_by_model_path_returns_none_for_unknown(self):
        """Test that unknown model path returns None."""
        result = get_conv_template_by_model_path("totally-unknown-model-xyz")
        self.assertIsNone(result)
```
**EN:** Test that unknown model path returns None. This test exercises `test_get_conv_template_by_model_path_returns_none_for_unknown` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that unknown model path returns None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_returns_none_for_unknown`。

### Lines 768-771: test case get conv template by model path vicuna / 测试用例 get conv template by model path vicuna
```python
    def test_get_conv_template_by_model_path_vicuna(self):
        """Test that vicuna model path is matched correctly."""
        result = get_conv_template_by_model_path("lmsys/vicuna-7b-v1.5")
        self.assertEqual(result, "vicuna_v1.1")
```
**EN:** Test that vicuna model path is matched correctly. This test exercises `test_get_conv_template_by_model_path_vicuna` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that vicuna model path is matched correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_vicuna`。

### Lines 773-776: test case get conv template by model path internvl / 测试用例 get conv template by model path internvl
```python
    def test_get_conv_template_by_model_path_internvl(self):
        """Test that internvl model path is matched correctly."""
        result = get_conv_template_by_model_path("OpenGVLab/InternVL2-8B")
        self.assertEqual(result, "internvl-2-5")
```
**EN:** Test that internvl model path is matched correctly. This test exercises `test_get_conv_template_by_model_path_internvl` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that internvl model path is matched correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_internvl`。

### Lines 778-781: test case get conv template by model path deepseek vl2 / 测试用例 get conv template by model path deepseek vl2
```python
    def test_get_conv_template_by_model_path_deepseek_vl2(self):
        """Test that deepseek-vl2 model path is matched correctly."""
        result = get_conv_template_by_model_path("deepseek-ai/deepseek-vl2")
        self.assertEqual(result, "deepseek-vl2")
```
**EN:** Test that deepseek-vl2 model path is matched correctly. This test exercises `test_get_conv_template_by_model_path_deepseek_vl2` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that deepseek-vl2 model path is matched correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_deepseek_vl2`。

### Lines 783-786: test case get conv template by model path whisper / 测试用例 get conv template by model path whisper
```python
    def test_get_conv_template_by_model_path_whisper(self):
        """Test that whisper model path is matched correctly."""
        result = get_conv_template_by_model_path("openai/whisper-large-v3")
        self.assertEqual(result, "whisper")
```
**EN:** Test that whisper model path is matched correctly. This test exercises `test_get_conv_template_by_model_path_whisper` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that whisper model path is matched correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_whisper`。

### Lines 788-791: test case get conv template by model path janus / 测试用例 get conv template by model path janus
```python
    def test_get_conv_template_by_model_path_janus(self):
        """Test that janus model path is matched correctly."""
        result = get_conv_template_by_model_path("deepseek-ai/Janus-Pro-7B")
        self.assertEqual(result, "janus-pro")
```
**EN:** Test that janus model path is matched correctly. This test exercises `test_get_conv_template_by_model_path_janus` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that janus model path is matched correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_janus`。

### Lines 793-796: test case get conv template by model path phi4 mm / 测试用例 get conv template by model path phi4 mm
```python
    def test_get_conv_template_by_model_path_phi4_mm(self):
        """Test that phi-4-multimodal model path is matched correctly."""
        result = get_conv_template_by_model_path("microsoft/phi-4-multimodal")
        self.assertEqual(result, "phi-4-mm")
```
**EN:** Test that phi-4-multimodal model path is matched correctly. This test exercises `test_get_conv_template_by_model_path_phi4_mm` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that phi-4-multimodal model path is matched correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_phi4_mm`。

### Lines 798-801: test case get conv template by model path llava next / 测试用例 get conv template by model path llava next
```python
    def test_get_conv_template_by_model_path_llava_next(self):
        """Test that llava-next-video-34b model path returns chatml-llava."""
        result = get_conv_template_by_model_path("llava-hf/llava-next-video-34b")
        self.assertEqual(result, "chatml-llava")
```
**EN:** Test that llava-next-video-34b model path returns chatml-llava. This test exercises `test_get_conv_template_by_model_path_llava_next` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that llava-next-video-34b model path returns chatml-llava. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_llava_next`。

### Lines 803-806: test case get conv template by model path paddle ocr / 测试用例 get conv template by model path paddle ocr
```python
    def test_get_conv_template_by_model_path_paddle_ocr(self):
        """Test that paddleocr model path is matched correctly."""
        result = get_conv_template_by_model_path("PaddleOCR/PaddleOCR-2.9")
        self.assertEqual(result, "paddle-ocr")
```
**EN:** Test that paddleocr model path is matched correctly. This test exercises `test_get_conv_template_by_model_path_paddle_ocr` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that paddleocr model path is matched correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_paddle_ocr`。

### Lines 808-811: test case get conv template by model path deepseek ocr / 测试用例 get conv template by model path deepseek ocr
```python
    def test_get_conv_template_by_model_path_deepseek_ocr(self):
        """Test that deepseek-ocr model path is matched correctly."""
        result = get_conv_template_by_model_path("deepseek-ai/deepseek-ocr-base")
        self.assertEqual(result, "deepseek-ocr")
```
**EN:** Test that deepseek-ocr model path is matched correctly. This test exercises `test_get_conv_template_by_model_path_deepseek_ocr` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that deepseek-ocr model path is matched correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_deepseek_ocr`。

### Lines 813-816: test case get conv template by model path points / 测试用例 get conv template by model path points
```python
    def test_get_conv_template_by_model_path_points(self):
        """Test that points model path is matched correctly."""
        result = get_conv_template_by_model_path("WePOINTS/points-v1.5")
        self.assertEqual(result, "points-v15-chat")
```
**EN:** Test that points model path is matched correctly. This test exercises `test_get_conv_template_by_model_path_points` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that points model path is matched correctly. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_points`。

### Lines 818-821: test case get conv template by model path minicpm v / 测试用例 get conv template by model path minicpm v
```python
    def test_get_conv_template_by_model_path_minicpm_v(self):
        """Test that minicpm-v model path returns minicpmv."""
        result = get_conv_template_by_model_path("openbmb/MiniCPM-V-2_6")
        self.assertEqual(result, "minicpmv")
```
**EN:** Test that minicpm-v model path returns minicpmv. This test exercises `test_get_conv_template_by_model_path_minicpm_v` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that minicpm-v model path returns minicpmv. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_minicpm_v`。

### Lines 823-826: test case get conv template by model path minicpm o / 测试用例 get conv template by model path minicpm o
```python
    def test_get_conv_template_by_model_path_minicpm_o(self):
        """Test that minicpm-o model path returns minicpmo."""
        result = get_conv_template_by_model_path("openbmb/MiniCPM-o-2_6")
        self.assertEqual(result, "minicpmo")
```
**EN:** Test that minicpm-o model path returns minicpmo. This test exercises `test_get_conv_template_by_model_path_minicpm_o` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that minicpm-o model path returns minicpmo. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_get_conv_template_by_model_path_minicpm_o`。

### Lines 829-829: class TestGenerateEmbeddingConvs declaration / 类 TestGenerateEmbeddingConvs 声明
```python
class TestGenerateEmbeddingConvs(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 830-841: test case text only / 测试用例 text only
```python
    def test_text_only(self):
        """Test generating embedding conversations with text only."""
        convs = generate_embedding_convs(
            texts=["Hello world"],
            images=[None],
            videos=[None],
            template_name="chatml",
        )
        self.assertEqual(len(convs), 1)
        self.assertEqual(len(convs[0].messages), 2)
        self.assertIn("Hello world", convs[0].messages[0][1])
        self.assertIsNone(convs[0].messages[1][1])  # assistant placeholder
```
**EN:** Test generating embedding conversations with text only. This test exercises `test_text_only` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test generating embedding conversations with text only. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_text_only`。

### Lines 843-854: test case with image / 测试用例 with image
```python
    def test_with_image(self):
        """Test generating embedding conversations with image."""
        convs = generate_embedding_convs(
            texts=["Describe"],
            images=["http://example.com/img.jpg"],
            videos=[None],
            template_name="chatml",
        )
        self.assertEqual(len(convs), 1)
        msg = convs[0].messages[0][1]
        self.assertIn("<image>", msg)
        self.assertIn("Describe", msg)
```
**EN:** Test generating embedding conversations with image. This test exercises `test_with_image` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test generating embedding conversations with image. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_with_image`。

### Lines 856-867: test case with video / 测试用例 with video
```python
    def test_with_video(self):
        """Test generating embedding conversations with video."""
        convs = generate_embedding_convs(
            texts=["Describe"],
            images=[None],
            videos=["http://example.com/vid.mp4"],
            template_name="chatml",
        )
        self.assertEqual(len(convs), 1)
        msg = convs[0].messages[0][1]
        self.assertIn("<video>", msg)
        self.assertIn("Describe", msg)
```
**EN:** Test generating embedding conversations with video. This test exercises `test_with_video` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test generating embedding conversations with video. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_with_video`。

### Lines 869-879: test case with image and video / 测试用例 with image and video
```python
    def test_with_image_and_video(self):
        """Test embedding conv with both image and video."""
        convs = generate_embedding_convs(
            texts=["Desc"],
            images=["http://example.com/img.jpg"],
            videos=["http://example.com/vid.mp4"],
            template_name="chatml",
        )
        msg = convs[0].messages[0][1]
        self.assertIn("<image>", msg)
        self.assertIn("<video>", msg)
```
**EN:** Test embedding conv with both image and video. This test exercises `test_with_image_and_video` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test embedding conv with both image and video. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_with_image_and_video`。

### Lines 881-892: test case none text / 测试用例 none text
```python
    def test_none_text(self):
        """Test embedding conv with None text (only media)."""
        convs = generate_embedding_convs(
            texts=[None],
            images=["http://example.com/img.jpg"],
            videos=[None],
            template_name="chatml",
        )
        msg = convs[0].messages[0][1]
        self.assertIn("<image>", msg)
        # None text should not produce "None" string
        self.assertNotIn("None", msg)
```
**EN:** Test embedding conv with None text (only media). This test exercises `test_none_text` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test embedding conv with None text (only media). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_none_text`。

### Lines 894-902: test case multiple items / 测试用例 multiple items
```python
    def test_multiple_items(self):
        """Test generating multiple embedding conversations."""
        convs = generate_embedding_convs(
            texts=["text1", "text2"],
            images=[None, None],
            videos=[None, None],
            template_name="chatml",
        )
        self.assertEqual(len(convs), 2)
```
**EN:** Test generating multiple embedding conversations. This test exercises `test_multiple_items` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test generating multiple embedding conversations. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multiple_items`。

### Lines 905-905: class TestGetFullMultimodalTextPrompt declaration / 类 TestGetFullMultimodalTextPrompt 声明
```python
class TestGetFullMultimodalTextPrompt(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 906-910: test case adds missing image tokens / 测试用例 adds missing image tokens
```python
    def test_adds_missing_image_tokens(self):
        """Test adding missing image tokens to prompt."""
        result = _get_full_multimodal_text_prompt("<image>", 3, "Describe this.")
        self.assertEqual(result.count("<image>"), 3)
        self.assertIn("Describe this.", result)
```
**EN:** Test adding missing image tokens to prompt. This test exercises `test_adds_missing_image_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test adding missing image tokens to prompt. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_adds_missing_image_tokens`。

### Lines 912-917: test case preserves existing tokens / 测试用例 preserves existing tokens
```python
    def test_preserves_existing_tokens(self):
        """Test that existing tokens in prompt are preserved."""
        result = _get_full_multimodal_text_prompt(
            "<image>", 2, "<image> What about this?"
        )
        self.assertEqual(result.count("<image>"), 2)
```
**EN:** Test that existing tokens in prompt are preserved. This test exercises `test_preserves_existing_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that existing tokens in prompt are preserved. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_preserves_existing_tokens`。

### Lines 919-922: test case all tokens present no addition / 测试用例 all tokens present no addition
```python
    def test_all_tokens_present_no_addition(self):
        """Test no addition when all tokens are already present."""
        result = _get_full_multimodal_text_prompt("<image>", 2, "<image> and <image>")
        self.assertEqual(result, "<image> and <image>")
```
**EN:** Test no addition when all tokens are already present. This test exercises `test_all_tokens_present_no_addition` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test no addition when all tokens are already present. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_all_tokens_present_no_addition`。

### Lines 924-927: test case more tokens than data raises / 测试用例 more tokens than data raises
```python
    def test_more_tokens_than_data_raises(self):
        """Test that more placeholders than data items raises ValueError."""
        with self.assertRaises(ValueError):
            _get_full_multimodal_text_prompt("<image>", 1, "<image> <image>")
```
**EN:** Test that more placeholders than data items raises ValueError. This test exercises `test_more_tokens_than_data_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that more placeholders than data items raises ValueError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_more_tokens_than_data_raises`。

### Lines 929-932: test case zero count with no tokens / 测试用例 zero count with no tokens
```python
    def test_zero_count_with_no_tokens(self):
        """Test zero modality count with no tokens in prompt."""
        result = _get_full_multimodal_text_prompt("<image>", 0, "Just text")
        self.assertEqual(result, "Just text")
```
**EN:** Test zero modality count with no tokens in prompt. This test exercises `test_zero_count_with_no_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test zero modality count with no tokens in prompt. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_zero_count_with_no_tokens`。

### Lines 934-938: test case video tokens / 测试用例 video tokens
```python
    def test_video_tokens(self):
        """Test adding missing video tokens."""
        result = _get_full_multimodal_text_prompt("<video>", 2, "Describe:")
        self.assertEqual(result.count("<video>"), 2)
        self.assertIn("Describe:", result)
```
**EN:** Test adding missing video tokens. This test exercises `test_video_tokens` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test adding missing video tokens. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_video_tokens`。

### Lines 940-948: test case tokens joined with newline / 测试用例 tokens joined with newline
```python
    def test_tokens_joined_with_newline(self):
        """Test that missing tokens are joined with newlines before prompt."""
        result = _get_full_multimodal_text_prompt("<image>", 3, "text")
        # 3 images, 0 in prompt → 3 added, joined by \n, then \n before text
        lines = result.split("\n")
        self.assertEqual(lines[0], "<image>")
        self.assertEqual(lines[1], "<image>")
        self.assertEqual(lines[2], "<image>")
        self.assertEqual(lines[3], "text")
```
**EN:** Test that missing tokens are joined with newlines before prompt. This test exercises `test_tokens_joined_with_newline` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that missing tokens are joined with newlines before prompt. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_tokens_joined_with_newline`。

### Lines 951-951: class TestGenerateChatConv declaration / 类 TestGenerateChatConv 声明
```python
class TestGenerateChatConv(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 952-952: supporting statements / 辅助语句
```python
    """Test generate_chat_conv with real Pydantic message objects."""
```
**EN:** This block contains supporting statements that connect surrounding definitions and keep module state consistent.
**CN:** 该代码块包含衔接周围定义的辅助语句，用于保持模块状态一致。

### Lines 954-956: method make request / 方法 make request
```python
    def _make_request(self, messages):
        """Create a real ChatCompletionRequest with given messages."""
        return ChatCompletionRequest(messages=messages, model="test")
```
**EN:** Create a real ChatCompletionRequest with given messages. This block implements `_make_request` and captures one focused piece of the module's behavior.
**CN:** Create a real ChatCompletionRequest with given messages. 该代码块实现 `_make_request`，承担模块行为中的一个聚焦逻辑片段。

### Lines 958-967: test case simple user message / 测试用例 simple user message
```python
    def test_simple_user_message(self):
        """Test basic user string message."""
        request = self._make_request(
            [ChatCompletionMessageUserParam(role="user", content="Hello")]
        )
        conv = generate_chat_conv(request, "chatml")
        # user message + blank assistant placeholder
        self.assertEqual(len(conv.messages), 2)
        self.assertIn("Hello", conv.messages[0][1])
        self.assertIsNone(conv.messages[1][1])
```
**EN:** Test basic user string message. This test exercises `test_simple_user_message` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test basic user string message. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_simple_user_message`。

### Lines 969-979: test case system then user / 测试用例 system then user
```python
    def test_system_then_user(self):
        """Test system message followed by user message."""
        request = self._make_request(
            [
                ChatCompletionMessageGenericParam(role="system", content="Be helpful"),
                ChatCompletionMessageUserParam(role="user", content="Hi"),
            ]
        )
        conv = generate_chat_conv(request, "chatml")
        self.assertEqual(conv.system_message, "Be helpful")
        self.assertIn("Hi", conv.messages[0][1])
```
**EN:** Test system message followed by user message. This test exercises `test_system_then_user` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test system message followed by user message. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_system_then_user`。

### Lines 981-997: test case system message as list / 测试用例 system message as list
```python
    def test_system_message_as_list(self):
        """Test system message given as a single-element list of text parts."""
        request = self._make_request(
            [
                ChatCompletionMessageGenericParam(
                    role="system",
                    content=[
                        ChatCompletionMessageContentTextPart(
                            type="text", text="System text"
                        )
                    ],
                ),
                ChatCompletionMessageUserParam(role="user", content="Hi"),
            ]
        )
        conv = generate_chat_conv(request, "chatml")
        self.assertEqual(conv.system_message, "System text")
```
**EN:** Test system message given as a single-element list of text parts. This test exercises `test_system_message_as_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test system message given as a single-element list of text parts. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_system_message_as_list`。

### Lines 999-1018: test case system message invalid list raises / 测试用例 system message invalid list raises
```python
    def test_system_message_invalid_list_raises(self):
        """Test that system message with non-text content raises ValueError."""
        request = self._make_request(
            [
                ChatCompletionMessageGenericParam(
                    role="system",
                    content=[
                        ChatCompletionMessageContentImagePart(
                            type="image_url",
                            image_url=ChatCompletionMessageContentImageURL(
                                url="http://example.com/img.jpg"
                            ),
                        )
                    ],
                ),
                ChatCompletionMessageUserParam(role="user", content="Hi"),
            ]
        )
        with self.assertRaises(ValueError):
            generate_chat_conv(request, "chatml")
```
**EN:** Test that system message with non-text content raises ValueError. This test exercises `test_system_message_invalid_list_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that system message with non-text content raises ValueError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_system_message_invalid_list_raises`。

### Lines 1020-1033: test case multi turn conversation / 测试用例 multi turn conversation
```python
    def test_multi_turn_conversation(self):
        """Test multi-turn user/assistant conversation."""
        request = self._make_request(
            [
                ChatCompletionMessageUserParam(role="user", content="What is 2+2?"),
                ChatCompletionMessageGenericParam(role="assistant", content="4"),
                ChatCompletionMessageUserParam(role="user", content="And 3+3?"),
            ]
        )
        conv = generate_chat_conv(request, "chatml")
        # 3 explicit messages + 1 blank assistant placeholder
        self.assertEqual(len(conv.messages), 4)
        self.assertEqual(conv.messages[1][1], "4")
        self.assertIsNone(conv.messages[3][1])
```
**EN:** Test multi-turn user/assistant conversation. This test exercises `test_multi_turn_conversation` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test multi-turn user/assistant conversation. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_multi_turn_conversation`。

### Lines 1035-1050: test case assistant message as list / 测试用例 assistant message as list
```python
    def test_assistant_message_as_list(self):
        """Test assistant message given as a single-element list of text parts."""
        request = self._make_request(
            [
                ChatCompletionMessageUserParam(role="user", content="Hi"),
                ChatCompletionMessageGenericParam(
                    role="assistant",
                    content=[
                        ChatCompletionMessageContentTextPart(type="text", text="Hello!")
                    ],
                ),
                ChatCompletionMessageUserParam(role="user", content="How are you?"),
            ]
        )
        conv = generate_chat_conv(request, "chatml")
        self.assertEqual(conv.messages[1][1], "Hello!")
```
**EN:** Test assistant message given as a single-element list of text parts. This test exercises `test_assistant_message_as_list` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test assistant message given as a single-element list of text parts. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_assistant_message_as_list`。

### Lines 1052-1071: test case assistant invalid list raises / 测试用例 assistant invalid list raises
```python
    def test_assistant_invalid_list_raises(self):
        """Test that assistant message with non-text content raises ValueError."""
        request = self._make_request(
            [
                ChatCompletionMessageUserParam(role="user", content="Hi"),
                ChatCompletionMessageGenericParam(
                    role="assistant",
                    content=[
                        ChatCompletionMessageContentImagePart(
                            type="image_url",
                            image_url=ChatCompletionMessageContentImageURL(
                                url="http://example.com/img.jpg"
                            ),
                        )
                    ],
                ),
            ]
        )
        with self.assertRaises(ValueError):
            generate_chat_conv(request, "chatml")
```
**EN:** Test that assistant message with non-text content raises ValueError. This test exercises `test_assistant_invalid_list_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that assistant message with non-text content raises ValueError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_assistant_invalid_list_raises`。

### Lines 1073-1081: test case string messages raises / 测试用例 string messages raises
```python
    def test_string_messages_raises(self):
        """Test that passing messages as a raw string raises ValueError."""
        request = self._make_request(
            [ChatCompletionMessageUserParam(role="user", content="Hi")]
        )
        # Manually override messages to be a string to trigger validation
        request.__dict__["messages"] = "not a list"
        with self.assertRaises(ValueError):
            generate_chat_conv(request, "chatml")
```
**EN:** Test that passing messages as a raw string raises ValueError. This test exercises `test_string_messages_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that passing messages as a raw string raises ValueError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_string_messages_raises`。

### Lines 1083-1107: test case user message with image / 测试用例 user message with image
```python
    def test_user_message_with_image(self):
        """Test user message with image content part."""
        request = self._make_request(
            [
                ChatCompletionMessageUserParam(
                    role="user",
                    content=[
                        ChatCompletionMessageContentTextPart(
                            type="text", text="What's in this image?"
                        ),
                        ChatCompletionMessageContentImagePart(
                            type="image_url",
                            image_url=ChatCompletionMessageContentImageURL(
                                url="http://example.com/cat.jpg"
                            ),
                        ),
                    ],
                )
            ]
        )
        conv = generate_chat_conv(request, "chatml")
        self.assertEqual(len(conv.image_data), 1)
        self.assertEqual(conv.image_data[0].url, "http://example.com/cat.jpg")
        msg = conv.messages[0][1]
        self.assertIn("What's in this image?", msg)
```
**EN:** Test user message with image content part. This test exercises `test_user_message_with_image` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test user message with image content part. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_user_message_with_image`。

### Lines 1109-1131: test case user message with video / 测试用例 user message with video
```python
    def test_user_message_with_video(self):
        """Test user message with video content part."""
        request = self._make_request(
            [
                ChatCompletionMessageUserParam(
                    role="user",
                    content=[
                        ChatCompletionMessageContentTextPart(
                            type="text", text="Describe this video"
                        ),
                        ChatCompletionMessageContentVideoPart(
                            type="video_url",
                            video_url=ChatCompletionMessageContentVideoURL(
                                url="http://example.com/vid.mp4"
                            ),
                        ),
                    ],
                )
            ]
        )
        conv = generate_chat_conv(request, "chatml")
        self.assertEqual(len(conv.video_data), 1)
        self.assertEqual(conv.video_data[0], "http://example.com/vid.mp4")
```
**EN:** Test user message with video content part. This test exercises `test_user_message_with_video` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test user message with video content part. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_user_message_with_video`。

### Lines 1133-1155: test case user message with audio / 测试用例 user message with audio
```python
    def test_user_message_with_audio(self):
        """Test user message with audio content part."""
        request = self._make_request(
            [
                ChatCompletionMessageUserParam(
                    role="user",
                    content=[
                        ChatCompletionMessageContentTextPart(
                            type="text", text="Transcribe this"
                        ),
                        ChatCompletionMessageContentAudioPart(
                            type="audio_url",
                            audio_url=ChatCompletionMessageContentAudioURL(
                                url="http://example.com/audio.wav"
                            ),
                        ),
                    ],
                )
            ]
        )
        conv = generate_chat_conv(request, "chatml")
        self.assertEqual(len(conv.audio_data), 1)
        self.assertEqual(conv.audio_data[0], "http://example.com/audio.wav")
```
**EN:** Test user message with audio content part. This test exercises `test_user_message_with_audio` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test user message with audio content part. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_user_message_with_audio`。

### Lines 1157-1197: test case user message image at prefix / 测试用例 user message image at prefix
```python
    def test_user_message_image_at_prefix(self):
        """Test image_token_at_prefix=True puts image token before text."""
        # Register a temporary template with image_token_at_prefix=True
        tmp_name = "_test_prefix_img"
        register_conv_template(
            Conversation(
                name=tmp_name,
                roles=("<|im_start|>user", "<|im_start|>assistant"),
                messages=[],
                sep_style=SeparatorStyle.CHATML,
                sep="<|im_end|>",
                image_token_at_prefix=True,
            )
        )
        try:
            request = self._make_request(
                [
                    ChatCompletionMessageUserParam(
                        role="user",
                        content=[
                            ChatCompletionMessageContentTextPart(
                                type="text", text="Describe"
                            ),
                            ChatCompletionMessageContentImagePart(
                                type="image_url",
                                image_url=ChatCompletionMessageContentImageURL(
                                    url="http://example.com/img.jpg"
                                ),
                            ),
                        ],
                    )
                ]
            )
            conv = generate_chat_conv(request, tmp_name)
            msg = conv.messages[0][1]
            # Image token should be BEFORE "Describe"
            img_pos = msg.find("<image>")
            txt_pos = msg.find("Describe")
            self.assertGreater(txt_pos, img_pos)
        finally:
            del chat_templates[tmp_name]
```
**EN:** Test image_token_at_prefix=True puts image token before text. This test exercises `test_user_message_image_at_prefix` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test image_token_at_prefix=True puts image token before text. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_user_message_image_at_prefix`。

### Lines 1199-1229: test case deepseek vl2 modality supplement / 测试用例 deepseek vl2 modality supplement
```python
    def test_deepseek_vl2_modality_supplement(self):
        """Test deepseek-vl2 modality supplement (add_token_as_needed path)."""
        request = self._make_request(
            [
                ChatCompletionMessageUserParam(
                    role="user",
                    content=[
                        ChatCompletionMessageContentTextPart(
                            type="text", text="Describe both"
                        ),
                        ChatCompletionMessageContentImagePart(
                            type="image_url",
                            image_url=ChatCompletionMessageContentImageURL(
                                url="http://example.com/img1.jpg"
                            ),
                        ),
                        ChatCompletionMessageContentImagePart(
                            type="image_url",
                            image_url=ChatCompletionMessageContentImageURL(
                                url="http://example.com/img2.jpg"
                            ),
                        ),
                    ],
                )
            ]
        )
        conv = generate_chat_conv(request, "deepseek-vl2")
        self.assertEqual(len(conv.image_data), 2)
        msg = conv.messages[0][1]
        # deepseek-vl2 uses _get_full_multimodal_text_prompt to add image tokens
        self.assertIn("Describe both", msg)
```
**EN:** Test deepseek-vl2 modality supplement (add_token_as_needed path). This test exercises `test_deepseek_vl2_modality_supplement` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test deepseek-vl2 modality supplement (add_token_as_needed path). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_deepseek_vl2_modality_supplement`。

### Lines 1231-1241: test case unknown role raises / 测试用例 unknown role raises
```python
    def test_unknown_role_raises(self):
        """Test that an unknown message role raises ValueError."""
        request = self._make_request(
            [ChatCompletionMessageUserParam(role="user", content="Hi")]
        )
        # Manually inject a message with unknown role
        from types import SimpleNamespace

        request.__dict__["messages"] = [SimpleNamespace(role="alien", content="Hi")]
        with self.assertRaises(ValueError):
            generate_chat_conv(request, "chatml")
```
**EN:** Test that an unknown message role raises ValueError. This test exercises `test_unknown_role_raises` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that an unknown message role raises ValueError. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_unknown_role_raises`。

### Lines 1243-1263: test case user message many images adds newline / 测试用例 user message many images adds newline
```python
    def test_user_message_many_images_adds_newline(self):
        """Test that >16 images triggers newline before text content."""
        image_parts = [
            ChatCompletionMessageContentImagePart(
                type="image_url",
                image_url=ChatCompletionMessageContentImageURL(
                    url=f"http://example.com/img{i}.jpg"
                ),
            )
            for i in range(17)
        ]
        content = [
            ChatCompletionMessageContentTextPart(type="text", text="Describe all")
        ] + image_parts
        request = self._make_request(
            [ChatCompletionMessageUserParam(role="user", content=content)]
        )
        conv = generate_chat_conv(request, "chatml")
        self.assertEqual(len(conv.image_data), 17)
        # With >16 images, text content is prefixed with "\n"
        self.assertIn("\nDescribe all", conv.messages[0][1])
```
**EN:** Test that >16 images triggers newline before text content. This test exercises `test_user_message_many_images_adds_newline` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that >16 images triggers newline before text content. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_user_message_many_images_adds_newline`。

### Lines 1266-1266: class TestGetModelType declaration / 类 TestGetModelType 声明
```python
class TestGetModelType(CustomTestCase):
```
**EN:** This section introduces the class and any class-level context used by later methods. It inherits from `CustomTestCase`.
**CN:** 该部分引入类定义以及后续方法会使用的类级上下文。 它继承自 `CustomTestCase`。

### Lines 1267-1270: test case nonexistent path returns none / 测试用例 nonexistent path returns none
```python
    def test_nonexistent_path_returns_none(self):
        """Test that a path without config.json returns None."""
        result = get_model_type("/nonexistent/path/abc123")
        self.assertIsNone(result)
```
**EN:** Test that a path without config.json returns None. This test exercises `test_nonexistent_path_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that a path without config.json returns None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_nonexistent_path_returns_none`。

### Lines 1272-1279: test case valid config returns model type / 测试用例 valid config returns model type
```python
    def test_valid_config_returns_model_type(self):
        """Test reading model_type from a real config.json file."""
        with tempfile.TemporaryDirectory() as tmpdir:
            config = {"model_type": "llama", "hidden_size": 4096}
            with open(os.path.join(tmpdir, "config.json"), "w") as f:
                json.dump(config, f)
            result = get_model_type(tmpdir)
            self.assertEqual(result, "llama")
```
**EN:** Test reading model_type from a real config.json file. This test exercises `test_valid_config_returns_model_type` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test reading model_type from a real config.json file. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_valid_config_returns_model_type`。

### Lines 1281-1288: test case config without model type returns none / 测试用例 config without model type returns none
```python
    def test_config_without_model_type_returns_none(self):
        """Test that config.json without model_type key returns None."""
        with tempfile.TemporaryDirectory() as tmpdir:
            config = {"hidden_size": 4096}
            with open(os.path.join(tmpdir, "config.json"), "w") as f:
                json.dump(config, f)
            result = get_model_type(tmpdir)
            self.assertIsNone(result)
```
**EN:** Test that config.json without model_type key returns None. This test exercises `test_config_without_model_type_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that config.json without model_type key returns None. 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_config_without_model_type_returns_none`。

### Lines 1290-1296: test case invalid json returns none / 测试用例 invalid json returns none
```python
    def test_invalid_json_returns_none(self):
        """Test that malformed config.json returns None (JSONDecodeError)."""
        with tempfile.TemporaryDirectory() as tmpdir:
            with open(os.path.join(tmpdir, "config.json"), "w") as f:
                f.write("not valid json{{{")
            result = get_model_type(tmpdir)
            self.assertIsNone(result)
```
**EN:** Test that malformed config.json returns None (JSONDecodeError). This test exercises `test_invalid_json_returns_none` by arranging inputs, invoking the relevant path, and checking the expected outcome.
**CN:** Test that malformed config.json returns None (JSONDecodeError). 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_invalid_json_returns_none`。

### Lines 1299-1300: direct execution entry point / 直接执行入口
```python
if __name__ == "__main__":
    unittest.main()
```
**EN:** This block enables the file to run as a script and dispatches into the module's test runner or main entry point.
**CN:** 该代码块使文件可以直接作为脚本运行，并转入模块的测试运行器或主入口。

## Key Concepts / 关键概念
- `TestConversationGetPrompt`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestConversationMethods`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestTemplateRegistry`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGenerateEmbeddingConvs`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGetFullMultimodalTextPrompt`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestGenerateChatConv`: Test generate_chat_conv with real Pydantic message objects. / 用于组织相关测试、夹具或辅助方法。
- `TestGetModelType`: Groups related tests, fixtures, or helper methods. / 用于组织相关测试、夹具或辅助方法。
- `TestConversationGetPrompt.test_add_colon_single`: Test prompt generation with ADD_COLON_SINGLE style. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_colon_single`。
- `TestConversationGetPrompt.test_add_colon_two`: Test prompt generation with ADD_COLON_TWO style (alternating separators). / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_add_colon_two`。
- `TestConversationGetPrompt.test_chatml`: Test prompt generation with CHATML style. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_chatml`。
- `TestConversationGetPrompt.test_llama3`: Test prompt generation with LLAMA3 style. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_llama3`。
- `TestConversationGetPrompt.test_no_colon_single`: Test prompt generation with NO_COLON_SINGLE style. / 该测试通过准备输入、调用相关路径并检查期望结果来验证 `test_no_colon_single`。

## Dependencies / 依赖关系
- **Standard library / 标准库**: `json`, `os`, `tempfile`, `unittest`
- **Internal modules / 内部模块**: `sglang.srt.entrypoints.openai.protocol`, `sglang.srt.parser.conversation`, `sglang.test.ci.ci_register`, `sglang.test.test_utils`

- **Total lines / 总行数**: 1300
