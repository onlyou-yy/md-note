## pm2基本信息

**PM2是什么**
 PM2（Process Manager 2 ）是具有内置负载均衡器的Node.js应用程序的生产运行时和进程管理器。 它允许您永久保持应用程序活跃，无需停机即可重新加载它们，并促进常见的Devops任务。

**PM2特性**

+ 日志管理：应用程序日志保存在服务器的硬盘中~/.pm2/logs/
+ 负载均衡：PM2可以通过创建共享同一服务器端口的多个子进程来扩展您的应用程序。这样做还允许您以零秒停机时间重新启动应用程序。
+ 终端监控：可以在终端中监控您的应用程序并检查应用程序运行状况（CPU使用率，使用的内存，请求/分钟等）。
+ SSH部署：自动部署，避免逐个在所有服务器中进行ssh。
+ 静态服务：支持静态服务器功能
+ 多平台支持：适用于Linux（稳定）和macOS（稳定）和Windows（稳定）

PM2安装 先安装nodejs,下载路径：http://nodejs.cn/#download 然后全局安装pm2

```bash
#npm方式
npm install pm2 -g
 
#yarn方式
yarn global add pm2
 
#linux
apt update && apt install sudo curl && curl -sL https://raw.githubusercontent.com/Unitech/pm2/master/packager/setup.deb.sh | sudo -E bash -
```

## PM2命令方式

### 启动服务

```shell
pm2 start app.js                //启动app.js应用
pm2 start app.js --name demo    //启动应用并设置name
pm2 start app.sh                //脚本启动
pm2 start "node index.js"       //执行命令启动
pm2 start npm --name demo -- run start       //执行命令启动,相当于是 pm2 start "npm start" --name demo
```

### 停止服务

```shell
pm2 stop all               #停止所有应用
pm2 stop [AppName]        #根据应用名停止指定应用
pm2 stop [ID]             #根据应用id停止指定应用
```

### 删除应用

```shell
pm2 delete all               #关闭并删除应用
pm2 delete [AppName]        #根据应用名关闭并删除应用
pm2 delete [ID]            #根据应用ID关闭并删除应用
```

### 创建开机自启动

```shell
pm2 startup
```

### 更新PM2

```shell
pm2 updatePM2
pm2 update
```

### 监听模式

```shell
pm2 start app.js --watch    #当文件发生变化，自动重启
```

当我们需要部署的是一个通过`npm`启动的服务，比如Nuxt的动态服务（非静态网站）时可以这样来设置命令

```shell
# 在packages.json中定义了命令 server: "npm run build && npm run start"
pm2 start npm --watch --name demo -- run server --ignore-watch="node_modules .vscode .history"
```

这样当项目中除了`node_modules .vscode .history`目录下的文件变化就会重新运行`npm run server`命令

可以运行`pm2 logs demo`来查看任务的启动状态

### 静态服务器

```shell
pm2 serve ./dist 9090        #将目录dist作为静态服务器根目录，端口为9090
```

### 启用群集模式（自动负载均衡）

```shell
#max 表示PM2将自动检测可用CPU的数量并运行尽可能多的进程
#max可以自定义，如果是4核CPU，设置为2者占用2个
pm2 start app.js -i max
```

### 重新启动

```shell
pm2 restart app.js        #同时杀死并重启所有进程。短时间内服务不可用。生成环境推荐使用reload
```

### 0秒停机重新加载

```shell
pm2 reload app.js        #重新启动所有进程，始终保持至少一个进程在运行
pm2 gracefulReload all   #优雅地以群集模式重新加载所有应用程序
```

### 查看启动列表

```shell
pm2 list
```

### 查看每个应用程序占用情况

```shell
pm2 monit
```

### 显示应用程序所有信息

```shell
pm2 show [Name]      #根据name查看
pm2 show [ID]        #根据id查看
```

### 日志查看

```shell
pm2 logs            #查看所有应用日志
pm2 logs [Name]    #根据指定应用名查看应用日志
pm2 logs [ID]      #根据指定应用ID查看应用日志
```

### 保存当前应用列表

```shell
pm2 save
```

### 重启保存的应用列表

```shell
pm2 resurrect
```

### 清除保存的应用列表

