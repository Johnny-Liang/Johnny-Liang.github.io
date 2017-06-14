## Ngrok 搭建及https证书签名

微信公众平台二次开发时，服务器必须要能通过外网访问，而且必须是80接口。我们一般会在自己的电脑上写代码，但是由于电信运营商将80端口屏蔽了，甚至很多人通过无线路由器上网，根本就没有公网ip。在这种情况下，我们每次都要上传代码到服务器对微信公众平台进行接口调试，十分的不方便。而ngro可以将内网映射到一个公网地址，这样就完美的解决了我们的问题。

ngrok官方为我们免费提供了一个服务器，我们只需要下载ngrok客户端即可正常使用，但是后来官方的服务越来越慢，直到ngrok官网被完全屏蔽。现在我们已经无法使用ngrok官方的服务器了。所以，接下来我们自行搭建属于自己的ngrok服务器，为自己提供方便快捷又稳定的服务，一劳永逸。

### 一、环境准备

    #VPS：操作系统为CentOS7（64位）。
    #域名：将一个域名或二级域名泛解析到VPS服务器上。例如将*.zuduia.com解析到VPS的IP。要注意，此时还需要将tunnel.zuduia.com的A记录设置为VPS的IP。

#### 1.安装GIT

    yum install git
    #注意git版本应大于1.7.9.5

#### 2.配置go环境

    #下载go1.4.2源码包
    wget https://storage.googleapis.com/golang/go1.7.linux-amd64.tar.gz
    #解压到/usr/local/
    tar -C /usr/local/ -zxf go1.7.linux-amd64.tar.gz

可从后面的链接中下载新的版本：https://golang.org/dl/

添加环境变量

    #添加下面两行代码
    export GOROOT=/usr/local/go
    export PATH=$GOROOT/bin:$PATH

在最后面添加：

    #添加下面两行代码
    export GOROOT=/usr/local/go
    export PATH=$GOROOT/bin:$PATH

保存并退出，然后编译/etc/profile文件，使之前的配置生效

    #查看go环境是否配置成功
    #显示go version go1.4.2 linux/amd64则说明go环境配成功
    #
    go version
### 二、准备编译Ngrok

#### 1、下载Ngrok源码包

    #下载ngrok源码包
    cd ~
    git clone https://github.com/inconshreveable/ngrok.git
    cd ngrok/

#### 2、为Base域名生成自签名证书

ngrok需要一个域名作为base域名，ngrok会为客户端分配base域名的子域名。例如：ngrok的base域名为tunnel.ngrok.com，客户端即可被分配子域名test.tunnel.ngrok.com。

使用ngrok官方服务时，base域名是ngrok.com，并且使用默认的SSL证书。现在自建ngrok服务器，所以需要重新为自己的base域名生成证书。

    #为base域名tunnel.mydomain.com生成证书
    openssl genrsa -out rootCA.key 2048
    openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=输入base域名" -days 5000 -out rootCA.pem
    openssl genrsa -out device.key 2048
    openssl req -new -key device.key -subj "/CN=输入base域名" -out device.csr
    openssl x509 -req -in device.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out device.crt -days 5000

`*一定要注意对应的base域名*`

执行完上述命令，正常情况下，该目录会多出device.crt、device.csr、device.key、rootCA.key、rootCA.pem、rootCA.srl六个文件，用它们来替换默认的证书文件即可。默认的证书文件在“./assets/client/tls”和“./assets/server/tls/”目录中
