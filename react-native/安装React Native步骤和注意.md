# React Native 安装步骤 和注意点！
---

title: "React Native 安装步骤 和注意点"
subtitle: " React Native 安装步骤 和注意点"
date: 2016-6-2 9:07:00
author: " zhengzeqin "

---

React Native项目[github地址:https://github.com/facebook/react-native]

React Native项目官网文档[http://facebook.github.io/react-native/docs/getting-started.html]

一.Homebrew安装  
1.安装终端输入:ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"[http://brew.sh/]
2.进行检查brew是否已经安装成功: brew -v 

二.Node.js安装

1.安装curl : brew install curl 
2.安装nvm:curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.31.1/install.sh | bash

重点：注意修改环境配置：
在.nvm文件目录下找到（或者新建）.bash_profile 文件  添加下面
export NVM_DIR="/Users/zhengzeqin/.nvm" //文件目录
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh" # This loads nvm

3.安装Node.js:nvm install node && nvm alias default node
【注意】如果现在采用是Node5.0版本的版本，官网是推荐安装npm 2，该版本比npm 3速度更加快。在安装完Node之后，命令行运行npm install  -g npm@2安装即可。

4.安装watchmam，该用于监控bug文件:brew install watchman


5.安装flow,flow是一个 JavaScript 的静态类型检查器，建议安装它，以方便找出代码中可能存在的类型错误，官网:http://www.flowtype.org/

三.React Native安装
npm install -g react-native-cli


最后使用命令生成工程 react-native init AwesomeProject

安装react 项目工程
react-native init AwesomeProject  
clone github 开源项目一般使用 npm install 安装项目
显示安装进度
npm config set loglevel=http
查看版本
npm -v

