
# 1 前言
 
参考文章：

java之Pattern类详解      https://blog.csdn.net/thlzjfefe/article/details/80769648



正则表达式就是一段字符串的通向公式，所以，正则表达式就是为了去匹配字符串，在应用时主要有两种用法：

1. 匹配字符串是否满足正则表达式，match
2. 用正则表达式去分割字符串，split

下面以Java的Pattern和Matcher类为例，举例匹配和分割的使用。

# 2 Pattern、Matcher用法

```java
package default_package.Pattern_Matcher_正则表达式;

import java.util.Arrays;
import java.util.regex.Matcher;
import java.util.regex.Pattern;

public class Main {
    public static void main(String[] args) {
        /*
        先预编译一个Pattern实例，用于以后多次使用
         */
        Pattern pattern_a_b = Pattern.compile("a*b");

        Matcher pattern_a_b_matcher = pattern_a_b.matcher("aaaaaaaab");
        System.out.println(pattern_a_b_matcher.matches());
        System.out.println(pattern_a_b_matcher.toMatchResult());


        Matcher pattern_a_b_matcher2 = pattern_a_b.matcher("aaaaaaaabaaa");
        System.out.println(pattern_a_b_matcher2.matches());


        System.out.println(pattern_a_b.matcher("bbbbbb").matches());


        /*
        一次性编译、一次使用
         */

        System.out.println(Pattern.matches("a*b", "aaaaaaaaaaab"));


        /*
        正则处理看字符串和某个Pattern是否匹配外，还可以把Pattern作为分割符，对一串字符串进行分割
         */
        System.out.println(
                Arrays.toString(pattern_a_b.split("caaaaaabdsdsdszzzzzaaabkk"))
        );

        System.out.println(
                /*
                limit参数控制应用模式的次数，从而影响结果数组的长度。

                如果 limit 大于零，那么模式至多应用 limit-1 次，数组的长度不大于 n，并且数组的最后条目将包含除最后的匹配定界符之外的所有输入。

                如果 limit 非正，那么将应用模式的次数不受限制，并且数组可以为任意长度。

                如果 limit 为零，那么应用模式的次数不受限制，数组可以为任意长度，同时！！！！！！将丢弃尾部空字符串。
                 */
                Arrays.toString(pattern_a_b.split("caaaaaabdsdsdszzzzzaaabkk", 1))
        );




    }
}
```
