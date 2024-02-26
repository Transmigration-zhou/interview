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

