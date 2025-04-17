# channel

### chan数据结构

```
type hchan struct {
    qcount   uint           // 当前队列中剩余元素个数
    dataqsiz uint           // 环形队列长度，即可以存放的元素个数
    buf      unsafe.Pointer // 环形队列指针
    elemsize uint16         // 每个元素的大小
    closed   uint32            // 标识关闭状态
    elemtype *_type         // 元素类型
    sendx    uint           // 队列下标，指示元素写入时存放到队列中的位置
    recvx    uint           // 队列下标，指示元素从队列的该位置读出
    recvq    waitq          // 等待读消息的goroutine队列
    sendq    waitq          // 等待写消息的goroutine队列
    lock mutex              // 互斥锁，chan不允许并发读写
}
从数据结构可以看出channel由队列、类型信息、goroutine等待队列组成


```

下图展示了一个可缓存6个元素的channel示意图：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_f1b42d200c5d94d02eeacef7c99aa81b_r.png)

- dataqsiz指示了队列长度为6，即可缓存6个元素；

- buf指向队列的内存，队列中还剩余两个元素；

- qcount表示队列中还有两个元素；

- sendx指示后续写入的数据存储的位置，取值[0, 6)；

- recvx指示从该位置读取数据, 取值[0, 6)；

  ​

### 等待队列

从channel读数据，如果channel缓冲区为空或者没有缓冲区，当前goroutine会被阻塞。
向channel写数据，如果channel缓冲区已满或者没有缓冲区，当前goroutine会被阻塞。

被阻塞的goroutine将会挂在channel的等待队列中：

- 因读阻塞的goroutine会被向channel写入数据的goroutine唤醒；
- 因写阻塞的goroutine会被从channel读数据的goroutine唤醒；



下图展示了一个没有缓冲区的channel，有几个goroutine阻塞等待读数据：

![img](https://www.topgoer.cn/uploads/gozhuanjia/images/m_f48c37e012c38de53aeb532c993b6d2d_r.png)



### 向channel写数据

向一个channel中写数据简单过程如下：

1. 如果等待接收队列recvq不为空，说明缓冲区中没有数据或者没有缓冲区，此时直接从recvq取出G,并把数据写入，最后把该G唤醒，结束发送过程；
2. 如果缓冲区中有空余位置，将数据写入缓冲区，结束发送过程；
3. 如果缓冲区中没有空余位置，将待发送数据写入G，将当前G加入sendq，进入睡眠，等待被读goroutine唤醒；

简单流程图如下：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_b235ef1f2c6ac1b5d63ec5660da97bd2_r.png)



### 从channel读数据

从一个channel读数据简单过程如下：

1. 如果等待发送队列sendq不为空，且没有缓冲区，直接从sendq中取出G，把G中数据读出，最后把G唤醒，结束读取过程；
2. 如果等待发送队列sendq不为空，此时说明缓冲区已满，从缓冲区中首部读出数据，把G中数据写入缓冲区尾部，把G唤醒，结束读取过程；
3. 如果缓冲区中有数据，则从缓冲区取出数据，结束读取过程；
4. 将当前goroutine加入recvq，进入睡眠，等待被写goroutine唤醒；

简单流程图如下：

