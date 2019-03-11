---
title: fastlane使用指北
date: 2019-01-17 10:14:37
tags: 
- iOS
- devops
---


[toc]

> [fastlane](https://docs.fastlane.tools) is the easiest way to automate beta deployments and releases for your iOS and Android apps. 
> fastlane为你iOS和Android应用的自动测试部署和发布提供一种简洁的方式


### 安装

1. 安装xcode-command-tools(Xcode自带此工具)

```shell
$ xcode-select --install
```
2. 安装fastlane

```shell
# Using RubyGems(推荐)
$ sudo gem install fastlane -NV

# Alternatively using Homebrew
$ brew cask install fastlane
```

### 安装Bundler
> Bundler provides a consistent environment for Ruby projects by tracking and installing the exact gems and versions that are needed.
> Bundler通过跟踪和安装所需的gem和版本，为Ruby项目提供了一致的环境。

```shell
gem install bundler
```

### 初始化fastlane项目

```shell
$ bundler exec fastlane init
//or fastlane init swift
```

#### 目录基本结构

```shell
.
├── Source
├── Foo.xcodeproj
├── Gemfile
├── Gemfile.lock
└─── fastlane
    ├── Appfile
    ├── Fastfile
    ├── Gymfile
    ├── Pluginfile
    ├── README.md
    └── report.xml
```

<!-- more -->
#### 相关文件

1. Gemfile / Pluginfile / Gemfile.lock
ruby gem包配置文件，版本控制文件，安装ruby工具库

2. Appfile
App的基本信息（app_identifier， apple_id），便于全局访问。

3. Fastfile
脚本文件，负责定义脚本方法，方法通过Action、Plugin组合执行相关流程，支持导入

4. Gymfile
Gym指令相关配置文件，配置Gym相关环境变量

### Appfile使用

设置
```ruby
app_identifier "net.sunapps.1"
apple_id "felix@krausefx.com"
team_id "Q2CBPJ58CC"

for_platform :ios do
    team_id '123' # for all iOS related things
        for_lane :test do
        app_identifier 'com.app.test'
    end
end
```

获取
```ruby
identifier = CredentialsManager::AppfileConfig.try_fetch_value(:app_identifier)
team_id = CredentialsManager::AppfileConfig.try_fetch_value(:team_id)
```

### Fastfile使用
脚本文件，负责定义脚本方法


#### 基本使用
[ruby](https://www.ruby-lang.org/zh_cn/)、[Action](https://docs.fastlane.tools/actions/)、[Plugin](https://docs.fastlane.tools/plugins/available-plugins/)使用
##### 定义一个脚本方法

```ruby

#支持ruby
lane :rubyTest do
    puts "Hello"
end

#使用Action
lane :actionTest do
    #执行pod install
    cocoapods
end

#使用Plugin
lane :actionTest do
    #获取info.plist的version_number
    version = versioning
    #打印
    puts version
end

```
##### 执行一个脚本方法

```shell
$ bundler exec fastlane rubyTest
> Hello

$ bundler exec fastlane actionTest
```

#### 导入功能

```ruby
import "../GeneralFastfile"

override_lane :from_general do
# ...
end

import_from_git(url: 'https://github.com/fastlane/fastlane')
# or
import_from_git(url: 'git@github.com:MyAwesomeRepo/MyAwesomeFastlaneStandardSetup.git',
path: 'fastlane/Fastfile')

```


### FAQ
