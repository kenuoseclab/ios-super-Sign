# -
### 超级签名，superSign，无需mac服务器

# 环境需求：

    系统环境：Center OS 7.2 * 1
    系统配置：2核4G 500G以上硬盘
    带宽：5M
    域名：已备案域名 * 1
    SSL证书：域名对应的SSL证书 * 1
    阿里云OSS： *1

### 与其使用别人的签名系统不如自己搭一套。证书安全也有保障，成本也低上很多。

由于近段时间政策变动，企业证书不断被封停。
市场上开发出一种新的签名方式，超级签名。

惯例，先上效果图：
![](https://raw.githubusercontent.com/wiki/oopww1992/ios-super-Sign/1.jpg)
![](https://raw.githubusercontent.com/wiki/oopww1992/ios-super-Sign/2.jpg)


签名原理
签名原理简单点说使用了苹果提供给开发者的Ad-Hoc分发通道，把安装设备当做开发设备进行分发。
既然签名用是 Ad-Hoc ，那么 Ad-Hoc 所具有的优劣势也一并继承了下来：
优势：
直接分发，安装即可运行，不需要用户做企业证书的信任操作  目前稳定，不会有证书吊销导致的业务风险(后续苹果政策风险非常高)
缺点：
单开发者账号的iPhone设备数量只有100个，导致分发成本非常高(99美元/1年/100个设备) 开发者账号需要预先写入安装设备的UDID，在工具链不通的情况下，获取用户的UDID相对困难和繁琐，而且手动写入UDID不存在商用可行性，当然目前这个缺点被解决了
整体架构
接下来我们就看看整套机制是如何进行的：
图片: https://uploader.shimo.im/f/ckMU4IvjMvcsaATL.png
1.设备安装描述文件后，会向服务器发送设备的UDID。2.服务器收到UDID后，将UDID注册到某个开发者账号下。3.再生成签名用的描述文件，给IPA签名。4.然后iPA传Server，使用itms-services方式让用户下载。
技术细节
1，获得用户UDID
苹果允许用户通过浏览器安装配置文件，来上传UDID到服务器。
服务器需返回301重定向到特定网站
.mobileConfig 文件实例：
图片: https://uploader.shimo.im/f/l1Rs6um7IEoKHpi9.png
获得udid
服务端接受UDID需要返回301，示例代码：
2，解析苹果给出的XML文件，并取得对应数据

图片: https://uploader.shimo.im/f/3JngchLiFOE1hXD7.png
解析XML并返回
在这个时候，服务器就已经有了用户的udid了。 
3，添加apple账号。
因为苹果存在二次验证，这里通过后台弹窗输入验证码。
图片: https://uploader.shimo.im/f/V3gomKRaC5IQG0G4.png
4，开发者中心更新UDID。
这里用到的框架是fastlane。框架使用起来简单易用。
调用实例代码：
图片: https://uploader.shimo.im/f/BjlLj0At9gUdStQd.png
更新UDID
更新完成UDID后，从苹果商店下载相关证书，并准备重签名。
5，重签名
看了很多文章，很多都只能运行在mac电脑上。  mac服务器成本昂贵。要支持高并发成本非常高。
在这里，我们使用isign来实现在linux服务器上也能重签名。
可能有人会说他们使用isign在ios13上无法安装，这里需要自己修复一下源码的bug
实例代码：
图片: https://uploader.shimo.im/f/lcal3uINBzkMN1h5.png
重签名
到此时，我们的重签名流程就已经完成了。
然后将ipa包上传到OSS服务上，并配置itms-service服务来做分发。
itms-service服务代码实例：
图片: https://uploader.shimo.im/f/cZp8slvQjl864RkD.png
itms-service服务稳健
tip：
另外，如果要通过isign实现企业的签名的话，因为isign的入参是pem文件，在这里，需要将企业证书的P12文件转换为pem文件。实例代码如下：

图片: https://uploader.shimo.im/f/KdBWoBjbMMcFzOcu.png
p12转pem
我们现在已经完成了全部流程。 大家可以加我qq 397294213 一起沟通成长。
图片: https://uploader.shimo.im/f/kfoRTA6enJICAwXM.png
后台

图片: https://uploader.shimo.im/f/DGLOR7m03jw5qUEZ.png
落地页
