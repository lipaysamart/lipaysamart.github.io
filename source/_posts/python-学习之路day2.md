---
title: 跟着AI学Python(day2)
date: 2023-04-05 21:22:49
tags:
  - Python
categories: Python
---

### 介绍
想要通过实践去提升自己的代码能力从而应用在工作当中，以下是我通过Microsoft的NewBing去获取Python中的练习题，并尝试自行解决，然后提交给Newbing检查代码。

跟着AI学习Python，让编程变得更加轻松和有趣。

### 练习
在这个学习过程中，我会跟随AI助教，每天完成AI出的三道题目，一步一步地掌握Python的基础知识和实用技巧。在实践中，我将花费30分钟的时间进行独立完成（可以Google，完成后交与AI进行批改，查看代码改进的地方，从而记录下来。以下我将会给出自己写的代码，与AI的代码, 和需要改进的地方。

#### 题目一
编写一个python脚本，可以批量检测一组服务器的运行状态，
包括CPU使用率，内存使用率，磁盘空间，网络连接等，并将结果保存到一个csv文件中。
{% tabs first %}
<!-- tab self code -->
```python
None
``` 
<!-- endtab -->
<!-- tab AI code -->
```python
# 导入需要的模块
import paramiko # 用于远程登录服务器
import psutil # 用于获取系统信息
import csv # 用于操作csv文件

# 定义一个函数，用于登录服务器并获取状态信息
def get_server_status(ip, username, password):
    # 创建一个ssh客户端对象
    ssh = paramiko.SSHClient()
    # 设置自动添加主机密钥
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    # 连接服务器
    ssh.connect(ip, username=username, password=password)
    # 创建一个sftp客户端对象，用于传输文件
    sftp = ssh.open_sftp()
    # 在服务器上创建一个临时文件，用于存储状态信息
    sftp.open("/tmp/status.txt", "w").close()
    # 在服务器上运行psutil命令，将状态信息写入临时文件
    ssh.exec_command("python -c 'import psutil; f = open(\"/tmp/status.txt\", \"w\"); f.write(str(psutil.cpu_percent())+\",\"+str(psutil.virtual_memory().percent)+\",\"+str(psutil.disk_usage(\"/\").percent)+\",\"+str(psutil.net_io_counters().bytes_sent)+\",\"+str(psutil.net_io_counters().bytes_recv))'")
    # 从服务器上下载临时文件到本地
    sftp.get("/tmp/status.txt", "status.txt")
    # 关闭sftp客户端
    sftp.close()
    # 关闭ssh客户端
    ssh.close()
    # 打开本地的临时文件，读取状态信息
    with open("status.txt", "r") as f:
        status = f.read().split(",")
        # 返回一个字典，包含服务器的ip和状态信息
        return {"ip": ip, "cpu": status[0], "memory": status[1], "disk": status[2], "sent": status[3], "recv": status[4]}

# 定义一个列表，用于存储要检测的服务器的信息，包括ip，用户名和密码
servers = [
    {"ip": "10.12.9.101", "username": "root", "password": "1"},
    {"ip": "10.12.9.200", "username": "root", "password": "1"},
    {"ip": "10.12.9.122", "username": "root", "password": "1"},
    {"ip": "10.12.9.121", "username": "root", "password": "1"}
]
# 定义一个列表，用于存储所有服务器的状态信息
status_list = []

# 遍历要检测的服务器列表，调用函数获取状态信息，并添加到状态列表中
for server in servers:
    status = get_server_status(server["ip"], server["username"], server["password"])
    status_list.append(status)

# 打开一个csv文件，用于保存状态列表中的数据
with open("status.csv", "w") as f:
    # 创建一个csv写入对象
    writer = csv.writer(f)
    # 写入表头
    writer.writerow(["ip", "cpu", "memory", "disk", "sent", "recv"])
    # 写入数据
    writer.writerows(status_list)

``` 
<!-- endtab -->
{% endtabs %}

#### 题目二
编写一个python脚本，可以使用paramiko模块远程登录到一台Linux服务器，并执行一些基本的命令，如查看系统信息，安装软件包，修改配置文件等，并将输出打印到屏幕上。
{% tabs second %}
<!-- tab self code -->
```python
# 导入需要的模块
import paramiko # 用于远程登录服务器

# 定义一个函数，用于登录服务器并执行命令
def run_command(ip, username, password, command):
    # 创建一个ssh客户端对象
    ssh = paramiko.SSHClient()
    # 设置自动添加主机密钥
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    # 连接服务器
    ssh.connect(ip, username=username, password=password)
    # 执行命令，并获取标准输出和错误输出
    stdin, stdout, stderr = ssh.exec_command(command)
    # 打印标准输出和错误输出
    print(stdout.read().decode())
    print(stderr.read().decode())
    # 关闭ssh客户端
    ssh.close()

# 定义要登录的服务器的信息，包括ip，用户名和密码
server = {"ip": "10.12.9.200", "username": "root", "password": "1"}

# 定义要执行的命令列表
commands = [
    "uname -a",
    "yum install -y wget ",
    "wget https://github.com/prometheus/node_exporter/releases/download/v1.3.0/node_exporter-1.3.0.linux-amd64.tar.gz",
    "tar xzf node_exporter-1.3.0.linux-amd64.tar.gz",
    "nohup /root/node_exporter-1.3.0.linux-amd64/node_exporter >/dev/null 2>&1 &",
]

# 遍历命令列表，调用函数执行命令，并打印分隔符
for command in commands:
    run_command(server["ip"], server["username"], server["password"], command)
    print("-"*50)
``` 
<!-- endtab -->
<!-- tab AI code -->
```python
# 导入需要的模块
import paramiko # 用于远程登录服务器

# 定义一个函数，用于登录服务器并执行命令
def run_command(ip, username, password, command):
    # 创建一个ssh客户端对象
    ssh = paramiko.SSHClient()
    # 设置自动添加主机密钥
    ssh.set_missing_host_key_policy(paramiko.AutoAddPolicy())
    # 连接服务器
    ssh.connect(ip, username=username, password=password)
    # 执行命令，并获取标准输出和错误输出
    stdin, stdout, stderr = ssh.exec_command(command)
    # 打印标准输出和错误输出
    print(stdout.read().decode())
    print(stderr.read().decode())
    # 关闭ssh客户端
    ssh.close()

# 定义要登录的服务器的信息，包括ip，用户名和密码
server = {"ip": "192.168.1.101", "username": "root", "password": "123456"}

# 定义要执行的命令列表
commands = [
    "uname -a", # 查看系统信息
    "yum install -y httpd", # 安装httpd软件包
    "sed -i 's/Listen 80/Listen 8080/g' /etc/httpd/conf/httpd.conf", # 修改httpd配置文件，将监听端口改为8080
    "systemctl restart httpd" # 重启httpd服务
]

# 遍历命令列表，调用函数执行命令，并打印分隔符
for command in commands:
    run_command(server["ip"], server["username"], server["password"], command)
    print("-"*50)
``` 
<!-- endtab -->
{% endtabs %}

