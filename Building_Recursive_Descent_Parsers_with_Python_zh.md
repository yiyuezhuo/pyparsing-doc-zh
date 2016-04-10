# 用Python构建递归下降解析器

by Paul McGuire

01/26/2006

什么是"解析"?解析是一处理一系列符号的过程，提取它们的意义。比较其日常，就是阅读句子中的单词并且理解其中的意思。
当应用程序需要处理以文本构成的数据，它们必须使用某种解析逻辑。这个逻辑扫描文本中的字符，字符组（单词），字符组
的模式以提取背后的命令或信息。

解析器程序经常是针对特定类型的文本的。这种文本可以是保险与医疗的表格;C语言头文件的函数声明;一个图的点边的连接关系描述;网页上的HTML
标记或者配置一个网络，修改或旋转一个3D图像，或描述冒险游戏中的游戏流程的脚本。在每个情况中，解析器处理一个特定的字符组/模式。这个
模式的集合就是这个解析器的语法。

作为例子，当解析一个字符串Hello, World! 你可能想要解析遵循这样的模式任意问候句式。Hello, World!以一个问候词开始:单词Hello。
存在很多问候次--Howdy,Greetings,Aloha,G'day之类的--所以你可以定义一个简单的语法，它以一个单独的问候词开始。后面跟了一个
逗号，再后面是问候的对象，也是一个单词。最后一些终结符(terminating punctuation)终结了这个问候，像感叹号等。
这样一个问候的语法粗略来看大概可以这样表示(`::` 读作 "由...组成")
 
```
word           :: group of alphabetic characters
salutation     :: word
comma				      :: ","
greetee				    :: word
endPunctuation	:: "!"
greeting				   :: salutation comma greetee endPunctuation
```

这是巴克斯范式(BNF)。存在很多语法变体表示诸如可选/必须(optional/manatory) 组成，重复，候选之类的语法概念。

一旦你已经以BNF确定了一个特定语法，你必须将其转化为可执行形式。一个常用的方式是以一个递归下降解析器
的形式来进行开发。一个这样的解析器定义了一个函数，其每次读入一个字符，高级函数调用底层函数，它们会
返回success如果匹配成功，否则抛出一个异常。

## Pyparsing是什么?

Pyparsing 是一个Python模块帮助你便捷创建递归下降解析器。这里一个用pyparsing实现的Hello, World!实例:

```python
from pyparsing import Word, Literal, alphas

salutation     = Word( alphas + "'" )
comma          = Literal(",")
greetee        = Word( alphas )
endPunctuation = Literal("!")

greeting = salutation + comma + greetee + endPunctuation
```

下面是pyparsing可以帮助开发者创建文本解析函数的一些特点:


* 语法是纯Python的，所以么有单独的语法定义文件被要求
* 不需要使用特殊的语法，除了 `+`表示且,`^`表示或(长模式，或者说"贪婪"匹配。),`|`表示第一次匹配，以及`~`表示非。
* 不需要单独的代码生成阶段
* 它隐式的跳过解析元素之间的空格或注释;避免使你的代码因为应该忽略的文本而变得乱糟糟的。

上面的pyparsing语法不只可以解析Hello, World! 还能解析这些:

* Hey, Jude!
* Hi, Mom!
* G'day, Mate!
* Yo, Adrian!
* Howdy, Pardner!
* Whattup, Dude!

Listing 1包含了完整的 Hello, World! 解析器代码，也有解析结果的输出。

**Listing 1**

```python
from pyparsing import Word, Literal, alphas

salutation     = Word( alphas + "'" )
comma          = Literal(",")
greetee        = Word( alphas )
endPunctuation = Literal("!")

greeting = salutation + comma + greetee + endPunctuation

tests = ("Hello, World!", 
"Hey, Jude!",
"Hi, Mom!",
"G'day, Mate!",
"Yo, Adrian!",
"Howdy, Pardner!",
"Whattup, Dude!" )

for t in tests:
        print t, "->", greeting.parseString(t)
```
```
Hello, World! -> ['Hello', ',', 'World', '!']
Hey, Jude! -> ['Hey', ',', 'Jude', '!']
Hi, Mom! -> ['Hi', ',', 'Mom', '!']
G'day, Mate! -> ["G'day", ',', 'Mate', '!']
Yo, Adrian! -> ['Yo', ',', 'Adrian', '!']
Howdy, Pardner! -> ['Howdy', ',', 'Pardner', '!']
Whattup, Dude! -> ['Whattup', ',', 'Dude', '!']
```

