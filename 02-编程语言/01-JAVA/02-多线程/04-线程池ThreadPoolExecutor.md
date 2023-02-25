## submit和execute区别
In Java, a thread pool is a collection of pre-initialized threads that are used to execute tasks. The `ThreadPoolExecutor` class in Java provides a way to create and manage thread pools.

Both the `execute` and `submit` methods are used to submit a task to a thread pool for execution. However, there are some differences between them:

1.  Return Value: The `execute` method has a void return type, while the `submit` method returns a `Future` object. This `Future` object can be used to check the status of the task, get the result of the task if it returns a value, and cancel the task.
    
2.  Exception Handling: The `execute` method does not throw any checked exceptions, while the `submit` method throws a `RejectedExecutionException` if the task cannot be accepted for execution.
    
3.  Task Parameters: The `execute` method takes a `Runnable` object as a parameter, while the `submit` method takes a `Callable` or a `Runnable` object as a parameter. The `Callable` interface is similar to the `Runnable` interface, but it allows the task to return a value.
    
4.  Execution: The `execute` method adds the task to the thread pool's queue and returns immediately, while the `submit` method submits the task to the thread pool and returns a `Future` object immediately. The task may not be executed immediately if all the threads in the thread pool are busy.
    

In summary, the `execute` method is used to submit a task for execution without any expectation of a result, while the `submit` method is used to submit a task for execution and obtain a `Future` object that can be used to check the status and get the result of the task.

**带返回值的 是阻塞的

