---
layout:     post
title:      "iOS并发(concurrency)概念浅析(转载)"
date:       2016-04-18
author:     "ShellHue -- 黄泽宇"
catalog: true
tags:
    - 转载
    - Concurrency
---

在进行iOS开发过程中，我们常会遇到网络请求、复杂计算、数据存取等比较耗时的操作，如果处理不合理，将对APP的流畅度产生较大影响。除了优化APP架构，并发（concurrency）是一个常用且较好的解决方法，但并发涉及串行、并发、并行、同步、异步、多线程、GCD、NSOperation和NSOperationQueue等诸多容易混淆的概念，为求概念清晰明了，还请茗茶静坐，听我徐徐道来。


# 一、线程和任务


线程(thread)和任务(task)是其他并发概念的基础，因此也是首要需理清的概念，以下是其要点，详细可参考Thread(computing)和Task(computing).


## 任务(task)


1）任务(task)是从程序中划分出来，可以独立执行的代码片段；
2) 任务间可以添加依赖关系，如B任务依赖A任务，`taskB.addDependency(taskA)`,这就意味着B任务的执行以A任务完成为前提。

需要注意的是一个任务是否可以添加依赖，完全取决于任务封装类和其相关管理的具体实现，GCD不支持任务依赖，`NSOperationQueue`就支持任务依赖。

下面是对一个任务的简单封装，并支持任务间的依赖。

```Swift
class Task {
  let taskBlock: () -> ()
  var denpendencies = [Task]()

  init(block: () -> ()) {
    taskBlock = block
  }

  func addDependency(task: Task) {
    denpendencies.append(task)
  }
}

// Init two custom tasks
var taskA = Task() {
  // do something
}

var taskB = Task() {
  // do something
}

// Add denpendency
taskB.addDependency(taskA)
```

## 线程(thread)


1) 线程是代码执行的独立路径，一条线程只能同时执行一行代码。

2）线程中代码管理是以任务为单位，一条线程朱行执行一个任务中的代码（任务可取消），完成后再逐行执行下一个任务中的代码。

3）一条线程跳出一个任务的执行，即意味着这个任务的完成。因此，一条线程不能执行taskA一段时间后，还未完成就开始执行taskB，然后又返回执行taskA


# 二、概念释疑


## 并行(parallelism)和并发(concurrency)


并行和并发都是指多个任务可以同时执行，都属于多线程编程概念，因此二者必然十分相近，容易混淆。二者区别只有一点，即是否多任务执行于严格的同一时刻。并发不是，并行是。

单核处理器时代（一个处理器同一时刻只能执行一条命令），为了实现多任务的同时执行，系统利用时间分片(time-slicing)技术，将处理器的执行时间切分为多个小片段，一会执行threadA，一会执行threadB，一会在执行threadA，即在多个线程（任务是在线程上执行的）之间来回跳动执行。虽不是真的多线程多任务同时执行，但由于处理器的处理速度非常快，在用户看来，仍然是同时执行的。这种伪多线程就是并发。

多喝处理器时代（不同处理器相互独立，可以同时执行各自的命令），多条线程完全可以严格同一时刻执行，这种真多线程就是并行。

```
// 三个线程的并发
thread1 -> |---A---|              ->|---A---|
                    \             /          \
thread2 ------------>|-----B-----|            \
thread3 --------------------------------------->|--------C--------|
```

上述代码是三个线程的并发执行，可以看出thread1,thread2和thread3不可能严格同一时刻执行，但也都获得了处理器的一小段执行时间。


```
// 三个线程的并行
thread1 -> |---A---|
thread2 ->      |-----B-----|        
thread3 ->    |--------C--------|
```

上述代码是三个线程的并行执行，可以看出thread1，thread2和thread3有一段时间同时执行。

现在的终端设备无论是手机还是PC处理器，大多已都是多核处理器，可以实现并行计算，但为了最大化的利用处理器的性能，现代处理器还是融合了time-slicling技术和多核技术，因此实际运行中，有时并发，有时并行。但相对来说，并发是个更广泛的概念。因此Apple的多线程编程叫做concurrency programming并发编程。汉语中，并发和并行的区别其实没那么清晰，可以互用，而且有时用并行更加明确，如串并行比串行、并发针对性更强。


