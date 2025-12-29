# kcat CentOS 7 快速开始指南

## 📥 下载

访问 [GitHub Actions](../../actions/workflows/build-centos7-binary.yml) → 选择最新构建 → 下载 Artifacts

## 🚀 快速安装

```bash
# 解压
tar -xzf kcat-centos7-static-*.tar.gz
cd release

# 安装（需要 root）
sudo ./install.sh

# 验证
kcat -V
```

## 💡 常用命令速查

### 查看集群信息
```bash
kcat -b broker:9092 -L
```

### 消费消息
```bash
# 从头消费
kcat -b broker:9092 -t topic -C -o beginning

# 消费最新
kcat -b broker:9092 -t topic -C -o end

# 消费 10 条后退出
kcat -b broker:9092 -t topic -C -c 10

# JSON 格式
kcat -b broker:9092 -t topic -C -J
```

### 生产消息
```bash
# 单条消息
echo "hello" | kcat -b broker:9092 -t topic -P

# 批量消息
cat file.txt | kcat -b broker:9092 -t topic -P

# 带 key
echo "key:value" | kcat -b broker:9092 -t topic -K: -P
```

### 使用认证
```bash
kcat -b broker:9092 -t topic -C \
  -X security.protocol=SASL_SSL \
  -X sasl.mechanism=PLAIN \
  -X sasl.username=user \
  -X sasl.password=pass
```

## 📝 配置文件

创建 `~/.config/kcat.conf`:
```properties
metadata.broker.list=broker1:9092,broker2:9092
security.protocol=SASL_SSL
sasl.mechanism=PLAIN
sasl.username=myuser
sasl.password=mypassword
```

然后直接使用：
```bash
kcat -t topic -C  # 自动读取配置
```

## ✅ 特性

- ✅ 完全静态链接，无需依赖
- ✅ 支持 CentOS 7 / RHEL 7
- ✅ 可在离线环境使用
- ✅ 包含 JSON、Avro、事务支持

## 🔗 更多信息

- [完整文档](BUILD_CENTOS7_BINARY.md)
- [官方文档](https://github.com/edenhill/kcat)
