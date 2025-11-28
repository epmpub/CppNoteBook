我来详细介绍如何在不同 Docker Engine 之间迁移镜像和容器。

## 迁移 Docker 镜像

### 方法 1：使用 save 和 load（推荐）

```bash
# 在源引擎上导出镜像
docker save -o myimage.tar myimage:tag

# 或导出多个镜像
docker save -o images.tar image1:tag image2:tag

# 切换到目标引擎
docker context use target-engine

# 导入镜像
docker load -i myimage.tar
```

### 方法 2：通过 Registry（最佳实践）

```bash
# 在源引擎上推送到 Registry
docker tag myimage:tag registry.example.com/myimage:tag
docker push registry.example.com/myimage:tag

# 在目标引擎上拉取
docker context use target-engine
docker pull registry.example.com/myimage:tag
```

### 方法 3：通过管道直接传输（无需中间文件）

```bash
# 源引擎导出并直接传输到目标引擎
docker save myimage:tag | docker --context target-engine load

# 或通过 SSH 传输到远程主机
docker save myimage:tag | ssh user@remote-host docker load
```

## 迁移 Docker 容器

### 方法 1：commit 容器为镜像后迁移

```bash
# 1. 将容器提交为镜像
docker commit container-name my-container-image:v1

# 2. 导出镜像
docker save -o container-backup.tar my-container-image:v1

# 3. 切换到目标引擎并导入
docker context use target-engine
docker load -i container-backup.tar

# 4. 从镜像创建新容器
docker run -d --name new-container my-container-image:v1
```

### 方法 2：使用 export 和 import（仅文件系统）

```bash
# 导出容器文件系统
docker export container-name > container-fs.tar

# 在目标引擎上导入为镜像
docker context use target-engine
docker import container-fs.tar myimage:imported

# 创建新容器
docker run -d --name new-container myimage:imported
```

⚠️ **注意**：`export` 会丢失镜像历史和元数据（如 CMD、ENTRYPOINT 等）

## 批量迁移脚本

### 迁移所有镜像

```bash
#!/bin/bash
# migrate-all-images.sh

SOURCE_CONTEXT="default"
TARGET_CONTEXT="remote"

# 获取所有镜像
docker context use $SOURCE_CONTEXT
images=$(docker images --format "{{.Repository}}:{{.Tag}}" | grep -v "<none>")

# 迁移每个镜像
for image in $images; do
    echo "Migrating $image..."
    docker save $image | docker --context $TARGET_CONTEXT load
done

echo "Migration complete!"
```

### 迁移特定容器及其数据卷

```bash
#!/bin/bash
# migrate-container.sh

CONTAINER_NAME=$1
SOURCE_CONTEXT="default"
TARGET_CONTEXT="remote"

docker context use $SOURCE_CONTEXT

# 1. 获取容器配置
docker inspect $CONTAINER_NAME > container-config.json

# 2. 提交为镜像
docker commit $CONTAINER_NAME temp-migration-image

# 3. 导出镜像
docker save temp-migration-image | docker --context $TARGET_CONTEXT load

# 4. 导出数据卷（如果有）
volumes=$(docker inspect -f '{{range .Mounts}}{{.Name}} {{end}}' $CONTAINER_NAME)
for vol in $volumes; do
    docker run --rm -v $vol:/data -v $(pwd):/backup alpine tar czf /backup/$vol.tar.gz -C /data .
done

# 5. 在目标引擎上创建容器
docker context use $TARGET_CONTEXT
# 需要根据 container-config.json 手动重建容器配置
```

## 使用 Docker Compose 迁移整个应用栈

```bash
# 1. 在源引擎上保存所有镜像
docker-compose config --images | xargs docker save -o app-images.tar

# 2. 传输文件到目标主机
scp app-images.tar docker-compose.yml user@remote-host:~/

# 3. 在目标引擎上加载和启动
ssh user@remote-host
docker load -i app-images.tar
docker-compose up -d
```

## 实用工具和技巧

### 使用压缩减少传输时间

```bash
# 导出并压缩
docker save myimage:tag | gzip > myimage.tar.gz

# 解压并导入
gunzip -c myimage.tar.gz | docker load
```

### 通过网络直接传输（适合远程主机）

```bash
# 通过 SSH 直接传输
docker save myimage:tag | pv | ssh user@remote-host 'docker load'

# 使用 netcat 在本地网络传输（更快）
# 接收端
docker load < <(nc -l -p 7000)

# 发送端
docker save myimage:tag | nc remote-host 7000
```

### 迁移持久化数据卷

```bash
# 备份数据卷
docker run --rm \
  -v source-volume:/source \
  -v $(pwd):/backup \
  alpine tar czf /backup/volume-backup.tar.gz -C /source .

# 在目标引擎上恢复
docker context use target-engine
docker volume create target-volume
docker run --rm \
  -v target-volume:/target \
  -v $(pwd):/backup \
  alpine tar xzf /backup/volume-backup.tar.gz -C /target
```

## 注意事项

1. **容器状态不保留**：迁移的是容器的文件系统，不是运行状态
2. **网络配置需重建**：自定义网络需要在目标引擎上重新创建
3. **数据卷需单独处理**：命名卷需要单独备份和恢复
4. **环境变量和配置**：记得保存容器的启动参数和环境变量
5. **架构兼容性**：确保目标引擎的 CPU 架构兼容（如 amd64 vs arm64）

你具体需要迁移什么类型的镜像或容器？是开发环境的简单迁移，还是生产环境的完整应用栈？