![img](https://www.topgoer.cn/uploads/gozhuanjia/images/m_933ca9af4c3ec1db0b94b8b4ec208d4b_r.png)

### 关闭channel

关闭channel时会把recvq中的G全部唤醒，本该写入G的数据位置为nil。把sendq中的G全部唤醒，但这些G会panic。

除此之外，panic出现的常见场景还有：

1. 关闭值为nil的channel
2. 关闭已经被关闭的channel
3. 向已经关闭的channel写数据



### select

使用select可以监控多channel，比如监控多个channel，当其中某一个channel有数据时，就从其读出数据。

```
package main

import (
    "fmt"
    "time"
)

func addNumberToChan(chanName chan int) {
    for {
        chanName <- 1
        time.Sleep(1 * time.Second)
    }
}

func main() {
    var chan1 = make(chan int, 10)
    var chan2 = make(chan int, 10)

    go addNumberToChan(chan1)
    go addNumberToChan(chan2)

    for {
        select {
        case e := <- chan1 :
            fmt.Printf("Get element from chan1: %d\n", e)
        case e := <- chan2 :
            fmt.Printf("Get element from chan2: %d\n", e)
        default:
            fmt.Printf("No element in chan1 and chan2.\n")
            time.Sleep(1 * time.Second)
        }
    }
}
```

程序中创建两个channel： chan1和chan2。函数addNumberToChan()函数会向两个channel中周期性写入数据。通过select可以监控两个channel，任意一个可读时就从其中读出数据。

从channel中读出数据的顺序是随机的，事实上select语句的多个case执行顺序是随机的。select的case语句读channel不会阻塞，尽管channel中没有数据。这是由于case语句编译后调用读channel时会明确传入不阻塞的参数，此时读不到数据时不会将当前goroutine加入到等待队列，而是直接返回



1. 只有一个缓冲区的管道，写入类似加锁，读取类似解锁操作。

   ```golang
   var count int
   var ch = make(chan int, 1)

   for count < 10{
     ch <- 1
     count += 1
     <- ch
   }

   ```

2. 对于值为nil的管道，无论读写都会阻塞，而且是永久阻塞。

3. 管道关闭，管道缓冲区还有数据，仍然可以读取到数据。

4. 一个管道同时仅允许被一个协程读写。


可以通过函数变量设定管道是只读管道还是只写管道

```golang
func readChan(ch <-chan int) {		//只读管道
	for i := range ch {
		fmt.Println(i)
	}
}

func writeChan(ch chan<- int) {		//只写管道
	for i := 0; i < 5; i++ {
		ch <- i
	}
}

func main() {
	ch := make(chan int, 10)
	writeChan(ch)
	go readChan(ch)
	time.Sleep(2 * time.Second)
	close(ch)
}


```



# Array

1. 数组是值类型，赋值和传参会复制整个数组，而不是指针。因此改变副本的值，不会改变本身的值

2. %p打印必须用指针承接

   ​

# Slice

### Slice数据结构

```
type slice struct {
    array unsafe.Pointer
    len   int
    cap   int
}
从数据结构看Slice很清晰, array指针指向底层数组，len表示切片长度，cap表示底层数组容量。
```



### 使用make创建Slice

使用make来创建Slice时，可以同时指定长度和容量，创建时底层会分配一个数组，数组的长度即容量。

例如，语句`slice := make([]int, 5, 10)`所创建的Slice，结构如下图所示：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_332a02ff2dc338bb2cce150a23d37b1c_r.png)

该Slice长度为5，即可以使用下标slice[0] ~ slice[4]来操作里面的元素，capacity为10，表示后续向slice添加新的元素时可以不必重新分配内存，直接使用预留内存即可。



### 使用数组创建Slice

使用数组来创建Slice时，Slice将与原数组共用一部分内存。

例如，语句`slice := array[5:7]`所创建的Slice，结构如下图所示：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_c6aff21b79ce0b735065a702cb84c684_r.png)

切片从数组array[5]开始，到数组array[7]结束（不含array[7]），即切片长度为2，数组后面的内容都作为切片的预留内存，即capacity为5。

数组和切片操作可能作用于同一块内存，这也是使用过程中需要注意的地方。



### 特殊切片

根据数组或切片生成新的切片一般使用`slice := array[start:end]`方式，这种新生成的切片并没有指定切片的容量，实际上新切片的容量是从start开始直至array的结束。

比如下面两个切片，长度和容量都是一致的，使用共同的内存地址：

```
sliceA := make([]int, 5, 10)
sliceB := sliceA[0:5]
```

根据数组或切片生成切片还有另一种写法，即切片同时也指定容量，即slice[start:end:cap], 其中cap即为新切片的容量，当然容量不能超过原切片实际值，如下所示：

