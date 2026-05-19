**Fast Startup = 注册表 + 休眠状态共同决定**
所以脚本必须 **检测 + 纠正**，而不是盲关。

------

## 一、Fast Startup 的真实判断条件（很关键）

Fast Startup 是否开启，取决于：

1. 注册表：

```text
HKLM\SYSTEM\CurrentControlSet\Control\Session Manager\Power
  HiberbootEnabled = 1  → Fast Startup ON
  HiberbootEnabled = 0  → Fast Startup OFF
```

1. 同时 **Hibernate 必须启用**
   如果 `powercfg /a` 里没有 Hibernate → Fast Startup 实际是 **不可用状态**

------

## 二、PowerShell 脚本（检测 → 关闭 → 验证）

> ⚠️ 需要 **管理员 PowerShell**
> （这是系统级电源策略）

```powershell
# ===============================
# Detect & Disable Fast Startup
# Windows 10 / 11
# ===============================

$regPath = 'HKLM:\SYSTEM\CurrentControlSet\Control\Session Manager\Power'
$regName = 'HiberbootEnabled'

Write-Host "Checking Fast Startup status..." -ForegroundColor Cyan

# 1. 检查注册表键是否存在
if (-not (Test-Path $regPath)) {
    Write-Error "Power registry path not found. Unexpected system state."
    return
}

$hiberboot = Get-ItemProperty -Path $regPath -Name $regName -ErrorAction SilentlyContinue

if ($null -eq $hiberboot) {
    Write-Warning "HiberbootEnabled not found. Fast Startup is likely unsupported."
    return
}

# 2. 判断 Fast Startup 状态
if ($hiberboot.$regName -eq 1) {
    Write-Host "Fast Startup is ENABLED." -ForegroundColor Yellow

    Write-Host "Disabling Fast Startup..." -ForegroundColor Cyan

    # 3. 关闭 Fast Startup（注册表）
    Set-ItemProperty -Path $regPath -Name $regName -Value 0

    # 4. 同时关闭 Hibernate（最彻底）
    Write-Host "Disabling Hibernate (this also guarantees Fast Startup is off)..." -ForegroundColor Cyan
    powercfg /hibernate off | Out-Null

    Write-Host "Fast Startup has been DISABLED." -ForegroundColor Green
}
else {
    Write-Host "Fast Startup is already DISABLED." -ForegroundColor Green
}

# 5. 验证 Hibernate 状态
Write-Host ""
Write-Host "Current sleep states:" -ForegroundColor Cyan
powercfg /a
```

------

## 三、脚本设计说明（为什么这样写）

### 为什么 **不只改注册表**？

- Fast Startup = **Hiberboot + Hibernate**
- 单独改注册表：
  - Windows 更新
  - OEM 电源策略
  - 可能 **下次又被打开**

### `powercfg /hibernate off` 的好处

- 永久禁用 Fast Startup
- 删除 `hiberfil.sys`
- 行为 **可预测、可重复**
- 对开发机是正收益

------

## 四、运行效果（你该看到什么）

### Fast Startup 开着时：

```text
Fast Startup is ENABLED.
Disabling Fast Startup...
Disabling Hibernate...
Fast Startup has been DISABLED.
```

### 已经关掉：

```text
Fast Startup is already DISABLED.
```

------

## 五、你可以如何用这段脚本（建议）

### 1️⃣ 开发机初始化脚本

放进你常用的：

- `setup.ps1`
- `bootstrap.ps1`
- 或 Ansible / DSC

### 2️⃣ 登录即检查（偏激进）

配合任务计划：

- Trigger：At logon
- Action：`powershell -ExecutionPolicy Bypass -File disable-fast-startup.ps1`

------

