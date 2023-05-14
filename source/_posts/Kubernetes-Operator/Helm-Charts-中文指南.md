---
title: Helm Charts 中文指南
date: 2023-03-24 18:18:59
tags:
  - Helm
  - Kubernetes
thumbnail:
categories: Kubernetes-Operator
---

### Helm包管理
{% tabs helm %}
<!-- tab helm指令参数 -->
```sh
helm completion                         # 生成指定shell的自动补全脚本
helm create                             # 用给定的名称创建一个新的图表
helm dependency                         # 管理图表的依赖关系
helm env                                # helm客户端环境信息
helm get                                # 下载命名发布的扩展信息
helm history                            # 获取发布历史
helm install                            # 安装一个图表
helm lint                               # 检查一个图表是否有可能的问题
helm list                               # 列出发布
helm package                            # 将一个图表目录打包成一个图表存档
helm plugin                             # 安装、列出或卸载Helm插件
helm pull                               # 从仓库下载一个图表并（可选地）在本地目录解压它
helm push                               # 将一个图表推送到远程
helm registry                           # 登录或登出一个注册表
helm repo                               # 添加、列出、移除、更新和索引图表仓库
helm rollback                           # 将一个发布回滚到之前的修订版
helm search                             # 在图表中搜索一个关键词
helm show                               # 显示一个图表的信息
helm status                             # 显示命名发布的状态
helm template                           # 在本地渲染模板
helm test                               # 运行一个发布的测试
helm uninstall                          # 卸载一个发布
helm upgrade                            # 升级一个发布
helm verify                             # 验证给定路径的图表是否已经签名并且有效
helm version                            # 打印客户端版本信息
```
<!-- endtab -->
<!-- tab helm简单示例 -->
```Yaml
helm upgrade <RELEASE> <CHART> <flags>  # 更新release配置
helm rollback <RELEASE> <REVISION>      # 回滚升级前的版本 
helm get <COMMEND> <RELEASE_NAME>       # 获取release的配置
helm history [RELEASE]                  # 查看历史版本号
helm template [NAME] [CHART] [flags]    # 用于在本地渲染图表模板并显示输出
```
<!-- endtab -->
<!-- tab helm常用用法 -->

<!-- endtab -->
{% endtabs %}

#### Template模板文件
所有模板文件都存储在 chart 的 templates/ 目录下面，当 Helm 渲染 charts 的时候，它将通过模板引擎传递该目录中的每个文件。模板的 Values 可以通过两种方式提供：

* Chart 开发人员可以在 chart 内部提供一个名为 values.yaml 的文件，该文件可以包含默认的 values 值内容。
* Chart 用户可以提供包含 values 值的 YAML 文件，可以在命令行中通过 helm install 来指定该文件。
当用户提供自定义 values 值的时候，这些值将覆盖 chart 中 values.yaml 文件中的相应的值。

### Charts文件结构
chart 被组织为一个目录中的文件集合，目录名称就是 chart 的名称（不包含版本信息）下面是一个 WordPress 的 chart  
{% tabs second helm %}
<!-- tab 文件结构 -->
```Yaml
wordpress/
  Chart.yaml                        # 包含当前 chart 信息的 YAML 文件
  LICENSE                           # 可选：包含 chart 的 license 的文本文件
  README.md                         # 可选：一个可读性高的 README 文件
  values.yaml                       # 当前 chart 的默认配置 values
  values.schema.json                # 可选: 一个作用在 values.yaml 文件上的 JSON 模式
  charts/                           # 包含该 chart 依赖的所有 chart 的目录
  crds/                             # Custom Resource Definitions
  templates/                        # 模板目录，与 values 结合使用时，将渲染生成 Kubernetes 资源清单文件
  templates/NOTES.txt               # 可选: 包含简短使用使用的文本文件
```
<!-- endtab -->

<!-- tab charts.yaml示例 -->
```Yaml
apiVersion: chart API 版本 (必须)
name: chart 名 (必须)
version: SemVer 2版本 (必须)
kubeVersion: 兼容的 Kubernetes 版本 (可选)
description: 一句话描述 (可选)
type: chart 类型 (可选)
keywords:
  - 当前项目关键字集合 (可选)
home: 当前项目的 URL (可选)
sources:
  - 当前项目源码 URL (可选)
dependencies: # chart 依赖列表 (可选)
  - name: chart 名称 (nginx)
    version: chart 版本 ("1.2.3")
    repository: 仓库地址 ("https://example.com/charts")
maintainers: # (可选)
  - name: 维护者名字 (对每个 maintainer 是必须的)
    email: 维护者的 email (可选)
    url: 维护者 URL (可选)
icon: chart 的 SVG 或者 PNG 图标 URL (可选).
appVersion: 包含的应用程序版本 (可选). 不需要 SemVer 版本
deprecated: chart 是否已被弃用 (可选, boolean)
```
<!-- endtab -->
{% endtabs %}



##### 模板文件
```Yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: deis-database
  namespace: deis
  labels:
    app.kubernetes.io/managed-by: deis
spec:
  replicas: 1
  selector:
    app.kubernetes.io/name: deis-database
  template:
    metadata:
      labels:
        app.kubernetes.io/name: deis-database
    spec:
      serviceAccount: deis-database
      containers:
        - name: deis-database
          image: {{ .Values.imageRegistry }}/postgres:{{ .Values.dockerTag }}     # docker镜像仓库 &docker 镜像 TAG
          imagePullPolicy: {{ .Values.pullPolicy }}  # 镜像拉取策略
          ports:
            - containerPort: 5432
          env:
            - name: DATABASE_STORAGE
              value: {{ default "minio" .Values.storage }}  # 存储后端默认设置为 'minio'
```

##### 为模板提供一些必须的 values 值的 values.yaml 文件如下所示：

```Yaml
imageRegistry: "quay.io/deis"
dockerTag: "latest"
pullPolicy: "Always"
storage: "s3"
```

values 文件的格式是 YAML，一个 chart 包可能包含一个默认的 values.yaml 文件，helm install 命令允许用户通过提供其他的 YAML 值文件来覆盖默认的值：

`helm install --values=myvals.yaml wordpress`
用这种方式来传递 values 值的时候，它们将合并到默认值文件中，比如有一个 myvals.yaml 文件如下所示：

`storage: "gcs"`
将其与 chart 的 values.yaml 文件合并后，得到的结果为：
```Yaml
imageRegistry: "quay.io/deis"
dockerTag: "latest"
pullPolicy: "Always"
storage: "gcs"
```
我们可以看到只有最后一个字段被覆盖了。