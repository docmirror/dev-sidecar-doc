
# 部署到cloudflare免费workers上

## 1、 注册cloudflare账号
https://www.cloudflare.com/  
验证完邮箱
## 2、创建workers
* 点击左侧Workers菜单
* 点击右边创建服务按钮
* 服务名称随意填写（YourWorkersName）
* 点击右下角的创建服务，创建成功后会自动进入服务配置页面

## 3、部署代理脚本

* 点击快速编辑按钮
* 删除左侧原有的代码
* 将下方代码粘贴进去
* 按照代码中注释部分进行修改
```js

addEventListener("fetch", event => {
  event.respondWith(eventHandler(event))
})

async function eventHandler(event) {
  const req = event.request
  const url = req.url
    // YourWorkersName.YourAccountName.修改为你的works地址
    // xxxxxxx改成任意一串字符，作为path，当做密码，不要公开
  const target = url.replace("https://YourWorkersName.YourAccountName.workers.dev/xxxxxxxx/","")
  req.url = target;
  if(target.startsWith("http")){
    return new Response("500")
  }
  const resp = await fetch("https://"+target,req)
  return resp
}
```

## 4、 点击部署按钮

## 5、 配置DevSidecar功能增强的代理服务端
 域名 = YourWorkersName.YourAccountName.workers.dev    
 路径 = xxxxxxxx

 点击应用即可
 
## 6、 测试访问