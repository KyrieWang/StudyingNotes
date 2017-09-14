#### dns查询
---

在Linux下面，可以使用dig、host、nslookup等命令来查看dns的查询过程。比如，使用dig命令来分析一下github.com的查询过程：
> *输入命令：dig github.com*

会输出下面的信息：

![github.com](http://upload-images.jianshu.io/upload_images/7109298-0072a74ab8f8744d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

其中：
1. QUESTION SECTION这一段表示需要查询域名 github.com的A（address）记录；
2. ANSWER SECTION这一段表示dns服务器的回复，github.com有2个A记录，也就是2个IP地址；
3. AUTHORITY SECTION显示了github.com的NS（Name Server）记录，即有哪些服务器负责管理github.com的dns记录；
4. ADDITIONAL SECTION显示了AUTHORITY SECTION中的域名服务器的IP地址。

#### 域名分层
---

dns服务器使用的是分级查询的方式来查询每个域名的IP地址。

在所有域名的尾部，都有一个**根域名 .root**，平时省略了而已，比如www.github.com的真正域名是www.github.com.root。根域名的下一级，叫做**顶级域名**，比如.com、.net等；再下一级叫做**次级域名**，比如www.github.com里的.github；在下一级是主机名，比如www.github.com里的www。

所以，域名的结构是这样的：**主机名.次级域名.顶级域名.根域名**

对于每一级的域名，都有自己的NS记录，指向该级域名的域名服务器，这些服务器里保存了下一级域名的各种记录。分级查询就是从根域名开始，依次查询每一级域名的NS记录，直到找到最终的主机IP地址，也就是从根域名服务器查询到顶级域名服务器的NS记录和A记录，再从顶级域名服务器查到次级域名服务器的NS记录和A记录，再从次级域名服务器查出主机名的IP地址。

#### 分级查询的过程
---

下面还是以dig命令来分析www.github.com的DNS分级查询过程。
> 输入以下命令：*dig +trace www.github.com*

首先会列出根域名的NS记录，也就是所有的根域名服务器：

![根域名服务器](http://upload-images.jianshu.io/upload_images/7109298-f17fe7e65c1a254e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

根据根域名服务器的地址，DNS服务器想这些地址查询www.github.com的顶级域名服务器.com的NS记录，最先回复的根域名服务器会被缓存，以后只向这台服务器请求。

接下来会显示.com域名的NS记录，同时返回的还有每条记录对应的IP地址：

![.com域名服务器](http://upload-images.jianshu.io/upload_images/7109298-20ce36f8d4e18d79.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

然后DNS域名服务器向这些顶级域名服务器查询www.github.com的次级域名github.com的NS记录：

![github.com域名服务器](http://upload-images.jianshu.io/upload_images/7109298-23c7e4c59fcc3134.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

在然后，DNS服务器向上面的NS服务器查询www.github.com的主机名：

![www.github.com主机名](http://upload-images.jianshu.io/upload_images/7109298-724a0529c6a6be32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

结果表示，www.github.com有两条A记录，这两个IP地址都可以访问到github。最先返回的NS服务器是ns1.p16.dynect.net。