```
    sliceA := make([]int, 5, 10)  //length = 5; capacity = 10
    sliceB := sliceA[0:5]         //length = 5; capacity = 10
    sliceC := sliceA[0:5:5]       //length = 5; capacity = 5
```

这切片方法不常见，在Golang源码里能够见到，不过非常利于切片的理解。



1. 变量声明切片，这个时候是个nil切片，不需要分配内存。

   ```golang
   var lst []int
   ```

2. 字面量初始化切片，这个时候是个空切片，不是nil。

   ```golang
   lst := []int{}
   ```

3. 推荐内置函数make创建切片，指定长度及预估空间可以有效减少切片扩容的内存分配及拷贝次数。

   ```golang
   make([]int,0,100)
   ```

4. 切片与原数组或者原切片共享底层空间，修改切片会影响原数组或者原切片。

5. 当切片空间不足时，append（）方法会先创建新的大容量切片（创建匿名数组，分配新的内存地址），将原有的数据拷贝到新切片中，再返回新切片。先扩容返回新切片，再append数据到新切片中。

6. make创建切片时，会在底层分配一个数组，数组的长度为切片的容量。

7. 使用数组创建切片时，切片与原数组公用一部分内存。

8. 扩容基本原则：

   1：如果原切片的容量小于1024，那么新切片的容量为原来的2倍；

   2：如果原切片容量大于或等于1024，则为原来的1.25倍；

9. copy函数不会产生扩容，copy数量为两个切片最小的数量。

10. 通过函数传递切片时，不是引用切片，而是拷贝切片指针。

  ```golang
  函数内部修改切片时，如果切片在函数内部扩容，则不会修改到外部切片，如果没有扩容则会修改外部切片值。
  ```

11. 切片容量等于数组的长度减去切片的low index。

12.  ​

    ```golang
    a := []int{1,2,3,4,5}
    b := a[1:4]
    b = append(b,0)
    由于b添加0的时候没有执行扩容操作，并且b的容量为5-1=4，但是b只有3个值，容量还有空余，那么b添加0会直接修改到a，所有5被修改为0
    ```

    ​

# Map

### map数据结构

```
type hmap struct {
    count     int // 当前保存的元素个数
    ...
    B         uint8
    ...
    buckets    unsafe.Pointer // bucket数组指针，数组的大小为2^B
    ...
}

```

下图展示一个拥有4个bucket的map：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_897a05f6373f7f966d00d1bfea6274d2_r.png)

本例中, `hmap.B=2`， 而hmap.buckets长度是2^B为4. 元素经过哈希运算后会落到某个bucket中进行存储。查找过程类似。

`bucket`很多时候被翻译为桶，所谓的`哈希桶`实际上就是bucket。



### bucket数据结构

bucket数据结构由`runtime/map.go:bmap`定义：

```
type bmap struct {
    tophash [8]uint8 //存储哈希值的高8位
    data    byte[1]  //key value数据:key/key/key/.../value/value/value...
    overflow *bmap   //溢出bucket的地址
}
```

每个bucket可以存储8个键值对。

- tophash是个长度为8的数组，哈希值相同的键（准确的说是哈希值低位相同的键）存入当前bucket时会将哈希值的高位存储在该数组中，以方便后续匹配。
- data区存放的是key-value数据，存放顺序是key/key/key/…value/value/value，如此存放是为了节省字节对齐带来的空间浪费。
- overflow 指针指向的是下一个bucket，据此将所有冲突的键连接起来。

注意：上述中data和overflow并不是在结构体中显示定义的，而是直接通过指针运算进行访问的。

下图展示bucket存放8个key-value对：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_7f0ba5a124641b1413279892581513c4_r.png)

### 哈希冲突

当有两个或以上数量的键被哈希到了同一个bucket时，我们称这些键发生了冲突。Go使用链地址法来解决键冲突。
由于每个bucket可以存放8个键值对，所以同一个bucket存放超过8个键值对时就会再创建一个键值对，用类似链表的方式将bucket连接起来。

