### java多线程概况

------------

#### 什么是多线程？

-----------

多线程意味着您在同一应用程序中具有多个*执行线程*。线程就像执行应用程序的独立CPU。因此，多线程应用程序就像具有多个CPU同时执行代码的不同部分的应用程序。

<img src="https://translate.google.com/website?sl=auto&tl=zh-CN&u=http://tutorials.jenkov.com/images/java-concurrency/introduction-1.png" alt="在其中执行两个线程的应用程序。" style="zoom:80%;" />

线程不等于CPU。通常，单个CPU将在多个线程之间共享其执行时间，并在给定的时间间隔内执行每个线程。也可以使应用程序的线程由不同的CPU执行。

![具有线程由不同线程执行的多个应用程序。](https://translate.google.com/website?sl=auto&tl=zh-CN&u=http://tutorials.jenkov.com/images/java-concurrency/introduction-2.png)



#### 为什么要多线程？

--------

为什么要在应用程序中使用多线程有几个原因。多线程的一些最常见原因是：

- 提高单个CPU的利用率。
- 更好地利用多个CPU或CPU内核。
- 关于响应性的更好的用户体验。
- 关于公平的更好的用户体验。

我将在以下各节中详细解释每个原因。 

> 更好地利用单个CPU

最常见的原因之一是能够更好地利用计算机中的资源。例如，如果一个线程正在**等待**对通过网络发送的请求的响应，则另一线程可以同时使用CPU来执行其他操作。此外，如果计算机具有多个CPU，或者该CPU具有多个执行核心，则多线程还可以帮助您的应用程序利用这些额外的CPU核心。

> 更好地利用多个CPU或CPU内核

如果计算机包含多个CPU或CPU包含多个执行核心，则您需要为应用程序使用多个线程才能使用所有CPU或CPU核心。单个线程最多只能使用一个CPU，如上所述，有时甚至不能完全利用单个CPU。

