# Netease-Music-Nginx-Proxy
这是一个多Server的代理程序，之所以需要多Server，是因为网易云音乐会向ip.ws.126.net发出IP查询请求，所以设置music.163.com和ip.ws.126.net两个代理Server。
## 安装与控制Nginx
```
# 安装Nginx
sudo yum install epel-release
sudo yum install nginx 
# 设置开机启动
sudo systemctl enable nginx
# 启动
sudo systemctl start nginx
# 停止
sudo systemctl stop nginx
# 重新启动（每次修改配置文件后需要重新启动使其生效）
sudo systemctl restart nginx
```

## 配置文件
首先添加Nginx Server配置文件
```
sudo nano /etc/nginx/conf.d/proxy-163.conf
```
文件的内容如下：
```
# Cahce settings
proxy_cache_path  /var/cache/nginx  levels=1:2    keys_zone=STATIC:10m inactive=24h  max_size=1g;

# Netease music proxy
server {
    listen       80;
    listen       [::]:80;
    server_name  music.163.com;

    location /weapi/feedback/weblog {
        add_header Set-Cookie "os=uwp; path=/";
        error_page 405 = $uri;
        alias /usr/share/nginx/html/163-uwp.json;
    }

    location / {
        proxy_redirect off;
        proxy_pass http://music.163.com/;
        # 用于伪装的IP，在网上查询一个国内的IP，替换掉
        proxy_set_header X-Real-IP x.x.x.x;
        proxy_cache            STATIC;
        proxy_cache_valid      200  1d;
        proxy_cache_use_stale  error timeout invalid_header updating
                               http_500 http_502 http_503 http_504;
    }
}


# IP query proxy
server {
    listen       80;
    listen       [::]:80;
    server_name  ip.ws.126.net;

    location /ipquery {
        alias /usr/share/nginx/html/163/ipquery.txt;
    }
}
```
在这里面我们还用到两个文件，分别是：/usr/share/nginx/html/163-uwp.json和/usr/share/nginx/html/ipquery.txt，内容分别为：
```
{"code":200,"uwp":1}
```
与：
```
var lo="广东省", lc="广州市";
var localAddress={city:"广州市", province:"广东省"}
```
/usr/share/nginx/html/ipquery.txt需要保存为GBK编码，我们可以使用以下命令：
```
sudo iconv -f UTF-8 -t GBK /usr/share/nginx/html/ipquery.txt
```

## 网络控制
服务器有防火墙或者安全组控制的，需要打开80端口的出入站权限。

## 测试
查询原始IP得到如下结果：
http://ip.ws.126.net/ipquery?ip=x.x.x.x
var lo="康乃狄克州", lc=""; var localAddress={city:"", province:"康乃狄克州"}

如果将其替换到/usr/share/nginx/html/ipquery.txt，网易云音乐就又会访问受限了。


## 参考
https://jixun.moe/post/ymusic-hosts-fix/
