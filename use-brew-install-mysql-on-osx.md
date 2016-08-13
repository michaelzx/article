#mac下使用brew安装mysql
- [安装](#安装)
- [启动](#启动)
- [配置](#配置)
- [关闭](#关闭)
- [重启](#重启)
- [作为服务启动](#作为服务启动)

##安装

```bash
brew install mysql
```
下载完成以后需要编译，此时需要占用很大的cpu，风扇会狂叫，没事，完了就好

##启动
```bash
sudo mysql.server start
```
此时可能会报错：
```bash
Starting MySQL
. ERROR! The server quit without updating PID file (/usr/local/var/mysql/xxxxx.local.pid).
```
这是由于权限问题造成的
可以通用一下命令修改权限
```bash
sudo chmod -R a+rwx /usr/local/var/mysql
```
再执行
```bash
sudo mysql.server start
```
显示
```bash
Starting MySQL
. SUCCESS!
```
启动成功

##配置

启动后，需要对mysql进行一些配置，可以通过以下命令进行初始化：
```bash
/usr/local/opt/mysql/bin/mysql_secure_installation
```
##关闭
```bash
sudo mysql.server stop
```
##重启
```bash
sudo mysql.server restart
```

##作为服务启动
```bash
brew services start mysql
```
