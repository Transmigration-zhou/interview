[toc]

## 1. Linux 常用命令

查看端口占用：「lsof -i :端口号」

删掉进程：「kill -9 pid」

查看CPU负载：「top」

查看CPU详细情况：「lscpu」

查看CPU核数：「nproc」

查看内存占用：「free」

查看磁盘使用情况: 「df -h」

查看磁盘 IO 情况：「iotop」

查看网络 IO 情况：「iftop」

查看当前IP地址：「ifconfig 」

查看网络的连通性：「ping」「netcat(nc)」

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
  lsof -i:{端口}
  ```

- 根据进程id查看进程信息

  ```shell
  ps -ef | grep {进程id}
  ```

## 5. 统计日志文件中ERROR出现的次数

```shell
grep "ERROR“ xxx.log | wc -l
```

> wc -c filename：显示一个文件的字节数
> wc -m filename：显示一个文件的字符数
> wc -l filename：显示一个文件的行数
> wc -L filename：显示一个文件中的最长行的长度
> wc -w filename：显示一个文件的字数

## 6. chmod 权限控制

chmod u=权限,g=权限,o=权限 file

u 表示该档案的拥有者，g 表示与该档案的拥有者属于同一个群体(group)者，o 表示其他以外的人，a 表示所有（包含上面三者）。

+表示增加指定权限，-表示取消指定权限。，=表示设置指定权限，覆盖原有的权限。

r=4，w=2，x=1 。此时其他的权限组合也可以用其他的八进制数字表示出来，如： rwx = 4 + 2 + 1 = 7 rw = 4 + 2 = 6 rx = 4 +1 = 5

```shell
# 设置所有人可以读写及执行
chmod 777 file
chmod u=rwx,g=rwx,o=rwx file
chmod a=rwx file
# 设置拥有者可读写，其他人不可读写执行
chmod 600 file
chmod u=rw,g=---,o=--- file
chmod u=rw,go-rwx file
```

## 7. 查看某个端口的连接数

```shell
netstat -n | grep 80 | wc -l
```

