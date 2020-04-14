# 关于try-catch-finally块中的return语句

在Java的异常处理逻辑中，try-catch-finally的代码是按顺序执行，即如果try语句块中没有出现异常，那么将执行finally语句块；如果try语句块中出现异常，那么将执行对应的catch语句块的内容，然后继续执行finally语句块的内容。如果在try-catch-finally块中出现return语句会对返回结果产生怎样的影响呢？

Java在try-catch-finally块中对于函数返回值的处理逻辑是这样的：在执行finally语句之前会先将返回值的复制一份副本到一个本地变量表中，当finlly块中的逻辑执行完成以后，将副本读到操作栈顶，作为返回值返回使用。详见《深入Java虚拟机：JVM高级特性与最佳实践》第6章`6.3.7节`异常表运作的例子。

根据上述原因我们可以很容易得出下代码的输出值为1：

```java
public static int inc() {
    int x;
    try {
        x = 1;
        return x;
    } finally {
        x = 3;
    }
}

public static void main(String[] args) {
    System.out.println("x = " + inc());
}
```

输出如下：

```shel
x = 1
```

但是，如果finally有return语句结果又会如何呢

```java
public static int inc() {
    int x;
    try {
        x = 1;
        return x;
    } finally {
        x = 3;
        return x;
    }
}

public static void main(String[] args) {
    System.out.println("x = " + inc());
}
```

输出如下：

```shell
x = 3
```

这是由于在finally块中有return语句，java虚拟机将先执行finally块中的return语句返回之后就不会再执行try中的return，从而将try块中的return覆盖了。这种写法，编译是可以编译通过的，但是编译器会给予警告，所以不推荐在finally中写return，这会破坏程序的完整性。

那么，不在finally块中进行return就一定没问题吗？

事实上，如果在finally块中操作的对象不是值类型而是引用类型，那么对于引用类型的对象的属性进行修改则会产生影响，代码如下：

```java
public static IntContainer inc() {
    IntContainer x = new IntContainer();
    try {
        return x;
    } finally {
        x.val++;
    }
}

public static class IntContainer {
    int val = 1;
}

public static void main(String[] args) {
    System.out.println("x.val = " + inc().val);
}
```

输出如下：

```shell
x.val = 2
```

这是由于方法返回的是对象的引用，对于引用本身finally块虽然没有进行修改，但是对于引用对象属性的修改还是会生效。

## 总结

1、finally中的代码总会被执行。

2、当try、catch中有return时，也会执行finally。return的时候，要注意返回值的类型，是否受到finally中代码的影响。

3、finally中有return时，会直接在finally中退出，导致try、catch中的return失效。

