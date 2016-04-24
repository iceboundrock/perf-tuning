#Android App崩溃解决和性能调优总结

##崩溃问题

###见招拆招vs打破沙锅问到底

修复Java崩溃往往是很相对简单的工作，因为崩溃系统里通常会记录下比较消息的调用栈。通过异常类型和调用栈基本就可以定位到出现问题的位置，有时候在出现问题的位置简单的加上try/catch就可以解决崩溃。但是，这样真的解决问题了么？未必！

给大家讲一个StackOverflow引发的<del>血案</del>思考。App中曾经报告了一个异常，原因是遍历某个List时引发了StackOverflow，我Google了很久并没有找到类似的异常，于是就想用try/catch防御一下算了。但是总是觉得不甘心，于是就翻查相关的代码，终于被我找到了幕后真凶。崩溃的原因是在向某个List添加元素后，我们会对其排序，只留下最前面的n个元素，一开始编写这个方法的同学充分考虑了性能问题，决定采用List.subList方法而不是删掉不需要的元素。然而每次subList都会产生一个新的SubAbstractList对象，其fullList字段指向原先的SubAbstractList，然后通过start和end两个索引标记sub list的区域，如下示意图：

```
  SubList_[M]   // 第M次subList之后的对象
    +fullList->SubList_[M-1]
                 +fullList->SubList_[M-2]
                             \
                              \
                              SubList_1
                                +fullList:list  // 最初的list对象
```

一定次数之后再调用SubAbstractList.listIterator就会不断的递归调用其fullList的SubAbstractList.listIterator，导致stack overflow error。彻底解决这个问题其实也非常简单，对最初的list排序之后不要用subList，而是直接删掉最后不要的元素即可。

这虽然是个简单的问题，但是也充分说明了一个道理，简单的try/catch并不能彻底解决问题，反而会掩盖问题并且导致更大的麻烦，如果上面的例子中只是简单的catch异常，会导致这个list永远无法被遍历，程序功能出错。


###一些常见的造成崩溃的坑和避免方法

* NPE

    Null Pointer Exception可能是我们日常工作中遇到的最多的异常问题，解决此问题非常简单直接，只要找到崩溃call stack最顶层的方法，在其中加入防御对象引用为null的语句即可。但是如上面谈到的，不要见招拆招，防御了null也许并不是问题的终结而是更严重问题的开始。所以我们需要更加进一步分析并没有什么完美的办法，但是我们可以在工作中

* 异步处理结束时不判断回掉中使用对象的状态
    
    为了不卡住UI线程，在Android开发中会大量的使用异步处理，也就是在工作线程中处理一些耗时操作，待操作完成后通过handler等机制把

* OOM

    Out Of Memory也是一类比较常见的崩溃。造成OOM的原因很多，

* NDK崩溃

    不少做安卓的同学对C/C++相对陌生，再加上native崩溃往往并不会像Java崩溃一样在log中打印出详细的call stack导致native代码崩溃成为老大难问题。其实Android NDK中已经为我们提供了很方便的工具来分析NDK的崩溃。

###一些解决崩溃的技巧

1. 使用retrace来恢复混淆过的调用栈
2. 使用这个脚本来分析ANR trace

```

grep  "waiting on <" traces.txt  | sort | uniq -c 
grep  "locked <" traces.txt  | sort | uniq -c 

```

###防御性编程

在编写公共类型的公共方法时，尽量的对参数和对象自身状态进行合法性检查。
    
##性能调优

兵法讲究攻心为上，攻城为下，性能调优亦是如此。在做性能优化时不要拘泥于技术层面，从产品层面优化往往有更加突出的效果。本文的讨论集中

###调优方法论

调优与其他事情类似，需要明确的目标才能收到效果。所以我们要先收集数据，再进行优化。没有数据依据的优化都是耍流氓。对于Android App，尤其是ME这种直播类App，我们最关心的指标只有一个：帧率。通常我们认为App的帧率保持在60FPS才是流畅的，如何让App的帧率保持在60FPS以上是我们优化ME的重中之重。
当然App还有其他性能指标，包括：

* CPU占用率
* GC频率
* 内存占用




*App调优的思维方式*

App调优与

App性能调优在某些方面


*优化RecyclerView*

尽量避免刷新

减少刷新

*优化动画*

安卓为我们提供了多种实现动画的方式，每种都有各自的优缺点。在ME优化过程中对动画做了一些优化，下面列举一些我们的实践供大家参考：

*优化逻辑*

当我们


1. 使用

###调优工具介绍

经验很重要，但是经验不能代替工具。Android性能分析工具

1. Android Studio 2.0。Android Stuido不仅仅是开发工具，开发者还可以在Android Studio中非常方便的收集性能数据。
2. Android手机自带的开发者工具

    2.1 
   
3. DDMS
4. 


###一些调优经验

*内存优化*

使用对象池避免无谓的分配


*卡顿、掉帧*

*高CPU*

*动画运行速度不均匀*

    


##所有问题的终极解决之道

#READ THE FXXKING CODE