# z-skill-publisher

把需求、资料或认可的工作流程整理成可复用的 Agent Skill，帮助创作者写出有明确判断、可观察产物和真实验收依据的指令。

适合从会话提炼方法、评估或改进既有 skill，以及整理独立发布包。它也支持 GitHub 发布，但必须有明确的账户、仓库、可见性和内容范围授权。

## 成果展示

下面“样例”要求给午餐选择流程输出最小 skill 候选，不创建文件。独立 CLI 的实际输出保留了名称 `z-lunch-picker`，没有重复加前缀，并将预算未知、限制冲突和价格未核实分别转成判断规则。

实际输出摘录：

```markdown
name: z-lunch-picker
```

这展示了把需求转为可执行规则的过程。完整结果见 [实际输出](examples/lunch-picker-output.md)，验证范围见 [本地验证摘要](examples/validation-summary.json)。实测环境为 Codex CLI 0.162.1，配置模型 gpt-6.1-sol，日期为 2026-10-23；它是单次任务证据，不是可靠性或优于其他 skill 的保证。

## 本地安装

前置条件：能发现本地 skills 的 Codex 客户端，以及提供 `sh`、`mkdir`、`cp`、`pwd` 的本地环境。普通编写任务不需要额外依赖；机械发布检查脚本需要 Python 3.9+，可选成熟秘密扫描需要预先可用的 gitleaks。模型与外部服务的使用成本按所在客户端和账户计算。

可用 Git 获取：

```sh
git clone https://github.com/score2021/z-skill-publisher.git
cd z-skill-publisher
```

也可从仓库页面选择 Code → Download ZIP 并解压。获取文件后，先进入含 `SKILL.md` 的根目录，再在终端执行以下 shell 命令。安装到 `~/.agents/skills/z-skill-publisher/`，同时检查旧路径和自定义 CODEX_HOME，遇到同名文件、目录、链接或断链会停止，保护原件。已有安装请按明确的更新范围处理，不直接覆盖。

```sh
set -eu
skill_name='z-skill-publisher'
skill_source="$(pwd -P)"
for skill_file in SKILL.md LICENSE agents/openai.yaml evals/cases.json references/authoring.md references/examples.md references/naming.md references/publishing.md references/readme.md references/testing.md scripts/check-release.py; do
  if [ ! -f "${skill_source}/${skill_file}" ]; then
    printf '缺少必要文件：%s\n' "${skill_file}" >&2
    exit 1
  fi
done
for skill_root in "${HOME}/.agents/skills" "${HOME}/.codex/skills" "${CODEX_HOME:-${HOME}/.codex}/skills"; do
  skill_existing="${skill_root}/${skill_name}"
  if [ -e "${skill_existing}" ] || [ -L "${skill_existing}" ]; then
    printf '停止：已有同名安装 %s\n' "${skill_existing}" >&2
    exit 1
  fi
done
skill_target="${HOME}/.agents/skills/${skill_name}"
mkdir -p "${HOME}/.agents/skills"
mkdir "${skill_target}"
mkdir "${skill_target}/agents" "${skill_target}/evals" "${skill_target}/references" "${skill_target}/scripts"
cp "${skill_source}/SKILL.md" "${skill_source}/LICENSE" "${skill_target}/"
cp "${skill_source}/agents/openai.yaml" "${skill_target}/agents/"
cp "${skill_source}/evals/cases.json" "${skill_target}/evals/"
for skill_reference in authoring.md examples.md naming.md publishing.md readme.md testing.md; do
  cp "${skill_source}/references/${skill_reference}" "${skill_target}/references/"
done
cp "${skill_source}/scripts/check-release.py" "${skill_target}/scripts/"
printf '已安装到 %s；请继续验证客户端发现与实际任务。\n' "${skill_target}"
```

在目标项目的新对话中输入 `$z-skill-publisher` 加具体需求；列表未刷新时重启客户端再试。终端调用时用单引号保护 `$`，避免被 shell 展开。其他客户端的发现规则和行为未实测。Codex 的路径与软链接规则见 [官方说明](https://learn.chatgpt.com/docs/build-skills)。

## 样例

下面整块复制到对话中即可，无需准备文件。预期输出名称与最小 SKILL.md 候选，预算未知时不编金额，限制冲突时不擅自放宽条件；不写文件、不安装、不发布。

```text
$z-skill-publisher 给一个按预算和饮食限制选工作日午餐的 skill 起名并输出最小 SKILL.md 候选正文，不写文件、不安装、不发布。基础名已经是 z-lunch-picker。预算未知时不要假设金额，冲突限制先说明。
```

完整创建请求可以这样写：

```text
$z-skill-publisher 在当前目录创建一个活动交接清单 skill，基础名 event-handoff。
输入是时间、场地、待办、负责人和风险的混合笔记。
已确认和待确认分开，负责人缺失标待认领，时间冲突显式指出。
附 README、可复制样例和成果展示，完成本地验证，暂不发布。
```

完整创建会在工作目录写文件，并按本次授权执行验证；不会因为名字带 publisher 就自动上传 GitHub。其范围应以实际执行报告为准。

## 工作方式与边界

- 评估与先方案：提交证据和建议，未获同意不修改目标。
- 创建：提炼行为增益，编写、打包并本地验收。
- 改进：保留有效定制，修改相关部分并复验，不重走无关流程。
- 本地发布包：限定文件与来源，整理 README、许可证、样例和展示。
- GitHub 发布：核对具体授权、范围和最终版本，发布后全新下载、安装并执行任务。

默认采用 `z-` 个人命名前缀，方便筛选，非官方强制；用户明确要求保留其他有效完整名称时尊重。既有 skill 不自动改名。

用户认可、工具成功和普遍有效分别判断。规则需要有适用条件与例外，按任务选择结构，简单任务不机械增加脚本和参考层。敏感材料不进入发布副本；格式与基础模式检查不保证归属，也不替代成熟秘密扫描。

## 验证范围

已进行结构与引用检查、独立 CLI 行为验证、命名默认及例外、仅评估与局部修改、普通 README 请求误触发检查，以及一个生成物的样例和新输入验证。生成物首次验收曾触及测试时限；后续检查发现安装命令和展示问题，修复后复验。具体限制保留在 [验证摘要](examples/validation-summary.json)。

[合成回归请求](evals/cases.json) 提供八类场景及所需原始文件；判据见 [testing.md](references/testing.md)。不是每次都要执行全部案例，也不声称测试覆盖所有输入或模型。发布和远端复验状态以实际发布记录为准。

## 目录与许可证

- `SKILL.md`：任务模式、核心判断和边界。
- `agents/openai.yaml`：UI 名称与调用提示。
- `references/`：提炼、命名、写作对照、README、验证与发布细则。
- `scripts/check-release.py`：只读机械范围、哈希和基础模式检查，可选调用 gitleaks。
- `evals/cases.json`：合成回归请求及最小材料。
- `examples/`：审核后的实际成果与验证摘要；不属于安装所需材料。

使用 [MIT License](LICENSE)，保留既有版权声明。本仓库发布通用方法与合成案例，不包含原始会话、执行日志、账号凭据、私人路径或业务数据。写作方法参考领域专家如何表达决策的思路；未把第三方项目的 skill 正文、代码配方或品牌资产打包到此仓库。
