---
title: 老中青三代Macbook对比测评
date: 2019-03-30 16:17:43
tags:
---

## 概述

机型 | 配置| 详细
 -----| ----- | ---
2013Late[^2013Late]| i5(双核), 8G | 2.4GHz-2.9GHz(3M)
2017[^2017Macbook] | i7(四核), 16G | 2.8GHz-3.8GHz(6M)
2018[^2018] | i9(六核), 32G | 2.9GHz-4.8GHz(12MB)

[^2013Late]: [MacBookPro(Retina,13英寸,2013年末)-技术规格](https://support.apple.com/kb/SP691?locale=zh_CN)
[^2017Macbook]: [MacBookPro(15英寸2017)-技术规格](https://support.apple.com/kb/SP756?locale=zh_CN)
[^2018]: [MacBookPro(15英寸2018)-技术规格](https://support.apple.com/kb/SP776?locale=zh_CN)

## 测试项目
1. Xcode编译
2. zip压缩
3. unzip解压

## 需要工具
**Xcode**: Apple平台IDE
**brew**: Mac下包管理工具
**coreutils**: GNU的shell工具集[^coreutils]

[^coreutils]: [解决Mac下date毫秒纳秒为Nil](https://www.dwhd.org/20181225_191707.html)

## xcode编译
### 项目构成
* 总文件数：9501

    ```shell
    $ ls -lR ~/Downloads/BaiheOnlineOtherBlanch | grep '^-' | wc -l
    > 9501
    ```
    
* 实现文件数：2243
    
    ```shell
    $ ls -lR ~/Downloads/BaiheOnlineOtherBlanch | grep -e '.xib$' -e '.m$' | wc -l
    > 2243
    ```
    
* 项目复杂度
    ```shell
    $ lizard ~/Downloads/BaiheOnlineOtherBlanch ~/Desktop
    ```
     CCN | AvgCCN | Avgtoken | Functions
     --- | --- | --- | --- 
      69859 | 2.6 | 83.9 | 26796 

<!-- more -->
### 测试脚本
```shell
#!bin/bash
beginTime=`gdate +'%s.%N'` \
&& \
xcodebuild -workspace ~/Downloads/BaiheOnlineOtherBlanch/BaiHe.xcworkspace -scheme BaiHe clean build 'platform=iOS Simulator,name=iPhone X' -configuration Debug -showBuildTimingSummary \
&& \
endTime=`gdate +'%s.%N'` \
&& \
echo "build total $(($endTime-$beginTime))"
```

## zip/unzip文件
### 测试目录
`～/.cocoapod/repo/master`
### 文件数
```shell
$ ls -lR ~/.cocoapods | grep -e  '^-' | wc -l
> 315554
```

### 测试脚本
#### zip
```shell
#!bin/bash
beginTime=`gdate +'%s.%N'` \
&& \
zip -r ~/demo.zip ~/Desktop/Users \
&& \
endTime=`gdate +'%s.%N'` \
&& \
echo "zip total $(($endTime-$beginTime))"
```
#### unzip
```shell
#!bin/bash
beginTime=`gdate +'%s.%N'` \
&& \
unzip ./demo.zip  \
&& \
endTime=`gdate +'%s.%N'` \
&& \
echo "unzip total $(($endTime-$beginTime))"
```

## 测试数据

### zip
{% echarts 400 '81%' %}
{
    legend: {},
    tooltip: {},
    dataset: {
        source: [
            ['product', '第一次', '第二次'],
            ['2013Late', 350.04, 358.01],
            ['2017', 248.05, 249.07],
            ['2018', 158.41, 154.60],
        ]
    },
    xAxis: {type: 'category'},
    yAxis: {},
    series: [
        {type: 'bar'},
        {type: 'bar'},
    ]
};
{% endecharts %}

### unzip

{% echarts 400 '81%' %}
{
    legend: {},
    tooltip: {},
    dataset: {
        source: [
            ['product', '第一次', '第二次'],
            ['2013Late', 423.47, 409.37],
            ['2017', 208.42, 206.45 ],
            ['2018', 142.92, 131.64],
        ]
    },
    xAxis: {type: 'category'},
    yAxis: {},
    series: [
        {type: 'bar'},
        {type: 'bar'},
    ]
};
{% endecharts %}


### xcodebuild

{% echarts 400 '81%' %}
{
    legend: {},
    tooltip: {},
    dataset: {
        source: [
            ['product', 'CompileC', 'Ld', 'CompileXIB', 'CompileSwiftSources', 'ProcessPCH', 'Total'],
            ['2013Late', 1857.69, 33.49, 282.49, 15.54, 0.1, 650.81],
            ['2017', 1611, 29, 18, 9, 0, 310.98],
            ['2018', 1550, 26, 43, 8, 4, 220.44],
        ]
    },
    xAxis: {type: 'category'},
    yAxis: {},
    series: [
        {type: 'bar'},
        {type: 'bar'},
        {type: 'bar'},
        {type: 'bar'}, 
        {type: 'bar'}, 
        {type: 'bar'},
    ]
};
{% endecharts %}

2018原始数据
```shell
Build Timing Summary
CompileC (3026 tasks) | 1550.000 seconds
CompileXIB (409 tasks) | 43.000 seconds
Ld (1 task) | 26.000 seconds
CompileAssetCatalog (1 task) | 15.000 seconds
CompileSwiftSources (1 task) | 8.000 seconds
CompileStoryboard (1 task) | 4.000 seconds
ProcessPCH (2 tasks) | 4.000 seconds
CodeSign (1 task) | 1.000 seconds
Libtool (92 tasks) | 0.000 seconds
CreateUniversalBinary (46 tasks) | 0.000 seconds
PhaseScriptExecution (3 tasks) | 0.000 seconds
Ditto (3 tasks) | 0.000 seconds
CopyPNGFile (84 tasks) | 0.000 seconds
LinkStoryboards (1 task) | 0.000 seconds
Touch (1 task) | 0.000 seconds
CompileDTraceScript (2 tasks) | 0.000 seconds
** BUILD SUCCEEDED **

build total 220.44037985801697
```

2017原始数据
```shell
Build Timing Summary
CompileC (3026 tasks) | 1611.000 seconds
Ld (1 task) | 29.000 seconds
CompileAssetCatalog (1 task) | 20.000 seconds
CompileXIB (409 tasks) | 18.000 seconds
CompileSwiftSources (1 task) | 9.000 seconds
CodeSign (1 task) | 2.000 seconds
ProcessPCH (2 tasks) | 0.000 seconds
Libtool (92 tasks) | 0.000 seconds
CreateUniversalBinary (46 tasks) | 0.000 seconds
PhaseScriptExecution (3 tasks) | 0.000 seconds
Ditto (3 tasks) | 0.000 seconds
CopyPNGFile (84 tasks) | 0.000 seconds
CompileStoryboard (1 task) | 0.000 seconds
LinkStoryboards (1 task) | 0.000 seconds
Touch (1 task) | 0.000 seconds
CompileDTraceScript (2 tasks) | 0.000 seconds
** BUILD SUCCEEDED ** [306.822 sec]

build total 310.98745083808899
```
2013Late原始数据
```shell
Build Timing Summary
CompileC (3026 tasks) | 1857.696 seconds
CompileXIB (409 tasks) | 282.493 seconds
CompileAssetCatalog (1 task) | 36.626 seconds
Ld (1 task) | 33.495 seconds
CompileSwiftSources (1 task) | 15.548 seconds
CopyPNGFile (84 tasks) | 8.401 seconds
CreateUniversalBinary (46 tasks) | 4.226 seconds
CodeSign (1 task) | 3.272 seconds
Libtool (92 tasks) | 2.889 seconds
PhaseScriptExecution (3 tasks) | 1.131 seconds
CompileStoryboard (1 task) | 1.080 seconds
CompileDTraceScript (2 tasks) | 0.161 seconds
ProcessPCH (2 tasks) | 0.115 seconds
Ditto (3 tasks) | 0.055 seconds
LinkStoryboards (1 task) | 0.029 seconds
Touch (1 task) | 0.009 seconds

** BUILD SUCCEEDED **

build total 650.81101489067078
```



## 参考





