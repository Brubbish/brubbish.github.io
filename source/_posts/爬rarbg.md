---
title: RARBG上种子收集脚本
abbrlink: 38266
date: 2021-12-05 00:00:00
---


跟着[使用winafl对迅雷的torrent解析逻辑进行fuzz](https://bbs.pediy.com/thread-265958.htm)做的时候收集BT语料库（种子种子....哈哈哈哈哈哈哈哈哈哈哈）的脚本      
收集RARBG上的一些.torrent文件

<!--more-->

弄了一半发现这个站对爬虫做了点限制，并且24小时内下载超过100个左右的种子就只能用磁链了

虽然磁链好像能转BT来着，不过没试过

只能手动切换代理，没写自动切换的代码，可参考：https://www.jianshu.com/p/5e4d7529476d

讲真选择爬RARBG真是...选了个麻烦的


```py
# -*- coding: utf-8 -*-

import requests
from bs4 import BeautifulSoup
import re
import time
from lxml import etree
import urllib.parse
import os
import random
import string


kv = {
    'Host': 'rarbgmirror.org',
    'Cache-Control': 'max-age=0',
    'DNT': '1',
    'Upgrade-Insecure-Requests': '1',
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/96.0.4664.55 Safari/537.36 Edg/96.0.1054.41',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9',
    'Accept-Encoding': 'gzip, deflate',
    'Accept-Language': 'zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6'}


cookie = ""
cookie_dict = {i.split("=")[0]: i.split("=")[-1] for i in cookie.split("; ")}


proxies = {
    "http": '127.0.0.1:7890',
    "https": '127.0.0.1:7890'
}


depth = 2
categories = { "movie": "14;17;42;44;45;46;47;48;50;51;52;54", "xxx": "2;4","tv": "2;18;41;49",
              "game": "2;27;28;29;30;31;32;40;53", "music": "2;23;24;25;26", "software": "2;33;34;43"}
opt = ["filename", "data", "size", "seeders", "leechers"]
orderby = ["ASC", "DESC"]

url = r"http://rarbgmirror.org"


def get_base():
    turl = []
    f = open(r"C:\Users\Bruce\Desktop\临时\\torrent\\url.txt", "r")
    for i in f:
        turl.append(i)
    f.close()
    return turl


def get_down(category):
    # ground = get_base()
    turl = []
    rr = re.compile(r'href="/torrent/.*?"')
    for j in range(1, depth+1):
        for select in opt:
            for order in orderby:
                catch = url + r"/torrents.php?category=" + categories[category] + \
                    r'&search=&order=' + select + \
                    r"&by=" + order + r"&page=" + str(j)
                print(catch)
                r = requests.get(
                    catch, headers=kv, cookies=cookie_dict, timeout=30, proxies=proxies, )

                if (('pure flooding' in r.text) or r.status_code != 200):
                    print("该换代理了?\n" + catch +
                          ' error:' + str(r.status_code))
                    os.system("pause")
                    r = requests.get(
                        catch, headers=kv, cookies=cookie_dict, timeout=30, proxies=proxies, )
                content = BeautifulSoup(r.text, 'lxml').find_all(
                    'tr', attrs={'class': 'lista2'})

                for res in content:
                    stub = rr.findall(str(res))[0].replace(
                        r'href="/torrent/', "")[:-1]
                    # if(stub not in ground):

                    turl.append(stub)
            time.sleep(2)
            
            
            download(turl,category)
            turl = []
            
            print("\033[0;32;47m%s %s ,page %s 访问完毕\033[0m" %
                  (categories[category], select, j))
        time.sleep(2)

    time.sleep(2)



def download(turl, category):
    fd = open(r"C:\Users\Bruce\Desktop\临时\\torrent\\url.txt", "a")

    for i in turl:

        try:
            p = requests.get(url=url + '/torrent/' + i, headers=kv,
                             cookies=cookie_dict, timeout=300,  proxies=proxies,)
            down_url = re.search(r'/download.php.*\.torrent', p.text).group()
        except:
            f = open(r"C:\Users\Bruce\Desktop\临时\torrent\miss.txt","w")
            f.write(url + '/torrent/' + i)
            f.close()
            print("miss one")
            continue

        fname = re.sub(r'/download.php.*?f=', "", down_url)
        fname = urllib.parse.unquote(fname)
        str = r"[\/\\\:\*\?\"\<\>\|]"  # '/ \ : * ? " < > |'
        fname = re.sub(str, "-", fname)

        down_url = url + down_url

        try:
            download = requests.get(
            url=down_url, headers=kv, cookies=cookie_dict, timeout=30, proxies=proxies,)
        except : #requests.exceptions.RequestException :
            print("网不好了？")
            input()
            
        if (('pure flooding' in download.text) or download.status_code != 200):
            print("该换代理了!\n")
            os.system("pause")
            download = requests.get(
                url=down_url, headers=kv, cookies=cookie_dict, timeout=30, proxies=proxies,)

        try:
            f = open(r'C:\Users\Bruce\Desktop\临时\torrent\\' +
                     category + r'\\' + fname, "wb")
        except:
            fname = ''.join(random.sample(
                string.ascii_letters + string.digits, 8)) + '.torrent'
            f = open(r'C:\Users\Bruce\Desktop\临时\torrent\\' +
                     category + '\\' + fname, "wb")
        f.write(download.content)
        f.close()

        print("\033[0;32;47m%s 已保存\033[0m" % (fname))

        fd.write(i + '\n')
        time.sleep(2)
    fd.close()


def main():
    for i in categories.keys():
        if not os.path.exists(r"C:\Users\Bruce\Desktop\临时\torrent\\" + i):
            os.mkdir(r"C:\Users\Bruce\Desktop\临时\torrent\\" + i)
        get_down(i)



if __name__ == '__main__':
    main()
# https://www.jianshu.com/p/5e4d7529476d


```