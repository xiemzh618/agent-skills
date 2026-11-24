# agent-skills

我的个人 Agent Skills 仓库。

## Skills

| Skill | 用途 |
| --- | --- |
| [lead-delegation](skills/lead-delegation/SKILL.md) | 由主 Agent 拆解并委派开发、调研或审查任务，验收并集成子 Agent 的产出。 |

## 使用 lead-delegation

在仓库根目录执行以下命令，将 Skill 链接到支持 `~/.agents/skills` 的客户端。

```sh
mkdir -p ~/.agents/skills
ln -s "$(pwd)/skills/lead-delegation" ~/.agents/skills/lead-delegation
```

示例请求：“用 `lead-delegation` 主导这次改动。让一个子 Agent 实现接口，另一个独立检查现有测试缺口。你负责验收和集成。”

主导委派的工作流参考 [BytePioneer-AI 的 lead-delegation](https://github.com/BytePioneer-AI/agent-skills/tree/main/skills/lead-delegation)。
