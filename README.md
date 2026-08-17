# dsh-desktop-skill

DSH Desktop（DeepSeek Harness 桌面端）专属技能与记忆库。

## 内容

| 技能 | 说明 |
| --- | --- |
| `SKILL.md` (dsh-plugin-profile-preference) | 本机 DSH 插件/皮肤安装规则：用户只使用 DSH Desktop，Web GUI 由 `desktop` profile 驱动，所有 `dsh plugin` 安装/更新必须目标 `--profile desktop`。附带完整安装流程、pnpm 门禁配置与排障知识。 |

## 安装到本机

将技能目录放入技能仓库（Windows 默认 `C:\Users\<user>\.agents\skills\`）：

```powershell
# 克隆
git clone https://github.com/mzk-C4/dsh-desktop-skill.git
# 复制技能目录（仓库根目录的 SKILL.md 即技能本体，按需放入以技能名命名的子目录）
```

> 注意：技能内容含本机环境信息（用户目录、安装路径），如需分享请先脱敏。
