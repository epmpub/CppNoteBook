列出 Docker 引擎中所有容器的方法：

## 基本命令

### 1. **列出正在运行的容器**

```bash
docker ps
```

### 2. **列出所有容器（包括停止的）**

```bash
docker ps -a
# 或
docker container ls -a
```

### 3. **只显示容器 ID**

```bash
docker ps -aq
```

## 自定义输出格式

### 4. **使用 --format 自定义列**

```bash
# 显示 ID、名称、状态
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Status}}"

# 显示更多信息
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"

# JSON 格式输出
docker ps -a --format "{{json .}}"
```

### 5. **常用的格式化字段**

```bash
# 完整的信息显示
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.CreatedAt}}\t{{.Size}}"
```

可用字段：

- `.ID` - 容器 ID
- `.Names` - 容器名称
- `.Image` - 使用的镜像
- `.Command` - 启动命令
- `.CreatedAt` - 创建时间
- `.Status` - 状态
- `.Ports` - 端口映射
- `.Size` - 容器大小
- `.Labels` - 标签
- `.Mounts` - 挂载卷

## 过滤容器

### 6. **按状态过滤**

```bash
# 只显示运行中的
docker ps --filter "status=running"

# 只显示停止的
docker ps -a --filter "status=exited"

# 其他状态：created, restarting, paused, dead
docker ps -a --filter "status=paused"
```

### 7. **按名称过滤**

```bash
# 包含特定名称的容器
docker ps -a --filter "name=nginx"

# 使用正则表达式
docker ps -a --filter "name=^/web"
```

### 8. **按镜像过滤**

```bash
# 使用特定镜像的容器
docker ps -a --filter "ancestor=nginx:latest"
```

### 9. **按标签过滤**

```bash
docker ps -a --filter "label=env=production"
docker ps -a --filter "label=app=myapp"
```

### 10. **按退出码过滤**

```bash
# 退出码为 0 的容器
docker ps -a --filter "exited=0"

# 退出码非 0 的容器
docker ps -a --filter "exited=1"
```

## 实用脚本

### 11. **列出所有容器的详细信息**

```bash
#!/bin/bash
# list-all-containers.sh

echo "=== All Containers ==="
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.CreatedAt}}"

echo -e "\n=== Running Containers: $(docker ps -q | wc -l) ==="
echo "=== Stopped Containers: $(docker ps -aq -f status=exited | wc -l) ==="
echo "=== Total Containers: $(docker ps -aq | wc -l) ==="
```

### 12. **列出容器并显示资源使用情况**

```bash
# 显示 CPU 和内存使用
docker stats --no-stream

# 持续监控
docker stats

# 只显示特定容器
docker stats container1 container2
```

### 13. **列出容器及其端口映射**

```bash
docker ps --format "table {{.Names}}\t{{.Ports}}"
```

### 14. **列出容器及其数据卷**

```bash
docker ps -a --format "table {{.Names}}\t{{.Mounts}}"
```

## 跨多个引擎列出容器

### 15. **列出所有 context 中的容器**

```bash
#!/bin/bash
# list-all-engines-containers.sh

for context in $(docker context ls -q); do
    echo "=== Context: $context ==="
    docker --context $context ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Image}}"
    echo ""
done
```

### 16. **统计各引擎的容器数量**

```bash
#!/bin/bash
# count-containers-all-engines.sh

for context in $(docker context ls -q); do
    running=$(docker --context $context ps -q | wc -l)
    stopped=$(docker --context $context ps -aq -f status=exited | wc -l)
    total=$(docker --context $context ps -aq | wc -l)
    
    echo "$context: Total=$total, Running=$running, Stopped=$stopped"
done
```

## 导出容器列表

### 17. **导出为 CSV**

```bash
# 生成 CSV 格式
docker ps -a --format "{{.ID}},{{.Names}},{{.Image}},{{.Status}},{{.CreatedAt}}" > containers.csv
```

### 18. **导出为 JSON**

```bash
docker ps -a --format "{{json .}}" | jq '.' > containers.json
```

### 19. **生成 HTML 报告**

```bash
docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}" | \
  awk 'BEGIN{print "<html><body><table border=1>"} 
       {print "<tr><td>"$1"</td><td>"$2"</td><td>"$3"</td><td>"$4"</td></tr>"} 
       END{print "</table></body></html>"}' > containers.html
```

## 高级查询

### 20. **列出最近创建的容器**

```bash
# 最近创建的 5 个容器
docker ps -a -n 5

# 或使用 --last
docker ps -a --last 10
```

### 21. **按创建时间排序**

```bash
# 需要结合其他工具
docker ps -a --format "{{.CreatedAt}}\t{{.Names}}" | sort
```

### 22. **查找占用空间最大的容器**

```bash
docker ps -a --size --format "table {{.Size}}\t{{.Names}}" | sort -h
```

## 实用别名

在 `~/.bashrc` 或 `~/.zshrc` 中添加：

```bash
# 列出所有容器
alias dps='docker ps -a'

# 列出正在运行的容器
alias dpsr='docker ps'

# 详细格式列出所有容器
alias dpsl='docker ps -a --format "table {{.ID}}\t{{.Names}}\t{{.Image}}\t{{.Status}}\t{{.Ports}}"'

# 只显示容器名称
alias dpsn='docker ps -a --format "{{.Names}}"'

# 统计容器数量
alias dcount='echo "Running: $(docker ps -q | wc -l), Stopped: $(docker ps -aq -f status=exited | wc -l)"'
```

## 使用 Docker API（高级）

```bash
# 通过 API 列出容器
curl --unix-socket /var/run/docker.sock http://localhost/containers/json?all=1 | jq '.'
```

这些命令涵盖了从基本到高级的各种列出容器的方法。你最常用的场景是什么？需要针对特定需求优化命令吗？