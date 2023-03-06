https://github.com/h2pl/Java-Tutorial#%E5%B9%B6%E5%8F%91

## synchronized可重入锁的实现

之前谈到过，每个锁关联一个线程持有者和一个计数器。当计数器为0时表示该锁没有被任何线程持有，那么任何线程都都可能获得该锁而调用相应方法。当一个线程请求成功后，JVM会记下持有锁的线程，并将计数器计为1。此时其他线程请求该锁，则必须等待。而该持有锁的线程如果再次请求这个锁，就可以再次拿到这个锁，同时计数器会递增。当线程退出一个synchronized方法/块时，计数器会递减，如果计数器为0则释放该锁。

## synchronized和juc包的区别
Lock是显式加锁，锁释放。而synchronized是隐式锁
Lock只有代码块锁，synchronized有代码块锁和方法锁

https://blog.csdn.net/dl962454/article/details/112909140