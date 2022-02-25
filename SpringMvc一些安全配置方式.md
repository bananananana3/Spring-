目前越来越多的应用和网站，开始注重安全性的问题，关于我们的web项目的几个安全知识点，不得不讲解一下，这里我主要讲述关于tomcat如何支持HTTPS连接访问，RSA公钥和私钥的制作。这个对于我们整个系统的安全性都起了关键性作用。对于RSA的一些说明，以及Java的实现方法这个我将会在下一个章节介绍。

## 1、tomcat项目部署SSL安全连接。

这里涉及两个概念，单向认证和双向认证。

单向认证，就是client->sever发送数据 ，server收到数据用私钥对data加密，然后把加密的data和公钥给client，client通过公钥对数据加密发送到server，server用私钥解密，如果解密成功，说明是来自该服务器的.简单的来说，就是客户端发送带了公钥加密的数据给服务器验证，但是这个步骤是没有经过客户端的验证的。只是对服务器进行验证，所以我们单向的认证，只需要配置站点就可以了。

双向认证，就是client->server发送数据的时候，把数据加密后并且带上client自己的证书，发送给server，server收到后，用client带的证书，对消息解密，然后用sever的证书对消息加密并且把server的证书发送给client，client收到后，用sever的证书对信息解密。然后用server的证书加密，再用client的证书加密。这样再把client的证书和加密信息发送给server。server收到信息后，用client的证书进行解密，这样可以确保是有client发送过来的。然后再用server端的私钥对消息解密。这样就可以得到明文了。

安装环境 :java+tomcat+单向认证

1、首先创建一个keystore,这里注意就是我们需要安装好Java环境，

在终端输入如下命令：

keytool -genkey -v -alias yeehot -keyalg RSA -validity 3650 -keystore ./yeehot.keystore

这里的yeehot,是自定义的名字。以及保存到当前目录下

这个时候会提醒你创建密码的,接着你按照如下的方式和提示输入自己相应的信息

![Spring MVC学习总结（5）——SpringMVC项目关于安全的一些配置与实现方式_数据加密](https://s4.51cto.com/images/blog/202108/06/14b636e63be2a6cea33b007ac0bba504.jpeg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

最后打一个字：是

按回车，完成操作。

对于tomcat配置

找到tomcat的conf目录，打开server.xml

![Spring MVC学习总结（5）——SpringMVC项目关于安全的一些配置与实现方式_数据_03](https://s8.51cto.com/images/blog/202108/06/a46476d7631a0c92100184ba9e7b78d8.jpeg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

找到原来已经注释的8443端口

<!--

<Connector port="8443" protocol="org.apache.coyote.http11.Http11Protocol"

maxThreads="150" SSLEnabled="true" scheme="https" secure="true"

clientAuth="false" sslProtocol="TLS" />

-->

然后我们根据这些信息改成我们自己的信息，我们首先把刚刚的keystore复制到这个文件夹下

<Connector SSLEnabled="true" acceptCount="100" clientAuth="false"

disableUploadTimeout="true" enableLookups="false" maxThreads="25"

port="8443" keystoreFile="./conf/yeehot.keystore" keystorePass="ming1314"

protocol="org.apache.coyote.http11.Http11NioProtocol" scheme="https"

secure="true" sslProtocol="TLS" />

这个时候我们启动tomcat

然后在服务器输入https://localhost:8443/可以看到下面的信息。说明我们配置成功了。可以访问https连接了。

我们点击下方的高级，可以访问tomcat的主页，此时也是使用HTTPS的连接的

## 2、RSA公私钥的制作。

首先，你必须按照openssl工具，并且编译，由于我这里是使用MAC系统，自带了openssl，直接输入openssl就可以了。

对于windows的可以直接去支付宝的商家服务那里下载一个二进制文件进行生成也行。如果找不到可以给我留言。

对于生成RAS公私钥主要有一下几个步骤。

生成RSA私钥

openssl>genrsa -out rsa_private_key.pem 1024

生成RSA公钥

openssl>rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem

将RSA私钥转换成PKCS8格式

openssl>pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM -nocrypt

如何生成证书：

1)输入openssl

2)这个时候生成RSA私钥：

genrsa -out rsa_private_key.pem 1024

3）接着我们生成公钥

rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem

4）将RSA私钥转换成PKCS8格式

这个时候在我们的目录下会生成我们需要的公钥和私钥

