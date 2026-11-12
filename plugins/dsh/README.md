# plugin: dsh

把 [DeepSeek Harness](https://www.npmjs.com/package/@deepseek-ai/dsh) 烘进 Termux bootstrap，
使 APK 安装后**打开即自动装好 dsh 并启动本地 Web UI**。

## 内容

`f-droid-patches/bootstrap-patches/dsh-autolaunch.patch`
往 `packages/bash/etc-bash.bashrc`（每个交互 shell 都会 source）追加一段：

1. 若 dsh 未安装：缺 npm 时先 `pkg install -y npm`，再
   `npm install -g --ignore-scripts --omit=optional @deepseek-ai/dsh@0.2.2-rc.1`
   + `@jiesou/dsh-commandcode-go-provider`（Go 订阅走 `/alpha/generate`）。
2. 打三个 Android 必需补丁（均在运行期幂等打一次）：
   - `node-addon-require-builtin` 无 android 预编译 → 换成走 `--expose-internals` 的垫片
   - `flock` 在 Android 不可用 → 换成进程内假锁（单进程宿主）
   - Worker `execArgv: []` 会丢掉 `--expose-internals` → 预加载脚本补回
3. `CC_VERSION` 钉到网关要求的最低版本（插件内置的是 1.74.0，网关要求 >= 0.18.10）。
4. 启动 `dsh web --port 3080 --no-open`，终端里打印访问地址。

## 运行期环境

```
DSH_HOME=$HOME/.dsh
DSH_PERMISSION_MODE=danger-full-access   # Android 无 sandbox 后端
DSH_TELEMETRY_DISABLED=1
```

首启后在浏览器打开 `http://127.0.0.1:3080`，到 Models 页存 `COMMANDCODE_API_KEY`，
再建会话。终端窗口关掉服务就停。

## 构建参数

```
--add nodejs,npm,git,bash,ca-certificates,termux-api,fd,ripgrep
--plugin dsh
--name com.dsh.termux
--architectures aarch64
--type f-droid
```

- **必须显式加 `npm`**：Termux 里 npm 是独立包。
- 不加 `nodejs-lts`：本仓库 bootstrap 的 `nodejs` 已带 `node:sqlite`（dsh 需要）。

## 注意

- 与 `plugin: pi` **同时启用会冲突**：两者都往 `.bashrc` 插入自动启动，先到的那个会占住 shell。
- 禁用自动进入：`touch ~/.no-dsh-autolaunch`，之后手动 `bash` 再执行安装脚本。
- 自定义端口：`export DSH_PORT=3081` 后再开 Termux。
