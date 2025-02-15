# 启动 Redis 并测试持久化

Redis 是一个高性能的键值存储系统，支持多种持久化方式（如 RDB 和 AOF）。以下是启动 Redis 并测试持久化的详细步骤。

---

## 一、启动 Redis

### 1. **通过命令行启动 Redis**
在终端中运行以下命令启动 Redis 服务器：
```bash
redis-server
```
默认情况下，Redis 会使用内置的配置文件启动。如果需要指定配置文件，可以运行：
```bash
redis-server /path/to/redis.conf
```

### 2. **通过 `systemd` 启动 Redis（如果已配置）**
如果 Redis 已配置为系统服务，可以使用以下命令启动：
```bash
sudo systemctl start redis
```
检查 Redis 服务状态：
```bash
sudo systemctl status redis
```

### 3. **验证 Redis 是否运行**
使用 `redis-cli` 连接到 Redis 服务器并测试：
```bash
redis-cli ping
```
如果返回 `PONG`，说明 Redis 已成功启动。

---

## 二、测试 Redis 持久化

Redis 支持两种持久化方式：
1. **RDB（快照）**：定期将内存中的数据保存到磁盘。
2. **AOF（追加文件）**：记录所有写操作日志，并在重启时重放。

### 1. **检查持久化配置**
打开 Redis 配置文件（通常位于 `/etc/redis/redis.conf`），检查以下配置项：

#### RDB 配置
```ini
save 900 1      # 在 900 秒内至少有 1 个键被修改时触发保存
save 300 10     # 在 300 秒内至少有 10 个键被修改时触发保存
save 60 10000   # 在 60 秒内至少有 10000 个键被修改时触发保存
dbfilename dump.rdb  # RDB 文件名
dir /var/lib/redis   # RDB 文件保存路径
```

#### AOF 配置
```ini
appendonly yes        # 启用 AOF
appendfilename "appendonly.aof"  # AOF 文件名
appendfsync everysec  # 每秒同步一次
```

### 2. **测试 RDB 持久化**
1. 启动 Redis 并插入一些数据：
   ```bash
   redis-cli
   ```
   在 Redis CLI 中运行：
   ```bash
   SET key1 value1
   SET key2 value2
   ```

2. 手动触发 RDB 快照：
   ```bash
   SAVE
   ```
   或者：
   ```bash
   BGSAVE
   ```
   - `SAVE` 会阻塞 Redis 直到快照完成。
   - `BGSAVE` 会在后台异步执行快照。

3. 检查 RDB 文件：
   RDB 文件默认保存在 `/var/lib/redis/dump.rdb`。检查文件是否存在：
   ```bash
   ls -l /var/lib/redis/dump.rdb
   ```

4. 重启 Redis 并验证数据：
   ```bash
   sudo systemctl restart redis
   redis-cli
   ```
   在 Redis CLI 中运行：
   ```bash
   GET key1
   ```
   如果返回 `value1`，说明 RDB 持久化成功。

---

### 3. **测试 AOF 持久化**
1. 启用 AOF（如果未启用）：
   编辑 Redis 配置文件 `/etc/redis/redis.conf`，确保以下配置：
   ```ini
   appendonly yes
   appendfilename "appendonly.aof"
   appendfsync everysec
   ```
   重启 Redis：
   ```bash
   sudo systemctl restart redis
   ```

2. 插入一些数据：
   ```bash
   redis-cli
   ```
   在 Redis CLI 中运行：
   ```bash
   SET key3 value3
   SET key4 value4
   ```

3. 检查 AOF 文件：
   AOF 文件默认保存在 `/var/lib/redis/appendonly.aof`。检查文件内容：
   ```bash
   cat /var/lib/redis/appendonly.aof
   ```
   你应该能看到类似以下的日志：
   ```
   *3
   $3
   SET
   $4
   key3
   $6
   value3
   ```

4. 重启 Redis 并验证数据：
   ```bash
   sudo systemctl restart redis
   redis-cli
   ```
   在 Redis CLI 中运行：
   ```bash
   GET key3
   ```
   如果返回 `value3`，说明 AOF 持久化成功。

---

## 三、持久化问题排查

### 1. **RDB 文件未生成**
- 检查 `save` 配置是否正确。
- 检查 Redis 日志文件（通常位于 `/var/log/redis/redis-server.log`），查看是否有错误信息。
- 确保 Redis 有权限写入 `dir` 配置的目录。

### 2. **AOF 文件未生成**
- 检查 `appendonly` 是否设置为 `yes`。
- 检查 Redis 日志文件，查看是否有错误信息。
- 确保 Redis 有权限写入 `dir` 配置的目录。

### 3. **数据丢失**
- 如果 RDB 和 AOF 都启用，Redis 会优先使用 AOF 文件恢复数据。
- 检查 AOF 文件是否损坏：
  ```bash
  redis-check-aof --fix /var/lib/redis/appendonly.aof
  ```
- 检查 RDB 文件是否损坏：
  ```bash
  redis-check-rdb /var/lib/redis/dump.rdb
  ```

---

## 四、总结
通过以上步骤，你可以成功启动 Redis 并测试其持久化功能。RDB 和 AOF 各有优缺点：
- **RDB**：适合备份和快速恢复，但可能会丢失最后一次快照后的数据。
- **AOF**：数据安全性更高，但文件体积较大，恢复速度较慢。

根据实际需求选择合适的持久化方式，或同时启用 RDB 和 AOF 以提高数据可靠性。如果遇到问题，可以检查 Redis 日志或使用 `redis-check-aof` 和 `redis-check-rdb` 工具修复文件。
