# ngrok
ngrok一键安装脚本，适用于Centos版本服务器  
默认安装路径  
git：/usr/local/git  
go：/usr/local/go  
ngrok：/usr/local/ngrok  

#使用说明：
	chmod +x ngrok.sh  
	sh ./ngrok.sh  
进行选择，安装

#客户端编译好存放的路径：
	/usr/local/ngrok/bin/  
需要根据客户端的操作系统环境编译，脚本也已经写好

详细教程看博客：[www.sunnyos.com](http://www.sunnyos.com "Sunny博客")  
搭建好免费提供的ngrok服务：[www.ngrok.cc](http://www.ngrok.cc "www.ngrok.cc")  

shell脚本一键安装ngrok
post by rocdk890 / 2016-7-5 15:24 Tuesday linux技术 发表评论
 最近要对内网进行调试和访问,买了3个花生棒才只能映射6个端口,实在是不能满足需要,刚好有个朋友给我说了ngrok也可以进行内网穿透,研究了几天,光安装就要搞死一帮人,还好在github上找到个一键安装ngrok的脚步,先共享给大家.
 系统:centos 6.x(64位)

脚本内容:
cat ngrok.sh
001
#!/bin/bash
002
# -*- coding: UTF-8 -*-
003
#############################################
004
#作者网名：Sunny                 #
005
#作者博客：www.sunnyos.com                    #
006
#作者QQ：327388905                           #
007
#作者QQ群:57914191                           #
008
#作者微博：http://weibo.com/2442303192        #
009
#############################################
010
# 获取当前脚本执行路径
011
SELFPATH=$(cd "$(dirname "$0")"; pwd)
012
GOOS=`go env | grep GOOS | awk -F\" '{print $2}'`
013
GOARCH=`go env | grep GOARCH | awk -F\" '{print $2}'`
014
echo '请输入一个域名'
015
read DOMAIN
016
install_yilai(){
017
    yum -y install zlib-devel openssl-devel perl hg cpio expat-devel gettext-devel curl curl-devel perl-ExtUtils-MakeMaker hg wget gcc gcc-c++ unzip
018
}
019
 
020
# 安装git
021
install_git(){
022
    unstall_git
023
    if [ ! -f $SELFPATH/git-2.6.0.tar.gz ];then
024
        wget https://www.kernel.org/pub/software/scm/git/git-2.6.0.tar.gz
025
    fi
026
    tar zxvf git-2.6.0.tar.gz
027
    cd git-2.6.0
028
    ./configure --prefix=/usr/local/git
029
    make
030
    make install
031
    ln -s /usr/local/git/bin/* /usr/bin/
032
    rm -rf $SELFPATH/git-2.6.0
033
}
034
 
035
# 卸载git
036
unstall_git(){
037
    rm -rf /usr/local/git
038
    rm -rf /usr/local/git/bin/git
039
    rm -rf /usr/local/git/bin/git-cvsserver
040
    rm -rf /usr/local/git/bin/gitk
041
    rm -rf /usr/local/git/bin/git-receive-pack
042
    rm -rf /usr/local/git/bin/git-shell
043
    rm -rf /usr/local/git/bin/git-upload-archive
044
    rm -rf /usr/local/git/bin/git-upload-pack
045
}
046
 
047
 
048
# 安装go
049
install_go(){
050
    cd $SELFPATH
051
    uninstall_go
052
    
# 动态链接库，用于下面的判断条件生效
053
    ldconfig
054
    
# 判断操作系统位数下载不同的安装包
055
    if [ $(getconf WORD_BIT) = '32' ] && [ $(getconf LONG_BIT) = '64' ];then
056
        
# 判断文件是否已经存在
057
        if [ ! -f $SELFPATH/go1.4.2.linux-amd64.tar.gz ];then
058
            wget http://www.golangtc.com/static/go/1.4.2/go1.4.2.linux-amd64.tar.gz
059
        fi
060
        tar zxvf go1.4.2.linux-amd64.tar.gz
061
    else
062
        if [ ! -f $SELFPATH/go1.4.2.linux-386.tar.gz ];then
063
            wget http://www.golangtc.com/static/go/1.4.2/go1.4.2.linux-386.tar.gz
064
        fi
065
        tar zxvf go1.4.2.linux-386.tar.gz
066
    fi
067
    mv go /usr/local/
068
    ln -s /usr/local/go/bin/* /usr/bin/
069
}
070
 
071
# 卸载go
072
 
073
uninstall_go(){
074
    rm -rf /usr/local/go
075
    rm -rf /usr/bin/go
076
    rm -rf /usr/bin/godoc
077
    rm -rf /usr/bin/gofmt
078
}
079
 
080
# 安装ngrok
081
install_ngrok(){
082
    uninstall_ngrok
083
    cd /usr/local
084
    if [ ! -f /usr/local/ngrok.zip ];then
085
        cd /usr/local/
086
        wget http://www.sunnyos.com/ngrok.zip
087
    fi
088
    unzip ngrok.zip
089
    export GOPATH=/usr/local/ngrok/
090
    export NGROK_DOMAIN=$DOMAIN
091
    cd ngrok
092
    openssl genrsa -out rootCA.key 2048
093
    openssl req -x509 -new -nodes -key rootCA.key -subj "/CN=$NGROK_DOMAIN" -days 5000 -out rootCA.pem
094
    openssl genrsa -out server.key 2048
095
    openssl req -new -key server.key -subj "/CN=$NGROK_DOMAIN" -out server.csr
096
    openssl x509 -req -in server.csr -CA rootCA.pem -CAkey rootCA.key -CAcreateserial -out server.crt -days 5000
097
    cp rootCA.pem assets/client/tls/ngrokroot.crt
098
    cp server.crt assets/server/tls/snakeoil.crt
099
    cp server.key assets/server/tls/snakeoil.key
100
    
# 替换下载源地址
101
    sed -i 's#code.google.com/p/log4go#github.com/keepeye/log4go#' /usr/local/ngrok/src/ngrok/log/logger.go
102
    cd /usr/local/go/src
103
    GOOS=$GOOS GOARCH=$GOARCH ./make.bash
104
    cd /usr/local/ngrok
105
    GOOS=$GOOS GOARCH=$GOARCH make release-server
106
    /usr/local/ngrok/bin/ngrokd -domain=$NGROK_DOMAIN -httpAddr=":80"
107
}
108
 
109
# 卸载ngrok
110
uninstall_ngrok(){
111
    rm -rf /usr/local/ngrok
112
}
113
 
114
# 编译客户端
115
compile_client(){
116
    cd /usr/local/go/src
117
    GOOS=$1 GOARCH=$2 ./make.bash
118
    cd /usr/local/ngrok/
119
    GOOS=$1 GOARCH=$2 make release-client
120
}
121
 
122
# 生成客户端
123
client(){
124
    echo "1、Linux 32位"
125
    echo "2、Linux 64位"
126
    echo "3、Windows 32位"
127
    echo "4、Windows 64位"
128
    echo "5、Mac OS 32位"
129
    echo "6、Mac OS 64位"
130
    echo "7、Linux ARM"
131
 
132
    read num
133
    case "$num" in
134
        [1] )
135
            compile_client linux 386
136
        ;;
137
        [2] )
138
            compile_client linux amd64
139
        ;;
140
        [3] )
141
            compile_client windows 386
142
        ;;
143
        [4] )
144
            compile_client windows amd64
145
        ;;
146
        [5] )
147
            compile_client darwin 386
148
        ;;
149
        [6] )
150
            compile_client darwin amd64
151
        ;;
152
        [7] )
153
            compile_client linux arm
154
        ;;
155
        *) echo "选择错误，退出";;
156
    esac
157
 
158
}
159
 
160
 
161
echo "请输入下面数字进行选择"
162
echo "#############################################"
163
echo "#作者网名：Sunny"
164
echo "#作者博客：www.sunnyos.com"
165
echo "#作者QQ：327388905"
166
echo "#作者QQ群:57914191"
167
echo "#作者微博：http://weibo.com/2442303192"
168
echo "#############################################"
169
echo "------------------------"
170
echo "1、全新安装"
171
echo "2、安装依赖"
172
echo "3、安装git"
173
echo "4、安装go环境"
174
echo "5、安装ngrok"
175
echo "6、生成客户端"
176
echo "7、卸载"
177
echo "8、启动服务"
178
echo "9、查看配置文件"
179
echo "------------------------"
180
read num
181
case "$num" in
182
    [1] )
183
        install_yilai
184
        install_git
185
        install_go
186
        install_ngrok
187
    ;;
188
    [2] )
189
        install_yilai
190
    ;;
191
    [3] )
192
        install_git
193
    ;;
194
    [4] )
195
        install_go
196
    ;;
197
    [5] )
198
        install_ngrok
199
    ;;
200
    [6] )
201
        client
202
    ;;
203
    [7] )
204
        unstall_git
205
        uninstall_go
206
        uninstall_ngrok
207
    ;;
208
    [8] )
209
        echo "输入启动域名"
210
        read domain
211
        echo "启动端口"
212
        read port
213
        /usr/local/ngrok/bin/ngrokd -domain=$domain -httpAddr=":$port"
214
    ;;
215
    [9] )
216
        echo "输入启动域名"
217
        read domain
218
        echo server_addr: '"'$domain:4443'"'
219
        echo "trust_host_root_certs: false"
220
 
221
    ;;
222
    *) echo "";;
223
esac

ps:
这个脚步只适合centos,其他linux的请另外找吧.

默认安装路径
git：/usr/local/git
go：/usr/local/go
ngrok：/usr/local/ngrok

客户端编译好存放的路径:
/usr/local/ngrok/bin/

ngrok服务端启动命令:
nohup /usr/local/ngrok/bin/ngrokd -domain="slogra.com" -tlsCrt="/usr/local/ngrok/server.crt" -tlsKey="/usr/local/ngrok/server.key" -httpAddr=":8080" -tunnelAddr=":4443" -log-level="INFO" >/var/log/ngrok.log 2>&1 &

ngrok客户端配置文件:
server_addr: "你自己的域名:4443"
tunnels:
  www:
   subdomain: "test" #定义服务器分配域名前缀，跟平台上的要一样
   proto:
    http: 80 #映射端口，不加ip默认本机
    https: 80
  web:
   subdomain: "web" #定义服务器分配域名前缀
   proto:
    http: 192.168.1.100:80 #映射端口，可以通过加ip为内网任意一台映射
    https: 192.168.1.100:80
  ssh:
   remote_port: 50001 #服务器分配tcp转发端口，如果不填写此项则又服务器分配
   proto:
    tcp: 22 #映射本地的22端口
  ssh1: #将由服务器分配端口
    proto:
      tcp: 21

ps:大家可以根据自己的需要修改客户端配置文件.

客户端启动命令:
1、进入到ngrok和ngrok.cfg所在的目录
2、启动单个服务 ./ngrok -config ngrok.cfg start www
3、启动多个服务 ./ngrok -config ngrok.cfg start www web ssh ssh1
4、后台运行可以使用 setsid ./ngrok -config ngrok.cfg start www

好了,大家可以根据自己的需要去配置.

ps:
感谢Sunny给我们提供了这样方便的安装脚本.
https://github.com/sunnyos/ngrok
