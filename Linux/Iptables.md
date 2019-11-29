| 参数                | 描述                                             |
| :------------------ | :----------------------------------------------- |
| -p, --protocol      | 协议，如TCP，UDP等。                             |
| -s, --source        | 可以是地址，网络名称，主机名等。                 |
| -d, --destination   | 地址，主机名，网络名称等                         |
| -j, --jump          | 指定规则的目标; 即如果数据包匹配该怎么办。       |
| -g, --goto chain    | 指定过程将在用户指定的链中继续处理。             |
| -i, --in-interface  | 命名接收数据包的接口。                           |
| -o, --out-interface | 发送数据包的接口名称。                           |
| -f, --fragment      | 该规则仅适用于分段数据包的第二个和后续片段。     |
| -c, --set-counters  | 使管理员能够以一条规则初始化数据包和字节计数器。 |



| 选项                     | 描述                                             |
| :----------------------- | :----------------------------------------------- |
| -A --append              | 将一个或多个规则添加到所选链的末尾。             |
| -C --check               | 检查与所选链中的规范匹配的规则。                 |
| -D --delete              | 从所选链中删除一个或多个规则。                   |
| -F --flush               | 逐个删除所有规则。                               |
| -I --insert              | 将一个或多个规则作为给定的规则编号插入所选链中。 |
| -L --list                | 显示所选链中的规则。                             |
| -n --numeric             | 以数字格式显示IP地址或主机名和邮政编号。         |
| -N --new-chain <name>    | 创建一个新的用户定义链。                         |
| -v --verbose             | 与list选项一起使用时提供更多信息。               |
| -X --delete-chain <name> | 删除用户定义的链。                               |

参考：

http://www.zsythink.net/archives/1199

[chrome-extension://klbibkeccnjlkjkiokjodocebajanakg/suspended.html#ttl=%E6%AF%8F%E5%A4%A9%E5%AD%A6%E4%B9%A0%E4%B8%80%E4%B8%AA%E5%91%BD%E4%BB%A4%EF%BC%9Aiptables%20Linux%20%E4%B8%8A%E7%9A%84%E9%98%B2%E7%81%AB%E5%A2%99%20%7C%20Verne%20in%20GitHub&pos=4000&uri=http://einverne.github.io/post/2017/01/iptables.html](chrome-extension://klbibkeccnjlkjkiokjodocebajanakg/suspended.html#ttl=每天学习一个命令：iptables Linux 上的防火墙 | Verne in GitHub&pos=4000&uri=http://einverne.github.io/post/2017/01/iptables.html)

https://blog.csdn.net/mafei852213034/article/details/79236800?tdsourcetag=s_pcqq_aiomsg