# nginx_waf

nginx\_waf是基于ngx\_lua,以lua脚本语言开发的防cc攻击软件。继承了nginx高并发，高性能的特点，可以以非常小的性能损耗来防范大规模的cc攻击。

下面介绍nginx\_waf防cc的一些特性：

1. 限制单个IP或IP段或UA在一定时间内的请求次数
2. 支持向访客发送带有验证码的页面，来进一步识别，以免误伤
3. 支持直接断开恶意访客的连接
4. 支持白名单/黑名单功能

## 部署nginx_waf
### 安装ngx_lua

按照ngx_lua官网手动安装

### 安装nginx_waf

假设我们把nginx_waf安装到/data/www/waf/，当然你可以选择安装在任意目录。

```
cd /data/www
git clone https://github.com/rwx------/nginx_waf.git
mv nginx_waf waf
```

### 生成验证码图片

为了支持验证码识别用户，我们需要先生成验证码图片。生成验证码图片需要系统安装有php，以及php-gd模块。
用以下命令执行getImg.php文件生成验证码

```
cd /data/www/waf/captcha/
/usr/local/php/bin/php getImg.php
```

大概要生成一万个图片，可能需要花几分钟的时间。

### 修改nginx.conf配置文件

向http区块输入如下代码：

```
lua_package_path "/data/www/waf/?.lua";
lua_shared_dict guard_dict 100m;
lua_shared_dict dict_captcha 70m;
init_by_lua_file '/data/www/waf/init.lua';
access_by_lua_file '/data/www/waf/runtime.lua';
lua_max_running_timers 1;
```

### todo
- 优化: iptonumber的计算，在init就进行预加载和计算
- 新增: redis支持，多机器共享黑/白/灰名单
- ~~优化: 无ua情况下的处理~~
- ~~优化: 关于封禁时以ip还是ip+ua的形式的选择开关~~
- ~~新增: log only 模式，不真正进行封禁操作，只记录日志。~~

### 感谢
该产品参考并大量借鉴了https://github.com/love_shell/ngx_lua_waf 和 https://github.com/centos-bz/HttpGuard ，感谢他们的工作.
相较于这两个模块
增加了ip段的黑名单白名单精确控制。
精简了部分功能。
可进行ua过滤
