# GitHub Actions 自动化打包使用说明

## 📋 文件位置

工作流文件：`.github/workflows/build_all_platforms_new.yml`

## 🚀 使用方法

### 方法 1：手动触发（推荐用于测试）

1. 进入 GitHub 仓库页面
2. 点击 **Actions** 标签
3. 选择 **Build All Platforms** 工作流
4. 点击右侧 **Run workflow** 按钮
5. 选择要构建的平台：
   - ✅ 构建 Android
   - ✅ 构建 iOS
   - ✅ 构建 macOS
   - ✅ 构建 Windows
   - ✅ 构建 Linux
   - ☐ 构建 Android TV（默认不选）
   - ☐ 创建 GitHub Release（默认不选）
6. 点击绿色的 **Run workflow** 按钮开始构建

### 方法 2：推送 Tag 自动触发（正式发布）

```bash
# 创建并推送 tag
git tag v1.0.0
git push origin v1.0.0
```

推送 tag 后会自动：
- 构建所有平台（Android、iOS、macOS、Windows、Linux、Android TV）
- 自动创建 GitHub Release
- 上传所有构建产物到 Release

## 📦 构建产物

构建完成后，可以在以下位置找到产物：

### Artifacts（临时文件，保留 7 天）
- **android-apk**: Android APK 文件
- **ios-ipa**: iOS IPA 文件（未签名）
- **macos-dmg**: macOS DMG 和 ZIP 文件
- **windows-msix**: Windows MSIX 和 ZIP 文件
- **linux-deb**: Linux DEB 和 ZIP 文件
- **android-tv-apk**: Android TV APK 文件

### Release（永久保存）
如果选择创建 Release 或推送 tag，所有文件会上传到 GitHub Release。

## 🔐 Android 签名配置（可选）

如果需要发布正式版本，需要配置 Android 签名：

1. 准备签名文件（keystore.jks）
2. 将签名文件转换为 Base64：
   ```bash
   base64 -i keystore.jks | pbcopy  # macOS
   # 或
   base64 -w 0 keystore.jks  # Linux
   # 或
   certutil -encode keystore.jks keystore.txt  # Windows
   ```

3. 在 GitHub 仓库设置中添加 Secrets：
   - **Settings** → **Secrets and variables** → **Actions** → **New repository secret**
   - 添加以下 Secrets：
     - `KEYSTORE_BASE64`: 签名文件的 Base64 编码
     - `STORE_PASSWORD`: KeyStore 密码
     - `KEY_PASSWORD`: Key 密码
     - `KEY_ALIAS`: Key 别名

如果没有配置签名，会自动构建 Debug 版本（仅用于测试）。

## 📱 各平台说明

### Android
- **有签名**: 构建 Release APK（可发布）
- **无签名**: 构建 Debug APK（仅测试）
- 输出格式：`app-arm64-v8a-release.apk`、`app-armeabi-v7a-release.apk`、`app-x86_64-release.apk`

### iOS
- 构建未签名的 IPA
- 需要自行签名后才能安装
- 输出格式：`ios_no_sign.ipa`

### macOS
- 构建 DMG 安装包和 ZIP 压缩包
- 输出格式：`.dmg`、`.zip`

### Windows
- 构建 MSIX 安装包和 ZIP 压缩包
- 输出格式：`.msix`、`.zip`

### Linux
- 构建 DEB 安装包和 ZIP 压缩包
- 输出格式：`.deb`、`.zip`

### Android TV
- 构建 Debug APK（用于电视盒子）
- 输出格式：`app-debug.apk`

## ⚙️ 高级配置

### 修改 Flutter 版本

编辑 `.github/workflows/build_all_platforms_new.yml`：

```yaml
env:
  FLUTTER_VERSION: '3.38.x'  # 修改这里
  JAVA_VERSION: '17'
```

### 修改版本信息

创建或编辑 `assets/app_version.json`：

```json
{
  "version": "1.0.0",
  "version_num": 10000,
  "version_desc": "- 新功能说明\n- Bug 修复",
  "prerelease": false,
  "download_url": "https://github.com/你的用户名/dart_simple_live/releases"
}
```

## 🐛 常见问题

### 1. 构建失败：找不到签名文件
**解决方法**：不配置签名 Secrets，会自动构建 Debug 版本

### 2. macOS 构建失败
**解决方法**：检查 `appdmg` 是否安装成功，或跳过 DMG 只构建 ZIP

### 3. Linux 构建失败：缺少依赖
**解决方法**：检查系统依赖是否完整安装（libmpv-dev、libgtk-3-dev 等）

### 4. 构建时间过长
**解决方法**：
- 只选择需要的平台进行构建
- 利用 GitHub Actions 缓存（已配置）

## 📊 构建时间参考

| 平台 | 预计时间 | Runner |
|------|---------|--------|
| Android | 10-15 分钟 | Ubuntu/macOS |
| iOS | 15-20 分钟 | macOS |
| macOS | 20-25 分钟 | macOS |
| Windows | 15-20 分钟 | Windows |
| Linux | 15-20 分钟 | Ubuntu |
| Android TV | 10-15 分钟 | Ubuntu |

**并行构建总时间**：约 25-30 分钟（所有平台同时进行）

## 🎯 最佳实践

1. **开发阶段**：只构建需要测试的平台，节省时间
2. **测试阶段**：构建主要平台（Android、Windows、macOS）
3. **正式发布**：推送 tag，自动构建所有平台并发布

## 📝 示例工作流

```bash
# 1. 开发完成后，提交代码
git add .
git commit -m "feat: 添加新功能"
git push

# 2. 手动触发 Actions 测试 Android 版本
# 在 GitHub Actions 页面手动运行，只选择 Android

# 3. 测试通过后，更新版本信息
# 编辑 assets/app_version.json

# 4. 创建 tag 并推送（自动构建所有平台并发布）
git tag v1.0.0
git push origin v1.0.0

# 5. 等待构建完成，检查 Release
```

## ✅ 完成

现在您可以开始使用 GitHub Actions 自动化打包了！
