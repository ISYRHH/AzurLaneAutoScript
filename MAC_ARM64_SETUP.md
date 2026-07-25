# Mac ARM64 运行 ALAS 完整指南

本指南基于博客 [ARM64 Mac 运行 AzurLaneAutoScript](http://www.binss.me/blog/run-azurlaneautoscript-on-arm64/) 及社区实践经验整理。

## 环境要求

- Mac with Apple Silicon (M1/M2/M3/M4 等 ARM64 芯片)
- macOS 系统
- 至少 8GB 内存（推荐 16GB 以上）

## 步骤 1: 安装 Android Studio 模拟器

### 1.1 安装 Android Studio

```bash
brew install --cask android-studio
```

### 1.2 创建虚拟设备

1. 启动 Android Studio
2. 在首页选择 **More Actions** → **Virtual Device Manager**
3. 点击 **Create Device**
4. 选择设备型号（推荐 Pixel 系列）
5. **重要**: 选择系统镜像时，选择 **Android 10** (API 29)
   - ⚠️ 不要选择太高的版本，会导致碧蓝航线黑屏
   - 选择 ARM64 架构的镜像（不是 x86）
6. 配置设备：
   - RAM: 建议 4GB
   - 内部存储: 建议 8GB 以上
   - 分辨率: 推荐 1280x720 或 1920x1080

### 1.3 启动模拟器并安装游戏

启动创建好的虚拟设备，然后安装碧蓝航线。

## 步骤 2: 准备 ALAS 代码

### 2.1 确认代码来源

**重要**: 必须通过 git clone 获取代码，不能直接下载 zip 文件！

如果你的代码是下载的 zip，请重新获取：

```bash
cd /Users/mio/github
# 如果已有目录，先备份或删除
mv AzurLaneAutoScript AzurLaneAutoScript.backup
# 重新克隆
git clone https://github.com/LmeSzinc/AzurLaneAutoScript.git
cd AzurLaneAutoScript
```

### 2.2 检查当前代码状态

```bash
cd /Users/mio/github/AzurLaneAutoScript
git status
```

## 步骤 3: 安装 Docker Desktop

### 3.1 安装

确保安装的是支持 Apple Silicon 的 Docker Desktop:

```bash
brew install --cask docker
```

或者从 [Docker 官网](https://www.docker.com/products/docker-desktop/) 下载 **Apple Chip** 版本。

### 3.2 启动 Docker Desktop

打开 Docker Desktop 应用，等待启动完成。

## 步骤 4: 拉取并运行 Docker 镜像

### 4.1 拉取镜像

```bash
docker pull binss/azurlaneautoscript:arm64
```

### 4.2 运行容器

#### 方式一：后台运行（推荐）

**使用 `-d` 参数在后台运行**，不会占用终端：

```bash
docker run -d \
  -e TZ=Asia/Shanghai \
  -v /etc/timezone:/etc/timezone:ro \
  -v /etc/localtime:/etc/localtime:ro \
  --volume=/Users/mio/github/AzurLaneAutoScript:/app/AzurLaneAutoScript:rw \
  -p 22267:22267 \
  --name azurlaneautoscript \
  --restart unless-stopped \
  binss/azurlaneautoscript:arm64
```

执行后会输出容器 ID，然后**终端会立即释放**，你可以继续执行其他命令。

**查看容器日志**:
```bash
# 查看实时日志
docker logs -f azurlaneautoscript

# 查看最后 100 行日志
docker logs --tail 100 azurlaneautoscript

# 按 Ctrl+C 退出日志查看（不会停止容器）
```

#### 方式二：前台运行（调试用）

**使用 `-it` 参数在前台运行**，可以看到实时输出：

```bash
docker run -it \
  -e TZ=Asia/Shanghai \
  -v /etc/timezone:/etc/timezone:ro \
  -v /etc/localtime:/etc/localtime:ro \
  --volume=/Users/mio/github/AzurLaneAutoScript:/app/AzurLaneAutoScript:rw \
  -p 22267:22267 \
  --name azurlaneautoscript \
  binss/azurlaneautoscript:arm64
```

**⚠️ 注意事项**:
- 终端会被占用，显示容器的实时输出
- **不要直接 Ctrl+C**，这会停止容器！
- 要退出但保持容器运行，按: **`Ctrl+P` 然后 `Ctrl+Q`** (先按 Ctrl+P，松开，再按 Ctrl+Q)
- 如果不小心 Ctrl+C 停止了，重新启动: `docker start azurlaneautoscript`

**参数说明**:
- `-d`: 后台运行（detached mode）
- `-it`: 前台运行（interactive + TTY）
- `-e TZ=Asia/Shanghai`: 设置时区
- `-v /etc/timezone:/etc/timezone:ro`: 同步时区信息（可选）
- `-v /etc/localtime:/etc/localtime:ro`: 同步本地时间（可选）
- `--volume=/Users/mio/github/AzurLaneAutoScript:/app/AzurLaneAutoScript:rw`: 挂载代码目录
  - **注意**: 请将 `/Users/mio/github/AzurLaneAutoScript` 改为你的实际路径
- `-p 22267:22267`: 映射端口，用于访问 Web 界面
- `--name azurlaneautoscript`: 容器名称
- `--restart unless-stopped`: 自动重启策略（仅后台模式推荐）

### 4.3 容器管理常用命令

```bash
# 启动已停止的容器
docker start azurlaneautoscript

# 停止运行中的容器
docker stop azurlaneautoscript

# 重启容器
docker restart azurlaneautoscript

# 查看容器状态
docker ps -a | grep azurlaneautoscript

# 查看容器日志
docker logs -f azurlaneautoscript

# 进入容器终端
docker exec -it azurlaneautoscript /bin/bash

# 完全删除容器（保留数据，因为代码在宿主机上）
docker stop azurlaneautoscript
docker rm azurlaneautoscript
# 然后重新运行 docker run 命令
```

## 步骤 5: 安装缺失的 Python 依赖

容器启动后，需要进入容器安装一些缺失的依赖。

### 5.1 进入容器终端

在 Docker Desktop 中:
1. 找到 **azurlaneautoscript** 容器
2. 点击容器，选择 **Terminal** 标签

或使用命令行:

```bash
docker exec -it azurlaneautoscript /bin/bash
```

### 5.2 安装依赖

在容器的终端中执行:

```bash
cd /app/pyroot/bin
./pip install onepush==1.3.0
./pip install pydantic==1.10.2
./pip install uiautomator2cache==0.3.0.1
```

**注意**: 版本号应该与 `requirements.txt` 中的一致。可以根据最新的 requirements.txt 调整。

### 5.3 重启容器

安装完依赖后，重启容器:

```bash
exit  # 退出容器终端
docker restart azurlaneautoscript
```

## 步骤 6: 配置 ALAS 连接模拟器

### 6.1 获取模拟器 ADB 端口

在 Mac 终端执行:

```bash
lsof -iTCP -sTCP:LISTEN -n -P | grep qemu
```

你会看到类似输出:

```
qemu-syst 43049 binss   42u  IPv4 0xbccf4cd072e9da6f      0t0  TCP 127.0.0.1:5555 (LISTEN)
```

记住端口号 `5555`（通常是这个）。

### 6.2 访问 ALAS Web 界面

在浏览器中打开:

```
http://localhost:22267
```

### 6.3 配置模拟器设置

在 ALAS Web 界面中，进入 **设置** → **模拟器** 页面，配置如下:

1. **模拟器 Serial**: `host.docker.internal:5555`
   - 这是 Docker 容器访问宿主机的特殊地址
   - 如果端口不是 5555，请修改为实际端口

2. **截图方案**: 选择 **DroidCast_raw** 或 **ADB_nc**
   - 推荐: `ADB_nc`（较稳定）
   - 备选: `DroidCast_raw`

3. **控制方案**: 选择 **Hermit** 或 **MaaTouch**
   - 推荐: `Hermit`
   - 如果 Hermit 有问题，改用 `MaaTouch`

4. 保存设置

### 6.4 启用 Hermit 权限（如果使用 Hermit）

1. ALAS 会自动在模拟器中安装 Hermit
2. 模拟器会弹出权限请求窗口
3. 按照提示开启 **无障碍服务** (Accessibility Service)
4. 如果遇到权限问题，可能需要手动推送 atx-agent:

```bash
# 下载 arm64 版本的 atx-agent
curl -L -o atx-agent.tar.gz https://github.com/openatx/atx-agent/releases/download/0.10.0/atx-agent_0.10.0_linux_arm64.tar.gz
tar -xzf atx-agent.tar.gz

# 推送到模拟器
adb push atx-agent /data/local/tmp/
adb shell chmod 755 /data/local/tmp/atx-agent
adb shell /data/local/tmp/atx-agent server -d
```

## 步骤 7: 测试运行

### 7.1 测试连接

在 ALAS Web 界面中:
1. 点击 **启动** 按钮
2. 观察日志输出
3. 确认能够正常连接模拟器并截图

### 7.2 常见问题

#### 问题 1: 无法连接模拟器

**解决方案**:
- 检查模拟器是否正常运行
- 检查 Serial 配置是否正确
- 尝试在容器内测试连接: `docker exec -it azurlaneautoscript adb connect host.docker.internal:5555`

#### 问题 2: Hermit 反复请求权限

**解决方案**:
- 改用 MaaTouch 控制方案
- 确保模拟器系统版本是 Android 10
- 手动推送 atx-agent（见步骤 6.4）

#### 问题 3: OCR 识别缓慢

**解决方案**:
- 确保使用的是 ARM64 原生镜像，不是 x86 模拟
- 检查 Docker 资源分配（在 Docker Desktop 设置中）

#### 问题 4: 容器启动后 git 报错

```
sh: 1: /app/AzurLaneAutoScript/toolkit/Git/mingw64/bin/git.exe: not found
```

**这是正常的！** 这是 Windows 版的 git 路径，在 macOS/Linux 上不适用。如果需要更新代码:

```bash
cd /Users/mio/github/AzurLaneAutoScript
git pull
docker restart azurlaneautoscript
```

#### 问题 5: ModuleNotFoundError

如果出现缺少模块的错误，进入容器安装:

```bash
docker exec -it azurlaneautoscript /bin/bash
cd /app/pyroot/bin
./pip install <缺失的模块名>
```

## 步骤 8: 更新 ALAS

由于代码目录是挂载到容器的，更新非常简单:

```bash
# 在宿主机上更新代码
cd /Users/mio/github/AzurLaneAutoScript
git pull

# 检查 requirements.txt 是否有变化
git diff HEAD@{1} requirements.txt

# 如果有依赖变化，进入容器安装新依赖
docker exec -it azurlaneautoscript /bin/bash
cd /app/pyroot/bin
./pip install -r /app/AzurLaneAutoScript/requirements.txt

# 重启容器
docker restart azurlaneautoscript
```

## 性能优化建议

1. **Docker 资源分配**:
   - 打开 Docker Desktop → Settings → Resources
   - 建议分配: CPU 4核，内存 8GB

2. **模拟器配置**:
   - 使用 Android 10 系统
   - 分配足够的 RAM (4GB)

3. **截图和控制方案**:
   - 截图推荐 `ADB_nc` 或 `DroidCast_raw`
   - 控制推荐 `Hermit` 或 `MaaTouch`

## 替代方案: 本地运行（不使用 Docker）

如果你更熟悉 Python 环境，也可以参考社区方案:
- [swordfeng 的启动器](https://github.com/swordfeng/alas-launcher) - 针对 Mac M4 优化

## 参考资源

- [原博客文章](http://www.binss.me/blog/run-azurlaneautoscript-on-arm64/)
- [ALAS 官方仓库](https://github.com/LmeSzinc/AzurLaneAutoScript)
- [ALAS Wiki - Docker 安装](https://github.com/LmeSzinc/AzurLaneAutoScript/wiki/Installation_en_docker)
- [B站教程视频](https://b23.tv/IuFp488)

## 常用命令速查

```bash
# 查看容器日志
docker logs azurlaneautoscript

# 进入容器
docker exec -it azurlaneautoscript /bin/bash

# 重启容器
docker restart azurlaneautoscript

# 停止容器
docker stop azurlaneautoscript

# 查看模拟器端口
lsof -iTCP -sTCP:LISTEN -n -P | grep qemu

# 连接 ADB
adb connect 127.0.0.1:5555

# 查看连接的设备
adb devices
```

---

**最后更新**: 2026-01-21  
**适用于**: Mac with Apple Silicon (ARM64)  
**ALAS 版本**: 当前主分支
