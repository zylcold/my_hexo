---
title: 编写Fastlane插件
date: 2019-03-11 10:21:14
tags:
- iOS
- Fastlane
---

## 概述 
Fastlane将一系列操作封装成lane，然后通过fastlane run [laneName]执行这些操作。

* 对于存在于Fastlane内的lane，Fastlane称之为**action**
* 对于发布在Gem或者远程的lane，Fastlane称之为**plugin** 
- - - -
## 前置依赖 
ruby 
gem
bundle 
fastlene 

## 相关指令
* **new_plugin** 创建一个plugin
* **add_plugin** 添加plugin到fastlane
* **update_plugins** 更新plugin
* **install_plugins** 从当前项目安装plugin
* **run** 运行制定的action
* **action** 列出指定action所有的选项

<!-- more -->

## 创建👨‍🔧

```shell
fastlane new_plugin [plugin_name] 
```
根据提示填写plugin信息后，fastlane便会在当前目录创建好plugin的目录 ~fastlane-plugin-[plugin_name]~

本文以**demo**项目为例
```shell
fastlane new_plugin demo
cd fastlane-plugin-demo
```
**目录结构** 

```shell
$ tree
. 
├── Gemfile 
├── LICENSE 
├── README.md 
├── Rakefile 
├── fastlane 
│   ├── Fastfile 
│   └── Pluginfile 
├── fastlane-plugin-demo.gemspec 
├── lib 
│   └── fastlane 
│   └── plugin 
│   ├── demo 
│   │   ├── actions 
│   │   │   └── demo_action.rb 
│   │   ├── helper 
│   │   │   └── demo_helper.rb 
│   │   └── version.rb 
│   └── demo.rb 
└── spec 
├── demo_action_spec.rb 
└── spec_helper.rb

```

**lib** 用于存放plugin的源代码 
**spec** 用于编写plugin的测试用例 
**fastlane-plugin-demo.gemspec** 是plugin的安装的配置文件 

## 编写✍️
在lib文件目录中，只需要关心两个文件**version.rb**和**plugin_name_action.rb** 
plugin中需要的相关依赖需要在**fastlane-plugin-demo.gemspe**中配置(类似CocoaPod中的spec)

~version.rb~ 里用来指明plugin的版本。 
~plugin_name_action.rb~ 用来写lane的相关实现

```ruby
def self.run(params) 
	UI.message("The qrcode plugin is working!") 
	UI.message("Hello, YunLong") 
end 
```

其中plugin_name_action.rb中有几个方法需要注意：

| func | 说明 | 
| --- |--- | 
|self.run | plugin从这开始执行| 
|self.return_type | 配置plugin执行后的返回值类型 | 
|self.return_value | 配置plugin执行后的返回值 | 
|self.available_options |配置plugin可用参数 | 
|self.is_supported? | 配置plugin支持平台 | 

## 测试👨‍💻
1. Bundle根据Gemfile的配置文件安装相关依赖 
2. fastlane执行Fastfile中的test路径

```shell
#1.安装依赖 
bundle install 
#2.执行 
bundle exec fastlane test 
```
 
```shell
[18:06:09]: The demo plugin is working! 
[18:06:09]: Hello, YunLong 
```


## 加载 📲
当前根目录
`bundle exec fastlane add_plugin demo`
fastlane  会默认从当前目录加载plugin

## 执行 🛠
`bundle exec  fastlane run demo`

```shell
[18:06:09]: The demo plugin is working! 
[18:06:09]: Hello, YunLong 
```

## 发布 🚀
两种发布方式：
1. 发布在Gems**推荐**
```ruby
# 发布
bundle install 
rake install 
rake release 

# 引用，Pluginfile
gem 'astlane-plugin-demo', "~>1.2"
```
3. 发布在远程仓库
```ruby

# 引用，Pluginfile
gem 'astlane-plugin-demo', git: "https://github/yunlongz/demo.git"
```
 
