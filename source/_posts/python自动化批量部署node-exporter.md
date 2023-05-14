---
title: python自动化批量部署node_exporter
date: 2023-04-09 18:12:01
tags:
  - monitor
  - Python
categories: Python
---
#### 介绍&环境
node_exporter是一个用Go编写的可插拔的度量收集器，它可以暴露*NIX内核和硬件相关的度量指标1。它可以通过HTTP端口9100（默认）监听和提供度量数据2。它可以被Prometheus实例配置和抓取，以监控主机系统的性能和状态。 **Centos7**

#### 自动化部署
我们需要定义一个主程序文件main.py和存储服务器信息的文件remote_server.py
{% tabs first %}
<!-- tab main -->
```python
# 主程序，使用多线程来批量登录并执行命令
import paramiko
import threading
from remote_server import servers_ex # 导入服务器信息

def run_command(server):
    # 定义一个函数，用来连接服务器并执行命令
    ssh = paramiko.SSHClient()
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    ssh.connect(server['hostname'], server['port'], server['username'], server['password'])
    # 这里可以根据需要修改要执行的命令
    for command in commands:
        # 执行命令，并获取标准输出和错误输出
        stdin, stdout, stderr = ssh.exec_command(command)
        # 打印标准输出和错误输出
        output = stdout.read()
        print(output)
    # 关闭ssh客户端
    ssh.close()

commands = [
    "uname -a",
    "hostname",
    "yum install -y wget ",
    "wget https://github.com/prometheus/node_exporter/releases/download/v1.3.0/node_exporter-1.3.0.linux-amd64.tar.gz",
    "tar xzf node_exporter-1.3.0.linux-amd64.tar.gz",
    "nohup /root/node_exporter-1.3.0.linux-amd64/node_exporter >/dev/null 2>&1 &",
]

threads = [] # 创建一个空列表，用来存储线程对象
for server in servers_ex: # 遍历服务器列表
    t = threading.Thread(target=run_command, args=(server,)) # 为每个服务器创建一个线程，并传入服务器信息作为参数
    threads.append(t) # 将线程对象添加到列表中

for t in threads: # 遍历线程列表
    t.start() # 启动每个线程

for t in threads: # 遍历线程列表
    t.join() # 等待每个线程结束

print('All done') # 所有线程结束后打印提示信息
```
<!-- endtab -->
<!-- tab server -->
```python
servers_ex = [
    {"hostname": "10.12.9.101", "port": "22", "username": "root", "password": "1", },
    {"hostname": "10.12.9.121", "port": "22", "username": "root", "password": "1", },
    {"hostname": "10.12.9.122", "port": "22", "username": "root", "password": "1", },
    {"hostname": "10.12.9.200", "port": "22", "username": "root", "password": "1", },
]
```
<!-- endtab -->
{% endtabs %}

#### 运行结果
登录服务器查看 `ps -ef | grep node_exporter `
![link](https://cdn.jsdelivr.net/gh/lipaysamart/Blog-pic/20230409183436.png)

#### Prometheus添加监控
```yaml
- job_name: "node-exporter"
  static_configs:
    targets:
      - 10.12.9.101:9100
      - 10.12.9.122:9100
      - 10.12.9.121:9100
      - 10.12.9.200:9100
```
#### 优化脚本
有时候node_exporter里面当中有很多我们不需要采集的参数，这时候我们就可以把它们进行丢弃或者只采集我们想要的指标，以保证我们的程序占用较少资源，减轻Prometheus的查询负载。
修改exec_command：
`nohup /root/node_exporter-1.3.0.linux-amd64/node_exporter --no-collector.hwmon --no-collector.nfs --no-collector.nfsd --no-collector.nvme --no-collector.dmi --no-collector.arp >/dev/null 2>&1 &`

