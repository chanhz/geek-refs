# RSA:从 SSH/SSL/TLS 到 HTTPS

## RSA

### openssl

**openssl生成新的RSA密钥对**
openssl 是一个生成公司密钥对的解决方案。使用如下命令，可以生成 PKCS #1 格式的 pem 私钥文件：
```bash
openssl genrsa -out privatekey.pem 2048
```
此时可以看到在当前目录下，生成了 privatekey.pem 私钥文件。

公钥呢？

可以使用 openssl 命令对 privatekey.pem 提取出公钥:
```bash
openssl rsa -in privatekey.pem -out publickey.pem -pubout
```

生成的公钥文件，页眉和页脚行分别是“BEGIN PUBLIC KEY”和“END PUBLIC KEY”.

什么是 pem 呢？

pem 文件其实以PCSK #1 格式描述了私钥的以下信息：

```
RSAPrivateKey ::= SEQUENCE {
     version           Version,
     modulus           INTEGER,  -- n
     publicExponent    INTEGER,  -- e
     privateExponent   INTEGER,  -- d
     prime1            INTEGER,  -- p
     prime2            INTEGER,  -- q
     exponent1         INTEGER,  -- d mod (p-1)
     exponent2         INTEGER,  -- d mod (q-1)
     coefficient       INTEGER,  -- (inverse of q) mod p
     otherPrimeInfos   OtherPrimeInfos OPTIONAL
 }
RSAPublicKey ::= SEQUENCE {
     modulus           INTEGER,  -- n
     publicExponent    INTEGER   -- e
} 
```


人类可读的形式解读 pem 文件
```bash
openssl rsa -in privatekey.pem -text
```
在 Java / iOS 开发中使用 RSA
Java 的 KeyStore 密钥生成算法支持 PKCS #1 的格式，而采用“更安全的” PKCS #8 格式，此时，可以使用 `openssl` 命令转换 pem 密钥文件：
```bash
openssl pkcs8 -in privatekey.pem -topk8 -nocrypt -out privatekey-pkcs8.pem
```
[PKCS #8 格式](https://tools.ietf.org/html/rfc5208#section-5)描述的信息如下：
```
PrivateKeyInfo ::= SEQUENCE {
   version                   Version,
   privateKeyAlgorithm       PrivateKeyAlgorithmIdentifier,
   privateKey                PrivateKey,
   attributes           [0]  IMPLICIT Attributes OPTIONAL }
```
请注意，无论是 PKCS #1 还是 PKCS #8 , 都采用了 DER 格式对数据进行编码。

PKCS #8 也可以转换成 PKCS #1:
```bash
openssl rsa -in pkcs8.pem -out pkcs1.pem
```

## SSH

SSH是一种网络协议，用于计算机之间的加密登录, 由芬兰学者 Tatu Ylonen 与 1995 年设计。

SSH 使用了公钥加密算法来加解密传输信息来保证安全。
整个过程是这样的：
（1）远程主机收到用户的登录请求，把自己的公钥发给用户。
（2）用户使用这个公钥，将登录密码加密后，发送回来。
（3）远程主机用自己的私钥，解密登录密码，如果密码正确，就同意用户登录。



## ssl/tls

## https

证书中心 CA




参考文献：

1. [SSH原理与运用（一）：远程登录](http://www.ruanyifeng.com/blog/2011/12/ssh_remote_login.html) 作者 阮一峰
2. [SSH原理与运用（二）：远程操作与端口转发](http://www.ruanyifeng.com/blog/2011/12/ssh_port_forwarding.html) 作者 阮一峰
3. [SSL/TLS协议运行机制的概述](https://www.ruanyifeng.com/blog/2014/02/ssl_tls.html) 作者 阮一峰
4. [对于RSA密钥使用openssl和java](http://xueliang.org/article/detail/20170807222857437)
5. [RFC 3447 - Public-Key Cryptography Standards (PKCS) #1: RSA Cryptography Specifications Version 2.1](https://tools.ietf.org/html/rfc3447#appendix-A.1.1)
6. [openssl RSA密钥格式PKCS1和PKCS8相互转换 - cocoajin - 博客园](https://www.cnblogs.com/cocoajin/p/10510574.html)
