# scala语法学习

## some、none、option用法



## Array,Seq,List区别

### List

标准的链表。

```scala
scala> List(1, 2, 3)
res0: List[Int] = List(1, 2, 3)
```

构造列表的两个基本单位是 **Nil** 和 **::**

**Nil** 也可以表示为一个空列表。

```scala
scala> 1::Nil
res24: List[Int] = List(1)
```

添加一个元素

```scala
scala> 22 +: res36
res39: List[Int] = List(22, 1, 1111)

scala> res36 :+ 22
res40: List[Int] = List(1, 1111, 22)
```





### 序列 Seq

序列有一个给定的顺序。

```scala
scala> Seq(1, 1, 2)
res3: Seq[Int] = List(1, 1, 2)
```

（请注意返回的是一个列表。因为`Seq`是一个特质；而列表是序列的很好实现。如你所见，`Seq`也是一个工厂单例对象，可以用来创建列表。）



## scala循环

普通用法：左箭头 <- 用于为变量 a 赋值。

迭代元素

```scala
object Test {
   def main(args: Array[String]) {
      var a = 0;
      val numList = List(1,2,3,4,5,6);

      // for 循环
      for( a <- numList ){
         println( "Value of a: " + a );
      }
   }
}
```



for  yield普通用法：

根据声明返回集合。

```scala
scala> val a = Array(1, 2, 3, 4, 5)
a: Array[Int] = Array(1, 2, 3, 4, 5)

scala> for (e <- a) yield e
res5: Array[Int] = Array(1, 2, 3, 4, 5)

scala> for (e <- a) yield e * 2
res6: Array[Int] = Array(2, 4, 6, 8, 10)

scala> for (e <- a) yield e % 2
res7: Array[Int] = Array(1, 0, 1, 0, 1)
```



for  yield高级用法：

迭代 `a*a`次，根据声明返回集合。

```scala
scala> a
res2: Array[Int] = Array(1, 2, 3)

scala> for(e <- a ; f <- a) yield (e,f)
res3: Array[(Int, Int)] = Array((1,1), (1,2), (1,3), (2,1), (2,2), (2,3), (3,1), (3,2), (3,3))
```



for yield总结：

- For each iteration of your *for* loop, yield generates a value which is remembered by the for loop (behind the scenes, like a buffer).
- When your for loop finishes running, it **returns** a collection of **all these yielded values**.
- The type of the collection that is returned is the **same type** that you were iterating over.



## Map使用

key和value之间用`->`连接。

对象声明使用`Map()`

```scala
df.write.format("jdbc").options(Map(
    "url" -> dataSource.url,
    "dbtable" -> dataSource.sourcetable,
    "driver" -> "ru.yandex.clickhouse.ClickHouseDriver",
    "user" -> dataSource.username,
    "password" -> dataSource.password
  ))
```





## scala下划线使用

### 第一：初始化的时候。

```scala
object Sample {

var name:String=_ 

	def main (args: Array[String]){ 
    name="hello world"  
    println(name) 
	} 
}


```

在这里，name也可以声明为null，例：var name:String=null。

这里的下划线和null的作用是一样的。

 

### 第二：引入的时候。

```scala
import math._
object Sample {
   def main (args: Array[String]){
    println(BigInt(123))
   }
}
```

这里的math._就相当于Java中的math.*; 即“引用包中的所有内容”。

 

### 第三：集合中使用。（最典型，最常用）

 ```scala
object Sample {
   def main (args: Array[String]){
    val newArry= (1 to 10).map(_*2)
   println(newArry)
   }
}
 ```


这里的下划线代表了集合中的“某（this）”一个元素。

这个用法很常见，在foreach等语句中也可以使用。

 

### 第四：模式匹配。

 ```scala
object Sample {
   def main (args: Array[String]){
     val value="a"
  val result=  value match{
       case "a" => 1
       case "b" => 2
       case _ =>"result"
     }
     println(result)
   }
}
 ```


在这里的下划线相当于“others”的意思，就像Java  switch语句中的“default”。

 

还有一种写法，是被Some“包”起来的，说明Some里面是有值的，而不是None。

```scala
object Sample {
  def main (args: Array[String]){
    val value=Some("a")
    val result=  value match{
      case Some(_) => 1
      case _ =>"result"
    }
    println(result)
  }
```



