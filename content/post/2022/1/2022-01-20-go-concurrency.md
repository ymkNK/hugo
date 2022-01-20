---
title: Go并发编程学习笔记-WaitGroup和Mutex
author: ymkNK
categories: [Go]
tags: [Go]
date: 2022-01-20 20:07:33 +0800
img: https://lllovol.oss-cn-beijing.aliyuncs.com/assets/img/post/4.jpeg
slug: 2022/1/go-concurrency
toc: true
---
##  原则
**不要用共享内存来通信，而是使用通信来共享内存**

## 注意
- 内存重排，机器码执行的顺序可能并不是代码的顺序
- Go的锁，不要使用值传递，不然每次都是复制新的值，这样的锁是失效的。这类bug可以被go vet检测出来（Go的所有的传递，都是值传递）

## sync.WaitGroup
debug.SetMaxThread(1)
```Go
func main() {
    var wg sync.WaitGroup
    var x int32 = 0
    for i := 0; i < 100; i++ {
        go func() {
			// wg.Add不应该放在这里，应该放在循坏外面
            wg.Add(1)
            atomic.AddInt32(&x, 1)
            wg.Done()
        }()
    }

    fmt.Println("等待片刻...")
    wg.Wait()
    fmt.Println(atomic.LoadInt32(&x))
}
```
这样写代码是不行，add放在主线程，done放在子协程中

### 如何实现
> [详解Go中WaitGroup设计](https://segmentfault.com/a/1190000040653924)

#### 简单思路
1. Add() 内置count记录多少任务完成，通过加锁来保证安全
2. Done() 完成后count--，加锁操作
3. Wait() 通过for循环加锁读取当前count，直到值为0

#### 实际实现
```Go
type WaitGroup struct {
   // https://golang.org/issues/8005#issuecomment-190753527
   // 使用时只能使用指针来传递，使用值传递会被go vet 检验出来
   noCopy noCopy
   // count Add()时增加这个值
   // wait  Wait() 时增加这个值
   // semap  信号量，与此相关的有两个操作，增加信号量，协程挂起；减少信号量，唤醒协程
   state1 [3]uint32
}
```

- WaitGroup使用CAS`atomic.CompareAndSwapUint64(statep, state, state+1)`代替锁，使用无锁的结构达到了线程安全的目的。将三个 count、wait、sema 定义成一个数组，来保证取count和wait的时候是原子性的
- 32位和64位的机器，这个state1的内存布局是不同的
- 使用int32(state >> 32)位移操作拿到counter
- uint32(state)拿到waiter

#### 引申
- 你知道ErrGroup吗？在日常开发中ErrGroup会比WaitGroup更实用. 因为所有如果有协程返回error, 可以通过ctx的机制，可以让其他的线程提前退出。
- waitGroup最多支持Add()多大的值？ int32
- waitGroup可以重复使用吗？ 可以 


## time.After不能和select、for放在一起用
如果一次性来5000次请求，会同时开5000个协程，然后执行一分钟，这样内存会直接爆炸。应该用taker
```Go
func longRunning(messages <-chan string) {
    for {
        select {
            case <-time.After(time.Minute):
        return
            case msg := <-messages:
            fmt.Println(msg)
        }
    }
}
```

## Mutex
1. 在defer中执行解锁，能够保证执行操作一定被执行
2. throw的panic不能恢复

### 如何实现
```Go
type Mutex struct {
   // 等待协程数|饥饿状态|是否被唤醒|是否被锁
   // 00000000 00000000 00000000 00000|0|0|0
   // 提问：为什么要有饥饿态呢？
   state int32
   // 信号量
   sema  uint32
}
```

#### 简单实现思路
- Lock() for循环直到CAS成功为止
- Unlock() 进行CAS

#### 实际实现
增加了一个饥饿态，其实就是一个队列。不断自旋来等待锁被释放、或者进入饥饿状态、或者不能再自旋。