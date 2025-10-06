---
title: 解决在升级 dotnet 9 之后 某些 P/Invoke 代码异常的问题
date: 2025-10-06
categories:
- .NET
tags:
- P/Invoke
---

最近在使用第三方的某sdk的时候，发现示例程序一切正常，但是同样的代码在 .net9 上就直接闪退

```C#
[DllImport("SoftLic64.dll",CallingConvention=CallingConvention.StdCall,CharSet=CharSet.Ansi)]
public static extern string ks_cmd(string cmdType, string cmdData);

```

在 x64dbg 里面显示的是 STATUS_STACK_BUFFER_OVERRUN ，不过我看不懂汇编，也没法分析和正常的调用有什么差异
```
INT3 断点 "DllMain (softlic64.dll)" 于 <softlic64.OptionalHeader.AddressOfEntryPoint> (00007FFBC51474F0) ！
EXCEPTION_DEBUG_INFO:
           dwFirstChance: 0
           ExceptionCode: C0000409 (STATUS_STACK_BUFFER_OVERRUN)
          ExceptionFlags: 00000001
        ExceptionAddress: kernelbase.00007FFD2E8D8711
        NumberParameters: 3
ExceptionInformation[00]: 0000000000000039
ExceptionInformation[01]: 00000086AF8FEE30
ExceptionInformation[02]: 0000000040024000
第二次异常于 00007FFD2E8D8711 (C0000409, STATUS_STACK_BUFFER_OVERRUN)！
```
起初以为是 dll 只支持 framework, 但是在改为 net8.0 之后代码又能正常运行了。
但是去 dotnet 官网看了一圈也没有发现什么端倪，于是就想去 issue 里搜关键词 `STATUS_STACK_BUFFER_OVERRUN` 碰碰运气 

结果还真发现了解决方法 [NET 9.0 RC2 NativeLibrary.Load crashes #107993](https://github.com/dotnet/runtime/issues/107993)

原来 .net9 的 Interop 默认启用了 CET 支持： https://learn.microsoft.com/zh-cn/dotnet/core/compatibility/interop/9.0/cet-support

解决方法是在项目文件里面禁用 CET 即可。
```xml
<CETCompat>false</CETCompat>
```
