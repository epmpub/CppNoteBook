安装 `postgresql-client-18`

安装 `postgresql-client-18`





这个报错**非常正常**，而且信息量很大 👍
结论先给：

> **你的系统里还没配置 PostgreSQL 官方（PGDG）APT 仓库**
> 所以 `apt` 根本不知道 `postgresql-client-18` 这个包的存在

下面一步一步来，**这是标准做法**。

------

## 一、为什么会 `Unable to locate package`

你现在的环境是：

- OS：Ubuntu 22.04 (jammy)
- `apt` 默认源：Ubuntu 官方仓库

👉 Ubuntu 官方仓库 **目前只提供 PostgreSQL 14**

```bash
apt search postgresql-client
```

你大概率只能看到：

```
postgresql-client-14
postgresql-client-common
```

📌 **PG 18 不在 Ubuntu 官方源里**

------

## 二、你已经装了 PG 18 Server，说明什么？

你之前能连上：

```text
server 18.0 (Debian 18.0-1.pgdg13+3)
```

说明你 **已经添加过 PGDG 仓库（但可能不完整）**，或者：

- 只加了 server
- 没加 client
- 或 apt key / source 有问题

------

## 三、正确安装 `postgresql-client-18`（标准步骤）

### ✅ Step 1：确认 Ubuntu 版本

```bash
lsb_release -cs
```

应该输出：

```
jammy
```

------

### ✅ Step 2：添加 PGDG 官方仓库（关键）

```bash
sudo apt install -y wget ca-certificates gnupg

wget -qO - https://www.postgresql.org/media/keys/ACCC4CF8.asc \
  | sudo gpg --dearmor -o /usr/share/keyrings/postgresql.gpg
echo "deb [signed-by=/usr/share/keyrings/postgresql.gpg] \
http://apt.postgresql.org/pub/repos/apt \
$(lsb_release -cs)-pgdg main" \
| sudo tee /etc/apt/sources.list.d/pgdg.list
```

------

### ✅ Step 3：更新包列表

```bash
sudo apt update
```

你现在应该能看到类似：

```
apt.postgresql.org
```

------

### ✅ Step 4：安装 PG 18 Client

```bash
sudo apt install postgresql-client-18
```

------

### ✅ Step 5：验证

```bash
psql --version
```

应该是：

```
psql (PostgreSQL) 18.0
```

------

## 四、如果你想“并存多个 psql 版本”

这是完全支持的 👍

```bash
ls /usr/lib/postgresql/
```

可能看到：

```
14/
18/
```

切换方式：

```bash
/usr/lib/postgresql/18/bin/psql demo
```

或设 PATH：

```bash
export PATH=/usr/lib/postgresql/18/bin:$PATH
```

------

## 五、如果你不想折腾（可选方案）

你也可以**继续用 psql 14 连接 PG 18**

✔️ CREATE DATABASE
✔️ CREATE SCHEMA
✔️ CREATE TABLE
✔️ 外键 / 事务

**都没问题**

⚠️ 只是 `\d+` 等元命令显示可能不完整

------

## 六、你现在最关键的下一步

建议你现在做其中一个：

### 👉 方案 A（推荐）

按上面的步骤装 `postgresql-client-18`

### 👉 方案 B（懒人）

继续用现在的 psql，直接建表

```sql
CREATE SCHEMA demo;
CREATE TABLE demo.orders (id BIGINT PRIMARY KEY);
```

------

如果你愿意，下一步我可以帮你：

- 检查 **PGDG 仓库是否已正确配置**
- 看你 **PG 18 server 是怎么装上的**
- 给你一份 **Ubuntu + PostgreSQL 18 的最小正确安装清单**

你想继续排哪一步？