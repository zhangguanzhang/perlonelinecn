
# 文件行距

## 1.双倍文件行距

    perl -pe '$\="\n"'

这行代码双倍文件的行距。这行代码中有三个地方我们要说明一下：“-p”,“-e”选项以及“$\”变量。
首先介绍“-e”选项。 “-e”选项用来在命令行的形式直接执行perl代码。通常用于执行一个很轻便程序的而懒得创建perl源文件的时候。通过用“-e”你就可以非常方便在命令执行代码。
接着说“-p”选项。“-p”选项用于假定perl代码运行于如下的一个循环环境中：

    while (<>) {
    # 此处为的perl代码
    } continue {
    print or die "-p failed: $!\n";
    }

这个结构遍历所有输入，执行你的代码并且输出“$_”的值。通过这种方式你可以有效的修改全部或者部分输入值。“$_”变量可以当成一个匿名变量，用来盛放好东东。
“$\”变量和awk中的ORS类似。附加在每一个打印操作后。如果没有任何的打印参数，则直接打印出“$_”的内容（好东东）。
在本例这行代码中“-e”执行的代码是 $\="\n"，代码等价于：

    while (<>) {
    $\ = "\n";
    } continue {
      print or die "-p failed: $!\n";
    }

实际执行的流程是，先读一行，并把它存储到“$_”中（包括当前行的换行符），“$\”被设置为换行符，并且调用打印函数。由于没有赋予参数，打印函数输出“$_”的内容，“$\”被附加到了最后。结果就是每一行无误修改的打印出来，同时后面跟着被赋值为换行符的“$\”。效果就是输入的文件被双倍行距了。

实际上不必要把每一行都设置“$\”为换行符。这可能最短的双倍文件行距的一行代码了。其他同样效果的代码有：

    perl -pe 'BEGIN { $\="\n" }'

这行代码是在Perl执行之前设置“$\”为换行符（BEGIN块是在任何代码执行之前先执行）。

    perl -pe '$_ .= "\n"'

这行代码相当于：

    while (<>) {

    $_ = $_ . "\n"

    } continue {

    print or die "-p failed: $!\n";

    }

代码附加一个额外的换行符给每一行，然后输出这行。

当然最简洁最酷的大概使用“s///”替换符了：

    perl -pe 's/$/\n/'

用换行符替换了正则表达式符号“$”，它匹配的是每一行的结束。这就相当于在行结尾增加了一个换行符。

##  2.双倍除了空行以外其他行的行距。

     perl -pe '$_ .= "\n" unless /^$/'

这个行代码双倍所有非空行行间距。它通过给每一个非空行添加一个换行符的方式实现。“unless”意思是“if not”。“unless /^$/”意思是“如果不是开始即结束的行”。条件“开始即结束的行”，实际上只有包含一个换行符的行。
把它扩展开来看的话：

    while (<>) {
    if ($_ !~ /^$/) {
    $_ .= "\n"
    }
    } continue {
    print or die "-p failed: $!\n";
    }

一个更好的例子是把包含空格和tab建的行都算在里面：

    perl -pe '$_ .= "\n" if /\S/'

这个代码每一行匹配“\S”. “\S”是一个正则表达式符，和“\s”的意思想反。“\s”是匹配所有的空字符。这样代码就是匹配至少含有一个非空字符（tab，竖直tab，空格，等等）的行，使其双倍行距。

## 3.三倍行距

    perl -pe '$\="\n\n"'

或者

   perl -pe '$_.="\n\n"'

代码和#1例子相似，除了在每一行最后添加两个换行符。

## 4.N倍行间距

   perl -pe '$_.="\n"x7'

这个行代码在每一个行后面插入7个换行符。我们注意到使用的是"\n" x 7来重复7次换行符的意思。“x”符和随后的N一起，表示重复N次的意思。
例如：

    perl -e 'print "foo"x5'

将会输出"foofoofoofoofoo"。

## 5.给每一行之前增加一个空白行。

    perl -pe 's//\n/'

这个行代码使用"s/pattern/replacement/"的替换符。他匹配“$_”变量中的第一个模式（正则表达式）并且用replacement替换掉。本例中模式为空，意味着匹配字符建的任何位置（本例中是第一个字符之前的位置），并且用换行符替换它。影响就是插入一个换行符在每行之前。

## 6.删除所有空行

   perl -ne 'print unless /^$/'

本行代码使用“-n”标志。-n”标志会假定perl在下循环的中执行你的代码：

    LINE:
    while (<>) {
    #你的代码在这执行
    }
    
其执行过程是：通过过“<>”读出每一行，并且把它存到变量“$_”中。这时你可以执行任何你想执行的。你可以在你主程序文本中指定他。

    LINE:
    while (<>) {
    print unless /^$/
    }

更进一步展开：

    LINE:
    while (<>) {
    print $_ unless $_ =~ /^$/
    }

这行代码输出所有非空行。其他形式地同功能代码：

    perl -lne 'print if length'


这行代码使用了“-l”命令行参数。“-l”自动chomp输入行（主要是删除了输入行最后的换行符）。接着测试行的长度。如果有任何字符，测试就为真，就打印出这行。

    perl -ne 'print if /\S/'

这个行代码功能和“print unless /^$/”大有不同，后者会打印出包含空格tabs的行，而前者不会。

## 7.删除连续的空行，只留下一行。

    perl -00 -pe ''

这代码很神奇对不？首先，他没有任何执行代码，-e执行部分为空。其次，它有一个看起来傻傻的“-00”命令行选项，这个选项开启大段落模式。一个段落是指两个换行符之间的文本。所有其他换行符将被忽略。段落被存储到“$_”中，“-p”选项吧他打印出来。
后来我有发现一个更短版本行代码。

    perl -00pe0

## 8.把所有的空行定制为固定N个连续空行。

    perl -00 -pe '$_.="\n"x4'

这个行代码综合了上一个代码（8）和#4的代码。他开启了大段落模式，接着添加（N-1）个换行符。在本例代码中，指定行间距为５倍行距（“\n”×4打印4次，加上本身自带一个段落换行符，总共5个）。