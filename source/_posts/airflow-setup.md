---
title: Airflow安装部署
date: 2019-06-03 15:21:03
tags:
- airflow
- 调度
- 大数据
---

## 1. 环境安装

### 1.1 Anaconda3
[Anaconda](https://www.anaconda.com/distribution/)用来管理python环境。
```bash
./Anaconda3-2019.03-Linux-x86_64.sh
```
<!--more-->
将conda环境变量加入path中,在`~/.bashrc`最后添加
```bash
export PATH=$PATH:~/anaconda3/bin
conda deactivate
```

### 1.2 Redis
[Redis](https://redis.io/download)用来作为消息队列支持airflow分布式部署。
```bash
TODO
```


## 2. Airflow安装

### 2.1 python包安装
```bash
conda create -n airflow pip
conda activate airflow
pip install apache-airflow
```
> 此时需要预先安装gcc编译器。
> * Ubuntu
> ```bash
> sudo apt-get install build-essential
> ```
> 如果没有源，改源为阿里的 `vim /etc/apt/source.list`
> ```
> deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse \
> deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse 
>
> deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse \
> deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
> 
> deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse \
> deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse \
> deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse \
> deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse \
> deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse \
> deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse 
> ```
> 更新apt-get后重新安装gcc。
> ```bash
> sudo apt-get update
> sudo apt-get upgrade
> ```
> 

## 2.2 Airflow时区调整
Airflow默认使用UTC时区，包括在webUI中的显示。目的是为了让多时区的协同工作不会遇到问题。
想要放弃该功能使用当前地区的的时间显示，目前只能修改源码。
1. 在AIRFLOW_HOME目录下修改airflow.cfg
```ini
default_timezone=Asia/Shanghai
```
2. 进入airflow的python安装目录。
```bash
cd $CONDA_HOME/envs/airflow/lib/python3.7/site-packages/airflow
```
3. 修改airflow/utils/timezone.py，在`utc = pendulum.timezone('UTC')`这行(第27行)代码下添加,
```python
from airflow import configuration as conf
try:
    tz = conf.get("core", "default_timezone")
    if tz == "system":
        utc = pendulum.local_timezone()
    else:
        utc = pendulum.timezone(tz)
except Exception:
    print("timezone.py config set failed!")
```
修改utcnow()函数 (在第69行)
```python
#d = dt.datetime.utcnow()  modified to ->
d = dt.datetime.now()
```

4. 修改airflow/utils/sqlalchemy.py，在`utc = pendulum.timezone(‘UTC’)` 这行(第37行)代码下添加。
```python
from airflow import configuration as conf
try:
    tz = conf.get("core", "default_timezone")
    if tz == "system":
        utc = pendulum.local_timezone()
    else:
        utc = pendulum.timezone(tz)
except Exception:
    print("sqlalchemy.py config set failed!")
```

5. 修改页面右上角时钟，修改airflow/www/templates/admin/master.html(第31行)
```javascript
//var UTCseconds = (x.getTime() + x.getTimezoneOffset()*60*1000);  modified to ->
var UTCseconds = x.getTime();

//"timeFormat":"H:i:s %UTC%",  modified to ->
"timeFormat":"H:i:s",
```

### 2.3 Airflow配置
TODO


## 3. Airflow的使用

### 3.1 启动Airflow
初始化DB
```bash
airflow initdb
```
运行webUI
```bash
airflow webserver [-p 8080]
```
开启调度
```bash
airflow scheduler
```

