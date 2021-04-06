---
title: 电信天翼光猫TEWA-768G/TEWA-708E/tewa-1000e破解超管密码
---
因搭建家用服务器、nas等需求，需要将光猫的拨号转到软路由上。
具体的操作是： 超级管理员登录→网络→网口→桥接模式
然而电信并不会告诉普通用户超级管理员密码，TEWA-708E甚至隐藏了管理后台的页面。

## 准备工作
1）一台能连接到光猫的电脑
2）一个U盘，需要是fat32格式

## 1.登录管理后台
虽然超级管理员密码拿不到，但是普通用户的密码是直接印在光猫背面的。
如果是windows电脑，打开cmd输入ipconfig，linux使用ifconfig，回车，查看当前网关。
当前网关即路由器的地址：192.168.x.1，在浏览器中输入192.168.x.1，就会导航到路由器管理后台的登录界面。<font color="red">需要注意的是这个页面并不是真实的管理后台页面，真实的管理后台地址应该是 192.168.x.1:8080/login.html</font>(如下图)
![](https://docsin.uniontech.com/wp-content/uploads/2021/03/5fbe8cb1f28c83894.png_e680.png)
此时输入用户名： useradmin 密码：光猫背面的密码
进入普通用户权限的管理后台，登录进去之后这一步就成功了！
<font color="red">插入U盘</font>

## 2.获取隐藏页面链接
1）打开 管理→设备管理
2）按f12打开调试工具，进入source，选择MD_Device_user.html
3）找到sessionkey <font color="red">sessionkey很快会过期，后续操作如果出现权限异常，请回到第三部重新来，耐心点多来几次</font>
![](https://docsin.uniontech.com/wp-content/uploads/2021/03/5fbe8cb2a1a5b425-1.png_e680-1.png)
4）找到【快速恢复】所对应的链接（图中4所对应的位置）
![](https://docsin.uniontech.com/wp-content/uploads/2021/03/5fbe8cb1c35fb2876.png_e680.png)
5）用浏览器打开 链接+sessionkey，例如：http://192.168.1.1:8080/usbbackupNaNd?action=backupeble&amp;sessionKey=357535522
如果提示<font color="red">Invlid Session Key</font>请回到步骤3）
看到以下界面，则这一步成功
![](https://docsin.uniontech.com/wp-content/uploads/2021/03/5fbe8cb2c910f8166.png_e680.png)

## 3.获取密码
1) USB分区选择，选中你的U盘，如果没有找到U盘，请检查格式是否是fat32
2) F12打开调试模式，删除“备份配置”按钮的disable属性。
3) 点击“备份配置”
4) 打开U盘，此时U盘里面多了一个文件夹，用vscode或者其他高级文本编辑器打开里面的文件就能看到password了
5) 输入192.168.x.1:8080/login.html 用户名：telecomadmin 密码： 复制文件里面的password，进入到管理后台，可以看到多了好多菜单。

## 4.桥接前的准备
假如你忘记了家里宽带的账号密码，这时候一定要记得备份，进 网络→网络设置
可以直接看到账号
然后f12去掉密码处password的password type，就能拿到base64之后的密码
随便找个base64解码网站，解码后得到真实密码
注意做好备份
然后就能设置桥接模式，使用自己的路由器或者软路由拨号啦！