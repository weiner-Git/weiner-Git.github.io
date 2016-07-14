# php使用curl进行https请求 错误提示ssl connect error

错误描述：对第三方提供的地址（https）转发数据,curl_error()输出ssl connect error，
但是查找一些其他https地址，如微信api，百度地址等测试下都没有错误信息。
网上查看了一些解决方法，下面是自己的解决过程：

curl中  
curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, FALSE);  
curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, FALSE);

1，升级nss
	在阿里云上写了一个demo测试，错误提示为：Cannot communicate securely with peer: no common encryption algorithm(s).这种情况下升级nss，问题解决。命令：yum update nss nss-util nss-sysinit nss-tools。网上所重启php，这里没有重启，更新升级后，问题直接解决（php 5.6.2）

2，	开发机代码运行环境 php5.6,错误提示ssl connect error, 用php5.5.18版本运行demo，没有问题。然后对比一下curl版本，参数都一样，后来把代码运行环境切换到5.5.18解决的这个问题。怀疑是5.6下curl的问题，但是没有找到问题原因。
