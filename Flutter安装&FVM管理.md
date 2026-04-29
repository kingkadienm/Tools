# Flutter + FVM 安装配置指南

## 核心思路

| ❌ 不推荐 | ✅ 推荐 |
|----------|--------|
| 官网下载 zip 解压 | Flutter SDK 由 FVM 管理 |
| 多版本手动切换 | FVM 由 `dart pub global` 安装 |
| 不用 FVM | 缓存统一放非系统盘 |

**PATH 只需指向：** `pub-cache/bin` + `fvm/default/bin`

---

## 目录规划

```
D:\flutter\                    # Windows
~/dev/flutter/                 # macOS
  ├─ flutter-sdk/              # 原始 SDK（仅用于安装 fvm）
  ├─ PUB_CACHE/                # dart pub global 缓存
  │   └─ bin/fvm
  └─ fvm/                      # FVM 管理的 Flutter 版本
      ├─ versions/
      └─ default/              # 全局默认版本
```

---

## Step 1: Clone Flutter SDK

```bash
# Windows: D:\flutter\
# macOS:   ~/dev/flutter/

git clone -b stable https://github.com/flutter/flutter.git flutter-sdk

# 验证
cd flutter-sdk
./bin/flutter --version
```

---

## Step 2: 配置环境变量

### Windows（系统变量）

| 变量名 | 值 |
|-------|-----|
| `PUB_HOSTED_URL` | `https://pub.flutter-io.cn` |
| `FLUTTER_STORAGE_BASE_URL` | `https://storage.flutter-io.cn` |
| `PUB_CACHE` | `D:\flutter\PUB_CACHE` |
| `FVM_CACHE_PATH` | `D:\flutter\fvm` |

**Path 添加（按顺序）：**
```
D:\flutter\flutter-sdk\bin
%PUB_CACHE%\bin
D:\flutter\fvm\default\bin
```

### macOS（~/.zshrc）

```bash
# 镜像
export PUB_HOSTED_URL=https://pub.flutter-io.cn
export FLUTTER_STORAGE_BASE_URL=https://storage.flutter-io.cn

# 缓存路径
export PUB_CACHE="$HOME/dev/flutter/pub-cache"
export FVM_CACHE_PATH="$HOME/dev/flutter/fvm"

# PATH
export PATH="$HOME/dev/flutter/flutter-sdk/bin:$PATH"
export PATH="$PUB_CACHE/bin:$PATH"
export PATH="$FVM_CACHE_PATH/default/bin:$PATH"
```

```bash
source ~/.zshrc
```

---

## Step 3: 安装 FVM

```bash
dart pub global activate fvm
fvm --version
```

---

## Step 4: 使用 FVM 管理 Flutter

```bash
# 安装版本
fvm install stable
fvm install 3.22.3

# 设置全局默认
fvm global stable

# 验证（应指向 fvm/default/bin/flutter）
where flutter   # Windows
which flutter   # macOS
```

---

## Step 5: 项目级版本管理

```bash
cd your-project
fvm use 3.22.3

# 之后使用
fvm flutter run
fvm flutter pub get
```

---

## FVM 命令速查

| 操作 | 命令 |
|-----|------|
| 安装版本 | `fvm install stable` / `fvm install 3.22.3` |
| 移除版本 | `fvm remove 3.16.9` |
| 项目级使用 | `fvm use 3.22.3` |
| 全局设置 | `fvm global stable` |
| 查看已安装 | `fvm list` |
| 查看可用版本 | `fvm releases` |
| 诊断 | `fvm doctor` |
| 运行 Flutter | `fvm flutter run` / `fvm flutter pub get` |

---

## 最终验证

```bash
where dart      # → flutter-sdk/bin
where fvm       # → PUB_CACHE/bin
where flutter   # → fvm/default/bin
fvm doctor
flutter doctor
```
