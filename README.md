# opencode-skill-tool-call-discipline

> opencode 技能：五条硬性规则，防止工具调用序列白白浪费轮次 —— 默认并行、正确工具一次到位、读窗口够宽、参数自检、已确认的事实不再重验。

隶属 [`opencode-codex-kit`](https://github.com/Yulimfish/opencode-codex-kit)。与 [`opencode-codex-guardrails`](https://github.com/Yulimfish/opencode-codex-guardrails) 里针对 `cat`/`head`/`tail`/`find`/`grep`/`sed`/`awk` 的软提示配套使用。

## 五条规则（TL;DR）

1. **默认并行。** 相互独立的工具调用**必须**放在同一条消息里。
2. **正确工具一次到位。** 文件操作用 Read/Edit/Write/Glob/Grep —— 永远不要 shell out 到 `cat`/`head`/`tail`/`find`/`sed`/`awk`。
3. **读窗口 ≥ 100 行。** 别再 30 行一片地反复重读。
4. **Edit/Write 前必先 Read。** 工具本身的 guardrail 会拦，别浪费一轮。
5. **每次调用前参数三查：** 绝对路径、字段名逐字照抄、拒绝编造未知值。

完整参考示例与反面模式见 `SKILL.md`。

## 安装

```bash
mkdir -p ~/.config/opencode/skills
git clone https://github.com/Yulimfish/opencode-skill-tool-call-discipline.git \
  ~/.config/opencode/skills/tool-call-discipline
```

## 许可

MIT © Yulimfish
