```python
现象1：
P@Duk1906 MINGW64 ~/Documents/GitHub/Duk1901/Python (master)
$ git push
fatal: unable to access 'https://github.com/Duk1906/Duk1901.git/': Failed to connect to github.com port 443: Timed out
            
现象2：浏览器打不开github地址
https://github.com/Duk1906/Duk1901
            

解决：
        
1. 查询GitHub 域名解析网址：
http://tool.chinaz.com/dns?type=1&host=github.com&ip=   

2. 本地ping一下，选择延迟最小的ip

PS C:\Users\P> ping 52.192.72.89
正在 Ping 52.192.72.89 具有 32 字节的数据:
来自 52.192.72.89 的回复: 字节=32 时间=114ms TTL=32
....
    最短 = 114ms，最长 = 114ms，平均 = 114ms  # 114已经是我电脑能解析到ip中延迟最小的了，作为对比，baidu.com才45ms

    
3. 修改host文件
Windows  C:\Windows\System32\drivers\etc\hosts   # 右键--属性--安全--编辑--添加对应用户写入权限 
Linux    /etc/hosts

4. 在hosts文件里添加延迟最小ip地址，如下，我的是52.192.72.89
52.192.72.89   github.com

5. 保存，重试，ok了


后续------------------------------------------
文档传上去，发现图片打不开，找到图片位置，查看图片地址，同样方法处理下raw.githubusercontent.com，问题解决
185.199.110.133  raw.githubusercontent.com



```

