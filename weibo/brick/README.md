# 微博平台性能挑战赛

## 规则

```
本次挑战赛借用阿里中间件团队的题目，题目叫码农来搬砖，题目的要求为：
实现一个客户端和一个服务器端，客户端把服务器端的一个1G的文件搬到客户端，文件中的每行字符串为一块砖头，字符串由随机的ascii32-127的字符组成，每行的长度为随机的1-200字节；
服务端必须是单进程并且只能监听一个端口；
客户端每个发起获取砖头的请求“线程“必须是一问一答，类似这样：
砖头 result = client.getZhuanTou();
服务端收到请求后，必须是按顺序获取下一块砖头，类似这样：
砖头 next = server.getNext();
不允许批量处理请求，也不允许服务端处理请求的”线程“批量返回砖头，服务端处理请求的”线程“每次只能处理一个请求，并返回一块砖头；
服务端或客户端需要对砖头进行处理，处理方式为去掉行中间的三分之一字符（从size/3字符开始去掉size/3个字符，除法向下取整)后将剩余部分以倒序的方式传输 例如 123456789 => 123789 => 987321
每块砖头需要标上序号，例如上面的123456789是第5行，那么最后的砖头结果应该为：5987321；
客户端需要顺序的输出最终处理过的砖头内容到一个文件中，此文件中的砖头的顺序要和服务端的原始文件完全一致，文件不需要写透磁盘（例如java里就是可以不强制调sync）；
不允许采用内核层面的patch；
不建议采用通信框架，例如netty之类的这种；
不限语言、通信协议和连接数。

比赛的运行方式：
服务端启动5s后启动客户端，客户端启动就开始计时，一直到客户端搬完所有砖头并退出计算为耗时时间；
每次启动前清除pagecache。

资源准备：
1，会准备20个虚拟机，给大家做测试及优化使用
2，准备10台机器，分5套来做验收环境，指定5个人来帮忙负责验收程序执行

验收方式：
服务端：
/data1/public/t.txt    为准备好的数据文件
/data1/$Email$ :   按各自email开头，如zhulei ，放置自己的需要执行的代码以及脚本，需各自编写启动脚本，如start.sh
tools.sh为公用准备的服务端启动脚本，每次会清除pagecache，如tools.sh start.sh 来启动服务端程序

客户端：
客户端在服务端启动5秒后启动
/data1/$Email$ 为各自运行的程序及启动脚本，各自接受的文件存储在  /data1/$Email$/data/a.txt 下
tools.sh为公用准备的客户端启动脚本，每次会清除pagecache,； 删除/data1/$Email$/data目录下得文件；启动客户端，并计时；对每次执行的耗时汇报到一个Dashboard
```

## V1版本：简单UDP应答，单进程，单线程，Channel通信

### 性能情况
```
Connecting to server at  127.0.0.1:1200
Connected to server at  127.0.0.1:1200
Begin to transfer brick
Transfer OK
receive =  785825619  Bytes
elapsed =  8m54.883491135s  us
bandwidth =  1.4010933641345003  MB/s
```