```shell
pm2 cleardump
```

### 保存并恢复PM2进程

```shell
pm2 update
```

## PM2配置文件方式

### 生成示例配置文件

```shell
pm2 ecosystem        #生成一个示例JSON配置文件
pm2 init
```

配置文件示例（实际使用自行删除）

```js
module.exports = {
    apps : [{
        name      : 'API',      //应用名
        script    : 'app.js',   //应用文件位置
        env: {
            PM2_SERVE_PATH: ".",    //静态服务路径
            PM2_SERVE_PORT: 8080,   //静态服务器访问端口
            NODE_ENV: 'development' //启动默认模式
        },
        env_production : {
            NODE_ENV: 'production'  //使用production模式 pm2 start ecosystem.config.js --env production
        },
        instances:"max",          //将应用程序分布在所有CPU核心上,可以是整数或负数
        watch:true,               //监听模式
        output: './out.log',      //指定日志标准输出文件及位置
        error: './error.log',     //错误输出日志文件及位置，pm2 install pm2-logrotate进行日志文件拆分
        merge_logs: true,         //集群情况下，可以合并日志
        log_type:"json",          //日志类型
        log_date_format: "DD-MM-YYYY",  //日志日期记录格式
    }],
    deploy : {
        production : {
            user : 'node',                      //ssh 用户
            host : '212.83.163.1',              //ssh 地址
            ref  : 'origin/master',             //GIT远程/分支
            repo : 'git@github.com:repo.git',   //git地址
            path : '/var/www/production',       //服务器文件路径
            post-deploy : 'npm install && pm2 reload ecosystem.config.js --env production'  //部署后的动作
        }
    }
};
```

 

PM2 管理可执行命令行程序可以使用`pm2 start -x './frpc' -n frp -- -c ./frpc.ini`来启动，注意 -x 是说明程序是可以执行程序，-- 后边的参数是传给命令行。
 PM2 管理 Python 类的程序，可以使用`pm2 start server.py --interpreter python -n ssr -- -p 8888 -m aes-256-cfb -O auth_aes128_md5 -o tls1.2_ticket_auth_compatible -k password`, 里边的 --interpreter 参数可以设置程序类型，-- 后边的参数是传给程序的。

 **命令示例大全：** 

```shell
npm install pm2 -g     # 命令行安装 pm2 
pm2 start app.js -i 4 #后台运行pm2，启动4个app.js 
                                # 也可以把'max' 参数传递给 start
                                # 正确的进程数目依赖于Cpu的核心数目
pm2 start app.js --name my-api # 命名进程
pm2 list               # 显示所有进程状态
pm2 monit              # 监视所有进程
pm2 logs               #  显示所有进程日志
pm2 stop all           # 停止所有进程
pm2 restart all        # 重启所有进程  pm2 reload all         # 0秒停机重载进程 (用于 NETWORKED 进程)
pm2 stop 0             # 停止指定的进程
pm2 restart 0          # 重启指定的进程
pm2 startup            # 产生 init 脚本 保持进程活着
pm2 web                # 运行健壮的 computer API endpoint (http://localhost:9615)
pm2 delete 0           # 杀死指定的进程
pm2 delete all         # 杀死全部进程


运行进程的不同方式：
pm2 start app.js -i max  # 根据有效CPU数目启动最大进程数目
pm2 start app.js -i 3      # 启动3个进程
pm2 start app.js -x        #用fork模式启动 app.js 而不是使用 cluster
pm2 start app.js -x -- -a 23   # 用fork模式启动 app.js 并且传递参数 (-a 23)
pm2 start app.js --name serverone  # 启动一个进程并把它命名为 serverone
pm2 stop serverone       # 停止 serverone 进程
pm2 start app.json        # 启动进程, 在 app.json里设置选项
pm2 start app.js -i max -- -a 23                   #在--之后给 app.js 传递参数
pm2 start app.js -i max -e err.log -o out.log  # 启动 并 生成一个配置文件
你也可以执行用其他语言编写的app  ( fork 模式):
pm2 start my-bash-script.sh    -x --interpreter bash
pm2 start my-python-script.py -x --interpreter python
```

 

转载自[PM2介绍及简易使用手册](http://auan.cn/internet/2059.html)