## Pyparsing是一个"组合器"

使用pyparsing模块时，你应当首先定义你语法最基础的部分。然后再组合它们成一个更复杂的解析表达式
以表示整体语法的不同变体。可以以下面的关系来对它们进行组合:

* 表达式可以跟在另一个后面，如"`if`关键词后面应该是一个括号包裹的布尔表达式"。
* 在一个特定位置的表达式可以以一些候选表达式表示，如"一个SQL命令可以以 SELECT,INSERT,UPDATE或DELETE 开头"。
* 表达式是可选的，如"一个电话号码可以有或没有一个括号包裹的区号开头"。
* 表达式是可以重复的，如"一个开的XML标记"可以包含零或多个属性"。

尽管一些复杂的语法会涉及数百个语法组合，但是大多数解析任务都可以以少数几个定义来表出。
使用BNF表示语法会帮助你思考与设计解析器，它也帮助你追踪你实现语法的进度通过使用pyparsing的函数与类。

## 定义一个简单的语法

最基础的语法块一般是指定某一字符集的字符串。作为例子，这是一个简单的BNF，其解析一个电话号码:

```
number :: '0'.. '9'*
phoneNumber	:: [ '(' number ')' ] number '-' number
```

Because this looks for dashes and parentheses within a phone number string, 
you can also define simple literal tokens for these punctuation marks:

```python
dash   = Literal( "-" )
lparen = Literal( "(" )
rparen = Literal( ")" )
```

To define the groups of numbers in the phone number, you need to handle groups of characters of varying lengths. 
For this, use the Word token:

```python
digits = "0123456789"
number = Word( digits )
```

The number token will match contiguous sequences made up of characters listed in the string digits; that is,
it is a "word" composed of digits (as opposed to a traditional word, which is composed of letters of the alphabet). 
Now you have enough individual pieces of the phone number, so you can string them together using the And class.

```python
phoneNumber = 
    And( [ lparen, number, rparen, number, dash, number ] )
```

This is fairly ugly, and unnatural to read. Fortunately, the pyparsing module defines operator methods to combine separate parse elements more easily.
A more legible definition uses `+` for And:

```python
phoneNumber = lparen + number + rparen + number + dash + number
```

For an even cleaner version, the `+` operator will join strings to parse elements, implicitly converting the strings to Literals. 
This gives the very easy-to-read:

```python
phoneNumber = "(" + number + ")" + number + "-" + number
```

Finally, to designate that the area code at the beginning of the phone number is optional, use pyparsing's Optional class:

```python
phoneNumber = Optional( "(" + number + ")" ) + number + "-" + number
```

## Using the Grammar

Once you've defined your grammar, the next step is to apply it to the source text. 
Pyparsing expressions support three methods for processing input text with a given grammar:

* The parseString method uses a grammar that completely specifies the contents of an input string, parses the string, 
and returns a collection of strings and substrings for each grammar construct.
* The scanString method uses a grammar that may match only parts of an input string, scans the string looking for matches, 
and returns a tuple that contains the matched tokens and their starting and ending locations within the input string.
* The transformString method is a variation on scanString. 
It applies any changes to the matched tokens and returns a single string representing the original input text, 
as modified by the individual matches.

The initial Hello, World! parser calls parseString and returns straightforward token results:

```
Hello, World! -> ['Hello', ',', 'World', '!']
```

Although this looks like a simple list of token strings, pyparsing returns data using a ParseResults object. 
In the example above, the results variable behaves like a simple Python list. 
In fact, you can index into the results just like a list:


```python
print results[0]
print results[-2]
```

will print:

```
Hello
World
```

ParseResults also lets you define names for individual syntax elements, 
making it easier to retrieve bits and pieces of the parsed text. 
This is especially helpful when a grammar includes optional elements, 
which can change the length and offsets of the returned token list.
By modifying the definitions of `salute` and `greetee`:

```python
salute  = Word( alphas+"'" ).setResultsName("salute")
greetee = Word( alphas ).setResultsName("greetee")
```

you can reference the corresponding tokens as if they were attributes of the returned results object:

```python
print hello, "->", results    
print results.salute
print results.greetee
```

Now the program will print:

