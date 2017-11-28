# 前端镜像

---

### `ini`配置文件

* `allow_url_fopen`开启,`composer`需要
* `open_basedir`因为性能问题不使用
* `post_max_size`设置`post`传输限制
* `date.timezone`设定时区
* `memory_limit`设定内存限制
* `file_uploads = off`上传文件功能关闭
* `upload_tmp_dir`虽然上传文件功能关闭, 还是把上传文件路径指向到别的位置
* `display_errors = off`不输出错误
* `error_reporting = E_ALL`错误级别,设定到`notice`
* `log_errors = on`开启记录错误日志, 因为默认配置文件会把错误输出指向到标准错误输出
* `expose_php = off`不返回`php版本号`
* `html_errors = off`关闭错误引导链接
* 修改`session`存储设置
	* `session.save_hander = memcached;`
	* `session.save_path = tcp://memcached-1:11211,tcp://memcached-2:11211`

##### `open_basedir`不使用

open_basedir开启后会影响I/O，因为每个调用的文件都需要判断是否在限制目录内.

使用open_basedir可以限制程序可操作的目录和文件,提高系统安全性.但会影响I/O性能导致系统执行变慢,因此需要根据具体需求,在安全与性能上做平衡.

##### 添加禁用函数

[禁用函数列表](./disableFunctions.md)

* 禁止修改文件权限
	* `chmod`改变文件模式.
* 禁止修改文件的属主或数组
	* `chgrp`改变文件所属的组.
	* `chown`改变文件的所有者.
* 修改工作路径.
	* `chroot`改变当前进程的根目录.
* 可以执行服务端命令
	* `passthru`执行外部程序并且显示原始输出
	* `exec`
	* `system`
	* `shell_exec`
	* `popen`
	* `proc_open`
* 运行时修改
	* `dl`运行时载入一个`PHP`扩展
	* `ini_set`可用于修改,设置 PHP 环境配置参数
	* `ini_alert`是`ini_set()`函数的一个别名函数,功能与`ini_set()`相同
	* `ini_restore`可用于恢复`PHP`环境配置参数到其初始值
* 获取系统信息函数.
	* `disk_total_space`获取总的硬盘大小
	* `disk_free_space`获取可用硬盘大小
	* `diskfreespace`获取可用硬盘大小
	* `phpinfo`获取`php`信息


### `phpfpm`配置文件

* 修改`pm.max_children/`从`5`到`100`.
* 修改`pm.start_servers`从`2`到`40`.
* 修改`pm.min_spare_servers`从`1`到`20`. 
* 修改`pm.max_spare_servers`从`3`到`60`.
* 开启慢日志`slowlog`,并指向`标准错误输入`.
* 设定慢日志的`php`执行时间标准`request_slowlog_timeout`为`5`秒.
* 开启日志功能, 并记录从`nginx`生成的`requestId`

#### 修改日志格式

使用`json`, 方便日志收集工作. 存储在`mongo`或者在`es`可以方便的搜索.

```shell
'{"request_id":"%{REQUEST_ID}e","remote_ip":"%R","server_time":"%t","request_method":"%m","request_uri":"%r%Q%q","status":"%s","script_filename":"%f","server_request_millsecond":"%{mili}d","peak_memory_kb":"%{kilo}M","total_request_cpu":"%C%%"}'
```

* `%{HTTP_REQUEST_ID}e` 代表使用环境变量`HTTP_REQUEST_ID`
* `%R`: 调用方的IP地址.
* `%u`: 调用用户.
* `%t`: 当收到请求时候的服务器时间.
* `%m`: 请求方法
* `%r`: 请求`URI`不包括`query`
* `%Q`: 如果请求`query string`存在输出`?`
* `%q`: 请求的`query string`
* `%s`: 请求状态码
* `%f`: 文件脚本名称
* `%{mili}d`: 服务器响应请求的处理时间. 毫秒单位.
* `%{kilo}M`: 峰值内存消耗. 单位是`KB`
* `%C`: CPU消耗的请求.
* `%%`: 百分号.

* `%{xxx}o`: 代表输出头信息.`%{xxx}o`代表输出`header`为`xxx`的信息.

#### `docker`启动项修改

因为开启了慢日志,`docker`需要添加`--cap-add=SYS_PTRACE`权限.

如果使用`dokcer 1.10.*`还需要额外添加

```
security_opt: 
- seccomp:unconfined
```

### 其他配置项修改

* 修改默认`docker`时区

### 问题

`enable_dl`

#### cig-fcgi ?

* fastcgi.logging
* cgi.fix_pathinfo