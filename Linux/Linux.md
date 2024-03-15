[toc]

## 1. Linux 常用命令

查看端口占用：「lsof -i :端口号」

删掉进程：「kill -9 pid」

查看CPU负载：「top」

查看内存占用：「free」

「netstat」通常用来查询系统的网络套接字连接情况

`netstat -a`：列出所有端口
`netstat -p`：显示正在使用socket的程序PID和名称
`netstat -s`：打印统计数据
`netstat -n`：显示IP地址，而不显示域名
`netstat -c`：持续输出
`netstat -l`：只显示监听端口
`netstat -at`：列出所有tcp端口
`netstat -atp` : 列出所有tcp连接端口及其应用程序
`netstat -lt`：只显示监听的tcp端口
`netstat -ltp`：只显示监听的tcp端口及其应用程序
`netstat -catp`：持续输出所有tcp连接端口及其应用程序

## 2. 如何在 nginx 的 access log 中查出请求量前 10 的 ip

**思路：每条访问记录占一行，获取第一个字段的ip，排序后去重，再按出现次数降序排序，最后打印前十个。**

```shell
cat access.log | awk '{print $1}' | sort | uniq -c | sort -nr | head -10
```

`cat access.log` 输出日志文件内容

`awk '{print $1}'` 打印每行的第一个字段，通常是IP地址。

`uniq -c` 去重后再行首标记重复次数

`sort -nr` 逆序排序

`head -10` 打印前十个

## 3. 服务器流量异常排查步骤

1. 查看进程流量信息，确定流量最大的进程

   ```shell
   iftop -P
   ```

2. 确定该端口号对应的应用进程PID

   ```shell
   lsof -i:端口号
   ```

3. 确定进程名称

##  4. 查看进程和端口信息

- 根据进程名称查看进程信息

  ```shell
  ps -ef | grep {你要查的进程名称}
  ```

- 根据进程id查看端口信息
  ```shell
  netstat -nap | grep {进程id}
  ```

- 根据端口查看进程id

  ```shell
  netstat -tunlp | grep {进程端口号}
  ```

- 根据进程id查看进程信息

  ```shell
  ps -ef | grep {进程id}
  ```

