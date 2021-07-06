#### Function.identity()的含义

Java 8允许在接口中加入具体方法。接口中的具体方法有两种，default方法和static方法，identity()就是Function接口的一个静态方法。
Function.identity()**返回一个输出跟输入一样的Lambda表达式对象**，等价于形如`t -> t`形式的Lambda表达式

      private static void identity() {
        Stream<String> stream = Stream.of("I", "love", "you", "too");
        Map<String, Integer> map = stream.collect(Collectors.toMap(Function.identity(), String::length));
        System.out.println(map);
    }

输出结果为：

```
     {love=4, too=3, I=1, you=3}
```

