## linux系统打开的文件文件可以被另一个进程删除吗 
linux访问一个文件： <br>
从根目录/的inode不断做3个操作 ==》1、读取目录文件 2、找到对应的inode 3、从对应块读取内容  <br>
如果3读取到的是一个目录文件，继续递归下去，知道读取到目标文件为止  <br>

每个文件都会有2个link计数器-- i_count 和 i_nlink <br>
i_count的意义是当前使用者的数量，也就是打开文件进程的个数；i_nlink的意义是介质连接的数量 <br>
可以理解为 i_count是内存引用计数器，i_nlink是硬盘引用计数器,当文件被某个进程引用时，i_count 就会增加；当创建文件的硬连接的时候，i_nlink  就会增加 <br>
i_nlink 是文件删除的充分条件，而 i_count 才是文件删除的必要条件 <br>

rm 操作只是将 i_nlink 置为 0 了 <br>

<https://www.jianshu.com/p/dde6a01c4094>
<https://www.jianshu.com/p/fda6526aad1b>
<https://jaycechant.info/2020/correct-way-to-delete-an-opened-file/>


## 什么是僵尸进程
正常情况下，子进程是通过父进程创建的，子进程在创建新的进程。 <br>
子进程的结束和父进程的运行是一个异步过程,即父进程永远无法预测子进程到底什么时候结束。 当一个 进程完成它的工作终止之后，它的父进程需要调用wait()或者waitpid()系统调用取得子进程的终止状态  <br>

#### 孤儿进程
一个父进程退出，而它的一个或多个子进程还在运行，那么那些子进程将成为孤儿进程。孤儿进程将被init进程(进程号为1)所收养，并由init进程对它们完成状态收集工作

#### 僵尸进程
一个进程使用fork创建子进程，如果子进程退出，而父进程并没有调用wait或waitpid获取子进程的状态信息，那么子进程的进程描述符仍然保存在系统中

问题及危害
unix提供了一种机制可以保证只要父进程想知道子进程结束时的状态信息： <br>
在每个进程退出的时候,内核释放该进程所有的资源,包括打开的文件,占用的内存等。 但是仍然为其保留一定的信息(包括进程号the process ID,退出状态the termination status of the process,运行时间the amount of CPU time taken by the process等)。直到父进程通过wait / waitpid来取时才释放<br>

如果进程不调用wait / waitpid的话， 那么保留的那段信息就不会释放，其进程号就会一直被占用，但是系统所能使用的进程号是有限的，如果大量的产生僵死进程，将因为没有可用的进程号而导致系统不能产生新的进程. 此即为僵尸进程的危害，应当避免  <br>

<https://www.cnblogs.com/anker/p/3271773.html>

## linux常见命令
## Linux查看系统负载，你能想到哪些命令？如果要查看网络IO呢？
## Linux根据进程查看所占用的端口，使用什么命令
## 端口占用后，如何处理？写一下命令
## 某个文件夹每个小时会生成很多小文件，你如何将小文件全部删除？（rsync）
## 如何统计nginx日志文件中的每个状态码出现的次数和对应的URL
## linux的fork了解吗？什么是写时复制机制？
## 软连接和硬链接了解吗？它们有什么区别？底层如何实现的？
## du和df的区别


## 对docker的理解
## docker的网络模型，你了解哪几种
## docker中数据持久化的方式由哪些？
## Docker Cgroups可以限制哪些资源？cupset和cpushare有啥区别？cpu用超了会怎么样？