---
title: SonarQube使用指东
date: 2019-01-28 16:17:45
tags:
- iOS
- DevOps
- CodeQuality
---
[toc]

### 概述
SonarQube是一个开源的代码质量管理系统。提供重复代码、编码标准、单元测试、代码覆盖率、代码复杂度、潜在Bug、注释和软件设计报告。基于Java开发。

#### 结构
SonarQube 平台由4个部分组成:
![20190127154852349584744.png](https://obect.yunlongzhu.com/20190127154852349584744.png)
1. SonarQubeServer
查看质量检测快照和配置SonarQube实例，提供搜索结果给可视化界面，保存到数据库
2. SonarQubeDatabase
存储 Sonarqube 的配置，存储项目、视图等的质量快照
3. Multiple SonarQube Plugins
本地化、 SCM、CI、身份验证等插件
4. One or more SonarScanners
用来分析项目代码并上报信息给 sonarqube
<!-- more -->

#### 工作流程
![20190127154852374274444.png](https://obect.yunlongzhu.com/20190127154852374274444.png)
1. 开发人员编写代码并提交到SCM
2. 触发自动构建，代码分析，SonarScanner执行扫描分析
3. SonarScanner将分析结果发送给SonarQubeServer
4. SonarqubeServer处理后将分析报告结果存储并展示
5. 开发人员通过SonarQubeUI界面查看结果

### 安装

#### SonarQubeServer
[SonarQubeServer](https://docs.sonarqube.org/latest/setup/install-server/)由于需要配置Java、数据库等环境，而且不参与代码分析工作，所以推荐使用Docker安装。
```Dockerfile
version: '3'
services:
    sonarqube:
        image: sonarqube:lts
        ports:
        - "9000:9000"
        - "9002:9002"
    networks:
        - sonarnet
    environment:
        - SONARQUBE_JDBC_URL=jdbc:postgresql://db:5432/sonar
        - SONARQUBE_JDBC_USERNAME=sonar
        - SONARQUBE_JDBC_PASSWORD=sonar@123
    volumes:
        - sonarqube_conf:/opt/sonarqube/conf
        - sonarqube_data:/opt/sonarqube/data
        - sonarqube_extensions:/opt/sonarqube/extensions
        - sonarqube_bundled-plugins:/opt/sonarqube/lib/bundled-plugins

db:
    image: postgres
    ports:
        - "5432:5432"
    networks:
        - sonarnet
    environment:
        - POSTGRES_USER=sonar
        - POSTGRES_PASSWORD=sonar@123
volumes:
    - postgresql_data:/var/lib/postgresql/data

networks:
    sonarnet:
        driver: bridge

volumes:
    sonarqube_conf:
    sonarqube_data:
    sonarqube_extensions:
    sonarqube_bundled-plugins:
    postgresql_data:
```
部署
```Shell
$ docker-compose up -d
```

#### 非官方Plugins
sonar的oc，swift官方分析插件目前仅针对非社区版用户，不过也有非官方版的插件[sonar-swift](https://github.com/Backelite/sonar-swift)可供使用。
`sonar-swift在7.x版本上存在兼容性问题，因此sonarqube推荐7.x以下(即lts版本)。`
Plugins通常安装在/opt/sonarqube/extensions/plugins下，将插件jar包，拷贝到相应位置即可，重启sonarqube。
```Shell
$ docker cp ./backelite-sonar-swift-plugin-0.4.2.jar  sonarqube_sonarqube_1:/opt/sonarqube/extensions/plugins
```

#### SonarScan
[SonarScan](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner)推荐使用Homebrew安装。
```Shell
$ brew install sonar-scanner
```

### 配置代码分析流程
代码分析分为6个指标：Complexity(复杂度)，Duplications(重复性)，Issues(存在问题)，Size(大小)，Tests(用例)，和Code coverage(代码覆盖率)

各个指标对应工具，空白表示插件自己处理

| Feature         | Supported    | MacOS  |      
|---------------|-----------|-----------|
| Complexity    |YES        |Uses [Lizard](https://github.com/terryyin/lizard)| 
| Duplications    |YES        |              | 
| Issues        |YES        | Uses [SwiftLint](https://github.com/realm/SwiftLint) and/or [Tailor](https://github.com/sleekbyte/tailor) for Swift. [OCLint](http://oclint-docs.readthedocs.io/en/stable/) and [Faux Pas](http://fauxpasapp.com/) for Objective-C|
| Size            |YES        |              |  
| Tests            |YES        | Uses xcodebuild + xcpretty [xcpretty](https://github.com/supermarin/xcpretty)    |
| Code coverage    |YES        | Uses [slather](https://github.com/venmo/slather)   | 

#### [Lizard](https://github.com/terryyin/lizard)
![20190127154857342460629.png](https://obect.yunlongzhu.com/20190127154857342460629.png)
> Lizard is an extensible Cyclomatic Complexity Analyzer for many imperative programming languages including C/C++ (doesn't require all the header files or Java imports)
> Lizard是一种可扩展的复杂度分析器，适用于多种编程语言，包括C/C++ (不需要所有头文件或Java导入)

1. 安装
```shell
$ [sudo] pip install lizard
#OR
$ sudo pip install lizard --install-option="--install-scripts=/usr/local/bin"
```

2. 安装fastlane-plugin-lizard
```shell
$ sudo gem install fastlane-plugin-lizard -n /usr/local/bin
#OR
$ fastlane add_plugin lizard
```

3. 使用
```shell
lizard mySource/ --xm lizard-report.xml
```
对应指标
![20190127154857593056435.png](https://obect.yunlongzhu.com/20190127154857593056435.png)

#### [xcpretty](https://github.com/xcpretty/xcpretty)
![20190127154857627392726.png](https://obect.yunlongzhu.com/20190127154857627392726.png)
> xcpretty is a fast and flexible formatter for xcodebuild
> xcpretty是为xcodebuild提供快速、可扩展的格式化工具

1. [安装](https://github.com/Backelite/sonar-swift)
原版xcpretty在SonarQube存在问题，因此需要安装特别修复的版本
```Shell
git clone https://github.com/Backelite/xcpretty.git
cd xcpretty
git checkout fix/duration_of_failed_tests_workaround
gem build xcpretty.gemspec
sudo gem install --both xcpretty-0.2.2.gem
```
2. 直接使用
```Shell
$ set -o pipefail && xcodebuild [flags] | xcpretty
```
3. 配合fastlane使用
```Shell
# Scanfile

#项目配置
scheme("BHIMMessageUIPodTests")
workspace("./BHIMMessageUIPod.xcworkspace")
#设备
devices(["iPhone X"])
#构建配置
clean(true)
build_for_testing(true)
code_coverage(true)
#导出配置
output_types("junit,json-compilation-database")
output_directory("./sonar-reports")


# 执行fastlane
fastlane run scan
```

#### [slather](https://github.com/SlatherOrg/slather)
![20190128154860707587252.png](https://obect.yunlongzhu.com/20190128154860707587252.png)

> Generate test coverage reports for Xcode projects & hook it into CI.
> 为Xcode projects生成测试覆盖率报告，将其连接到CI中。

1. 安装
```Shell
$ sudo gem install slather -n /usr/local/bin
```
2. 使用
->开启测试覆盖率功能，scheme->edit->Test->Options->勾选"Gather coverage data"
![20190128154860739640041.png](https://obect.yunlongzhu.com/20190128154860739640041.png)
->使用
```Shell
$ slather coverage -s --scheme YourXcodeSchemeName path/to/project.xcodeproj

#OR
$ slather coverage -s --scheme YourXcodeSchemeName --workspace path/to/workspace.xcworkspace path/to/project.xcodeproj
```
3. 配置文件，slather支持读取.slather.yml下的配置文件
```YML
# .slather.yml
coverage_service: cobertura_xml
workspace: BHIMMessageUIPod.xcworkspace
xcodeproj: BHIMMessageUIPod.xcodeproj
scheme: BHIMMessageUIPodTests
configuration: Debug
source_directory: ./BHIMMessageUIPod/
output_directory: ./sonar-reports/
```
4. fastlane
```Ruby
slather
sh("mv", "../sonar-reports/cobertura.xml", "../sonar-reports/coverage_slather.xml")
```
5. 相关指标
![2019012815486085332967.png](https://obect.yunlongzhu.com/2019012815486085332967.png)

#### [OCLint](http://docs.oclint.org)
> OCLint is a static code analysis tool for improving quality and reducing defects by inspecting C, C++ and Objective-C code and looking for potential problems like possible bugs, unused code, complicated code, redundant code, code smells, bad practices, and so on.
> OCLint是一种静态代码分析工具，通过检查C、C++和Objective-C代码，查找潜在的问题，如可能的错误、未使用的代码、复杂的代码、冗余代码、代码气味、不良做法等，来提高质量和减少缺陷。

1. 安装
```Shell
$ brew tap oclint/formulae
$ brew install oclint
```
2. [配置(YML)](http://docs.oclint.org/en/stable/howto/rcfile.html)
oclint将依次加载(/etc/oclint, ~/.oclint and .oclint)中的配置文件。[规则](http://docs.oclint.org/en/stable/rules/index.html)
```YML
disable-rules:
    - LongLine
rulePaths:
    - /etc/rules
rule-configurations:
    - key: CYCLOMATIC_COMPLEXITY
value: 15
    - key: NPATH_COMPLEXITY
value: 300
output: oclint.xml
report-type: xml
max-priority-1: 20
max-priority-2: 40
max-priority-3: 60
enable-clang-static-analyzer: false
```

### 参考
[基于 SonarQube 的可视化代码质量检查](https://kongsansan.com/2019/01/23/SonarQube_on_macos/)
[Continuous Code Quality Inspection with SonarQube](https://www.red-gate.com/simple-talk/dotnet/net-development/continuous-code-quality-inspection-sonarqube/)
[Mobile App Development and Continuous Delivery – SonarQube (2/7)](https://techblog.xavient.com/mobile-app-development-and-continuous-delivery-27/)
[Code Coverage Reports in SonarQube for Swift on macOS](https://medium.com/@agavatar/code-coverage-reports-in-sonarqube-for-swift-on-macos-49797b6a8fea)
[iOS Project analyze 1 use sonarqube server docker image and sonarqube scanner](https://medium.com/@n913239/ios-project-analyze-1-use-sonarqube-server-docker-image-and-sonarqube-scanner-e148b2978364)
