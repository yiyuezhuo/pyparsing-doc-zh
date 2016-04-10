<img src="image/icon.png">

# Pyparsing 导引

Paul McGuire 著

yiyuezhuo 译

---

Copyright C 2008 O'Reilly Media, Inc.

ISBN:9780596514235

Released: October 4, 2007

---


目录

* [Pyparsing是什么?](#what)
* [Pyparsing程序的简单形式](#basic)
* ["Hello World"](#hello)
* [什么使Pyparsing变得不同？](diff)
* [从表中解析数据-使用Parse Action和ParseResults](#table)
* [从网页解析数据](#page)
* [一个简单的S表达式解析器](#SS)
* [一个复杂的S表达式解析器](#CS)
* [解析搜索字符串](#search)
* [100行代码以内的搜索引擎](#engine)
* [结论](#conclusion)
* [索引](#index)


> "我需要解析这个日志文件..."
>
> "只是要从网页中提取数据..."
>
> "我们需要一个简单的命令行解释器..."
>
> "我们的源代码需要移植到新API集上..."

这些工作要求每天都让开发者们条件反射般的骂娘"擦，又要写一个解析器！"

解析不十分严格格式的数据形式的任务经常出现在开发者面前。有时其是一次性的，像内部使用的API升级程序。
其他时候，解析程序作为在命令行驱动的程序中的内建函数。

如果你在Python中编程，你可以简化这些工作，通过使用Python的内建字符串方法，比如split(),index()以及startwith().

让这项工作又变得讨厌的是我们经常不只是对字符串分割和索引，对于一些复杂的语法定义来说。比如:

```
y = 2 * x + 10   
```

它每个符号间都有空分隔,是容易解析的，对于这种空格分离的形式。不幸的是，很少有用户会如此这般使用空格，算术表达式经常像这样写出:

```
y = 2*x + 10
y = 2*x+10
y=2*x+10
```

直接对最后一个字符串运用`str.split`方法会导致返回原字符串（作为一个列表的唯一实例），而不会分离出这些单独的元素`y`,`=2`,等等.

处理这种超越str.split的解析任务的工具是正则表达式或lex/yacc。正则表达式用一个字符串去描述文本模式以便匹配。
那个字符串使用特殊符号(像`|`,`+`,`.`,`*`,`?`)去表示不同的解析概念像alternation(多选),repetition(重复)
以及wildcards(通配符).Lex/yacc是则先拆出标记，然后应用处理代码到提取出的标记上。Lex/yacc
使用一个单独的标记定义文件，然后产生lex中间文件以及处理过程代码模板给程序员扩展，以定制不同的应用行为。

>*历史注释*
>
>这些文本处理技术最早在1970年以C实现，现在它们仍在广大的领域发挥作用。内置电池的Python拥有标准库
>`re`来提供对正则表达式的支持。你还可以下到一些免费的lex/yacc风格的解析器模块，其提供了对python的接口。


这些传统工具的主要问题在于它们独特的标记系统需要被精确映射到Python的代码上。比如lex/yacc风格工具往往要单独进行一个代码生成阶段。

实践中，解析器编写看起来陷入到一个怪圈中:写代码，解析示例文本，找到额外的特殊情况等等。
组合正则表达式符号，额外的代码生成步骤，很可能使这个循环过程可能会不断的陷入挫折。

## <a name="whatsthis">Pyparsing是什么?

Pyparsing是纯python的，易于使用。Pyparsing提供了一系列类让你可以以单独的表达式元素开始来构建解析器。
其表达式使用直觉的符号组合，如`+`表示将一个表达式加到另一个后面。`|`,`^`表示解析多选
(意为匹配第一个或匹配最长的).表达式的重复可以以类的形式表示，如`OneOrMore`,`ZeroOrMore`,`Optional`.

作为例子，一个正则表达式处理IP地址后面跟着一个美式电话号码的情况需要这样写:

```
(\d{1,3}(?:\.\d{1,3}){3})\s+(\(\d{3}\)\d{3}-\d{4})
```

对比一下，类似的表达式用pyparsing写是这个样子

```python
ipField = Word(nums, max=3)
ipAddr = Combine( ipField + "." + ipField + "." + ipField + "." + ipField )
phoneNum = Combine( "(" + Word(nums, exact=3) + ")" + Word(nums, exact=3) + "?" + Word(nums, exact=4) )
userdata = ipAddr + phoneNum
```
尽管更长，但pyparsing版本更易读，也更容易被回朔和更新，比如可以更容易从此移植去处理其他国家的电话号码格式，

>Python新手？
>
>我已经收到很多邮件，他们告诉我使用pyparsing也是他们第一次使用python编程。他们发现pyparsing易于学习，
>容易改写内部的例子完成应用。如果你是刚开始使用python，你可能对于阅读这些例子感到一点困难。
>Pyparsing的使用并不要求任何高级的python知识。对于它所需要的那一部分，有一些网络教程资源，比如python的[官网]
(>www.python.org).
>
>为了更好的使用pyparsing,你应当更熟悉python的语言特性，如缩进语法，数据类型，
>以及`for item in itemSequence` 式循环控制语法。
>Pyparsing使用object.attribute式标记，就像python的内建容器类，元组，表以及字典。
>
>这本书的例子使用了python的lambda表达式，本质上就是单行函数；lambda表达式对于定义简单的解析操作特别有用。
>
>列表解析和生成器表达式的知识是有用的，它们可以用在在解析标记结果的截断，但这并不是必须的。


Pyparsing是：

* 100%纯python,没有的动态链接库(DLLs)或者共享库包含其中，所以你可以在python2.3能够通过编译的任何地方使用它。
* 解析表达式使用标准的python类标记和符号表示。没有单独的代码生成过程也没有特殊符号和标记，这将使得你的应用易于开发，理解和维护。
* 对于常见的模式准备了辅助方法:
 * C,C++,Java,Python,HTML注释
 * 引号字符串(使用单个或双引号，除了\',\''转义情况外)
 * HTML与XML标签(包含上下级以及属性操作)
 * 逗号分隔以及被限制的列表表达式
* 轻量级封装-Pyparsing的代码包含在单个python文件中，容易放进site-packages目录下，或者被你的应用直接包含。
* 宽松的许可证，MIT许可证使得你可以随意进行非商用或商业应用。

## <a name="basic">Pyparsing程序的简单形式

典型的pyparsing程序具有以下结构:
* import pyparsing模块
* 使用pyparsing类和帮助方法定义语法
* 使用语法解析输入文本
* 处理文本解析的结果


### 从Pyparsing出导入名字

通常，使用`from pyparsing import *`是不被python风格专家鼓励的。因为它污染了本地变量命名空间，
因其从不明确的模块中引入的不知道数量的名字。无论如何，在pyparsing开发工作中，很难想象pyparsing定义的名字会被使用，
而且这样写简化了早期的语法开发。在语法最终完成后，你可以回到传统风格的引用，或专门from导入你需要的那些名字。

### 定义语法
语法是你的定义的文本模式，这个模式被应用于输入文本提取信息。在pyparsing中，语法由一个或多个Python语句构成，而模式的组合则使用pyparsing的类和辅助对象去指定组合的元素。Pyparsing允许你使用像`+`,`|`,`^`这样的操作符来简化代码。作为例子，假如我使用pyparsing的`Word`类去定义一个典型的程序变量名字，其由字母符号或字母数字或下划线构成。我将以Python语句这样描述:

```python
identifier = Word(alphas,alphanums+'_')
```

我也想解析常数，如整数和浮点数。另一个简单的定义的Word对象，它应当包含数字，也许还包含小数点。

```python
number= Word(nums+'.')
```

从这里，我然后定义一个简单的赋值语句像这样:

```python
assignmentExpr = identifier + "=" +(identifier|number)
```

现在我们可以解析像这样的内容了:

```
a = 10
a_2=100
pi=3.14159
goldenRatio = 1.61803
E =mc2
```

在程序的这个部分，你可以附加任何解析时回调函数(或称为解析动作parse actions)或为语法定义名字去减轻之后指派它们的工作。
解析动作是非常有力的特性对于pyparsing，之后我们将论述它的细节，

> 实践:BNF范式初步
> 在写python代码实现语法之前，将其先写在纸上是有益的，如:
> * 帮助你澄清你的想法
> * 指导你设计解析器
> * 提前演算，就像你在执行你的解析器
> * 帮助你知道设计的界限
> 幸运的是，在设计解析器过程中，有一个简单的符号系统用来描绘解析器，
>它被称为BNF(Backus-Naur Form)范式.你可以在这里获得BNF的[好例子](http://en.wikipedia.org/wiki/backus-naur_form) >。你并不需要十分严格的遵循它，只要它能刻画你的语法想法即可。
>
> 在这本书里我们用到了这些BNF记号:
> * `::=` 表示"被定义为"
> * `+` 表示“一个或更多”
> * `*` 表示“零个或更多”
> * 被[]包围的项是可选的
> * 连续的项序列表示被匹配的标记必须在序列中出现
> * `|` 表示两个项之一会被匹配

### 使用语法解析输入文本

在早期版本的pyparsing中，这一步被限制为使用`parseString`方法，像这样:

```python
assignmentTokens = assignmentExpr.parseString("pi=3.14159")
```

来得到被匹配的标记。

现在你可以使用更多的方法，全部列举如下:

* `parseString` 应用语法到给定的输入文本（从定义上看，如果这个文本可以应用多次规则也只会运用到第一次上）
* `scanString` 这是个生成器函数，给定文本和上界下界，其会试图返回所有解析结果
* `searchString` scanString的简单封装，返回你给定文本的全部解析结果，放在一个列表中。
* `transformString` scanString的另一个封装，还附带了替换操作。

现在，让我们继续研究`parseString`,稍后我将给你们展示其他选择的更多细节。

### 处理解析后的文本

当然，如何处理解析文本得到的返回值是最重要的。在大多数解析工具中，通常会返回一个匹配到的标记的列表供未来进一步解释使用。
Pyparsing则返回一个更强的对象，被称为ParseResults.在最简单的形式中，ParseResults可以被打印和连接像python列表一样。
作为例子，继续我们赋值表达式的例子，下面的代码:

```python
assignmentTokens = assignmentExpr.parseString("pi=3.14159")
print assignmentTokens
```

会打印出

```
['pi','=','3.14159']
```

但是ParseResults也支持解析文本中的个域(individual fields),如果语法为返回值的某些成分指派了名字。

这里我们通过给表达式里的元素取名字加强它们(左项成为lhs,右项称为rhs)，我们就能在ParseResults里连接这些域，
就像它们是返回的对象的属性一样。

```python
assignmentExpr = identifier.setResultsName("lhs") + "=" + \
(identifier | number).setResultsName("rhs")
assignmentTokens = assignmentExpr.parseString( "pi=3.14159" )
print assignmentTokens.rhs, "is assigned to", assignmentTokens.lhs
```

将打印出

```
3.14159 is assigned to pi
```

现在介绍进入转入细节部分了，让我们看一些例子

## <a name="hello">Hello,World

Pyparsing有很多例子，其中有一个简单地"Hello World"解析器。这个简单的例子也被
[O'Reilly,ONLamp.com](http://onlamp.com)的文章
[用Python建立递归下降解析器](Building_Recursive_Descent_Parsers_with_Python_en.md)所使用.
在这一节，我也使用类似的例子以介绍简单的pyparsing解析工具。

当前"Hello,World!"的解析模式被限制为:

```
word, word !
```

这过于受限了，让我们扩展语法以适应更多的情况。比如说应当可以解析以下情况:

```
Hello, World!
Hi, Mom!
Good morning, Miss Crabtree!
Yo, Adrian!
Whattup, G?
How's it goin', Dude?
Hey, Jude!
Goodbye, Mr. Chips!
```

写一个这样的解析器的第一步是分析这些文本的抽象模式。像我们之前推荐的那样，让我们用BNF范式来表达。
用自然语言表达这个意思，我们可以说:"一个这样的句子由一个或多个词(作为问候词)，后跟一个逗号，
后跟一个或多个附加词(作为问候的对象)"，结尾则使用一个感叹号或问好。用BNF可以这样表达:

```
greeting ::= salutation comma greetee endpunc
salutation ::= word+
comma ::= ,
greetee ::= word+
word ::= a collection of one or more characters, which are any alpha or ' or .
endpunc ::= ! | ?
```

这个BNF几乎可以直译为pyparsing的语言，通过使用pyparsing的`Word`,`Literal`,`OneOrMore`以及辅助方法`oneOf`。
(BNF与pyparsing的一个区别在于BNF喜欢使用传统的由上自下的语法定义，pyparsing则使用由底至上的方式。
因为我们要保证我们使用的元素在上面已经定义过了)

```python
word = Word(alphas+"'.")
salutation = OneOrMore(word)
comma = Literal(",")
greetee = OneOrMore(word)
endpunc = oneOf("! ?")
greeting = salutation + comma + greetee + endpunc
```

`oneOf`使定义更容易，比较两种等价写法

```python
endpunc = oneOf("! ?")
endpunc = Literal("!") | Literal("?")
```

`oneOf`也可以直接传入由字符串构成的列表，直接传字符串也是先以空格分离成那样的列表的

使用我们的解析器解析那些简单字符串可以得到这样的结果。

```python
['Hello', ',', 'World', '!']
['Hi', ',', 'Mom', '!']
['Good', 'morning', ',', 'Miss', 'Crabtree', '!']
['Yo', ',', 'Adrian', '!']
['Whattup', ',', 'G', '?']
["How's", 'it', "goin'", ',', 'Dude', '?']
['Hey', ',', 'Jude', '!']
['Goodbye', ',', 'Mr.', 'Chips', '!']
```

每个东西都被很好的解析了出来。但是我们的结果缺乏结构。对这个解析器而言，如果我们想要提取出句子的左边部分-即问候部分，
我们还需要做一些工作，迭代结果直到我们碰上了逗号:

```python
for t in tests:
  results = greeting.parseString(t)
  salutation = []
  for token in results:
    if token == ",":
      break
    salutation.append(token)
  print salutation
```

很好!我们应该已经实现了一个不错的字符-字符的扫描器。幸运的是，我们的解析器可以足够智能以避免之后繁琐工作。

当我们直到问候及问候对象是不同的逻辑部分之后，我们可以使用pyparsing的Group类来为返回结果赋予更多的结构。
我们修改salutation和greetee为

```python
salutation = Group( OneOrMore(word) )
greetee = Group( OneOrMore(word) )
```

于是我们的结果看起来更有组织性了:

```
［'Hello'], ',', ['World'], '!']
［'Hi'], ',', ['Mom'], '!']
［'Good', 'morning'], ',', ['Miss', 'Crabtree'], '!']
［'Yo'], ',', ['Adrian'], '!']
［'Whattup'], ',', ['G'], '?']
［"How's", 'it', "goin'"], ',', ['Dude'], '?']
［'Hey'], ',', ['Jude'], '!']
［'Goodbye'], ',', ['Mr.', 'Chips'], '!']
```

然后我们可以使用简单的列表拆包实现不同部分赋值:

```python
for t in tests:
  salutation, dummy, greetee, endpunc = greeting.parseString(t)
  print salutation, greetee, endpunc
```

会打印出:

```
['Hello'] ['World'] !
['Hi'] ['Mom'] !
['Good', 'morning'] ['Miss', 'Crabtree'] !
['Yo'] ['Adrian'] !
['Whattup'] ['G'] ?
["How's", 'it', "goin'"] ['Dude'] ?
['Hey'] ['Jude'] !
['Goodbye'] ['Mr.', 'Chips'] !
```

注意我们用dummy变量记入了解析出的逗号。这些逗号在解析中是很有用的，比如让我们分隔问候部分和问候对象部分。
但在结果中我们对逗号不感兴趣，它应当从结果中消失。你可以使用`Suppress`对象包住逗号定义以抑制其出现。

```python
comma = Suppress( Literal(",") )
```

你可以以不同的等价方式表达以上语句

```python
comma = Suppress( Literal(",") )
comma = Literal(",").suppress()
comma = Suppress(",")
```

使用以上形式之一，我们解析出的结果变成这个样子:

```
［'Hello'], ['World'], '!']
［'Hi'], ['Mom'], '!']
［'Good', 'morning'], ['Miss', 'Crabtree'], '!']
［'Yo'], ['Adrian'], '!']
［'Whattup'], ['G'], '?']
［"How's", 'it', "goin'"], ['Dude'], '?']
［'Hey'], ['Jude'], '!']
［'Goodbye'], ['Mr.', 'Chips'], '!']
```

所以现在结果控制代码可以丢掉dummy变量了，只需:

```python
for t in tests:
  salutation, greetee, endpunc = greeting.parseString(t)
```

现在我们有了一个不错的解析器并可以处理它的返回结果。让我们开始处理测试数据，首先，让我们将问候以及问候对象加进它们的表里：

```python
salutes = []
greetees = []
for t in tests:
  salutation, greetee, endpunc = greeting.parseString(t)
  salutes.append( ( " ".join(salutation), endpunc) )
  greetees.append( " ".join(greetee) )
```

我们还有其他一些小变化:

* 使用`" ".join(list)`去将拆出来的标记列表转回简单的字符串
* 保存问候语与行末的符号来区分问候是How are you?这种疑问句还是Hello!感叹句所表示的。

现在我们收集一些名字和问候语，我们可以使用它们产生一些新的句子:

```python
for i in range(50):
  salute = random.choice( salutes )
  greetee = random.choice( greetees )
  print "%s, %s%s" % ( salute[0], greetee, salute[1] )
```

现在我们可以看到全新的问候了:

```
Hello, Miss Crabtree!
How's it goin', G?
Yo, Mr. Chips!
Whattup, World?
Good morning, Mr. Chips!
Goodbye, Jude!
Good morning, Miss Crabtree!
Hello, G!
Hey, Dude!
How's it goin', World?
Good morning, Mom!
How's it goin', Adrian?
Yo, G!
Hey, Adrian!
Hi, Mom!
Hello, Mr. Chips!
Hey, G!
Whattup, Mr. Chips?
Whattup, Miss Crabtree?
...
```

我们也可以模拟一些介绍通过以下代码:

```python
for i in range(50):
  print '%s, say "%s" to %s.' % ( random.choice( greetees ),"".join( random.choice( salutes ) ),random.choice( greetees ) )
```

看起来像这样!

```
Jude, say "Good morning!" to Mom.
G, say "Yo!" to Miss Crabtree.
Jude, say "Goodbye!" to World.
Adrian, say "Whattup?" to World.
Mom, say "Hello!" to Dude.
Mr. Chips, say "Good morning!" to Miss Crabtree.
Miss Crabtree, say "Hi!" to Adrian.
Adrian, say "Hey!" to Mr. Chips.
Mr. Chips, say "How's it goin'?" to Mom.
G, say "Whattup?" to Mom.
Dude, say "Hello!" to World.
Miss Crabtree, say "Goodbye!" to Miss Crabtree.
Dude, say "Hi!" to Mr. Chips.
G, say "Yo!" to Mr. Chips.
World, say "Hey!" to Mr. Chips.
G, say "Hey!" to Adrian.
Adrian, say "Good morning!" to G.
Adrian, say "Hello!" to Mom.
World, say "Good morning!" to Miss Crabtree.
Miss Crabtree, say "Yo!" to G.
...
```

好了，我们已经见识了pyparsing模块。通过使用一些极其简单的pyparsing类和方法，我们就达成了十分强大的表达能力。

## <a name="diff">什么使得Pyparsing显得不同？

Pyparsing被设计为满足一些特别的目标，其中包括语法必须易写易理解而且能够很容易修改一个解析器去适应新的需求。
这些目标在于极简化解析器设计任务使pyparsing用户能聚焦于解析而不是在解析库与元语法之间挣扎，下面是pyparsing之禅.

### 语法规则的编写应是自然易读的python程序，而且其形式为python程序员所熟悉。

Pyparsing以以下方式实现该目标

* 使用操作符组合解析器要素。python支持操作符重载，利用它我们可以超越常规的对象式语法结构，
  使得我们的解析器表达式更易读。

  比如说:
  ```python
  streetAddress = And( [streetNumber, name,Or( [Literal("Rd."), Literal("St.")] ) ] )
  ```
  可以被写成
  ```python
  streetAddress = streetNumber + name + ( Literal("Rd.") | Literal("St.") )
  ```
* 很多pyparsing的属性设置方法会返回调用对象本身，所以一些属性调用可以组成一个链。作为例子，
  下面的解析器定义了interger，并为其指定了名字和解析动作,解析动作将字符串转回python整数。使用上述特性，可以将
  ```python
  integer = Word(nums)
  integer.Name = "integer"
  integer.ParseAction = lambda t: int(t[0])
  ```
  写为
  ```python
  integer = Word(nums).setName("integer").setParseAction(lambda t:int(t[0]))
  ```

### 类名比特殊符号好读并且好理解

这可能是pyparsing与正则表达式或类似的工具之间最明显的区别了。简介处的IP地址-电话号码例子就展现了这一点。
但正则表达式在它自己的控制符号出现在它想模式匹配的文本中时，会陷入理解上的真正困境。
结果我们会写出一个转义斜杠的大杂烩。这里有一个正则表达式试图匹配C式函数，其可以有一个或多个参数，其由单词或整数构成:

```
(\w+)\((((\d+|\w+)(,(\d+|\w+))*)?)\)
```

括号是控制字符还是要匹配的字符并不是一目了然的，而如果输入文本包括`\`,`.`,`*`或`?`情况将变得更糟糕。而pyparsing版本描述类似的表达式则为:

```python
Word(alphas)+ "(" + Group( Optional(Word(nums)|Word(alphas) + ZeroOrMore("," + Word(nums)|Word(alphas))) ) + ")"
```

这当然更好读。由于`x + ZeroOrMore(","+x)`的形式过于普遍，我们有一个pyparsing的辅助方法，`delimitedList`方法，它等价于这个表达式。通过使用delimitedList，我们的pyparsing写法可以更简化为:

```python
Word(alphas)+ "(" + Group( Optional(delimitedList(Word(nums)|Word(alphas))) ) + ")"
```

### 语法定义中的空格的扰乱

对于"特殊符号不特殊"问题，正则表达式必须明确声明空格在输入文本的位置。在C函数例子中，正则表达式应当匹配到

```
abc(1,2,def,5)
```

而不是

```
abc(1, 2, def, 5)
```

不幸的是，声明可选空格出现或不出现并不容易，其结果是\s*的表达形式从头到尾到处都是，使我们所要匹配的目标更加模糊：

```
(\w+)\s*\(\s*(((\d+|\w+)(\s*,\s*(\d+|\w+))*)?)\s*\)
```

对应的，pyparsing无视两个要素之间的空格，所以对应的表达式还是:

```python
Word(alphas)+ "(" + Group( Optional(delimitedList(Word(nums)|Word(alphas))) ) + ")"
```

而并不需要对空格多说什么。

类似的概念也可以应用到注释上，其可能会出现在代码的任何地方(不是那种只能在任一行最后的那种)。想象注释像空格一样出现在参数之间，正则表达式会变成什么样子，不过在pyparsing中，只需要这些代码就可以解决。

```python
cFunction = Word(alphas)+ "(" + Group( Optional(delimitedList(Word(nums)|Word(alphas))) ) + ")"
cFunction.ignore( cStyleComment )
```

### 结果应当比单纯的列表形式多更多东西

Pyparsing的解析结果是ParseResult类的实例。其可以被当成列表操作(使用[],len,iter或分片等)。
但它也可以编程嵌套(nested)形式，字典风格的以及对象风格(点字段调用风格)。C函数例子中的解析结果一般看作:

```
['abc', '(', ['1', '2', 'def', '5'], ')']
```

你可以看到参数被放进了一个子列表中，这使得进一步解析更容易。而如果语法定义中给结果中的某些结构命了名，
则我们可以使用字段引用法取代索引值法来减少出错概率。

这些高级引用技术在处理更复杂的语法问题时是至关重要的。

### 在解析时间就执行的预处理

当解析时，解析器对输入文本进行很多检查:测试不同的字符串或匹配一些模式，如两个引号间的字符串。
如果匹配到一个字符串，则立即解析(post-parsing)代码可以执行一个变换，将其转为python整数类型或字符串类型对象。

pyparsing支持在解析时调用的回调函数(成为解析行为)你可以附加一个单独的表达式给语法。
解析器会在匹配到对应形式时调用这些函数。作为例子，从文本中提取双引号围成的字符串，一个简单地解析行为会移除掉引号。像这样:

```python
quotedString.setParseAction( lambda t: t[0][1:-1] )
```

就够了。不需要检查开头结尾是不是引号.这个函数只有匹配到这样的形式时才会调用。

解析行为也可以用来执行不同的检查，像测试匹配到的词是不是一个给定列表中的词，并在不是时抛出一个`ParseException`。
解析行为也可以返回一个列表或一个对象，如将输入文本编译为一系列可执行或能调用的对象。解析行为在pyparsing中是个有力的工具。

### 语法必须对改变具有更强的适应性和健壮性

解析器中最普遍的死亡陷进在你编写时很难躲开。一个简单的模式匹配执行器可能日渐变得更复杂和笨拙。
吐过输入文本的数据不能被匹配却被需要被匹配，于是解析器需要加上一个补丁以解决这一新变化。或者去修改它的语言规则。
在这一切发生后一段时间，补丁开始干扰早先的语法定义，而之后越来越多的补丁使问题变得越来越困难。
当一个修改在你最后一次修改几个月发生时，你去重新理解它花去的时间会超出你的想象，这都增加了困难。

pyparsing也没有解决这一问题，但是其单独定义的语法风格和结构使问题得以缓解-单独的要素容易找到也易读
，易于修改和扩展。这是pyparsing的使用者们发给我的一句话：“我可以只写一个自定义方法，
但是我过去的经验反应一旦我创建一个pyparsing语法，它就会自动变得有组织而且容易维护和扩展。”

## <a name="table">从表格文件中解析数据-使用解析行为和ParseResults

作为我们第一个例子，让我们看一个处理给定橄榄球比赛文件信息文件的程序。
该文件每一行记录一场比赛的信息，其中包括时间，比赛双方和他们的比分。

```
09/04/2004 Virginia 44 Temple 14
09/04/2004 LSU 22 Oregon State 21
09/09/2004 Troy State 24 Missouri 14
01/02/2003 Florida State 103 University of Miami 2
```

这些数据的BNF形式是简洁明了的

```
digit ::= '0'..'9'
alpha ::= 'A'..'Z' 'a'..'z'
date ::= digit+ '/' digit+ '/' digit+
schoolName ::= ( alpha+ )+
score ::= digit+
schoolAndScore ::= schoolName score
gameResult ::= date schoolAndScore schoolAndScore
```

我们以覆盖BNF定义来开始我们的解析器构建工作。就像扩展"Hello，World！"程序一样，我们将先设计好构建块，
然后再将它们组合起来构成更复杂的语法。

```python
# nums and alphas are already defined by pyparsing
num = Word(nums)
date = num + "/" + num + "/" + num
schoolName = OneOrMore( Word(alphas) )
```

注意我们可以使用+操作符或组合pyparsing表达式和字符串符号(literals)。通过组合这些加单的元素为更大的表达式，我们可以完成语法定义。

```python
score = Word(nums)
schoolAndScore = schoolName + score
gameResult = date + schoolAndScore + schoolAndScore
```

我们使用`gameResult`对象去解析输入文本的每一行:

```python
tests = """\
  09/04/2004 Virginia 44 Temple 14
  09/04/2004 LSU 22 Oregon State 21
  09/09/2004 Troy State 24 Missouri 14
  01/02/2003 Florida State 103 University of Miami 2""".splitlines()
for test in tests:
  stats = gameResult.parseString(test)
  print stats.asList()
```

就像我们曾在"Hello,World"解析器里看的那样，我们从这个语法中得到一个没有结构性的表。

```
['09', '/', '04', '/', '2004', 'Virginia', '44', 'Temple', '14']
['09', '/', '04', '/', '2004', 'LSU', '22', 'Oregon', 'State', '21']
['09', '/', '09', '/', '2004', 'Troy', 'State', '24', 'Missouri', '14']
['01', '/', '02', '/', '2003', 'Florida', 'State', '103', 'University', 'of',
'Miami', '2']
```

对此的第一个改进是将返回的有关日期的分散的数据组合成简单的MM/DD/YYYY型字符串。我们只要用`Combine`类将表达式包起来就行了。

```python
date = Combine( num + "/" + num + "/" + num )
```

解析结果变为

```
['09/04/2004', 'Virginia', '44', 'Temple', '14']
['09/04/2004', 'LSU', '22', 'Oregon', 'State', '21']
['09/09/2004', 'Troy', 'State', '24', 'Missouri', '14']
['01/02/2003', 'Florida', 'State', '103', 'University', 'of', 'Miami', '2']
```

`Combine`实际上为我们做了两件事。第一是将匹配的标签合并进一个字符串，而它还使这些文本连在一起。

下一个改进是将学校名字组合起来。因为`Combine`的默认行为要求标记相邻，所以我们将不使用它。
作为替代，我们定义一个在解析时运行的过程，组合和返回单个字符串的标记.向前面提到的那样，
这类过程通过解析行为实现，它们在解析过程时执行一些函数。

对这个例子，我们将定义一个解析行为，其接受被解析的标记，使用字符串的join函数，返回组合后的字符串，
这个解析行为被一个python的lambda表达式描绘。这个解析行为与lambda表达式的绑定是通过调用一个叫`setParseAction`的函数，像这样:

```python
schoolName.setParseAction( lambda tokens: " ".join(tokens) )
```

这类手法的另一个用法是用于进行超越表达式定义的语法匹配的额外语义确认。作为例子，
之前的data的表达式会接受像03023/808098/29921这样的字符串作为有意义的数据，而这显然不是我们所期望的。
一个解析行为的对输入日期的赋意义化可以通过使用`time.strptime`方法去分析时间字符串。

```python
time.strptime(tokens[0],"%m/%d/%Y")
```

如果`strptime`检查失败，则它会抛出一个`ValueError`异常。Pyparsing使用它独特的异常类，`PrseException`，
去作为表达式匹配与否的信号。解析行为可以抛出它们独有的异常去标示，哪怕语法判定通过，但却由一些高级的语义判定所触发。
我们的解析行为将看起来像这样：

```python
def validateDateString(tokens):
  try:
    time.strptime(tokens[0], "%m/%d/%Y")
  except ValueError,ve:
    raise ParseException("Invalid date string (%s)" % tokens[0])
  date.setParseAction(validateDateString)
```

如果我们修改我们数据的第一行的输入为19/04/2004，则我们得到一个异常:

```
pyparsing.ParseException: Invalid date string (19/04/2004)(at char 0),(line:1,col:1)
```

另一个对解析结果的改进途径是使用pyparsing的Group类。Group不改变标签本身，而是将它们编组到一个子列表里。
Group在赋予解析结果结构上很有用处。

```python
score = Word(nums)
schoolAndScore = Group( schoolName + score )
```

随着编组和组合，解析结果现在看起来很有结构性了。

```
['09/04/2004', ['Virginia', '44'], ['Temple', '14'］
['09/04/2004', ['LSU', '22'], ['Oregon State', '21'］
['09/09/2004', ['Troy State', '24'], ['Missouri', '14'］
['01/02/2003', ['Florida State', '103'], ['University of Miami', '2'］
```

最终，我们将增加一个或多个解析行为去执行对数字字符串到真正数字的转换。

这在解析行为中的使用里是非常普遍的。它也显示出pyparsing可以返回结构化的数据而不只是一些被解析的字符串组成的表。
这个解析行为也可以更简单通过一个lambda表达式表示:

```python
score = Word(nums).setParseAction( lambda tokens : int(tokens[0]) )
```

又一次，我们可以定义我们的解析行为执行一个类型转换而不需要进行错误处理，如果输入的字符串不能被转换到数字类型。因为它是被表达式`Word(nums)`解析出来的，这就保证了它一定会在解析行为中有意义。

我们的返回结果开始变得像真正的对象式的数据记录。

```
['09/04/2004', ['Virginia', 44], ['Temple', 14］
['09/04/2004', ['LSU', 22], ['Oregon State', 21］
['09/09/2004', ['Troy State', 24], ['Missouri', 14］
['01/02/2003', ['Florida State', 103], ['University of Miami', 2］
```

至此，数据又有结构又有正确的类型了。故而我们可以对它进行一些真正的操作，如将比赛结果按时间排序，
标出胜利的队等。`parseString`返回的`ParseResults`对象允许我们索引数据通过嵌套下标。

但如果我们使用这种结构，事情立即相应的变得丑陋。

```python
for test in tests:
  stats = gameResult.parseString(test)
  if stats[1][1] != stats[2][1]:
    if stats[1][1] > stats[2][1]:
      result = "won by " + stats[1][0]
    else:
      result = "won by " + stats[2][0]
  else:
    result = "tied"
  print "%s %s(%d) %s(%d), %s" % (stats[0], stats[1][0], stats[1][1],stats[2][0], stats[2][1], result)
```

使用索引不仅使得代码难以理解，而且由于其非常依赖结果的次序，如果我们的语法包含一些可选项字段，
我们可能要使用一些其他方法去测试这些字段并且据此调整下标。这使得我们的解析器非常脆弱。

我们可以使用多重变量赋值以缩减索引的使用像我们在'"Hello,World!"中做的那样。

```python
for test in tests:
  stats = gameResult.parseString(test)
  gamedate,team1,team2 = stats # <- assign parsed bits to individual variable names
  if team1[1] != team2[1]:
    if team1[1] > team2[1]:
      result = "won by " + team1[0]
    else:
      result = "won by " + team2[0]
  else:
    result = "tied"
  print "%s %s(%d) %s(%d), %s" % (gamedate, team1[0], team1[1], team2[0], team2[1],result)
```

> *最佳实践:使用ResultsNames*
>
> 但是这依旧使我们对所处理数据的次序过于敏感。
取代其的，我们可以在语法中定义名字。为了做到这一点，我们插入setResults-Name到我们的语法里，所以表达式将标记出标签。

```python
schoolAndScore = Group(
schoolName.setResultsName("school") +score.setResultsName("score") )
gameResult = date.setResultsName("date") + schoolAndScore.setResultsName("team1") +schoolAndScore.setResultsName("team2")
```

而这个代码生成的结果更易读。

```python
if stats.team1.score != stats.team2.score
  if stats.team1.score > stats.team2.score:  
    result = "won by " + stats.team1.school
  else:
    result = "won by " + stats.team2.school
else:
  result = "tied"
print "%s %s(%d) %s(%d), %s" % (stats.date, stats.team1.school, stats.team1.score,stats.team2.school, stats.team2.score, result)
```

这次代码获得改善，通过引用名字引用单个标记而不是索引，这使得过程代码免疫标记顺序的变化以及可选数据字段的干扰。

创建`ParseResults`时使用结果名字将允许你使用字典风格的来引用标记。作为例子，你可以使用`ParseResults`对象支持数据在格式化字符串中使用，这么做可以简化输出代码:

```python
print "%(date)s %(team1)s %(team2)s" % stats
```

它给出下面结果：

```
09/04/2004 ['Virginia', 44] ['Temple', 14]
09/04/2004 ['LSU', 22] ['Oregon State', 21]
09/09/2004 ['Troy State', 24] ['Missouri', 14]
01/02/2003 ['Florida State', 103] ['University of Miami', 2]
```

`ParseResults`也装备了keys(),items(),values()方法，同时支持以python关键字`in`进行测试。

>*来写令人兴奋的!*
>
>pyparsing的最新版本(1.4.7)包含了让给表达式追加名字的更简单的记法。缩减代码的效果见此例:
> ```python
>schoolAndScore =Group( schoolName("school") +score("score") )
>gameResult = date("date") +schoolAndScore("team1") +schoolAndScore("team2")
> ```
> 现在没有理由不为你的解析结果命名了！

为了调试，你可以调用dump()返回已经组织化过的名字与值的层次结构。这是一个调用stats.dump()方法为第一行输入文本的结果:

```
print stats.dump()
['09/04/2004', ['Virginia', 44],
['Temple', 14］
- date: 09/04/2004
- team1: ['Virginia', 44]
 - school: Virginia
 - score: 44
- team2: ['Temple', 14]
 - school: Temple
 - score: 14
```

最终，你可以产生一个XML文件模拟类似的结构。但你需要额外指定一个根元素名称:

```
print stats.asXML("GAME")

<GAME>
 <date>09/04/2004</date>
 <team1>
  <school>Virginia</school>
  <score>44</score>
 </team1>
 <team2>
  <school>Temple</school>
  <score>14</score>
 </team2>
</GAME>
```

这里还有最后一个问题需要考虑，我们的解析器总将输入文本当成合法的。其将执行语法直到遇到终结，
然后返回匹配的结果，哪怕输入文本还有很多未被解析。作为例子，这个语句:

```python
word = Word("A")
data = "AAA AA AAA BA AAA"
print OneOrMore(word).parseString(data)
```

将不抛出一个异常，而是简化输出为:

```
['AAA', 'AA', 'AAA']
```

你可能本来想解析出更多的AAA，而那个B只是个错误。即使不是这样，额外的文本有时也是令人不安的。

如果你想避免这个情况，你应当使用`StringEnd`类。并将其“加”到你要解析的文本后面。

此时若没有解析整个文本，则会抛出`ParseException`错误于解析结束的点上。注意尾部有空格无关紧要
，pyparsing会自动跳过它们。

在我们现在的应用中，增加了`stringEnd`到我们的解析表达式里将保护我们遭遇意外的匹配。

```
09/04/2004 LSU 2x2 Oregon State 21
```

作为:

```
09/04/2004 ['LSU', 2] ['x', 2]
```

这看上去就像LSU和X学院打成了平局。为了避免这个错误，若追加了`ParseException`看起来就会像这样:

```
pyparsing.ParseException: Expected stringEnd (at char 44), (line:1, col:45)
```

这是解析器的完整代码:

```python
from pyparsing import Word, Group, Combine, Suppress, OneOrMore, alphas, nums,alphanums, stringEnd, ParseException
import time
num = Word(nums)
date = Combine(num + "/" + num + "/" + num)
def validateDateString(tokens):
  try:
    time.strptime(tokens[0], "%m/%d/%Y")
  except ValueError,ve:
    raise ParseException("Invalid date string (%s)" % tokens[0])
date.setParseAction(validateDateString)
schoolName = OneOrMore( Word(alphas) )
schoolName.setParseAction( lambda tokens: " ".join(tokens) )
score = Word(nums).setParseAction(lambda tokens: int(tokens[0]))
schoolAndScore = Group( schoolName.setResultsName("school") + score.setResultsName("score") )
gameResult = date.setResultsName("date") + schoolAndScore.setResultsName("team1") + schoolAndScore.setResultsName("team2")
tests = """\
09/04/2004 Virginia 44 Temple 14
09/04/2004 LSU 22 Oregon State 21
09/09/2004 Troy State 24 Missouri 14
01/02/2003 Florida State 103 University of Miami 2""".splitlines()
for test in tests:
  stats = (gameResult + stringEnd).parseString(test)
  if stats.team1.score != stats.team2.score:
    if stats.team1.score > stats.team2.score:
      result = "won by " + stats.team1.school
    else:
      result = "won by " + stats.team2.school
  else:
    result = "tied"
print "%s %s(%d) %s(%d), %s" % (stats.date, stats.team1.school, stats.team1.score,stats.team2.school, stats.team2.score, result)
# or print one of these alternative formats
#print "%(date)s %(team1)s %(team2)s" % stats
#print stats.asXML("GAME")
```

## <a name="page">从网页数据中提取数据
翻译TODO...

## <a name="SS">一个简易S表达式解析器
翻译TODO...

## <a name="CS">一个完整的S表达式解析器
翻译TODO...

## <a name="search">解析搜索字符串
翻译TODO...

## <a name="engine">100行代码以内的搜索引擎
略翻译TODO...

## <a name="conclusion">结论

我在PyCon'06发表关于pyparsing的介绍，在那我推荐pyparsing作为正则表达式的替代品，
或处理一些不寻常的问题。之后我被问到一个问题，"pyparsing不能用来做什么？"我踌躇了一会儿，
然后我指出pyparsing并不是处理任何情况的最佳工具-对于一个很有结构性的数据，
最好的方法就是直接上`str.split()`。我也不推荐使用pyparsing去处理XML-它有解析它自己的专用库，
而且往往比pyparsing能处理更多东西。

但我认为pyparsing是一个显然的好工具对于处理命令行程序，网页爬虫以及文本文件(如测试文件或分析输出文件)。
pyparsing已经被嵌入到很多python附加模块当中，可以到pyparsing的
[wiki](http://pyparsing.wikispaces.com/whosusingpyparsing)查询最新进展。

我已经为pyparsing写了一些文档，但我可能花费了更多时间去开发代码示例去演示不同的pyparsing代码技术。
我的经验是很多开发者都想得到示范性源代码，并在手头的特定工作上直接运用它们。最近，
我已经开始收到邮件，他们要求更多的正式文档，所以我希望这个手册可以帮助那些想要在pyparsing取得成果的人们。

### 获得更多帮助

这是一些pyparsing用户可用的网络资源，而且它们的数量一直在增长

* pyparsing wiki(http://pyparsing.wikispaces.com):这个wiki是最早提供pyparsing新消息的资源处。
它包含了安装信息，FAQ，以及使用了pyparsing的项目列表。一个例子页面会给出不同的“如何做”例子，
包括对算术表达式，国际象棋标记，JSON数据,SQL语句(它们也包括在pyparsing的分发代码中)的。
pyparsing的更新，描述和其他事件被在News页面上更新。而主页的讨论频道是一个用户之间交流思想，
提出问题的好地方。

* pyparsing邮件组(pyparsing-users@lists.sourceforge.net)。另一个普遍使用的提出pyparsing问题的资源。
以前的邮件组信息可以再pyparsing的SourceForge项目上找到，http://sourceforge.net/projects/pyparsing.

* comp.lang.python:这个世界性的讨论组是一个关乎python的一般讨论组，但很可能pyparsing相关的频道将在此建立。
这是个好地方去问关于python用法或特殊模块或特殊领域的研究的问题。如果你在这个列表中Google"pyparsing"，你将找到很多讨论。

## <a name="index">索引

翻译TODO...
