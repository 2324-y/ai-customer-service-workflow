# 部署与运维指南
## 1. 基础环境依赖
所有服务基于Docker容器化部署，统一环境版本，规避环境差异问题：
- Docker、Docker Compose
- Dify：1.16.0+
- n8n：2.35.5+
- Redis：7-alpine

## 2. 核心环境配置（关键踩坑点）
### 2.1 Dify 环境变量配置
在Dify `.env` 文件中配置内网访问权限，解决SSRF拦截问题：
```env
SSRF_PROXY_ALLOW_PRIVATE_IPS=true
SSRF_PROXY_ALLOW_LIST=http://n8n:5678,http://172.19.0.15:5678
```

### 2.2 Docker 网络打通命令
```bash
docker network connect docker_default n8n
docker network connect docker_ssrf_proxy_network n8n
```

### 2.3 接口URL规范
Dify调用n8n接口**禁止使用localhost**，统一使用容器内网域名：
`http://n8n:5678/webhook/xxx`

## 3. n8n 数据表初始化
部署前需创建9张核心业务数据表，支撑全功能业务：
`orders`、`logistics`、`return_orders`、`users`、`size_charts`、`coupons`、`sessions`、`messages`、`tool_calls`

## 4. 核心凭证配置
### 4.1 Redis 凭证
- Host：redis
- Port：6379
- Password：difyai123456

### 4.2 第三方凭证
- Dify API：对应应用密钥（app-xxxxxx）
- 飞书API：App ID、App Secret（用于人工通知推送）

## 5. 部署流程
1. 启动Docker容器，部署Redis、Dify、n8n服务
2. 配置环境变量与Docker网络
3. n8n导入工作流JSON文件，初始化业务数据表
4. Dify导入DSL流程、配置大模型供应商、上传知识库
5. 配置第三方API凭证，启动服务测试连通性