还有一种表示队列

```scala
object Sample {
  def main (args: Array[String]){
    val value=1 to 5
    val result=  value match{
      case Seq(_,_*) => 1
      case _ =>"result"
    }
    println(result)
  }
}

```

 

### 第五：函数中使用。

```scala
object Sample {
   def main (args: Array[String]){
    val set=setFunction(3.0,_:Double)
     println(set(7.1))
   }
  def setFunction(parm1:Double,parm2:Double): Double = parm1+parm2
}
```

这是Scala特有的“偏函数”用法。



一个匿名的函数传递给一个方法或者函数的时候，scala会尽量推断出参数类型。例如一个完整的匿名函数作为参数可以写为

```scala
scala> def compute(f: (Double)=>Double) = f(3)
compute: (f: Double => Double)Double

//传递一个匿名函数作为compute的参数
scala> compute((x: Double) => 2 * x)
res1: Double = 6.0
```

如果参数`x`在`=>`右侧只出现一次，可以用`_`替代这个参数，简写为

```scala
scala> compute(2 * _)
res2: Double = 6.0
```

更常见的使用方式为

```scala
scala> (1 to 9).filter(_ % 2 == 0)
res0: scala.collection.immutable.IndexedSeq[Int] = Vector(2, 4, 6, 8)

scala> (1 to 3).map(_ * 3)
res1: scala.collection.immutable.IndexedSeq[Int] = Vector(3, 6, 9)
```

以上所说的为一元函数，那么对于二元函数，即有两个参数x和y的函数，是如何使用`_`的？

可以参考sortWith方法的定义 `def   sortWith(lt: (T, T) ⇒ Boolean): Array[T]`
这个方法需要的参数是一个二元函数，而且函数参数的类型为T，例如

```scala
scala> List(10, 5, 8, 1, 7).sortWith(_ < _)
res0: List[Int] = List(1, 5, 7, 8, 10)
scala> List(10, 5, 8, 1, 7).sortWith(_ > _)
res13: List[Int] = List(10, 8, 7, 5, 1)
```

可以用`_`分别表示二元函数中的参数x和y。

 

### 第六：元组Tuple。

```scala
object Sample {
   def main (args: Array[String])={
     val value=(1,2)
     print(value._1)
   }
}
```



### 第七：传参。

 ```scala
object Sample {
   def main (args: Array[String])={
    val result=sum(1 to 5:_*)
     println(result)
   }
  def sum(parms:Int*)={
    var result=0
    for(parm <- parms)result+=parm
    result
  }
}
 ```

 当函数接收的参数不定长的时候，假如你想输入一个队列，可以在一个队列后加入“:_*”，因此，这里的“1 to 5”也可以改写为：“Seq(1,2,3,4,5)”。





### 第八：下划线和其他符号组合的使用方式

- 下划线与等号（_=）
   自定义setter方法，请参见[《Overriding def with var in Scala》](https://www.jianshu.com/p/4a3362ec22de)
   
- 下划线与星号（_*）
   1.变长参数
   例如定义一个变长参数的方法sum，然后计算1-5的和，可以写为

  ```scala
  scala> def sum(args: Int*) = {
       | var result = 0
       | for (arg <- args) result += arg
       | result
       | }
  sum: (args: Int*)Int

  scala> val s = sum(1,2,3,4,5)
  s: Int = 15
  ```

  但是如果使用这种方式就会报错

  ```scala
  scala> val s = sum(1 to 5)
  <console>:12: error: type mismatch;
   found   : scala.collection.immutable.Range.Inclusive
   required: Int
         val s = sum(1 to 5)
                       ^
  ```

  这种情况必须在后面写上`: _*`将`1 to 5`转化为参数序列

  ```scala
  scala> val s = sum(1 to 5: _*)
  s: Int = 15
  ```



  2.变量声明中的模式
  例如，下面代码分别将arr中的第一个和第二个值赋给first和second。

  类似前端es6的解构。

  ```scala
  scala> val arr = Array(1,2,3,4,5)
  arr: Array[Int] = Array(1, 2, 3, 4, 5)

  scala> val Array(1, 2, _*) = arr

  scala> val Array(first, second, _*) = arr
  first: Int = 1
  second: Int = 2
  ```



