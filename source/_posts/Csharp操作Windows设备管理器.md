---
title: C#操作Windows设备管理器
date: 2018-11-05 21:24:08
categories:
- C#
tags:
- C#
---

> Windows Management Instrumentation (WMI) 
> C# 代码操作 Windows10 设备管理器，需要引用 System.Management 动态库，其中主要涉及到`ManagementObjectSearcherClass`和`ManagementObject`，前者用于检索管理对象的集合，后则表示WMI实例。

# System.Management Namespace

提供对一组丰富的管理信息和管理事件（它们是关于符合 Windows Management Instrumentation (WMI) 基础结构的系统、设备和应用程序的）的访问。应用程序和服务可以使用从[ManagementObjectSearcher](https://docs.microsoft.com/zh-cn/dotnet/api/system.management.managementobjectsearcher?view=netframework-4.7.2)和[ManagementQuery](https://docs.microsoft.com/zh-cn/dotnet/api/system.management.managementquery?view=netframework-4.7.2)派生的类，查询感兴趣的管理信息（例如在磁盘上还剩多少可用空间、当前 CPU 利用率是多少、某一应用程序正连接到哪一数据库等等）；或者应用程序和服务可以使用[ManagementEventWatcher](https://docs.microsoft.com/zh-cn/dotnet/api/system.management.managementeventwatcher?view=netframework-4.7.2)类预订各种管理事件。这些可访问的数据可以来自分布式环境中托管的和非托管的组件。

参考连接：https://docs.microsoft.com/zh-cn/dotnet/api/system.management?view=netframework-4.7.2

# 示例代码

引用`System.Management`

获取所有设备

```csharp
 public void GetDeviceAll()
 {
   string wmi_ql = "SELECT * FROM Win32_PnPEntity";
   ManagementObjectSearcher searcher = new ManagementObjectSearcher(wmi_ql);
   foreach (ManagementObject mgt in searcher.Get())
   {
     Console.WriteLine(Convert.ToString(service["Name"]));

   }
 }
```

获取指定设备返回 `ManagementObject`

```csharp
        public ManagementObject GetDeviceByName(string deviceName)
        {
            string wmi_ql= "SELECT * From Win32_PnPEntity";
            ManagementObjectSearcher searcher = new ManagementObjectSearcher(wmi_ql);
            ManagementObjectCollection collection = searcher.Get();

            foreach (ManagementObject manage in collection)
            {
                if (manage["Name"].ToString() == deviceName)
                {
                    return manage;
                }
            }
            return null;
        }
```

禁用/启用设备

```csharp
//禁用设备
public bool Disable(ManagementObject mb_device)
{
  try
  {
    mb_device.InvokeMethod("Disable", null);
    return true;
  }
  catch
  {
    return false;
  }
}

 //启用设备
 public bool Enable(ManagementObject mb_device)
 {
   try
   {
     mb_device.InvokeMethod("Enable", null);
     return true;
   }
   catch
   {
     return false;
   }

 }
```

# devcon 命令行操作设备管理器

官网文档：https://docs.microsoft.com/zh-cn/windows-hardware/drivers/devtest/devcon-classes

部分指令可能要提升到管理员权限才能成功。

```powershell
devcon classes //列出所有设备类，包括系统上不使用的设备类
devcon listclass printers //列出指定设备类下的所有设备
devcon disable =printer //禁用打印机
devcon enable =printer //启用打印机
```
