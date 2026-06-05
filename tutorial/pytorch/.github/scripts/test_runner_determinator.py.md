# test_runner_determinator.py — Code Analysis / 代码分析

## Source / 来源
- **File / 文件**: `.github/scripts/test_runner_determinator.py`
- **Repository / 仓库**: `/root/xw/pytorch`
- **Purpose (EN)**: Implements repository automation helpers used by CI, release engineering, or GitHub maintenance workflows. This particular file is a focused test module for adjacent automation helpers.
- **用途 (CN)**: 实现仓库自动化辅助脚本，供 CI、发布工程或 GitHub 维护流程使用。 该文件还是一个针对相邻自动化辅助逻辑的聚焦测试模块。

## Line-by-Line Analysis / 逐行分析
### Lines 1-23 / 第 1-23 行

````python
from unittest import main, TestCase
from unittest.mock import Mock, patch

import runner_determinator as rd


USER_BRANCH = "somebranch"
EXCEPTION_BRANCH = "main"


class TestRunnerDeterminatorIssueParser(TestCase):
    def test_parse_settings(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 25
            otherExp:
                rollout_perc: 0
                default: false
        ---

        Users:
        @User1,lf
````

- EN: This block imports dependencies such as `unittest`, `unittest.mock`, `runner_determinator`; introduces classes like `TestRunnerDeterminatorIssueParser`; defines callable units such as `test_parse_settings`; parses command-line arguments for script entry points; acts as a test block that checks expected behavior.
- CN: 该代码块导入当前模块运行所需的依赖；定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；为脚本入口解析命令行参数；作为测试代码块校验预期行为。

### Lines 24-39 / 第 24-39 行

````python
        @User2,lf,otherExp

        """

        settings = rd.parse_settings(settings_text)

        self.assertTupleEqual(
            rd.Experiment(rollout_perc=25),
            settings.experiments["lf"],
            "lf settings not parsed correctly",
        )
        self.assertTupleEqual(
            rd.Experiment(rollout_perc=0, default=False),
            settings.experiments["otherExp"],
            "otherExp settings not parsed correctly",
        )
````

- EN: This block implements local helper logic for test runner determinator.
- CN: 该代码块实现与 test runner determinator 相关的局部辅助逻辑。

### Lines 41-57 / 第 41-57 行

````python
    def test_parse_settings_with_invalid_experiment_name_skips_experiment(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 25
            -badExp:
                rollout_perc: 0
                default: false
        ---

        Users:
        @User1,lf
        @User2,lf,-badExp

        """

        settings = rd.parse_settings(settings_text)
````

- EN: This block defines callable units such as `test_parse_settings_with_invalid_experiment_name_skips_experiment`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 59-81 / 第 59-81 行

````python
        self.assertTupleEqual(
            rd.Experiment(rollout_perc=25),
            settings.experiments["lf"],
            "lf settings not parsed correctly",
        )
        self.assertNotIn("-badExp", settings.experiments)

    def test_parse_settings_in_code_block(self) -> None:
        settings_text = """

        ```
        experiments:
            lf:
                rollout_perc: 25
            otherExp:
                rollout_perc: 0
                default: false
        ```

        ---

        Users:
        @User1,lf
````

- EN: This block defines callable units such as `test_parse_settings_in_code_block`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 82-97 / 第 82-97 行

````python
        @User2,lf,otherExp

        """

        settings = rd.parse_settings(settings_text)

        self.assertTupleEqual(
            rd.Experiment(rollout_perc=25),
            settings.experiments["lf"],
            "lf settings not parsed correctly",
        )
        self.assertTupleEqual(
            rd.Experiment(rollout_perc=0, default=False),
            settings.experiments["otherExp"],
            "otherExp settings not parsed correctly",
        )
````

- EN: This block implements local helper logic for test runner determinator.
- CN: 该代码块实现与 test runner determinator 相关的局部辅助逻辑。

### Lines 99-119 / 第 99-119 行

````python
    def test_parse_all_branches_setting(self) -> None:
        settings_text = """
        ```
        experiments:
            lf:
                rollout_perc: 25
                all_branches: true
            otherExp:
                all_branches: True
                rollout_perc: 0
        ```

        ---

        Users:
        @User1,lf
        @User2,lf,otherExp

        """

        settings = rd.parse_settings(settings_text)
````

- EN: This block defines callable units such as `test_parse_all_branches_setting`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 121-143 / 第 121-143 行

````python
        self.assertTupleEqual(
            rd.Experiment(rollout_perc=25, all_branches=True),
            settings.experiments["lf"],
            "lf settings not parsed correctly",
        )
        self.assertTrue(settings.experiments["otherExp"].all_branches)
        self.assertTupleEqual(
            rd.Experiment(rollout_perc=0, all_branches=True),
            settings.experiments["otherExp"],
            "otherExp settings not parsed correctly",
        )

    def test_parse_users(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            otherExp:
                rollout_perc: 0
        ---

        Users:
        @User1,lf
````

- EN: This block defines callable units such as `test_parse_users`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 144-165 / 第 144-165 行

````python
        @User2,lf,otherExp

        """

        users = rd.parse_users(settings_text)
        self.assertEqual(
            [rd.UserExperimentConfig("lf", 100)],
            users["User1"],
        )
        self.assertEqual(
            [
                rd.UserExperimentConfig("lf", 100),
                rd.UserExperimentConfig("otherExp", 100),
            ],
            users["User2"],
        )

    def test_parse_users_without_settings(self) -> None:
        settings_text = """

        @User1,lf
        @User2,lf,otherExp
````

- EN: This block defines callable units such as `test_parse_users_without_settings`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 167-189 / 第 167-189 行

````python
        """

        users = rd.parse_users(settings_text)
        self.assertEqual(
            [rd.UserExperimentConfig("lf", 100)],
            users["User1"],
        )
        self.assertEqual(
            [
                rd.UserExperimentConfig("lf", 100),
                rd.UserExperimentConfig("otherExp", 100),
            ],
            users["User2"],
        )

    def test_parse_users_with_rollout_perc(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            arc:
                rollout_perc: 0
        ---
````

- EN: This block defines callable units such as `test_parse_users_with_rollout_perc`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 191-213 / 第 191-213 行

````python
        Users:
        @User1,lf,arc:10
        @User2,arc:50
        @User3,lf

        """

        users = rd.parse_users(settings_text)
        self.assertEqual(
            [
                rd.UserExperimentConfig("lf", 100),
                rd.UserExperimentConfig("arc", 10),
            ],
            users["User1"],
        )
        self.assertEqual(
            [rd.UserExperimentConfig("arc", 50)],
            users["User2"],
        )
        self.assertEqual(
            [rd.UserExperimentConfig("lf", 100)],
            users["User3"],
        )
````

- EN: This block implements local helper logic for test runner determinator.
- CN: 该代码块实现与 test runner determinator 相关的局部辅助逻辑。

### Lines 215-237 / 第 215-237 行

````python
    def test_parse_users_invalid_percentage_defaults_to_100(self) -> None:
        """Non-numeric percentage like arc:abc should default to 100%."""
        settings_text = """
        @User1,arc:abc
        """

        users = rd.parse_users(settings_text)
        self.assertEqual(
            [rd.UserExperimentConfig("arc", 100)],
            users["User1"],
        )

    def test_parse_users_negative_percentage_clamped_to_zero(self) -> None:
        """Negative percentage like arc:-5 should be clamped to 0."""
        settings_text = """
        @User1,arc:-5
        """

        users = rd.parse_users(settings_text)
        self.assertEqual(
            [rd.UserExperimentConfig("arc", 0)],
            users["User1"],
        )
````

- EN: This block defines callable units such as `test_parse_users_invalid_percentage_defaults_to_100`, `test_parse_users_negative_percentage_clamped_to_zero`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 239-261 / 第 239-261 行

````python
    def test_parse_users_over_100_percentage_clamped(self) -> None:
        """Percentage over 100 like arc:200 should be clamped to 100."""
        settings_text = """
        @User1,arc:200
        """

        users = rd.parse_users(settings_text)
        self.assertEqual(
            [rd.UserExperimentConfig("arc", 100)],
            users["User1"],
        )

    def test_parse_users_opt_out_ignores_percentage(self) -> None:
        """Opt-out entries like -lf should not parse a percentage."""
        settings_text = """
        @User1,-lf
        """

        users = rd.parse_users(settings_text)
        self.assertEqual(
            [rd.UserExperimentConfig("-lf", 100)],
            users["User1"],
        )
````

- EN: This block defines callable units such as `test_parse_users_over_100_percentage_clamped`, `test_parse_users_opt_out_ignores_percentage`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 264-280 / 第 264-280 行

````python
class TestRunnerDeterminatorGetRunnerPrefix(TestCase):
    def test_opted_in_user(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            otherExp:
                rollout_perc: 0
        ---

        Users:
        @User1,lf
        @User2,lf,otherExp

        """
        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("lf.", result.prefix, "Runner prefix not correct for User1")
````

- EN: This block introduces classes like `TestRunnerDeterminatorGetRunnerPrefix`; defines callable units such as `test_opted_in_user`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 282-297 / 第 282-297 行

````python
    def test_explicitly_opted_out_user(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 100
            otherExp:
                rollout_perc: 0
        ---

        Users:
        @User1,-lf
        @User2,lf,otherExp

        """
        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("", result.prefix, "Runner prefix not correct for User1")
````

- EN: This block defines callable units such as `test_explicitly_opted_out_user`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 299-314 / 第 299-314 行

````python
    def test_explicitly_opted_in_and_out_user_should_opt_out(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 100
            otherExp:
                rollout_perc: 0
        ---

        Users:
        @User1,-lf,lf
        @User2,lf,otherExp

        """
        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("", result.prefix, "Runner prefix not correct for User1")
````

- EN: This block defines callable units such as `test_explicitly_opted_in_and_out_user_should_opt_out`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 316-333 / 第 316-333 行

````python
    def test_opted_in_user_two_experiments(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            otherExp:
                rollout_perc: 0
        ---

        Users:
        @User1,lf
        @User2,lf,otherExp

        """
        result = rd.get_runner_prefix(settings_text, ["User2"], USER_BRANCH)
        self.assertEqual(
            "lf.otherExp.", result.prefix, "Runner prefix not correct for User2"
        )
````

- EN: This block defines callable units such as `test_opted_in_user_two_experiments`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 335-351 / 第 335-351 行

````python
    def test_opted_in_user_two_experiments_default(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            otherExp:
                rollout_perc: 0
                default: false
        ---

        Users:
        @User1,lf
        @User2,lf,otherExp

        """
        result = rd.get_runner_prefix(settings_text, ["User2"], USER_BRANCH)
        self.assertEqual("lf.", result.prefix, "Runner prefix not correct for User2")
````

- EN: This block defines callable units such as `test_opted_in_user_two_experiments_default`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 353-373 / 第 353-373 行

````python
    def test_opted_in_user_two_experiments_default_exp(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            otherExp:
                rollout_perc: 0
                default: false
        ---

        Users:
        @User1,lf
        @User2,lf,otherExp

        """
        result = rd.get_runner_prefix(
            settings_text, ["User2"], USER_BRANCH, frozenset(["lf", "otherExp"])
        )
        self.assertEqual(
            "lf.otherExp.", result.prefix, "Runner prefix not correct for User2"
        )
````

- EN: This block defines callable units such as `test_opted_in_user_two_experiments_default_exp`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 375-397 / 第 375-397 行

````python
    def test_opted_in_user_two_experiments_default_exp_2(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            otherExp:
                rollout_perc: 0
                default: false
        ---

        Users:
        @User1,lf
        @User2,lf,otherExp

        """
        result = rd.get_runner_prefix(
            settings_text, ["User2"], USER_BRANCH, frozenset(["otherExp"])
        )
        self.assertEqual(
            "otherExp.", result.prefix, "Runner prefix not correct for User2"
        )

    @patch("random.uniform", return_value=50)
````

- EN: This block defines callable units such as `test_opted_in_user_two_experiments_default_exp_2`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 398-415 / 第 398-415 行

````python
    def test_opted_out_user(self, mock_uniform: Mock) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 25
            otherExp:
                rollout_perc: 25
        ---

        Users:
        @User1,lf
        @User2,lf,otherExp

        """
        result = rd.get_runner_prefix(settings_text, ["User3"], USER_BRANCH)
        self.assertEqual("", result.prefix, "Runner prefix not correct for user")

    @patch("random.uniform", return_value=10)
````

- EN: This block defines callable units such as `test_opted_out_user`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 416-437 / 第 416-437 行

````python
    def test_opted_out_user_was_pulled_in_by_rollout(self, mock_uniform: Mock) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 25
            otherExp:
                rollout_perc: 25
        ---

        Users:
        @User1,lf
        @User2,lf,otherExp

        """

        # User3 is opted out, but is pulled into both experiments by the 10% rollout
        result = rd.get_runner_prefix(settings_text, ["User3"], USER_BRANCH)
        self.assertEqual(
            "lf.otherExp.", result.prefix, "Runner prefix not correct for user"
        )

    @patch("random.uniform", return_value=10)
````

- EN: This block defines callable units such as `test_opted_out_user_was_pulled_in_by_rollout`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 438-460 / 第 438-460 行

````python
    def test_opted_out_user_was_pulled_in_by_rollout_excl_nondefault(
        self, mock_uniform: Mock
    ) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 25
            otherExp:
                rollout_perc: 25
                default: false
        ---

        Users:
        @User1,lf
        @User2,lf,otherExp

        """

        # User3 is opted out, but is pulled into default experiments by the 10% rollout
        result = rd.get_runner_prefix(settings_text, ["User3"], USER_BRANCH)
        self.assertEqual("lf.", result.prefix, "Runner prefix not correct for user")

    @patch("random.uniform", return_value=10)
````

- EN: This block defines callable units such as `test_opted_out_user_was_pulled_in_by_rollout_excl_nondefault`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 461-477 / 第 461-477 行

````python
    def test_opted_out_user_was_pulled_in_by_rollout_filter_exp(
        self, mock_uniform: Mock
    ) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 25
            otherExp:
                rollout_perc: 25
                default: false
        ---

        Users:
        @User1,lf
        @User2,lf,otherExp

        """
````

- EN: This block defines callable units such as `test_opted_out_user_was_pulled_in_by_rollout_filter_exp`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 479-501 / 第 479-501 行

````python
        # User3 is opted out, but is pulled into default experiments by the 10% rollout
        result = rd.get_runner_prefix(
            settings_text, ["User3"], USER_BRANCH, frozenset(["otherExp"])
        )
        self.assertEqual(
            "otherExp.", result.prefix, "Runner prefix not correct for user"
        )

    @patch("random.uniform", return_value=25)
    def test_opted_out_user_was_pulled_out_by_rollout_filter_exp(
        self, mock_uniform: Mock
    ) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 10
            otherExp:
                rollout_perc: 50
                default: false
        ---

        Users:
        @User1,lf
````

- EN: This block defines callable units such as `test_opted_out_user_was_pulled_out_by_rollout_filter_exp`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 502-523 / 第 502-523 行

````python
        @User2,lf,otherExp

        """

        # User3 is opted out, but is pulled into default experiments by the 10% rollout
        result = rd.get_runner_prefix(settings_text, ["User3"], USER_BRANCH)
        self.assertEqual("", result.prefix, "Runner prefix not correct for user")

    def test_lf_prefix_always_comes_first(self) -> None:
        settings_text = """
        experiments:
            otherExp:
                rollout_perc: 0
            lf:
                rollout_perc: 0
        ---

        Users:
        @User1,lf
        @User2,otherExp,lf

        """
````

- EN: This block defines callable units such as `test_lf_prefix_always_comes_first`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 525-546 / 第 525-546 行

````python
        result = rd.get_runner_prefix(settings_text, ["User2"], USER_BRANCH)
        self.assertEqual(
            "lf.otherExp.", result.prefix, "Runner prefix not correct for user"
        )

    def test_ignores_commented_users(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            otherExp:
                rollout_perc: 0
        ---

        Users:
        #@User1,lf
        @User2,lf,otherExp

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("", result.prefix, "Runner prefix not correct for user")
````

- EN: This block defines callable units such as `test_ignores_commented_users`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 548-567 / 第 548-567 行

````python
    def test_ignores_extra_experiments(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            otherExp:
                rollout_perc: 0
            foo:
                rollout_perc: 0
        ---

        Users:
        @User1,lf,otherExp,foo

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual(
            "lf.otherExp.", result.prefix, "Runner prefix not correct for user"
        )
````

- EN: This block defines callable units such as `test_ignores_extra_experiments`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 569-584 / 第 569-584 行

````python
    def test_disables_experiment_on_exception_branches_when_not_explicitly_opted_in(
        self,
    ) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 100
        ---

        Users:
        @User,lf,otherExp

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], EXCEPTION_BRANCH)
        self.assertEqual("", result.prefix, "Runner prefix not correct for user")
````

- EN: This block defines callable units such as `test_disables_experiment_on_exception_branches_when_not_explicitly_opted_in`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 586-604 / 第 586-604 行

````python
    def test_allows_experiment_on_exception_branches_when_explicitly_opted_in(
        self,
    ) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 100
                all_branches: true
        ---

        Users:
        @User,lf,otherExp

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], EXCEPTION_BRANCH)
        self.assertEqual("lf.", result.prefix, "Runner prefix not correct for user")

    @patch("random.uniform", return_value=5)
````

- EN: This block defines callable units such as `test_allows_experiment_on_exception_branches_when_explicitly_opted_in`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 605-621 / 第 605-621 行

````python
    def test_opted_in_user_with_rollout_perc_enabled(self, mock_uniform: Mock) -> None:
        """User opted in with 10% rollout, random=5 -> enabled"""
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
        ---

        Users:
        @User1,lf:10

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("lf.", result.prefix, "Runner prefix not correct for user")

    @patch("random.uniform", return_value=50)
````

- EN: This block defines callable units such as `test_opted_in_user_with_rollout_perc_enabled`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 622-644 / 第 622-644 行

````python
    def test_opted_in_user_with_rollout_perc_disabled(self, mock_uniform: Mock) -> None:
        """User opted in with 10% rollout, random=50 -> disabled"""
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
        ---

        Users:
        @User1,lf:10

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("", result.prefix, "Runner prefix not correct for user")

    def test_opted_in_user_without_rollout_perc_always_enabled(self) -> None:
        """User opted in without percentage (default 100%) -> always enabled"""
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
        ---
````

- EN: This block defines callable units such as `test_opted_in_user_with_rollout_perc_disabled`, `test_opted_in_user_without_rollout_perc_always_enabled`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 646-667 / 第 646-667 行

````python
        Users:
        @User1,lf

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("lf.", result.prefix, "Runner prefix not correct for user")

    @patch("random.uniform", return_value=15)
    def test_multiple_requesters_uses_min_perc(self, mock_uniform: Mock) -> None:
        """Two requesters with different rollout_percs, uses the minimum (10%)."""
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
        ---

        Users:
        @User1,lf:10
        @User2,lf:50

        """
````

- EN: This block defines callable units such as `test_multiple_requesters_uses_min_perc`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 669-691 / 第 669-691 行

````python
        # random=15, min_perc=10 -> 15 > 10 -> disabled
        result = rd.get_runner_prefix(settings_text, ["User1", "User2"], USER_BRANCH)
        self.assertEqual("", result.prefix, "Runner prefix not correct for user")

    @patch("random.uniform", return_value=5)
    def test_multiple_requesters_uses_min_perc_enabled(
        self, mock_uniform: Mock
    ) -> None:
        """Two requesters with different rollout_percs, min=10%, random=5 -> enabled."""
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
        ---

        Users:
        @User1,lf:10
        @User2,lf:50

        """

        result = rd.get_runner_prefix(settings_text, ["User1", "User2"], USER_BRANCH)
        self.assertEqual("lf.", result.prefix, "Runner prefix not correct for user")
````

- EN: This block defines callable units such as `test_multiple_requesters_uses_min_perc_enabled`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 693-709 / 第 693-709 行

````python
    def test_opt_out_overrides_rollout_perc(self) -> None:
        """Opt-out (-lf) wins over opt-in with rollout_perc (lf:50)."""
        settings_text = """
        experiments:
            lf:
                rollout_perc: 100
        ---

        Users:
        @User1,-lf,lf:50

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("", result.prefix, "Runner prefix not correct for user")

    @patch("random.uniform", return_value=5)
````

- EN: This block defines callable units such as `test_opt_out_overrides_rollout_perc`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 710-732 / 第 710-732 行

````python
    def test_opted_in_user_with_rollout_perc_two_experiments(
        self, mock_uniform: Mock
    ) -> None:
        """User opted into lf at 100% and otherExp at 10%, random=5 -> both enabled"""
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            otherExp:
                rollout_perc: 0
        ---

        Users:
        @User1,lf,otherExp:10

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual(
            "lf.otherExp.", result.prefix, "Runner prefix not correct for user"
        )

    @patch("random.uniform", return_value=50)
````

- EN: This block defines callable units such as `test_opted_in_user_with_rollout_perc_two_experiments`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 733-751 / 第 733-751 行

````python
    def test_opted_in_user_with_rollout_perc_partial_enable(
        self, mock_uniform: Mock
    ) -> None:
        """User opted into lf at 100% and otherExp at 10%, random=50 -> only lf enabled"""
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            otherExp:
                rollout_perc: 0
        ---

        Users:
        @User1,lf,otherExp:10

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("lf.", result.prefix, "Runner prefix not correct for user")
````

- EN: This block defines callable units such as `test_opted_in_user_with_rollout_perc_partial_enable`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 753-769 / 第 753-769 行

````python
    def test_opted_in_user_with_zero_rollout_perc(self) -> None:
        """User opted in with 0% rollout -> never enabled"""
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
        ---

        Users:
        @User1,lf:0

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("", result.prefix, "Runner prefix not correct for user")

    @patch("random.uniform", return_value=5)
````

- EN: This block defines callable units such as `test_opted_in_user_with_zero_rollout_perc`; iterates through collections, records, or generated items; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；遍历集合、记录或生成项；作为测试代码块校验预期行为。

### Lines 770-789 / 第 770-789 行

````python
    def test_arc_opted_in_user_with_rollout_perc_enabled(
        self, mock_uniform: Mock
    ) -> None:
        """User opted into arc with 10% rollout, random=5 -> arc enabled"""
        settings_text = """
        experiments:
            arc:
                rollout_perc: 0
        ---

        Users:
        @User1,arc:10

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("mt-", result.prefix)
        self.assertTrue(result.use_arc)

    @patch("random.uniform", return_value=50)
````

- EN: This block defines callable units such as `test_arc_opted_in_user_with_rollout_perc_enabled`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 790-807 / 第 790-807 行

````python
    def test_arc_opted_in_user_with_rollout_perc_disabled(
        self, mock_uniform: Mock
    ) -> None:
        """User opted into arc with 10% rollout, random=50 -> arc disabled"""
        settings_text = """
        experiments:
            arc:
                rollout_perc: 0
        ---

        Users:
        @User1,arc:10

        """

        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("", result.prefix)
        self.assertFalse(result.use_arc)
````

- EN: This block defines callable units such as `test_arc_opted_in_user_with_rollout_perc_disabled`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 810-826 / 第 810-826 行

````python
class TestRunnerDeterminatorArcExperiment(TestCase):
    ARC_SETTINGS = """
        experiments:
            arc:
                rollout_perc: 0
        ---

        Users:
        @User1,arc
        @User2,lf

        """

    def test_arc_opted_in_user_returns_mt_prefix(self) -> None:
        result = rd.get_runner_prefix(self.ARC_SETTINGS, ["User1"], USER_BRANCH)
        self.assertEqual("mt-", result.prefix)
        self.assertTrue(result.use_arc)
````

- EN: This block introduces classes like `TestRunnerDeterminatorArcExperiment`; defines callable units such as `test_arc_opted_in_user_returns_mt_prefix`; acts as a test block that checks expected behavior.
- CN: 该代码块定义新的类来封装状态与行为；定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 828-847 / 第 828-847 行

````python
    def test_arc_opted_in_user_canary_returns_c_mt_prefix(self) -> None:
        result = rd.get_runner_prefix(
            self.ARC_SETTINGS, ["User1"], USER_BRANCH, is_canary=True
        )
        self.assertEqual("c-mt-", result.prefix)
        self.assertTrue(result.use_arc)

    def test_arc_not_enabled_returns_use_arc_false(self) -> None:
        result = rd.get_runner_prefix(self.ARC_SETTINGS, ["User2"], USER_BRANCH)
        self.assertFalse(result.use_arc)

    def test_arc_not_enabled_no_experiments_returns_use_arc_false(self) -> None:
        settings_text = """
        experiments:
            arc:
                rollout_perc: 0
        ---

        Users:
        @User1,arc
````

- EN: This block defines callable units such as `test_arc_opted_in_user_canary_returns_c_mt_prefix`, `test_arc_not_enabled_returns_use_arc_false`, `test_arc_not_enabled_no_experiments_returns_use_arc_false`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 849-869 / 第 849-869 行

````python
        """
        result = rd.get_runner_prefix(settings_text, ["User3"], USER_BRANCH)
        self.assertEqual("", result.prefix)
        self.assertFalse(result.use_arc)

    @patch("random.uniform", return_value=10)
    def test_arc_rollout_percentage(self, mock_uniform: Mock) -> None:
        settings_text = """
        experiments:
            arc:
                rollout_perc: 25
        ---

        Users:

        """
        result = rd.get_runner_prefix(settings_text, ["User3"], USER_BRANCH)
        self.assertEqual("mt-", result.prefix)
        self.assertTrue(result.use_arc)

    @patch("random.uniform", return_value=50)
````

- EN: This block defines callable units such as `test_arc_rollout_percentage`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 870-892 / 第 870-892 行

````python
    def test_arc_rollout_percentage_not_selected(self, mock_uniform: Mock) -> None:
        settings_text = """
        experiments:
            arc:
                rollout_perc: 25
        ---

        Users:

        """
        result = rd.get_runner_prefix(settings_text, ["User3"], USER_BRANCH)
        self.assertEqual("", result.prefix)
        self.assertFalse(result.use_arc)

    def test_arc_opted_out_user(self) -> None:
        settings_text = """
        experiments:
            arc:
                rollout_perc: 100
        ---

        Users:
        @User1,-arc
````

- EN: This block defines callable units such as `test_arc_rollout_percentage_not_selected`, `test_arc_opted_out_user`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 894-913 / 第 894-913 行

````python
        """
        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("", result.prefix)
        self.assertFalse(result.use_arc)

    def test_arc_exception_branch_not_enabled(self) -> None:
        result = rd.get_runner_prefix(self.ARC_SETTINGS, ["User1"], EXCEPTION_BRANCH)
        self.assertEqual("", result.prefix)
        self.assertFalse(result.use_arc)

    def test_arc_exception_branch_all_branches(self) -> None:
        settings_text = """
        experiments:
            arc:
                rollout_perc: 0
                all_branches: true
        ---

        Users:
        @User1,arc
````

- EN: This block defines callable units such as `test_arc_exception_branch_not_enabled`, `test_arc_exception_branch_all_branches`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 915-935 / 第 915-935 行

````python
        """
        result = rd.get_runner_prefix(settings_text, ["User1"], EXCEPTION_BRANCH)
        self.assertEqual("mt-", result.prefix)
        self.assertTrue(result.use_arc)

    def test_arc_takes_precedence_over_lf(self) -> None:
        settings_text = """
        experiments:
            lf:
                rollout_perc: 0
            arc:
                rollout_perc: 0
        ---

        Users:
        @User1,lf,arc

        """
        result = rd.get_runner_prefix(settings_text, ["User1"], USER_BRANCH)
        self.assertEqual("mt-", result.prefix)
        self.assertTrue(result.use_arc)
````

- EN: This block defines callable units such as `test_arc_takes_precedence_over_lf`; acts as a test block that checks expected behavior.
- CN: 该代码块定义可调用函数来执行具体自动化步骤；作为测试代码块校验预期行为。

### Lines 938-939 / 第 938-939 行

````python
if __name__ == "__main__":
    main()
````

- EN: This block branches on runtime conditions or configuration choices.
- CN: 该代码块依据运行时条件或配置选择进行分支。


## Key Concepts / 关键概念
- EN: Organizes CI fan-out, job selection, or validation coverage across many execution environments.
  CN: 组织 CI 的分片扩展、作业选择或跨多种执行环境的校验覆盖。
- EN: Automates packaging, versioning, container publishing, or branch management for release engineering.
  CN: 为发布工程自动化打包、版本管理、容器发布或分支维护。
- EN: Talks to GitHub concepts or APIs to label, comment on, triage, or otherwise maintain repository state.
  CN: 围绕 GitHub 概念或 API 进行交互，以完成打标、评论、分诊或其他仓库维护工作。
- EN: Test coverage — the code validates neighboring automation behavior with assertions and expected fixtures.
  CN: 测试覆盖——代码通过断言和预期样例验证邻近自动化逻辑。

## Dependencies / 依赖关系
- Standard-library imports / 标准库导入: `unittest`, `unittest.mock`
- Internal imports / 内部导入: none
- External imports / 外部导入: `runner_determinator`
- Classes / 类: `TestRunnerDeterminatorIssueParser`, `TestRunnerDeterminatorGetRunnerPrefix`, `TestRunnerDeterminatorArcExperiment`
- Functions / 函数: none
- Test entry points / 测试入口: none
