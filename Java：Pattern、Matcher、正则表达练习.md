
# 1 前言
 
参考文章：

java之Pattern、Matcher详细用法      https://blog.csdn.net/zhanngle/article/details/1750556


Learn Regex    https://github.com/ziishaned/learn-regex/blob/master/translations/README-cn.md


正则表达式语法  https://www.runoob.com/regexp/regexp-syntax.html


4种先行断言和后行断言   https://blog.csdn.net/hudie1234567/article/details/6642181


正则表达式之捕获组和非捕获组    https://www.jianshu.com/p/2547f0e3e809


实践：不以某字符串开头    https://segmentfault.com/a/1190000019266662

---

正则表达式就是一段字符串的通向公式，所以，正则表达式就是为了去匹配字符串，在应用时主要有两种用法：

1. 匹配字符串是否满足正则表达式，match，匹配整个字符串
2. 用正则表达式去分割字符串，split
3. 用正则字符串寻找，find

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


        Pattern pattern_2 = Pattern.compile("<.*?>");
        Matcher matcher2 = pattern_2.matcher(input);

        if (matcher2.find()) {
            System.out.println(matcher2.toMatchResult().start());
            System.out.println(matcher2.toMatchResult().end());

        }

        System.out.println(Arrays.toString(pattern_2.split(input)));


    }
}
```

# 3 正则表达式语法

正则表达式(regular expression)描述了一种字符串匹配的模式（pattern），可以用来检查一个串是否含有某种子串、将匹配的子串替换或者从某个串中取出符合某个条件的子串等。

例如：

runoo+b，可以匹配 runoob、runooob、runoooooob 等，+ 号代表前面的字符必须至少出现一次（1次或多次）。

runoo*b，可以匹配 runob、runoob、runoooooob 等，* 号代表字符可以不出现，也可以出现一次或者多次（0次、或1次、或多次）。

colou?r 可以匹配 color 或者 colour，? 问号代表前面的字符最多只可以出现一次（0次、或1次）。

构造正则表达式的方法和创建数学表达式的方法一样。也就是用多种元字符与运算符可以将小的表达式结合在一起来创建更大的表达式。正则表达式的组件可以是单个的字符、字符集合、字符范围、字符间的选择或者所有这些组件的任意组合。

正则表达式是由普通字符（**例如**字符 a 到 z）以及特殊字符（称为"元字符"）组成的文字模式。模式描述在搜索文本时要匹配的一个或多个字符串。正则表达式作为一个模板，将某个字符模式与所搜索的字符串进行匹配。

下面介绍一下普通字符和特殊字符。

## 3.1 普通字符
普通字符包括没有显式指定为元字符的所有可打印和不可打印字符。这包括所有大写和小写字母、所有数字、所有标点符号和一些其他符号。

打印字符就是在黑乎乎的终端能显式看到的字符，不可打印字字符就是实际在终端已经输出了，但是并不能看到，比如，换行符，如果忽略闪烁的光标，就知道已经下移一行了。

非打印字符也是正则表达式的组成部分，也可以匹配。比如：

```java
System.out.println("line67 " + Pattern.matches("\n+", "\n\n"));
```

非打印字符包括：

|字符|描述|
| ---  |                 ---                  |
| \cx  | 匹配由x指明的控制字符。例如， \cM 匹配一个 Control-M 或回车符。x 的值必须为 A-Z 或 a-z 之一。否则，将 c 视为一个原义的 'c' 字符。|
| \f   | 匹配一个换页符。等价于 \x0c 和 \cL。|
| \n   | 匹配一个换行符。等价于 \x0a 和 \cJ。|
| \r   | 匹配一个回车符。等价于 \x0d 和 \cM。|
| \s   | 匹配任何空白字符，包括空格、制表符、换页符等等。等价于 [ \f\n\r\t\v]。注意 Unicode 正则表达式会匹配全角空格符。|
| \S   | 匹配任何非空白字符。等价于 [^ \f\n\r\t\v]。|
| \t   | 匹配一个制表符。等价于 \x09 和 \cI。|
| \v   | 匹配一个垂直制表符。等价于 \x0b 和 \cK。|


## 3.2 特殊字符

所谓特殊字符，就是一些有特殊含义的字符，如上面说的 runoo*b 中的 \*，简单的说就是表示任何字符串的意思。如果要查找字符串中的 \* 符号，则需要对 \* 进行转义，比如：

```java
/*
1*2 去匹配  1*2，但是正则表达式里*是元字符，需要转译一下，于是使用  1\*2 作为正则表达式，
但是，那个 \ 自身要作为转译符使用，自身还要被转译一下，因此，使用  1\\*2  作为最终的正则匹配表达式

bash里面所见即所得，可直接使用  1\*2  匹配   1*2
*/
System.out.println("line69 " + Pattern.matches("1\\*2", "1*2"));
```

同时，就像上面所做的一样，许多元字符要求在试图匹配它们时特别对待。若要匹配这些特殊字符，必须首先使字符"转义"，即，将反斜杠字符\ 放在它们前面。下表列出了正则表达式中需要用两个斜杠转译为正常字符的特殊字如下：

| 特殊字符 |  描述  |
|  ---    | --- |
| $	    | 匹配输入字符串的结尾位置。如果设置了 RegExp 对象的 Multiline 属性，则 $ 也匹配 '\n' 或 '\r'。要匹配 $ 字符本身，请使用 \$。|
| ( )	| 标记一个子表达式的开始和结束位置。子表达式可以获取供以后使用。要匹配这些字符，请使用 \( 和 \)。|
| *	    | 匹配前面的子表达式零次或多次。要匹配 * 字符，请使用 \\*。 |
| +	    | 匹配前面的子表达式一次或多次。要匹配 + 字符，请使用 \\+。 |
| .	    | 匹配除换行符 \n 之外的任何单字符。要匹配 . ，请使用 \\. 。|
| [	    | 标记一个中括号表达式的开始。要匹配 [，请使用 \\[。        |
| ?	    | 匹配前面的子表达式  或者  字符零次或一次，或指明一个非贪婪限定符。要匹配 ? 字符，请使用 \\?。|
| \	    | 将下一个字符标记为或特殊字符、或原义字符、或向后引用、或八进制转义符。例如， 'n' 匹配字符 'n'。'\n' 匹配换行符。序列 '\\\\' 匹配 "\\"，而 '\\(' 则匹配 "("。|
| ^	    | 匹配输入字符串的开始位置，除非在方括号表达式中使用，当该符号在方括号表达式中使用时，表示不接受该方括号表达式中的字符集合。要匹配 ^ 字符本身，请使用 \\^。|
| {     | 标记限定符表达式的开始。要匹配 {，请使用 \\{。  |
| |	    | 指明两项之间的一个选择。要匹配 |，请使用 \\|。  |


这些特殊字符都有各自的作用，从特殊字符的作用来看，分为以下几种特殊字符：

1. 限定符
2. 定位符
3. 选择
4. 反向引用

接下来分别介绍以上4中特殊字符的作用。

### 3.2.1 限定符
就是限定表达式 或者 字符 出现次数的特殊字符，比如：最常见的\*号，限定的次数就是0 ~ 无穷次，还有 \+ 号，\?号，还有稍微不常见的形式{n, }  {n,m}。

各个限定符的具体示意如下：

|   字符   |     描述   |
| ---   |  ---   |
|  *	| 匹配前面的子表达式零次或多次。例如，zo* 能匹配 "z" 以及 "zoo"。* 等价于{0,}。 |
|  +    | 匹配前面的子表达式一次或多次。例如，'zo+' 能匹配 "zo" 以及 "zoo"，但不能匹配 "z"。+ 等价于 {1,}。|
| ?     | 匹配前面的子表达式零次或一次。例如，"do(es)?" 可以匹配 "do" 、 "does" 中的 "does" 、 "doxy" 中的 "do" 。? 等价于 {0,1}。|
| {n}	| n 是一个非负整数。匹配确定的 n 次。例如，'o{2}' 不能匹配 "Bob" 中的 'o'，但是能匹配 "food" 中的两个 o。|
| {n,}	| n 是一个非负整数。至少匹配n 次。例如，'o{2,}' 不能匹配 "Bob" 中的 'o'，但能匹配 "foooood" 中的所有 o。'o{1,}' 等价于 'o+'。'o{0,}' 则等价于 'o*'。|
| {n,m}	| m 和 n 均为非负整数，其中n <= m。最少匹配 n 次且最多匹配 m 次。例如，"o{1,3}" 将匹配 "fooooood" 中的前三个 o。'o{0,1}' 等价于 'o?'。请注意在逗号和两个数之间不能有空格。|

比如，匹配一个正整数：

```java
System.out.println(
                Pattern.matches(
                        "[1-9][0-9]{0,}",
                        "84491789"
                )
        );
