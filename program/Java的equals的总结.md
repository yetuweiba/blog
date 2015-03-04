# Java里的equals总结

前段时间一直在工作中使用Java，由于有一些C++功底，于是简单看了一下Java相关的语法便开始编写代码，结果在创建一个自定义类，并将自定义类放入ArrayList中，之后查找ArrayList是否有此元素的时候，发现怎么也查询不到对应的元素。在网上搜了一下资料，发现原因是没有重写对象的equals()方法，导致无法查找到对应的对象。之后由查了与之联系的相关资料，便有了以下的总结。

这篇总结的形式是提出个问题，然后给出问题的答案。这是目前学习知识的一种尝试，可以让学习更有目的。

###Q1.什么时候应当重写对象的equals方法？
答：一般在我们需要进行值比较的时候，是需要重写对象的equals方法的。而例外情况在《effective java》的第7条“在改写equals的时候请遵守通用约定”中清楚描述了。

我们知道，在Java中，每个对象都继承于Object.如果不重写，则默认的equals代码如下所示：
```
public boolean euqals(Object obj){
    return this == obj;
}
```
由上面的代码可以看出，equal默认是使用“==”来判断两个对象是否相等。两个对象使用“==”比较的是对象的地址，只有两个引用指向的对象相同的时候，“==”才返回true。所以，在开头的例子中，就需要重写equals方法，让两个对象有equals的时候。

###Q2.如何重写equals？
答：首先，当改写equals方法时，需要保证满足它的通用约定。这些约定如下所示：

- 自反性，对于任意的引用值x,x.equals(x)一定为true。
- 对称性，对于任意的引用值x和y，当且仅当y.equals(x)时，x.equals(y)也一定返回true.
- 传递性，对于任意的引用值x,y,z。如果x.equals(y)返回true，y.euqals(z)返回true，则x.equals(z)也返回true。
- 一致性，对于任意的引用值x和y，如果用于equals比较的对象信息没有修改，那么，多次调用x.equals(y)要么一致返回true,要么一致返回false.
- 非空性，所有的对象都必须不等于null。

其实我觉的一个简单的方法是参照String的equals方法即可，官方出版，满足各种要求。其代码如下所示
```
public boolean equals(Object anObject) {
    if (this == anObject) {
        return true;
    }
    if (anObject instanceof String) {
        String anotherString = (String)anObject;
        int n = count;
        if (n == anotherString.count) {
            char v1[] = value;
            char v2[] = anotherString.value;
            int i = offset;
            int j = anotherString.offset;
            while (n– != 0) {
                if (v1[i++] != v2[j++])
                    return false;
            }
            return true;
        }
    }
    return false;
}
```
函数的解释如下所示：
1. 使用==检查“实参是否是指向对象的一个引用”。
2. 使用instanceof检查实参是否和本对象同类，如果不同类，就不相等。
3. 将实参转换为正确的类型。
4. 根据类的定义，检查实现此对象**值相等**的各个条件。

更详细的信息，还是请看《effective java》的第7条“在改写equals的时候请遵守通用约定”。
###Q3.修改equals时需要注意什么？
答：大致需要注意以下几点：

**若修改equals方法，也请修改hashCode方法**

