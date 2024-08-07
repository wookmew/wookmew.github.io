---
title: Python AES解密
date: 2017-11-24 11:33:16
tags: 密码
categories: Python
---

> [AES](https://zh.wikipedia.org/wiki/%E9%AB%98%E7%BA%A7%E5%8A%A0%E5%AF%86%E6%A0%87%E5%87%86)即高级加密标准，这次准备是采用AES256/CBC/PKCS7Padding。  

<!--more--> 

## 遇到的坑  
调试过程中遇到了不少坑，说白了就是客户端加密出来的我解不出来，我加密的客户端解不出来，各自加密各自都能解密。当时很是费解，因为虽然我们都是调的各自的库(自己使用的是pycrypto)，但是在密钥一致的情况下，不应该是这个情况。根据[WIKI](https://zh.wikipedia.org/wiki/%E9%AB%98%E7%BA%A7%E5%8A%A0%E5%AF%86%E6%A0%87%E5%87%86)上的描述，也该是如此。    
询问客户端key，iv是如何生成，得到的回复是指定16位的key之后(IOS表示只能输入16位)，剩下的都是自动填充，至于填充的是什么，告诉我的是0。但是这不对啊，几次反复之后我们决定固定写死iv，我也不去尝试解通过AES128来解了，既然客户端确定是AES256／CBC，那就先把这个试个遍再说。最后在sf上找到了这篇[Encrypt & Decrypt using PyCrypto AES 256](https://stackoverflow.com/questions/12524994/encrypt-decrypt-using-pycrypto-aes-256)，简单的改写了下:  

```Python
import json
import hashlib
import base64
from Crypto import Random
from Crypto.Cipher import AES


class AESCipher(object):

    def __init__(self, key, iv):
        self.bs = 32
        self.iv = iv
        self.key = hashlib.sha256(key.encode()).digest()

    def encrypt(self, raw):
        raw = self._pad(raw)
        iv = self.iv
        cipher = AES.new(self.key, AES.MODE_CBC, iv)
        return base64.b64encode(cipher.encrypt(raw))

    def decrypt(self, enc):
        enc = base64.b64decode(enc)
        iv = self.iv
        cipher = AES.new(self.key, AES.MODE_CBC, iv)
        return self._unpad(cipher.decrypt(enc))

    def _pad(self, s):
        return s + (self.bs - len(s) % self.bs) * chr(self.bs - len(s) % self.bs)

    @staticmethod
    def _unpad(s):
        return s[:-ord(s[len(s)-1:])]  # 去除末尾的填充字符 
```    

主要就是改了下iv的生成处理方式，在经过客户端的调试(同样进行iv固定)，最终解决了这个问题。刚爬出AES，只闻道友请留步，又掉进JSON的坑。

## 关于JSON解析  
关于json的解析，很简单，这里给个简单的示例。对于dictionaries，keys需要是字符串类型(字典中任何非字符串类型的key在编码时会先转换为字符串)：  

```Python
In [1]: a = {1:2}

In [2]: import json

In [3]: json.dumps(a)
Out[3]: '{"1": 2}'

In [4]: json.loads(json.dumps(a))
Out[4]: {u'1': 2}
```  

试了下，**ValueError: Extra data: line 5 column 2 - line 5 column 8 (char 74 - 80)**。嗯，意料之外，情理之外。先冷静一下，又看了下json的文档，没有太大的发现，又来盯着报错看。即**line 5 column 2 - line 5 column 8**，等等，第五行第二列到第五行第八列引起的报错，马萨卡，这里是薛定谔的空格么。  
恍惚中想到了pprint，决定先打印出来看看，**'{\n  "h" : "haha",\n  "hehe" : "hehe",\n  "heiheihei" : "heiheihei"\n}\x06\x06\x06\x06\x06\x06'**。幸运女神对我微笑，strip一波带走，总算扯完这个问题了，路上绕了不少，聊以此记。  

## 后记  
自己在进行测试的时候，发现json.loads还是会报错，strip只针对了一种情况。这个就尴尬了，看来自己只是治标不治本。想到AES明文串有长度要求，即16的倍数，所以会有个自动填充的问题。默认情况下，会使用 PKCS5 的方法进行补白，方式就是将剩余的位数作为二进制的charcode填充直到对齐。也就是**_pad**这个方法，那对应的将这些留白删掉就可以了。自己之前未曾使用unpad模块，这里加上就大功告成了。 


