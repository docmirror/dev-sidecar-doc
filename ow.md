# 梯子原理

1. 通过两层代理，将目标请求链接隐藏在https之中被加密，规避GFW的https握手特征检查
2. 通过二级路径（下图的xxxxxxxx），规避GFW的试探性钓鱼检查


```
浏览器访问：               https://www.google.com    
                                |
DevSidecar【第一层代理】：  https://yourdomain.com/xxxxxxxx/www.google.com/
                                |
GFW：                          GFW
                                |
境外Nginx【第二层代理】：    获取到xxxxxxxx之后的域名和地址，代理到https://www.goolge.com
                                |
DevSidecar：               返回给DevSidecar
                                |
浏览器访问：                返回给浏览器

```

在GFW看来你的流量就是在访问`yourdomain.com`这个正常的大网站而已

优点：
> 简单易懂有效    
> 可以套CDN，隐藏服务器IP

缺点：
> 1、 仅支持HTTPS     
> 2、 二层代理并没有对tls协议再次封装，仅仅只是简单的代理转发。      
> 所以服务端可以篡改内容，存在安全风险，为了安全，最好是自建服务端。    
> 理论上可以在`yourdomain.com/xxxxxxxx`下封装与目标网站的tls，防篡改

总结两点：
> 大道至简：做的越多，错的越多。简单最有效，大隐隐于市。      
> 降维打击：安全我都不要了，你奈我何。(自建服务器可以解决)

## 自建服务端步骤
配置非常简单，会搭nginx即可

###  1. 准备工作
* 一台境外服务器
* 一个域名，自签名证书
* 下载DevSidecar

我的服务器是[1核1G的香港主机](https://www.ucloud.cn/site/active/kuaijie.html?invitation_code=C1xF886DAFF2658)       
如果你没有合适的境外主机，可以点击链接去购买，新用户还是挺划算的

### 2. nginx配置
```
 server {
    listen 443 ssl;  
    server_name yourdomain.com ; # 修改为你的域名
    ssl_certificate /app/ssl/你的域名证书.crt;   # 修改为你的证书
    ssl_certificate_key /app/ssl/你的域名证书.key; # 修改为你的证书
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:HIGH:!aNULL:!MD5:!RC4:!DHE;
    ssl_prefer_server_ciphers on;
    
   
    location ^~/xxxxxxxx/ {  # xxxxxxxx 改成你自己随便任意的前缀地址
        resolver 1.1.1.1 ipv6=off;
        if ( $http_dspassword != 'your password' ){ # 你的密码，如果不配置密码，去掉它即可
            return 403;
        }
        if ( $request_uri ~ /xxxxxxxx/([^/]+)/(.*) ){ # 将xxxxxxxx修改为你路径前缀
            set  $_host $1;
            set  $_uri $2;
         }
        proxy_pass $scheme://$_host/$_uri;
        proxy_redirect https://yourdomain.com/xxxxxxxx/ /;  # 修改为你的域名和路径前缀
        proxy_buffers   256 4k;
        proxy_max_temp_file_size 0k;
        proxy_set_header referer $scheme://$_host;
        proxy_set_header Host $_host;
        proxy_ssl_server_name on;
    }
    location / {  # 其他访问全部拒绝，规避GFW的钓鱼试探
       resolver 1.1.1.1;
       return 404; # 也可以改成403、502等其他错误， 或者返回自己的伪装网站
    }
}
```
### 3. DevSidecar配置
按如下设置       
应用---> 功能增强 ---> 代理服务端    

将代理服务端修改为如下地址，应用即可
```
域名：yourdomain.com  路径：xxxxxxxx  密码：yourpassword
```

> `xxxxxxxx`一定要修改成你自己的，你把它也当成是一个密码     
> 注意保护好 `域名、路径 和密码`，不要公开
