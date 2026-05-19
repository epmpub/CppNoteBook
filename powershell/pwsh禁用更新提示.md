#### 配置 PowerShell 配置文件

通过配置文件永久禁用更新提示（即使不用 `-NoLogo` 也能屏蔽）：

1. 先创建 / 编辑 PowerShell 配置文件：

   ```
   # 打开当前用户的配置文件（若不存在会提示创建）
   notepad $PROFILE
   ```

2. 在配置文件中添加以下内容（二选一，推荐第一种）：

   ```
   # 方式 1：仅禁用更新提示（保留其他启动信息）
   $env:POWERSHELL_UPDATECHECK = 'Off'
   
   # 方式 2：彻底禁用所有启动徽标/提示（等效于 -NoLogo）
   # [Microsoft.PowerShell.PSConsoleReadLine]::Options.EchoMode = 'None'（仅辅助，核心还是 -NoLogo 或上面的环境变量）
   ```

3. 保存文件并重启 pwsh，更新提示就会消失