![Spring MVC学习总结（5）——SpringMVC项目关于安全的一些配置与实现方式_双向认证_06](https://s6.51cto.com/images/blog/202108/06/16cede8e39f76e7f77c98b35ab348852.jpeg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

完整的操作如下：

![Spring MVC学习总结（5）——SpringMVC项目关于安全的一些配置与实现方式_双向认证_07](https://s5.51cto.com/images/blog/202108/06/e94d7d7ff5d3cf5612f354e67c2929df.jpeg?x-oss-process=image/watermark,size_16,text_QDUxQ1RP5Y2a5a6i,color_FFFFFF,t_100,g_se,x_10,y_10,shadow_90,type_ZmFuZ3poZW5naGVpdGk=)

# SpringBoot写API接口,如何提高API的安全性,数据加解密方案

# 如何保证API调用时数据的安全性:

- 通信使用https
- 请求签名，防止参数被篡改
- 身份确认机制，每次请求都要验证是否合法
- APP中使用ssl pinning防止抓包操作
- 对所有请求和响应都进行加解密操作



![img](https://upload-images.jianshu.io/upload_images/19619362-bf6a7ef8faf466ab.png?imageMogr2/auto-orient/strip|imageView2/2/w/926/format/webp)

image.png

# Springboot中使用AES对称算法加密解密数据

1.在pom.xml文件中加入依赖



```xml
        <dependency>
            <groupId>commons-codec</groupId>
            <artifactId>commons-codec</artifactId>
            <version>1.10</version>
        </dependency>
```

1. 创建com.zhlab.demo.encrypt包,存放加解密核心代码
    代码不一一讲解了，目录如下:

![img](https:////upload-images.jianshu.io/upload_images/19619362-4567711897eef280.png?imageMogr2/auto-orient/strip|imageView2/2/w/388/format/webp)

代码结构

1. 加解密工具核心类:AesEncryptUtil.java



```java
package com.zhlab.demo.encrypt.utils;

import org.apache.tomcat.util.codec.binary.Base64;

import javax.crypto.Cipher;
import javax.crypto.KeyGenerator;
import javax.crypto.spec.SecretKeySpec;
import java.nio.charset.StandardCharsets;

/**
 * @ClassName AesEncryptUtil
 * @Description //AES加密解密工具
 * @Author singleZhang
 * @Email 405780096@qq.com
 * @Date 2021/1/7 0007 下午 3:28
 **/
public class AesEncryptUtil {
    private static final String KEY = "d7b85f6e214abcda";
    private static final String ALGORITHM_STR = "AES/ECB/PKCS5Padding";

    public static String base64Encode(byte[] bytes) {
        return Base64.encodeBase64String(bytes);
    }

    public static byte[] base64Decode(String base64Code) throws Exception {
        return Base64.decodeBase64(base64Code);
    }

    public static byte[] aesEncryptToBytes(String content, String encryptKey) throws Exception {
        KeyGenerator kgen = KeyGenerator.getInstance("AES");
        kgen.init(128);
        Cipher cipher = Cipher.getInstance(ALGORITHM_STR);
        cipher.init(Cipher.ENCRYPT_MODE, new SecretKeySpec(encryptKey.getBytes(), "AES"));
        return cipher.doFinal(content.getBytes(StandardCharsets.UTF_8));
    }

    public static String aesEncrypt(String content, String encryptKey) throws Exception {
        return base64Encode(aesEncryptToBytes(content, encryptKey));
    }

    public static String aesDecryptByBytes(byte[] encryptBytes, String decryptKey) throws Exception {
        KeyGenerator kgen = KeyGenerator.getInstance("AES");
        kgen.init(128);
        Cipher cipher = Cipher.getInstance(ALGORITHM_STR);
        cipher.init(Cipher.DECRYPT_MODE, new SecretKeySpec(decryptKey.getBytes(), "AES"));
        byte[] decryptBytes = cipher.doFinal(encryptBytes);
        return new String(decryptBytes, StandardCharsets.UTF_8);
    }

    public static String aesDecrypt(String encryptStr, String decryptKey) throws Exception {

        return aesDecryptByBytes(base64Decode(encryptStr), decryptKey);
    }

// TODO TEST

    public static void main(String[] args) throws Exception {
        String content = "SingleZhang2021";
        System.out.println("加密前：" + content);

        String encrypt = aesEncrypt(content, KEY);
        System.out.println(encrypt.length() + ":加密后：" + encrypt);

        String decrypt = aesDecrypt(encrypt, KEY);
        System.out.println("解密后：" + decrypt);
    }
}
```

1. 前端如果是web,需要一个js加密包crypto-js.js来实现AES加密
    前端部分代码如下:



```jsx
    var key = CryptoJS.enc.Utf8.parse("abcdef0123456780");
    //加密
    function Encrypt(word) {
        var srcs = CryptoJS.enc.Utf8.parse(word);
        var encrypted = CryptoJS.AES.encrypt(srcs, key, {
            mode: CryptoJS.mode.ECB,
            padding: CryptoJS.pad.Pkcs7
        });
        return encrypted.toString();
    }
    //解密
    function Decrypt(word) {
        var decrypt = CryptoJS.AES.decrypt(word, key, {
            mode: CryptoJS.mode.ECB,
            padding: CryptoJS.pad.Pkcs7
        });
        return CryptoJS.enc.Utf8.stringify(decrypt).toString();
    }
```

1. 在接口中，通过注解来选择性加解密,可以避免性能降低的风险



```kotlin
    /**
     * post请求->一个参数使用@Decrypt注解解密
     *
     * @param username
     * @return
     * @throws Exception
     */
    @Encrypt
    @Decrypt
    @PostMapping("/postStrData2")
    public String postStrData2(@RequestBody String username) throws Exception {
        System.out.println("username: " + username);

        return "加密字符串";
    }

    /**
     * post请求->发送并获取实体类数据,使用@Decrypt注解解密
     */
    @Encrypt
    @Decrypt
    @PostMapping("/encryptDto")
    public UserDto encryptDto(@RequestBody UserDto dto) {
        System.err.println(dto.getId() + "\t" + dto.getName());
        return dto;
    }
```

1. 在启动类上加上@EnableEncrypt 注解



```java
@EnableEncrypt
@SpringBootApplication(exclude = { DataSourceAutoConfiguration.class })
public class DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoApplication.class, args);
    }
}
```

好了,做完以上的工作,请求接口的数据就相对比之前安全许多,至少对一些爬虫工程师会造成一点困扰。

代码已传git

> [https://gitee.com/kaixinshow/springboot-note](https://links.jianshu.com/go?to=https%3A%2F%2Fgitee.com%2Fkaixinshow%2Fspringboot-note) 16Springboot-encrypt



