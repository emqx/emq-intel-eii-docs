## 快速开始

本文将介绍如何使用 EMQ 软件基于英特尔® 边缘洞见平台收集 Modbus 协议数据并进行数据可视化。

## 目标系统要求

最低系统要求如下：

- 第8代或更高的Intel® Core™处理器，带有Intel® HD Graphics。
- 第8代或更高的Intel® Xeon®处理器。
- 至少4 GB RAM。
- 至少128 GB硬盘空间。
- 互联网连接。
- Ubuntu* 20.04

## 1: 准备工作

首先可以登录到在线的文件服务器下载并安装必要的软件。

地址:http://47.111.177.175:8080/login

用户名:**emqx**

密码:**emqxa303.**

![](../img/2.png)

1. docker image选择本地load加载方式，那么请下载这里所有的文件。

   

2. docker image在线pull拉取的方式，只下载`docker-image`目录以外的文件即可。

## 2: 安装 Docker

请将下载好的软件上传到Linux服务器上。

1. 安装docker

~~~shell
sudo chmod +x get-docker.sh
sudo ./get-docker.sh
~~~

2. 安装docker-compose

~~~shell
sudo install ./docker-compose-linux-x86_64 /usr/local/bin/docker-compose
~~~

3. 加载镜像

如果选择服务器本地load镜像,请分别load所有docker镜像，这里可以使用docker-image目录中的脚本。

~~~shell
sudo chmod +x ./load_image.sh
sudo ./load_image.sh
~~~

## 3: 运行 EII Time Series

1. 解压`IEdgeInsights.tar.gz`

    ~~~shell
    sudo tar -zxvf IEdgeInsights.tar.gz
    ~~~

2. 运行docker-compose命令启动相关docker容器，当容器都为Up状态则说明启动成功。

   ~~~shell
   cd ./IEdgeInsights/build
   sudo docker-compose up -d
   sudo docker-compose ps -a
   ~~~

   <img src="../img/3.png" style="zoom:50%;" />

4. 运行Modbus模拟器

   ~~~shell
   mv modbus_linux_amd64 modbus
   chmod +x modbus 
   ./modbus
   ~~~

## 4: 配置 Neuron 

1. 使用用户名：**admin** 和密码：**0000.** 登录到 Neuron Web 控制台 **[http://localhost:7000](http://localhost:7000/)**。

2. 添加南向 Modbus TCP 插件。

   <img src="../img/4.png" style="zoom:50%;" />
   
   添加`Modbus TCP`插件，名称设置为`Modbus`
   
   <img src="../img/5.png" style="zoom:50%;" />

3.配置Modbus驱动，填写地址、端口等信息。

<img src="../img/6.png" style="zoom:50%;" />

4.单击 **Modbus** 驱动添加组以及点位信息。可以从 **Modbus tag.xlsx** 导入预先定义好的点表

<img src="../img/7.png" style="zoom:50%;" />

5.添加北向应用，选择ekuiper。

<img src="../img/8.png" style="zoom:50%;" />



<img src="../img/9.png" style="zoom:50%;" />

6.此处保持默认配置即可。默认端口为 7081。

<img src="../img/10.png" style="zoom:50%;" />

7.然后添加eKuiper订阅，选择刚刚添加的南向Modbs驱动插件上报数据。

<img src="../img/11.png" style="zoom:50%;" />

## 5: 配置 eKuiper

1.通过 **http://localhost:9082** 登录 ekuiper Web 控制台，用户名：**admin** 密码：**public。**

<img src="../img/12.png" style="zoom:50%;" />

3.添加流：**NeuronStream**，流类型为**neuron**，选择配置组为default。

<img src="../img/13.png" style="zoom:50%;" />

设置配置组

<img src="../img/14.png" style="zoom:50%;" />

4.配置完成后，点击提交

<img src="../img/15.png" style="zoom:50%;" />

6.添加规则，规则如下。如果要处理数据，可以阅读[eKuiper文档](https://ekuiper.org/docs/zh/latest/)

<img src="../img/16.png" style="zoom:50%;" />

<img src="../img/17.png" style="zoom:50%;" />

SQL内容如下所示

~~~sql
SELECT * from NeuronStream
~~~

7.添加并配置sink output为MQTT Sink.

<img src="../img/18.png" style="zoom:50%;" />

配置MQTT broker地址，MQTT topic，MQTT clientid，MQTT 版本，qos，username，password等信息然后点击提交。

<img src="../img/19.png" style="zoom:50%;" />

## 6: 配置 EMQX

1.登录 EMQX Web 控制台 **http://localhost:18083**，用户名：**admin** 密码：**public.**首次登录会提示修改密码根据提示修改密码即可。

2.查看MQTT客户端

<img src="../img/20.png" style="zoom:50%;" />

3.新建并配置Influxdb资源，配置Influxdb等相关信息。

地址:**ia_datastore**

端口:**8086**

数据库:**datain**

用户名:**admin**

密码:**emqxa303**

<img src="../img/21.png" style="zoom:50%;" />

3.添加规则引擎添加规则并配置操作以将数据保存到 InfluxDB 数据库。

<img src="../img/22.png" style="zoom:50%;" />

这里使用SQL过滤掉一些点数据，写入到influxdb数据库中。

~~~sql
SELECT

  payload.node_name as node_name,
  payload.group_name as group_name,
  payload.timestamp as timestamp,
  payload.values.tag1 as tag1,
  payload.values.tag2 as tag2,
  payload.values.tag3 as tag3

FROM

  "NeuronData"

~~~

4.添加动作，数据持久化到InfluxDB，动作配置如下。

选择前面建好的资源，并配置写入的Measurement，timestamp，fields，tags等关键信息。

<img src="../img/23.png" style="zoom:50%;" />

<img src="../img/24.png" style="zoom:50%;" />

## 7: 配置 Grafana

1. 使用用户名：**root** 和密码：**eii123** 登录 Grafana Web 控制台 **https://localhost:3000**。

2. 导入dashboard配置。

   <img src="../img/25.png" style="zoom:50%;" />
   
   上传导入本地的**grafanna.json**

   <img src="../img/26.png" style="zoom:50%;" />

结果如下所示

<img src="../img/27.png" style="zoom:50%;" />

## 参考文档

[EMQX文档](https://docs.emqx.com/zh/enterprise/v4.4/)

[Neuron文档](https://neugates.io/docs/zh/latest/)

[eKuiper文档](https://ekuiper.org/docs/zh/latest/)

[EII文档](https://eiidocs.intel.com/)

