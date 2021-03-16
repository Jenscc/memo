##  release

```
cat /etc/centos-release
```



## port

|                                            |                                                              |
| ------------------------------------------ | ------------------------------------------------------------ |
| 查看已开放的端口                           | firewall-cmd --list-ports                                    |
| 开启80端口                                 | firewall-cmd --zone=public(作用域) --add-port=80/tcp(端口和访问类型) --permanent(永久生效) |
| 删除                                       | firewall-cmd --zone= public --remove-port=80/tcp --permanent |
| 重启防火墙,配置立即生效                    | firewall-cmd --reload                                        |
| *停止防火墙,如果要开放的端口太多可以先停止 | systemctl stop firewalld.service                             |
| 查看监听的端口                             | netstat -lnpt                                                |
| 检查端口被哪个进程占用                     | netstat -lnpt \|grep 5672                                    |
| 中止进程                                   | kill -9 6832                                                 |

firewall-cmd --zone=public --add-port=9001/tcp --permanent

## sudo

```bash
adduser xxx
```

vi /etc/sudoers

添加

```
xxx ALL=(ALL) NOPASSWD:ALL
```

