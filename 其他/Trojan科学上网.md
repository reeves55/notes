# Trojan科学上网



* Centos8 x64系统
* 谷歌云vps



## 1. 申请域名

申请网站：https://www.namecheap.com/

支付方式：```visa```、```mastercard```



支付完成，到Dashboard -> Advanced DNS里面

![image-20200211221419282](/Users/xiao/Library/Application Support/typora-user-images/image-20200211221419282.png)



**注意**：刚注册未验证邮箱地址的，需要先验证，否则不会给你解析的

![image-20200211221600768](/Users/xiao/Library/Application Support/typora-user-images/image-20200211221600768.png)



## 2. 一键安装Torjan

```shell
curl -O https://raw.githubusercontent.com/atrandys/trojan/master/trojan_mult.sh && chmod +x trojan_mult.sh && ./trojan_mult.sh
```



安装完成会给一个已经配置好的Windows客户端的下载地址，下载下来，那Mac系统的话就先下载Mac版本的客户端，然后手动配置一下，Mac客户端下载地址：







参考：https://xugaoxiang.com/2020/01/02/trojan-installation/