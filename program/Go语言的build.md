# Go语言的构建方法


趁着近期要换工作的空闲时间，看了一下Go语言，与C++相比，Go语言的确在不少地方轻便了不少，例如：增加了内置的字符串类型、多个返回值、支持协程、简单的构建方法等等。使得在生产效率方面有了不少的提高。今天这里对Go语言的构建方法做个简单的总结。

在C/C++的工程中，极少使用单个命令来编译代码，一般是通过一些工具来进行自动化的编译，刚开始的时候手动写makefile，再后来是繁复的Autotools，之后又出现了CMake，按照时间的推移，所需我们做的工作越来越少，例如在Autotools我们大致需要如下工作：

    1. autoscan扫描工作目录，之后手动修改生成的configure.ac。
    2. 使用aclocal命令，通过configure.ac来生成aclocal.m4。
    3. 使用autoconf命令生成configure脚本。
    4. 使用autoheader命令生成config.h.in。
    5. 手动创建Makefile.am文件，按照工程需要配置后再使用automake命令来生成Makefile.in文件。
    6. 再执行configure脚本，最后生成Makefile。

完成上述步骤后，才可以make，make install 来完成工程的编译的安装。而之后的CMake则简便不少，只需要配置几个CMakeList.txt，之后执行CMake命令，即可生成编译和安装所需的Makefile文件。

虽然随着技术的发展，C/C++会有更好的构建方法出现。但目前看来，是摆脱不了Makefile的，而在Go1发布时，舍弃了Makefile，直接引入了更为方便的方法：**Go命令行工具**。

Go命令行工具直接舍弃的工程文件的概念，只通过目录结构和包的名字来推倒工程结构和构建顺序，下面我们使用一个简单的例子(取自《Go语言编程》)来说明下Go中的基本工程管理方法。

这个例子是一个基于命令行的计算器。基本用法如下所示：

    $calc help
    USAGE: calc command [argument] ...
    
    The Command are:
    sqrt     Square root of a non-negative value
    add      Addation of two values
    
    $ calc sqrt 4 #对4进行开方
    2
    $ calc add 1 2
    3
    

根据需求，我们可以把工程分为两个部分，主程序和算法库，这样，当我们算法进行更新的时候，只修改实现就可以了，而不用修改对外的接口，这样就可以达到低耦合。工程目录如下所示：
<pre>
*calculator*
|----*src*
　　|----*calc*
　　　　|----calc.go
　　|----*simplemath*
　　　　|----add.go
　　　　|----add_test.go
　　　　|----sqrt.go
　　　　|----sqrt_test.go
|----*bin*
|----*pkg*
</pre>
在上面，斜体表示目录，正常文章表示文件，**xx_test.go表示是对xx.go的单元测试文件**，这是Go工程的命名规则。同时，工程下的目录src表示是源码目录，bin表示安装后的可执行程序目录，pkg表示包目录，这也是Go工程的命名规则。下面就是这个工程的代码了。
**calc.go**
<pre><code>// calc.go
package main

import (
	"fmt"
	"os"
	"simplemath"
	"strconv"
)

var Usage = func() {
	fmt.Println("USAGE: calc command [arguments]...")
	fmt.Println("\nThe commands are: \n\tAddition of two values.\n\tsqrt\tSquare root of a non-negative value")
}

func main() {
	args := os.Args
	if args == nil || len(args) < 2 {
		Usage()
		return
	}

	switch args[0] {
	case "add":
		if len(args) != 3 {
			fmt.Println("USAGE:calc add <integer1> <integer2>")
			return
		}
		v1, err1 := strconv.Atoi(args[1])
		v2, err2 := strconv.Atoi(args[2])
		if err1 != nil || err2 != nil {
			fmt.Println("USAGE:calc add <integer1> <integer2>")
			return
		}
		ret := simplemath.Add(v1, v2)
		fmt.Println("Result: ", ret)
	case "sqrt":
		if len(args) != 2 {
			fmt.Println("USAGE: calc sqrt <integer>")
			return
		}
		v, err := strconv.Atoi(args[1])
		if err != nil {
			fmt.Println("USAGE: calc sqrt <integer>")
			return
		}
		ret := simplemath.Sqrt(v)
		fmt.Println("Result: ", ret)
	default:
		Usage()
	}
}
</code>
</pre>
**add.go**
<pre><code>// add.go
package simplemath

func Add(a int, b int) int {
	return a + b
}
</code></pre>
**add_test.go**
<pre><code>// add_test.go
package simplemath

import "testing"

func TestAdd1(t *testing.T) {
	r := Add(1, 2)
	if r != 3 {
		t.Errorf("Add(1, 2) failed, Got %d, expected 3", r)
	}
}
</code></pre>
**sqrt.go**
<pre><code>// sqrt.go
package simplemath

import "math"

func Sqrt(i int) int {
	v := math.Sqrt(float64(i))
	return int(v)
}
</code></pre>
由于篇幅问题，单元测试代码在此省略。

完成代码后，就要进行编译了。首先需要设置环境变量**GOPATH**的值，将**calcuator的目录赋给GOPATH**,保存后重新载入即可。假设calcuator的目录是"~/gobuild",那么在linux下可以执行以下命令：
<pre>
export GOPATH=~/gobuild/calcuator
source ~/.bashrc
</pre>
设置完环境变量后，就可以开始构建工程了，进入calcuator的目录，执行命令：
<pre>cd bin
go build calc</pre>
之后就可以在目录下发现名字为calc的可执行程序。按照先前的功能进行试验，即可以看到对应的执行结果。

以上就是Go进行构建的过程，按照Go的要求组织好目录后，真正进行构建的就只是**go build calc**这条命令。可以说是非常简单快捷。而同样，进行单元测试，在bin目录下执行命令：
<pre>go test simplemath</pre>
即可。

以上就是Go进行构建的一个简单总结。由于是刚开始接触Go语言，如果有错误的地方，请指正。谢谢
xiaoniu
[2/30]