## 串并行于线程


### 串行（serial）和并行


串行和并行主要区别在于一个任务的执行是否以上一个任务的完成为前提。串行中，一个任务的执行必须一个以上任务执行结束为前提，并行中，一个任务的执行与上一个任务的执行状态无关。以排队买票为例，串行像单个买票队伍，单个卖票窗口，必须一个一个来，并行像单个买票队伍，多个卖票窗口，多个人可以同时买票。


```
// 三个串行任务
|-----A-----||----B----||---C---|
```

上文为三个串行任务，taskA完成后才执行taskB，最后执行taskBlock


```
// 三个并发任务
|-----A-----|
  |---B---|
 |-----C-----|
```

上文为三个并行任务，taskA早于taskC开始，却晚于taskC结束


### 串并行与线程


串并行主要关注多个人物之间的相互依赖关系，与线程无关。但实际中，任务是在线程中执行的，是否串行一定在单线程上执行，并行一定在多个线程中执行呢？并非如此。

单线程既可以实现串行，也可以实现并行。

```
// 单线程串行
thread -> |-----A-----||----B----||---C---|

// 单线程并行（理论上，实际中不可行）
          A-Start ----------------------------------- A-End
            |  B-Start -----------------------------------| ---- B-End
            |   |     C-Start ------------------ C-End    |       |
            V   V       V                         V       V       V
thread ->   |-A-|---B---|-C-|-A-|-C-|--A--|-B-|-C-|---A---|---B---|
```

需要指出的是单线程内的并行已经类似单核处理器，并不是本文提及的常规线程，现实中也不常见。

多线程既可以实现串行，也可以实现并行。实际上，多线程串行和并行都很常见。

```
//多线程串行
thread1 -> |----A-----|   
                       \  
thread2 --------------->|-----B-----------|   
                                           \   
thread3 ----------------------------------->|-------C------|
//多线程并发
 thread1 ->     |----A-----|
 thread2 ----->     |-----B-----------|
 thread3 --------->     |-------C----------|
```


## 同步（synchronize）、异步（asynchronous）与线程


同步与异步是站在当前线程的角度，考察添加任务到新线程后，何时返回到当前线程执行下面的代码问题，也就是新添加的线程阻不阻塞当前的线程。


### 同步

```Swift
// viewDidLoad()在主线程中执行，因此当前线程为主线程
override viewDidLoad() {
  super.viewDidLoad()
  let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRRIPRITY_DEFAULT, 0)
  dispatch_sync(queue) {
    //block1
    print("-----1-----") //1
    return
  }
  print("-----2-----") //2
}
```

block1是添加到系统全局队列中的新任务，由于是同步的，因此block1执行返回后，才会回到主线程，执行//2以及之后的代码。执行结果

```
-----1-----
-----2-----
```


### 异步

```Swift
// viewDidLoad()在主线程中执行，因此当前线程为主线程
override viewDidLoad() {
  super.viewDidLoad()
  let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRRIPRITY_DEFAULT, 0)
  dispatch_async(queue) {
    //block1
    print("-----1-----") //1
    return
  }
  print("-----2-----") //2
}
```

block1是添加到系统全局队列的新任务，由于是异步的，因此block1添加到全局队列后（会在另外一个线程上执行），不等到执行完成，就会返回到当前主线程，执行//2以及之后的代码。所以输出结果可能为21 12。但由于block1和主线程中的任务都是不耗时的简单任务，而创建新的线程是要消耗一定时间的，因此输出的结果可能是：


```
-----2-----
-----1-----
```


### 同异步结合的情形


如果同异步结合：

```Swift
override viewDidLoad() {
	super.viewDidLoad()
	let queue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0)
	dispatch_async(queue) {
		//block1
		print("-----A-----") //1
		dispatch_async(dispatch_get_main_queue()) {
			//block2
			print("-----B-----") //2
		}
		print("-----C-----") //3
		return
	}
	print("-----D-----") //4
	while(true) { } //5
	print("-----E-----") //6
}
```

