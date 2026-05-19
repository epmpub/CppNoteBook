删除 Docker 容器的各种方法：

## 基本删除命令

### 1. **删除单个容器**

```bash
# 删除已停止的容器
docker rm container_name

# 或使用容器 ID
docker rm container_id

# 强制删除（即使正在运行）
docker rm -f container_name
```

### 2. **删除多个容器**

```bash
# 删除多个指定容器
docker rm container1 container2 container3

# 强制删除多个容器
docker rm -f container1 container2 container3
```

## 批量删除

### 3. **删除所有已停止的容器**

```bash
docker container prune

# 不显示确认提示
docker container prune -f
```

### 4. **删除所有容器（包括运行中的）**

```bash
# 停止所有容器后删除
docker stop $(docker ps -aq)

docker rm $(docker ps -aq) ** 注意 **

# 或一步完成（强制删除）
docker rm -f $(docker ps -aq)
```

### 5. **删除所有已停止的容器（传统方法）**

```bash
docker rm $(docker ps -aq -f status=exited)
```

## 按条件删除

### 6. **删除特定状态的容器**

```bash
# 删除所有退出的容器
docker rm $(docker ps -aq -f status=exited)

# 删除所有已创建但未启动的容器
docker rm $(docker ps -aq -f status=created)

# 删除所有死亡的容器
docker rm $(docker ps -aq -f status=dead)
```

### 7. **按名称模式删除**

```bash
# 删除名称包含 "test" 的所有容器
docker rm $(docker ps -aq --filter "name=test")

# 删除名称以 "tmp_" 开头的容器
docker rm $(docker ps -aq --filter "name=^tmp_")
```

### 8. **按镜像删除容器**

```bash
# 删除使用特定镜像的所有容器
docker rm $(docker ps -aq --filter "ancestor=nginx:latest")
```

### 9. **按退出码删除**

```bash
# 删除退出码非 0 的容器
docker rm $(docker ps -aq -f exited=1)

# 删除退出码为 0 的容器
docker rm $(docker ps -aq -f exited=0)
```

### 10. **按时间删除（旧容器）**

```bash
# 删除创建超过 24 小时的容器
docker container prune --filter "until=24h"

# 删除创建超过 7 天的容器
docker container prune --filter "until=168h"
```

## 实用脚本

### 11. **安全删除脚本（带确认）**

```bash
#!/bin/bash
# safe-remove-containers.sh

echo "=== Containers to be removed ==="
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

read -p "Are you sure you want to remove ALL containers? (yes/no): " confirm

if [ "$confirm" == "yes" ]; then
    docker rm -f $(docker ps -aq)
    echo "All containers removed!"
else
    echo "Operation cancelled."
fi
```

### 12. **删除旧的测试容器**

```bash
#!/bin/bash
# cleanup-test-containers.sh

# 删除名称包含 test 或 tmp 的已停止容器
docker ps -aq --filter "name=test" --filter "status=exited" | xargs -r docker rm
docker ps -aq --filter "name=tmp" --filter "status=exited" | xargs -r docker rm

echo "Test containers cleaned up!"
```

### 13. **按项目清理容器**

```bash
#!/bin/bash
# cleanup-project.sh

PROJECT_NAME=$1

if [ -z "$PROJECT_NAME" ]; then
    echo "Usage: $0 <project_name>"
    exit 1
fi

echo "Removing containers for project: $PROJECT_NAME"
docker rm -f $(docker ps -aq --filter "label=project=$PROJECT_NAME")
```

### 14. **定期清理脚本**

```bash
#!/bin/bash
# periodic-cleanup.sh

echo "=== Docker Cleanup Started ==="

# 删除已停止超过 24 小时的容器
echo "Removing stopped containers older than 24h..."
docker container prune -f --filter "until=24h"

# 删除未使用的网络
echo "Removing unused networks..."
docker network prune -f

# 删除悬空镜像
echo "Removing dangling images..."
docker image prune -f

# 删除未使用的卷（谨慎使用）
# docker volume prune -f

echo "=== Cleanup Complete ==="
```

## Docker Compose 项目清理

