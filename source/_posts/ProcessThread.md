---
title: Python 进程与线程
date: 2018-02-08 16:49:01
tags:
	- Python
	- Process
	- Thread
categories: Python
---
Python 多进程、多线程的理解和简单使用。
<!-- more -->

## 进程
### 多任务的引入
+ 现实生活中,有很多的场景中的事情是同时进行的，比如开车的时候 手和脚共同来驾驶汽车，再比如唱歌跳舞也是同时进行的；

+ 如下程序，来模拟“唱歌跳舞”这件事情
```
from time import sleep

def sing():
    for i in range(3):
        print("正在唱歌...%d"%i)
        sleep(1)

def dance():
    for i in range(3):
        print("正在跳舞...%d"%i)
        sleep(1)

if __name__ == '__main__':
    sing() #唱歌
    dance() #跳舞
```
+ 运行结果如下： 
![](http://image.ixysec.com/image/process01.png)
+ 很显然刚刚的程序并没有完成唱歌和跳舞同时进行的要求
+ 如果想要实现“唱歌跳舞”同时进行，那么就需要一个新的方法，叫做：多任务

### 多任务的概念
什么叫“多任务”呢？简单地说，就是操作系统可以同时运行多个任务。打个比方，你一边在用浏览器上网，一边在听MP3，一边在用Word赶作业，这就是多任务，至少同时有3个任务正在运行。还有很多任务悄悄地在后台同时运行着，只是桌面上没有显示而已。

现在，多核CPU已经非常普及了，但是，即使过去的单核CPU，也可以执行多任务。由于CPU执行代码都是顺序执行的，那么，单核CPU是怎么执行多任务的呢？

答案就是操作系统轮流让各个任务交替执行，任务1执行0.01秒，切换到任务2，任务2执行0.01秒，再切换到任务3，执行0.01秒……这样反复执行下去。表面上看，每个任务都是交替执行的，但是，由于CPU的执行速度实在是太快了，我们感觉就像所有任务都在同时执行一样。

真正的并行执行多任务只能在多核CPU上实现，但是，由于任务数量远远多于CPU的核心数量，所以，操作系统也会自动把很多任务轮流调度到每个核心上执行。

![](http://image.ixysec.com/image/process02.gif)

### 进程的创建-fork

#### 进程 VS 程序
编写完毕的代码，在没有运行的时候，称之为程序
正在运行着的代码，就称为进程
进程，除了包含代码以外，还有需要运行的环境等，所以和程序是有区别的
#### fork()
Python的os模块封装了常见的系统调用，其中就包括fork，可以在Python程序中轻松创建子进程：
```
import os
# 注意，fork函数，只在Unix/Linux/Mac上运行，windows不可以
pid = os.fork()

if pid == 0:
    print('哈哈1')
else:
    print('哈哈2')
```
运行结果： 
![](http://image.ixysec.com/image/process03.jpg)
##### 说明：
+ 程序执行到os.fork()时，操作系统会创建一个新的进程（子进程），然后复制父进程的所有信息到子进程中
+ 然后父进程和子进程都会从fork()函数中得到一个返回值，在子进程中这个值一定是0，而父进程中是子进程的pid号

在Unix/Linux操作系统中，提供了一个fork()系统函数，它非常特殊。

普通的函数调用，调用一次，返回一次，但是fork()调用一次，返回两次，因为操作系统自动把当前进程（称为父进程）复制了一份（称为子进程），然后，分别在父进程和子进程内返回。

子进程永远返回0，而父进程返回子进程的ID。

这样做的理由是，一个父进程可以fork出很多子进程，所以，父进程要记下每个子进程的ID，而子进程只需要调用getppid()就可以拿到父进程的ID。

#### getpid()、getppid()
```
import os

rpid = os.fork()
if rpid<0:
    print("fork调用失败。")
elif rpid == 0:
    print("我是子进程（%s），我的父进程是（%s）"%(os.getpid(),os.getppid()))
    x+=1
else:
    print("我是父进程（%s），我的子进程是（%s）"%(os.getpid(),rpid))

print("父子进程都可以执行这里的代码")
```
运行结果：
![](http://image.ixysec.com/image/process04.jpg)

### 多进程修改全局变量
```
import os
import time

num = 0

pid = os.fork()

if pid == 0:
    num += 1
    print("子进程---num=%d"%num)
else:
    time.sleep(1)
    print("父进程---num=%d"%num)

```
运行结果: 
![](http://image.ixysec.com/image/process05.jpg)

#### 总结：
+ 多进程中，每个进程中所有数据（包括全局变量）都各有拥有一份，互不影响

### 多次fork问题
如果在一个程序，有2次的fork函数调用，是否就会有3个进程呢？
```
import os
import time

# 注意，fork函数，只在Unix/Linux/Mac上运行，windows不可以
pid = os.fork()
if pid == 0:
    print('哈哈1')
else:
    print('哈哈2')

pid = os.fork()
if pid == 0:
    print('哈哈3')
else:
    print('哈哈4')

time.sleep(1)
```
运行结果： 
![](http://image.ixysec.com/image/process06.jpg)
#### 说明：
![](http://image.ixysec.com/image/Snip20160829_6.png)
#### 父子进程的执行顺序
父进程、子进程执行顺序没有规律，完全取决于操作系统的调度算法

### 进程的创建-multiprocessing
如果你打算编写多进程的服务程序，Unix/Linux无疑是正确的选择。由于Windows没有fork调用，难道在Windows上无法用Python编写多进程的程序？

由于Python是跨平台的，自然也应该提供一个跨平台的多进程支持。multiprocessing模块就是跨平台版本的多进程模块。

multiprocessing模块提供了一个Process类来代表一个进程对象，下面的例子演示了启动一个子进程并等待其结束：
```
from multiprocessing import Process
import os

# 子进程要执行的代码
def run_proc(name):
    print('子进程运行中，name= %s ,pid=%d...' % (name, os.getpid()))

if __name__=='__main__':
    print('父进程 %d.' % os.getpid())
    p = Process(target=run_proc, args=('test',))
    print('子进程将要执行')
    p.start()
    p.join()
    print('子进程已结束')
```
![](http://image.ixysec.com/image/process07.jpg)
#### 说明
+ 创建子进程时，只需要传入一个执行函数和函数的参数，创建一个Process实例，用start()方法启动，这样创建进程比fork()还要简单。
+ join()方法可以等待子进程结束后再继续往下运行，通常用于进程间的同步。

#### Process语法结构如下：
Process([group [, target [, name [, args [, kwargs]]]]])
+ target：表示这个进程实例所调用对象；
+ args：表示调用对象的位置参数元组；
+ kwargs：表示调用对象的关键字参数字典；
+ name：为当前进程实例的别名；
+ group：大多数情况下用不到；

Process类常用方法：
+ is_alive()：判断进程实例是否还在执行；
+ join([timeout])：是否等待进程实例执行结束，或等待多少秒；
+ start()：启动进程实例（创建子进程）；
+ run()：如果没有给定target参数，对这个对象调用start()方法时，就将执行对象中的run()方法；
+ terminate()：不管任务是否完成，立即终止；

Process类常用属性：
+ name：当前进程实例别名，默认为Process-N，N为从1开始递增的整数；
+ pid：当前进程实例的PID值；

#### 实例1
```
from multiprocessing import Process
import os
from time import sleep

# 子进程要执行的代码
def run_proc(name, age, **kwargs):
    for i in range(10):
        print('子进程运行中，name= %s,age=%d ,pid=%d...' % (name, age,os.getpid()))
        print(kwargs)
        sleep(0.5)

if __name__=='__main__':
    print('父进程 %d.' % os.getpid())
    p = Process(target=run_proc, args=('test',18), kwargs={"m":20})
    print('子进程将要执行')
    p.start()
    sleep(1)
    p.terminate()
    p.join()
    print('子进程已结束')
```
运行结果:
![](http://image.ixysec.com/image/process08.jpg)
#### 实例2
```
from multiprocessing import Process
import time
import os

#两个子进程将会调用的两个方法
def  worker_1(interval):
    print("worker_1,父进程(%s),当前进程(%s)"%(os.getppid(),os.getpid()))
    t_start = time.time()
    time.sleep(interval) #程序将会被挂起interval秒
    t_end = time.time()
    print("worker_1,执行时间为'%0.2f'秒"%(t_end - t_start))

def  worker_2(interval):
    print("worker_2,父进程(%s),当前进程(%s)"%(os.getppid(),os.getpid()))
    t_start = time.time()
    time.sleep(interval)
    t_end = time.time()
    print("worker_2,执行时间为'%0.2f'秒"%(t_end - t_start))

#输出当前程序的ID
print("进程ID：%s"%os.getpid())

#创建两个进程对象，target指向这个进程对象要执行的对象名称，
#args后面的元组中，是要传递给worker_1方法的参数，
#因为worker_1方法就一个interval参数，这里传递一个整数2给它，
#如果不指定name参数，默认的进程对象名称为Process-N，N为一个递增的整数
p1=Process(target=worker_1,args=(2,))
p2=Process(target=worker_2,name="dongGe",args=(1,))

#使用"进程对象名称.start()"来创建并执行一个子进程，
#这两个进程对象在start后，就会分别去执行worker_1和worker_2方法中的内容
p1.start()
p2.start()

#同时父进程仍然往下执行，如果p2进程还在执行，将会返回True
print("p2.is_alive=%s"%p2.is_alive())

#输出p1和p2进程的别名和pid
print("p1.name=%s"%p1.name)
print("p1.pid=%s"%p1.pid)
print("p2.name=%s"%p2.name)
print("p2.pid=%s"%p2.pid)

#join括号中不携带参数，表示父进程在这个位置要等待p1进程执行完成后，
#再继续执行下面的语句，一般用于进程间的数据同步，如果不写这一句，
#下面的is_alive判断将会是True，在shell（cmd）里面调用这个程序时
#可以完整的看到这个过程，大家可以尝试着将下面的这条语句改成p1.join(1)，
#因为p1需要2秒以上才可能执行完成，父进程等待1秒很可能不能让p1完全执行完成，
#所以下面的print会输出True，即p1仍然在执行
p1.join()
print("p1.is_alive=%s"%p1.is_alive())
```
执行结果:
![](http://image.ixysec.com/image/process09.jpg)

### 进程的创建-Process子类
创建新的进程还能够使用类的方式，可以自定义一个类，继承Process类，每次实例化这个类的时候，就等同于实例化一个进程对象，请看下面的实例：
```
from multiprocessing import Process
import time
import os

#继承Process类
class Process_Class(Process):
    #因为Process类本身也有__init__方法，这个子类相当于重写了这个方法，
    #但这样就会带来一个问题，我们并没有完全的初始化一个Process类，所以就不能使用从这个类继承的一些方法和属性，
    #最好的方法就是将继承类本身传递给Process.__init__方法，完成这些初始化操作
    def __init__(self,interval):
        Process.__init__(self)
        self.interval = interval

    #重写了Process类的run()方法
    def run(self):
        print("子进程(%s) 开始执行，父进程为（%s）"%(os.getpid(),os.getppid()))
        t_start = time.time()
        time.sleep(self.interval)
        t_stop = time.time()
        print("(%s)执行结束，耗时%0.2f秒"%(os.getpid(),t_stop-t_start))

if __name__=="__main__":
    t_start = time.time()
    print("当前程序进程(%s)"%os.getpid())        
    p1 = Process_Class(2)
    #对一个不包含target属性的Process类执行start()方法，就会运行这个类中的run()方法，所以这里会执行p1.run()
    p1.start()
    p1.join()
    t_stop = time.time()
    print("(%s)执行结束，耗时%0.2f"%(os.getpid(),t_stop-t_start))
```
### 进程池Pool
当需要创建的子进程数量不多时，可以直接利用multiprocessing中的Process动态成生多个进程，但如果是上百甚至上千个目标，手动的去创建进程的工作量巨大，此时就可以用到multiprocessing模块提供的Pool方法。

初始化Pool时，可以指定一个最大进程数，当有新的请求提交到Pool中时，如果池还没有满，那么就会创建一个新的进程用来执行该请求；但如果池中的进程数已经达到指定的最大值，那么该请求就会等待，直到池中有进程结束，才会创建新的进程来执行，请看下面的实例：
```
from multiprocessing import Pool
import os,time,random

def worker(msg):
    t_start = time.time()
    print("%s开始执行,进程号为%d"%(msg,os.getpid()))
    #random.random()随机生成0~1之间的浮点数
    time.sleep(random.random()*2) 
    t_stop = time.time()
    print(msg,"执行完毕，耗时%0.2f"%(t_stop-t_start))

po=Pool(3) #定义一个进程池，最大进程数3
for i in range(0,10):
    #Pool.apply_async(要调用的目标,(传递给目标的参数元祖,))
    #每次循环将会用空闲出来的子进程去调用目标
    po.apply_async(worker,(i,))

print("----start----")
po.close() #关闭进程池，关闭后po不再接收新的请求
po.join() #等待po中所有子进程执行完成，必须放在close语句之后
print("-----end-----")
```
运行结果:
![](http://image.ixysec.com/image/process10.jpg)

multiprocessing.Pool常用函数解析：
+ apply_async(func[, args[, kwds]]) ：使用非阻塞方式调用func（并行执行，堵塞方式必须等待上一个进程退出才能执行下一个进程），args为传递给func的参数列表，kwds为传递给func的关键字参数列表；
+ apply(func[, args[, kwds]])：使用阻塞方式调用func
+ close()：关闭Pool，使其不再接受新的任务；
+ terminate()：不管任务是否完成，立即终止；
+ join()：主进程阻塞，等待子进程的退出， 必须在close或terminate之后使用；

#### apply堵塞式
```
from multiprocessing import Pool
import os,time,random

def worker(msg):
    t_start = time.time()
    print("%s开始执行,进程号为%d"%(msg,os.getpid()))
    #random.random()随机生成0~1之间的浮点数
    time.sleep(random.random()*2) 
    t_stop = time.time()
    print(msg,"执行完毕，耗时%0.2f"%(t_stop-t_start))

po=Pool(3) #定义一个进程池，最大进程数3
for i in range(0,10):
    po.apply(worker,(i,))

print("----start----")
po.close() #关闭进程池，关闭后po不再接收新的请求
po.join() #等待po中所有子进程执行完成，必须放在close语句之后
print("-----end-----")
```

### 进程间通信-Queue
Process之间有时需要通信，操作系统提供了很多机制来实现进程间的通信。
#### Queue的使用
可以使用multiprocessing模块的Queue实现多进程之间的数据传递，Queue本身是一个消息列队程序，首先用一个小实例来演示一下Queue的工作原理：
```
from multiprocessing import Queue
q=Queue(3) #初始化一个Queue对象，最多可接收三条put消息
q.put("消息1") 
q.put("消息2")
print(q.full())  #False
q.put("消息3")
print(q.full()) #True

#因为消息列队已满下面的try都会抛出异常，第一个try会等待2秒后再抛出异常，第二个Try会立刻抛出异常
try:
    q.put("消息4",True,2)
except:
    print("消息列队已满，现有消息数量:%s"%q.qsize())

try:
    q.put_nowait("消息4")
except:
    print("消息列队已满，现有消息数量:%s"%q.qsize())

#推荐的方式，先判断消息列队是否已满，再写入
if not q.full():
    q.put_nowait("消息4")

#读取消息时，先判断消息列队是否为空，再读取
if not q.empty():
    for i in range(q.qsize()):
        print(q.get_nowait())
```
运行结果:
![](http://image.ixysec.com/image/process11.jpg)
#### 说明
初始化Queue()对象时（例如：q=Queue()），若括号中没有指定最大可接收的消息数量，或数量为负值，那么就代表可接受的消息数量没有上限（直到内存的尽头）；

+ Queue.qsize()：返回当前队列包含的消息数量；
+ Queue.empty()：如果队列为空，返回True，反之False ；
+ Queue.full()：如果队列满了，返回True,反之False；
+ Queue.get([block[, timeout]])：获取队列中的一条消息，然后将其从列队中移除，block默认值为True；
    1. 如果block使用默认值，且没有设置timeout（单位秒），消息列队如果为空，此时程序将被阻塞（停在读取状态），直到从消息列队读到消息为止，如果设置了timeout，则会等待timeout秒，若还没读取到任何消息，则抛出"Queue.Empty"异常；
    2. 如果block值为False，消息列队如果为空，则会立刻抛出"Queue.Empty"异常；
+ Queue.get_nowait()：相当Queue.get(False)；
+ Queue.put(item,[block[, timeout]])：将item消息写入队列，block默认值为True；
    1. 如果block使用默认值，且没有设置timeout（单位秒），消息列队如果已经没有空间可写入，此时程序将被阻塞（停在写入状态），直到从消息列队腾出空间为止，如果设置了timeout，则会等待timeout秒，若还没空间，则抛出"Queue.Full"异常；
    2. 如果block值为False，消息列队如果没有空间可写入，则会立刻抛出"Queue.Full"异常；
+ Queue.put_nowait(item)：相当Queue.put(item, False)；

#### Queue实例
我们以Queue为例，在父进程中创建两个子进程，一个往Queue里写数据，一个从Queue里读数据：
```
from multiprocessing import Process,Queue
import os,time,random

def write(q):
    for value in ["A","B","C"]:
        print("Put %s to queue..."%value)
        q.put(value)
        time.sleep(random.random())

def read(q):
    while True:
        if not q.empty():
            value = q.get(True)
            print("Get %s from queue..."%value)
            time.sleep(random.random())
        else:
            break
if __name__ == "__main__":
    q = Queue()
    pw = Process(target=write, args=(q,))
    pr = Process(target=read, args=(q,))

    pw.start()
    pw.join()

    pr.start()
    pr.join()

    print("")
    print("所以数据都写入并且读完")
```
运行结果：
![](http://image.ixysec.com/image/process12.jpg)

#### 进程池中的Queue
如果要使用Pool创建进程，就需要使用multiprocessing.Manager()中的Queue()，而不是multiprocessing.Queue()，否则会得到一条如下的错误信息：

RuntimeError: Queue objects should only be shared between processes through inheritance.

下面的实例演示了进程池中的进程如何通信：
```
from multiprocessing import Manager,Pool
import os,time,random

def reader(q):
    print("reader启动(%s),父进程为(%s)"%(os.getpid(),os.getppid()))
    for i in range(q.qsize()):
        print("reader从Queue获取到消息：%s"%q.get(True))

def writer(q):
    print("writer启动(%s),父进程为(%s)"%(os.getpid(),os.getppid()))
    for i in "zzb":
        q.put(i)

if __name__ == "__main__":
    print("(%s) start"%os.getpid())
    q = Manager().Queue()
    po = Pool()
    po.apply(writer,(q,))
    po.apply(reader,(q,))
    po.close()
    po.join()
    print("(%s) End"%os.getpid())

```
运行结果:
![](http://image.ixysec.com/image/process13.jpg)

---
## 线程
### 多线程-threading
python的thread模块是比较底层的模块，python的threading模块是对thread做了一些包装的，可以更加方便的被使用

#### 使用threading模块
+ 单线程执行
```
import time
def saySorry():
    print("亲爱的，我错了，我能吃饭了吗？")
    time.sleep(1)

if __name__ == "__main__":
    for i in range(5):
        saySorry()
```
运行结果：
![](http://image.ixysec.com/image/thread01.gif)

+ 多线程执行
```
import threading
import time

def saySorry():
    print("亲爱的，我错了，我能吃饭了吗？")
    time.sleep(1)

if __name__ == "__main__":
    for i in range(5):
        t = threading.Thread(target=saySorry)
        t.start() #启动线程，即让线程开始执行
```
运行结果：
![](http://image.ixysec.com/image/thread02.gif)

说明
1. 可以明显看出使用了多线程并发的操作，花费时间要短很多
2. 创建好的线程，需要调用start()方法来启动

#### 主线程会等待所有的子线程结束后才结束
```
import threading
from time import sleep,ctime

def sing():
    for i in range(3):
        print("正在唱歌...%d"%i)
        sleep(1)

def dance():
    for i in range(3):
        print("正在跳舞...%d"%i)
        sleep(1)

if __name__ == '__main__':
    print('---开始---:%s'%ctime())

    t1 = threading.Thread(target=sing)
    t2 = threading.Thread(target=dance)

    t1.start()
    t2.start()

    #sleep(5) # 屏蔽此行代码，试试看，程序是否会立马结束？
    print('---结束---:%s'%ctime())
```
运行结果：
![](http://image.ixysec.com/image/thread03.gif)

#### 查看线程数量
```
import threading
from time import sleep,ctime

def sing():
    for i in range(3):
        print("正在唱歌...%d"%i)
        sleep(1)

def dance():
    for i in range(3):
        print("正在跳舞...%d"%i)
        sleep(1)

if __name__ == '__main__':
    print('---开始---:%s'%ctime())

    t1 = threading.Thread(target=sing)
    t2 = threading.Thread(target=dance)

    t1.start()
    t2.start()

    while True:
        length = len(threading.enumerate())
        print('当前运行的线程数为：%d'%length)
        if length<=1:
            break

        sleep(0.5)
```
运行结果：
![](http://image.ixysec.com/image/thread04.gif)

### threading注意点
#### 线程执行代码的封装
通过上一小节，能够看出，通过使用threading模块能完成多任务的程序开发，为了让每个线程的封装性更完美，所以使用threading模块时，往往会定义一个新的子类class，只要继承threading.Thread就可以了，然后重写`run`方法

示例如下：
```
import threading
import time

class MyThread(threading.Thread):
    def run(self):
        for i in range(3):
            time.sleep(1)
            msg = "I'm "+self.name+' @ '+str(i) #name属性中保存的是当前线程的名字
            print(msg)

if __name__ == '__main__':
    t = MyThread()
    t.start()
```
运行结果：
![](http://image.ixysec.com/image/thread05.gif)

说明
+ python的threading.Thread类有一个run方法，用于定义线程的功能函数，可以在自己的线程类中覆盖该方法。而创建自己的线程实例后，通过Thread类的start方法，可以启动该线程，交给python虚拟机进行调度，当该线程获得执行的机会时，就会调用run方法执行线程。

#### 线程的执行顺序
```
import threading
import time

class MyThread(threading.Thread):
    def run(self):
        for i in range(3):
            time.sleep(1)
            msg = "I'm "+self.name+' @ '+str(i)
            print(msg)
def test():
    for i in range(5):
        t = MyThread()
        t.start()
if __name__ == '__main__':
    test()
```
执行结果：(运行的结果可能不一样，但是大体是一致的)
```
    I'm Thread-1 @ 0
    I'm Thread-2 @ 0
    I'm Thread-5 @ 0
    I'm Thread-3 @ 0
    I'm Thread-4 @ 0
    I'm Thread-3 @ 1
    I'm Thread-4 @ 1
    I'm Thread-5 @ 1
    I'm Thread-1 @ 1
    I'm Thread-2 @ 1
    I'm Thread-4 @ 2
    I'm Thread-5 @ 2
    I'm Thread-2 @ 2
    I'm Thread-1 @ 2
    I'm Thread-3 @ 2
```
说明
从代码和执行结果我们可以看出，多线程程序的执行顺序是不确定的。当执行到sleep语句时，线程将被阻塞（Blocked），到sleep结束后，线程进入就绪（Runnable）状态，等待调度。而线程调度将自行选择一个线程执行。上面的代码中只能保证每个线程都运行完整个run函数，但是线程的启动顺序、run函数中每次循环的执行顺序都不能确定。

#### 总结
1. 每个线程一定会有一个名字，尽管上面的例子中没有指定线程对象的name，但是python会自动为线程指定一个名字。
2. 当线程的run()方法结束时该线程完成。
3. 无法控制线程调度程序，但可以通过别的方式来影响线程调度的方式。
4. 线程的几种状态
![](http://image.ixysec.com/image/thread06.png)

### 多线程-共享全局变量
```
from threading import Thread
import time

g_num = 100

def work1():
    global g_num
    for i in range(3):
        g_num += 1
    print("----in work1, g_num is %d---"%g_num)

def work2():
    global g_num
    print("----in work2, g_num is %d---"%g_num)

print("---线程创建之前g_num is %d---"%g_num)

t1 = Thread(target=work1)
t1.start()

#延时一会，保证t1线程中的事情做完
time.sleep(1)

t2 = Thread(target=work2)
t2.start()
```
运行结果:
```
---线程创建之前g_num is 100---
----in work1, g_num is 103---
----in work2, g_num is 103---
```
#### 列表当做实参传递到线程中
```
from threading import Thread
import time

def work1(nums):
    nums.append(44)
    print("----in work1---",nums)


def work2(nums):
    #延时一会，保证t1线程中的事情做完
    time.sleep(1)
    print("----in work2---",nums)

g_nums = [11,22,33]

t1 = Thread(target=work1, args=(g_nums,))
t1.start()

t2 = Thread(target=work2, args=(g_nums,))
t2.start()
```
运行结果:
```
----in work1--- [11, 22, 33, 44]
----in work2--- [11, 22, 33, 44]
```
#### 总结：
+ 在一个进程内的所有线程共享全局变量，能够在不适用其他方式的前提下完成多线程之间的数据共享（这点要比多进程要好）
+ 缺点就是，线程是对全局变量随意遂改可能造成多线程之间对全局变量的混乱（即线程非安全）

### 进程、线程对比
#### 功能
+ 进程，能够完成多任务，比如 在一台电脑上能够同时运行多个QQ
+ 线程，能够完成多任务，比如 一个QQ中的多个聊天窗口

![](http://image.ixysec.com/image/thread07.jpg)

#### 定义的不同
+ 进程是系统进行资源分配和调度的一个独立单位.
+ 线程是进程的一个实体,是CPU调度和分派的基本单位,它是比进程更小的能独立运行的基本单位.线程自己基本上不拥有系统资源,只拥有一点在运行中必不可少的资源(如程序计数器,一组寄存器和栈),但是它可与同属一个进程的其他的线程共享进程所拥有的全部资源.

#### 区别
+ 一个程序至少有一个进程,一个进程至少有一个线程.
+ 线程的划分尺度小于进程(资源比进程少)，使得多线程程序的并发性高。
+ 进程在执行过程中拥有独立的内存单元，而多个线程共享内存，从而极大地提高了程序的运行效率
+ 线程不能够独立执行，必须依存在进程中

#### 优缺点
线程和进程在使用上各有优缺点：线程执行开销小，但不利于资源的管理和保护；而进程正相反。

### 同步
#### 多线程开发可能遇到的问题
假设两个线程t1和t2都要对num=0进行增1运算，t1和t2都各对num修改10次，num的最终的结果应该为20。
但是由于是多线程访问，有可能出现下面情况：

在num=0时，t1取得num=0。此时系统把t1调度为”sleeping”状态，把t2转换为”running”状态，t2也获得num=0。然后t2对得到的值进行加1并赋给num，使得num=1。然后系统又把t2调度为”sleeping”，把t1转为”running”。线程t1又把它之前得到的0加1后赋值给num。这样，明明t1和t2都完成了1次加1工作，但结果仍然是num=1。

```
from threading import Thread
import time

g_num = 0

def test1():
    global g_num
    for i in range(1000000):
        g_num += 1

    print("---test1---g_num=%d"%g_num)

def test2():
    global g_num
    for i in range(1000000):
        g_num += 1

    print("---test2---g_num=%d"%g_num)


p1 = Thread(target=test1)
p1.start()

# time.sleep(3) #取消屏蔽之后 再次运行程序，结果会不一样，，，为啥呢？

p2 = Thread(target=test2)
p2.start()

print("---g_num=%d---"%g_num)
```
运行结果(可能不一样，但是结果往往不是2000000)：
```
---g_num=284672---
---test1---g_num=1166544
---test2---g_num=1406832
```
取消屏蔽之后，再次运行结果如下：
```
---test1---g_num=1000000
---g_num=1041802---
---test2---g_num=2000000
```
问题产生的原因就是没有控制多个线程对同一资源的访问，对数据造成破坏，使得线程运行的结果不可预期。这种现象称为“线程不安全”。

#### 什么是同步
同步就是协同步调，按预定的先后次序进行运行。如:你说完，我再说。
"同"字从字面上容易理解为一起动作
其实不是，"同"字应是指协同、协助、互相配合。
如进程、线程同步，可理解为进程或线程A和B一块配合，A执行到一定程度时要依靠B的某个结果，于是停下来，示意B运行;B依言执行，再将结果给A;A再继续操作。

#### 解决问题的思路
对于本小节提出的那个计算错误的问题，可以通过`线程同步`来进行解决

思路，如下:
1. 系统调用t1，然后获取到num的值为0，此时上一把锁，即不允许其他现在操作num
2. 对num的值进行+1
3. 解锁，此时num的值为1，其他的线程就可以使用num了，而且是num的值不是0而是1
4. 同理其他线程在对num进行修改时，都要先上锁，处理完后再解锁，在上锁的整个过程中不允许其他线程访问，就保证了数据的正确性

### 互斥锁
当多个线程几乎同时修改某一个共享数据的时候，需要进行同步控制

线程同步能够保证多个线程安全访问竞争资源，最简单的同步机制是引入互斥锁。

互斥锁为资源引入一个状态：锁定/非锁定。

某个线程要更改共享数据时，先将其锁定，此时资源的状态为“锁定”，其他线程不能更改；直到该线程释放资源，将资源的状态变成“非锁定”，其他的线程才能再次锁定该资源。互斥锁保证了每次只有一个线程进行写入操作，从而保证了多线程情况下数据的正确性。

threading模块中定义了Lock类，可以方便的处理锁定：
```
#创建锁
mutex = threading.Lock()
#锁定
mutex.acquire([blocking])
#释放
mutex.release()
```
其中，锁定方法acquire可以有一个blocking参数。

+ 如果设定blocking为True，则当前线程会堵塞，直到获取到这个锁为止（如果没有指定，那么默认为True）
+ 如果设定blocking为False，则当前线程不会堵塞

使用互斥锁实现上面的例子的代码如下：
```
from threading import Thread, Lock
import time

g_num = 0

def test1():
    global g_num
    for i in range(1000000):
        #True表示堵塞 即如果这个锁在上锁之前已经被上锁了，那么这个线程会在这里一直等待到解锁为止 
        #False表示非堵塞，即不管本次调用能够成功上锁，都不会卡在这,而是继续执行下面的代码
        mutexFlag = mutex.acquire(True) 
        if mutexFlag:
            g_num += 1
            mutex.release()

    print("---test1---g_num=%d"%g_num)

def test2():
    global g_num
    for i in range(1000000):
        mutexFlag = mutex.acquire(True) #True表示堵塞
        if mutexFlag:
            g_num += 1
            mutex.release()

    print("---test2---g_num=%d"%g_num)

#创建一个互斥锁
#这个所默认是未上锁的状态
mutex = Lock()

p1 = Thread(target=test1)
p1.start()


p2 = Thread(target=test2)
p2.start()

print("---g_num=%d---"%g_num)
```
运行结果：
```
---g_num=61866---
---test1---g_num=1861180
---test2---g_num=2000000
```
可以看到，加入互斥锁后，运行结果与预期相符。

#### 上锁解锁过程
当一个线程调用锁的acquire()方法获得锁时，锁就进入“locked”状态。
每次只有一个线程可以获得锁。如果此时另一个线程试图获得这个锁，该线程就会变为“blocked”状态，称为“阻塞”，直到拥有锁的线程调用锁的release()方法释放锁之后，锁进入“unlocked”状态。
线程调度程序从处于同步阻塞状态的线程中选择一个来获得锁，并使得该线程进入运行（running）状态。

#### 总结
锁的好处：

+ 确保了某段关键代码只能由一个线程从头到尾完整地执行

锁的坏处：

+ 阻止了多线程并发执行，包含锁的某段代码实际上只能以单线程模式执行，效率就大大地下降了
+ 由于可以存在多个锁，不同的线程持有不同的锁，并试图获取对方持有的锁时，可能会造成死锁

### 多线程-非共享数据
对于全局变量，在多线程中要格外小心，否则容易造成数据错乱的情况发生
#### 非全局变量是否要加锁呢？
```
import threading
import time

class MyThread(threading.Thread):
    # 重写 构造方法
    def __init__(self,num,sleepTime):
        threading.Thread.__init__(self)
        self.num = num
        self.sleepTime = sleepTime

    def run(self):
        self.num += 1
        time.sleep(self.sleepTime)
        print('线程(%s),num=%d'%(self.name, self.num))

if __name__ == '__main__':
    mutex = threading.Lock()
    t1 = MyThread(100,5)
    t1.start()
    t2 = MyThread(200,1)
    t2.start()
```
运行结果:
![](http://image.ixysec.com/image/thread08.gif)
#### 小总结
+ 在多线程开发中，全局变量是多个线程都共享的数据，而局部变量等是各自线程的，是非共享的

### 死锁
在线程间共享多个资源的时候，如果两个线程分别占有一部分资源并且同时等待对方的资源，就会造成死锁。
即锁1解锁的条件是锁2要解锁,但是锁2解锁的条件又是锁1要解锁
```
import threading
import time

class MyThread1(threading.Thread):
    def run(self):
        if mutexA.acquire():
            print(self.name+'----do1---up----')
            time.sleep(1)

            if mutexB.acquire():
                print(self.name+'----do1---down----')
                mutexB.release()
            mutexA.release()

class MyThread2(threading.Thread):
    def run(self):
        if mutexB.acquire():
            print(self.name+'----do2---up----')
            time.sleep(1)
            if mutexA.acquire():
                print(self.name+'----do2---down----')
                mutexA.release()
            mutexB.release()

mutexA = threading.Lock()
mutexB = threading.Lock()

if __name__ == '__main__':
    t1 = MyThread1()
    t2 = MyThread2()
    t1.start()
    t2.start()
```
运行结果： 
![](http://image.ixysec.com/image/thread09.gif)
此时已经进入到了死锁状态，可以使用ctrl-z退出
#### 说明
![](http://image.ixysec.com/image/thread10.png)
#### 避免死锁
+ 程序设计时要尽量避免（银行家算法）百度一下
+ 添加超时时间等
```
acquire(timeout=2)延时2秒，如果2秒还没有解锁，就不继续等待
```

### 同步应用
多个线程有序执行
```
from threading import Thread,Lock
from time import sleep

class Task1(Thread):
    def run(self):
        while True:
            if lock1.acquire():
                print("------Task 1 -----")
                sleep(0.5)
                lock2.release()

class Task2(Thread):
    def run(self):
        while True:
            if lock2.acquire():
                print("------Task 2 -----")
                sleep(0.5)
                lock3.release()

class Task3(Thread):
    def run(self):
        while True:
            if lock3.acquire():
                print("------Task 3 -----")
                sleep(0.5)
                lock1.release()

#使用Lock创建出的锁默认没有“锁上”
lock1 = Lock()
#创建另外一把锁，并且“锁上”
lock2 = Lock()
lock2.acquire()
#创建另外一把锁，并且“锁上”
lock3 = Lock()
lock3.acquire()

t1 = Task1()
t2 = Task2()
t3 = Task3()

t1.start()
t2.start()
t3.start()
```
运行结果:
```
------Task 1 -----
------Task 2 -----
------Task 3 -----
------Task 1 -----
------Task 2 -----
------Task 3 -----
------Task 1 -----
------Task 2 -----
------Task 3 -----
------Task 1 -----
------Task 2 -----
------Task 3 -----
------Task 1 -----
------Task 2 -----
------Task 3 -----
...省略...
```
#### 总结
+ 可以使用互斥锁完成多个任务，有序的进程工作，这就是线程的同步

### 生产者与消费者模式
Python的Queue模块中提供了同步的、线程安全的队列类，包括FIFO（先入先出)队列Queue，LIFO（后入先出）队列LifoQueue，和优先级队列PriorityQueue。这些队列都实现了锁原语（可以理解为原子操作，即要么不做，要么就做完），能够在多线程中直接使用。可以使用队列来实现线程间的同步。

用FIFO队列实现上述生产者与消费者问题的代码如下：
```
import threading
import time

#python2中
from Queue import Queue

#python3中
# from queue import Queue

class Producer(threading.Thread):
    def run(self):
        global queue
        count = 0
        while True:
            if queue.qsize() < 1000:
                for i in range(100):
                    count = count +1
                    msg = '生成产品'+str(count)
                    queue.put(msg)
                    print(msg)
            time.sleep(0.5)

class Consumer(threading.Thread):
    def run(self):
        global queue
        while True:
            if queue.qsize() > 100:
                for i in range(3):
                    msg = self.name + '消费了 '+queue.get()
                    print(msg)
            time.sleep(1)


if __name__ == '__main__':
    queue = Queue()

    for i in range(500):
        queue.put('初始产品'+str(i))
    for i in range(2):
        p = Producer()
        p.start()
    for i in range(5):
        c = Consumer()
        c.start()
```
#### Queue的说明
1. 对于Queue，在多线程通信之间扮演重要的角色
2. 添加数据到队列中，使用put()方法
3. 从队列中取数据，使用get()方法
4. 判断队列中是否还有数据，使用qsize()方法

#### 生产者消费者模式的说明
+ 为什么要使用生产者和消费者模式
在线程世界里，生产者就是生产数据的线程，消费者就是消费数据的线程。在多线程开发当中，如果生产者处理速度很快，而消费者处理速度很慢，那么生产者就必须等待消费者处理完，才能继续生产数据。同样的道理，如果消费者的处理能力大于生产者，那么消费者就必须等待生产者。为了解决这个问题于是引入了生产者和消费者模式。
+ 什么是生产者消费者模式
生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。

这个阻塞队列就是用来给生产者和消费者解耦的。纵观大多数设计模式，都会找一个第三者出来进行解耦.

### ThreadLocal
在多线程环境下，每个线程都有自己的数据。一个线程使用自己的局部变量比使用全局变量好，因为局部变量只有线程自己能看见，不会影响其他线程，而全局变量的修改必须加锁。

#### 使用函数传参的方法
但是局部变量也有问题，就是在函数调用的时候，传递起来很麻烦：
```
def process_student(name):
    std = Student(name)
    # std是局部变量，但是每个函数都要用它，因此必须传进去：
    do_task_1(std)
    do_task_2(std)

def do_task_1(std):
    do_subtask_1(std)
    do_subtask_2(std)

def do_task_2(std):
    do_subtask_2(std)
    do_subtask_2(std)
```
每个函数一层一层调用都这么传参数那还得了？用全局变量？也不行，因为每个线程处理不同的Student对象，不能共享。
#### 使用全局字典的方法
如果用一个全局dict存放所有的Student对象，然后以thread自身作为key获得线程对应的Student对象如何？
```
global_dict = {}

def std_thread(name):
    std = Student(name)
    # 把std放到全局变量global_dict中：
    global_dict[threading.current_thread()] = std
    do_task_1()
    do_task_2()

def do_task_1():
    # 不传入std，而是根据当前线程查找：
    std = global_dict[threading.current_thread()]
    ...

def do_task_2():
    # 任何函数都可以查找出当前线程的std变量：
    std = global_dict[threading.current_thread()]
    ...
```
这种方式理论上是可行的，它最大的优点是消除了std对象在每层函数中的传递问题，但是，每个函数获取std的代码有点low。

有没有更简单的方式？
#### 使用ThreadLocal的方法
ThreadLocal应运而生，不用查找dict，ThreadLocal帮你自动做这件事：
```
import threading

# 创建全局ThreadLocal对象:
local_school = threading.local()

def process_student():
    # 获取当前线程关联的student:
    std = local_school.student
    print('Hello, %s (in %s)' % (std, threading.current_thread().name))

def process_thread(name):
    # 绑定ThreadLocal的student:
    local_school.student = name
    process_student()

t1 = threading.Thread(target= process_thread, args=('zzb',), name='Thread-A')
t2 = threading.Thread(target= process_thread, args=('老王',), name='Thread-B')
t1.start()
t2.start()
t1.join()
t2.join()
```
执行结果：
```
Hello, zzb (in Thread-A)
Hello, 老王 (in Thread-B)
```

说明
全局变量local_school就是一个ThreadLocal对象，每个Thread对它都可以读写student属性，但互不影响。你可以把local_school看成全局变量，但每个属性如local_school.student都是线程的局部变量，可以任意读写而互不干扰，也不用管理锁的问题，ThreadLocal内部会处理。

可以理解为全局变量local_school是一个dict，不但可以用local_school.student，还可以绑定其他变量，如local_school.teacher等等。

ThreadLocal最常用的地方就是为每个线程绑定一个数据库连接，HTTP请求，用户身份信息等，这样一个线程的所有调用到的处理函数都可以非常方便地访问这些资源。

#### 小结
一个ThreadLocal变量虽然是全局变量，但每个线程都只能读写自己线程的独立副本，互不干扰。ThreadLocal解决了参数在一个线程中各个函数之间互相传递的问题

### GIL 全局解释器锁
实际上python的多线程是个假的。
因为GIL的影响，同时只有一个线程在执行
解决：
关键部分用c语言编写。
