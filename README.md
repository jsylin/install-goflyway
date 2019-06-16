# install-goflyway
install goflyway on my vps
goflyway的客户端与服务端均为同一个文件，下载对应系统版本即可运行。以下内容对应v1.0.11b以上版本，并假设服务端IP为1.2.3.4，监听在8100端口上。  

从源码编译  
goflyway基于golang，所以请首先安装go环境并正确配置$GOPATH和$GOROOT路径（参考）。然后：  

go get -u github.com/coyove/goflyway/cmd/goflyway  
完成后goflyway会被编译到$GOPATH/bin目录下，您也可以将$GOPATH/bin这个路径加入到$PATH中以直接调用goflyway。  
  
对于客户端，请将$GOPATH/src/github.com/coyove/goflyway/chinalist.txt这个文件拷贝至goflyway的同目录下，但是这一步不是必须的：在缺乏chinalist.txt文件的情况下，goflyway只会通过IP判断国内外流量，而chinalist可以更快的匹配（无匹配时才会查询IP）。

运行  
在服务器上执行  
  
./goflyway -k=KEY  
即可启动服务端，KEY为自定义密码，默认监听8100。本地执行  
   
./goflyway -k=KEY -up="1.2.3.4:8100"  
启动客户端连接。  
  
客户端启动后，请设置本地代理为127.0.0.1:8100（协议为HTTP或SOCKS5代理）。8100端口为默认值，可以使用-l="ip:port"或-p="ip:port"命令修改。
  
对于服务端，-l同样用于修改其监听地址，请不要忘了同时修改客户端-up命令中的端口号。  
  
简单认证  
在服务端和客户端使用-a username:password启用用户认证（username和password请自行设定，不要忘了中间的冒号），开启后您需要用户名和密码才能够连接客户端的HTTP或SOCKS5代理。
  
全局代理  
客户端使用-g开启全局代理，也可以从web控制台切换。从web控制台开启全局代理后，已缓存的域名不受全局影响。  
  
半加密  
客户端可以使用-partial开启半加密模式，这样HTTPS传输的数据只有前18kb会被加密以降低CPU占用。HTTP流量不受该命令影响。如果使用SOCKS5代理，请不要使用该命令。
  
中间人模式  
请将ca.pem添加至系统的信任证书列表中，然后使用命令:  
  
-up="http://1.2.3.4:8100"  
打开中间人模式，注意该模式只有在使用HTTP代理时有效，不支持HTTP2，不支持文件的下载进度显示。中间人模式下HTTPS和HTTP均会通过HTTP请求发出，但是body和一些关键的HTTP头会被加密。开启中间人模式后WebSocket默认不可用。
  
前置代理  
中间人模式下您可以使用公共的HTTP代理服务器作为前置，以达到加速或隐藏IP的目的。这里我们假设域名example.com已经指向1.2.3.4，代理服务器10.20.30.40:80。客户端使用以下命令连接：
   
-up="http://[username:password@]10.20.30.40:80/example.com:8100"  
（代理仅支持Basic验证，不支持NTLM等方法）   
  
cmd\mitm-relay是一个特殊的HTTP代理服务器脚本，其可以部署在BAE、SAE之类的Cloud App Engine上，依靠托管在国内服务器上的优势，达到加速客户端连接服务端速度的目的。以BAE为例，假定域名为example.duapp.com，客户端使用以下命令连接：
  
-up="fwd://example.duapp.com:80/1.2.3.4:8100"  
CloudFlare  
如果example.com使用了cloudflare加速，服务端需运行在80/8080等指定端口上，客户端使用以下命令连接（请不要遗漏:80端口）：
  
-up="cf://example.com:80"  
WebSocket 连接模式  
若需要通过WebSocket协议与服务端通信，客户端使用以下命令连接：
  
-up="ws://1.2.3.4:8100"  
若连接 至cloudflare，则默认已经使用了websocket模式。WebSocket中继可以使用该脚本。  
  
HTTPS 前置代理  
goflyway可以使用支持CONNECT命令的HTTPS代理服务器作为前置来连接服务端，HTTP和SOCKS5均支持此方法。假设代理服务器10.20.30.40:80，客户端使用以下命令连接：
  
-up="https://[username:password@]10.20.30.40:80/1.2.3.4:8100"  