### 15. **删除 Compose 项目的所有容器**

```bash
# 停止并删除容器
docker-compose down

# 删除容器和卷
docker-compose down -v

# 删除容器、卷和镜像
docker-compose down -v --rmi all

# 强制删除（不等待容器优雅停止）
docker-compose down --remove-orphans -t 0
```

### 16. **删除特定 Compose 项目**

```bash
# 在项目目录中
cd /path/to/project
docker-compose down -v

# 或指定项目名称
docker-compose -p myproject down -v
```

## 高级删除选项

### 17. **删除容器并显示详细信息**

```bash
# 逐个删除并显示详情
for container in $(docker ps -aq); do
    echo "Removing: $(docker inspect --format='{{.Name}}' $container)"
    docker rm -f $container
done
```

### 18. **删除容器前备份日志**

```bash
#!/bin/bash
# backup-and-remove.sh

CONTAINER=$1
BACKUP_DIR="./container-logs"

mkdir -p $BACKUP_DIR

echo "Backing up logs for $CONTAINER..."
docker logs $CONTAINER > "$BACKUP_DIR/${CONTAINER}_$(date +%Y%m%d_%H%M%S).log" 2>&1

echo "Removing container..."
docker rm -f $CONTAINER

echo "Done!"
```

### 19. **删除并导出容器（保留数据）**

```bash
#!/bin/bash
# export-and-remove.sh

CONTAINER=$1

echo "Exporting $CONTAINER..."
docker export $CONTAINER > "${CONTAINER}_backup.tar"

echo "Removing $CONTAINER..."
docker rm -f $CONTAINER

echo "Container exported to ${CONTAINER}_backup.tar"
```

## 跨多个引擎删除

### 20. **删除所有引擎中的容器**

```bash
#!/bin/bash
# remove-all-engines.sh

for context in $(docker context ls -q); do
    echo "=== Cleaning context: $context ==="
    docker --context $context rm -f $(docker --context $context ps -aq)
done
```

### 21. **清理特定引擎**

```bash
# 切换到特定引擎并清理
docker context use remote-engine
docker container prune -f

# 或直接指定 context
docker --context remote-engine container prune -f
```

## 系统级清理

### 22. **完整的系统清理**

```bash
# 删除所有停止的容器、未使用的网络、悬空镜像和构建缓存
docker system prune

# 包括所有未使用的镜像（不仅是悬空的）
docker system prune -a

# 包括卷（⚠️ 危险：会删除数据）
docker system prune -a --volumes

# 强制执行，不提示确认
docker system prune -af --volumes
```

### 23. **查看可清理的空间**

```bash
docker system df

# 详细信息
docker system df -v
```

## 注意事项和最佳实践

### ⚠️ **删除前的检查清单**

```bash
# 1. 列出要删除的容器
docker ps -a

# 2. 检查容器的数据卷
docker inspect container_name | grep -A 10 Mounts

# 3. 备份重要日志
docker logs container_name > backup.log

# 4. 检查容器的网络连接
docker inspect container_name | grep -A 10 Networks

# 5. 确认容器可以重建
docker inspect container_name --format='{{.Config.Image}}'
```

## 实用别名

在 `~/.bashrc` 或 `~/.zshrc` 中添加：

```bash
# 删除所有已停止的容器
alias dclean='docker container prune -f'

# 删除所有容器（强制）
alias drma='docker rm -f $(docker ps -aq)'

# 系统清理
alias dsysclean='docker system prune -af'

# 删除已退出的容器
alias drme='docker rm $(docker ps -aq -f status=exited)'

# 停止并删除所有容器
alias dstopall='docker stop $(docker ps -aq) && docker rm $(docker ps -aq)'
```

## 错误处理

### 24. **安全删除（处理不存在的容器）**

```bash
# 使用 xargs 避免错误
docker ps -aq | xargs -r docker rm -f

# 或在脚本中检查
if [ $(docker ps -aq | wc -l) -gt 0 ]; then
    docker rm -f $(docker ps -aq)
else
    echo "No containers to remove"
fi
```

你需要删除特定类型的容器，还是想设置定期清理任务？