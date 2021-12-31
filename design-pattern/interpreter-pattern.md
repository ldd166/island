**解释器模式(Interpreter Pattern)——自定义语言的实现**

说明：设计模式系列文章是读`刘伟`所著`《设计模式的艺术之道(软件开发人员内功修炼之道)》`一书的阅读笔记。个人感觉这本书讲的不错，有兴趣推荐读一读。详细内容也可以看看此书作者的博客`https://blog.csdn.net/LoveLion/article/details/17517213`

## 模式概述

解释器模式是一种使用频率相对较低但学习难度较大的设计模式，它用于描述如何使用面向对象语言构成一个简单的语言解释器。

在某些情况下，为了更好地描述某一些特定类型的问题，我们可以创建一种新的语言，这种语言拥有自己的表达式和结构，即文法规则，这些问题的实例将对应为该语言中的句子。此时，可以使用解释器模式来设计这种新的语言。对解释器模式的学习能够加深我们对面向对象思想的理解，并且掌握编程语言中文法规则的解释过程。

### 模式定义

解释器模式定义如下：

> 解释器模式(Interpreter Pattern)：定义一个语言的文法，并且建立一个解释器来解释该语言中的句子，这里的“语言”是指使用规定格式和语法的代码。解释器模式是一种类行为型模式。

### 模式结构图

由于表达式可分为终结符表达式和非终结符表达式，因此解释器模式的结构与组合模式的结构有些类似，但在解释器模式中包含更多的组成元素，它的结构如下图所示：

