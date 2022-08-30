## YApi  可视化接口管理平台 （多级目录分支）

体验地址：

[https://yapi.feiyanyun.com/](https://yapi.feiyanyun.com/)  
账号：`admin@123.com`
密码：`123456`

<br/>

**喜欢的老铁,github求给个star❤️** 
[仓库地址](https://github.com/zybieku/yapi) 


> 参考图片 

  ![img](./yapi_menu.png)

<br/>

---

<br/>

### 分支介绍 
  * fork 官方 **api v1.10.2**
  - 主要支持`多级目录`,`添加子目录`
  - 支持不同层级`目录拖拽`
  - 支持多级`目录搜索`

---


### 内网部署
  - 需要事先安装 nodejs，mongodb  
  - 这里采用不同安装，部署的方式，大体上步骤和官方一样 


---

<br/>

 **方式一**：`zip包解压安装`  
    - 1 下载zip包，可选 [CSDN源码包](https://download.csdn.net/download/zybieku/34093967) , 或者 [百度网盘，提取码: 5qgk](https://pan.baidu.com/s/1aoA_KoNw9pyEx0Sw8ktxwg)    
    - 2 切换到 yapi 目录，修改config配置  
    - 3 切换到 vendors 目录，运行 npm run install-server （初始数据库，有库数据略过）  
    - 4 node server/app.js 启动（pm2亦可）  


---
<br/>

 #### 
  **方式二**  `git下载依赖`

  **1.创建工程目录**
  
  ```shell
   mkdir yapi && cd yapi   #或者手动创建目录   
   git clone https://github.com/zybieku/yapi.git vendors --depth=1 
  ```
 
  **2.修改配置,安装依赖**
  > config.json里面的内容，具体看官方

  ```shell
   #复制完成后请修改相关配置
   cp vendors/config_example.json ./config.json 
   # 指令打开config，或者用鼠标打开
   vi ./config.json 
   #再进入vendors
   cd vendors
   npm install --production --registry https://registry.npm.taobao.org
   #安装程序会初始化数据库，管理员账号名可在 config.json 配置
    npm run install-server 
  ``` 

 **5.启动（也可以使用pm2）** 

  ```shell
    #启动服务器后，#请访问 127.0.0.1:{config.json配置的端口}
    node server/app.js 
     # linux 后台模式 注意 nohup 与 & exit
    nohup  node server/app.js exit    
  ```
---
 **常见问题**

 - 1. 依赖报错
 一般依赖报错是由于 yapi的很多依赖库版本有点旧 ，需要手动锁定版本

 - 2. node-sass node-gyp  安装不上 
   可能是node-gyp没安装  

   建议node14,高版本和node-sass的版本匹配
   ```shell
    npm install -g node-gyp
    npm rebuild node-gyp
   ```
 
 - 3. 没有ykit指令    
   npm install -g ykit

---
   
 #### 
#### 方式三  `docker 容器`  
**1. Build Yapi 镜像**
```
# 仓库根目录下执行
docker build . -t zybieku/yapi
```
**2. MongoDB 部署**
```
# 运行 mongo-db
docker run -d \
  --name yapi-mongodb \ 
  --restart always \
  -p 27017:27017 \
  -v $PWD/mongo-data:/data/db \
  -e MONGO_INITDB_DATABASE=yapi \ 
  -e MONGO_INITDB_ROOT_USERNAME=yapipro \
  -e MONGO_INITDB_ROOT_PASSWORD=yapipro1024 \
  mongo
```
初始化数据库：
```
# 进入 mongodb 容器
docker exec -it yapi-mongodb /bin/sh
# 进入 mongo cli
mongsh
# 以下命令在 mongo cli 执行
use admin;
db.auth("yapipro", "yapipro1024");
# 创建 yapi 数据库
use yapi;
# 创建给 yapi 使用的账号和密码，限制权限
db.createUser({
  user: 'yapi',
  pwd: 'yapi123456',
  roles: [
 { role: "dbAdmin", db: "yapi" },
 { role: "readWrite", db: "yapi" }
  ]
});
# 退出 Mongo Cli
exit
# 退出容器
exit
```


在宿主机的当前目录，根据自己修改创建一个 Yapi 配置文件 config.json
```
{
   "port": "3000",
   "adminAccount": "admin@gmail.com",
   "timeout":120000,
   "db": {
     "servername": "宿主机IP",
     "DATABASE": "yapi",
     "port": 27017,
     "user": "yapi",
     "pass": "yapi123456",
     "authSource": ""
   },
   "mail": {
     "enable": false,
     "host": "smtp.gmail.com",
     "port": 465,
     "from": "*",
     "auth": {
       "user": "hexiaohei1024@gmail.com",
       "pass": "xxx"
     }
   }
}
```

**3. 启动 Yapi**
初始化数据库表
```
docker run -d --rm \
  --name yapi-init \
  -v $PWD/config.json:/yapi/config.json \
   yapipro/yapi \
  server/install.js
```
初始化管理员账号在上面的 config.json 配置中 admin@gmail.com，初始密码是 yapi.pro或者ymfe.org，可以登录后进入个人中心修改
```
docker run -d \
  --name yapi \
  --restart always \
  -p 3000:3000 \
  -v $PWD/config.json:/yapi/config.json \
  zybieku/yapi server/app.js
```
在服务器上验证 yapi 启动是否成功
```
curl localhost:3000
```