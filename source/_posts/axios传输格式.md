---
title: axios时间传输格式
date: 2022-01-28 19:56:36
tags: Java
---

默认的axios在传输时间时，会导致时间推前8个小时（估计时区的问题）。解决方法如下：

1. 安装时间格式化组件（自己写应该也可以）

   ```shell
   npm install -s moment
   ```

2. 覆盖Date原型方法 toISOString

   ```js
   import moment from "moment"
   Date.prototype.toISOString = function () {
       return moment(this).format('YYYY-MM-DD HH:mm:ss')
   }
   ```
   
3. 后台时间格式解析配置（Java Spring）(可选)

   实体类上的时间属性上添加如下注解

   ```java
   @JsonFormat(shape = JsonFormat.Shape.STRING, pattern="yyyy-MM-dd HH:mm:ss")
   ```

参考资料来源：https://blog.csdn.net/phker/article/details/112062307

