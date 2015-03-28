#Go语言中的管道(Channel)总结

管道(Channel)是Go语言中比较重要的部分，经常在Go中的并发中使用。今天尝试对Go语言的管道来做以下总结。总结的形式采用问答式的方法，让答案更有目的性。

####Q1.管道是什么？
管道是Go语言在语言级别上提供的goroutine间的**通讯方式**，我们可以使用channel在多个goroutine之间传递消息。channel是**进程内**的通讯方式，是不支持跨进程通信的，如果需要进程间通讯的话，可以使用Socket等网络方式。

以上是管道的概念，下面我们就看下管道的语法。

####Q2.管道的语法

整个Go语言的语法都比较简洁，管道也不例外，其语法如下所示：

在此应当注意，管道是类型相关的，即一个管道只能传递一种类型的值。管道中的数据是**先进先出**的。

<pre><code>// 声明方式，在此ElemType是指此管道所传递的类型
var chanName chan ElemType
// 声明一个传递类型为int的管道
var ch chan int
// 声明一个map，元素是bool型的channel
var m map[string] chan bool

// 定义语法,定义需要使用内置函数make()即可，下面这行代码是声明+定义一个整型管道
ch := make(chan int)
// 事先定义好管道的size,下面这行代码定义管道的size为100
ch := make(chan int, 100)

// 由管道中读写数据，<-操作符是与最左边的chan优先结合的
// 向管道中写入一个数据，在此需要注意：向管道中写入数据通常会导致程序阻塞,直到有
// 其他goroutine从这个管道中读取数据
ch<- value
// 读取数据，注意：如果管道中没有数据，那么从管道中读取数据会导致程序阻塞，直到有数据
value := <-ch

// 单向管道
var ch1 chan<- float64     // 只能向里面写入float64的数据，不能读取
var ch2 <-chan int         // 只能读取int型数据

// 关闭channel，直接调用close()即可
close(ch)
// 判断ch是否关闭，判断ok的值，如果是false,则说明已经关闭(关闭的话读取是不会阻塞的)
x, ok := <-ch
</code></pre>

####Q3.管道的使用场景

在第一个问题中，我们已经知道管道可以做进程间通讯，Go中自带了对协程的支持(关键字go)，而管道就是各个协程间通讯的一个方法。这里我们举些简单的小例子来说明一下管道如何在协程中使用。

<pre><code>package main
import "fmt"

func print() {
    fmt.Println("Hello world")
}

func main() {
    for i := 0; i < 10; i++ {
        go print()
    }
}
</code></pre>

上面的代码意思大致是：使用协程来并行输出10次 "Hello world", 但是大家运行上面代码的时候，会发现不会有输出。这是因为虽然使用go关键字进行了协程的创建，但是还没有等到执行的时候，main函数已经退出来了，进程已经关闭，所以起来的协程也不会被执行。

如果你有C相关的多线程经验时，可已经将协程改为线程，之后调用线程的join方法，让主线程等待子线程执行完毕后再退出。而在Go语言中，这个时候我们就可以利用管道的写入阻塞和读取阻塞来完成类似线程join的行为。代码如下所示：

<pre><code>package main
import "fmt"

func print(ch chan int) {
    fmt.Println("Hello world")
    ch<- 1
}

func main() {
    chs := make([]chan int)
    for i := 0; i < 10; i++ {
        chs[i] = make(chan int)
        go print(chs[i])
    }
	
    for _, ch := range(chs){
        <-ch
    }
}
</code></pre>

通过以上代码，我们就可以完成了并行输出10此Hello world 的效果。

有一个问题留给大家，如果将 print改为
<pre><code>func print(ch chan int){
    ch<- 1
    fmt.Println("Hello world")
</code></pre>
会打印出什么呢？

由于水平有限，难免会有错误，请大家指正。
谢谢。
[3/30]



