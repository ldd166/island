
> [!WARNING]
> 本专题大多内容来源于我读《ANTLR 4权威指南》的随手笔记以及个人实践，仅供参考学习，`请勿用于任何商业用途，后果自负`，如涉及侵权或有错误之处，及时 [联系本人](https://bytesfly.github.io/blog/#/about/?id=%f0%9f%92%8c-%e8%81%94%e7%b3%bb) 。

下面学习如何编写语法。

## 如何定义语法规则

一种语言模式就是一种递归的语法结构。

我们需要从一系列有代表性的输入文件中归纳出一门语言的结构。在完成这样的归纳工作后，我们就可以正式使用`ANTLR`语法来表达这门语言了。

编写语法和编写软件很相似，差异在于我们处理的是语言规则，而非函数或者过程(`procedure`)。

在深入研究语法的细节之前，一件大有裨益的事情是：**讨论语法的整体结构以及如何建立初始的语法框架**。

语法由一个为该语法命名的头部定义和一系列可以相互引用的语言规则组成。

```text
grammar MyG;
rule1: <<stuff>>;
rule2: <<more stuff>>;
...
```

设计良好的语法反映了编程世界中的功能分解或者自顶向下的设计。这意味着我们对语言结构的辨识是从最粗的粒度开始，一直进行到最详细的层次，并把它们编写成为语法规则。所以，我们的第一个任务是找到最粗粒度的语言结构，将它作为我们的起始规则。在英语中，我们可以使用`sentence`规则作为起始规则。对于一个`XML`文件，我们可以使用`document`规则作为起始规则。

设计起始规则的内容实际上就是使用`英语伪代码`来描述输入文本的整体结构，这和我们编写软件的过程有点类似。

例如，一个CSV文件就是一系列以换行符为终止的行(`a comma-separated-value file is a sequence of rows terminated by newlines`)。其中，`is a`左侧的单词`file`就是规则名，右侧的全部内容就是规则定义中的`<<stuff>>`，即：

```text
file: <<sequence of rows that are terminated by newlines>>;
```

接着，我们描述起始规则右侧所指定的那些元素。它右侧的名词通常是词法符号或者尚未定义的规则。其中，词法符号是那些我们的大脑能够轻易识别出的单词、标点符号或者运算符。正如英语语句中的单词是最基本元素一样，词法符号是文法的基本元素。起始规则引用了其他的、需要进一步细化的语言结构， 如上面的例子中的“行”`row`。

一个行就是一系列由逗号分隔的字段（`a row is a sequence of fields separated by commas`）。接下来，一个字段就是一个数字或者字符串（`a field is a number or string`）。我们的伪代码如下所示：

```text
file: <<sequence of rows that are terminated by newlines>> ;
row: <<sequence of fields separated by commas>> ;
field: <<number or string>> ;
```

当我们完成对规则的定义后，我们的语法草稿就成形了。在刚开始的时候，辨识一条语法规则并使用伪代码编写右侧的内容是一项充满挑战的工作，不过，它会随着你为不同语言编写语法的过程变得越来越容易。


## 使用ANTLR语法表达语言

现在我们拥有了伪代码，还需要将它翻译为`ANTLR`标记，从而得到一个能够正常工作的语法。

常见的语言模式包括：序列(`sequence`)、选择(`choice`)、词法符号依赖(`token dependency`)，以及嵌套结构(`nested phrase`)。

在之前的 [快速上手](antlr4/quick-start.md) ，我们见过这些模式的一些例子。随着学习的深入，我们会用正式的语法规则将特定的模式表达出来，通过这种方式，我们就能够掌握基本的`ANTLR`标记的用法。

### 序列模式

登录一台POP服务器并获取第一条消息的指令序列为：

```text
USER parrt
PASS secret
RETR 1
```

其中指令`RETR 1`是由`RETR`关键字(保留字)，一个操作数和一个换行符构成。

使用`ANTLR`语法来表述这样的序列，只需要按照顺序将它们列出即可：

````antlrv4
retr : 'RETR' INT '\n' ; // 匹配“关键字－整数－换行符”序列
INT :   [0-9]+ ;
WS  :   [ \t]+ -> skip ;
````

注意，可以直接使用类似`'RETR'`的常量字符串来表示任意简单字符序列，诸如关键字或者标点符号等。

使用语法规则来为编程语言的特定结构命名，这就好像我们在编程时将若干个语句组合成一个函数。在上例中，我们将`RETR`命名为`retr`规则。这样，在语法的其他地方，就可以直接把规则名作为简称来引用`RETR`。

序列模式的变体包括：
- 带终止符的序列模式
- 带分隔符的序列模式

CSV文件同时使用了这两种模式。上面我们定义出了CSV文件的语法规则，下面用`ANTLR`语法来表达：

```antlrv4
file : (row '\n')* ; // 以换行符作为终止符的序列
row : field (',' field)* ; // 以','作为分隔符的序列
field: INT | STRING ;
```

简单解释一下：

- `file`规则使用带终止符的序列模式来匹配零个或多个`row'\n'`序列。其中序列中的每个元素都以`\n`字符结束；
- `row`规则使用带分隔符的序列模式来匹配一个`field`后面跟着零个或多个`'，'field`序列的情形。 `,`隔开了所有的`field`；

### 选择模式

在`ANTLR`的规则中，使用`|`符号表示`或者`的含义，称作备选分支(`alternatives`)。

比如上面CSV语法中用到的`field: INT | STRING ;`，表示字段可以是整数或者字符串。

上文 [快速上手的四则运算案例](antlr4/quick-start.md#使用Visitor构建计算器) 中，就用到了选择模式，如下：

```antlrv4
stat:   expr NEWLINE                # printExpr
    |   ID '=' expr NEWLINE         # assign
    |   NEWLINE                     # blank
    ;

expr:   expr op=('*'|'/') expr      # MulDiv
    |   expr op=('+'|'-') expr      # AddSub
    |   INT                         # int
    |   ID                          # id
    |   '(' expr ')'                # parens
    ;
```

也就是说，当语法规则中有允许这样也允许那样的含义时，就能使用`选择模式`。

### 词法符号依赖模式

如果在某个语句中看到了某个符号，就必须在同一个语句中找到和它配对的另外一个符号。为表达出这种语义，在语法中，我们使用一个序列来指明所有配对的符号，通常这些符号会把其他元素分组或者包裹起来。

上文 [快速上手的数组转换案例](antlr4/quick-start.md#使用Listener转换数组) 中，就用到了词法符号依赖模式，如下：

```antlrv4
/** A rule called init that matches comma-separated values between {...}. */
init  : '{' value (',' value)* '}' ;  // must match at least one value
```

### 嵌套模式

如果一条语法规则定义中的伪代码引用了它自身，就需要一条递归规则（自引用规则）。

上文 [快速上手的四则运算案例](antlr4/quick-start.md#使用Visitor构建计算器) 中，也用到了递归，如下：

```antlrv4
expr:   expr op=('*'|'/') expr      # MulDiv
    |   expr op=('+'|'-') expr      # AddSub
    |   INT                         # int
    |   ID                          # id
    |   '(' expr ')'                # parens
    ;
```

语言结构上的递归自然而然地使得语言规则发生了递归。

### 总结

<br/>

| 语言模式                                                     | 描述 |
| ------------------------------------------------------------ | -------- |
| 序列模式 | 它是一个有限长度或者任意长度的序列，序列中的元素可以是词法符号或者子规则。序列模式的例子包括变量声明（类型后面紧跟着标识符）和整数序列，例子：<br/> `retr : 'RETR' INT NEWLINE ; // 匹配“关键字－整数－换行符”序列`     |
| 带终止符的序列模式 | 它是一个任意长的、可能为空的序列，该序列由一个词法符号分隔开，通常是分号或者换行符，其中的元素可以是词法符号或者子规则。这样的例子包括类Java语言的语句集合和一些用换行符来分隔的数据格式。例子：<br/> `(statement ';')*  // Java的语句集合`  <br/> `(row NEWLINE)* // 多行数据`  | 
| 带分隔符的序列模式 | 它是一个任意长的、可能为空的序列，该序列由一个词法符号分隔开，通常是逗号、分号或是句号，其中的元素可以是词法符号或者子规则。这样的例子包括函数定义中的参数表、函数调用时传递的参数表、某些语句之间有分隔符却无终止符的编程语言，以及目录名。例子： <br/> `expr (',' expr)* // 函数调用时传递的参数` <br/> `( expr (',' expr)* )? // 函数调用时传递的参数是可选的` <br/> `'/'? name ('/' name)* // 简化的目录名` <br/> `stat ('.' stat)* // 若干个SmallTalk语句 `  |
| 选择模式 | 它是一组备选分支的集合。这样的例子包括不同种类的类型、语句、表达式或者`XML`标签。举例：<br/> `type : 'int' \| 'float' ;` <br/> `stat : ifstat \| whilestat \| 'return' expr ';' ;` <br/> `expr : '(' expr ')' \| INT \| ID ;` <br/> `tag : '<' Name attribute* '>' \| '<' '/'  Name '>' ;`    |
| 词法符号依赖模式 | 一个词法符号需要和一个或者多个后续词法符号匹配。这样的例子包括配对的圆括号、花括号、方括号和尖括号。例子：<br/> `'(' expr ')' // 嵌套表达式` <br/> `ID '[' expr ']' // 数组索引表达式` <br/> `'{' stat* '}' // 花括号包裹的若干个语句` <br/> `'<' ID (',' ID)* '>' // 泛型声明`    |
| 嵌套模式 | 它是一种自相似的语言结构。这样的例子包括表达式、Java的内部类、嵌套的代码块以及嵌套的Python函数定义。例子：<br/> `expr : '(' expr ')' \| ID ;` <br/> `classDef : 'class' ID '{' (classDef \| method \| field)*  '}' ;`    |

## 常见的词法结构

和语法分析器一样，词法分析器也使用规则来描述种类繁多的语言结构。在`ANTLR`中，我们使用的是几乎完全相同的标记。唯一的差别在于，语法分析器通过输入的词法符号流来识别特定的语言结构，而词法分析器通过输入的字符流来识别特定的语言结构。

由于词法规则和文法规则的结构相似，`ANTLR`允许二者在同一个语法文件中同时存在。不过，由于词法分析和语法分析是语言识别过程中的两个不同阶段，我们必须告诉`ANTLR`每条规则对应的阶段。它是通过这种方式完成的：

> 词法规则以大写字母开头，而文法规则以小写字母开头。

例如，`ID`是一个词法规则名，而`expr`是一个文法规则名。

对于关键字、运算符和标点符号，我们无须声明词法规则，只需要在文法规则中直接使用单引号将它们括起来即可，例如`'while'`、`'*'`，以及`'++'`。有些开发者更愿意使用类似`MULT`的词法规则来引用`'*'`，以避免对其的直接使用。这样，在改变乘法运算符的时候，只需修改`MULT`规则，而无须逐个修改引用了`MULT`的文法规则。

下面看下常见的词法结构。


| 词法符号类型       | 举例 |
| ----------------| -------- |
| 匹配标识符 | `ID : [a-zA-Z]+ ; // 匹配1个或多个大小写字母` |
| 匹配数字 | `INT : [0-9]+ ; // 匹配1个或多个数字`    |
| 匹配字符串常量 | `STRING : '"' .*? '"' ; // 匹配两个双引号之间的任意字符序列`  |
| 匹配注释和空白字符 | `WS : [ \t\r\n]+ -> skip ; // 匹配一个或多个空白字符并将它们丢弃`  |

### 匹配标识符

```antlrv4
INT : '0'..'9'+ ; // 匹配1个或者多个数字

ID : ('a'..'z'|'A'..'Z')+ ; // 匹配1个或多个大小写字母
```
这个让我们感到新鲜的是范围运算符：'a'..'z'，它的意思是从a到z的所有字符。

类似`ID`的规则有时候会和其他词法规则或者字符串常量值产生冲突，例如`if`、`for`、`while`等关键字。

```antlrv4
grammar KeywordTest;

rule : IF | FOR | WHILE | ID ;

IF : 'if' ;
FOR : 'for' ;
WHILE : 'while' ;

ID : [a-zA-Z]+ ; // 不会匹配 if, for, while
```

`ANTLR`对混合了词法规则和文法规则的语法文件的处理机制：

> 首先，`ANTLR`从文法规则中筛选出所有的字符串常量，并将它们和词法规则放在一起。字符串常量被隐式定义为词法规则，然后放置在文法规则之后、显式定义的词法规则之前。`ANTLR`词法分析器解决歧义问题的方法是优先使用位置靠前的词法规则。这意味着，`ID`规则必须定义在所有的关键字规则之后。`ANTLR`将为字符串常量隐式生成的词法规则放在显式定义的词法规则之前，所以它们总是拥有最高的优先级。

### 匹配数字

定义一个简化版的浮点数：

> 一个浮点数以一列数字为开头，后面跟着一个点，然后是可选的小数部分；浮点数的另外一种格式是，以点为开头，后面是一列数字。一个单独的点不是一个合法的浮点数定义。

```antlrv4
FLOAT: DIGIT+ '.' DIGIT*  // 匹配 10., 3.14等
     | '.' DIGIT+         // 匹配 .618等
     ;

fragment
DIGIT : [0-9] ;  // 匹配单个数字
```

这里使用了一条辅助规则`DIGIT`，这样就不用重复书写`[0-9]`了。将一条规则声明为`fragment`可以告诉`ANTLR`，该规则本身不是一个词法符号，它只会被其他的词法规则使用。这意味着我们不能在文法规则中引用`DIGIT`。

### 匹配字符串常量

```antlrv4
STRING : '"' .*? '"' ; // 匹配两个双引号之间的任意字符序列
```
其中，点号通配符匹配任意的单个字符。因此，`.*`就是一个循环，它匹配零个或多个字符组成的任意字符序列。显然，它可以一直匹配到文件结束，但这没有任何意义。

为解决这个问题，`ANTLR`通过标准正则表达式的标记（`？`后缀）提供了对非贪婪匹配子规则（`nongreedy subrule`）的支持。

非贪婪匹配的基本含义是:
> 获取一些字符，直到发现匹配后续子规则的字符为止。

更准确的描述是，在保证整个父规则完成匹配的前提下，非贪婪的子规则匹配数量最少的字符。

`.*`是贪婪的，因为它贪婪地消费掉一切匹配的字符。

这样的`STRING`规则还不够完善，因为它不允许其中出现双引号。为了解决这个问题，很多语言都定义了以`\`开头的转义序列。

在这些语言中，如果希望在一个被双引号包围的字符串中使用双引号，我们就需要使用`\"`。下面规则能够支持常见的转义字符：

```antlrv4
STRING: '"' (ESC|.)*? '"' ;
fragment
ESC: '\\"' | '\\\\' ; // 双字符序号\"和\\
```
`ANTLR`语法本身需要对转义字符`\`进行转义，因此我们需要`\\`来表示单个反斜杠字符。

### 匹配注释和空白字符

当词法分析器匹配到我们刚刚定义过的那些词法符号的时候，它会将匹配到的词法符号放入词法符号流，输送给语法分析器。之后，由语法分析器来检查词法符号流的语法结构。

但是，当词法分析器匹配到注释和空白字符的时候，我们通常希望将它们丢弃。这样，语法分析器就不必处理注释和空白字符了。

定义需要被丢弃的词法符号的方法和定义正常的词法符号的方法一样。我们只需要使用`skip`指令通知词法分析器将它们丢弃即可。

例如，下面匹配类`Java`语言中的单行和多行注释：

```antlrv4
LINE_COMMENT : '//' .*? '\r'? '\n' -> skip ; // 匹配单行注释
COMMENT : '/*' .*? '*/' -> skip ; // 匹配多行注释
```

词法分析器可以接受许多种位于`->`操作符之后的指令，`skip`只是其中之一。例如，我们能够使用`channel`指令将某些词法符号放入一个`隐藏的通道`并输送给语法分析器。

大多数编程语言将空白字符看作词法符号间的分隔符，并将它们忽略（`Python`是一个例外，它使用空白字符来达到某些语法上的目的：换行符代表一条命令的终止，特定数量的缩进指明嵌套的层级）。

下列规则告诉`ANTLR`丢弃空白字符：

```antlrv4
WS : [ \t\r\n]+ -> skip ; // 匹配一个或多个空白字符并将它们丢弃
```

有了上面这些语法设计的基础，就能动手写写`ANTLR`的案例了，更多代码见： [https://github.com/bytesfly/antlr-demo](https://github.com/bytesfly/antlr-demo)

后续文章将更多专注于`ANTLR`的实战与应用。