## ## 分析pandownload，aria2下载原理

wireshark抓包工具监听网卡

下载一份300mRAR压缩包

首先模拟ie登陆到百度云，获取下载直链

请求如下

```
POST /rest/2.0/pcs/file?app_id=250528&method=locatedownload&check_blue=1&es=1&esl=1&path=%2Fsoftware%2FJAVA%2FIDEA%2FJetBrains+IntelliJ+IDEA+14.0.3.rar&ver=4.0&dtype=1&err_ver=1.0&ehps=0&clienttype=8&channel=00000000000000000000000000000000&version=6.8.9.1&devuid=BDIMXV2-O_A82A5806AB47B6877C42D007E68A2589-C_0-D_F391D5823BBD-M_336BC7BDBD52-V_0ADD64E7&rand=1ffa72277838fd5647dedf6ac8da2831997ebbc6&time=1587191031&vip=0&wp_retry_num=2 HTTP/1.1
Host: d.pcs.baidu.com
Accept: */*
User-Agent: netdisk;6.8.9.1;PC;PC-Windows;10.0.17763;WindowsBaiduYunGuanJia
Cookie: BAIDUID=2A970F53A8C83C07C127D0281CA65714:FG=1; BDUSS=mlsMEt5eG1LRE5tRE1pb3U0ODJQeWI5ejY2Q1JHQkJFM1pLem0tUXRiWkg5M2hkSVFBQUFBJCQAAAAAAAAAAAEAAADxAXxHc3Vu2LxzaGluAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAEdqUV1HalFdU; pan_login_way=1; PANWEB=1; STOKEN=03d12ad633f79eb87ebc05b93898ec3d3ac939ae655ac4430f9674574f89e01c; SCRC=7657dfe35b86dc962a211591c09366ff; PANPSC=7085118518127003151%3ACU2JWesajwAEGUL6ymdPznt6%2Fu0CRQmzyo1yblh16HE6uSsH3QG3sEci2O0YbEjpTJupUz138hgZg8H0D%2BBoQeFWO2pyF0kqh3eRtCOoiQkO1MJzpmpaMb9TYRmD%2FUzPlHxi%2BpK%2BDGnS%2Fb0eg7IHE7tbT0VT6%2Bsu3w1F3sQDl37Z81l%2BOzvAFj9yggFRCkDaO6gB4KX1MEY%3D
Content-Length: 1
Content-Type: application/x-www-form-urlencoded
```

部分响应

```html
HTTP/1.1 200 OK
Connection: keep-alive
Content-Type: text/plain; charset=utf-8
Date: Sat, 18 Apr 2020 06:34:32 GMT
Flow-Level: 3
Http-X-Isis-Logid: 2518910272396149066
Remote-Ip: d.pcs.baidu.com
Server: nginx
Vary: Accept-Encoding
X-Bs-Pcs-Extra: v_user_id=1199309297&user_terminal=winygj&iv=0&digest=00a21d7463882a474c6c24f4dbe68192&visitor_ip=101.88.211.210&method=locatedownload&user_type=default&user_id=1199309297&app_id=250528&file_type=rar&user_operator=ct&file_size=296188888&fs_id=1045229247321623&freeispflag=-100&xflag=
X-Bs-Remote-Ip: 10.192.103.39
X-Bs-Request-Id: bmowMi1vYmplY3QwMC1yMDMtMDItMDk5Lm5qMDIuYmFpZHUuY29tOjEwLjE5Mi4xMDMuMzk6MjAwMDoyNTE4OTEwMjcyMzk2MTQ5MDY2OjIwMjAtMDQtMTggMTQ6MzQ6MzI=
X-Pcs-Member: 0
X-Poms-Key: 140000a21d7463882a474c6c24f4dbe68192493f786c000011a77bd8
Yld: 213067250784416991
Yme: ZIGW+iozQE0UaSsGTHb+qnFItfkASwL2tANGySKDmO0=
Transfer-Encoding: chunked

c66
{"client_ip":"101.88.211.210","urls":[{"url":"https://d1.baidupcs.com/file/00a21d7463882a474c6c24f4dbe68192?bkt=en-713b21d6dbc3139822170d1309366a6c284f82ac8427cb1dea73e766f15bdbe91840a6a6dcf5bdaa32aed73bb3105686e4342626cb14bdfe0e4d742632ad4d93&xcode=6b4ea02c85e1d51035bc2a4f124832a2923b516c071f254a4b92fc81fa1d912d75dcc1656c1a807199acac2ee322c89fe801268afdb15995&fid=21217187-250528-1045229247321623&time=1587191672&sign=FDTAXUGERLQlBHSKfa-DCb740ccc5511e5e8fedcff06b081203-%2Fpq92kVEd77qNCkpy7eQsqajpSc%3D&to=d1&size=296188888&sta_dx=296188888&sta_cs=411&sta_ft=rar&sta_ct=7&sta_mt=4&fm2=MH%2CYangquan%2CAnywhere%2C%2Cshanghai%2Cct&ctime=1422534759&mtime=1585294292&resv0=-1&resv1=0&resv2=rlim&resv3=5&resv4=296188888&vuk=21217187&iv=0&htype=&randtype=&esl=1&newver=1&newfm=1&secfm=1&flow_ver=3&pkey=en
```

通过js获取json中多个uri下载源，rank估计为uri速度排名

交给aria2处理，通过多线程指定下载区间range:bytes=0-4194303等，并行下载再组装实现下载速率翻倍

响应 ：Content-Range: bytes 0-4194303/296188888

```html
Accept: */*,application/metalink4+xml,application/metalink+xml
Host: nbcache00.baidupcs.com
Want-Digest: SHA-512;q=1, SHA-256;q=1, SHA;q=0.1
Range: bytes=0-4194303

HTTP/1.1 206 Partial Content
Server: JSP3/2.0.14
Date: Sat, 18 Apr 2020 06:34:33 GMT
Content-Type: application/x-rar-compressed
Content-Length: 4194304
Connection: keep-alive
ETag: 00a21d7463882a474c6c24f4dbe68192
Last-Modified: Fri, 08 Nov 2019 07:47:41 GMT
Expires: Tue, 21 Apr 2020 06:24:00 GMT
Content-Range: bytes 0-4194303/296188888
```