```
G'day, Mate! -> ["G'day", ',', 'Mate', '!']
G'day
Mate
```

Results names can greatly help to improve the readability and maintainability of your parsing programs.

In the case of the phone number grammar, you can parse an input string containing a list of phone numbers, one after the next, as:

```python
phoneNumberList = OneOrMore( phoneNumber )
data            = phoneNumberList.parseString( inputString )
```

This will return data as a pyparsing ParseResults object, containing a list of all of the input phone numbers.

Pyparsing includes some helper expressions, such as `delimitedList`, 
so that if your input were a comma-separated list of phone numbers, 
you could simply change phoneNumberList to:

```python
phoneNumberList = delimitedList( phoneNumber )
```

This will return the same list of phone numbers that you had before. 
(`delimitedList` supports any custom string or expression as a delimiter,
 but comma delimiters are the most common, and so they are the default.)

If, instead of having a string containing only phone numbers, you had a complete mailing list of names, addresses, 
zip codes, and phone numbers, you could extract the phone numbers using scanString. scanString is a Python generator function, 
so you must use it in a for loop, list comprehension, or generator expression.

```
for data,dataStart,dataEnd in 
    phoneNumber.scanString( mailingListText ):
    .
    .
    # do something with the phone number tokens, 
    # returned in the 'data' variable
    .
    .
```

Lastly, if you had the same mailing list but wished to hide the numbers from, say, a potential telemarketer, 
you could transform the string by attaching a parse action that just changes all phone numbers to the string (000)000-0000. 
Replacing the input tokens with a fixed string is a common parse action, 
so pyparsing provides a built-in function replaceWith to make this very simple:

```python
phoneNumber.setParseAction( replaceWith("(000)000-0000") )
sanitizedList = 
    phoneNumber.transformString( originalMailingListText )
```

## When Good Input Goes Bad

Pyparsing will process input text until it runs out of matching text for its given parser elements. 
If it finds an unexpected token or character and there is no matching parsing element, 
then pyparsing will raise a ParseException. ParseExceptions print out a diagnostic message by default; 
they also have attributes to help you locate the line number, column, text line, and annotated line of text.

If you provide the input string Hello, World? to your parser, you will receive the exception:

```python
pyparsing.ParseException: Expected "!" (at char 12), (line:1, col:13)
```

At this point, you can choose to fix the input text or make the grammar more tolerant of other syntax 
(in this case, supporting question marks as valid sentence terminators).

## A Complete Application

Consider an application where you need to process chemical formulas, such as NaCl, H2O, or C6H5OH. For this application, 
the chemical formula grammar will be one or more element symbols, each followed by an optional integer. 
In BNF-style notation, this is:

```
integer			   	:: '0'..'9'+
cap					      :: 'A'..'Z'
lower					    :: 'a'..'z'
elementSymbol	:: cap lower*
elementRef		  :: elementSymbol [ integer ]
formula				   :: elementRef+
```

The pyparsing module handles these concepts with the classes Optional and OneOrMore. 
The definition of the elementSymbol will use the two-argument constructor Word: 
the first argument lists the set of valid leading characters, 
and the second argument gives the set of valid body characters. Using the pyparsing module, a simple version of the grammar is:

```python
caps       = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
lowers     = caps.lower()
digits     = "0123456789"

element    = Word( caps, lowers )
elementRef = element + Optional( Word( digits ) )
formula    = OneOrMore( elementRef )

elements   = formula.parseString( testString )
```

So far, this program is an adequate tokenizer, processing the following formulas into their appropriate tokens. 
The default behavior for pyparsing is to return all of the parsed tokens within a single list of matching substrings:

```
H2O		  -> ['H', '2', 'O']
C6H5OH	-> ['C', '6', 'H', '5', 'O', 'H']
NaCl		 -> ['Na', 'Cl']
```

Of course, you want to do some processing with these returned results, beyond simply printing them out as a list. 
Assume that you want to compute the molecular weight for each given chemical formula. 
The program somewhere defines a dictionary of chemical symbols and their corresponding atomic weight:

```
atomicWeight = {
    "O"  : 15.9994,
    "H"  : 1.00794,
    "Na" : 22.9897,
    "Cl" : 35.4527,
    "C"  : 12.0107,
    ...
    }
```

Next it would be good to establish a more logical grouping in the parsed chemical symbols and associated quantities, to return a structured set of results. Fortunately, the pyparsing module provides the Group class for just this purpose. By changing the elementRef declaration from:

