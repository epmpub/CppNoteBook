# python 3.11 安装指引

查看系统自带的python 是3.10.6

**具体步骤:**

首先安装pkg-config:

```bash
sudo apt install pkg-config
```

​  1. install the libraries and dependencies necessary to build Python:

```bash
sudo apt install build-essential zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev libsqlite3-dev wget libbz2-dev
```

​  2 .Download the latest release’s source code from the [Python download page](https://www.python.org/downloads/source/) using the [`wget`](https://linuxize.com/post/wget-command-examples/) command:

```bash
wget https://www.python.org/ftp/python/3.11.5/Python-3.11.5.tgz
```

​        3 解压文件:

```basic
tar -xf Python-3.11.5.tgz
```

​  4 进入目录,开始配置.

```bash
cd Python-3.11.5
./configure --enable-optimizations
```

5 开始编译:

```shell
make -j <CPU个数> # cpu个数通过nproc命令看;
```

6 开始安装:

```bash
sudo make altinstall
```

7 测试:

```bash
python3.11 --version

Output:
Python 3.11.3
```
