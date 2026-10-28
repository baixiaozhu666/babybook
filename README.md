# 宝宝成长册（babybook）

一个单文件网页 + Capacitor 壳，打包成安卓 APK。给不擅长电脑的家人用来记录孩子成长。

## 目录

```
babybook/
├─ .github/workflows/build-apk.yml   # GitHub Actions 打包配方
├─ capacitor.config.json             # App 名字 / 包名 / webDir
├─ package.json                      # 依赖（Capacitor 8.5.3 + App/Filesystem/Share 插件）
├─ www/index.html                    # ★ 界面全部在这一个文件里
└─ android/                          # 生成物，已 gitignore，CI 每次重建
```

## 改代码后的流程

1. 编辑 `www/index.html`
2. `git add -A && git commit -m "改了什么" && git push`
3. 仓库 Actions 页等 3–5 分钟，下载 Artifacts 里的 apk

## 关键参数

| 项 | 值 |
|---|---|
| 包名 | `com.babybook.growth` |
| 显示名 | 宝宝成长册 |
| minSdk | 24（安卓 7.0+） |
| compileSdk | 36（Android 16） |
| targetSdk | 36 → 构建时补丁改成 **34**（见下） |
| 工具链 | Node 22 / JDK 21 / AGP 8.13 / Gradle 8.14.3 |
| 产物 | `app-debug.apk`，约 4 MB |

## 三个原生插件（网页里用 `Capacitor.registerPlugin` 取）

| 插件 | 用途 |
|---|---|
| `@capacitor/app` | 接管安卓返回键（弹层打开时只关弹层，不退出 app） |
| `@capacitor/filesystem` | 把备份 json 写到缓存目录 |
| `@capacitor/share` | 调起系统分享面板，把备份发到微信/QQ/网盘 |

网页里全部通过 `isNative` 判断，在浏览器打开时自动降级（下载 / 无返回键接管），所以同一个 `www/index.html` 既能双击看，也能打包。

## targetSdk 为什么要降到 34

安卓 15 起，`targetSdk ≥ 35` 的应用强制“内容延伸到状态栏下面”，表现为标题被时间和电量压住。降到 34 就不再强制，系统自动给状态栏留高度。配方里用 `sed` 改 `android/variables.gradle`，因为 `android/` 是每次 CI 重新生成的，改不了仓库里的文件。

## 签名怎么固定下来的

`android/` 每次 CI 重建，debug 签名默认也是每次新生成，导致“新版本覆盖安装失败、只能卸载”。

现在的做法（不需要任何密钥文件进仓库）：

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.android/debug.keystore
    key: android-debug-keystore-v1
```

第一次构建时 Gradle 生成 `~/.android/debug.keystore`，任务结束由 cache 保存；之后每次构建先把它取回来，签名就一致了。

构建日志里有“打印签名指纹”一步，输出 `SHA1: XX:XX:...`。**两次构建的这串字符相同 = 签名稳定，可以覆盖安装。**

风险：GitHub 的缓存 7 天不访问会被清理。缓存没了以后签名会变，届时需要卸载重装一次。要彻底稳妥，得把 keystore 放进 GitHub Secrets（需要你手动粘贴几个值），目前先用缓存方案。

## 已知的坑

- **`.github` 是隐藏目录**：用网页拖拽上传时最容易漏，漏了 Actions 不会触发。
- **本机没有系统级 git**：需要装 Git for Windows；DSH 自带的便携 git 路径随会话变化，不能用。
- **沙箱内建不了 `<工作区>\.cache`**（Permission denied）：npm 缓存改用 `babybook/npm-cache`，已 gitignore。
- **生成本地 keystore 会被安全策略拦住**（工具侧禁止触碰 keystore 文件），所以签名走 Actions 缓存方案。
- **拍照按钮**：用 `<input capture="environment">`，能不能直接调起相机取决于外壳是否转发该参数；不行就退回相册选择。
- **照片存在 IndexedDB 里（base64）**：一张 200–400 KB，上千张可能撑爆配额。计划改成存文件 + 只记路径。
- **没有云同步**：数据只在手机上，换机/卸载即丢，导出备份是必选项。

## 命令讲解

逐条拆解（cd / git remote / git push / npm / cap / gradlew 等）见 `<工作区>/out/命令讲解-20261028.md`。