```python
elementRef = element + Optional( Word( digits ) )
```

to:

```python
elementRef = Group( element + Optional( Word( digits ) ) )
```

you will now get the results grouped by chemical symbol:

```
H2O		  -> [['H', '2'], ['O']]
C6H5OH -> [['C', '6'], ['H', '5'], ['O'], ['H']]
NaCl		 -> [['Na'], ['Cl']]
```

The last simplification is to include a default value for the quantity part of elementRef, 
using the default argument for the constructor of the Optional class:

```python
elementRef = Group( element + Optional( Word( digits ), 
                                default="1" ) )
```

Now every elementRef will return a pair of values: the element's chemical symbol and the number of atoms of that element, 
with "1" implied if no quantity is given. Now the test formulas return a very clean list of ordered pairs of element symbols and their respective quantities:

```
H2O		-> [['H', '2'], ['O', '1']]
C6H5OH	-> [['C', '6'], ['H', '5'], ['O', '1'], ['H', '1']]
NaCl		-> [['Na', '1'], ['Cl', '1']]
```

The final step is to compute the atomic weight for each. Add a single line of Python code after the call to parseString:

```python
wt = sum( [ atomicWeight[elem] * int(qty) 
                    for elem,qty in elements ] )
```

giving the results:

```
H2O		  -> [['H', '2'], ['O', '1']] (18.01528)
C6H5OH	-> [['C', '6'], ['H', '5'], ['O', '1'], ['H', '1']]
        (94.11124)
NaCl		 -> [['Na', '1'], ['Cl', '1']] (58.4424)
```

Listing 2 contains the entire pyparsing program.

**Listing 2**

```python
from pyparsing import Word, Optional, OneOrMore, Group, ParseException

atomicWeight = {
    "O"  : 15.9994,
    "H"  : 1.00794,
    "Na" : 22.9897,
    "Cl" : 35.4527,
    "C"  : 12.0107
    }
    
caps = "ABCDEFGHIJKLMNOPQRSTUVWXYZ"
lowers = caps.lower()
digits = "0123456789"

element = Word( caps, lowers )
elementRef = Group( element + Optional( Word( digits ), default="1" ) )
formula = OneOrMore( elementRef )

tests = [ "H2O", "C6H5OH", "NaCl" ]
for t in tests:
    try:
        results = formula.parseString( t )
        print t,"->", results,
    except ParseException, pe:
        print pe
    else:
        wt = sum( [atomicWeight[elem]*int(qty) for elem,qty in results] )
        print "(%.3f)" % wt
```

```
H2O		  -> [['H', '2'], ['O', '1']] (18.015)
C6H5OH	-> [['C', '6'], ['H', '5'], ['O', '1'], ['H', '1']] (94.111)
NaCl	 	-> [['Na', '1'], ['Cl', '1']] (58.442)
```

One of the nice by-products of using a parser is the inherent validation it performs on the input text. 
Note that in the calculation of the wt variable, there was no need to test that the qty string was all numeric, 
or to catch ValueError exceptions an invalid argument raised. If qty weren't all numeric, and therefore a valid argument to int(), 
it would not have passed the parser.

## An HTML Scraper

As a final example, consider the development of a simple HTML "scraper." It is not a comprehensive HTML parser, 
as such a parser would require scores of parse expressions. Fortunately, it is not usually necessary to have a complete HTML grammar 
definition to be able to extract significant pieces of data from most web pages, 
especially those autogenerated by CGI or other application programs.

This example will extract data with a minimal parser, targeted to work with a specific web page--in this case, 
the page kept by NIST listing publicly available network time protocol (NTP) servers. 
This routine could be part of a larger NTP client application that, during its initialization, 
would look up what NTP servers are currently available.

To begin developing an HTML scraper, you must first see what sort of HTML text you will need to process. 
By visiting the web site and viewing the returned HTML source, 
you can see that the page lists the names and IP addresses of the NTP servers in an HTML table:


```
Name				        IP Address		Location
time-a.nist.gov	129.6.15.28	NIST, Gaithersburg, Maryland
time-b.nist.gov	129.6.15.29	NIST, Gaithersburg, Maryland
```

The underlying HTML source for this table uses `<table>`, `<tr>`, and `<td>` tags to structure the NTP server data:

