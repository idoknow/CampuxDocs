# 环境 & 外部组件

## 要求

### 对你的要求

- 具有很强的资料查找能力、自学能力、解决问题能力
- 了解 Docker 和 Docker Compose 的基本使用
- 了解如何配置 MongoDB 和 Redis
- 了解如何配置 MinIO
- 了解如何配置 Nginx 或 Caddy

### 系统环境

- 已安装 Docker 和 Docker Compose
- 仅支持 x86_64 架构 Linux
- 主机必须能无障碍高速访问 ghcr.io（推荐香港、台湾、日本等地区的服务器）
    - 测试方法：`docker pull ghcr.io/idoknow/campux:main`

推荐在同一台宿主机上运行所有容器，若要如此做，请先创建一个专用网络：

```bash
docker network create -d bridge shared-network
```

### 注意

后续提到的配置内容中，`<xxx>`这种以尖括号包裹的内容，表示需要你自行替换为你的内容，记得去掉尖括号！

## 部署外部组件

以下组件建议每个组件一个目录，单独一个 docker-compose.yaml 文件，也可以根据需求自行调整。  

> 以下这些组件的问题，请自行查找其文档

### MongoDB

参考 docker-compose.yaml

```yaml
services:
  mongodb:
    image: mongo
    restart: always
    container_name: mongodb
    volumes:
      - ./mongodata:/data/db
    # ports:
    #   - "27017:27017"
    networks:
      - shared-network
    environment:
      - MONGO_INITDB_ROOT_USERNAME=root
      - MONGO_INITDB_ROOT_PASSWORD=<把这里改成你的密码>
      - TZ=Asia/Shanghai

networks:
  shared-network:
    external: true
```

```bash
docker compose up -d
```

### Redis

在目录下创建一个 redis.conf 文件，内容如下：

```conf
requirepass <你的密码>
```

参考 docker-compose.yaml

```yaml
services:
  redis:
    image: redis:7.2.4
    container_name: redis
    restart: always
    environment:
      - TZ=Asia/Shanghai
    volumes:
      - ./data:/data
      - ./redis.conf:/usr/local/etc/redis/redis.conf
    networks:
      - shared-network
    command: redis-server /usr/local/etc/redis/redis.conf

networks:
  shared-network:
    external: true
```

```bash
docker compose up -d
```

### MinIO

参考 docker-compose.yaml

```yaml
services:
  minio:
    image: quay.io/minio/minio
    container_name: minio
    environment:
      - MINIO_ROOT_USER=root
      - MINIO_ROOT_PASSWORD=<把这里改成你的密码>
    ports:
      - "9000:9000"
      - "9090:9090"
    volumes:
      - './data/minio:/data'
    command: server /data --console-address ":9090"
    networks:
      - shared-network

networks:
  shared-network:
    external: true
```

```bash
docker compose up -d
```

你应该会自己推测出 MinIO 的访问地址，提醒注意区分 9000 端口是 MinIO API 端口，9090 端口是控制台端口。  
访问 MinIO 控制台，创建一个 bucket，记住 bucket 名称，生成 Access Key 和 Secret Key，记下来。
