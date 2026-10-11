# plugin: pi

把 [pi coding agent](https://pi.dev) 烘进 Termux bootstrap，使 APK 安装后**打开即进入 pi**。

## 内容

- `f-droid-patches/bootstrap-patches/pi-autolaunch.patch`
  往 `packages/bash/etc-bash.bashrc`（每个交互 shell 都会 source）追加一段：
  - 若 pi 未安装：缺 npm 时先 `pkg install -y npm`，再 `npm install -g --ignore-scripts @earendil-works/pi-coding-agent`；
  - 修 pi 启动器 shebang（Termux 无 `/usr/bin/env`，需指向 `$PREFIX/bin/node`）；
  - 直接 `node <pi_cli>` 启动 pi。
  - 防重入用 `PI_AUTOLAUNCHED`；创建 `~/.no-pi-autolaunch` 可禁用自动进入。

## 构建参数（关键）

```
--add nodejs,npm,git,termux-api,ca-certificates,fd,ripgrep
```

- **必须显式加 `npm`**：Termux 里 `npm` 是独立包（`nodejs` 只“推荐”它），只加 nodejs 不会带 npm。
- `fd` / `ripgrep` 为 pi 的可选工具（不加只是启动时警告）。
- `--plugin pi`、`--name com.pi.termux`、`--architectures aarch64`。

## ⚠️ SELinux / W^X（重要运行前提）

部分 ROM（如小米 HyperOS/Android 14）的 SELinux 策略**禁止 `untrusted_app` 执行 `app_data_file`**
（`avc denied { execute_no_trans }`）。Termux 的安装器要 exec 自带的 `xz` 解压 bootstrap，
被拒后表现为 **bootstrap 解压失败（usr 只出 etc/）**，且之后 shell 也无法 exec（`login: Permission denied`）。

在有 root（Magisk）的机器上加一条策略即可：

```
magiskpolicy --live "allow untrusted_app app_data_file file execute_no_trans"
magiskpolicy --live "allow untrusted_app app_data_file file execute"
```

持久化：放进 Magisk 模块的 `post-fs-data.sh` / `service.sh`。
在 SELinux 允许 Termux 式执行的 ROM 上无需此步。

## 另见

构建/验证细节见仓库根 `dev.md`。
