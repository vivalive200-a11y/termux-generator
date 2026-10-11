# termux-pi

基于 [robertkirkman/termux-generator](https://github.com/robertkirkman/termux-generator) 的自定义分支，
产出一个 **安装后打开即进入 pi 编码 agent** 的 Termux APK（Android）。

## 目标

手机装一个 APK → 打开 → 直接是 `pi`（本地在 Termux 里跑，非远程遥控）。

## 机制

- 上游 `termux-generator` 用 GitHub Actions（`.github/workflows/build-termux.yml`，`workflow_dispatch`）
  在云端构建：克隆 `termux/termux-packages` → 应用补丁 → 编译/组装 bootstrap + app → 产出 APK。
- 注入点：`termux-packages/packages/bash/etc-bash.bashrc` 会被交互 shell source。插件 `plugin/pi` 往它追加
  “首启装 npm+pi + 修正 shebang + 拉起 pi”。

## 构建参数

```
--name com.pi.termux
--add nodejs,npm,git,termux-api,ca-certificates,fd,ripgrep   # npm 必须显式加（独立包）
--plugin pi
--architectures aarch64
--type f-droid
```

触发：

```
gh workflow run build-termux.yml -R vivalive200-a11y/termux-generator \
  -f add="nodejs,npm,git,termux-api,ca-certificates,fd,ripgrep" \
  -f name="com.pi.termux" -f architectures="aarch64" -f plugin="pi"
```

产物：artifact `Termux APKs or bootstraps`。其中 `...-termux-app_apt-android-7-debug_universal.apk`
（含 `assets/bootstrap-aarch64.tar.xz`）就是要装的 App。

## 关键发现（2026-10-11 实机 Redmi Note 12 Turbo / HyperOS / 已 root）

1. **MIUI 安装闸门**：`adb install` 报 `INSTALL_FAILED_USER_RESTRICTED`。手机上“开发者选项 →
   USB 安装”需开；且每次 `adb install` 会弹“USB安装提示/继续安装”，**10 秒倒计时自动拒绝**，
   需在倒计时内点“继续安装”（可用 `adb shell input tap`）。或 `settings put global adb_install_need_confirm 0` 后重试。
2. **SELinux 是核心坑**：该 ROM 拒绝 `untrusted_app` 执行 `app_data_file`
   （`avc denied execute_no_trans`）。导致 (a) app 原生解压 bootstrap 失败（安装器要 exec 自带 xz），
   (b) shell 无法 exec `login`。加 Magisk 策略后**两者皆解决**（原生解压成功、shell 正常）。
   详见 `plugins/pi/README.md`。
3. **npm 是独立包**：本版 Termux 的 `nodejs` 只“推荐” npm，bootstrap 里 `bin/node` 在但**无 npm/npx**。
   → 构建必须 `--add npm`。
4. **pi shebang**：pi 的 `cli.js` shebang 为 `#!/usr/bin/env node`，Termux 无 `/usr/bin/env` → 需改指向
   `$PREFIX/bin/node`（插件脚本已处理）。
5. **dpkg 不可用**：实测 `pkg install` 的 dpkg 解包报 `error creating directory './data/data/com.pi.termux':
   Permission denied`。故 npm 走构建期烘入（而非运行期 pkg），pi 走 npm registry 安装。

## 实机验证（已通过）

- App 安装后打开 → 自动进入 `pi v1.1.0`（在 Redmi Note 12 Turbo 上）。
- 环境：Termux 0.118.0 / node v26.4.0 / npm 11.20.0 / pi 1.1.0。

## 状态

- [x] fork + 插件 + 补丁（可干净套用）
- [x] 首版构建 + 下载 + 实机安装
- [x] 打通 SELinux / npm / shebang 全部环节，实机进入 pi
- [ ] 用 `--add npm,fd,ripgrep` 重建自足版 APK
- [ ] 验证重建版（无需手动补 npm）

## 待办 / 已知局限

- 运行前提：目标 ROM 若同款 SELinux 限制，需 root 加策略（见插件 README）。
- 剪贴板仅文本（Termux 限制）。
- 产物为 debug 签名；分发需自行签名。
- 禁用自动进入 pi：`touch ~/.no-pi-autolaunch`。
