
在.NET中，**CancellationTokenSource**、**CancellationToken**和**Task**是处理异步操作和取消任务的重要工具。本文将通过一些简单的例子，帮助你理解它们的用法和协作方式。




---


#### CancellationTokenSource


**CancellationTokenSource** 是一个取消操作的触发器。它用于生成和管理**CancellationToken**，并控制取消信号的发出。


##### 常用属性和方法


* **Token**: 返回一个与此源关联的`CancellationToken`。
* **Cancel()**: 触发取消操作。
* **CancelAfter(milliseconds)**: 指定时间后触发取消操作。
* **Dispose()**: 释放资源。


##### 示例



```
var cts = new CancellationTokenSource();
CancellationToken token = cts.Token;

Task.Run(() => {
    for (int i = 0; i < 10; i++)
    {
        if (token.IsCancellationRequested)
        {
            Console.WriteLine("Task canceled");
            break;
        }
        Console.WriteLine($"Task running: {i}");
        Thread.Sleep(500);
    }
});

Thread.Sleep(2000);
cts.Cancel();

```

#### CancellationToken


**CancellationToken** 是用于传播取消请求的轻量级结构。它由`CancellationTokenSource`生成。


##### 常用属性和方法


* **IsCancellationRequested**: 是否收到取消请求。
* **ThrowIfCancellationRequested()**: 如果已请求取消，抛出`OperationCanceledException`。
* **Register(Action)**: 注册一个取消时触发的回调。


##### 示例



```
var cts = new CancellationTokenSource();
CancellationToken token = cts.Token;

Task.Run(() => {
    token.Register(() => Console.WriteLine("Cancellation registered"));

    try
    {
        for (int i = 0; i < 10; i++)
        {
            token.ThrowIfCancellationRequested();
            Console.WriteLine($"Task running: {i}");
            Thread.Sleep(500);
        }
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("Task was canceled");
    }
});

Thread.Sleep(2000);
cts.Cancel();

```

#### Task与CancellationToken


**Task** 是.NET中的异步操作单元。结合`CancellationToken`可以在任务运行时取消它。


##### 示例：取消任务



```
var cts = new CancellationTokenSource();
CancellationToken token = cts.Token;

Task task = Task.Run(() => {
    for (int i = 0; i < 10; i++)
    {
        if (token.IsCancellationRequested)
        {
            Console.WriteLine("Task canceled");
            break;
        }
        Console.WriteLine($"Task running: {i}");
        Thread.Sleep(500);
    }
}, token);

Thread.Sleep(2000);
cts.Cancel();

try
{
    task.Wait();
}
catch (AggregateException ex)
{
    foreach (var inner in ex.InnerExceptions)
    {
        if (inner is TaskCanceledException)
        {
            Console.WriteLine("Task cancellation exception caught");
        }
    }
}

```

##### 示例：带超时的任务



```
var cts = new CancellationTokenSource(3000); // 3秒后自动取消
CancellationToken token = cts.Token;

Task.Run(() => {
    try
    {
        for (int i = 0; i < 10; i++)
        {
            token.ThrowIfCancellationRequested();
            Console.WriteLine($"Task running: {i}");
            Thread.Sleep(1000);
        }
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("Task canceled due to timeout");
    }
});

```

#### 小结


1. 使用`CancellationTokenSource`来控制取消。
2. 通过`CancellationToken`将取消信号传递给任务或方法。
3. 任务中可以通过`ThrowIfCancellationRequested`或检查`IsCancellationRequested`响应取消请求。
4. 合理使用`Register`可以处理取消时的回调逻辑。


通过灵活运用这些工具，你可以编写更高效、可控的异步程序。


![](https://images.cnblogs.com/cnblogs_com/chenyishi/1348350/o_240408130234_wx.png) 本博客参考[wgetcloud全球加速服务机场](https://wa7.org)。转载请注明出处！