```

请注意，**限定符出现在范围表达式之后**。因此，它应用于整个范围表达式，在本例中，只指定从 0 到 9 的数字（包括 0 和 9）。而不使用 + 限定符，因为在第二个位置或后面的位置不一定需要有一个数字。也不使用 ? 字符，因为使用 ? 会将整数限制到只有两位数。

如果你想设置 0~99 的两位数，可以使用下面的表达式来至少指定一位但至多两位数字，可以这样匹配：

```java
System.out.println(
                Pattern.matches(
                        "[0-9]{1,2}",
                        "98"
                )
        );
```

上面的表达式的缺点是，只能匹配两位数字，而且可以匹配 0、00、01、10 99 的章节编号仍只匹配开头两位数字。改进下，匹配 1~99 的正整数表达式如下：

```java

System.out.println(
                Pattern.matches(
                        "[1-9][0-9]?",
                        "10"
                )
        );
//或者
System.out.println(
        Pattern.matches(
                "[1-9][0-9]{0,1}",
                "10"
        )
);

```

\* 和 + 限定符都是贪婪的，因为它们会尽可能多的匹配文字，只有在它们的后面加上一个 ? 就可以实现非贪婪或最小匹配。

```java
Pattern pattern_1 = Pattern.compile("<.*>");

String input = "<h1>jeason_chan</h1>";

