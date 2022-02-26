---
title: 不知道怎么描述的脚本，用来检测ip存活啥的
hidden: false
abbrlink: 31550
date: 2021-11-06 00:00:00
---

自用的脚本，正常人应该用不到

<!--more-->
- 解决了：
最近的任务每次都要一个个排查同事给的资产...    
不同漏扫工具接受的格式不一样   
- 使用了：
正则   
使用socket、ping3和httpx（可用http2.0，某些时候比requests好用一些）进行检测 
输出为csv格式

```py
# -*- coding: utf-8 -*- 

from os import error
import re
import socket
import csv
import httpx
from ping3 import ping, verbose_ping


url_down = lambda url: print("\033[0,33;47m%s 不可访问(URL)\033[0m" % url)
connect_down = lambda url: print("\033[0,31;47m%s 不可访问(ping不通)\033[0m" % url)
url_as_200 = lambda url: print("\033[0;32;47m%s 可访问(200)\033[0m" % url)
ping_ok = lambda url: print("\033[0;32;47m%s 可访问(能ping通）\033[0m" % url)



f = open(r"C:\Users\Bruce\Desktop\target.txt","r+")
awvs = csv.writer(open(r"C:\Users\Bruce\Desktop\awvs.csv","a",newline=""))
nessus = csv.writer(open(r"C:\Users\Bruce\Desktop\nessus.csv","a",newline=""))
Tip = csv.writer(open(r"C:\Users\Bruce\Desktop\troubleip.csv","a",newline=""))

header = {
    'user-agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/85.0.4183.121 Safari/537.36',
    'accept-language': 'zh-CN,zh;q=0.9'
}
client = httpx.Client(http2=True, verify=False)

def save(ip,port,url):

    if(url == "" and port != ""):
        nessus.writerow([ip+ ":" +port])
        awvs.writerow([ip+ ":" +port])
  
    elif(port == "" and url == ""):
        nessus.writerow([ip])
        awvs.writerow([ip])

    else:
        awvs.writerow([url])
        
        url = url.replace("https://","")
        url = url.replace("http://","")
        url = re.sub( r'/.*' , "" , url )
        url = re.sub( r':.*' , "" , url )

        nessus.writerow([url])


def testip(i):
    # url  去http 和 /
    url = i.replace("https://","")
    url = i.replace("http://","")
    url = re.sub( r'/.*' , "" , url )
    url = re.sub( r':.*' , "" , url )

    # 找ip
    try:
        #ip = socket.gethostbyaddr(url) #name
        ip = socket.gethostbyname(url) #_ex
    except: #socket.herror
        #后续
        connect_down(i)
        Tip.writerow([i])
        print(url)
        return 
    
    # 有port
    s = socket.socket()
    s.settimeout(3)
    if(":" in url):
        port = re.search(i,r":[0-9]+")[1:]
        try:
            s.connect((ip,port))
            ping_ok(ip + ":" + port)
            save(ip,port,"")
            return
            #这里要不保存ip:port和ip？
        except:
            connect_down(url)
            Tip.writerow([i])
            return
    s.close()    

    # 无port，ping3
    # res = ping(ip,s)
    if( ping(ip,src_addr = None) != None):
        save(ip,"","")
        ping_ok(i)
    else:
        Tip.writerow([i])


'''        
url/ip/ip+port

awvs: 可用http://ip:port/.*，不能ip:port/.*，其他都行
nessus：ip、domain
'''


def main():

    pattern = re.compile(r'[a-z]',re.I)

    for i in f:
        #url
        i = i[:-1]
 
        url = ""
        if(re.search("http",i) == None):
            url = "http://"
        url = url + i
    
        
        try:
            #r = requests.get(url, timeout=10)
            r = client.get( url, headers=header, timeout=10)
        except:
            url_down(i)
            testip(url)
            continue
        
        if(r.status_code == 200):
            url_as_200(url)
            save("","",url)
                


if __name__ == '__main__':
    main()
'''
检测url能否访问
    能->保存
    不能->检测ip(:port)是否可达()

ip(:port)
    给url->获得ip   ~~（byname 和 byaddr有啥区别？）~~
        没有的话保存url
    
    给ip:port(/*.html)->
            检测ip(:port)是否可达

保存：
三类：url/ip/ip+port

    AWVS：
    Nessus：

'''

```