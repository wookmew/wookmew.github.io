---
title: requests解析页面过慢一解
date: 2017-06-24 15:22:33
tags: requests 
categories: Python
---
> 近日遇到的一个解析xml的问题，主要出在requests的返回上。顺带吐槽下，google “python xml”时，出现的内容大都是[xml.etree.ElementTree](https://docs.python.org/2/library/xml.etree.elementtree.html)，但实践之后，还是觉得更胜一筹。  

<!--more-->

## 关于content与text  

之前自己爬取网站的时候，一般都用text进行处理，并没有出现太大差错。但这次访问的xml信息比较大，比如像[apple排行版](http://ax.itunes.apple.com/WebObjects/MZStoreServices.woa/ws/RSS/topfreeapplications/limit=200/genre=6015/xml)信息的抓取，整体数据保存下来已经有1.3M了。
经过测试，response.text用时47.2s,response.content用时2.4s。好吧，一下子就解决了自己解析时间过长的问题(之前一直是以为xml.etree.ElementTree效率太慢，但发现换到bs效果还是不理想)。
我之后跑到github上看了[requests](https://github.com/requests/requests)的源码。

关于[content的源码](https://github.com/requests/requests/blob/master/requests/models.py#L813)  
    
```Python
@property
    def content(self):
        """Content of the response, in bytes."""

        if self._content is False:
            # Read the contents.
            if self._content_consumed:
                raise RuntimeError(
                    'The content for this response was already consumed')

            if self.status_code == 0 or self.raw is None:
                self._content = None
            else:
                self._content = bytes().join(self.iter_content(CONTENT_CHUNK_SIZE)) or bytes()

        self._content_consumed = True
        # don't need to release the connection; that's been handled by urllib3
        # since we exhausted the data.
        return self._content
```

关于[text的源码](https://github.com/requests/requests/blob/master/requests/models.py#L833)   
    
```Python
@property
    def text(self):
        """Content of the response, in unicode.
        If Response.encoding is None, encoding will be guessed using
        ``chardet``.
        The encoding of the response content is determined based solely on HTTP
        headers, following RFC 2616 to the letter. If you can take advantage of
        non-HTTP knowledge to make a better guess at the encoding, you should
        set ``r.encoding`` appropriately before accessing this property.
        """

        # Try charset from content-type
        content = None
        encoding = self.encoding

        if not self.content:
            return str('')

        # Fallback to auto-detected encoding.
        if self.encoding is None:
            encoding = self.apparent_encoding

        # Decode unicode from given encoding.
        try:
            content = str(self.content, encoding, errors='replace')
        except (LookupError, TypeError):
            # A LookupError is raised if the encoding was not found which could
            # indicate a misspelling or similar mistake.
            #
            # A TypeError can be raised if encoding is None
            #
            # So we try blindly encoding.
            content = str(self.content, errors='replace')

        return content
```  

可以看到在注释中分别有**Content of the response, in bytes.**，**Content of the response, in unicode.**两句。好吧，如此一来一目了然，text函数进行了解码操作。这也解决了为啥解析过程中cpu居高不下的问题，一开始自己还尝试多进程去解，对自己感到无语。

## 一些后记
之后抱着疑惑在[官方文档](http://cn.python-requests.org/zh_CN/latest/user/quickstart.html#id3)翻阅一番，找到了。还是自己看文档不够仔细啊，折腾了徐久，写下这篇聊以警醒吧。


    