```html
<table border="0" cellpadding="3" cellspacing="3" frame="" width="90%">
                <tr align="left" valign="top">
                        <td><b>Name</b></td>
                        <td><b>IP Address</b></td>
                        <td><b>Location</b></td>
                </tr>
                <tr align="left" valign="top" bgcolor="#c7efce">
                        <td>time-a.nist.gov</td>
                        <td>129.6.15.28</td>
                        <td>NIST, Gaithersburg, Maryland</td>
                </tr>
                <tr align="left" valign="top">
                        <td>time-b.nist.gov</td>
                        <td>129.6.15.29</td>
                        <td>NIST, Gaithersburg, Maryland</td>
                </tr>
```

This table is part of a much larger body of HTML, but pyparsing allows you to define a parse expression that matches only 
a subset of the total input text and to scan for text that matches the given parse expression. 
So you need only define the minimum amount of grammar required to match the desired HTML source.

The program should extract the IP addresses and locations of those servers, 
so you can focus your grammar on just those columns of the table. 
Informally, you want to extract the values that match the pattern

```html
<td> IP address </td> <td> location name </td>
```

You do want to be a bit more specific than just matching on something as generic as 
`<td>` any text `</td>` `<td> `more any text `</td>`, 
because so general an expression would match the first two columns of the table instead of the second two 
(as well as the first two columns of any table on the page!). Instead, 
use the specific format of the IP address to help narrow your search pattern by 
eliminating any false matches from other table data on the page.

To build up the elements of an IP address, start by defining an integer, then combining four integers with intervening periods:

```python
integer   = Word("0123456789")
ipAddress = integer + "." + integer + "." + integer + "." + integer
```

You will also need to match the HTML tags `<td>` and `</td>`, so define parse elements for each:

```python
tdStart = Literal("<td>")
tdEnd   = Literal("</td>")
```

In general, `<td>` tags can also contain attribute specifiers for alignment, color, and so on. 
However, this is not a general-purpose parser, only one written specifically for this web page, which fortunately does not use complicated 
`<td>` tags. (The latest version of pyparsing includes a helper method for constructing HTML tags, which supports attribute specifiers in opening tags.)

Finally, you need some sort of expression to match the server's location description. 
This is actually a rather freely formatted bit of text--there's no knowing whether it will include alphabetic data, commas, periods, 
or numbers--so the simplest choice is to just accept everything up to the terminating `</td>` tag. 
Pyparsing includes a class named SkipTo for this kind of grammar element.

You now have all the pieces you need to define the time server text pattern:

```python
timeServer = tdStart + ipAddress + tdEnd + tdStart + SkipTo(tdEnd) + tdEnd
```

To extract the data, invoke `timeServer.scanString`, 
which is a generator function that yields the matched tokens and the start and end string positions for each matching set of text. 
This application uses only the matched tokens.

**Listing 3**

```python
from pyparsing import *
import urllib

# define basic text pattern for NTP server 
integer = Word("0123456789")
ipAddress = integer + "." + integer + "." + integer + "." + integer
tdStart = Literal("<td>")
tdEnd = Literal("</td>")
timeServer =  tdStart + ipAddress + tdEnd + tdStart + SkipTo(tdEnd) + tdEnd

# get list of time servers
nistTimeServerURL = "http://tf.nist.gov/service/time-servers.html"
serverListPage = urllib.urlopen( nistTimeServerURL )
serverListHTML = serverListPage.read()
serverListPage.close()

for srvrtokens,startloc,endloc in timeServer.scanString( serverListHTML ):
    print srvrtokens
```

Running the program in Listing 3 gives the token data:

```
[' <td>', '129', '.', '6', '.', '15', '.', '28', '</td> ', ' <td>', 'NIST, Gaithersburg, Maryland', '</td> ']
[' <td>', '129', '.', '6', '.', '15', '.', '29', '</td> ', ' <td>', 'NIST, Gaithersburg, Maryland', '</td> ']
[' <td>', '132', '.', '163', '.', '4', '.', '101', '</td> ', ' <td>', 'NIST, Boulder, Colorado', '</td> ']
[' <td>', '132', '.', '163', '.', '4', '.', '102', '</td> ', ' <td>', 'NIST, Boulder, Colorado', '</td> ']
:
```

