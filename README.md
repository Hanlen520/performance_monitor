# performace_monitor
## 介绍
#### 已完成如下功能<br>
1、监控整个服务器的CPU使用率、剩余内存大小、磁盘IO和网络带宽<br>
2、监控指定端口的CPU使用率、内存占用大小<br>
3、针对java应用，可以监控jvm大小和垃圾回收情况；当Full GC频率过高时，可发送邮件提醒<br>
4、当系统剩余内存过低时，可发送邮件提醒；也可设置自动清理缓存<br>
5、可随时启动/停止监控指定端口<br>
6、当端口重启后，可自动重新监控<br>
7、可按照指定时间段可视化监控结果<br>
8、支持集群部署<br>

#### 实现过程
1、为保证监控结果准确性，直接使用Linux系统命令获取数据；且可视化时未做任何数据处理<br>
2、使用基于协程的http框架`aiohttp`满足高并发<br>
3、服务端前端使用`jinjia2`模板渲染<br>
4、采用线程池+队列的方式实现同时监控多个端口<br>
5、客户端每隔5s向服务端注册本机IP和端口<br>
6、服务端每隔5s会查询所有已注册的客户端的状态<br>
7、使用influxDB数据库存储监控数据；数据可设置自动过期时间<br>

## 使用
1. 克隆 performance_monitor
   ```shell
   git clone https://github.com/leeyoshinari/performance_monitor.git
   ```
   master文件夹是服务端，只需部署一个即可；slave文件夹是客户端，部署在需要监控的服务器上<br>

2. 分别修改master和slave文件夹里的配置文件 `config.ini`

3. 部署InfluxDB数据库
   
4. 分别运行master和slave文件夹中的`server.py`
   ```shell
   nohup python3 server.py &
   ```

5. 页面访问<br>
   （1）从机（客户端）启动后，输入`http://ip:port`可以看到页面显示服务器的CPU核数、总内存和磁盘号<br>
   ![slave home](https://github.com/leeyoshinari/performance_monitor/blob/master/master/templates/slave.jpg)
   
   （2）主机（服务端）启动后，输入`http://ip:port`可以看到首页，页面展示已经注册的从机（客户端）的IP和注册时间<br>
   ![master home](https://github.com/leeyoshinari/performance_monitor/blob/master/master/templates/home.jpg)
   
   （3）主机（服务端）启动后，输入`http://ip:port/startMonitor`可以看到监控页面；点击开始监控按钮，即可在指定的服务器上开始监控指定的端口；点击停止监控按钮，即可在指定的服务器上停止监控指定的端口；点击获取监控列表按钮，可以查看当前已经监控的端口<br>
   ![startMonitor](https://github.com/leeyoshinari/performance_monitor/blob/master/master/templates/monitor.jpg)
   
   （4）主机（服务端）启动后，输入`http://ip:port/Visualize`可以看到可视化页面；点击画图按钮，即可将指定服务器上的指定端口的监控数据可视化<br>
   ![Visualize](https://github.com/leeyoshinari/performance_monitor/blob/master/master/templates/visual.jpg)
   
## 打包
pyinstaller既可以将python脚本打包成Windows环境下的可执行文件，也可以打包成Linux环境下的可执行文件。打包完成后，可快速在其他环境上部署该监控服务，而不需要安装python3.7+环境和第三方包。<br>

pyinstaller安装过程自行百度，下面直接进行打包：<br>

1. 打包master<br>
    (1)安装好python环境，安装第三方包，确保程序可以正常运行；<br>
    (2)进入master文件夹，开始打包：<br>
    ```shell
    pyinstaller server.py -p draw_performance.py -p config.py -p Email.py -p logger.py -p process.py -p request.py -p __init__.py --hidden-import draw_performance --hidden-import config --hidden-import logger --hidden-import Email --hidden-import process --hidden-import request
    ```
    `打包过程可能提示缺少一些模块，请按照提示安装对应的模块`<br>
    (3)打包完成后，在当前路径下会生成dist文件夹，进入`dist/server`即可找到可执行文件`server`;<br>
    (4)将配置文件`config.ini`拷贝到`dist/server`文件夹下，并修改配置文件；<br>
    (5)将模板文件`templates`和静态文件`static`拷贝到`dist/server`文件夹下；<br>
    (6)将`dist/server`整个文件夹拷贝到其他环境，启动server
    ```shell
    nohup ./server &
    ```

2. 打包slave<br>
    (1)安装好python环境，安装第三方包，确保程序可以正常运行；<br>
    (2)进入slave文件夹，开始打包：<br>
    ```shell
    pyinstaller server.py -p performance_monitor.py -p logger.py -p config.py -p __init__.py --hidden-import logger --hidden-import performance_monitor --hidden-import config
    ```
    (3)打包完成后，在当前路径下会生成dist文件夹，进入`dist/server`即可找到可执行文件`server`;<br>
    (4)将配置文件`config.ini`拷贝到`dist/server`文件夹下，并修改配置文件；<br>
    (5)将`dist/server`整个文件夹拷贝到其他环境，启动server
    ```shell
    nohup ./server &
    ```

## 注意
1. 服务器必须支持以下命令：`jstat`、`top`、`iostat`、`netstat`、`ps`、`top`，如不支持，请安装。

2. 如果你不知道怎么在Linux服务器上安装好Python3.7+，[请点我](https://github.com/leeyoshinari/performance_monitor/wiki/Python-3.7.x-%E5%AE%89%E8%A3%85)。

3. 如果你对监控要求不高，可以使用单机版，其对第三方模块依赖较少。如需获取，请切换至`single`分支，或[点我](https://github.com/leeyoshinari/performance_monitor/tree/single)。

## Requirements
1. aiohttp>=3.6.2
2. aiohttp_jinja2>=1.2.0
3. jinja2>=2.10.1
4. matplotlib<=3.2.0
5. influxdb>=5.2.3
6. requests
7. Python 3.7+
