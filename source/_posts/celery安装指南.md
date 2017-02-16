---
title: celery安装指南
date: 2017-02-16 14:22:00
tags: [celery,install]
---

Celery作为一个处理消息的中间件，主要由生产者，broker，消费者三部分组成。其中broker作为一个消息转运者的角色存在，将生产者产生的消息转送给消费者。而作为消息运输者的broker目前最常用的有RabbitMQ，Redis等。
这里选用RabbitMQ，下面介绍RabbitMQ的安装方式：

1. 首先需要安装erlang
  > 1.1 添加源
    wget https://packages.erlang-solutions.com/erlang-solutions-1.0-1.noarch.rpm
    rpm -Uvh erlang-solutions-1.0-1.noarch.rpm
    1.2 安装erlang
    sudo yum install erlang

2. 安装RabbitMQ Server
  > rpm --import https://www.rabbitmq.com/rabbitmq-release-signing-key.asc
    yum install rabbitmq-server-3.6.6-1.noarch.rpm

3. 启动RabbitMQ Server
  > 随系统启动，以管理与身份运行如下命令：
    chkconfig rabbitmq-server
    手动启动或关闭，使用如下命令：
    /sbin/service rabbitmq-server stop/start

4. 使用pip安装celery
  > 如果没有安装pip，请先安装pip：
    wget "https://pypi.python.org/packages/source/p/pip/pip-1.5.4.tar.gz#md5=834b2904f92d46aaa333267fb1c922bb" --no-check-certificate
    python setup.py install
    pip install celery

