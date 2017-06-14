## Ngrok 搭建及https证书签名

微信公众平台二次开发时，服务器必须要能通过外网访问，而且必须是80接口。我们一般会在自己的电脑上写代码，但是由于电信运营商将80端口屏蔽了，甚至很多人通过无线路由器上网，根本就没有公网ip。在这种情况下，我们每次都要上传代码到服务器对微信公众平台进行接口调试，十分的不方便。而ngro可以将内网映射到一个公网地址，这样就完美的解决了我们的问题。

ngrok官方为我们免费提供了一个服务器，我们只需要下载ngrok客户端即可正常使用，但是后来官方的服务越来越慢，直到ngrok官网被完全屏蔽。现在我们已经无法使用ngrok官方的服务器了。所以，接下来我们自行搭建属于自己的ngrok服务器，为自己提供方便快捷又稳定的服务，一劳永逸。

### 环境准备

    #VPS：操作系统为CentOS7（64位）。
    #域名：将一个域名或二级域名泛解析到VPS服务器上。例如将*.zuduia.com解析到VPS的IP。要注意，此时还需要将tunnel.zuduia.com的A记录设置为VPS的IP。

### 1.安装GIT

    yum install git
    #注意git版本应大于1.7.9.5

### 2.配置go环境



### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