首先这个是语言的一个[约定](http://docs.oracle.com/javase/7/docs/api/java/lang/Object.html#hashCode())，这么做的一个原因是当此对象作为哈希容器的元素时，需要依赖hashCode,对象默认的hashCode是返回一个此对象特有的hashCode，不同的对象的hashCode返回值是不一样的，而哈希容器处理元素时，是按照对象的哈希值将对象分配到不同的桶中，若我们不重写对象的hashCode,那么值相等的对象产生的哈希值也会不同，这样当在哈希容器中查找时，会找不到对应的元素。

更详细的信息请看《effective Java》的第8条“改写equals时总是要改写hashCode”。

**重写时保证函数声明的正确**
请注意equals的声明是
```
public boolean equals(Object obj)
```
参数类型是**Object**,如果参数类型是此对象类型的话，如下：
```
class Point{
final int x;
final int y;
public void Point(int x, int y)
    this.x = x;
    this.y = y;
}
public boolean euqals(Point obj){
         return (this.x == obj.x && this.y == obj.y);
    }
}
```
下面代码执行是按照我们的预期执行的。
```
Point a(1, 2);
Poinr b(1, 2);
System.out.println(a.equals(b));// 输出true
```
但是如果将类A放入容器中，则会出问题
```
import java.util.HashSet;

HashSet<Point> coll = new HashSet<Point>();
coll.add(a);
System.out.println(coll.contains(b));// 输出false
```
这是由于HashSet中的contains方法中调用的是equals(**Object** obj)，而Point中的equals(Object obj)仍是Object的equals，这个方法在前面已经说过了，比较的是对象的地址，所以在coll中调用contains(b)时，当然得不到true。

**当有继承关系时注意equals的正确**
当一个类重写equals方法后，另一个类继承此类，此时，可能会违反前面说到的**对称性**，代码如下所示：
```
public class ColoredPoint extends Point { 
    private final Color color;
    public ColoredPoint(int x, int y, Color color) {
        super(x, y);
        this.color = color;
    }

    @Override 
    public boolean equals(Object other) {
        boolean result = false;
        if (other instanceof ColoredPoint) {
            ColoredPoint that = (ColoredPoint) other;
            result = (this.color.equals(that.color) && super.equals(that));
        }
        return result;
    }
}
```
当我们作比较时
```
Point p = new Point(1, 2);
ColoredPoint cp = new ColoredPoint(1, 2, Color.RED);
System.out.println(p.equals(cp)); //输出ture
System.out.println(cp.equals(p)); //输出false
```
原因是当调用Point.equals的时候，只比较了Point的x和y坐标，同时ColoredPoint也是Point类型，所以上面第三行代码相等，而调用ColoredPoint的时候，Point不是ColoredPoint类型，这样就导致第四行代码输出false。

若我们忽略Color的信息来比较呢，例如将ColoredPoint的equals方法改为：
```
@overwrite
public boolean equals(Object obj){
    if((obj instanceof Point)){
        return false;
    }
    
    if(!(obj instanceof ColoredPoint)){
        return obj.equals(this);
    }
    
    return super.equals(obj) && ((ColoredPoint)obj).color == color;
}
```
这样就保证了对称性，但是却违反了传递性，即下面的情况：
```
ColoredPoint cp1 = new ColoredPoint(1, 2, Color.RED);
Point p = new Point(1, 2);
ColoredPoint cp2 = new ColoredPoint(1, 2, Color.BLUE);
System.out.println(cp1.equals(p)); //true
System.out.println(p.equals(cp2)); //true
System.out.println(cp1.equals(cp2)); //false
```
面对这种情况，大致有两种解决方案，一种[酷壳的文章--如何在Java中避免equals方法的隐藏陷阱](http://coolshell.cn/articles/1051.html)的最后一条，断绝了Point和ColoredPoint相等的可能，这是一种处理方法，认为Point和ColoredPoint是不同的。另一种方法是effective Java上提出的，**使用聚合而不是继承**，将Point作为ColoredPoint的一个成员变量。目前我倾向于这种方法，因为聚合比继承更灵活，耦合更低。这种方法的代码如下所示：
```
class ColoredPoint{
    private final Point point;
    private final Color color;
    
    public Point asPoint(){
        return point;
    }
    
    public boolean equals(Object obj){
        boolean ret = false;
        if(obj instanceof ColoredPoint){
            ColoredPoint that = (ColoredPoint)obj;
            ret = that.point.equals(point) && color.equals(that.color);
        }
        return ret;
    }
}
```
当ColoredPoint需要比较坐标时，可以调用asPoint方法来转化为坐标进行比较。其他情况比较坐标和颜色，这样就可以解决上面关于对称性和传递性的问题了。

以上就是全文的内容，由于水平有限，文章中难免会有错误，希望大家指正。谢谢

**[1/30]**

参考资料：
1. [effective Java](http://www.amazon.cn/Sun-%E5%85%AC%E5%8F%B8%E6%A0%B8%E5%BF%83%E6%8A%80%E6%9C%AF%E4%B8%9B%E4%B9%A6-Effective-Java%E4%B8%AD%E6%96%87%E7%89%88-Joshua-Bloch/dp/B001PTGR52/ref=sr_1_1?ie=UTF8&qid=1425477146&sr=8-1&keywords=effective+java)
2. [如何在Java中避免equals方法的隐藏陷阱](http://coolshell.cn/articles/1051.html)
3. [Java中equals()与hashCode()方法详解](http://bijian1013.iteye.com/blog/1972404)

