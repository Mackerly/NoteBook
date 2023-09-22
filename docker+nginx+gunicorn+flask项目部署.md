- [docker+nginx+gunicorn+flask项目部署](#dockernginxgunicornflask项目部署)
    - [使用虚拟环境创建flask项目](#使用虚拟环境创建flask项目)
    - [gunicorn部署flask项目](#gunicorn部署flask项目)
    - [gunicorn+nginx配置](#gunicornnginx配置)

# docker+nginx+gunicorn+flask项目部署
flask属于轻量级python的web框架，其流行程度可以与django媲美。因为是轻量型，所以对于开发一些中小型项目就非常方便。不过flask自带的server速度较慢，测试环境还可以，真正实际使用起来还是很多问题。同时在部署时会移植到linux系统中，稳定性更好。

### 使用虚拟环境创建flask项目
在使用flask来开发项目时，为了保证项目移植的顺平性（如在windows中开发的项目移植到linux中），通常会采用env虚拟环境方式，将pip安装的一系列第三方库放在虚拟环境env目录下。移动整个项目工程也就会将虚拟环境迁移走。

- 建立虚拟环境env，并激活使用

首先新建一个flask工程目录，并使用python -m venv env命令创建虚拟环境目录：
```bash
mkdir Flask_Proj
cd Flask_Proj
python -m venv env   #创建虚拟环境目录env
```
上述命令执行完成后，就会在Flask_Proj目录下新建一个env目录，并有如下内容：
```bash
[hadoop@big01 env]$ ll
total 4
drwxrwxr-x. 2 hadoop hadoop 202 May 17 21:03 bin
drwxrwxr-x. 2 hadoop hadoop   6 May 17 20:53 include
drwxrwxr-x. 3 hadoop hadoop  23 May 17 20:53 lib
lrwxrwxrwx. 1 hadoop hadoop   3 May 17 20:53 lib64 -> lib
-rw-rw-r--. 1 hadoop hadoop  69 May 17 20:53 pyvenv.cfg
```
然后使用source命令激活bin目录下的activate，就可以激活虚拟环境使用了：
```bash
source env/bin/activate
```
反过来如果想退出虚拟环境，使用deactivate即可。

- 有了这个env虚拟环境后，在当前工程目录下pip install flask，开启安装flask库。如果默认pypi官方链接速度较慢，可以使用：
```python
pip install -i https://mirrors.aliyun.com/pypi/simple flask
```
到底是国内镜像，速度不是一般的快。

安装完成后，可以去看一下这个库不是放在python默认安装目录里，而是放在刚创建的虚拟环境目录env里的lib文件夹下，路径为：env/lib/python3.7/site-packages。

flask安装成功后，可以在工程目录下新建一个main.py文件，在其中输入如下内容：
```python
#main.py
from flask import Flask
app = Flask(__name__)

@app.route('/')
def index():
    return 'jianhua_helloworld2020'

if __name__ == '__main__':
    app.run(port=2021,host='0.0.0.0')  #host设置为0.0.0.0，可以允许外部远程访问
```
然后使用python直接运行这个文件，就可以开启一个测试的web服务：
```bash
[hadoop@big01 asmarket]$ python main.py
 * Serving Flask app "main" (lazy loading)
 * Environment: production
   WARNING: This is a development server. Do not use it in a production deployment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:2021/ (Press CTRL+C to quit)
```
此时可以在外部浏览器上访问这个地址，端口号为2021：
![这是图片](./images/1.webp "Magic Gardens")

### gunicorn部署flask项目
上述在flask工程项目中创建env虚拟环境，是为了保证许多依赖的第三方库版本的一致。如上在启动了flask自带的server后，可以实现外部访问。但这种方式仅适用于测试，无法用于实际部署，因此一般推荐使用gunicorn来搭建flask服务器。

Gunicorn (独角兽)是一个高效的Python WSGI Server,通常用它来运行 wsgi application(由我们自己编写遵循WSGI application的编写规范) 或者 wsgi framework(如Django,Paster),地位相当于Java中的Tomcat。

- 安装gunicorn

gunicorn是一个第三方库，可以直接使用pip来安装：
```bash
pip install -i https://mirrors.aliyun.com/pypi/simple gunicorn
```
- 使用gunicorn命令

基本使用方式：
```bash
gunicorn --workers=3 main:app 
#--workers=3表示三个进程，main:app，其中main为之前flask工程中的main.py，意味这将main.py对象实例化为app。
```

允许上述命令后，就会出现如下提示：
```bash
(env) [hadoop@big01 asmarket]$ gunicorn --workers=4 te:app
[2020-05-17 22:21:04 +0800] [9123] [INFO] Starting gunicorn 20.0.4
[2020-05-17 22:21:04 +0800] [9123] [INFO] Listening at: http://127.0.0.1:8000 (9123)
[2020-05-17 22:21:04 +0800] [9123] [INFO] Using worker: sync
[2020-05-17 22:21:04 +0800] [9126] [INFO] Booting worker with pid: 9126
[2020-05-17 22:21:04 +0800] [9127] [INFO] Booting worker with pid: 9127
[2020-05-17 22:21:05 +0800] [9128] [INFO] Booting worker with pid: 9128
[2020-05-17 22:21:05 +0800] [9129] [INFO] Booting worker with pid: 9129
```
可以看到上述监听地址为：127.0.0.0，端口为8000。工作模式为sync，即同步工程模式。这两种参数都可以进行修改，其中监听地址和端口号可以在上述命令后添加 -b ip:port方式实现：
```bash
gunicorn --workers=3 main:app -b 0.0.0.0:2021
#在shell命令窗口：

(env) [hadoop@big01 asmarket]$ gunicorn --workers=4 te:app -b 0.0.0.0:2021
[2020-05-17 22:26:29 +0800] [9162] [INFO] Starting gunicorn 20.0.4
[2020-05-17 22:26:29 +0800] [9162] [INFO] Listening at: http://0.0.0.0:2021 (9162)
[2020-05-17 22:26:29 +0800] [9162] [INFO] Using worker: sync
[2020-05-17 22:26:29 +0800] [9165] [INFO] Booting worker with pid: 9165
[2020-05-17 22:26:29 +0800] [9166] [INFO] Booting worker with pid: 9166
[2020-05-17 22:26:29 +0800] [9167] [INFO] Booting worker with pid: 9167
[2020-05-17 22:26:29 +0800] [9168] [INFO] Booting worker with pid: 9168
```
此时同样可以在外部浏览器中访问，获得的效果与直接使用flask来搭建server服务一致。

对于工作模式，默认是sync，即同步模式。这种模式就是说在调用的时候，必须等待调用返回结果后，决定后续的行为。而异步则是在调用这个job的时候，不用等待其执行结果，还可以执行其他job。

举个例子：

打电话问酒店晚上有没有房间，如果是同步通信机制，酒店前台会礼貌的说：“请您稍等，我查一下"，等她查到结果了就告诉你结果，这个过程中你的电话是一直通着的，在等她的结果，决定住她们家还是去别的酒店。如果是异步通信机制，酒店前台会礼貌的说：”我先查一下，一会给您回过去。“然后她把电话挂了。你马上就可以拿手机看看周边是否有便利的餐馆。等她查到了，她会主动给你打电话。而此时餐馆也查好了，可以开启美好的旅行了。

如果要更换为异步模式，可以使用gevent。此时还需要pip来安装gevent。
```bash
gunicorn --workers=3 main:app -b 0.0.0.0:2021 -k 'gevent'
```
- 使用参数配置文件设定

使用上述脚本命令还是不方便的，gunicorn可以使用-c参数，就是使用配置文件。将一些参数设定放在该配置文件里:
```python
import os
bind='0.0.0.0:5001'   #绑定监听ip和端口号
workers=3               #同时执行的进程数，推荐为当前CPU个数*2+1
backlog=2048            #等待服务客户的数量，最大为2048，即最大挂起的连接数
worker_class="gevent" #sync, gevent,meinheld   #工作模式选择，默认为sync，这里设定为gevent异步
max_requests=1000       #默认的最大客户端并发数量
daemon=True           # 是否后台运行
reload=True            # 当代码有修改时，自动重启workers。适用于开发环境。
pidfile='./gunicore.pid'    #设置pid文件的文件名
loglevel='debug'     # debug error warning error critical
accesslog='log/gunicorn.log'  #设置访问日志
errorlog='log/gunicorn.err.log' #设置问题记录日志
```
将上述内容存放在config.py文件中，然后在命令行输入：
```bash
gunicorn -c config.py main:app
```
如下为本次实践时配置的config.py参数：
```python
import  os
from  gevent import monkey
monkey.patch_all()
import multiprocessing
debug = False
bind = "0.0.0.0:5001"
pidfile = "gunicorn.pid"
accesslog="/home/hadoop/asmarket/logs/gunicorn.log"
workers = multiprocessing.cpu_count()*2 + 1
worker_class = "gevent"
daemon=True
```
然后开启运行，此时gunicorn设置为后台看守进程，先直接从外部浏览器访问，然后查看log文件，内容如下：
```bash
[hadoop@big01 logs]$ more gunicorn.log 
192.168.58.1 - - [17/May/2020:23:30:44 +0800] "GET / HTTP/1.1" 200 23 "-" "Mozi
lla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/
68.0.3440.106 Safari/537.36"
192.168.58.1 - - [17/May/2020:23:30:46 +0800] "GET / HTTP/1.1" 200 23 "-" "Mozi
lla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/
68.0.3440.106 Safari/537.36"
192.168.58.1 - - [17/May/2020:23:30:47 +0800] "GET / HTTP/1.1" 200 23 "-" "Mozi
lla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/
68.0.3440.106 Safari/537.36"
```
至此基本的gunicorn+flask异步服务部署就实现了。

### gunicorn+nginx配置
有了gunicorn和gevent后，gunicorn可以实现多进程http服务，不过其性能还是相对nginx这种专业的web服务要差一些，主要体现在对高并发的处理、安全问题、静态资源文件的处理等。因此一般情况会在gunicorn之上再配置一层nginx服务。其基本架构示意如下（图来源于百度）：
![这是图片](./iamges/2.webp "Magic Gardens")
- docker部署nginx

由于nginx采用安装方式还相对比较麻烦，可以直接使用docker来部署。不过当然首先在root账户下安装docker服务：
```bash
#yum安装docker
yum install docker
#启动docker进程服务
systemctl start docker
systemctl enable docker  
```
有了docker后，使用docker的search和pull服务就可以将nginx拉取到本机上：
```bash
[root@big01 ~]# docker pull nginx
Using default tag: latest
Trying to pull repository docker.io/library/nginx ... 
latest: Pulling from docker.io/library/nginx
afb6ec6fdc1c: Pull complete 
b90c53a0b692: Pull complete 
11fa52a0fdc0: Pull complete 
Digest: sha256:30dfa439718a17baafefadf16c5e7c9d0a1cde97b4fd84f63b69e13513be7097
Status: Downloaded newer image for docker.io/nginx:latest
```
然后使用docker 查看镜像：
```bash
[root@big01 ~]# docker images
REPOSITORY              TAG                 IMAGE ID            CREATED             SIZE
docker.io/nginx         latest              9beeba249f3e        2 days ago          127 MB
docker.io/hello-world   latest              bf756fb1ae65        4 months ago        13.3 kB
```
接下来可以运行nginx容器：
```bash
[root@big01 ~]# docker run --name mynginx -p 8080:80 -d nginx
79b2f668784f866869f41ab08468784cf5f694fb451486250f37eaaa11808411
```
此时可以从外部浏览器访问获得默认的nginx响应页面：
![这是图片](./images/3.webp "Magic Gardens")
下面对nginx访问页面做一个映射，因为如果要进入docker内部访问的话还是很不方便的，因此一般情况将docker镜像作为一个服务，而将实际的资源进行一个映射。在本地机器上放置资源，映射到容器内部，nginx访问内部文件路径时就映射到访问外部本地资源上了，这样便于资源分配以及web文件的管理。这里就是增加一个docker的-v参数，格式为本地资源:容器资源。

先进入docker内部，查看nginx的配置文件：
```bash
[root@big01 nginx]# docker exec -it 16528ae739c4 /bin/bash
root@16528ae739c4:/# cd /etc/nginx/conf.d/
root@16528ae739c4:/etc/nginx/conf.d# more default.conf 
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;
    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
```
默认访问的路径为/usr/share/nginx/html下的html文件，这个可以后续修改。我们可以先测试一下，将这个路径映射到本地机器上。这样需要重新run一个镜像：
```bash
[root@big01 nginx]# docker run --name mynginxt -v /usr/share/nginx/:/usr/share/nginx/html -p 8021:80 -d nginx
16528ae739c40d28a68be67b10aac42343b97b5fa468565127eaf051ea25886c
```
上述命令中：-v /usr/share/nginx/:/usr/share/nginx/html，就是将本地的/usr/share/nginx目录映射到容器内部的/usr/share/nginx/html目录中，如果我们在本地的nginx目录下新建一个index.html网页，那访问的时候就是访问这个新建的index.html网页。如下新建一个简单网页并保存为index.html。
```html
<html>
<head>
  <meta charset="utf-8">
</head>
<body>
<h1>我的第一个标题</h1>
<p>我的第一个段落。</p>
</body>
</html>
```
接下来就可以在外部浏览器访问，注意端口现在为8021。

![这是图片](./images/4.webp "Magic Gardens")

- nginx+gunicorn部署

上述gunicorn部署时，ip为0.0.0.0，端口号为5001。在使用nginx代理这个服务时，修改nginx相应的配置文件即可。不过因为是docker部署，因此还需使用docker来操作。此时也可以将配置文件映射到外部宿主机上。

首先启动gunicorn+flask项目服务：
```bash
[hadoop@big01 asmarket]$ gunicorn -c config.py main:app
```
然后在/usr/share/nginx目录下新建一个nginx.conf文件，在其中输入如下内容：
```bash
server {
    listen 80;
    server_name asmarket.com; # 这是HOST机器的外部域名，用地址也行

    location / {
        proxy_pass http://0.0.0.0:5001; # 这里是指向 gunicorn host 的服务地址
        proxy_set_header Host $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
  }
```
将其映射到nginx容器里的default.conf配置文件：
```bash
[root@big01 nginx]# docker run --name mynginx -v /usr/share/nginx/nginx.conf:/etc/nginx/conf.d/default.conf -d nginx
39026ba7d80eac3d59d00ead25d7275e5f2b125dde3a9e3379dffb5c10ef9662
```
这样在启动nginx时直接使用的就是刚才新建立的nginx.conf配置文件。

此时nginx已经启动了:
```bash
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                   PORTS                  NAMES
39026ba7d80e        nginx               "nginx -g 'daemon ..."   7 minutes ago       Up 7 minutes             80/tcp                 mynginx
```
然后就可以运行flask项目了。

如何验证确实是代理了gunicornweb服务，可以直接使用nginx原有的默认80端口访问，如果出现错误，说明nginx已经代理了web服务，否则就是没成功。

- supervisor进程守护

nginx一般不会莫名其妙被关闭，但gunicorn是一个进程，完成有有可能因为一些原因被关闭或者阻塞，为了保证gunicorn进程，需要使用看护进程插件。这里使用supervisor来解决这个问题。

supervisor专门用户linux端进程管理，首先使用pip安装一下这个插件：
```bash
[root@big01 ~]# pip install supervisor
```
安装成功后，可以创建一个配置文件：
```bash
# 设置默认配置
$ echo_supervisord_conf > /etc/supervisord.conf
$ vi /etc/supervisord.conf
```
这个配置文件放在/etc/目录下，名为supervisor.conf。接下来就可以修改其配置了：
```bash
[program:myapp]
command=/usr/local/bin/gunicorn -c config.py main:app               
directory=/home/hadoop/asmarket
autostart=true                ; start at supervisord start (default: true)
startsecs=1                   ; # of secs prog must stay up to be running (def. 1)
startretries=3                ; max # of serial start failures when starting (default 3)
exitcodes=0                   ; 'expected' exit codes used with autorestart (default 0)
stdout_logfile=/home/hadoop/asmarket/logs/main.logs
stdout_logfile_maxbytes=50MB   ; max # logfile bytes b4 rotation (default 50MB)
user=root
```
修改完成后，直接使用supervisord来执行：
```bash
supervisord -c supervisor.conf
```
这样myapp的进程就启动了。可以使用supervisorctl status命令来查看当前进程状态：

```bash
supervisorctl status
```
