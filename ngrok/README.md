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

    #替换默认的证书文件
    cp rootCA.pem assets/client/tls/ngrokroot.crt
    cp device.crt assets/server/tls/snakeoil.crt
    cp device.key assets/server/tls/snakeoil.key

`*若域名已经经过签名获取证书，可自行上传证书至对应文件夹中*`

### 三、开始编译ngrok

#### 1、编译服务器端ngrokd


ngrokd就是ngrok服务器端，默认编译为Linux的执行文件，我们的VPS本身就是Linux的，所以直接make编译就好。

    #编译ngrokd（服务器端）
    make release-server


显示下面的内容则表示编译成功：

    GOOS="" GOARCH="" go get github.com/jteeuwen/go-bindata/go-bindata
    bin/go-bindata -nomemcopy -pkg=assets -tags=release \
    	-debug=false \
    	-o=src/ngrok/client/assets/assets_release.go \
    	assets/client/...
    bin/go-bindata -nomemcopy -pkg=assets -tags=release \
    	-debug=false \
    	-o=src/ngrok/server/assets/assets_release.go \
    	assets/server/...
    go get -tags 'release' -d -v ngrok/...
    github.com/inconshreveable/mousetrap (download)
    github.com/rcrowley/go-metrics (download)
    Fetching https://gopkg.in/inconshreveable/go-update.v0?go-get=1
    Parsing meta tags from https://gopkg.in/inconshreveable/go-update.v0?go-get=1 (status code 200)
    get "gopkg.in/inconshreveable/go-update.v0": found meta tag main.metaImport{Prefix:"gopkg.in/inconshreveable/go-update.v0", VCS:"git", RepoRoot:"https://gopkg.in/inconshreveable/go-update.v0"} at https://gopkg.in/inconshreveable/go-update.v0?go-get=1
    gopkg.in/inconshreveable/go-update.v0 (download)
    github.com/kardianos/osext (download)
    github.com/kr/binarydist (download)
    Fetching https://gopkg.in/inconshreveable/go-update.v0/check?go-get=1
    Parsing meta tags from https://gopkg.in/inconshreveable/go-update.v0/check?go-get=1 (status code 200)
    get "gopkg.in/inconshreveable/go-update.v0/check": found meta tag main.metaImport{Prefix:"gopkg.in/inconshreveable/go-update.v0", VCS:"git", RepoRoot:"https://gopkg.in/inconshreveable/go-update.v0"} at https://gopkg.in/inconshreveable/go-update.v0/check?go-get=1
    get "gopkg.in/inconshreveable/go-update.v0/check": verifying non-authoritative meta tag
    Fetching https://gopkg.in/inconshreveable/go-update.v0?go-get=1
    Parsing meta tags from https://gopkg.in/inconshreveable/go-update.v0?go-get=1 (status code 200)
    Fetching https://gopkg.in/yaml.v1?go-get=1
    Parsing meta tags from https://gopkg.in/yaml.v1?go-get=1 (status code 200)
    get "gopkg.in/yaml.v1": found meta tag main.metaImport{Prefix:"gopkg.in/yaml.v1", VCS:"git", RepoRoot:"https://gopkg.in/yaml.v1"} at https://gopkg.in/yaml.v1?go-get=1
    gopkg.in/yaml.v1 (download)
    github.com/inconshreveable/go-vhost (download)
    github.com/alecthomas/log4go (download)
    github.com/nsf/termbox-go (download)
    github.com/mattn/go-runewidth (download)
    github.com/gorilla/websocket (download)
    go install -tags 'release' ngrok/main/ngrokd



我们可以在./bin/目录中找到文件ngrokd。可以先运行测试一下。

    #执行ngrokd
    #
    ./bin/ngrokd -domain="tunnel.mydomain.com" -httpAddr=":8080"
    #不指定httpAddr参数时默认使用80端口，可通过ngrokd -h查看帮助


出现类似以下内容，则说明我们的服务器端ngrokd正常运行了。

    [23:18:27 CST 2016/08/23] [INFO] (ngrok/log.(\*PrefixLogger).Info:83) [registry] [tun] No affinity cache specified
    [23:18:27 CST 2016/08/23] [INFO] (ngrok/log.Info:112) Listening for public http connections on [::]:8080
    [23:18:27 CST 2016/08/23] [INFO] (ngrok/log.Info:112) Listening for public https connections on [::]:443
    [23:18:27 CST 2016/08/23] [INFO] (ngrok/log.Info:112) Listening for control and proxy connections on [::]:4443
    [23:18:27 CST 2016/08/23] [INFO] (ngrok/log.(\*PrefixLogger).Info:83) [metrics] Reporting every 30 seconds

之后Ctrl+C退出ngrokd，继续来编译ngrok客户端。


#### 2、编译客户端ngrok

编译linux客户端很简单，一条命令就搞定

    #编译Linux客户端
    make release-client
    #编辑mac客户端
    GOOS=darwin GOARCH=amd64 make release-client

    #编辑windows客户端
    #切到go的安装目录
    cd /usr/local/go/src
    #给Go编译器加上交叉编译windows/amd64程序的功能
    GOOS=windows GOARCH=amd64 ./make.bash

显示以下内容，没有任何报错的话，客户端ngrok就编译成功了，我们在./bin/目录下就可以找到执行文件ngrok

    bin/go-bindata -nomemcopy -pkg=assets -tags=release \
    	-debug=false \
    	-o=src/ngrok/client/assets/assets_release.go \
    	assets/client/...
    bin/go-bindata -nomemcopy -pkg=assets -tags=release \
    	-debug=false \
    	-o=src/ngrok/server/assets/assets_release.go \
    	assets/server/...
    go get -tags 'release' -d -v ngrok/...
    go install -tags 'release' ngrok/main/ngrok


### 四、运行并测试

#### 1、启动服务器端ngrokd


    #执行ngrokd
    ./bin/ngrokd -domain="tunnel.mydomain.com"
    #默认80端口若被占用可用参数 -httpAddr=":8080" 指定其他端口。
    #若要自定义证书可通过 -tlsCrt 和 -tlsKey 参数指定自定义签名证书


#### 2、写客户端配置文件

在ngrok客户端所在目录下建立文件ngrok.cfg，用记事本等文本编辑器写入以下内容并保存。

    #配置文件ngrok.cfg的内容
    #
    server_addr: "zuduia.com:4443"
    trust_host_root_certs: false
    #zuduia.com指向生成时证书签名指向的base域名


#### 3、启动本地ngrok，映射本地HTTP port

    #启动ngrok客户端
    #注意：如果不加参数-subdomain=test，将会随机自动分配子域名。
    #
    ngrok -config=ngrok.cfg -subdomain=test 3000

正常情况下，客户端上会显示以下内容，表示成功连接到服务器端。

#客户端ngrok正常执行显示的内容

    ngrok                                                  (Ctrl+C to quit)

    Tunnel Status     online
    Version           1.7/1.7
    Forwarding        http://test.zuduia.com -> 127.0.0.1:80
    Forwarding        https://test.zuduia.com -> 127.0.0.1:80
    Web Interface     127.0.0.1:4040
    # Conn            0
    Avg Conn Time     0.00ms

打开浏览器，分别在地址栏中输入http://localhost 和http://test.zuduia.com，如果后者正常显示并且和http://localhost 显示的内容相同，则证明我们已经成功了。
