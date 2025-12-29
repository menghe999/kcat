# CentOS 7 静态二进制包说明

## 🎯 新功能

本项目现在提供**自动构建的 CentOS 7 静态二进制包**，可直接在 CentOS 7 离线环境使用，无需安装任何依赖！

## 📦 获取方式

### 方式 1: GitHub Actions（推荐）

1. 访问 [Actions 页面](../../actions/workflows/build-centos7-binary.yml)
2. 点击最新的成功构建
3. 下载 `kcat-centos7-static-binary` artifact

### 方式 2: Release 页面

当推送版本标签时，二进制包会自动发布到 [Releases](../../releases)

### 方式 3: 手动触发

在 Actions 页面点击 "Run workflow" 按钮手动触发构建

## ⚡ 快速开始

```bash
# 1. 下载并解压
tar -xzf kcat-centos7-static-*.tar.gz
cd release

# 2. 直接使用
chmod +x kcat
./kcat -V

# 3. 或安装到系统
sudo ./install.sh
```

## ✨ 特性

| 特性 | 说明 |
|------|------|
| 🔒 **静态链接** | 无需安装 librdkafka、openssl 等依赖 |
| 📦 **单文件** | 仅一个可执行文件，约 2-3 MB |
| 🌐 **离线可用** | 完全支持离线环境部署 |
| 🎯 **CentOS 7** | 专为 CentOS 7 / RHEL 7 优化 |
| ⚙️ **完整功能** | 包含 JSON、Avro、事务、消费者组等所有特性 |
| 🔄 **自动构建** | 每次代码更新自动构建最新版本 |

## 📚 文档

- [完整安装和使用指南](BUILD_CENTOS7_BINARY.md)
- [快速开始](QUICK_START_CENTOS7.md)
- [原项目 README](../README.md)

## 🔧 技术细节

### 构建环境
- **容器**: CentOS 7 Docker
- **编译器**: GCC (CentOS 7 默认版本)
- **链接方式**: 完全静态链接

### 包含的依赖（已静态编译）
- librdkafka (最新稳定版)
- yajl (JSON 支持)
- zlib (压缩支持)
- openssl (SSL/TLS 支持)
- cyrus-sasl (SASL 认证支持)

### 系统要求
- CentOS 7 / RHEL 7 / Oracle Linux 7 (x86_64)
- Linux kernel 3.10+
- GLIBC 2.17+

## 📋 使用示例

### 基础操作

```bash
# 查看集群元数据
kcat -b kafka-broker:9092 -L

# 消费消息
kcat -b kafka-broker:9092 -t my-topic -C

# 生产消息
echo "hello world" | kcat -b kafka-broker:9092 -t my-topic -P
```

### 高级功能

```bash
# 使用 SASL 认证
kcat -b broker:9092 -t topic -C \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanism=PLAIN \
  -X sasl.username=user \
  -X sasl.password=pass

# JSON 格式输出
kcat -b broker:9092 -t topic -C -J

# 消费者组
kcat -b broker:9092 -G my-group topic1 topic2

# 事务生产
cat data.txt | kcat -b broker:9092 -t topic -P \
  -X transactional.id=my-producer
```

## 🔍 验证二进制

```bash
# 查看文件信息
file kcat
# 输出: kcat: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked

# 检查依赖（应该只有系统库或无输出）
ldd kcat

# 查看版本
./kcat -V
```

## 🐛 故障排查

### 权限问题
```bash
chmod +x kcat
```

### 连接问题
```bash
# 测试网络
telnet kafka-broker 9092

# 启用调试日志
kcat -b broker:9092 -L -d broker,topic
```

### 配置问题
```bash
# 使用配置文件
kcat -F /path/to/config.conf -t topic -C

# 列出所有配置选项
kcat -X list
```

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

## 📄 许可证

与原项目相同，采用 BSD 2-Clause License

## 🔗 相关链接

- [GitHub Actions Workflow](../.github/workflows/build-centos7-binary.yml)
- [kcat 官方仓库](https://github.com/edenhill/kcat)
- [librdkafka 文档](https://github.com/edenhill/librdkafka)

---

**注意**: 此二进制包由 GitHub Actions 自动构建，确保与最新代码同步。建议定期更新以获取最新功能和安全修复。
