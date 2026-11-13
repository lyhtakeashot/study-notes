# 一.前提检查：系统版本为win11，cpu虚拟化开启
# 二.检查对应功能是否正常开启
1. win + i打开“设置”，选择“系统设置”
   ![[屏幕截图 2026-09-22 104554.png]]
2. 划到最下面打开“可选功能”，打开”更多windows功能“![[屏幕截图 2026-09-22 104611.png]]
3. 划到最下面勾选“试用linux的windows子系统”和“虚拟机平台”![[屏幕截图 2026-09-22 104619.png]]
4. 点击确定后，重启电脑
# 三.wsl更新到最新版（这里有多条途径，我分点区分，不是流程）
## 1. win + r打开运行对话框输入cmd，输入指令
![[屏幕截图 2026-09-22 104748.png]]
```cmd
wsl --update
```
![[屏幕截图 2026-09-22 104930.png]]优点：不用找第三方途径
缺点：因为使用的是Microsoft store或海外的微软服务器下载，网络不稳定，下载速度极慢
## 2. win + r打开运行对话框输入cmd，然后回车，输入指令
![[Pasted image 20260922105212.png]]
```cmd
wsl --update --web-download
```
![[屏幕截图 2026-09-22 105157.png]]
优点：如果有梯子或者加速器比如（Watt Toolkit）下载速度会稍微好一点，但也聊胜于无；不用找三方下载途径
缺点：需要第三方工具，用的是微软的CDN服务器下载文件，还是比较慢
## 3. 打开浏览器，搜索
https://github.com/microsoft/WSL/releases
![[Pasted image 20260922105343.png]]
直接下载最新版的release如wsl.2.6.3.0.x64.msi。下载位置没有需求，下载完直接双击，按照提示完成安装后文件可以删掉
优点：速度比前两种方法都要快，但还是kb的下载速度
缺点：需要第三方工具，还是不方便
## 4.使用国内的镜像站(这里只是举两个例子)
清华镜像站[https://mirrors.tuna.tsinghua.edu.cn/github-release/WhitewaterFoundry/Pengwin/releases/](https://mirrors.tuna.tsinghua.edu.cn/github-release/microsoft/WSL2/)
中科大镜像站
[`https://mirrors.ustc.edu.cn/github-release/microsoft/WSL2/`](https://mirrors.ustc.edu.cn/github-release/microsoft/WSL2/)
优点：比前三个的下载速度都要快
缺点：容易404，具体原因不清楚
## 5.尝试其他github加速代理
在原始github链接前加上代理前缀即可
https://githubproxy.cc/
https://ghfast.top/
https://gh-proxy.com/
https://ghproxy.net
https://gh.llkk.cc
比如：
https://githubproxy.cc/https://github.com/microsoft/WSL/releases/download/2.6.3/wsl.2.6.3.0.x64.msi
![[Pasted image 20260922105440.png]]
优点：下载速度极快，校园网都能跑到5mb以上
缺点：下载链接容易失效
# 四.下载ubuntu
打开微软商店搜索ubuntu，下载后打开就行
![[Pasted image 20260922105528.png]]