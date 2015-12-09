

#####摘要：
根据内容进行hash的一个数字，这个数字是不可逆的
#####签名
对摘要用自己的私钥进行加密得出的结果


上面两个可以参考[Message Digests and Digital Signatures](http://www.diablotin.com/librairie/networking/puis/ch06_05.htm)

#####非对称加密，公钥私钥：
公钥加密的内容只有私钥能解开（公钥不能解密），私钥加密的内容只有公钥能解开。公钥是公开的，私钥是自己保存，不能公开的。
公钥私钥的

一个应用是：认证是否本人，如验证A是否A，B用A的公钥加密一段内容给A，A用自己的私钥解密，给回B，B对A传回的内容进行验证。

###android相关
packageinfo.signatures 0  是证书签名（META-INT下面那个.rsa）

[重签名](http://www.cnblogs.com/findyou/p/3801273.html)

##参考资料
[CA](http://baike.baidu.com/item/CA#7)

[https数字签名](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html)

[Using Android Volley With Self-Signed SSL Certificate](http://ogrelab.ikratko.com/using-android-volley-with-self-signed-certificate/)