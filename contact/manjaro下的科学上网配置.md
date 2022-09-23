# manjaro下的科学上网配置

首先要下载一个clash for windows并完成安装

然后从节点服务商处获取到订阅的节点文件，然后输入那个下载节点文件的地址到clash的profile内自动下载规则文件

然后在终端处理一下命令

```shell
su
cd root/ect/profile.d
vim xxx.sh
```

在xxx.sh内输入

```shell
export https_proxy=http://127.0.0.1:7890                                                                                                                                                     ✔  32s  
export http_proxy=http://127.0.0.1:7890
export all_proxy=http://127.0.0.1:7890
```

打开clash上的start with linux，重启电脑

在终端键入

```shell
curl -v -o /dev/null https://www.google.com
```

若连上了即可科学上网

如git带等软件可能需要单独配置代理
