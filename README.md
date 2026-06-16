# prompt-architect

一个用于**写、审、查提示词**的 Claude skill（技能插件）。模式提炼自生产级大模型系统提示词与通用提示词工程实践，覆盖通用提示词工程 + agent / MCP 工具定义两块。

## 它能做什么

三种模式，一句话触发：

- **WRITE（写）** —— 描述任务，它先问清角色/约束/输出格式，再按骨架起草一份系统提示词、agent 指令或工具定义，交付前自检。
- **AUDIT（审）** —— 贴入现成提示词，它按一张 23 维质检量表逐条打分，给出「维度 · 严重度 · 原文 · 改写」。
- **REFERENCE（查）** —— 问「工具描述怎么写才不会被乱调用」之类，直接给标准 + 例子，当速查手册用。

内容沉淀在四个文件里：

```
prompt-architect/
├── SKILL.md                       # 模式判断 + 工作流 + 导航
└── references/
    ├── patterns.md                # 提示词模式速查（14 节 + 故障对照表）
    ├── tool-definitions.md        # agent & MCP / 工具定义设计标准
    └── audit-checklist.md         # 审查评分量表（23 维）
```

## 安装

**前提**：Claude Code 或支持 skill 的 Claude 客户端。

方式一（手动）：把 `prompt-architect/` 目录放进个人技能目录

```bash
cp -r prompt-architect ~/.claude/skills/
```

方式二（安装包）：在支持的客户端里导入 `prompt-architect.skill`。

装好无需重启。

## 使用

直接说，或 `/prompt-architect`：

- 「帮我写一个 XX 场景的系统提示词」
- 「审查一下我这段提示词：……」
- 「MCP 工具的 description 怎么写才规范？」

`examples/` 里有一个真实产出示例：批量 PE/VC 公允价值估值报告的提示词。

## 说明

- 它沉淀的是**通用提示词工程方法**，与具体模型无关——帮你打磨的提示词喂给任何大模型都能用。
- 这些模式是经验启发，不是铁律，请结合场景调整。
- 它用于设计正当的提示词与防护逻辑，**不用于**越狱、套取他人系统提示词或绕过安全机制。

## License

MIT