下图展示产生冲突后的map：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_a8b9e5919d9951a71c1c36445dd68521_r.png)

bucket数据结构指示下一个bucket的指针称为overflow bucket，意为当前bucket盛不下而溢出的部分。事实上哈希冲突并不是好事情，它降低了存取效率，好的哈希算法可以保证哈希值的随机性，但冲突过多也是要控制的，后面会再详细介绍



### 负载因子

负载因子用于衡量一个哈希表冲突情况，公式为：

```
负载因子 = 键数量/bucket数量
```

例如，对于一个bucket数量为4，包含4个键值对的哈希表来说，这个哈希表的负载因子为1.

哈希表需要将负载因子控制在合适的大小，超过其阀值需要进行rehash，也即键值对重新组织：

- 哈希因子过小，说明空间利用率低
- 哈希因子过大，说明冲突严重，存取效率低

每个哈希表的实现对负载因子容忍程度不同，比如Redis实现中负载因子大于1时就会触发rehash，而Go则在在负载因子达到6.5时才会触发rehash，因为Redis的每个bucket只能存1个键值对，而Go的bucket可能存8个键值对，所以Go可以容忍更高的负载因子。



### 扩容的前提条件

为了保证访问效率，当新元素将要添加进map时，都会检查是否需要扩容，扩容实际上是以空间换时间的手段。
触发扩容的条件有二个：

1. 负载因子 > 6.5时，也即平均每个bucket存储的键值对达到6.5个。

2. overflow数量 > 2^15时，也即overflow数量超过32768时。

   ​

### 增量扩容

当负载因子过大时，就新建一个bucket，新的bucket长度是原来的2倍，然后旧bucket数据搬迁到新的bucket。
考虑到如果map存储了数以亿计的key-value，一次性搬迁将会造成比较大的延时，Go采用逐步搬迁策略，即每次访问map时都会触发一次搬迁，每次搬迁2个键值对。

![img](https://www.topgoer.cn/uploads/gozhuanjia/images/m_2f0122f26e5d66ca91e6820ace6b379b_r.png)

hmap数据结构中oldbuckets成员指身原bucket，而buckets指向了新申请的bucket。新的键值对被插入新的bucket中。
后续对map的访问操作会触发迁移，将oldbuckets中的键值对逐步的搬迁过来。当oldbuckets中的键值对全部搬迁完毕后，删除oldbuckets。

搬迁完成后的示意图如下：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_b1178e0a3cea02c9386e5f5eaa6f99a6_r.png)

数据搬迁过程中原bucket中的键值对将存在于新bucket的前面，新插入的键值对将存在于新bucket的后面



### 等量扩容

所谓等量扩容，实际上并不是扩大容量，buckets数量不变，重新做一遍类似增量扩容的搬迁动作，把松散的键值对重新排列一次，以使bucket的使用率更高，进而保证更快的存取。
在极端场景下，比如不断地增删，而键值对正好集中在一小部分的bucket，这样会造成overflow的bucket数量增多，但负载因子又不高，从而无法执行增量搬迁的情况，如下图所示：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_f3a5989c90204df9304d5ae246f3db72_r.png)

上图可见，overflow的bucket中大部分是空的，访问效率会很差。此时进行一次等量扩容，即buckets数量不变，经过重新组织后overflow的bucket数量会减少，即节省了空间又会提高访问效率。



### 查找过程

根据key值算出哈希值
取哈希值低位与hmap.B取模确定bucket位置
取哈希值高位在tophash数组中查询
如果tophash[i]中存储值也哈希值相等，则去找到该bucket中的key值进行比较
当前bucket没有找到，则继续从下个overflow的bucket中查找。
如果当前处于搬迁过程，则优先从oldbuckets查找



### 插入过程

根据key值算出哈希值
取哈希值低位与hmap.B取模确定bucket位置
查找该key是否已经存在，如果存在则直接更新值
如果没找到将key，将key插入



1. map为nil的时候，添加，修改元素会paic。

2. map并发操作需要加锁，不然会panic。

   ​