Looking at these results, a couple of things immediately jump out.
One is that the parser records each IP address as a series of separate tokens, 
one for each subfield and delimiting period. It would be nice if pyparsing were to do a bit of work during the parsing process 
to combine these fields into a single-string token. Pyparsing's Combine class will do just this. 
Modify the ipAddress definition to read:

```python
ipAddress = Combine( integer + "." + integer + "." + integer + "." + integer )
```

to get a single-string token returned for the IP address.

The second observation is that the results include the opening and closing HTML tags that mark the table columns. 
While the presence of these tags is important during the parsing process, 
the tags themselves are not interesting in the extracted data. 
To have them suppressed from the returned token data, construct the tag literals with the suppress method.

```python
tdStart = Literal("<td>").suppress()
tdEnd   = Literal("</td>").suppress()
```

**Listing 4**

```python
from pyparsing import *
import urllib

# define basic text pattern for NTP server 
integer = Word("0123456789")
ipAddress = Combine( integer + "." + integer + "." + integer + "." + integer )
tdStart = Literal("<td>").suppress()
tdEnd = Literal("</td>").suppress()
timeServer =  tdStart + ipAddress + tdEnd + tdStart + SkipTo(tdEnd) + tdEnd

# get list of time servers
nistTimeServerURL = "http://tf.nist.gov/service/time-servers.html"
serverListPage = urllib.urlopen( nistTimeServerURL )
serverListHTML = serverListPage.read()
serverListPage.close()

for srvrtokens,startloc,endloc in timeServer.scanString( serverListHTML ):
    print srvrtokens
```

Now run the program in Listing 4. Your returned token data has substantially improved:

```
['129.6.15.28', 'NIST, Gaithersburg, Maryland']
['129.6.15.29', 'NIST, Gaithersburg, Maryland']
['132.163.4.101', 'NIST, Boulder, Colorado']
['132.163.4.102', 'NIST, Boulder, Colorado']
```
Finally, add result names to these tokens, so that you can access them by attribute name. 
The easiest way to do this is in the definition of `timeServer`:

```python
timeServer = tdStart + ipAddress.setResultsName("ipAddress") + tdEnd 
        + tdStart + SkipTo(tdEnd).setResultsName("locn") + tdEnd
```

Now you can neaten up the body of the for loop and access these tokens just like members in a dictionary:

```python
servers = {}

for srvrtokens,startloc,endloc in timeServer.scanString( serverListHTML ):
    print "%(ipAddress)-15s : %(locn)s" % srvrtokens
    servers[srvrtokens.ipAddress] = srvrtokens.locn
```

Listing 5 contains the finished running program.

**Listing 5**

```python
from pyparsing import *
import urllib

# define basic text pattern for NTP server 
integer = Word("0123456789")
ipAddress = Combine( integer + "." + integer + "." + integer + "." + integer )
tdStart = Literal("<td>").suppress()
tdEnd = Literal("</td>").suppress()
timeServer = tdStart + ipAddress.setResultsName("ipAddress") + tdEnd + \
             tdStart + SkipTo(tdEnd).setResultsName("locn") + tdEnd

# get list of time servers
nistTimeServerURL = "http://tf.nist.gov/service/time-servers.html"
serverListPage = urllib.urlopen( nistTimeServerURL )
serverListHTML = serverListPage.read()
serverListPage.close()

servers = {}
for srvrtokens,startloc,endloc in timeServer.scanString( serverListHTML ):
    print "%(ipAddress)-15s : %(locn)s" % srvrtokens
    servers[srvrtokens.ipAddress] = srvrtokens.locn

print servers
```

At this point, you've successfully extracted the NTP servers and their IP addresses and populated a program variable 
so that your NTP-client application can make use of the parsed results.

## In Conclusion

Pyparsing provides a basic framework for creating recursive-descent parsers, 
taking care of the overhead functions of scanning the input string, 
handling expression mismatches, selecting the longest of matching alternatives, 
invoking callback functions, and returning the parsed results. 
This leaves developers free to focus on their grammar design and the design and implementation of corresponding token processing. 
Pyparsing's nature as a combinator allows developers to 
scale their applications from simple tokenizers up to complex grammar processors. 
It is a great way to get started with your next parsing project!

[Download pyparsing from SourceForge](http://pyparsing.sourceforge.net/).

[Paul McGuire](http://www.onlamp.com/pub/au/2557) is a senior manufacturing systems consultant at Alan Weber & Associates. 
In his spare time, he administers the pyparsing project on SourceForge.
