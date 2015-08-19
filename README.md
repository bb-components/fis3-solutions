fis3 解决方案规范
=======================

目前基于 fis 的解决方案越来越多，以后还会有更多，所以希望对各种解决方案有一个统一的说明。

编写此规范主要有以下几个出发点：

* 让初学者全面地认识什么是 fis 解决方案。
* 如果需要自定义解决方案，此文档可以用来作为有效的参考资料。
* 以此规范作为标准，统一各个解决方案，让使用者根据后端不同的技术选型能快速地适应到其他解决方案中。

**如果有任何问题或建议，请提交 issues**

## 什么是解决方案？

解决方案是一个针对特定后端和特定模板引擎，包含：模板语法糖、线下调试、模块化开发、组件化开发、目录规范、后端运行时框架以及子站点拆分等一系列有助于提高前端开发效率的各种方案集合。

### 模板语法糖

### 线下调试

#### 页面预览

一个完整的解决方案，应当集成一个简单的服务器，支持页面的简单预览，而无需**等待**后端开发人员提供环境支持，达到并行开发的效果。

* 对于静态的 html 页面直接预览即可。
* 对于动态模板页面可以结合用户提供假数据自动与模板变量绑定完成预览。

预览规则为 `http://ip:port/path/in/project`, 即页面文件在项目中的路径。

#### 页面重定向

该简单的 server 除了能够直接页面预览之外，还需支持 url rewrite 功能，用来模拟线上访问。如：当在调试服务器上访问`http://ip:port/user`的时候，可以重定向到 `http://ip:port/page/user/index.tpl`

用户可以通过项目根目录的 `server.conf` 来配置重定向规则。

示例：

```ini
# 重定向 / => /page/home.tpl
rewrite \/$ /page/home.tpl

# 重定向用户查看页面
# /user/1 => /page/user/view?id=1
rewrite ^\/user\/(\d*)$ /page/user/view.tpl?id=$1

# redirect /jump /page/about.tpl
redirect \/jump /page/about.tpl
```

语法为

```
指令名称 匹配规则（用来匹配原始请求地址） 目标地址 
```

* `指令` 应当至少支持以下两种指令：

  1. `rewrite` 重定向页面，浏览器地址栏不会发生变化。
  2. `redirect` 跳转页面，浏览器地址栏发生变化。
* `匹配规则` 统一使用正则来配置，应当支持分组。如：`\/user\/(\d+)$`。
* `目标地址` 可以是服务器内任意资源路径或者访问路径，可以通过 `$数字` 来获取正则规则中分组的捕获。

#### 数据 mock

数据 mock 分为两块。

1. **模板数据**

  对于动态的模板页面，需要支持结合用户提供的假数据完成简单预览功能。

  假数据应该支持两种形式：

  1. 静态 json 数据，以 xxx.json 文件提供。数据内容用 json 格式存放。

    ```json
    {
      "title": "用户列表",
      "desc": "页面描述"
    }
    ```
  2. 根据后端选型，通过一种特定脚本支持动态数据。如：`xxx.jsp` 或者 `xxx.php`。

    ```php
    <?php

    // 支持动态逻辑，甚至去线上拉去真实数据。

    return array(
      "timestamp" => time(),
    );
    ```

  模板页面中模板数据应当根据`假数据`存放规范自动加载相应的`假数据`文件，并完成绑定。

  假定`假数据文件`全部存放在 `mock` 文件夹下面，页面文件全部存放在 `page` 目录下面，那么当访问 `page/a/b/c.tpl` 页面时，应当按以下顺序尝试加载 `假数据`，并所有数据合并起来。

  * /mock/page.json
  * /mock/page/a.json
  * /mock/page/a/b.json
  * /mock/page/a/b/c.json

  动态`假数据`文件应当也有同样的加载策略。
2. **ajax 数据**

  可以结合 url rewrite 和静态 json 文件，完全模拟异步 ajax 数据。

  如：/mock/ajax/user/list.json

  ```json
  {
    "data" => [
      {
        "id": 1,
        "name": "foo"
      }
    ],
    "status": 0,
    "message": "ok"
  }
  ```

  /server.conf

  ```
  rewrite ^\/user\/list /mock/ajax/user/list.json
  ```

  当用户请求 `http://ip:port/user/list` 时，返回的是 list.json 中的 json 静态数据。

  除了 url rewrite 和 静态 json 文件结合外，还需支持 url rewrite 和动态脚本结合，满足动态数据模拟的需求。

  ```php
  {
    "data": <?php echo time();?>,
    "status": 0,
    "message": "ok"
  }
  ```

  当然如果是动态脚本，返回的数据类型可以由脚本编写者定，可以是 `xml` 也可以是 `jsonp` 等等。

### 前端目录规范

### 模块化开发

## 现有解决方案集合
