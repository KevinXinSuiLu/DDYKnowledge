# Swift6-多线程

> ## 相关概念

* 进程      
指在系统中正在运行的一个应用程序，进程拥有独立运行所需的全部资源（例如：正在运行的QQ就是一个进程）。

* 线程     
指程序中独立运行的代码段（例如：接收QQ消息的代码），一个进程是由一或多个线程组成。

* 多线程     
1个进程中可以开启多条线程，每条线程可以并行（同时）执行不同的任务，进程只负责资源的调度和分配，线程才是程序真正的执行单元，负责代码的执行。

* 单线程与多线程对比
 - 单线程程序：只有一个线程，代码顺序执行，容易出现代码阻塞（页面假死）。
 - 多线程程序：有多个线程，线程间独立运行，能有效的避免代码阻塞，并且提高程序的运行性能。

* 线程相关
 - 同步线程：同步线程会阻塞当前线程去执行线程内的任务，执行完之后才会反回当前线程。
 - 异步线程：异步线程不会阻塞当前线程，会开启其他线程去执行线程内的任务。
 - 串行队列：线程任务按先后顺序逐个执行（需要等待队列里面前面的任务执行完之后再执行新的任务）。
 - 并发队列：多个任务按添加顺序一起开始执行（不用等待前面的任务执行完再执行新的任务），但是添加间隔往往忽略不计，所以看着像是一起执行的。
 - 并发VS并行：并行是基于多核设备的，并行一定是并发，并发不一定是并行。

* 死锁     
死锁是指两个或两个以上的进程在执行过程中，由于竞争资源或者由于彼此通信而造成的一种阻塞的现象，若无外力作用，它们都将无法推进下去

例如主线程串行队列同步执行任务引起死锁     

```
DispatchQueue.main.sync {
	print("死锁了不执行")
}
```

自定义串行队列同步任务 嵌套 该自定义串行队列同步任务，产生死锁

```
let serialQueue = DispatchQueue(label: "com.ddy.serialQueue")
serialQueue.sync {
	print("执行了1")
	serialQueue.sync {
		print("死锁了不执行")
	}
}

```

自定义串行队列异步任务 嵌套 该自定义串行队列同步任务， 产生死锁

```
let serialQueue = DispatchQueue(label: "com.ddy.serialQueue")
serialQueue.async {
	print("执行了1")
	serialQueue.sync {
		print("死锁了不执行")
	}
}
print("执行了3")
```

总结：遇到串行同步要小心

* 死锁的四个必要条件

 - 互斥条件：一个资源每次只能被一个进程使用。
 - 请求与保持条件：一个进程因请求资源而阻塞时，对已获得的资源保持不放。
 - 不剥夺条件:进程已获得的资源，在末使用完之前，不能强行剥夺。
 - 循环等待条件:若干进程之间形成一种头尾相接的循环等待资源关系。

* 避免死锁的方法    
 1. 破坏“互斥”条件:就是在系统里取消互斥。若资源不被一个进程独占使用，那么死锁是肯定不会发生的。但一般“互斥”条件是无法破坏的。因此，在死锁预防里主要是破坏其他三个必要条件，而不去涉及破坏“互斥”条件。
 2. 破坏“请求和保持”条件:在系统中不允许进程在已获得某种资源的情况下，申请其他资源。即要想出一个办法，阻止进程在持有资源的同时申请其他资源。
方法：要求每个进程提出新的资源申请前，释放它所占有的资源。这样，一个进程在需要资源S时，须先把它先前占有的资源R释放掉，然后才能提出对S的申请，即使它可能很快又要用到资源R。
 3. 破坏“不可抢占”条件：允许对资源实行抢夺。
方法一：如果占有某些资源的一个进程进行进一步资源请求被拒绝，则该进程必须释放它最初占有的资源，如果有必要，可再次请求这些资源和另外的资源。
方法二：如果一个进程请求当前被另一个进程占有的一个资源，则操作系统可以抢占另一个进程，要求它释放资源。只有在任意两个进程的优先级都不相同的条件下，该方法才能预防死锁。
 4. 破坏“循环等待”条件：将系统中的所有资源统一编号，进程可在任何时刻提出资源申请，但所有申请必须按照资源的编号顺序（升序）提出。这样做就能保证系统不出现死锁。
方法：利用银行家算法避免死锁。

