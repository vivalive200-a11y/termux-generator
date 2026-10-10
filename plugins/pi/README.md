# plugin: pi

把 [pi coding agent](https://pi.dev) 烘进 Termux bootstrap：

- `f-droid-patches/bootstrap-patches/pi-autolaunch.patch`
  往 `packages/bash/etc-bash.bashrc`（每个交互 shell 都会 source）追加一段：
  首启用 npm 自动安装 `@earendil-works/pi-coding-agent`，随后直接进入 `pi`。
  用 `PI_AUTOLAUNCHED` 防重入；创建 `~/.no-pi-autolaunch` 可禁用自动进入。

配套构建参数（见根目录 dev.md）：

```
--add nodejs,git,termux-api,ca-certificates --plugin pi --name com.pi.termux --architectures aarch64
```
