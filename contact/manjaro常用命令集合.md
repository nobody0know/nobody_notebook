# manjaro常用命令集合

## manjaro原生命令

```shell
sudo pacman -Syu  #系统更新
pacman -Syy       #强制更新


pacman -Ss keyword  #在仓库中搜索含关键字的包（常用）    pacman -Ss ‘^fcitx-’   查看已安装软件
pacman -Qs keyword  #搜索已安装的包（常用）             pacman -Qs ‘^fcitx-’
pacman -Qi package_name #查询本地安装包的详细信息
pacman -Ql package_name #列出该包的文件
pacman -Fs keyword  #按文件名查找软件库
pacman -Si package_name #显示远程软件包的详尽的信息
pacman -Qii package_name    #使用两个 -i 将同时显示备份文件和修改状态
pacman -Ql package_name #要获取已安装软件包所包含文件的列表
pacman -Fl package_name #查询远程库中软件包包含的文件
pacman -Qk package_name #检查软件包安装的文件是否都存在
pacman -Fo /path/to/file_name   #查询文件属于远程数据库中的哪个软件包
pacman -Qdt             #要罗列所有不再作为依赖的软件包(孤立orphans)
pacman -Qet             #要罗列所有明确安装而且不被其它包依赖的软件包
pactree package_name    #要显示软件包的依赖树
whoneeds package_name   #检查一个安装的软件包被那些包依赖    pkgtoolsAUR中的whoneeds
pactree -r package_name #检查一个安装的软件包被那些包依赖


pacman -S package_name  #执行 pacman -S firefox 将安装 Firefox（常用）    你也可以同时安装多个包，只需以空格分隔包名即
pacman -Sy package_name #与上面命令不同的是，该命令将在同步包数据库后再执行安装。    
pacman -Sv package_name #在显示一些操作信息后执行安装。 
pacman -U local_package_name    #安装本地包，其扩展名为pkg.tar.gz或pkg.tar.xz    
pacman -U url   #安装一个远程包（不在 pacman 配置的源里面）   例：pacman -U http://www.example.com/repo/example.pkg.tar.xz


pacman -R    package-name #只删除包，保存其全部已经安装的依赖关系
pacman -Rs  package-name #删除包，删除其所有没有被其他已安装软件包使用的依赖关系
pacman -Rsc package-name #删除包，删除所有依赖这个软件包的程序
pacman -Rd  package-name #删除包，不检查依赖
```

## yay常用命令

```shell
yay -S package # 从 AUR 安装软件包
yay -Rns package # 删除包
yay -Syu # 升级所有已安装的包
yay -Ps # 打印系统统计信息
yay -Qi package # 检查安装的版本
#yay 安装命令不需要加 sudo
```

## 解压dep包常用命令

```shell
sudo debtap xxxxxxx.deb
```

# AUR安装PKGBUILD文件

```shell
makepkg -si #于此文件的目录下使用此命令
```

## vim按了ctrl+z的处理方法

ctrl+z 退出后，在显示[1]+ Stopped vi xxx.xxx时，终端直接输入fg 1（中括号中显示的数字，即作业号，若只有一个，作业号可忽略）这样就会重回vim编辑界面了，然后正常退出即可。
ls -a 一下，会看到隐藏的.swp文件 删除了此文件即可，再次使用vim打开文件就不会出现上述界面了。
