---
name: system-diagnostics
description: 诊断Windows系统问题，包括崩溃、重启、性能问题等
---

# 系统诊断技能

## 目的
诊断Windows系统问题，分析崩溃原因，提供解决方案。

## 适用场景
- 电脑突然重启或崩溃
- 游戏或应用程序导致系统不稳定
- 硬件温度过高
- 驱动程序问题
- 系统性能问题

## 工作流程

### 1. 收集信息
- 询问问题发生的具体情况
- 了解问题发生的时间、频率
- 获取系统基本信息

### 2. 诊断检查
- 检查系统事件日志
- 检查崩溃转储文件
- 检查硬件状态
- 检查驱动程序状态

### 3. 分析原因
- 分析崩溃原因
- 识别问题模式
- 评估严重程度

### 4. 提供解决方案
- 提供具体解决步骤
- 解释每个步骤的作用
- 提供预防措施

## 诊断命令模板

### 系统信息检查
```powershell
# 获取系统信息
systeminfo

# 获取CPU信息
Get-WmiObject Win32_Processor | Select-Object Name, NumberOfCores

# 获取内存信息
Get-WmiObject Win32_PhysicalMemory | Select-Object Manufacturer, Capacity, Speed
```

### 事件日志检查
```powershell
# 检查系统事件日志
Get-WinEvent -FilterHashtable @{LogName='System'; Level=1,2} -MaxEvents 50

# 检查应用程序事件日志
Get-WinEvent -FilterHashtable @{LogName='Application'; Level=1,2} -MaxEvents 50

# 检查崩溃事件（Event ID 41）
Get-WinEvent -FilterHashtable @{LogName='System'; ID=41} -MaxEvents 10
```

### 硬件状态检查
```powershell
# 检查CPU温度
Get-WmiObject MSAcpi_ThermalZoneTemperature -Namespace 'root/wmi'

# 检查硬盘健康状态
Get-PhysicalDisk | Select-Object FriendlyName, MediaType, HealthStatus

# 检查GPU温度和功耗
nvidia-smi --query-gpu=temperature.gpu,power.draw,utilization.gpu --format=csv
```

### 崩溃转储分析
```powershell
# 检查崩溃转储文件
Get-ChildItem C:\Windows\Minidump\* -ErrorAction SilentlyContinue | Sort-Object LastWriteTime -Descending

# 检查事件查看器中的崩溃记录
Get-WinEvent -FilterHashtable @{LogName='System'; Id=1001} -MaxEvents 10
```

## 常见问题诊断

### 1. 游戏导致重启
```powershell
# 检查崩溃前的事件
Get-WinEvent -FilterHashtable @{LogName='System'; StartTime=(Get-Date).AddHours(-1)} -MaxEvents 50

# 检查GPU温度
nvidia-smi --query-gpu=temperature.gpu,power.draw --format=csv

# 检查电源事件
Get-WinEvent -FilterHashtable @{LogName='System'; Id=41} -MaxEvents 5
```

### 2. 驱动程序问题
```powershell
# 检查有问题的驱动程序
Get-WmiObject Win32_PnPEntity | Where-Object {$_.ConfigManagerErrorCode -ne 0}

# 检查驱动程序版本
Get-WmiObject Win32_PnPSignedDriver | Select-Object DeviceName, DriverVersion
```

### 3. 硬件故障
```powershell
# 检查硬盘SMART状态
Get-PhysicalDisk | Get-StorageReliabilityCounter | Select-Object Temperature, Wear

# 检查内存错误
Get-WinEvent -FilterHashtable @{LogName='System'; ProviderName='Memory'} -MaxEvents 10
```

## 解决方案模板

### 游戏崩溃解决步骤
1. **更新显卡驱动程序**
   - 访问NVIDIA/AMD官网下载最新驱动
   - 执行清洁安装

2. **检查游戏设置**
   - 降低图形设置
   - 关闭垂直同步
   - 限制帧率

3. **监控硬件温度**
   - 使用HWMonitor或MSI Afterburner
   - 确保温度在安全范围内

4. **验证游戏文件**
   - 使用Steam/Origin验证游戏文件完整性
   - 重新安装游戏

### 系统不稳定解决步骤
1. **运行系统文件检查器**
   ```powershell
   sfc /scannow
   ```

2. **检查磁盘错误**
   ```powershell
   chkdsk C: /f /r
   ```

3. **更新Windows**
   - 安装所有Windows更新
   - 更新所有驱动程序

4. **检查恶意软件**
   - 运行Windows Defender全面扫描
   - 使用Malwarebytes进行二次扫描

## 注意事项
- 始终以管理员权限运行诊断命令
- 在修改系统设置前创建还原点
- 记录所有诊断结果以便跟踪
- 提供清晰的步骤说明

## 示例交互

**用户**: 我的电脑在玩游戏时突然重启了
**助手**: 我来帮你诊断这个问题。首先检查系统事件日志：
```powershell
Get-WinEvent -FilterHashtable @{LogName='System'; ID=41} -MaxEvents 5
```
然后检查GPU温度：
```powershell
nvidia-smi --query-gpu=temperature.gpu,power.draw --format=csv
```
根据结果，我会提供具体的解决方案。