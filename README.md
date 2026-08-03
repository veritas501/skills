# skills

My collection of Claude Code agent skills, installable to the local Claude CLI via symlink.

## 概览

这是一个存放 [Claude Code](https://claude.com/claude-code) agent skills 的仓库。每个 skill 是一个目录，内含 `SKILL.md`，用 frontmatter 描述触发时机与能力。`scripts/symlink_skills.sh` 把仓库里所有 skill 软链接到 `~/.claude/skills`，让本地 Claude CLI 能直接调用。集中管理、版本化、一键安装。

## 快速开始

### 前置条件

- 已安装 Claude CLI（skills 装到 `~/.claude/skills`）
- Bash

### 安装

- 方式1：npx skills

仓库根目录运行，把本地所有 skill 装进 agent：

```bash
npx skills add . -g
```

详见 [vercel-labs/skills](https://github.com/vercel-labs/skills)。

- 方式2：软链接

仓库根目录运行：

```bash
bash scripts/symlink_skills.sh
```

脚本把 `skills/` 下每个含 `SKILL.md` 的目录软链接到 `~/.claude/skills/<name>`（可通过位置参数指定其他目标目录：`bash scripts/symlink_skills.sh <目标目录>`）。已存在的同名实体目录会被替换为软链接。

## 技能清单

### Common

| Skill | 用途 |
|-------|------|
| `skill-writer` | 写/审/改 agent skill（SKILL.md），约束 skill 与 agent 能力互补而非冗余 |
| `readme-writer` | 从首次读者视角写/审/改 README，约束 agent 不编命令、不泄 secret、不写成百科 |
| `handoff` | 上下文过长或质量下降时，生成跨会话交接摘要 |

### Plan

方案规划三件套，按 discuss → write → execute 顺序使用：

| Skill | 用途 |
|-------|------|
| `plan-discuss` | 与用户讨论/设计方案——广度优先、一次一点，避免一次性设计倾倒 |
| `plan-write` | 把已定方案写成多文件实现计划，分离顶层设计与实现细节 |
| `plan-execute` | 按 plan-write 计划目录逐任务执行，防止偏离顶层设计 |
