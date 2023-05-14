---
title: Linux sed 指令集
date: 2023-03-20 21:04:50
tags:
  - sed
catagories: Linux-System-Operator
thumbnail:
---
{% notel primary Introduce%}
sed命令是一种非常强大和灵活的文本处理工具，它可以让您快速地对文本进行各种编辑操作，而无需打开一个交互式的文本编辑器。sed命令可以在管道中使用，也可以直接修改原文件，非常适合批量处理大量的文本数据。sed命令还支持正则表达式，可以让您更精确地匹配和替换文本。如果您想要提高您的Linux技能和效率，那么学习sed命令是一个不错的选择。
{% endnotel %}
---
### Sed常用指令
{% tabs sed tab %}
<!-- tab 参数说明 -->
* -a    新增， a 的后面可以接字串，而这些字串会在新的一行出现(目前的下一行)～
* -c    取代， c 的后面可以接字串，这些字串可以取代 n1,n2 之间的行！
* -d    删除，因为是删除啊，所以 d 后面通常不接任何东东；
* -i    插入， i 的后面可以接字串，而这些字串会在新的一行出现(目前的上一行)；
* -p    打印，亦即将某个选择的数据印出。通常 p 会与参数 sed -n 一起运行～
* -s    替换，可以直接进行替换！通常这个 s 的动作可以搭配正则表达式

<!-- endtab -->
<!-- tab 使用实例 -->
```sh
    sed 5a\newline filename             # 在filename文件第5行的下一行插入newline,将结果输出到标准输出
    nl filename | sed '3,7d'            # <nl>带行号输出<filename>sed动作是删除<d> 3-7行,也可以夹带正则
    nl filename | sed '8a after hello word'         # 在第八行的结尾加上 after hello word 字样
    nl filename | sed '8i first hello word'         # 在第八行前加上 first hello word 字样
    nl filename | sed -n '3,8p'                         # 列出filename文件内的<3-8>行信息
    nl filename | sed -n '/hello/p'                     # 列出filename文件有hello关键词的行
    nl filename | sed -n '/hello/d'                     # 删除filename文件有hello关键词的行
    nl filename | sed -n '/hello/{s/hello/hi/;p;q}/'      #找到hello关键词替换为hi <q>退出
    sed 's/Oldstring/Newstring/g'  filename    # 替换filename文件中的<Oldstring>为<Newstring> <g>全局替换
    sed -i 's/Oldstring/Newstring/g'  filename          # 使其修改生效
    sed -i 's/Oldstring/Newstring/g'  path/to/file.*    #批量替换目录下file开头的文件
```
<!-- endtab -->
{% endtabs %}