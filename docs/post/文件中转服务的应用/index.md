
> 前段时间做微信公众号服务碰到的一个问题，写公司内部微信管理平台时调微信api获取的图片地址都是带防盗链的，在浏览器端一般都是显示一张水印：

![](http://cdn.billyhu.com/blogUploads/1522323552000.png)

>原因：官方输出图片的时候，判断了来源(Referer)，就是从哪个网站访问这个图片，如果是你的网站去加载这个图片，那么Referer对应网站地址了，肯定是黑名单的。

>只要在访问时发送空的Referer，顺带模仿一个user-agent，则可以正常加载：

![](http://cdn.billyhu.com/blogUploads/1522323817000.jpeg)

## 文件中转服务（代理）

**node版本的代码：**

```js
const superagent = require('superagent');
const express = require('express');
const router = express.Router();

router.get('/fileproxy', function(req, res, next){
    let url = req.query.url;

    if (!url) {
        res.send({iret:-9, message:'please add your fileurl~~~'});
        return;
    }
    superagent.get(req.query.url)
        .set('Referer', '')
        .end(function(err, result) {

            if (err) {
                res.send({iret:-1, message:'get file fail~'});
                return;
            }

            res.writeHead(200, {
                'Content-Type': 'image/*'
            });

            res.end(result.body);
            
            return;
        });

})
```
上面用superagent模块发送了空的Referer，把图片文件代理到了自己服务器。所以最后的做法就是在图片链接中加上自己中转服务的地址前缀，我用自己的小鸡给提供了个中转地址，感兴趣的可以试一哈：

[http://x.billyhu.com/fileproxy](http://x.billyhu.com/fileproxy)

例如：

[http://x.billyhu.com/fileproxy?url=http://mmbiz.qpic.cn/mmbiz_jpg/88GuXHWRSgghpv0rkibtqF3MdTUzSlYWbE3l7oyMlDfb2s61POTIrcVTKqR9LvVc3icsfvjk1grbHwFt8kzsCd7g/0?wx_fmt=jpeg](http://www.billyhu.com/fileproxy?url=http://mmbiz.qpic.cn/mmbiz_jpg/88GuXHWRSgghpv0rkibtqF3MdTUzSlYWbE3l7oyMlDfb2s61POTIrcVTKqR9LvVc3icsfvjk1grbHwFt8kzsCd7g/0?wx_fmt=jpeg)

### 后续
TODO:这种模式还有很多应用场景发掘，再有在平时用到的我再慢慢补上来

