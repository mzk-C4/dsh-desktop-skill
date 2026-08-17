---
name: dsh-plugin-profile-preference
description: 本机 DSH 插件/皮肤安装规则：用户只使用 DSH Desktop（桌面端），Web GUI 由 desktop profile 驱动，所有 dsh plugin 安装/更新必须目标 `--profile desktop`，禁止装到 web profile。附带本机完整安装流程与排障知识（CLI 调用方式、pnpm 门禁、验证方法）。
whenToUse: 用户要求安装/更新/卸载 DSH 插件、皮肤、web-ui 全家桶、任务看板、皮肤中心等；或询问为什么插件/皮肤没出现在 GUI；或提到 profile / plugin add / dsh-web-ui。
---

# DSH 插件安装规则（本机专属记忆）

## 铁律（必须遵守）

1. **本机 DeepSeek Harness 用户只使用桌面端（DSH Desktop，Electron）**。
2. DSH Desktop 的 Web GUI 由 **`desktop` profile** 驱动：
   - profile 目录：`C:\Users\HP\.dsh\profiles\desktop`
   - 选择记录：`C:\Users\HP\AppData\Roaming\DSH Desktop\profile-selection\state.json`（`active: "desktop"`）
3. **任何插件/皮肤安装、更新、卸载，一律使用 `--profile desktop`**：
   ```sh
   dsh plugin --profile desktop add <package>
   dsh plugin --profile desktop update <package>
   dsh plugin --profile desktop remove <package>
   ```
4. **禁止**把插件装进 `web` profile（`C:\Users\HP\.dsh\profiles\web`）——web profile 仅供 CLI 使用，DSH Desktop 不会加载它，装了等于没装（皮肤中心、任务看板等都不会出现）。
5. 安装前先做筛查：
   - 确认 `profile-selection\state.json` 的 active profile 是 `desktop`；
   - 检查目标包是否已存在于 `C:\Users\HP\.dsh\profiles\desktop\node_modules\@linxin666\`（或对应 scope）；
   - 若发现之前误装到 web profile，在 desktop profile 重装，并提示用户 web 那份可留可卸。

## 本机 dsh CLI 的正确调用方式

`dsh` 不在 PATH。必须用 Electron 的 Node 模式调用（与 DSH Desktop 自带 `pnpm.cmd` 同款）：

```powershell
$env:ELECTRON_RUN_AS_NODE = "1"
$env:PATH = "C:\Users\HP\AppData\Roaming\DSH Desktop\runtime-commands\private\node-bin;$env:PATH"
& "D:\Program Files\DeepSeek Harness\DSH Desktop\DSH Desktop.exe" `
  --import "file:///C:/Users/HP/AppData/Roaming/DSH%20Desktop/runtime-commands/private/clear-env.mjs" `
  "D:\Program Files\DeepSeek Harness\resources\host\node_modules\@deepseek-ai\dsh\lib\bin.js" `
  plugin --profile desktop add <package>
```

要点：
- 直接运行 host 的 `bin.js` 会崩溃（Electron crashpad），必须走 `DSH Desktop.exe` + `ELECTRON_RUN_AS_NODE=1`；
- 网络：本机 Windows schannel 组件异常（`SEC_E_NO_CREDENTIALS`），curl/git 直连 GitHub 失败；npm/pnpm 走 Node(OpenSSL) 直连 registry.npmjs.org 正常（Clash 代理可用时也可走 `http://127.0.0.1:7897`，但代理可能中途退出）。

## 本机 profile 的 pnpm 配置（已修好，勿回退）

`C:\Users\HP\.dsh\profiles\desktop\pnpm-workspace.yaml` 必须包含（安装 @linxin666 全家桶所需）：

```yaml
nodeLinker: hoisted
autoInstallPeers: false
minimumReleaseAgeExclude:
  - '@linxin666/*'
allowBuilds:
  cloudflared: true
  cpu-features: true
  ssh2: true
```

- 缺 `allowBuilds` → `ERR_PNPM_IGNORED_BUILDS`；
- 缺 `minimumReleaseAgeExclude` → pnpm 11 门禁静默装回旧版（如 0.1.15 而非 0.1.18）；
- `web` profile 的 `pnpm-workspace.yaml` 同样已配置（2026-08-16）。

## 已安装状态（截至 2026-08-16）

- `desktop` profile：`@linxin666/dsh-web-ui-all@0.1.18`（皮肤中心 + 10 款皮肤：qq98/xp/blue-fantasy/dragon-heir/miku/minecraft/qq2006/ths/trading/whale-song；另有 task-board/git-graph/pet/ssh/live-stats/liangshen 等 13 个子包）。
- 皮肤启用入口：GUI「设置 → 皮肤中心」，先试穿再应用。
- cpu-features/ssh2 原生模块构建失败属环境缺 node-gyp，可忽略（可选加速组件）。

## 安装后验证流程

1. 配置层：`dsh --profile desktop --dump-config` 输出含 `@linxin666/dsh-web-ui-all` 与 `ui-skin-center` 等 insert 行；
2. 运行时（GUI 已重启后）：GET `http://127.0.0.1:6031/` 的 HTML 中 `window.__DSH_BOOT__` 的 `entries` 数组应包含 `@linxin666/dsh-*` 条目（bundle URL 形如 `/plugins/@linxin666/<pkg>/client.js?rev=...`）；
3. 插件改动必须**完全退出并重启 DSH Desktop**（刷新页面无效）。

## 排障速查

- **GUI 里没有插件/皮肤中心** → 先查 `__DSH_BOOT__` entries；没有 @linxin666 → 查 active profile 是否 desktop、包是否装在 desktop profile；
- **手动 boot 报 `Cannot find package '@linxin666/...' imported from cordis-plugin-loader`** → 裸包名解析依赖 DSH Desktop 主进程注册的 Node hooks（`app.asar.unpacked\lib\module-resolution.js` 的 `installProfilePackageResolver`，把 loader 的 bare import 重定向到 profile 目录）；CLI 手动跑 `dsh --profile desktop web` 不带该 hooks 时会失败，属正常，不代表 Desktop 起不来；
- **`ERR_PNPM_IGNORED_BUILDS` / 旧版本回退** → 见上文 pnpm 配置；
- **GUI 端口**：DSH Desktop 的 web 服务监听 `127.0.0.1:6031`（进程为 DSH Desktop 主进程）。
