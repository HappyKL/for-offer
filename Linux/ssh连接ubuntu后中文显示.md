# ssh连接ubuntu后中文显示？？？



## 描述

![image-20191119194000798](%E5%AE%89%E8%A3%85%E7%9A%84ubuntu%E9%80%9A%E8%BF%87ssh%E8%BF%9E%E6%8E%A5%E5%90%8E%E4%B8%AD%E6%96%87%E6%98%BE%E7%A4%BA.assets/image-20191119194000798.png)





## 解决

```shell
vim /etc/environment

LANG=zh_CN.UTF-8
LANGUAGE=en_US:en
LC_CTYPE="zh_CN.UTF-8"
LC_NUMERIC="zh_CN.UTF-8"
LC_TIME="zh_CN.UTF-8"
LC_COLLATE="zh_CN.UTF-8"
LC_MONETARY="zh_CN.UTF-8"
LC_MESSAGES="zh_CN.UTF-8"
LC_PAPER="zh_CN.UTF-8"
LC_NAME="zh_CN.UTF-8"
LC_ADDRESS="zh_CN.UTF-8"
LC_TELEPHONE="zh_CN.UTF-8"
LC_MEASUREMENT="zh_CN.UTF-8"
LC_IDENTIFICATION="zh_CN.UTF-8"
LC_ALL=zh_CN.UTF-8

source /etc/environment
```

![image-20191122142545738](ssh%E8%BF%9E%E6%8E%A5ubuntu%E5%90%8E%E4%B8%AD%E6%96%87%E6%98%BE%E7%A4%BA%EF%BC%9F%EF%BC%9F.assets/image-20191122142545738.png)

