# CentOS 7 静态二进制构建说明

## 概述

本项目提供了自动化的 GitHub Actions workflow，用于构建可在 CentOS 7 离线环境使用的 kcat 静态链接二进制文件。

## 如何获取二进制文件

### 方式 1: 从 GitHub Actions 下载（推荐）

1. 访问项目的 [Actions 页面](../../actions/workflows/build-centos7-binary.yml)
2. 点击最新的成功构建（绿色✓标记）
3. 在页面底部的 "Artifacts" 部分找到 `kcat-centos7-static-binary`
4. 点击下载压缩包

### 方式 2: 从 Release 下载（仅限标签版本）

如果推送了版本标签（如 `v1.7.1`），二进制文件会自动发布到 Release 页面：

1. 访问项目的 [Releases 页面](../../releases)
2. 找到对应版本
3. 下载 `kcat-centos7-static-*.tar.gz` 文件

### 方式 3: 手动触发构建

1. 访问 [Actions 页面](../../actions/workflows/build-centos7-binary.yml)
2. 点击 "Run workflow" 按钮
3. 选择分支后点击 "Run workflow"
4. 等待构建完成后下载 Artifacts

## 安装和使用

### 在 CentOS 7 系统上安装

```bash
# 1. 解压下载的文件
tar -xzf kcat-centos7-static-*.tar.gz
cd release

# 2. 查看版本信息
cat VERSION.txt

# 3. 方式 A: 直接使用（无需安装）
chmod +x kcat
./kcat -V

# 4. 方式 B: 安装到系统（需要 root 权限）
sudo ./install.sh

# 5. 验证安装
kcat -V
```

### 卸载

```bash
cd release
sudo ./uninstall.sh
```

## 离线环境使用

此二进制文件是**完全静态链接**的，包含以下特性：

✅ **无需安装任何依赖库**
- 不需要 librdkafka
- 不需要 libyajl
- 不需要 openssl
- 不需要 libsasl2

✅ **支持的系统**
- CentOS 7 (x86_64)
- RHEL 7 (x86_64)
- Oracle Linux 7 (x86_64)
- 其他兼容的 EL7 系统

✅ **包含完整功能**
- JSON 格式支持
- Avro 序列化支持
- 事务支持
- 消费者组支持
- Mock 集群支持

## 使用示例

### 基本命令

```bash
# 查看帮助
kcat -h

# 查看版本
kcat -V

# 列出集群元数据
kcat -b kafka-broker:9092 -L

# 列出特定主题的元数据
kcat -b kafka-broker:9092 -t my-topic -L
```

### 消费消息

```bash
# 从头开始消费
kcat -b kafka-broker:9092 -t my-topic -C -o beginning

# 消费最新消息
kcat -b kafka-broker:9092 -t my-topic -C -o end

# 消费特定分区
kcat -b kafka-broker:9092 -t my-topic -p 0 -C

# 使用消费者组
kcat -b kafka-broker:9092 -G my-group my-topic

# JSON 格式输出
kcat -b kafka-broker:9092 -t my-topic -C -J

# 消费指定数量的消息后退出
kcat -b kafka-broker:9092 -t my-topic -C -c 10
```

### 生产消息

```bash
# 从标准输入生产消息
echo "hello world" | kcat -b kafka-broker:9092 -t my-topic -P

# 从文件生产消息（每行一条消息）
cat messages.txt | kcat -b kafka-broker:9092 -t my-topic -P

# 带 key 的消息（使用 : 作为分隔符）
echo "key1:value1" | kcat -b kafka-broker:9092 -t my-topic -K: -P

# 生产到特定分区
echo "message" | kcat -b kafka-broker:9092 -t my-topic -p 0 -P

# 使用事务
cat data.txt | kcat -b kafka-broker:9092 -t my-topic -P \
  -X transactional.id=my-producer
```

### 高级用法

```bash
# 使用 SASL 认证
kcat -b kafka-broker:9092 -t my-topic -C \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanism=PLAIN \
  -X sasl.username=user \
  -X sasl.password=pass

# 使用配置文件
kcat -b kafka-broker:9092 -t my-topic -C -F /path/to/config.conf

# 查询特定时间戳的 offset
kcat -b kafka-broker:9092 -Q -t my-topic:0:1609459200000

# 自定义输出格式
kcat -b kafka-broker:9092 -t my-topic -C \
  -f 'Topic: %t, Partition: %p, Offset: %o, Key: %k, Value: %s\n'
```

## 配置文件示例

创建 `~/.config/kcat.conf`:

```properties
# Broker 配置
metadata.broker.list=kafka-broker1:9092,kafka-broker2:9092

# 安全配置
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.username=myuser
sasl.password=mypassword

# SSL 配置
ssl.ca.location=/path/to/ca-cert
ssl.certificate.location=/path/to/client-cert
ssl.key.location=/path/to/client-key

# 性能调优
socket.keepalive.enable=true
socket.timeout.ms=60000
```

使用配置文件：
```bash
kcat -t my-topic -C  # 自动读取 ~/.config/kcat.conf
```

## 故障排查

### 问题：权限被拒绝

```bash
# 解决方案：添加执行权限
chmod +x kcat
```

### 问题：找不到命令

```bash
# 解决方案 1：使用完整路径
/path/to/kcat -V

# 解决方案 2：安装到系统
sudo ./install.sh
```

### 问题：连接 Kafka 失败

```bash
# 检查网络连接
telnet kafka-broker 9092

# 使用详细日志
kcat -b kafka-broker:9092 -L -d broker,topic

# 增加超时时间
kcat -b kafka-broker:9092 -L -m 30
```

### 问题：GLIBC 版本不兼容

如果遇到 GLIBC 版本问题，说明系统版本过低。此二进制文件需要：
- GLIBC 2.17 或更高（CentOS 7 默认版本）
- Linux kernel 3.10 或更高

检查系统版本：
```bash
ldd --version
uname -r
```

## 技术细节

### 构建过程

1. 使用 CentOS 7 Docker 容器作为构建环境
2. 安装必要的构建工具和依赖
3. 运行 `bootstrap.sh` 脚本：
   - 下载 librdkafka 源码
   - 下载 yajl 源码
   - 静态编译所有依赖
   - 链接生成单一可执行文件
4. 使用 `strip` 移除调试符号以减小体积
5. 打包为 tar.gz 格式

### 二进制特性

- **架构**: x86_64
- **链接方式**: 静态链接
- **大小**: 约 2-3 MB（strip 后）
- **依赖**: 仅依赖 Linux 系统调用

### 验证二进制

```bash
# 查看文件类型
file kcat

# 检查动态链接库（应该只有系统库或无输出）
ldd kcat

# 查看符号表
nm kcat | head

# 检查二进制大小
ls -lh kcat
```

## 自动化构建触发条件

workflow 会在以下情况自动运行：

1. **推送到 master 分支** - 每次代码更新都会构建
2. **Pull Request** - PR 时会构建以验证
3. **版本标签** - 推送 `v*` 标签时构建并发布到 Release
4. **手动触发** - 可随时手动运行

## 相关链接

- [GitHub Actions Workflow](.github/workflows/build-centos7-binary.yml)
- [kcat 官方文档](https://github.com/edenhill/kcat)
- [librdkafka 文档](https://github.com/edenhill/librdkafka)

## 支持

如有问题，请：
1. 查看 [Issues](../../issues)
2. 提交新的 Issue
3. 查看 kcat 官方文档
