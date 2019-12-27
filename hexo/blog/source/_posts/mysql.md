---
title: mysql
date: 2019-10-31 17:18:29
tags:
---

#介绍
本文将介绍#```influxdb + grafana + telegraf#```搭建的服务器监控平台，可以监控cpu,硬盘等信息。

具体作用如下：

telegraf收集服务的各指标信息。
influxdb存储收集的信息
grafana显示收集出来的数据

另外本文都是在centos6.9系统下进行的。

安装
telegraf
telegraf是安装在被监控的服务器上的。
具体各平台的安装可参考官方文档
这里仅介绍centos的安装方法

新建yum安装文件

```vim /etc/yum.repos.d/influxdb.repo```
在文件中写入一下内容：

[influxdb]
name = InfluxDB Repository - RHEL \$releasever
baseurl = https://repos.influxdata.com/rhel/\$releasever/\$basearch/stable
enabled = 1
gpgcheck = 1
gpgkey = https://repos.influxdata.com/influxdb.key
执行安装

sudo yum install telegraf
安装成功，如下图


image
启动服务：

sudo service telegraf start
image

验证安装结果：

telegraf --version
image
influxdb安装
influxDB的安装可以参考官方的安装方法
官方安装文档

我这边使用的是centos，所以下面我会把Centos下的安装过程记录下来。
在Centos终端依次执行以下命令即可：

wget https://dl.influxdata.com/influxdb/releases/influxdb-1.3.7.x86_64.rpm
sudo yum localinstall influxdb-1.3.7.x86_64.rpm
image

image

image
安装完成后，执行启动服务的命令：

sudo service influxdb start
image

至此，我们的influxDB便是安装完成了。

grafana安装
可参考官方安装文档

wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana-4.6.2-1.x86_64.rpm 
sudo yum localinstall grafana-4.6.2-1.x86_64.rpm
安装成功：


image
启动服务（centos7）

systemctl start grafana-server
然后即可通过IP：3000端口访问了。


image

默认管理员的用户名和密码为
admin
admin

配置
telegraf配置
配置文件路径：

/etc/telegraf/telegraf.conf
编辑该配置文件

数据源配置(outputs.influxdb节点)：
[[outputs.influxdb]]
  urls = ["http://192.168.18.118:8086"]  #infulxdb地址
  database = "telegraf" #数据库
  precision = "s"
  timeout = "5s"
  username = "admin" #帐号
  password = "admin" #密码
cpu配置(inputs.cpu节点):
[[inputs.cpu]]
  ## Whether to report per-cpu stats or not
  percpu = true
  ## Whether to report total system cpu stats or not
  totalcpu = true
  ## If true, collect raw CPU time metrics.
  collect_cpu_time = false
  ## If true, compute and report the sum of all non-idle CPU states.
  report_active = false
硬盘配置（inputs.disk）
  ## By default, telegraf gather stats for all mountpoints.
  ## Setting mountpoints will restrict the stats to the specified mountpoints.
  # mount_points = ["/"]

  ## Ignore some mountpoints by filesystem type. For example (dev)tmpfs (usually
  ## present on /run, /var/run, /dev/shm or /dev).
  ignore_fs = ["tmpfs", "devtmpfs", "devfs"]
内存配置（inputs.mem）
# no configuration
# Read metrics about system load & uptime
配置完成后重启telegraf

service telegraf restart
grafana配置
至此，我们的telegraf已经能完整的采集数据，并且将数据录入到influxdb中，接下来就是配置grafana来显示数据。
首先我们要用管理员账户登录grafana。登录之后大致是这样的，如果是第一次登录界面，可能稍稍有点不一样。


image
添加datasource
点击左上角的logo,弹出菜单，选择DataSource，如下图：


image
完成上面步骤之后，进入以下界面，并点击add data source按钮。
image
进入add data source界面
image

name: datasource别名，自己随便定义
type: 数据库类型，这里选择influxdb即可
url: 即influxdb的地址，http://xxx.xxx.xxx:8086
access: 可选择直连和代理，我这里就直接选择直连了。
HTTP Auth: http 认证，可不填，保持默认。
image

database: influxdb中的数据库名称
user: influxdb的用户名
password: influxdb的用户密码
最后输入完成之后点击add按钮即可添加数据源。
添加dashboard
按照下图顺序，可配置该dashboard基本信息


image

image
点击左上角的logo,弹出菜单，选择Dashboards下的+ New，如下图:
image
进入以下界面，点击gragh, 新增图表


image
之后进入以下界面，下图即是新建的图表，但是现在这个图表还缺少数据，接下来就是数据的编辑，如下图，在图表的title部位单击，会弹出个菜单，选择其中的edit进行编辑。
image
编辑的界面如下：


image

general: 一般配置
metrics: 数据配置
axes: x,y坐标配置
legend: 图例配置
display: 显示配置
alert: 警告配置
下面分别介绍下上述的配法

general
image

如上图，下面就重要栏位解释下
title: 图表标题
Description: 图表描述
span: 宽度，将屏幕平分12等份，此栏位输入所占比例
height: 高度，单位px

metrics
image
如上图，

选择已添加过的数据源
编写查询条件区域
菜单按钮
是否可用开关
删除查询语句
新增查询语句
可以看到2区域的查询语句是通过选择选取的，当然也可以直接输入sql语句即可，见下图，点击菜单按钮，切换编辑模式，即可直接手写sql语句。


image
Axes
image

如图：
LeftY: 左y轴
RightY: 右y轴
X-Axis: x轴
在y坐标上可配置是否显示，单位，刻度，最小值，最大值，小数点精确位数，已经显示文本。
x轴上可配置是否显示，以及x轴模式。

Display
image

如图，display可以配置显示的样式等。

Alert
image

如图，Alert可配置一些预警条件。

完成
最后结果截图
![Image text](https://img-blog.csdn.net/20180419165758798)


