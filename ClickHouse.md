# ClickHouse

## Installation:

```shell
sudo apt-get install -y apt-transport-https ca-certificates dirmngr
sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv 8919F6BD2B48D754

echo "deb https://packages.clickhouse.com/deb stable main" | sudo tee \
    /etc/apt/sources.list.d/clickhouse.list
sudo apt-get update

sudo apt-get install -y clickhouse-server clickhouse-client

sudo service clickhouse-server start
clickhouse-client # or "clickhouse-client --password" if you've set up a password.
```



## Connect:

```shell
clickhouse-client --host 192.168.2.4 --port 10000
```



## Change Time Zone:

​	change OS Time Zone.

​	![image-20231115201948547](C:\Users\sheng\AppData\Roaming\Typora\typora-user-images\image-20231115201948547.png)

## Restart Service:

```shell
sudo service clickhouse-server restart
```

## Config YAML file:

```yaml
root@vm-ubuntu:/home/ubuntu# cat /etc/clickhouse-server/config.d/my.yml
listen_host: 0.0.0.0
tcp_port: 10000
```

Python Demo Code:

```python
import clickhouse_connect

client = clickhouse_connect.get_client(host='192.168.2.4',port=8123, username='default', password='...')
client.command('CREATE TABLE new_table (key UInt32, value String, metric Float64) ENGINE MergeTree ORDER BY key')
row1 = [1000, 'String Value 1000', 5.233]
row2 = [2000, 'String Value 2000', -107.04]
data = [row1, row2]
client.insert('new_table', data, column_names=['key', 'value', 'metric'])
result = client.query('SELECT max(key), avg(metric) FROM new_table')
print(result.result_rows)
```

python Demo Code:

```python
import clickhouse_connect

client = clickhouse_connect.get_client(host='192.168.2.4',port=8123, username='default', password='...')

result = client.query('SELECT timezone()')
print(result.result_rows)
print(client.server_version)

```



# Redis

# Installation:

```shell
sudo apt install lsb-release curl gpg
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] https://packages.redis.io/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/redis.list

sudo apt-get update
sudo apt-get install redis

```

restart service:

```shell
sudo service redis-server restart
```

change protect-mode:

```shell
# By default protected mode is enabled. You should disable it only if
# you are sure you want clients from other hosts to connect to Redis
# even if no authentication is configured.
 protected-mode no

```

# Vector sink to clickhouse server



Configuration:

```toml
[sources.stdin_logs]
type = "stdin"
decoding.codec = "bytes"

[sinks.clickhouse]
type = "clickhouse"
inputs = ["stdin_logs"]
compression = "gzip"
endpoint = "http://192.168.2.4:8123"
database = "nginxdb"
table = "access_logs"
skip_unknown_fields = true
auth.strategy = "basic"
auth.user = "default"
auth.password = "..."

```

## vector config:



```toml
[api]
enabled = true

[sources.stdin]
type = "stdin"

[transforms.tf01]
type = "remap"
inputs = [ "stdin" ]
drop_on_abort = true
source = """
. = parse_json!(.message)
 del(.Id)
 .newID = "GZ"
 .OLDID = .newID
 .timestamp = now()
 .csv = push(parse_csv!("foo bar",delimiter: " "),.timestamp)
 .zz = append(.csv,[3,4])
 """
 timezone = "local"


[sinks.clickhouse]
type = "clickhouse"
inputs = ["tf01"]
compression = "gzip"
endpoint = "http://192.168.2.4:8123"
database = "nginxdb"
table = "access_logs"
skip_unknown_fields = true
auth.strategy = "basic"
auth.user = "default"
auth.password = "..."


[sinks.out]
inputs = ["tf01"]
type = "console"
encoding.codec = "json"
```



vector.yaml

(default YAML)

```yaml
sources:
  in:
    type: "stdin"

sinks:
  out:
    inputs: [ "in" ]
    type: "clickhouse"
    compression: "gzip"
    endpoint: "http://192.168.2.4:8123"
    database: "nginxdb"
    table: "access_logs"
    skip_unknown_fields: true
    auth:
      strategy: "basic"
      user: "default"
      password: "..."
```



footprint: n.脚印，足迹；占用空间；覆盖区





FAQ:

1. ClickHouse client version 23.10.4.25 (official build).
   Connecting to 192.168.2.4:10000 as user default.
   Connected to ClickHouse server version 23.10.3 revision 54466.

   ClickHouse server version is older than ClickHouse client. It may indicate that the server is out of date and can be upgraded.

   Warnings:

   *Delay accounting is not enabled, OSIOWaitMicroseconds will not be gathered. Check /proc/sys/kernel/task_delayacct

