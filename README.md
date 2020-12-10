![OpenWrt logo](/logo.svg)

# 序章

### 包是否安全
   本包直接Fork了openwrt的源码包增加了一些常用软件   
   所有资源均来自大佬们的GitHub,我只做了集成到1907的工作
   
### 为啥要整这个包不直接使用官方发布的
   一是官方包只有256M装一点软件就满了,虽然可以通过dd命令增加容量,不过容量看着不舒服也不方便修改  
   二是使用docker之类的需要内核集成,不是说安装ipk就能使用  
   三是 折腾着好玩

### 为啥不使用Lean大的包
   想体验1907原版,然后我刷了lean大的包udp有点异常,做减法感觉不如做加法简单所以就整了个
	
### 为啥不整合qbit,aria2之类的
   这些ipk下载都很方便感觉没啥必要  
   恩山大佬的qbit下源码还要恩山币我木有  
   他貌似也没发github后续更新也麻烦  

### 没有L大的sfe不快乐了？
   其实L大的sfe貌似就修改了下pdnsd和屏蔽一些ip,需要的可以自己手动改改  
   bbr加速也可以自己装一个,不过本地基本用处不大不建议  
	Routing/NAT 分载1907自带,在"防火墙"里开启  

### 包是否能轻松编译
   我编译完自己固件后make distclean然后再编译了一次  
   花了我不少时间,环境如果都正常的话编译应该正常的  
   编译失败基本都是环境没安,或者包下不下来,建议开全局下载  
   如果使用win的子系统编译看最后,win下有个Path的bug  
   还有,win的子系统用win下的代理软件可能无法代理成功,建议  
   ```
   export http_proxy="http://localhost:port"   
   export https_proxy="http://localhost:port"
   ```  
   开启代理(如果没看到http代理端口配置选项的话端口可能是socket端口+1)  
   	
       
# 编译指南

1.首先装好 Ubuntu 64bit,推荐 Ubuntu 18以上版本  
2.命令行输入 `sudo apt-get update` ，然后输入 `sudo apt-get -y install build-essential asciidoc binutils bzip2 gawk gettext git libncurses5-dev libz-dev patch python3 python2.7 unzip zlib1g-dev lib32gcc1 libc6-dev-i386 subversion flex uglifyjs git-core gcc-multilib p7zip p7zip-full msmtp libssl-dev texinfo libglib2.0-dev xmlto qemu-utils upx libelf-dev autoconf automake libtool autopoint device-tree-compiler g++-multilib antlr3 gperf wget curl swig rsync`  
3.安装失败他也会结束,需要确保环境都安装成功,不然编译的时候脑壳疼.  
4.使用 `git clone -b openwrt-19.07.5.1 https://github.com/SamuelZon258/openwrt.git` 命令下载好源代码,然后 `cd openwrt` 进入目录  
5. `./scripts/feeds update -a && ./scripts/feeds install -a` :下载并安装软件  
6. `make menuconfig` 选择需要的软件  
(如果全都需要也可以直接执行`cp -r ./.config.bat ./.config`,不建议全添加再去除,可能会有没必要的依赖残留)  

### 必选:  
>Target System (Broadcom BCM63xx)  --->选择你要需要生成那种CPU的固件(如x86)  
>Target Profile (Alcatel RG100A)  --->固件界面显示的型号(如x86_64)  
		
### 推荐修改:  
>Target Images ---> Root filesystem partition size : root的容量(我自己设置的是1G)  
>LuCI --->  
>>Collections :web界面样式(需要LuCI就必选)  
>>>Modules --->  
>>>>luci-compat :不选打开luci设置页面会报错  
	 Translations ---> Chinese Simplified (zh_Hans) : web界面加入中文  
>Base system ---> dnsmasq-full:建议用来替代dnsmasq,不然后面装某些有依赖的软件会麻烦  
>Global build settings ---> Kernel build options ---> Enable miscellaneous LXC related options(某些软件"如docker"需要的内核组件)  

### 需要时才添加:  
>Base system --->  
>>block-mount :挂载点  
		
>Network --->  
>>File Transfer --->  
>>>vsftpd :ftp服务端  
>>>wget :下载器   

>LuCI --->(大部分是我自己添加的luci)  
>>luci_app_dockerman ：docker控制器(编译内核需要加入POSIX_MQUEUE和kamailio5-mod-mqueue,或者直接加入LXC)  
  luci-app-serverchan ：微信推送路由器信息  
  luci-app-wrtbwmon ： 实时流量监控(安装微信推送必须也选上)  
  luci-app-ssr-plus :emmm(这个依赖dnsmasq-full,不取消dnsmasq会有冲突,后面有说如何取消)  
  luci-app-passwall :emmm*2(19.07需要选上Libraries ---> luci-lib-ipkg)  
  luci-app-dnsmasq-ipset : dnsmasq-full的Luci  
  luci-app-vlmcsd : kms服务端  
  luci-app-unblockmusic : 解锁网易云灰色歌曲  
  luci-app-baidupcs-web ： 百度云盘(修改了KFERMercer的部分内容,不然在19.07下可能不显示也打不开页面)  
				
### 建议取消:  
>Base system ---> dnsmasq:因为已经有dnsmasq-full了  

7.`make -j8 download V=s` 预先下载dl库,不执行也行,后续make会自动下载  
8.`make -j$(($(nproc) + 1)) V=s || make -j1 V=99` (开始编译,编译失败则输出日志)  


# 友情提示

### 常用命令：
`make clean` :清除bin目录  
`make dirclean` :clean + 清除交叉编译链工具以及工具链目录  
`make distclean` :清除所有相关的东西，包括下载的副本，配置文件，feed内容等  
编译失败三件套,一般直接clone就编译应该用不着  

`./scripts/feeds clean` : 清理软件目录命令  

`make packages/XXX/compile V=s` : 单独编译某个ipk,需要先编译完固件,然后在`make menuconfig`中选中(不用傻傻的为了一个软件编译整个固件了)  

如果你使用windows的子系统Ubuntu来编译固件可能会遇到$PATH与windows的混杂编译失败,只需要执行下  
`echo "[interop]\nenabled=false\nappendWindowsPath=false" | sudo tee /etc/wsl.conf`  
然后再在cmd中执行  
```
net stop LxssManager  
net start LxssManager
```
即可

如果遇到国内网站dns解析问题可以执行`vi /etc/pdnsd.conf`,将server中的ip=xxx修改成ip=114.114.114.114,114.114.115.115然后重启路由

# License
OpenWrt is licensed under GPL-2.0
