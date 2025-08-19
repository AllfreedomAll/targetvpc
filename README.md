# TargetVPC - 服务器监控代理

[![Python](https://img.shields.io/badge/Python-3.11+-blue.svg)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.104.1+-green.svg)](https://fastapi.tiangolo.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

TargetVPC 是一个高性能的服务器监控代理系统，专为分布式环境设计。它能够实时收集服务器关键指标，支持多种告警通知方式，并提供灵活的配置管理和数据查询API。

## 🚀 功能特性

### 📊 系统监控
- **CPU使用率监控** - 实时监控CPU使用情况
- **内存使用率监控** - 监控内存占用和可用性
- **磁盘使用率监控** - 监控磁盘空间使用情况
- **可配置的监控间隔** - 支持自定义数据收集频率

### 🔔 智能告警
- **多通道告警支持**
  - 📧 邮件告警
  - 🚀 飞书机器人告警
  - 🔔 钉钉机器人告警
- **阈值告警** - 支持 >, >=, <, <= 等操作符
- **告警冷却机制** - 防止告警风暴
- **告警记录上报** - 自动向管理端上报告警信息

### ⚙️ 配置管理
- **动态配置同步** - 从管理端自动获取最新配置
- **IP自动识别** - 自动识别服务器IP地址
- **配置热更新** - 无需重启即可应用新配置

### 🌐 API接口
- **RESTful API** - 基于FastAPI构建
- **指标数据查询** - 支持时间范围查询
- **健康检查接口** - 系统状态监控
- **配置查询接口** - 获取当前配置信息

### 🗄️ 数据存储
- **Redis存储** - 高性能内存数据库
- **数据保留策略** - 可配置的数据保留时间
- **时间序列数据** - 支持历史数据查询

## 🏗️ 系统架构

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   管理端Admin   │    │   TargetVPC     │    │   告警通道      │
│                 │◄──►│                 │───►│                 │
│ - 配置管理      │    │ - 指标收集      │    │ - 邮件         │
│ - 告警记录      │    │ - 阈值检查      │    │ - 飞书         │
│ - 数据查询      │    │ - 告警发送      │    │ - 钉钉         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
                                │
                                ▼
                       ┌─────────────────┐
                       │     Redis       │
                       │                 │
                       │ - 指标存储      │
                       │ - 配置缓存      │
                       └─────────────────┘
```

## 📋 系统要求

- **Python**: 3.11+
- **操作系统**: Linux, macOS, Windows
- **内存**: 至少 512MB RAM
- **磁盘**: 至少 100MB 可用空间
- **网络**: 需要网络连接以访问管理端和告警服务

## 🛠️ 安装部署

### 方式一：Docker部署（推荐）

1. **克隆项目**
```bash
git clone <repository-url>
cd targetvpc
```

2. **启动服务**
```bash
docker-compose up -d
```

3. **验证部署**
```bash
curl http://localhost:8001/health
```

### 方式二：本地安装

1. **创建虚拟环境**
```bash
python -m venv venv
source venv/bin/activate  # Linux/macOS
# 或
venv\Scripts\activate     # Windows
```

2. **安装依赖**
```bash
pip install -r requirements.txt
```

3. **配置Redis**
```bash
# 启动Redis服务
redis-server

# 或使用Docker
docker run -d -p 6379:6379 redis:7-alpine
```

4. **启动服务**
```bash
python -m src.main
```

## ⚙️ 配置说明

### 配置文件位置
- 主配置文件: `config/default.json`
- 环境变量: 支持通过环境变量覆盖配置

### 配置项说明

```json
{
    "redis": {
        "host": "localhost",        // Redis主机地址
        "port": 6379,              // Redis端口
        "db": 0,                   // Redis数据库编号
        "password": null           // Redis密码（可选）
    },
    "admin": {
        "base_url": "http://localhost:8000",           // 管理端基础URL
        "config_endpoint": "/api/servers/config/",     // 配置获取接口
        "alert_endpoint": "/api/alert-records/report/" // 告警上报接口
    },
    "metrics": {
        "collection_interval": 10,  // 指标收集间隔（秒）
        "retention_days": 7         // 数据保留天数
    },
    "alerts": {
        "cooldown_minutes": 5       // 告警冷却时间（分钟）
    },
    "server": {
        "host": "0.0.0.0",         // 服务监听地址
        "port": 8001               // 服务监听端口
    }
}
```

### 环境变量配置

```bash
# Redis配置
export REDIS_HOST=your-redis-host
export REDIS_PORT=6379
export REDIS_PASSWORD=your-password

# 管理端配置
export ADMIN_BASE_URL=http://your-admin-server:8000

# 服务配置
export SERVER_HOST=0.0.0.0
export SERVER_PORT=8001
```

## 📡 API接口

### 健康检查
```http
GET /health
```

### 获取配置
```http
GET /api/config
Headers: X-Client-IP: <client-ip>
```

### 查询指标数据
```http
POST /api/metrics
Content-Type: application/json

{
    "start_time": "2024-01-01T00:00:00Z",
    "end_time": "2024-01-01T23:59:59Z",
    "indexes": ["cpu", "memory", "disk"]
}
```

### 上报告警
```http
POST /api/alert
Content-Type: application/json

{
    "index": "cpu",
    "threshold": 80.0,
    "current_value": 85.0,
    "timestamp": 1704067200
}
```

## 🔧 告警配置

### 飞书机器人配置
```json
{
    "type": "feishu",
    "webhook_url": "https://open.feishu.cn/open-apis/bot/v2/hook/YOUR_TOKEN",
    "enabled": true
}
```

### 钉钉机器人配置
```json
{
    "type": "dingtalk",
    "webhook_url": "https://oapi.dingtalk.com/robot/send?access_token=YOUR_TOKEN",
    "enabled": true
}
```

### 邮件告警配置
```json
{
    "type": "email",
    "smtp_server": "smtp.gmail.com",
    "smtp_port": 587,
    "username": "your-email@gmail.com",
    "password": "your-password",
    "to_emails": ["admin@company.com"],
    "enabled": true
}
```

## 📊 监控指标

### 系统指标
- **cpu**: CPU使用率百分比 (0-100%)
- **memory**: 内存使用率百分比 (0-100%)
- **disk**: 磁盘使用率百分比 (0-100%)

### 指标格式
```json
{
    "timestamp": 1704067200,
    "metrics": {
        "cpu": 45.2,
        "memory": 67.8,
        "disk": 23.1
    }
}
```

## 🚀 性能优化

### 配置建议
- **监控间隔**: 生产环境建议设置为 30-60 秒
- **数据保留**: 根据存储容量调整保留天数
- **告警冷却**: 避免频繁告警，建议 5-10 分钟

### 资源使用
- **CPU**: 通常 < 1%
- **内存**: 约 50-100MB
- **网络**: 根据监控频率和告警频率而定

## 🔍 故障排查

### 常见问题

1. **无法连接Redis**
   ```bash
   # 检查Redis服务状态
   redis-cli ping
   
   # 检查网络连接
   telnet localhost 6379
   ```

2. **配置同步失败**
   ```bash
   # 检查管理端服务状态
   curl http://your-admin-server:8000/health
   
   # 检查网络连通性
   ping your-admin-server
   ```

3. **告警发送失败**
   ```bash
   # 检查告警配置
   # 验证webhook URL有效性
   # 检查网络防火墙设置
   ```

### 日志查看
```bash
# Docker环境
docker-compose logs targetvpc

# 本地环境
# 日志输出到控制台，建议重定向到文件
python -m src.main > targetvpc.log 2>&1
```

## 🧪 测试

### 运行测试
```bash
python test_targetvpc.py
```

### 手动测试API
```bash
# 健康检查
curl http://localhost:8001/health

# 获取配置
curl -H "X-Client-IP: 127.0.0.1" http://localhost:8001/api/config

# 查询指标
curl -X POST http://localhost:8001/api/metrics \
  -H "Content-Type: application/json" \
  -d '{"start_time":"2024-01-01T00:00:00Z","end_time":"2024-01-01T23:59:59Z","indexes":["cpu"]}'
```

## 📈 扩展开发

### 添加新的监控指标
1. 在 `src/metrics/collector.py` 中添加新的收集方法
2. 在 `collect_all_metrics()` 方法中集成新指标
3. 更新配置中的阈值设置

### 添加新的告警通道
1. 在 `src/alerts/handlers/` 目录下创建新的处理器
2. 实现 `send_alert()` 方法
3. 在 `AlertSender` 中注册新的处理器

### 自定义存储后端
1. 继承 `Storage` 基类
2. 实现必要的方法
3. 更新配置以使用新的存储后端

## 🤝 贡献指南

1. Fork 项目
2. 创建功能分支 (`git checkout -b feature/AmazingFeature`)
3. 提交更改 (`git commit -m 'Add some AmazingFeature'`)
4. 推送到分支 (`git push origin feature/AmazingFeature`)
5. 打开 Pull Request

## 📄 许可证

本项目采用 MIT 许可证 - 查看 [LICENSE](LICENSE) 文件了解详情。

## 📞 联系方式

- 项目维护者: guo.ling
- 邮箱: guo.ling@outlook.com
- 项目地址: [https://github.com/AllfreedomAll/targetvpc]

## 🙏 致谢

感谢以下开源项目的支持：
- [FastAPI](https://fastapi.tiangolo.com/) - 现代、快速的Web框架
- [psutil](https://github.com/giampaolo/psutil) - 跨平台系统监控库
- [Redis](https://redis.io/) - 高性能内存数据库
- [Uvicorn](https://www.uvicorn.org/) - 轻量级ASGI服务器

---

**注意**: 在生产环境中使用前，请确保：
- 修改默认密码和敏感配置
- 配置适当的防火墙规则
- 设置监控和告警阈值
- 定期备份配置和数据 
