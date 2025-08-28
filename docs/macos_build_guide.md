# macOS 编译指南

本文档详细说明了在 macOS 平台上编译 Anx Reader 时可能遇到的问题及解决方案。

## 常见问题及解决方案

### 1. Shell Script 构建警告

**问题描述：**
```
warning: Run script build phase 'Run Script' will be run during every build because it does not specify any outputs. To address this issue, either add output dependencies to the script phase, or configure it to run in every build by unchecking "Based on dependency analysis" in the script phase. (in target 'Flutter Assemble' from project 'Runner')
```

**解决方案：**

在 `macos/Runner.xcodeproj/project.pbxproj` 文件中，找到 `3399D490228B24CF009A79C7 /* ShellScript */` 段落，为其添加输出路径：

```pbxproj
3399D490228B24CF009A79C7 /* ShellScript */ = {
    isa = PBXShellScriptBuildPhase;
    alwaysOutOfDate = 1;
    buildActionMask = 2147483647;
    files = (
    );
    inputFileListPaths = (
    );
    inputPaths = (
    );
    outputFileListPaths = (
    );
    outputPaths = (
        "${PROJECT_DIR}/Flutter/ephemeral/.app_filename",
    );
    runOnlyForDeploymentPostprocessing = 0;
    shellPath = /bin/sh;
    shellScript = "echo \"$PRODUCT_NAME.app\" > \"$PROJECT_DIR\"/Flutter/ephemeral/.app_filename && \"$FLUTTER_ROOT\"/packages/flutter_tools/bin/macos_assemble.sh embed\n";
};
```

### 2. 代码签名错误

**问题描述：**
```
/Users/xxx/Projects/reader/anx-reader/macos/Runner.xcodeproj: error: No signing certificate "Mac Development" found: No "Mac Development" signing certificate matching team ID "28W956D5K8" with a private key was found. (in target 'Runner' from project 'Runner')
```

**解决方案：**

#### 方案一：开发环境（推荐用于本地测试）

1. **修改代码签名配置**

在 `macos/Runner.xcodeproj/project.pbxproj` 文件中，找到 Debug 配置段落并修改：

```pbxproj
33CC10FC2044A3C60003C045 /* Debug */ = {
    isa = XCBuildConfiguration;
    baseConfigurationReference = 33E5194F232828860026EE4D /* AppInfo.xcconfig */;
    buildSettings = {
        ASSETCATALOG_COMPILER_APPICON_NAME = AppIcon;
        CLANG_ENABLE_MODULES = YES;
        CODE_SIGN_ENTITLEMENTS = Runner/DebugProfile.entitlements;
        CODE_SIGN_IDENTITY = "-";
        CODE_SIGN_STYLE = Manual;
        COMBINE_HIDPI_IMAGES = YES;
        DEVELOPMENT_TEAM = "";
        // ... 其他配置保持不变
    };
    name = Debug;
};
```

2. **修改应用权限配置**

修改 `macos/Runner/DebugProfile.entitlements` 文件，禁用沙盒：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>com.apple.security.app-sandbox</key>
    <false/>
    <key>com.apple.security.cs.allow-jit</key>
    <true/>
    <key>com.apple.security.files.user-selected.read-write</key>
    <true/>
    <key>com.apple.security.network.client</key>
    <true/>
    <key>com.apple.security.network.server</key>
    <true/>
</dict>
</plist>
```

#### 方案二：生产环境（用于分发）

如果需要发布到 Mac App Store 或进行代码签名分发，需要：

1. **申请 Apple 开发者账户**
2. **在 Xcode 中配置团队和证书**
3. **恢复代码签名设置**：
   ```pbxproj
   CODE_SIGN_IDENTITY = "Apple Development";
   CODE_SIGN_STYLE = Automatic;
   DEVELOPMENT_TEAM = "YOUR_TEAM_ID";
   ```
4. **重新启用沙盒**：
   ```xml
   <key>com.apple.security.app-sandbox</key>
   <true/>
   ```

## 完整编译步骤

### 1. 环境准备

确保已安装：
- Flutter SDK
- Xcode (最新版本)
- CocoaPods

### 2. 检查代码签名身份

```bash
security find-identity -v -p codesigning
```

如果输出 `0 valid identities found`，说明没有安装开发证书，建议使用方案一进行开发。

### 3. 清理和构建

```bash
# 进入项目目录
cd /path/to/anx-reader

# 清理项目
flutter clean

# 获取依赖
flutter pub get

# 生成多语言文件
flutter gen-l10n

# 生成代码
dart run build_runner build --delete-conflicting-outputs

# 构建 macOS 应用
flutter build macos --debug
```

### 4. 运行应用

```bash
flutter run -d macos
```

## 重要说明

### 开发环境配置

- **适用场景**：本地开发和测试
- **限制**：生成的应用无法在其他机器上运行或分发
- **优点**：无需 Apple 开发者账户，配置简单

### 生产环境配置

- **适用场景**：应用分发、Mac App Store 上架
- **要求**：需要 Apple 开发者账户和相应证书
- **注意**：必须启用沙盒和正确的权限配置

## 故障排除

### 构建失败

1. **清理项目**：`flutter clean`
2. **删除 Pods**：`rm -rf macos/Pods macos/Podfile.lock`
3. **重新安装依赖**：`cd macos && pod install`
4. **重新构建**：`flutter build macos --debug`

### 权限问题

如果应用运行时出现权限相关错误：
1. 检查 `DebugProfile.entitlements` 文件配置
2. 确认沙盒设置是否正确
3. 验证网络、文件访问权限是否已添加

### 代码签名问题

1. 使用 `security find-identity -v -p codesigning` 检查证书
2. 在 Xcode 中检查团队配置
3. 验证 Provisioning Profile 是否正确

## 相关链接

- [Flutter macOS 部署文档](https://docs.flutter.dev/deployment/macos)
- [Apple 开发者文档](https://developer.apple.com/documentation/)
- [Xcode 代码签名指南](https://developer.apple.com/documentation/xcode/code-signing-your-app)
