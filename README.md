# sleep-thinking

一个让 AI agent「用最少 token 记住最多东西」的知识加载技能。

知识平时**沉睡**在记忆库（Obsidian vault）中，不占一分上下文；需要时才被**唤醒**——清醒调用（L2）、深眠唤起（L3），而常驻意识只保留一张极简索引（L1）。

```
L1 浅意识（常驻）  名称 + 一句话简介          → 每个知识只有一行进入上下文
L2 清醒思考（命中） Schema + 参数 + few-shot  → 意图判定命中才注入全文
L3 深眠唤起（触发） 错误 / 超时 / 长业务逻辑  → 仅片段级检索，读完全身而退
```

## 它解决什么问题

- **上下文膨胀**：启动只预载元数据索引，庞大资料库彻底脱离系统提示词与常规对话
- **意图错配**：先判定意图、再注入完整 Schema + 参数 + 2-3 个 few-shot 示例、最后才生成动作——预取是浪费，误判注入是污染
- **深水失联**：工具报错、执行超时、极长业务规则（复杂流程文档）时，才按 `grep → 图遍历 → 向量库` 路径开闸检索，且只注入片段、带来源标注、读完即走
- **记忆失序**：索引与笔记成对维护（先索引行、后笔记正文），双向链接成图，人机共享同一记忆库

## 三层协议

| 层 | 行为 | 内容 | 进入时机 |
|---|---|---|---|
| **L1 元数据索引层** | 常驻 | 名称 + 一句话简介 | 会话开始 |
| **L2 意图理解层** | 动态注入 | Schema + 参数定义 + few-shot | 意图匹配 |
| **L3 深水执行层** | 按需 RAG | `deep/` 长文档的片段 | 错误 / 超时 / 长业务逻辑 |

完整协议见 [SKILL.md](SKILL.md)；L1 索引文件规范见 [L1-INDEX-SPEC.md](L1-INDEX-SPEC.md)；L3 检索路径见 [L3-DEEP-RETRIEVAL.md](L3-DEEP-RETRIEVAL.md)。

## 安装

### skills.sh（Claude Code / Codex / Cursor / 任何支持 Agent Skills 的 agent）

```bash
npx skills@latest add <your-username>/sleep-thinking
```

### DeepSeek Harness（DSH）

把 `SKILL.md` 及两个资源文件放入技能目录：

```
~/.dsh/skills/sleep-thinking/
├── SKILL.md
├── L1-INDEX-SPEC.md
└── L3-DEEP-RETRIEVAL.md
```

### 手动（任何 agent）

把 `SKILL.md` 全文追加到 agent 的指令文件（CLAUDE.md / AGENTS.md / .cursorrules 等）。

## 记忆库（可选但推荐）

技能默认与 **Obsidian vault** 配合作为长期记忆载体（纯 Markdown + frontmatter + `[[wikilinks]]`，人机共享）。vault 路径按以下顺序解析：

1. 环境变量 `OBSIDIAN_VAULT_PATH`
2. agent 配置中声明的 vault 路径
3. `~/Documents/AgentMemory`

首次使用时对 agent 说「初始化记忆索引」，技能会按 L1-INDEX-SPEC 建立索引文件、创建 `deep/`（L3 深水区）与 `examples/`（few-shot 库）。

没有 vault 也能用：三层协议退化为对技能库与项目文档的加载纪律，L3 检索 `deep/` 变为检索项目内的长文档目录。

## 使用示例

| 你说 | agent 做什么 |
|---|---|
| 任意对话开始 | L1 定位：只读待办 + 项目概览（2 个文件） |
| 「检索知识 X」 | L1 索引判定 → L2 注入命中笔记全文 → 回答 |
| 工具报错 / 卡住 | L3 开闸：grep 定位 → 片段读取 → 来源标注 |
| 「记住 X」 | 先加索引行 → 再写笔记（frontmatter + 示例区） |
| 「今天到这」 | recap：session 总结 + 索引更新 + 待办回写 |

## 设计来源

三层加载策略与写作规范受以下项目启发：

- [mattpocock/skills](https://github.com/mattpocock/skills)：技能组织、调用二分、上下文指针、信息层级
- [obsidian-agent-memory-skills](https://github.com/AdamTylerLynch/obsidian-agent-memory-skills)：Obsidian 记忆库实践
- [writing-for-agents](https://github.com/mattpocock/skills/tree/main/skills/productivity/writing-for-agents)：写给 agent 的文档规范

## License

[MIT](LICENSE)
