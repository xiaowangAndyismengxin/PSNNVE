# Redis 权限问题解决总结

## 一、问题背景
在使用 Redis 过程中，可能会遇到因权限不足导致的各种问题，如持久化失败、无法启动等。以下是解决 Redis 权限问题的详细步骤和要点总结。

## 二、权限检查要点

### （一）配置文件目录权限
- **检查文件**：通常 Redis 配置文件位于 `/etc/redis/redis.conf`。
- **检查命令**：使用 `ls -l /etc/redis/redis.conf` 查看权限。
- **权限要求**：确保文件所有者或所属组为 Redis 运行用户（通常是 `redis`），且有读取权限。例如 `-rw-r----- 1 redis redis 85844 Mar  4  2022 /etc/redis/redis.conf` 表示 `redis` 用户有读写权，所属组有读取权。

### （二）数据存储目录权限
- **检查目录**：Redis 数据存储目录一般是 `/var/lib/redis`。
- **检查命令**：使用 `ls -ld /var/lib/redis` 查看权限。
- **权限要求**：目录所有者或所属组为 Redis 运行用户，且有读写和执行权限。如 `drwxr-xr-x 2 redis redis 4096 Feb  9 18:30 /var/lib/redis` 满足要求。这串字符详细解释如下：
    - **文件权限部分**：`drwxr-xr-x` 。其中，`d` 表示这是一个目录（directory）。后面九个字符每三个为一组，分别对应文件所有者（user）、所属组（group）和其他用户（others）的权限。`rwx` 对于文件所有者，代表具有读（`r`，read）、写（`w`，write）和执行（`x`，execute）权限，意味着所有者可以读取目录中的内容、在目录中创建或删除文件和子目录，以及进入该目录；`r - x` 对于所属组用户，具有读和执行权限，但没有写权限，即可以读取目录内容和进入目录，但不能修改目录中的文件或子目录；`r - x` 对于其他用户，同样具有读和执行权限，没有写权限，与所属组用户权限相同。
    - **硬链接数**：`2` 表示该目录的硬链接数。对于目录来说，硬链接数通常等于其下一级子目录的数量加上 2。其中，`.` 表示当前目录本身，`..` 表示上级目录，这两个目录项会使得目录的硬链接数至少为 2。如果该目录下还有其他子目录，每增加一个子目录，硬链接数就会加 1。
    - **所有者和所属组**：`redis redis` ，第一个 `redis` 表示该目录的所有者是 `redis` 用户，第二个 `redis` 表示该目录所属的组是 `redis` 组。这决定了哪些用户和组对该目录具有相应的权限。
    - **文件大小或目录占用空间**：`4096` 表示该目录所占用的磁盘空间大小为 4096 字节。在 Linux 系统中，目录本身也会占用一定的磁盘空间来存储其内部的元数据，如目录项列表等。
    - **最后修改时间**：`Feb  9 18:30` 显示该目录最后一次被修改的时间是 2 月 9 日 18 点 30 分。
    - **目录路径**：`/var/lib/redis` 这是该目录在文件系统中的完整路径，表示它位于 `/var/lib/` 目录下，名为 `redis`。通常，`/var/lib/` 目录用于存储各种应用程序的运行时数据和状态信息，`redis` 目录很可能是 Redis 数据库存储数据、配置文件等相关内容的位置。

### （三）日志文件目录权限
1. **查找日志路径**：在 Redis 配置文件中找到 `logfile` 配置项，确定日志文件路径，如 `logfile /var/log/redis/redis-server.log`。
2. **检查目录权限**：使用 `ls -ld /var/log/redis` 查看，确保目录所有者或所属组为 Redis 运行用户，且有写入权限。

### （四）网络端口权限
- **检查内容**：Redis 默认监听 6379 端口，需确保 Redis 进程有绑定该端口的权限。
- **检查命令**：可使用 `sudo netstat -tuln | grep 6379` 或 `sudo ss -tuln | grep 6379` 检查端口监听情况。若有 Redis 进程监听该端口，则权限正常。

### （五）进程运行状态
- **查看服务状态**：使用 `sudo systemctl status redis-server` 查看 Redis 服务状态。
- **查看日志信息**：使用 `sudo journalctl -u redis-server` 查看 Redis 日志，若日志无权限相关错误信息，则说明运行中未遇权限问题。

## 三、权限修改操作

### （一）修改文件或目录所有者和所属组
使用 `chown` 命令，例如将 `/var/lib/redis` 目录及其子内容的所有者和所属组修改为 `redis`：
```bash
sudo chown -R redis:redis /var/lib/redis
```
`-R` 选项表示递归操作，会对目录下所有文件和子目录生效。

### （二）修改文件或目录权限
使用 `chmod` 命令，例如将 `/var/lib/redis` 目录及其子内容的权限设置为 `755`：
```bash
sudo chmod -R 755 /var/lib/redis
```
权限 `755` 表示所有者有读写执行权限（`7 = 4 + 2 + 1`），组用户和其他用户有读和执行权限（`5 = 4 + 1`）。

## 四、其他相关问题及解决

### （一）内存过度提交未启用问题
- **问题表现**：启动日志提示 `WARNING Memory overcommit must be enabled!`。
- **临时解决方法**：执行 `sudo sysctl vm.overcommit_memory=1`，立即生效，但重启后失效。
- **永久解决方法**：编辑 `/etc/sysctl.conf` 文件，添加或修改 `vm.overcommit_memory = 1`，保存后执行 `sudo sysctl -p` 使配置生效。

### （二）端口被占用问题
- **问题表现**：启动日志显示 `Warning: Could not create server TCP listening socket *:6379: bind: Address already in use`。
- **查找占用进程**：使用 `sudo lsof -i :6379` 查找占用端口 6379 的进程信息及进程 ID（PID）。
- **终止占用进程**：使用 `sudo kill -9 <PID>` 强制终止进程（`-9` 代表 `SIGKILL` 信号，强制终止且不允许进程做善后处理）。
- **修改 Redis 监听端口（可选）**：编辑 Redis 配置文件 `/etc/redis/redis.conf`，将 `port` 配置项修改为其他未被占用的端口，如 `port 6380`，保存后启动 Redis 时指定配置文件 `redis-server /etc/redis/redis.conf`。 
