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


缺点
> 二层代理内部并没有实现将请求目标网站的握手流量封包解包，仅仅只是简单的代理转发。      
> 所以服务端可以篡改内容，存在安全风险，为了安全，最好是自建服务端。

## 自建服务端步骤
配置非常简单，会搭nginx即可

###  1. 准备工作
* 一台境外服务器
* 一个域名，自签名证书
* 下载DevSidecar

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
    location / {  //其他访问全部拒绝，规避GFW的钓鱼试探
       resolver 1.1.1.1;
       deny all;
    }
}
```
### 3. DevSidecar配置
按如下设置       
应用---> 不好说 ---> 代理服务端    

将代理服务端修改为如下地址，应用即可
```
yourdomain.com/xxxxxxxx
```

> 这个xxxxxx要修改成你自己的，你把它当成密码     
> 注意保护好 `yourdomain.com/xxxxxxxx`，不要公开