block1是添加到系统全局队列中的新任务，由于是异步的，因此block1添加全局队列后（会在另外一个线程上执行），不等到执行完成，就返回到当前主线程，执行//4及以后的代码，结果是block1所在的线程与主线程同时执行，因此理论上，D和A谁先输出不一定。但由于block1和主线程中的任务都是不耗时的简单任务，而创建新的线程是要消耗一定时间的（主线程一直存在，不用新创建），因此一般输出结果为DA。

block1所在线程输出完A后，将block2添加到主调度队列中，由于是异步的，因此block2添加主调度队列后（会在主线程上执行），不等到执行完成，就返回到block2所在的线程，继续执行，因此A和C一定会输出，且C一定在A之后输出。但block2却不一定能执行，因为block1在执行时，主线程也在执行（主线程是串行单线程，任务按顺序一个一个执行），如果此时主线程执行到//5对应的死循环，则block2一定不能被执行，B一定不能被输出，如果此时主线程尚未执行到//5对应的死循环，block2已经添加到主线程中，则block2会被执行，B能被输出。但由于主线程无需另外创建，block1（所对应的线程需另外创建）执行到添加block2到主调度队列时，主线程很可能已经执行到//5对应的死循环，因此block2很可能不被执行。
//6前有个死循环，因此E一定不会被输出。

因此可能的输出结果是；DAC ADC ADCB DACB ACDB ACBD ABDC ABCD

但很可能的输出结果为：

```
-----D-----
-----A-----
-----C-----
```


### 同异步与串并行

串行和同步，并行与异步似是完全不同的概念，一个关注任务的独立关系，一个看中的是返回的时机。但事实上，串行与同步近似，并行与异步相同。他们指代的事情几乎完全相同。

就同步和串行而言，需要任务执行结束时才能返回，其实就是一个任务执行完成后，才能执行其他的任务，反映的就是串行依赖关系。

而异步和并行就更相同了，不等任务执行完成，就直接返回，反映的是并发任务之间的独立性。

当然，同异步所暗含的串行和并行是当前线程的任务与新线程的任务之间的相互关系。


# GCD和NSOperationQueue

GCD(grand central dispatch)和NSOperationQueue二者均是系统级的多线程封装。在使用时，我们只需创建任务队列即可，其他的如线程创立、任务分配等，均由系统自动处理。不得不说，这让多线程编程变得更加高效，更简单，当然并不是没有坑。

需要强调的是，GCD和NSOperationQueue的使用核心是任务和任务队列，暂时可以忘了线程这烦人的概念。

关于GCD和NSOperationQueue网上已经有不少高质量的文章对其详细介绍，我推荐[iOS并行开发：从NSOperationQueue何调度队列开始](http://www.cocoachina.com/ios/20160201/15179.html)，其对基本概念、使用方法等的介绍非常清晰详尽，我这里就不再赘述，只写一些我认为容易忽略却影响认知深度的小知识点。当然你英语过硬，直接去看官方文档[ConcurrencyProgrammingGuide](https://developer.apple.com/library/ios/documentation/General/Conceptual/ConcurrencyProgrammingGuide/Introduction/Introduction.html)是最好的。


## GCD

GCD是基于C的API，因此比较底层。

GCD所管理的调度队列主要有三类，串行队列（private dispatch queue）、并行队列（global dispatch queue）和主队列（main dispatch queue）。

我们常用的`dispatch_get_global_queue(_:_:)`所获得的dispatch queue就是全局调度队列，并发，而且全局调度队列是全局共用的。每一个优先级的全局调度队列只有一个实体。四种不同优先级的全局调度队列对应的四种优先级的教程，同一个优先级的全局调度队列可以同时拥有多条相应优先级的线程。

`dispatch_get_main_queue()`所获得的是主调度队列，主调度队列是串行队列。


## NSOperationQueue

NSOperationQueue是对GCD的OC封装，相对于GCD具有更多先进的特性，如可以添加NSOperation依赖，取消NSOperation等。

NSOperationQueue是并发队列，且不遵循先进先出FIFO排序原则。


----

原文地址：[http://shellhue.github.io/2016/03/29/concurrency/](http://shellhue.github.io/2016/03/29/concurrency/)
