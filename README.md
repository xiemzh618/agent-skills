# agent-skills

我的个人 Agent Skills 仓库。

## Skills

| Skill | 用途 |
| --- | --- |
| [lead-delegation](skills/lead-delegation/SKILL.md) | 由主 Agent 拆解并委派开发、调研或审查任务，验收并集成子 Agent 的产出。 |
| [trim-cot-leakage](skills/trim-cot-leakage/SKILL.md) | 清理注释和文档里残留的推理过程、失效引用和变更叙述。 |
| [find-simplifications](skills/find-simplifications/SKILL.md) | 审计代码中的过度设计，给出有证据支撑的简化候选。 |

## 安装

在仓库根目录执行需要的链接命令，以便在支持 `~/.agents/skills` 的客户端中使用。

```sh
mkdir -p ~/.agents/skills
ln -s "$(pwd)/skills/lead-delegation" ~/.agents/skills/lead-delegation
ln -s "$(pwd)/skills/trim-cot-leakage" ~/.agents/skills/trim-cot-leakage
ln -s "$(pwd)/skills/find-simplifications" ~/.agents/skills/find-simplifications
```

示例请求：“用 `lead-delegation` 主导这次改动。让一个子 Agent 实现接口，另一个独立检查现有测试缺口。你负责验收和集成。”
