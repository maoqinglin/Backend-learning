## 1、Please select Android SDK

从github clone 代码到本地放到AS后发现，发现并不能点“Run”键运行app，并报错Error:Please select Android SDK：

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170614190154669?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2lsc2NoYW4wMjAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)


最后在File->Project Structure中将Build tools version修改，问题解决。

![è¿éåå¾çæè¿°](https://img-blog.csdn.net/20170614190034279?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvd2lsc2NoYW4wMjAx/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

