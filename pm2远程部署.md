# pm2 远程部署

## 目录

- [安装node和pm2](#安装node和pm2)
- [生成配置文件](#生成配置文件)
- [修改配置文件](#]修改配置文件)
- [上传代码](#上传代码)
- [远程部署](#远程部署)
- [docker部署](#docker部署)
- [Force deployment](#Force deployment)
- [查看实时log](#查看实时log)
- [Tips](#Tips)

# 安装node和pm2

## 使用包管理工具安装node

`refer: https://nodejs.org/en/download/package-manager/#debian-and-ubuntu-based-linux-distributions`

在ubuntu里用sudo apt-get install nodejs安装Node.js后，会发现terminals里运行node命令（比如node –-version）时候会有No such file or directory的错误。引起这个错误的主要的主要原因是Node.js在ubuntu上默认被装到了/usr/bin/nodejs目录下，所以默认只能用nodejs来调用。

### 解决方案一:

`The nodejs-legacy package installs a node symlink that is needed by many modules to build and run correctly. The Node.js modules available in the distribution official repositories do not need it.`

### 解决方案二:

`sudo ln -s /usr/bin/nodejs /usr/bin/node`

## 安装pm2

`sudo npm install pm2 -g`

# 生成配置文件

在项目目录下执行 pm2 ecosystem , 生成一个 ecosystem.json 文件, 该文件是 pm2 远程部署的配置文件.

要把该文件添加到 git 仓库, 因为远程服务器也要使用这个文件.

# 修改配置文件

```
{
    //程序启动相关的配置
    "apps": [{
        //进程名
        "name": "dianying",

        //程序入口文件
        "script": "app.js",

        //代码更新之后,进程会自动重启
        "watch": true,

        //配置 out log 的文件路径
        "out_file": "/root/app_logs/dianying/out.log",

        //配置 error log 的文件路径
        "error_file": "/root/app_logs/dianying/err.log",

        //默认情况下,log会有一个进程id的后缀,该配置是把后缀去掉
        "merge_logs": true,

        //给log加时间戳前缀
        "log_date_format": "YYYY-MM-DD HH:mm",

        //环境变量
        "env": {
            "SVC_APP": "http://192.168.0.2:21002/saas_app/v1",
            "MONGODB_CONNECTION_STR": "root:Boluome123@192.168.0.7:27017,192.168.0.10:27017/boluome?authSource=admin&replicaSet=foba",
            "SMS_SVC": "https://dev.api.boluomeet.com",
            "SVC_PROMOTION": "http://192.168.0.6:8010/check_promotions",
            "SVC_ORDER": "http://192.168.0.2:21003",
            "REDIS_TCP_ADDR": "192.168.0.11",
            "SVC_BASIS": "http://192.168.0.2:21001/basis/v1",
            "MONGODB_CONNECTION_URL": "mongodb://root:Boluome123@192.168.0.10:27017,192.168.0.7:27017/boluome?authSource=admin&replicaSet=foba",
            "NODE_DEBUG": "DEBUG",
            "SVC_PAY": "http://192.168.0.2:20000",
            "REDIS_TCP_PORT": "6379"
        },
        "env_pro": {
            "NODE_ENV": "production"
        }
    }],

    //部署相关的配置
    //配置了一套生产环境(pro)和开发环境(dev)
    //pm2 deploy <configuration_file> <environment> 的 environment 参数用来指定部署哪一套环境
    "deploy": {
        "pro": {
            //远程服务器的登录名
            "user": "root",

            //远程服务器地址
            "host": "139.198.3.124",

            //git分支
            "ref": "origin/master",

            //git 仓库地址
            "repo": "git@github.com:kpboluome/oto_saas_dianying.git",

            //指定远程服务器的代码目录
            "path": "~/app/dianying",

            //post-deploy 钩子
            //该脚本在远程服务器执行
            //在远程服务器拉取完最新代码以后,安装 npm package, 然后通过 pm2 启动进程
            "post-deploy": "npm install && pm2 startOrRestart ~/apps/dianying/current/ecosystem.json --env pro"
        },
        "dev": {
            "user": "root",
            "host": "139.198.3.124",
            "ref": "origin/master",
            "repo": "git@github.com:kpboluome/oto_saas_dianying.git",
            "path": "~/app/dianying",
            "post-deploy": "npm install && pm2 startOrRestart ~/apps/dianying/current/ecosystem.json --env dev",
            "env": {
                "NODE_ENV": "dev"
            }
        }
    }
}

```

# 把最新代码上传到 git 服务器

```
git add .

git commit -m "update something"

git push

```

# 远程部署

## 初始化远程服务器的代码目录

如果远程服务器还没有配置文件中指定的代码目录, 可以用以下命令创建目录

pm2 deploy \<configuration_file\> \<environment\> setup

```
pm2 deploy ecosystem.json pro setup
```

## 发布代码

pm2 deploy \<configuration_file\> \<environment\>

```
pm2 deploy ecosystem.json pro
```

因为在配置文件中配置了 post-deploy 钩子:

```
npm install && pm2 startOrRestart ~/apps/dianying/current/ecosystem.json --env pro
```

所以代码部署到服务器之后,服务器会自动启动进程

#docker部署


```
pm2-docker [app.js or ecosystem.json]
```

so, 修改 ecosystem.json 配置文件中的 post-deploy 钩子

把
```
npm install && pm2 startOrRestart ~/apps/dianying/current/ecosystem.json --env pro
```
替换成
```
npm install && pm2-docker ~/apps/dianying/current/ecosystem.json --env pro
```

这样可以让程序在远程服务器上部署到docker中.

# Force deployment

如果本地代码有修改,而且没有上传到 git 服务器,这种情况下执行部署命令并不会成功,会返回以下结果

```
--> Deploying to pro environment
--> on host 139.198.3.124

  commit or stash your changes before deploying

Deploy failed
```

这种情况下可以加上 --force 参数强制部署

```
pm2 deploy ecosystem.json pro --force
```

# 查看实时log

pm2 logs [\<App name\> | \<id\>]

App name: 进程名字(可选)

id: 进程id(可选)

如果不传参数,可以查看所有进程的实时log

如果只需要查看某个进程的实时log,就加上进程名字或者id

```
pm2 logs
```

```
pm2 logs 0
```

```
pm2 logs dianying
```

# Tips

- 本地代码如果有修改,而且没有提交,可以用 --force 参数强制部署

- 加上 post-deploy 钩子, 可以让服务器拉取代码之后,自动安装 node_modules, 自动重启进程

```
"post-deploy": "npm install && pm2 startOrRestart ~/apps/dianying/current/ecosystem.json --env pro"
```

- 保证远程服务器有从 git server 拉取代码的权限

- 可以配置多套环境,真对不同的环境配置不同的环境变量. 例如配置了一套 pro 环境,那么可以在 env_pro: {} 中定义相应的环境变量

- pm2 deploy \<configuration_file\> \<environment\> \<command\>

  configuration_file 参数有一个默认值:ecosystem.json    所以如果配置文件名字是 ecosystem.json, 那么可以省略掉这个参数

