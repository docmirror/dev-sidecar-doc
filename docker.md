# docker镜像启动
安装步骤
* 安装docker
* 安装docker-compose
* 启动nginx容器
* 配置dev-sidecar
* go 

## 1、安装docker
如果你是centos，执行如下命令即可
```shell
sudo yum install -y yum-utils
sudo yum-config-manager \
--add-repo \
https://download.docker.com/linux/centos/docker-ce.repo

sudo yum install docker-ce docker-ce-cli containerd.io
sudo systemctl enable docker.service
sudo systemctl start docker
```

如果不是centos，请按如下官方步骤安装好docker

https://docs.docker.com/engine/install/centos/


## 2、安装docker-compose

```shell
sudo curl -L "https://github.com/docker/compose/releases/download/1.28.6/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
更多安装信息，请参考官方文档

https://docs.docker.com/compose/install/

## 3、启动nginx容器
* 先 clone 本仓库到本地
* 复制你的证书文件到`ds-nginx/ssl`目录下
* 修改`ds-nginx`下的`docker-compose.yml`文件(按照里面的提示修改)
* 将`ds-nginx`整个目录，上传到你服务器的`~/deploy/`目录下
* 执行启动命令
```shell
cd ~/deploy/ds-nginx/
docker-compose up -d
```
## 4、修改dev-sidecar服务端配置
按如下设置         
应用---> 功能增强 ---> 代理服务端       
填上一步时配置的三个变量（域名、路径、密码），应用即可      
![](./image/server.png)   