# String

```
type stringStruct struct {
    str unsafe.Pointer
    len int
}

- stringStruct.str：字符串的首地址；
- stringStruct.len：字符串的长度；

string数据结构跟切片有些类似，只不过切片还有一个表示容量的成员，事实上string和切片，准确的说是byte切片经常发生转换。这个后面再详细介绍。

```

1. 字符串不能修改。
2. 字符串不为nil。
3. 反单引号不需要转义符换行，双引号需要转义符换行\n。
4. 字符串使用+号拼接，会触发内存分配及内存拷贝，消耗内存资源，一般使用join方法。
5. string和[]byte转换会发生内存拷贝，有一定开销。
6. 字符串长度指的时字节数，而不是字符数。




# defer

###  defer数据结构

源码包`src/src/runtime/runtime2.go:_defer`定义了defer的数据结构：

```
type _defer struct {
    sp      uintptr   //函数栈指针
    pc      uintptr   //程序计数器
    fn      *funcval  //函数地址
    link    *_defer   //指向自身结构的指针，用于链接多个defer
}
```

我们知道defer后面一定要接一个函数的，所以defer的数据结构跟一般函数类似，也有栈地址、程序计数器、函数地址等等。

与函数不同的一点是它含有一个指针，可用于指向另一个defer，每个goroutine数据结构中实际上也有一个defer指针，该指针指向一个defer的单链表，每次声明一个defer时就将defer插入到单链表表头，每次执行defer时就从单链表表头取出一个defer执行。

下图展示多个defer被链接的过程：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_226214a05ea08033680d03d624a60de3_r.png)

从上图可以看到，新声明的defer总是添加到链表头部。

函数返回前执行defer则是从链表首部依次取出执行，不再赘述。

一个goroutine可能连续调用多个函数，defer添加过程跟上述流程一致，进入函数时添加defer，离开函数时取出defer，所以即便调用多个函数，也总是能保证defer是按LIFO方式执行的

1. defer定义的延迟函数参数在defer语句出现时就已经确定下来了

   ```golang
   func a() {
       i := 0
       defer fmt.Println(i)
       i++
       return
   }
   i为0，传参的时候已经确定了参数值
   ```

2. defer定义顺序与实际执行顺序相反，栈的形式，先入后出(LIFO)

3. return不是原子操作，执行过程是: 保存返回值(若有)–>执行defer（若有）–>执行ret跳转

   ```
   func deferFuncReturn() (result int) {
       i := 1

       defer func() {
          result++
       }()

       return i
   }
   返回步骤是 result = i =>  result ++ 
   ```

4. 申请资源后立即使用defer关闭资源是好习惯

5. recover一定要被defer方法直接调用，不然会返回nil，捕获不到错误。

   ```golang
   func IsPanic() bool {
       if err := recover(); err != nil {
           fmt.Println("Recover success...")
           return true
       }

       return false
   }

   func UpdateTable() {
       // defer中决定提交还是回滚
       defer func() {
           if IsPanic() {
               // Rollback transaction
           } else {
               // Commit transaction
           }
       }()

       // Database update operation...
   }

   recover 在IsPanic函数中判断 没有被defer直接调用，所以不能正确捕获错误。
   ```


# select

```
type scase struct {
    c           *hchan         // chan
    kind        uint16
    elem        unsafe.Pointer // data element
}
scase.c为当前case语句所操作的channel指针，这也说明了一个case语句只能操作一个channel。
scase.kind表示该case的类型，分为读channel、写channel和default，三种类型分别由常量定义：

caseRecv：case语句中尝试读取scase.c中的数据；
caseSend：case语句中尝试向scase.c中写入数据；
caseDefault： default语句
scase.elem表示缓冲区地址，根据scase.kind不同，有不同的用途：

scase.kind == caseRecv ： scase.elem表示读出channel的数据存放地址；
scase.kind == caseSend ： scase.elem表示将要写入channel的数据存放地址；
```

