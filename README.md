# Code Comments

An agent skill for writing and reviewing English code comments. It asks agents
to verify behavior against implementations and callers, explain decisions and
failure consequences, and cover declarations and operations according to their
explanatory role.

The rules apply to handwritten code across languages. The included examples use
Go. This skill was extracted from TokenRouter's `.agents/skills/code-comments`.

[中文说明](#中文说明)

## Install in Codex

For your user account:

```sh
mkdir -p ~/.agents/skills
git clone https://github.com/smartcmd/code-comments.git ~/.agents/skills/code-comments
```

For a single project, run from its repository root:

```sh
mkdir -p .agents/skills
git clone https://github.com/smartcmd/code-comments.git .agents/skills/code-comments
```

Choose one location. If that destination already exists, inspect it before
replacing or updating it. A project-local clone has its own Git history; to
version the skill with your project, copy `SKILL.md`, `agents/`, and `references/`
from a separate clone into `.agents/skills/code-comments/`.

Codex discovers skills in these locations. If the skill does not appear, restart
Codex. See the [official skill documentation](https://learn.chatgpt.com/docs/build-skills)
for discovery and invocation details.

## Use

Mention the skill in a Codex CLI or IDE prompt:

```text
Use $code-comments to review the comments in this package. Report missing
explanations and unsupported claims before editing.
```

```text
Use $code-comments to improve comments in the files changed by this task.
Verify ownership, defaults, and failure behavior against the implementation.
Keep executable code unchanged.
```

Automatic selection is enabled in `agents/openai.yaml`. To make the skill part of
a project's working instructions, add this to its `AGENTS.md`:

```markdown
Before writing, modifying, or reviewing handwritten code, load and follow
`.agents/skills/code-comments/SKILL.md`. Apply its coverage, placement, and
evidence requirements to the task's scope.
```

## What the skill requires

- Cover public and private declarations, fields, file overviews, and required
  local operations within the task's scope.
- Choose between definition, calling-rule, and design explanations based on the
  behavior the code owns.
- Explain defaults, ownership, ordering, cancellation, recovery, and exceptional
  results where they affect callers or maintainers.
- Check claims against code and consumers. Use executable probes when an
  uncertain language or concurrency behavior needs verification.
- Review the whole file for accuracy, readability, placement, and repetition.

The skill sets minimum word, sentence, and paragraph counts by role. These are
mandatory when using it; the full table and classification rules are in
[SKILL.md](SKILL.md#mandatory-coverage-and-minimum-lengths). Meeting a count is one
part of review. Each explanation also needs supported facts and a clear purpose.

New files receive full required coverage. In existing files, the scope includes
changed declarations and operations, affected explanations, and a file overview
for each edited production file. Generated code and other listed exemptions are
described in [Scope](SKILL.md#scope).

## Files

| File | Purpose |
| --- | --- |
| [SKILL.md](SKILL.md) | Coverage, explanatory roles, minimum lengths, placement, and review rules |
| [agents/openai.yaml](agents/openai.yaml) | Codex display metadata and automatic selection policy |
| [references/examples.md](references/examples.md) | Declaration, field, operation, and test examples; review failures |
| [references/file-overviews.md](references/file-overviews.md) | Purpose and design overviews with stated assumptions |
| [references/evaluation.md](references/evaluation.md) | Procedure for comparing skill revisions |

## Project conventions

The skill includes two conventions scoped to TokenRouter: packages with at least
15 handwritten source files require `doc.go`, and its `tools/declorder` formatter
can move detached comments. Other projects should follow their own package
documentation and formatting rules. The general instruction to inspect overview
placement after formatting applies wherever formatting can move comments.

The bundle contains Markdown instructions and YAML metadata. Its examples state
assumed behavior; verify those assumptions before adapting an example to real
code. Comment review and code tests provide different evidence, and both should
be reported according to the work performed.

## 中文说明

这个 skill 用于编写和审查手写代码中的英文注释。它要求先核实实现、调用方和测试，
再解释用途、调用条件、设计选择和失败后果。规则适用于多种语言，示例使用 Go。

安装时，选择上面的用户级或项目级命令。用户级目录为 `~/.agents/skills/code-comments`，
项目级目录为 `.agents/skills/code-comments`。安装后可以在 Codex CLI 或 IDE 中使用：

```text
使用 $code-comments 审查这个包的注释，先报告缺失的解释和缺乏代码依据的表述。
```

```text
使用 $code-comments 完善本次修改涉及的注释，核实默认值、所有权和异常路径，保持代码行为不变。
```

规则覆盖公开和私有声明、字段、文件概述及指定的局部操作，并按解释职责规定最低长度。
审查时还要检查事实依据、段落组织、重复内容和放置位置。新增文件需要完整覆盖；
已有文件按修改范围和受影响的解释处理。

本仓库保留了来自 TokenRouter 的技能原文及三份参考文档。`doc.go` 的文件数量门槛和
`tools/declorder` 说明属于该项目的专用约定，其他项目应采用自己的包文档与格式化规则。
