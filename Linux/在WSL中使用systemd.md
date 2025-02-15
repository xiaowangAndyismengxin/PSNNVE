# 如何在 WSL 中启用 `systemd`

`systemd` 是 Linux 系统中用于管理服务和进程的初始化系统。在 WSL（Windows Subsystem for Linux）中，默认情况下可能未启用 `systemd`。以下是如何在 WSL 2 中启用 `systemd` 的详细步骤。

---

## 前提条件
1. **WSL 版本**：确保 WSL 版本为 **0.67.6 或更高版本**。
   - 检查 WSL 版本：
     ```bash
     wsl --version
     ```
   - 如果版本过低，更新 WSL：
     ```bash
     wsl --update
     ```
     或者从 Microsoft Store 下载最新版本。

2. **WSL 2**：确保你的 Linux 发行版运行在 WSL 2 上。
   - 检查 WSL 版本：
     ```bash
     wsl -l -v
     ```
   - 如果未使用 WSL 2，可以转换：
     ```bash
     wsl --set-version <发行版名称> 2
     ```

---

## 启用 `systemd` 的步骤

### 1. 打开 Linux 发行版的命令行
启动你的 Linux 发行版（如 Ubuntu），并进入根目录：
```bash
cd /
```

### 2. 编辑 `wsl.conf` 文件
1. 使用 `nano` 编辑器打开 `/etc/wsl.conf` 文件：
   ```bash
   sudo nano /etc/wsl.conf
   ```

2. 在文件中添加以下内容：
   ```ini
   [boot]
   systemd=true
   ```

3. 保存并退出：
   - 按 `Ctrl + X` 退出。
   - 按 `Y` 确认保存更改。
   - 按 `Enter` 确认文件名。

---

### 3. 重启 WSL
1. 关闭所有 WSL 实例：
   - 打开 PowerShell 或命令提示符，运行以下命令：
     ```bash
     wsl --shutdown
     ```

2. 重新启动你的 Linux 发行版。

---

### 4. 验证 `systemd` 是否启用
1. 在 Linux 发行版中运行以下命令，检查 `systemd` 是否已启用：
   ```bash
   systemctl list-unit-files --type=service
   ```
   如果显示服务列表，说明 `systemd` 已成功启用。

2. 检查 `systemd` 是否正在运行：
   ```bash
   ps aux | grep systemd
   ```
   如果看到 `systemd` 进程，说明已成功启用。

---

## 其他注意事项
1. **`wsl.conf` 和 `.wslconfig` 的区别**：
   - **`wsl.conf`**：用于配置特定 Linux 发行版的设置（如启用 `systemd`）。
   - **`.wslconfig`**：用于全局配置所有 WSL 实例的设置（如内存限制、CPU 核心数等）。

2. **自动挂载设置**：
   - 如果需要调整自动挂载设置（如挂载 Windows 驱动器），可以在 `wsl.conf` 中添加以下内容：
     ```ini
     [automount]
     enabled = true
     options = "metadata,umask=22,fmask=11"
     ```

3. **已知问题**：
   - 启用 `systemd` 后，某些服务可能无法正常工作。如果遇到问题，可以尝试禁用 `systemd`（将 `systemd=true` 改为 `systemd=false`）并重启 WSL。

---

## 总结
通过以上步骤，你可以在 WSL 2 中成功启用 `systemd`，从而使用 `systemctl` 命令管理服务和进程。如果遇到问题，请检查 WSL 版本和配置文件是否正确，并参考官方文档获取更多帮助。
