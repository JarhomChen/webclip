# webclip

相关介绍请查看我的blog  https://blog.jarhom.com/blog/html/16051737162084.html

# iOS Web Clip生成和签名以及发布

目前众多iOS开发者面对最多的烦恼莫过于App Store的上架问题，很多类型的App都没办法走正常方式上架，绝大部分公司和个人采用`企业签`和`超级签`的方式分发，这种方式最大问题在于掉签和成本；
接下来我要介绍的是`Web Clip（桌面便签）`的方式，原理和iPhone的safari里面直接生成便签的原理是一致的，`注意这种方式只支持纯H5类型的App`，我将介绍利用安装描述文件的方式安装Web Clip；

#1、生成.mobileconfig 文件（即Web Clip描述文件）

###1）首先， 在Mac的App Store上安装 `Apple Configurator 2`
![2020072401](https://blog.jarhom.com/md/iOS%E5%BC%80%E5%8F%91/media/15954985485962/2020072401.png)

###2）使用 Apple Configurator 2 创建 webclip 描述文件
* 打开后，点击【文件】-【新建描述文件】或者直接使用快捷键 ⌘+N 新建一个描述文件
* 选择左边菜单的【通用】输入相关信息，其中名称和标识符，就是我们常规创建 bundleID的写法，注意不要跟其他App的bundle ID 雷同，保持唯一性即可
![2020072402](https://blog.jarhom.com/md/iOS%E5%BC%80%E5%8F%91/media/15954985485962/2020072402.png)

* 选择最底下的【Web Clip】然后输入输入相关信息。其中，标签就是显示在手机桌面的名称，URL 即是点击图标要显示的链接地址，可可移除是否勾选可自行选择（勾选的话，就意味着可以在桌面长按图标点击X删除，不勾选的话，就有点流氓的味道，需要在设置的描述文件里删除），接着上传快捷方式的图标
![2020072403](https://blog.jarhom.com/md/iOS%E5%BC%80%E5%8F%91/media/15954985485962/2020072403.png)

* 到目前为止 `.mobileconfig` 文件基本创建完成。

# 2、对 `.mobileconfig`  文件进行签名 （可选）
### 1）使用苹果开发者账号生成的开发证书签名
* 选择左边的菜单【Web Clip】后，选择顶部菜单栏的【文件】，选择 【给描述文件签名...】
![2020072404](https://blog.jarhom.com/md/iOS%E5%BC%80%E5%8F%91/media/15954985485962/2020072404.jpg)

* 选择在钥匙串里面的证书进行签名，证书必须有效
![2020072405](https://blog.jarhom.com/md/iOS%E5%BC%80%E5%8F%91/media/15954985485962/2020072405.jpg)

* 签名完成，签名完成后 `.mobileconfig` 将不能修改

### 2）使用域名SSL证书签名

* 首先，你得先到相关的CA证书颁发机构申请SSL证书，证书申请我这边不再累述，可自行百度下。
我使用的是阿里云直接申请的 `DigiCert 免费版 SSL` ，进入阿里云控制台下载证书，下载Nginx使用的证书格式类型（pem）
![2020072406](https://blog.jarhom.com/md/iOS%E5%BC%80%E5%8F%91/media/15954985485962/2020072406.jpg)

* 使用下载下来的`SSL证书`进行签名，如果您想通过Apple Configurator 2 工具使用Mac的钥匙串里面的证书签名，也可以将SSL证书先导入您的Mac电脑钥匙串中，我这边直接使用openssl命令执行

```#-in ./app.mobileconfig 未签名webclip描述文件路径
#-out ~/app.mobileconfig 签名后的描述文件输出路径
#-signer ./xx.public.pem SSL证书公钥路径，包含证书链
#-inkey ./xxx.private.key SSL证书私钥路径
#-certfile  ./xx.public.pem SSL证书文件，包含证书链

openssl smime -sign -in ./app.mobileconfig -out app_signed.mobileconfig -signer ./xx.public.pem -inkey ./xxx.private.key -certfile ./xx.public.pem -outform der -nodetach
```

* 签名完成，SSL签名完成后 `.mobileconfig` 文件一样不可修改


# 3、创建Web页面对用户进行引导安装

对于大部分的用户，你得引导他下载  `.mobileconfig` 文件并安装到自己的iPhone手机上; 

以下，我随便找了个彩票分析站点使用的方法和模板进行介绍。

目录如下：
![2020072407](https://blog.jarhom.com/md/iOS%E5%BC%80%E5%8F%91/media/15954985485962/2020072407.jpg)

**app.mobileconfig** 为上面我们签名后的 .mobileconfig 文件

**app.mobileprovision** 为我们开发其他App的任何一个描述性文件，主要目的是用户打开了这个文件会自动跳转到手机【设置】的【描述文件】页面

**index.html （可选）** 这个是为了我们的app打开后，能隐藏顶部的域名网址工具栏，`原理就是创建 iframe 来加载目标页面，然后利用js语句将iframe的宽高撑满屏幕`；

```
<!DOCTYPE html>
<html lang="zh-CN">
    <head>
        <meta charset="utf-8">
        <meta http-equiv="X-UA-Compatible" content="IE=edge">
        <meta name="viewport" content="width=device-width, user-scalable=no, initial-scale=1.0, maximum-scale=1.0, minimum-scale=1.0">
        <meta name="apple-mobile-web-app-capable" content="yes" />
        <title>
            首页
        </title>

    </head>
    <body onload="refit()" style="padding:0;margin:0;">
       <iframe src="https://blog.jarhom.com/" style="width:100%;height:100%;"  frameborder="0" scrolling="yes"  id="bdIframe"></iframe>
    </body>
</html>
<script>

function refit(){

 oIframe = document.getElementById('bdIframe');
   deviceWidth = document.documentElement.clientWidth;
      deviceHeight = document.documentElement.clientHeight;
      oIframe.style.width = (Number(deviceWidth)) + 'px'; 
      oIframe.style.height = (Number(deviceHeight)) + 'px';

    }

</script>
```

**install.html** 这个是引导用户安装页面，里面就放了`app.mobileconfig` 和 `app.mobileprovision` 这两个文件的链接

引导安装页面：https://xxx.com/install.html 

安装成功后：页面加载的是 `app.mobileconfig` 中 URL 配置的链接；

注意：如果需要去掉页面头部的地址栏，需要把 `app.mobileconfig`  中 URL 配置为 `index.html` 页面地址，然后 index.html 页面中的 iframe 的 src属性设置为`app.mobileconfig` 原来的URL。

至此，整个Web Clip的生成和签名以及安装过程就完成了

**相关文件代码**：[https://github.com/JarhomChen/webclip.git](https://github.com/JarhomChen/webclip.git)

**注意：以上涉及的url地址最好全部使用https协议，不然会有警告提示。app.mobileconfig  和  app.mobileprovision 两个文件地址非https协议无法正常打开**


