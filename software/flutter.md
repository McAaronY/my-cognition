# Flutter 开发完整指南

## 环境配置

### 系统要求

- **操作系统**: Windows 10/11, macOS 10.14+, Linux Ubuntu 20.04+
- **磁盘空间**: 至少 2.8 GB 可用空间
- **内存**: 推荐 8GB 以上

### macOS 环境配置

#### 安装步骤

1. **下载 Flutter SDK**

   ```bash
   git clone https://github.com/flutter/flutter.git -b stable
   ```

2. **配置环境变量**
   在 `~/.zshrc` 或 `~/.bash_profile` 中添加：

   ```bash
   export PATH="$PATH:[PATH_TO_FLUTTER_GIT_DIRECTORY]/flutter/bin"
   export PATH="$PATH:[PATH_TO_FLUTTER_GIT_DIRECTORY]/flutter/bin/cache/dart-sdk/bin"
   ```

3. **安装 Xcode**

   - 从 Mac App Store 安装 Xcode
   - 配置 Xcode 命令行工具：

   ```bash
   sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
   sudo xcodebuild -runFirstLaunch
   ```

4. **安装 Homebrew（推荐）**

   ```bash
   /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
   ```

5. **安装 Android Studio**
   - 下载并安装 Android Studio
   - 安装 Flutter 和 Dart 插件

#### macOS 特定命令

```bash
# 重新加载shell配置
source ~/.zshrc

# 接受Xcode许可
sudo xcodebuild -license

# 安装CocoaPods（用于iOS开发）
sudo gem install cocoapods
```

### Linux 环境配置

#### 系统要求

- **推荐发行版**: Ubuntu 20.04 LTS 或更高版本
- **架构**: x86_64

#### 安装步骤

1. **下载 Flutter SDK**

   ```bash
   git clone https://github.com/flutter/flutter.git -b stable
   ```

2. **安装系统依赖**

   ```bash
   sudo apt-get update
   sudo apt-get install curl file git unzip xz-utils zip libglu1-mesa clang cmake ninja-build pkg-config libgtk-3-dev
   ```

3. **配置环境变量**
   在 `~/.bashrc` 中添加：

   ```bash
   export PATH="$PATH:[PATH_TO_FLUTTER_GIT_DIRECTORY]/flutter/bin"
   ```

4. **安装 Android Studio**
   - 下载并安装 Android Studio
   - 配置 Android SDK

#### Linux 特定命令

```bash
# 重新加载配置
source ~/.bashrc

# 安装额外的开发库
sudo apt-get install libssl-dev

# 配置udev规则（用于Android设备调试）
sudo usermod -a -G plugdev $LOGNAME
```

### 通用配置步骤

1. **运行环境检查**

   ```bash
   flutter doctor
   ```

2. **接受 Android 许可**

   ```bash
   flutter doctor --android-licenses
   ```

3. **安装桌面开发支持（可选）**

   ```bash
   # macOS
   flutter config --enable-macos-desktop

   # Linux
   flutter config --enable-linux-desktop
   ```

4. **配置 VS Code（推荐）**
   - 安装 Flutter 和 Dart 扩展
   - 配置 Flutter SDK 路径

### 常见问题解决

#### macOS 问题

- **"command not found: flutter"**: 重新加载 shell 配置
- **Xcode 许可问题**: 运行 `sudo xcodebuild -license`
- **CocoaPods 安装失败**: 使用 Homebrew 安装 `brew install cocoapods`

#### Linux 问题

- **缺少 libssl-dev**: `sudo apt-get install libssl-dev`
- **权限问题**: 将用户添加到 plugdev 组
- **图形库问题**: 安装 libgtk-3-dev

### 验证安装

```bash
# 创建测试项目
flutter create my_app

# 运行项目
cd my_app
flutter run
```

## 项目结构

```
my_flutter_app/
├── android/          # Android平台特定代码
├── ios/              # iOS平台特定代码
├── lib/              # Dart主要代码目录
│   ├── main.dart     # 应用入口点
│   ├── models/       # 数据模型
│   ├── services/     # 业务逻辑服务
│   ├── widgets/      # 自定义组件
│   └── pages/        # 页面文件
├── test/             # 测试文件
├── pubspec.yaml      # 项目依赖配置
└── README.md
```

## 核心概念

### Widget 组件系统

Flutter 一切皆 Widget，分为两大类：

**StatelessWidget** - 无状态组件

```dart
class MyText extends StatelessWidget {
  final String content;

  const MyText({Key? key, required this.content}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return Text(content);
  }
}
```

**StatefulWidget** - 有状态组件

```dart
class Counter extends StatefulWidget {
  @override
  _CounterState createState() => _CounterState();
}

class _CounterState extends State<Counter> {
  int _count = 0;

  void _increment() {
    setState(() {
      _count++;
    });
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text('Count: $_count'),
        ElevatedButton(
          onPressed: _increment,
          child: Text('Increment'),
        ),
      ],
    );
  }
}
```

### 布局系统

常用布局组件：

- **Container**: 通用容器
- **Row/Column**: 水平/垂直布局
- **Stack**: 层叠布局
- **ListView**: 滚动列表
- **GridView**: 网格布局

### 导航系统

```dart
// 页面跳转
Navigator.push(
  context,
  MaterialPageRoute(builder: (context) => SecondPage()),
);

// 命名路由
MaterialApp(
  routes: {
    '/home': (context) => HomePage(),
    '/profile': (context) => ProfilePage(),
  },
);
```

## 开发流程

### 创建新项目

```bash
flutter create my_app
cd my_app
flutter run
```

### 添加依赖

在 `pubspec.yaml` 中：

```yaml
dependencies:
  flutter:
    sdk: flutter
  http: ^1.1.0
  provider: ^6.0.5

dev_dependencies:
  flutter_test:
    sdk: flutter
```

## 状态管理

### Provider 方案

```dart
class CounterModel extends ChangeNotifier {
  int _count = 0;

  int get count => _count;

  void increment() {
    _count++;
    notifyListeners();
  }
}
```

## 网络请求

```dart
import 'package:http/http.dart' as http;

Future<void> fetchData() async {
  final response = await http.get(Uri.parse('https://api.example.com/data'));
  if (response.statusCode == 200) {
    // 处理响应数据
  }
}
```

## 调试与构建

### 热重载

```bash
flutter run --hot-reload
```

### 发布构建

```bash
# Android
flutter build apk --release
flutter build appbundle --release

# iOS
flutter build ios --release

# macOS
flutter build macos --release

# Linux
flutter build linux --release
```

## 最佳实践

1. **代码组织**: 按功能模块组织文件
2. **状态管理**: 选择合适的方案并保持一致
3. **性能优化**: 使用 const 构造函数，避免不必要的重建
4. **错误处理**: 完善的异常捕获和用户提示

## 常用包推荐

- **状态管理**: provider, riverpod, bloc
- **网络请求**: http, dio
- **路由**: go_router, auto_route
- **本地存储**: shared_preferences, hive

## 学习资源

- 官方文档: flutter.dev
- Flutter Gallery 应用
- Pub.dev 包仓库
