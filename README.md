# 宝宝成长册（babybook）

一个单文件网页 + Capacitor 壳，打包成安卓 APK。给不擅长电脑的家人用来记录孩子成长。

## 目录

```
babybook/
├─ .github/workflows/build-apk.yml   # GitHub Actions 打包配方
├─ capacitor.config.json             # App 名字 / 包名 / webDir
├─ package.json                      # 依赖（Capacitor 8.5.3）
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
| compileSdk / targetSdk | 36（Android 16） |
| 工具链 | Node 22 / JDK 21 / AGP 8.13 / Gradle 8.14.3 |
| 产物 | `app-debug.apk`，约 5 MB |

## 已知的坑

- **debug 包签名每次构建都不同**：覆盖安装会失败，只能卸载重装（数据清空）。正式使用前要生成固定 keystore 并配置签名。
- **`.github` 是隐藏目录**：用网页拖拽上传时最容易漏，漏了 Actions 不会触发。
- **本机没有系统级 git**：需要装 Git for Windows；DSH 自带的便携 git 路径随会话变化，不能用。
- **沙箱内建不了 `<工作区>\.cache`**（Permission denied）：npm 缓存改用 `babybook/npm-cache`，已 gitignore。
- **WebView 里点“选照片”**：Capacitor 支持文件选择器；拍照功能需要额外声明 CAMERA 权限，暂未配。
- **没有云同步**：数据存在手机浏览器的 IndexedDB 里，换机/卸载即丢，导出备份是必选项。

## 命令讲解

逐条拆解（cd / git remote / git push / npm / cap / gradlew 等）见 `<工作区>/out/命令讲解-20261028.md`。
