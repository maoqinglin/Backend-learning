# 一、基础

专业名词：

1）**PLMN定义**
PLMN（Public Land Mobile Network，公共陆地移动网络），由政府或它所批准的经营者，为公众提供陆地移动通信业务目的而建立和经营的网络。
PLMN = MCC + MNC，例如中国移动的PLMN为46000，中国联通的PLMN为46001

2）APN

APN在GPRS骨干网中用来标识要使用的外部PDN（Packet data network，分组数据网，即常说的Internet），在GPRS网络中代表外部数据网络的总称。APN实际上就是对一个外部PDN的标识，这些PDN包括企业内部网、Internet、WAP网站、行业内部网等专用网络。

3）APDU

插入手机(Mobile Equipment)的SIM卡与手机本身存在主从关系。ME通过数据交换协议APDU(Application Protocol Data Unit)与SIM卡进行数据的交换

ME与SIM之间的通信模式是半双工的(half-duplex)，就是说存在两个通信通道，一个是请求命令通道，另一个是数据返回通道，但两者不能同时发送数据。与此相对应，APDU有两种命令:

- Command APDU: ME向SIM卡请求某个数据时需要发送的命令；
- Response APDU: SIM响应ME的命令请求，返回给ME时的APDU.

4）SIM

**Subscriber Identity Module**（用户识别模块），**SIM卡是一个装有微处理器的芯片卡**。除了**CPU**之外，SIM卡上面还有**程序存储器ROM**、**工作存储器RAM**、**数据存储器EEPROM**，**以及串行通信单元**。

**Subscriber**和**User**，都有“用户”的意思。

通信行业里，通常用**subscriber**指代手机用户。

UIM和SIM除了技术制式上的区别之外，在外型尺寸、作用功能方面几乎没有什么不同，所以大家**通常也把UIM称为SIM**。

 SIM卡有很不错的安全性，它里面存放着用户的信息，**其中最核心的就是鉴权密钥(KI，AKey等)，网络靠这个来认证用户，**大致相当于银行卡的密码吧。这个鉴权密钥非常重要，如果泄露就会造成冒用，所以对它的保护很严格，写进卡之后无法读出来，也绝不会出现在终端和网络交互的消息中。



4）eSIM

eSIM是指**Embedded-SIM（嵌入式SIM卡）**，本质上还是一张SIM卡，只不过它变成了一颗SON-8的封装IC，直接嵌入到电路板上。

eSIM是可编程的，支持通过OTA（空中写卡）对SIM卡进行远程配置，实现运营商配置文件的下载、安装、激活、去激活及删除。

![img](https://pic3.zhimg.com/80/v2-d6324ce3c2b5e8711ee1b626ef99ef22_hd.jpg)



5）softSIM和vSIM

如果说eSIM至少还算是一个硬件，那么，softSIM和vSIM干脆一不做二不休，**彻底消灭了硬件**。

 基本特征是**终端商控制写入SoftSim的信息，可以截断用户和运营商之间的联系，改为由终端商向用户出售通信服务**。这时候运营商就被边缘化了
VSIM
• Non-physical SIM card which can simulate physical
SIM card behavior

**Remote SIM**
**<span style="color:red">Mean the card file data or authentication algorithm is kept in the server.</span>** In case of access it, we need to send the request to server side to handle it.

**Soft SIM/Local SIM**
Means the card fully data include authentication algorithm has been kept in software or local device. We don’t need to connect to server in case of try to access it.



二．Apn参数的组成
例：移动apn，把所有的属性都放在一起如下

apn carrier=”中国移动彩信 (China Mobile)”
mcc=”460”
mnc=”00”
apn=”cmwap”
proxy=”10.0.0.172”
port=”80”
mmsc=”http://mmsc.monternet.com”
mmsproxy=”10.0.0.172”
mmsport=”80”
user=”mms”
password=”mms”
type=”mms”
authtype=”1”
protocol=”IPV4V6”
/>
其对应的属性定义如下：

Carrier：apn的名字，可为空，只用来显示apn列表中此apn的显示名字。
Mcc：由三位数组成。 用于识别移动用户的所在国家;
Mnc：由两位或三位组成。 用于识别移动用户的归属PLMN。 MNC的长度（两位或三位数）取决于MCC的值。
Apn：APN网络标识（接入点名称），是APN参数中的必选组成部分。此标识由运营商分配。
Proxy：代理服务器的地址
Port：代理服务器的端口号
Mmsc：MMS中继服务器/多媒体消息业务中心，是彩信的交换服务器。
Mmsproxy：彩信代理服务器的地址
Mmsport：彩信代理服务器的端口号
Protocol：支持的协议，不配置默认为IPV4。
User：用户
Password：密码
Authtype：apn的认证协议，PAP为口令认证协议，是二次握手机制。CHAP是质询握手认证协议，是三次握手机制。


# 二、业务

无网购买套餐：

点亮本地 softsim---》与model通信，验证softsim信息---》直接向基站请求鉴权

## 1、softsim

softsim模板信息替换---》



## 2、remotesim







# 二、通信

## 1、大小端

- **大端字节序**：高位字节在前，低位字节在后，这是人类读写数值的方法。
- **小端字节序**：低位字节在前，高位字节在后，即以`0x1122`形式储存。

```java 
/* 大端字节序 */
i = (data[3]<<0) | (data[2]<<8) | (data[1]<<16) | (data[0]<<24);

/* 小端字节序 */
i = (data[0]<<0) | (data[1]<<8) | (data[2]<<16) | (data[3]<<24);
```





## 2、Netty







## 3、model





## 4、卡池







#### 方法1

Ctrl+Shift+Alt+K

#### 方法2

Code - Convert Java File To Kotlin File