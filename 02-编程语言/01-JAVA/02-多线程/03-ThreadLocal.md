---
dg-publish: false
---
ThreadLocal出现OOM内存溢出的场景和原理分析:https://blog.csdn.net/GoGleTech/article/details/78318712

## ThreadLocal使用场景
比如像用户登录令牌解密后的信息传递、用户权限信息、从用户系统中获取到的用户名
## ThreadLocal慎用的场景

第一点（线程池里线程调用ThreadLocal）：因为线程池里对线程的管理都是线程复用的方法，所以在线程池里线程非常难结束，更有可能的是永远不会结束。这就意味着线程的持续时间是不可估测的，甚至会与JVM的生命周期一致。

第二点（在异步程序里）：ThreadLocal的参数传递是不可靠的，因为线程将请求发送后，不会在等待远程返回结果就继续向下运行了，真正的返回结果得到以后，可能是其它的线程在处理。

第三点：在使用完ThreadLocal，推荐要调用一下remove（）方法，这样会防止内存溢出这种情况的发生，因为ThreadLocal为弱引用。如果ThreadLocal在没有被外部强引用的情况下，在垃圾回收的时候是会被清理掉的，如果是强引用那就不会被清理。