[死锁参考](https://blog.csdn.net/yanxiaolx/article/details/51944048)

* 线程安全    

	一段线程安全的代码（对象），可以同时被多个线程或并发的任务调度，不会产生问题，非线程安全的只能按次序被访问。所有Mutable对象都是非线程安全的，所有Immutable对象都是线程安全的，使用Mutable对象，一定要用同步锁来同步访问（@synchronized）。   
	  
	互斥锁：能够防止多线程抢夺造成的数据安全问题，但是需要消耗大量的资源    
原子属性（atomic）加锁    

	atomic: 原子属性，为setter方法加锁，将属性以atomic的形式来声明，该属性变量就能支持互斥锁了。 
	   
	nonatomic: 非原子属性，不会为setter方法加锁，声明为该属性的变量，客户端应尽量避免多线程争夺同一资源。    
	
> ##  几种多线程技术比较

* pthread   
优点: 跨平台，可移植性强   
缺点: 程序员管理生命周期，使用难度大。一般不用

* NSThread (抽象层次：低)     
优点：轻量级，简单易用，可以直接操作线程对象   
缺点: 需要自己管理线程的生命周期，线程同步。线程同步对数据的加锁会有一定的系统开销。

* Cocoa NSOperation (抽象层次：中)      
优点：不需要关心线程管理，数据同步的事情，可以把精力放在学要执行的操作上。基于GCD，是对GCD 的封装，比GCD更加面向对象    
缺点: NSOperation是个抽象类，使用它必须使用它的子类，可以实现它或者使用它定义好的两个子类NSInvocationOperation、NSBlockOperation.

* GCD 全称Grand Center Dispatch (抽象层次：高)      
优点：是 Apple 开发的一个多核编程的解决方法，简单易用，效率高，速度快，基于C语言，更底层更高效，并且不是Cocoa框架的一部分，自动管理线程生命周期（创建线程、调度任务、销毁线程）。    
缺点: 使用GCD的场景如果很复杂，就有非常大的可能遇到死锁问题。   

GCD抽象层次最高，使用也简单，因此，苹果也推荐使用GCD  

* 为什么要用 GCD 呢？
	- GCD 可用于多核的并行运算   
	- GCD 会自动利用更多的 CPU 内核（比如双核、四核）   
	- GCD 会自动管理线程的生命周期（创建线程、调度任务、销毁线程）   
	- 程序员只需要告诉 GCD 想要执行什么任务，不需要编写任何线程管理代码  

	
> ## GCD

1.主线程串行队列（main queue）：提交至Main queue的任务会在主线程中执行，Main queue 可以通过```DispatchQueue.main```来获取,主队列一定伴随主线程，但主线程不一定伴随主队列。    
2.全局并发队列（Global queue）：全局并发队列由整个进程共享，有Qos优先级别。Global queue 可以通过调用```DispatchQueue.global()```函数来获取（可以设置优先级）    
3.自定义队列(Custom queue): 可以为串行，也可以为并发。Custom queue 可以通过```DispatchQueue(label: String)和DispatchQueue(label: String, attributes: DispatchQueue.Attributes.concurrent)```来获取；   
4.队列组 (Group queue)：将多线程进行分组，最大的好处是可获知所有线程的完成情况。
Group queue 可以通过调用dispatch_group_create()来创建，通过dispatch_group_notify 可以直接监听组里所有线程完成情况。 

* 主线程串行队列(The main queue)

	1.主线程串行队列同步执行任务，在主线程运行时，会产生死锁
	
	```
	DispatchQueue.main.sync {
		print("死锁了不执行")
	}
	```
	
	2.主线程串行队列异步执行任务，在主线程运行，不会产生死锁
	
	```
	DispatchQueue.main.async {
		print("执行了")
	}
	```
	
	3.安全异步主线程主队列
	
	```
	import Foundation

	extension DispatchQueue {
		fileprivate static var currentQueueLabel: String? {
		let cString = __dispatch_queue_get_label(nil)
		return String(cString: cString)
    	}
    	// "com.apple.main-thread"
    	fileprivate static var isMainQueue: Bool {
		return currentQueueLabel == self.main.label
    	}
	}    
	
	func ddyMainAsyncSafe(_ execute: @escaping () -> Void) {
		DispatchQueue.isMainQueue ? execute() : DispatchQueue.main.async(execute: execute)
	}
	
	// 调用
	ddyMainAsyncSafe {
		print("主线程主队列刷新UI")
	}	
	```
	
	附：RxSwift中判断主线程主队列方式
	
	```
	extension DispatchQueue {
		private static var token: DispatchSpecificKey<()> = {
			let key = DispatchSpecificKey<()>()
			DispatchQueue.main.setSpecific(key: key, value: ())
			return key
		}()

		static var isMain: Bool {
			return DispatchQueue.getSpecific(key: token) != nil
		}
	}
```

注意：主线程串行队列由系统默认生成，无法调用conQueue.resume()和Queue.suspend()来控制执行继续或中断。