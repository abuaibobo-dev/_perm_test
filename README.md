# WebView Wrapper

最小可用的离线 Android 应用：一个全屏 WebView 加载 APK 内的 `assets/index.html`。

不申请网络权限，不依赖任何远程资源。它的存在意义只有一个：验证 **GitHub Actions 能在这台跑不了 aapt2 的设备之外把 Debug APK 构出来**。

## 结构

```
app/src/main/
├── AndroidManifest.xml
├── assets/index.html                  # 离线页面
├── java/.../MainActivity.java         # WebView 宿主
└── res/values/strings.xml
.github/workflows/build.yml            # 编译 Debug APK + 打 Release
```

## 本地构建

本机（Android PRoot / aarch64 / musl）**跑不了** `assembleDebug`——缺 aapt2 与 linker64。
所以只在 CI 构建：

```bash
# 推 main 即触发
git push origin main

# 或手动触发
gh workflow run build.yml --repo <owner>/<repo>
```

## 产物

CI 成功后：

- **Release**：`https://github.com/<owner>/<repo>/releases/tag/build-<run_number>`
- **Artifact**：run 页面的 Artifacts 区域

两条路都拿到同一个 `app-debug.apk`。

## 技术选型

| 项 | 值 | 原因 |
|---|---|---|
| AGP | 8.5.2 | 稳定，配合 Gradle 8.9 |
| Gradle | 8.9 | CI 用 `gradle/actions/setup-gradle` 显式指定，不依赖 wrapper |
| JDK | 17 | AGP 8.5 要求 17+ |
| compileSdk / targetSdk | 34 | 免去新 SDK 的 preview 噪声 |
| minSdk | 24 | 覆盖 ~98% 在用设备 |
| 语言 | Java | 无 Kotlin 插件，构建链最短 |
| 依赖 | 零 | WebView 是 framework 自带，不需要 androidx |
