[Java的参数是值传递，不是引用传递，不要误解了](https://github.com/denglei1024/denglei1024.github.io/issues/6)

## 引言
没有特殊情况，Java中的参数是按值传递给方法的，也就是把变量的副本传递给该方法。对于值类型，将值的副本传递给方法。对于引用类型，将引用的副本传递给方法。
术语“按值传递”和“按引用传递”有明确定义，但是现在有点滥用，我们一起来纠正这个说法。

## 按值传递
一种传递形参的方法，这种方法将实参复制到对应的形参中，形参和实参存储在不同的位置上。

## 按引用传递
调用函数时将实际参数的地址传递到函数中，那么在函数中对参数所进行的修改，将影响到实际参数。


## 测试示例

我这里写了一段测试代码，请你一起来运行看下结果。

```java
public class Example {  
    public String Value;  
  
    public static void main(String[] args) {  
        Example ex = new Example();  
        ex.Value ="hello";  
        fooObject(ex);  
        System.out.println("ex="+ ex.Value); 
    }
  
    public static void fooObject(Example a){  
        a.Value += ", world!";  // AAA
        a = new Example();      // BBB
        a.Value = "你好，世界！";  // CCC
    }  
}
```

如果你也运行的话，应该会和我得到一样的输出结果：
```
ex=hello, world!
```

Java中引用类型变量，存储的是数据在堆里的指针，传递到方法的是指针的副本，假设变量`ex`的存储的指针是10，那么在`fooObject`函数的参数`a`存储的指针也是10。运行到 AAA 行，通过指针修改了堆中的数据，此时`ex`也是受影响。

运行到 BBB 行时，会在堆内存申请新的空间，假设这块空间的指针是 89，那么此时 变量`a`存储的就是 89，在 CCC 行修改堆中的数据，对`ex`是没有影响的，因此最后的打印结果是 AAA 行改变后的值。
![image.png](https://note-1251668647.cos.ap-nanjing.myqcloud.com/20240509180156.png)


## 总结

这篇文章主要通过示例解释Java中的对象是按值传递的，没有按引用传递的说法。



## 参考资料：
- [Java值传递详解](https://javaguide.cn/java/basis/why-there-only-value-passing-in-java.html#%E5%BD%A2%E5%8F%82-%E5%AE%9E%E5%8F%82)