#### 题目三
编写一个python脚本，可以使用requests模块调用一个RESTful API，并解析返回的JSON数据，根据不同的状态码和响应内容进行相应的处理，如打印信息，发送邮件通知，重试请求等。
{% tabs frist %}
<!-- tab self code -->
```python
None
``` 
<!-- endtab -->
<!-- tab AI code -->
```python
# 导入需要的模块
import requests # 用于调用RESTful API
import json # 用于解析JSON数据
import smtplib # 用于发送邮件
from email.mime.text import MIMEText # 用于构造邮件内容

# 定义一个函数，用于调用RESTful API，并解析返回的JSON数据
def call_api(url, headers, data):
    # 尝试发送POST请求，并获取响应对象
    try:
        response = requests.post(url, headers=headers, data=data)
    # 如果发生异常，打印异常信息，并返回None
    except Exception as e:
        print(e)
        return None
    # 如果响应状态码是200，表示请求成功
    if response.status_code == 200:
        # 尝试解析响应内容为JSON格式，并返回字典对象
        try:
            result = json.loads(response.text)
            return result
        # 如果解析失败，打印异常信息，并返回None
        except Exception as e:
            print(e)
            return None
    # 如果响应状态码是其他值，表示请求失败，打印状态码和响应内容，并返回None
    else:
        print(response.status_code)
        print(response.text)
        return None

# 定义一个函数，用于发送邮件通知
def send_email(sender, password, receiver, subject, content):
    # 创建一个MIMEText对象，用于构造邮件内容
    message = MIMEText(content, "plain", "utf-8")
    # 设置发件人，收件人和主题
    message["From"] = sender
    message["To"] = receiver
    message["Subject"] = subject
    # 创建一个SMTP对象，用于连接邮件服务器
    smtp = smtplib.SMTP_SSL("smtp.qq.com", 465)
    # 登录发件人的邮箱账号和授权码
    smtp.login(sender, password)
    # 发送邮件，并打印结果
    smtp.sendmail(sender, receiver, message.as_string())
    print("Email sent successfully.")
    # 关闭SMTP对象
    smtp.quit()

# 定义要调用的API的URL，请求头和请求体
url = "https://api.example.com/v1/user/login"
headers = {"Content-Type": "application/json"}
data = {"username": "admin", "password": "123456"}

# 定义要发送的邮件的相关信息，包括发件人，授权码，收件人，主题和内容
sender = "123456789@qq.com"
password = "abcdefg"
receiver = "987654321@qq.com"
subject = "API call result"
content = ""

# 调用函数，获取API返回的结果
result = call_api(url, headers, data)

# 如果结果不是None，表示请求成功并解析成功
if result is not None:
    # 根据不同的响应内容进行相应的处理
    # 如果响应内容中有code字段，并且值为0，表示登录成功，打印用户信息，并将内容写入邮件内容中
    if "code" in result and result["code"] == 0:
        print("Login successful.")
        print("User info:", result["data"])
        content += "Login successful.\n"
        content += "User info: " + str(result["data"]) + "\n"
    # 如果响应内容中有code字段，并且值为1，表示登录失败，打印错误信息，并将内容写入邮件内容中
    elif "code" in result and result["code"] == 1:
        print("Login failed.")
        print("Error info:", result["msg"])
        content += "Login failed.\n"
        content += "Error info: " + result["msg"] + "\n"
    # 如果响应内容中没有code字段，或者值为其他值，表示未知的响应内容，打印原始响应内容，并将内容写入邮件内容中
    else:
        print("Unknown response.")
        print("Response info:", result)
        content += "Unknown response.\n"
        content += "Response info: " + str(result) + "\n"
# 如果结果是None，表示请求失败或解析失败，打印错误信息，并将内容写入邮件内容中
else:
    print("API call failed or JSON parse failed.")
    content += "API call failed or JSON parse failed.\n"

# 调用函数，发送邮件通知，并附上邮件内容
send_email(sender, password, receiver, subject, content)
``` 
<!-- endtab -->
{% endtabs %}