Matcher matcher = pattern_1.matcher(input);
if (matcher.matches()) {
    System.out.println(matcher.toMatchResult().start());
    System.out.println(matcher.toMatchResult().end());

}

Pattern pattern_2 = Pattern.compile("<.*?>");
Matcher matcher2 = pattern_2.matcher(input);

if (matcher2.find()) {
    System.out.println(matcher2.toMatchResult().start());
    System.out.println(matcher2.toMatchResult().end());

}

System.out.println(Arrays.toString(pattern_2.split(input)));

/*
0
20
result:java.util.regex.Matcher$ImmutableMatchResult@100fc185
0
4
[, jeason_chan]


对于正则分割，开头的长度为0的字符串并不会被丢，只有末尾的才limit=0时才会背丢弃

*/
```


### 3.2.2 定位符
使用定位符修饰正则表达式之后，会在字符串的特定位置进行正匹配。

定位符用来描述字符串或单词的边界，^ 和 $ 分别指**整个字符串**的开始与结束，\b 描述**单词的前或后边界**，\B 表示**非单词边界**。

|  字符  |  描述   |
| ---    |   ---   |
| ^	  | 匹配输入字符串开始的位置。如果设置了 RegExp 对象的 Multiline 属性，^ 还会与 \n 或 \r 之后的位置匹配。|
| $	  | 匹配输入字符串结尾的位置。如果设置了 RegExp 对象的 Multiline 属性，$ 还会与 \n 或 \r 之前的位置匹配。|
| \b  |	匹配一个单词边界，即字与空格间的位置。|
| \B  |	非单词边界匹配。|

举例：

若要在搜索章节标题时使用定位点，下面的正则表达式匹配一个章节标题，该标题只包含两个尾随数字，并且出现在行首：

```
^Chapter [1-9][0-9]{0,1}
```

真正的章节标题不仅出现行的开始处，而且它还是该行中仅有的文本。它**既出现在行首又出现在同一行的结尾**。下面的表达式能确保指定的匹配只匹配章节而不匹配交叉引用。通过创建只匹配一行文本的开始和结尾的正则表达式，就可做到这一点。

```
^Chapter [1-9][0-9]{0,1}$
```

匹配单词边界稍有不同，但向正则表达式添加了很重要的能力。**单词边界是单词和空格之间的位置。非单词边界是任何其他位置。**下面的表达式匹配单词 Chapter 的开头三个字符，因为这三个字符出现在单词边界后面：

```
\bCha
```

\b 字符的位置是非常重要的。如果它位于要匹配的字符串的开始，它在单词的开始处查找匹配项。如果它位于字符串的结尾，它在单词的结尾处查找匹配项。例如，下面的表达式匹配单词 Chapter 中的字符串 ter，因为它出现在单词边界的前面：

```
ter\b
```

下面的表达式匹配 Chapter 中的字符串 apt，但不匹配 aptitude 中的字符串 apt：

```
\Bapt
```

字符串 apt 出现在单词 Chapter 中的非单词边界处，但出现在单词 aptitude 中的单词边界处。对于 \B 非单词边界运算符，位置并不重要，因为匹配不关心究竟是单词的开头还是结尾。

### 3.2.3 选择




### 3.2.4 反向引用


## 3.x 

```
^ABc.*\.(?!(meta|system|System|trash)).*


ABc_test.meta_aaa
ABc_aaaa.system_aaa
ABc_aaaa.System_aaa
ABc_aaaa.trash_aaa

(只能匹配以下两个)
ABc_aaaa.root_aaa
ABc_aaaa.usertable_aaa

=========================================

(^(?!wic).*)|(^wic.*(meta|system|System|trash)).*


aaa,aaaa
ABc_test.meta_aaa
ABc_aaaa.system_aaa
ABc_aaaa.System_aaa
ABc_aaaa.trash_aaa

（仅下面两个无法匹配）
ABc_aaaa.root_aaa
ABc_aaaa.usertable_aaa

```