![](https://img2020.cnblogs.com/blog/1546632/202111/1546632-20211110151111558-1489644000.png)

在解释器模式结构图中包含如下几个角色：

- AbstractExpression（抽象表达式）：在抽象表达式中声明了抽象的解释操作，它是所有终结符表达式和非终结符表达式的公共父类。
- TerminalExpression（终结符表达式）：终结符表达式是抽象表达式的子类，它实现了与文法中的终结符相关联的解释操作，在句子中的每一个终结符都是该类的一个实例。通常在一个解释器模式中只有少数几个终结符表达式类，它们的实例可以通过非终结符表达式组成较为复杂的句子。
- NonterminalExpression（非终结符表达式）：非终结符表达式也是抽象表达式的子类，它实现了文法中非终结符的解释操作，由于在非终结符表达式中可以包含终结符表达式，也可以继续包含非终结符表达式，因此其解释操作一般通过递归的方式来完成。
- Context（环境类）：环境类又称为上下文类，它用于存储解释器之外的一些全局信息，通常它临时存储了需要解释的语句。

### 模式伪代码

通常在解释器模式中提供了一个环境类`Context`，用于存储一些全局信息，通常在`Context`中包含了一个`HashMap`或`ArrayList`等类型的集合对象，存储一系列公共信息，用于在进行具体的解释操作时从中获取相关信息。其典型代码片段如下：

```java
import java.util.HashMap;
import java.util.Map;

public class Context {

    private Map<String, Object> map = new HashMap<>();

    public void assign(String key, Object value) {
        // 往环境类中设值
    }

    public Object lookup(String key) {
        // 获取存储在环境类中的值
        return null;
    }
}
```

在解释器模式中，每一种终结符和非终结符都有一个具体类与之对应，正因为使用类来表示每一条文法规则，所以系统将具有较好的灵活性和可扩展性。

对于所有的终结符和非终结符，我们首先需要抽象出一个公共父类，即抽象表达式类，其典型代码如下：

```java
public abstract class AbstractExpression {

    abstract void interpret(Context ctx);

}
```

终结符表达式和非终结符表达式类都是抽象表达式类的子类，对于终结符表达式，其代码很简单，主要是对终结符元素的处理，其典型代码如下所示：

```java
public class TerminalExpression extends AbstractExpression {
    @Override
    void interpret(Context ctx) {
        // 终结符表达式的解释操作
    }
}
```

对于非终结符表达式，其代码相对比较复杂，因为可以通过非终结符将表达式组合成更加复杂的结构，对于包含两个操作元素的非终结符表达式类，其典型代码如下：

```java
public class NonterminalExpression extends AbstractExpression {

    private AbstractExpression left;
    private AbstractExpression right;

    @Override
    void interpret(Context ctx) {
        // 递归调用每一个组成部分的interpret()方法
        // 在递归调用时指定组成部分的连接方式，即非终结符的功能
    }
}
```

## 模式实例

没有具体的例子总会让人感到抽象、难以理解。下面通过实例来体会解释器模式的应用。

### 加减法解释器

> 需求： 当输入字符串表达式为“1+2+3–4+1”时，将输出计算结果为3。

先来学习如何表示一个语言的文法规则以及如何构造一棵抽象语法树。

每一个输入表达式，例如“1+2+3–4+1”，都包含了三个语言单位，可以使用如下文法规则来定义：

```text
expression ::= value | operation

operation ::= expression '+' expression | expression '-'  expression

value ::= an integer //一个整数值
```

其中`|`表示或，如文法规则`boolValue ::= 0 | 1`表示终结符表达式`boolValue`的取值可以为`0`或者`1`。


上面文法规则包含三条语句，第一条表示表达式的组成方式，其中`value`和`operation`是后面两个语言单位的定义，每一条语句所定义的字符串如`operation`和`value`称为语言构造成分或语言单位，符号`::=`表示`定义为`的意思，其左边的语言单位通过右边来进行说明和定义，语言单位对应终结符表达式和非终结符表达式。

- 本规则中的`operation`是非终结符表达式，它的组成元素仍然可以是表达式，可以进一步分解;
- `value`是终结符表达式，它的组成元素是最基本的语言单位，不能再进行分解。

除了使用文法规则来定义一个语言，在解释器模式中还可以通过一种称之为抽象语法树(`Abstract Syntax Tree, AST`)的图形方式来直观地表示语言的构成，每一棵抽象语法树对应一个语言实例，如加法/减法表达式语言中的语句“1+2+3–4+1”，可以通过如下图所示抽象语法树来表示：

![](https://img2020.cnblogs.com/blog/1546632/202111/1546632-20211110164752865-1754398807.png)

在该抽象语法树中，可以通过终结符表达式`value`和非终结符表达式`operation`组成复杂的语句，每个文法规则的语言实例都可以表示为一个抽象语法树，即每一条具体的语句都可以用类似上图所示的抽象语法树来表示。

抽象语法树描述了如何构成一个复杂的句子，通过对抽象语法树的分析，可以识别出语言中的终结符类和非终结符类。

下面看代码实现。这里先不管输入的字符串表达式如何生成抽象语法树，重点看解释器模式用在哪里。

```java
/**
 * 抽象表达式
 */
public abstract class AbstractExpression {
    public abstract int interpret();
}

/**
 * 单值表达式
 */
public class Value extends AbstractExpression {

    private final String expr;

    public Value(String expr) {
        this.expr = expr;
    }

    @Override
    public int interpret() {
        return Integer.parseInt(expr);
    }
}

/**
 * 操作符号
 */
public enum Operator {
    /**
     * 加号(+)
     */
    PLUS,

    /**
     * 减号(-)
     */
    MINUS;
}

/**
 * 非单值表达式
 */
public class Expr extends AbstractExpression {

    private final AbstractExpression left;
    private final AbstractExpression right;
    private final Operator operator;

    public Expr(AbstractExpression left, AbstractExpression right, Operator operator) {
        this.left = left;
        this.right = right;
        this.operator = operator;
    }

    @Override
    public int interpret() {
        int leftValue = left.interpret();
        int rightValue = right.interpret();
        if (operator == Operator.PLUS) {
            return leftValue + rightValue;
        } else {
            return leftValue - rightValue;
        }
    }
}
```

测试一下“1+2+3–4+1”表达式，如下：

```java
AbstractExpression a = new Expr(new Value("1"), new Value("2"), Operator.PLUS);
AbstractExpression b = new Expr(a, new Value("3"), Operator.PLUS);
AbstractExpression c = new Expr(b, new Value("4"), Operator.MINUS);
AbstractExpression d = new Expr(c, new Value("1"), Operator.PLUS);

int result = d.interpret();
System.out.println(result);
```

这个过程中，能感受到`一切皆对象`的设计思想吗？

到这里，你可能要问了，如何把用户的输入生成抽象语法树呢，对于简单的规则可能用堆栈等常见数据结构就能轻松实现，但是复杂语句的解析可能就涉及到文法、词法分析、语法分析等编译原理相关知识。

当然也别怕，有成熟的工具已经帮我们搞定了生成抽象语法树的过程，有兴趣的朋友欢迎阅读本人精心整理的 [ANTLR专题](https://bytesfly.github.io/island/#/antlr4/introduction) 。


## 模式总结

解释器模式可以说是所有设计模式中难度较大、使用频率较低的一个模式。

但是在一些特殊场景与领域也是一把强大的利器。感兴趣可以阅读我整理的 [ANTLR专题](https://bytesfly.github.io/island/#/antlr4/introduction) ，可以加深对解释器模式的理解。

解释器模式为自定义语言的设计和实现提供了一种解决方案，它用于定义一组文法规则并通过这组文法规则来解释语言中的句子。

虽然解释器模式的使用频率不是特别高，但是它在正则表达式、XML文档解释等领域还是得到了广泛使用。

### 主要优点

- 易于改变和扩展文法。由于在解释器模式中使用类来表示语言的文法规则，因此可以通过继承等机制来改变或扩展文法。
- 实现文法较为容易。在抽象语法树中每一个表达式节点类的实现方式都是相似的，这些类的代码编写都不会特别复杂，还可以通过一些工具自动生成节点类代码。

### 主要缺点

- 对于复杂文法难以维护。在解释器模式中，每一条规则至少需要定义一个类，因此如果一个语言包含太多文法规则，类的个数将会急剧增加，导致系统难以管理和维护，此时可以考虑使用语法分析程序等方式来取代解释器模式。
- 执行效率较低。由于在解释器模式中使用了大量的循环和递归调用，因此在解释较为复杂的句子时其速度很慢，而且代码的调试过程也比较麻烦。

### 适用场景

- 可以将一个需要解释执行的语言中的句子表示为一个抽象语法树
- 一个语言的文法较为简单