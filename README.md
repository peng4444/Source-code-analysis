# Source-code-analysis
some source code analysis with you!
# 源码分析系列
[源码解析博客](http://www.iocoder.cn/)
## Java基础源码
[java源码解析](https://www.cnblogs.com/shuangyueliao/p/11456677.html)
```markdown
**String具有不变性的原因：**
String被final修饰，它不可能被继承，也就是任何对String的操作方法，都不会被继承覆写
String中保存数据的是一个char数组的value，它被final修饰，它的内存地址一旦赋值无法修改
**String相等判断源码**
```
```java
public final class String implements java.io.Serializable, Comparable<String>, CharSequence{
    /** The value is used for character storage. */
    private final char value[];
        
    public boolean equals(Object anObject) {
        // 判断内存地址是否相同
        if (this == anObject) {
            return true;
        }
        // 待比较的对象是否是 String，如果不是 String，直接返回不相等
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            // 两个字符串的长度是否相等，不等则直接返回不相等
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                // 依次比较每个字符是否相等，若有一个不等，直接返回不相等
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
}
```

### List
[ArrayList源码分析](https://www.cnblogs.com/aweicy-1/p/12677597.html)
## Spring
[Spring源码阅读](https://github.com/seaswalker/spring-analysis)
[Spring容器启动源码解析](https://mp.weixin.qq.com/s?__biz=MzIxNTAwNjA4OQ==&mid=2247485830&idx=1&sn=2d92b019b2f0d8ee876abc1fb4113955&chksm=979fa760a0e82e76168162b3c20d389e6666088421ed51b9b9e53bd37773d068a958f289b943&mpshare=1&scene=23&srcid=&sharer_sharetime=1572097584959&sharer_shareid=d812adcc01829f0f7f8fb06aea118511#rd)

## Netty
[Netty Hello World 入门源码分析](https://www.cnblogs.com/rickiyang/p/12562408.html)
# 源码分析

## Netty源码分析

[netty源码解析](https://www.cnblogs.com/java-chen-hao/category/1537744.html)

## FastJSON源码分析

## SpringMVC源码分析

[[面试官：你分析过SpringMVC的源码吗？](https://www.cnblogs.com/javazhiyin/p/10717591.html)]

[一些源码分析]( [https://www.cnblogs.com/wyq1995/tag/%E6%BA%90%E7%A0%81/](https://www.cnblogs.com/wyq1995/tag/源码/) )