1. select是Golang在语言层面提供的多路IO复用的机制，其可以检测多个channel是否ready(即是否可读或可写)

2. select语句中除default外，各case执行顺序是随机的

3. select语句中如果没有default语句，则会阻塞等待任一case

4. 空select协程被阻塞，同时Golang自带死锁检测机制，当发现当前协程再也没有机会被唤醒时，则会panic。所以上述程序会panic。

5. select中已经关闭的chan也是可读的，所以不会阻塞协程。

6. select语句中除default外，每个case操作一个channel，要么读要么写

7. select语句中读操作要判断是否成功读取，关闭的channel也可以读取

   ```golang
   select{
     case value,ok := <- chan:		//ok判断是否正确读出，关闭的通道也是可读的，此时ok=false
     	.......
   }
   ```

   ​

# range

1. 遍历过程中可以视情况放弃接收index或value，可以一定程度上提升性能

2. 遍历channel时，如果channel中没有数据，可能会阻塞

3. 尽量避免遍历过程中修改原数据

4. 遍历slice前会先获取slice的长度len_temp作为循环次数，循环体中，每次循环会先获取元素值，如果for-range中接收index和value的话，则会对index和value进行一次赋值。由于循环开始前循环次数就已经确定了，所以循环过程中新添加的元素是没办法遍历到的。

   ​



# Mutex

```
type Mutex struct {
    state int32
    sema  uint32
}
Mutex.state表示互斥锁的状态，比如是否被锁定等。
Mutex.sema表示信号量，协程阻塞等待该信号量，解锁的协程释放信号量从而唤醒等待信号量的协程。
```

Mutex.state是32位的整型变量，内部实现时把该变量分成四份，用于记录Mutex的四种状态。

下图展示Mutex的内存布局：

