## 

Ubuntu 24.04 中控制笔记本合盖行为的核心是修改`systemd-logind`服务的配置文件，通过设置参数让系统忽略合盖动作，以下是详细步骤，两种编辑工具可选，操作后立即生效：

### 方法一：用 nano 编辑配置文件（终端通用）

1. **打开配置文件**

   按下

   ```
   Ctrl+Alt+T
   ```

   打开终端，输入以下命令，以管理员权限编辑合盖行为的核心配置文件：

   bash

   运行

   ```bash
   sudo nano /etc/systemd/logind.conf
   ```

2. **修改关键参数**

   ```bash
   # 原本注释状态的内容
   #HandleLidSwitch=suspend
   #HandleLidSwitchExternalPower=suspend
   #HandleLidSwitchDocked=suspend
   
   # 修改后未注释的内容
   HandleLidSwitch=ignore  # 用电池时合盖无动作
   HandleLidSwitchExternalPower=ignore  # 接外接电源时合盖无动作
   HandleLidSwitchDocked=ignore  # 接扩展坞/外接显示器时合盖无动作
   ```

3. **保存并退出**

   ```bash
   sudo systemctl restart systemd-logind.service
   ```

1. 后笔记本散热会受影响，若需长时间合盖运行，建议确保笔记本放置在通风处，避免高温损坏硬件。