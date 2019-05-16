---
title: 破解mongo客户端Studio3T
tags: 
- 工具
- 破解
categories: 
- 工具
- 破解
---



# 破解mongo客户端Studio3T

[https://github.com/linG5821/Studio3TCrack](https://github.com/linG5821/Studio3TCrack)

从上述网站下载jar包。修改Studio 3T的应用名称，去空格，空格会引起执行脚本时候失败。

![](https://blog.airaccoon.cn/img/bed/20190515/1557908006162.png)

![](https://blog.airaccoon.cn/img/bed/20190515/1557907992480.png)



```shell
# 将下载的jar文件复制到应用jar包目录
cp studio-3t-start-2019-2.1.jar /Applications/Studio3T.app/Contents/Resources/app/
# 进入应用用jar包目录
cd /Applications/Studio3T.app/Contents/Resources/app
# 执行脚本
java -jar -XstartOnFirstThread ./studio-3t-start-2019-2.1.jar
# 执行完成后，会报错，同时自动打开Studio3T应用。不用管错误，新打开的应用就是破解了的。
```