---
title: react-native run-ios 踩坑记录
date: 2019-08-23 01:08:55
tags:
---


> React-Native mac 运行报错 error Failed to build iOS project. We ran "xcodebuild" command but it exited with error code 65. 

原因：第三方依赖包下载太慢。

解决办法：手动下载。

- 查看第三方依赖包

```
cat node_modules/react-native/scripts/ios-install-third-party.sh
```

- 下载
https://github.com/google/glog/archive/v0.3.5.tar.gz
https://github.com/google/double-conversion/archive/v1.1.6.tar.gz
https://github.com/react-native-community/boost-for-react-native/releases/download/v1.63.0-0/boost_1_63_0.tar.gz
https://github.com/facebook/folly/archive/v2018.10.22.00.tar.gz

- 新建文件夹 .rncache，将下载的四个文件移入到目录下
```
mkdir ~/.rncache
```

- 可有可无的一步，手动运行脚本，因为下面run-ios的时候回解压四个压缩包到指定目录，无需手动执行
```
node_modules/react-native/scripts/ios-install-third-party.sh
```

- 重新安装npm依赖，运行

```
rm -rf node_modules 
npm cache clean --force

npm install 

react-native run-ios
```

> react native undefined is not an object RNGestureHandlerModule.State

```
rm -rf node_modules
rm -rf package_lock.json
npm install
npm install --save react-navigation               //这两步install可以不做，如果package.json里有的话
npm install --save react-native-gesture-handler   //这两步install可以不做，如果package.json里有的话
react-native link
```



> Error: Unable to resolve module `./DrawerLayout` from `/Users/xinyingshi/Projects/github-demo/MeiTuan/node_modules/react-navigation-drawer/dist/views/DrawerView.js`: The module `./DrawerLayout` could not be found from `/Users/xinyingshi/Projects/github-demo/MeiTuan/node_modules/react-navigation-drawer/dist/views/DrawerView.js`.


https://github.com/react-navigation/react-navigation/issues/5564




