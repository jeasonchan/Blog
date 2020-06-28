```bash
# 定义变量时不加美元符号
your_name="jeason"
echo $your_name

# 注意！变量名和等号之间不能有空格！
# 变量名的命名和其他语言类似，并且，不能使用bash中的关键字作为变量名，不能含有标点符号
# 毫无疑问，严格区分大小写！

# 除了显示的赋值，还可以用语句/表达式的值给变量赋值，比如：
for file in `ls /`  # ` ` 只是两个左上角的符号，里面放要执行的命令，返回值就是命令的控制台输出
do
    echo $file
done

echo "---------------------"

for file in $(ls /)  # 可见，通过 $ 符号可以将变量或者表达式的值取出来，这里取出来的是文件名称的列表
do
    echo $file
done

echo "--------------------"

# 使用变量，肯定是要用美元符号取变量的值的，但是，更加推荐用花括号将变量包起来，能帮助解释器识别变量的边界，如：
my_name="jeason"
echo $my_name
echo ${my_name}
echo "my name is $my_name"
echo "my name is ${my_name}chan"  # 此处能完美的识别变量

for language in C++ Python Java JavaScripts
do
    echo "I am good at ${language}"
done

# 对变量取值时加上花括号是最最最标准的写法，是优秀的习惯！！！！！！！！！

echo "-----------------------"
# 只读变量，定义完一个变量后，使用readonly对变量本身（而不是里面的值）进行操作，使其变为值只读
my_url="11111111"
readonly my_url
my_url="22222222"  # 控制台提示，变量是只读变量

echo "--------------------"

# 删除变量
# 使用unset 命令删除变量，变量被删除后便不能再次使用，同时，unset操作无法删除只读变量
test_var1="11111"
echo ${test_var1}
unset test_var1
echo ${test_var1}  # 这句话其实打印的是一行空值，感觉不是真正意义上的删除

echo "------------"

# 根据变量的作用域，可以分为三种变量：
# 1、局部变量，局部变量是在脚本中定义的，仅在当前shell实例中有效，并且由shell启动的程序也无法直接访问访问这个变量，
# 除非是以传递参数的方式，将该变量中的值传递给别的程序作为启动时的入参
# 2、环境变量，计算上任意运行的程序都能访问到的变量值，比如，CPU的核心数，必要的时候，shell也可以自定义环境变量
# 3、shell变量，shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，
# 这些变量保证了shell的正常运行

# shell字符串
# 字符串是shell编程中最常用最有用的数据类型（除了数字和字符串，也没啥其它类型好用了），
# 字符串可以用单引号，也可以用双引号，也可以不用引号。
# 单引号就是坑，不推荐，可以完全只使用双引号！要注意，不加引号也可以表示字符串，尤其是一些指令的返回值。
your_name="jeason"
str="hello,I am \"${your_name}\"!"
echo ${str}
# 可见，双引号里可以有变量，双引号里可以有转译符

# 获取字符串长度
string="123456789"
echo ${string} length is ${#string}

# 提取字符串
string="12345678910"
length=${#string}
echo ${string} of 1 to ${length} is ${string:1:${length}}

# 查找字符串中特定的字符首次出现的位置
string="123456789ksjcsgqgaldnaaljabckagdq"
expr index ${string} 1  # 输出结果是1……竟然不是0……
echo `expr index ${string} 1`  # 用反引号！！！！将命令的终端输出取出来
echo ${expr index ${string} 1} # 报错，看来美元符号只能用来取变量的值，不能去表达式的值！！！


# shell数组
# shell中，用括号表示数组，数组中的元素使用空格符号进行分隔
array_name=(1111 2222 3333 4444)
file_name[0]=1
file_name[1]=2
file_name[6]=2 # 可以不使用连续的下标，而且下标的范围没有限制

for name in ${array_name[@]}   # 千万不要忘记使用$取出变量名代表的数据！！！要不然直接当成一个字符串了！！！！
do
    echo ${name}
done

for name in ${file_name} # 使用@表示取出所有的元素，就只会表示取出第一个元素，和C++的数组指针有点相似的感觉
do
    echo ${name}
done

# 读取数组的元素，一般只通过${数组名[索引号]}的形式读取，如
array_name=(1111 2222 3333 4444)
echo ${array_name[0]} # 正常读取单个元素
echo ${array_name[@]} # 读取出所有的元素，显然元素直接是以空格分隔的
echo ${array_name} # 不写索引，就只读取第一个

# 获取数组元素的个数
echo ${#array_name[@]}

# 获取数组中单个元素的长度
echo ${#array_name[0]}

```