![null](https://www.topgoer.cn/uploads/gozhuanjia/images/m_45a91868c2c9d5dc2617e9fda0e46049_r.png)

- Locked: 表示该Mutex是否已被锁定，0：没有锁定 1：已被锁定。
- Woken: 表示是否有协程已被唤醒，0：没有协程唤醒 1：已有协程唤醒，正在加锁过程中。
- Starving：表示该Mutex是否处于饥饿状态，0：没有饥饿 1：饥饿状态，说明有协程阻塞了超过1ms。
- Waiter: 表示阻塞等待锁的协程个数，协程解锁时根据此值来判断是否需要释放信号量。

协程之间抢锁实际上是抢给Locked赋值的权利，能给Locked域置1，就说明抢锁成功。抢不到的话就阻塞等待Mutex.sema信号量，一旦持有锁的协程解锁，等待的协程会依次被唤醒。



###  加锁自旋

加锁时，如果当前Locked位为1，说明该锁当前由其他协程持有，尝试加锁的协程并不是马上转入阻塞，而是会持续的探测Locked位是否变为0，这个过程即为自旋过程。

自旋时间很短，但如果在自旋过程中发现锁已被释放，那么协程可以立即获取锁。此时即便有协程被唤醒也无法获取锁，只能再次阻塞。

自旋的好处是，当加锁失败时不必立即转入阻塞，有一定机会获取到锁，这样可以避免协程的切换。

自旋必须满足以下所有条件：

- 自旋次数要足够小，通常为4，即自旋最多4次
- CPU核数要大于1，否则自旋没有意义，因为此时不可能有其他协程释放锁
- 协程调度机制中的Process数量要大于1，比如使用GOMAXPROCS()将处理器设置为1就不能启用自旋
- 协程调度机制中的可运行队列必须为空，否则会延迟协程调度

### 自旋优势

自旋的优势是更充分的利用CPU，尽量避免协程切换。因为当前申请加锁的协程拥有CPU，如果经过短时间的自旋可以获得锁，当前协程可以继续运行，不必进入阻塞状态。

### 自旋劣势

如果自旋过程中获得锁，那么之前被阻塞的协程将无法获得锁，如果加锁的协程特别多，每次都通过自旋获得锁，那么之前被阻塞的进程将很难获得锁，从而进入饥饿状态。

为了避免协程长时间无法获取锁，自1.8版本以来增加了一个状态，即Mutex的Starving状态。这个状态下不会自旋，一旦有协程释放锁，那么一定会唤醒一个协程并成功加锁。（饥饿模式）

### 饥饿模式

自旋过程中能抢到锁，一定意味着同一时刻有协程释放了锁，我们知道释放锁时如果发现有阻塞等待的协程，还会释放一个信号量来唤醒一个等待协程，被唤醒的协程得到CPU后开始运行，此时发现锁已被抢占了，自己只好再次阻塞，不过阻塞前会判断自上次阻塞到本次阻塞经过了多长时间，如果超过1ms的话，会将Mutex标记为”饥饿”模式，然后再阻塞。

处于饥饿模式下，不会启动自旋过程，也即一旦有协程释放了锁，那么一定会唤醒协程，被唤醒的协程将会成功获取锁，同时也会把等待计数减1。

### 避免死锁

加锁后立即使用defer对其解锁，可以有效的避免死锁。



# RWMutex

```
type RWMutex struct {
    w           Mutex  //用于控制多个写锁，获得写锁首先要获取该锁，如果有一个写锁在进行，那么再到来的写锁将会阻塞于此
    writerSem   uint32 //写阻塞等待的信号量，最后一个读者释放锁时会释放信号量
    readerSem   uint32 //读阻塞的协程等待的信号量，持有写锁的协程释放锁后会释放信号量
    readerCount int32  //记录读者个数
    readerWait  int32  //记录写阻塞时读者个数
}
读写锁内部仍有一个互斥锁，用于将两个写操作隔离开来，其他的几个都用于隔离读操作和写操作。
```

1. 写锁需要阻塞写锁：一个协程拥有写锁时，其他协程写锁定需要阻塞
2. 写锁需要阻塞读锁：一个协程拥有写锁时，其他协程读锁定需要阻塞
3. 读锁需要阻塞写锁：一个协程拥有读锁时，其他协程写锁定需要阻塞
4. 读锁不能阻塞读锁：一个协程拥有读锁时，其他协程也可以拥有读锁


# 逃逸分析

所谓逃逸分析（Escape analysis）是指由编译器决定内存分配的位置，不需要程序员指定。
函数中申请一个新的对象

- 如果分配在栈中，则函数执行结束可自动将内存回收；
- 如果分配在堆中，则函数执行结束可交给GC（垃圾回收）处理；



1. 如果函数外部没有引用，则优先放到栈中；
2. 如果函数外部存在引用，则必定放到堆中；
3. 函数外部没有引用的对象，也有可能放到堆中，比如内存过大超过栈的存储能力。
4. 函数参数为interface类型，比如fmt.Println(a …interface{})，编译期间很难确定其参数的具体类型，也会产生逃逸。
5. 闭包引用会引起逃逸分析





1. 栈上分配内存比在堆中分配内存有更高的效率
2. 栈上分配的内存不需要GC处理
3. 堆上分配的内存使用完毕会交给GC处理
4. 逃逸分析目的是决定内分配地址是栈还是堆
5. 逃逸分析在编译阶段完成




# GC

1. go 1.3 版本gc模式使用标记清除法，1.5版本改成三色标记法，1.8版本改成三色标记法及混合写屏障机制
2. 标记清除法的缺点，需要stw并且stw时间太长，需要扫描整个heap堆栈信息，会产生heap堆栈碎片。
3. 三色标记法（广度优先算法）：
   1. 默认当前所有对象为白色，然后从根节点开始，查找根节点引用的第一层对象，把根节点引用的第一层对象设置成灰色对象。
   2. 查找所有灰色对象的下层对象，把下层对象设置成灰色对象，并且原有的灰色对象设置成黑色对象。
   3. 循环进行第二部操作，直到所有的引用对象变成黑色对象，然后把所有的白色对象删除。
   4. 不足在于，没有进行stw，那么标记的过程中，有可能黑色对象会重新引用新的白色对象。正好该白色对象的上游没有灰色对象，那么标记结束后，这个白色对象不会被标记到，那么就会被回收，这无疑是有问题 的。
4. 强三色不变式，不允许黑色对象引用白色对象
5. 弱三色不变式，如果白色对象被灰色对象引用，或者白色对象上游存在灰色对象，那么白色对象可以被黑色对象引用。
6. 强弱三色不变式引入屏障机制。
   1. 插入屏障：在对象A引用对象B的时候，对象B会置为灰色对象。（强三色不变式，由于保证栈的运行速度，插入屏障不在栈上使用，然而栈上也可能出现黑色对象引用白色对象的情况，所以三色标记结束结束时需要stw重新扫描一次栈上的对象，确保栈上新增加的白色对象，被修改为黑色对象。缺点：需要stw，时间再10~100ms之间）
   2. 删除屏障：被删除的对象，如果自身是灰色或者白色， 那么被标记灰色。（弱三色不变式，开始标记时会进行stw，保存当前快照，结束后对比，缺点：回收精度低，被删除对象仍然可以活到下一轮GC判断再删除。）
7. 三色标记法+混合写屏障机制
   1. GC时首先扫描栈上的所有可达对象，标记为黑色对象，之后添加的栈对象也为黑色对象。(不进行二次扫描，就无需stw)
   2. 被删除的对象被标记为灰色对象
   3. 被添加的对象被标记为灰色对象
8. gc触发条件
   1. 内存大小阈值， 内存达到上次gc后的2倍
   2. 达到定时时间 ，2m interval



# GMP

1. G代表协程，M代表内核线程，P代表处理器

2. 协程占用内存小一般为4KB，线程为4MB，进程为4GB。

3. 全局队列存着等待运行的G。

4. P列表存着未运行的P

5. P的本地队列，存着当前P下运行和等待运行G，存的数量有限，不超过256个，G'优先加入到P的本地队列，如果队列满了，则会把本地队列中一半的G移动到全局队列。

6. 运行必须有GMP三个组件。默认P的数量为cpu核心数，可以通过命令GOMAXPROCS来设置P的数量。

7. 线程M想运行任务就得获取 P，首先从 P 的本地队列获取 G，P 队列为空时，M 会尝试从全局队列拿一批 G 放到 P 的本地队列，如果全局队列没有G，那么它会从其他 P 的本地队列偷一半放到自己 P 的本地队列。偷的过程会把其他P的本地队列分为两半，然后把队列的后半拿到自己的本地队列中。

8. 当本线程的G因为某些原因调用阻塞时，会把M和G绑定，然后把P给别的M使用，如果没有空闲的M那么会新创建一个M继续运行。如果阻塞结束，那么M会先找回原先的P，如果P处于空闲状态，那么会更M重新组合，如果P已经跟别的M处于运行状态，那么M会去找P列表，查看是否有空闲的P，如果有就组合，没有的话那么M就会存放到休眠队列，G会放到全局队列。

9. 一个 goroutine 最多占用 CPU 10ms，防止其他 goroutine 被饿死

10. M0是服务启动时启动的第一个线程，初始化完成后他跟别的M没什么区别

11. 每个M创建的时候都默认会有一个G0，G0负责调度G的流程。

12. 如果一次性创建太多的G，那么当P的本地队列满了之后，会把P的本地队列分为两份，把前半部分打乱，放到全局队列中，然后把后半部分往前移，新创建的G也放到全局队列中。

13. 创建G的时候，运行的G会尝试唤醒空闲的P和M去组合执行。如果组合的MP没有G可以运行，那么M会处于自旋状态，不断寻找G，首先去全局队列找，没有的话再去其他P的本地队列找。（work stealing模式）

14. M创建的G会优先放再当前P的本地队列中。

15. 有多少个P就最多能并行多少个协程。

    ​

    ![img](https://img.kancloud.cn/76/4f/764f7be119026cc16314e87628e4013f_1920x1080.jpeg)

    ​




