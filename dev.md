# termux-pi

基于 [robertkirkman/termux-generator](https://github.com/robertkirkman/termux-generator) 的自定义分支，
产出一个 **安装后打开即进入 pi 编码 agent** 的 Termux APK（Android）。

## 目标

手机装一个 APK → 打开 → 直接是 `pi`（本地在 Termux 里跑，非远程遥控）。

## 机制

- 上游 `termux-generator` 用 GitHub Actions（`.github/workflows/build-termux.yml`，`workflow_dispatch`）
  在云端构建：克隆 `termux/termux-packages` → 应用补丁 → 编译/组装 bootstrap + app → 产出 APK。
- 注入点：`termux-packages/packages/bash/etc-bash.bashrc` 会被每个交互 shell source
  （`etc-profile` 中已确认）。插件 `plugin/pi` 往它追加"首启装 pi + 拉起 pi"。

## 构建参数

```
--name com.pi.termux            # 自定义包名，与官方 Termux 共存不冲突
--add nodejs,git,termux-api,ca-certificates   # pi 运行所需，烘进 bootstrap
--plugin pi                     # 注入 pi 自动安装+启动
--architectures aarch64         # 现代手机
--type f-droid
```

## 触发

```
gh workflow run build-termux.yml -R vivalive200-a11y/termux-generator \
  -f add="nodejs,git,termux-api,ca-certificates" \
  -f name="com.pi.termux" \
  -f architectures="aarch64" \
  -f plugin="pi"
```

产物：workflow 的 artifact `Termux APKs or bootstraps`（`*.apk`）。

## 注意

- 自定义包名 → bootstrap 需重编译（含 nodejs），云端构建较慢（可能 1h+）。
- 首启需联网（npm 装 pi）。想彻底离线需把 pi 的 node_modules 烘进 bootstrap（未做，APK 会 +几十 MB）。
- 剪贴板仅文本（Termux 限制），贴图不支持。
- 产物为 debug 签名；分发需自行签名。
- 禁用自动进入 pi：`touch ~/.no-pi-autolaunch`。

## 状态

- [x] fork 仓库、克隆到本目录
- [ ] 提交插件、激活 Actions
- [ ] 触发构建
- [ ] 下载并验证 APK
- [ ] （有手机时）adb install 验证
