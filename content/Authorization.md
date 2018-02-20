# Authorization（授权）

> 授权是评估请求中信息的过程。此信息可用于对照从文件、数据库或LDAP目录获得的关于用户的信息进行验证。授权发生在身份验证之前，不涉及密码检查。我们可以使用各种逻辑和比较来确定用户是否被授权连接到网络。我们还可以确定他们使用网络的时间或服务质量。这些都是授权的组成部分，本章将对此进行讨论。

在本章中，我们将:

* 查看如何对用户应用限制
* 查看FreeRADIUS如何执行授权
* FreeRADIUS下的unlang处理语言探讨
* 使用unlang创建数据重置计数器

那么，让我们继续吧...

## 实施限制

授权本质上是关于限制的。基于某些检查，可以限制用户。限制可应用于以下两个位置之一:

* 在RADIUS服务器上
* 在NAS上

当Access-Request数据包发送到RADIUS服务器时，在身份验证过程中确定限制。Accounting-Request数据包不会也无法确定限制。

在RADIUS服务器上应用限制时，服务器返回访问拒绝数据包，其中应包括指定拒绝原因的Reply-Message AVP。

在NAS应用限制时，RADIUS服务器返回一个访问接受数据包，其中包括NAS应应用的AVPs。这意味着您必须确保NAS收到正确的AVPs以实施限制，并且它首先还支持这些AVPs。

## 在FreeRADIUS授权

这一部分可以看作是我们到目前为止所涉及的主题的概述，也可以看作是在获得授权进行更多实践练习之前的复习。

* 请求从NAS (客户端)发送到FreeRADIUS (服务器)。
* 这些请求由在FreeRADIUS配置中定义的虚拟服务器处理。默认虚拟服务器称为默认服务器。
* 处理传入请求的方式取决于虚拟服务器文件内各个部分的配置。
* 请求本身按逻辑顺序由各个部分处理。授权部分始终在验证部分之前处理访问请求数据包。preacct部分同样总是在accounting部分之前处理Accounting-Request包。
* 虽然不能更改顺序，但我们在其中有很大的灵活性来处理请求。

下一部分将介绍处理请求的方法。通过使用比较和逻辑，我们可以控制请求的流程和操作属性。

## unlang介绍

FreeRADIUS中提供的unlang语言在授权到新高度时具有灵活性。unlang不是一种成熟的编程语言，而是一种处理语言。unlang的目的是实现策略，而不是替换使用Perl或Python创建的复杂脚本。unlang坚持一个基本语法，包括条件语句和变量操作。unlang代码不会编译，而是由FreeRADIUS服务器解释。解释发生在服务器读取配置文件时，这通常发生在启动期间。unlang标记的使用仅限于配置文件中的指定部分，不能在模块中使用。

unlang的一个关键特性是能够使用条件语句来控制处理请求的进程。

> FreeRADIUS为unlang安装了一个手册页，您可以参考:
> `$>man unlang`

我们将演示如何使用各种条件语句来显示如何处理请求。

## 使用条件语句

条件语句很简单，但功能强大，以至于它们仍然是任何软件的构建块。unlang功能有两种实现条件检查的方法:

1. `if`语句。这包括作为语句一部分的`else`和`elsif`选项。
2. `switch`语句

本节将介绍`if`语句的各种用途。

### 使用unlang中的`if`语句

if语句本身不是非常复杂。它具有以下格式:

```
if(condition){...}
```

条件部分由于其多种可能性而变得复杂。

### 使用if语句获取返回代码

现在，我们将查看模块的返回代码，并使用此代码与指定的条件进行比较。FreeRADIUS中的每个模块都需要在调用后返回代码。

此代码的值随后可以用作if语句中的条件检查。

### 使用if语句授权用户

如果用户不在users文件中，本练习将使用if条件拒绝访问请求。

编辑FreeRADIUS配置目录下的sites-available/default虚拟服务器，并在authorize部分的files条目下添加以下行:

```
if(noop){reject}
```

在调试模式下重新启动FreeRADIUS，并尝试使用users文件中不存在的用户名和密码进行身份验证。我们假设ali在任何地方都没有定义:

`radtest ali passme 127.0.0.1 100 testing123`

您应该得到一个拒绝访问数据包。

### 刚才发生了什么？

我们在authorize部分的files模块下添加了条件检查。要激活此更改，必须重新启动FreeRADIUS。这将导致radius服务器读取和解释unlang代码。然后通过向服务器发送访问请求分组来测试unlang代码。

我们将首先查看if条件，然后查看满足if条件时采取的操作。

## 模块返回码

如果查看FreeRADIUS的调试输出，您将看到每个模块如何返回代码。下面列出了可用的返回代码及其含义:

Module return code | Description
---|---notfound |Information was not found（未找到信息）noop |The module did nothing（模块什么也没做）ok |The module succeeded（模块成功）updated |The module updated the request （For example, it set the Auth-Type internal AVP）（模块更新了请求。“例如，它设置授权类型内部AVP”）fail |The module failed（模块失败）reject |The module rejected the request（模块拒绝了请求）userlock |The user was locked out（用户被锁定）
invalid |The configuration was invalid（配置无效）handled |The module handled the request itself（模块本身处理请求）

我们可以使用unlang的if语句来测试模块中指定的返回代码。为此，我们必须向unlang提示它必须测试模块返回代码。因此，我们在if语句的条件下将要测试的返回代码指定为非引号字符串。

> 如果条件是未加引号的字符串，并且是上表中列出的模块返回代码之一，则unlang会将此字符串与最新模块的返回代码进行比较。

你可能认为我们应该测试notfound而不是noop。files模块在用户不在users文件中时返回noop而不是notfound。在创建条件测试之前，请记住在某些情况下测试或找出模块返回的值。如果你做不到这一点，结果可能与你预期的不同。

## unlang关键词

当满足if条件时，我们可以采取各种措施。unlang使用关键字处理请求。if语句就是这样一个关键字。还有update关键字，在处理属性时使用，本章稍后将讨论。然后还有一个关键字列表，可以在if语句中使用。如果用户不在users文件中，我们使用reject关键字立即拒绝请求。下表列出了可在if语句中使用的关键字及其对请求的影响。

Keywords | Description
---|---noop | Do nothing.ok |Instructs the server that the request was processed properly.This keyword can be used to over-ride earlier failures, if the local administrator determines that the failures are not catastrophic.（指示服务器请求已正确处理。如果本地管理员确定故障不是灾难性的，则此关键字可用于忽略早期故障。）fail |  Causes the request to be treated as if a failure had occurred.（使请求被视为已发生故障。）reject | Causes the request to be rejected immediately.（导致请求立即被拒绝。）

请注意，虽然名称与模块的返回代码相同，但模块返回代码与这些关键字之间存在差异。这些关键字是unlang的一部分，并在if语句中使用。如果if语句中没有定义关键字，则默认情况下它将返回noop。

除了这些关键字，我们还可以指定任何FreeRADIUS模块的名称。模块名称被视为关键字。`if`语句也可以嵌套。

## 使用条件语句的其他测试

条件语句为我们提供了多种测试功能。例如，在授权过程中可以使用这些属性来检查NAS是否为Access-Request提供了必需的属性。我们甚至可以使用逻辑运算符组合测试来创建在授权用户之前必须满足的复杂条件。本节将介绍另外两个条件测试，它们可以用作构建块来创建灵活的授权策略。

这些练习假定为没有动过sites-available/default文件，并且可以独立完成。我们还假设users文件包含一个名为Alice的用户，密码为passme (这与前面所有章节中定义和使用的用户相同)。

### 检查属性是否存在

我们可以检查指定的AVP是否存在。如果在条件中将属性的名称指定为未加引号的字符串，则unlang将检查请求中是否存在此AVP。

1.编辑FreeRADIUS配置目录下的sites-available/default虚拟服务器，并在authorize部分的files条目下添加以下内容:

```
if(Framed-Protocol){reject}
```

2.在调试模式下重新启动FreeRADIUS，并尝试以Alice身份进行身份验证，同时在radtest命令的末尾添加1。这将Framed-Protocol AVP 包括在请求中:

`radtest alice passme 127.0.0.1 100 testing123 1`

您应该得到一个拒绝访问数据包。

我们可以从调试输出中看到如何评估`if`语句。将请求中包含Framed-Protocol时的输出与缺少Framed-Protocol时的输出进行比较。

> 虽然我们在概念验证过程中使用Framed-Protocol属性是为了方便，但在现实世界中，Framed-Protocol AVP既可以存在于访问接受数据包中，也可以存在于访问应答数据包中。它指示要用于框架访问的框架。最常见的值是点对点协议( PPP )。
以下是存在Framed-Protocol时的输出:

```
++? if (Framed-Protocol)? Evaluating (Framed-Protocol) -> TRUE++? if (Framed-Protocol) -> TRUE++- entering if (Framed-Protocol) {...}+++[reject] returns reject++- if (Framed-Protocol) returns reject
```

如果缺少Framed-Protocol，则输出如下:


```
++? if (Framed-Protocol)? Evaluating (Framed-Protocol) -> FALSE++? if (Framed-Protocol) -> FALSE
```

请记住，此条件测试仅检查请求中是否存在AVP。它不检查AVP的特定值。我们将在本章后面测试某些值。

到目前为止，我们有两种类型的非引号字符串:

* unlang将noop解释为最后一个模块的返回代码。
* Framed-Protocol被unlang标记为一个属性。

如果未加引号的字符串都不是，会发生什么情况？答案如下:

* 一个word等于true.。
* 数字0等于false,，其他数字等于true。

> 如果要检查属性是否不存在，只需在属性前面添加感叹号`!`。unlang中的感叹号是逻辑“否”，用于测试条件是否不存在。

## 使用逻辑表达式对用户进行身份验证

unlang还支持条件语句中的逻辑AND (`&&`)和逻辑OR (`||`)。在本练习中，我们将拒绝不在users文件中或请求中存在Framed-Protocol AVP的用户。

1.编辑FreeRADIUS配置目录下的sites-available/default虚拟服务器，并在authorize（授权）部分的files条目下添加以下行:


```
if((noop)||(Framed-Protocol)){reject}
```

2.在调试模式下重新启动FreeRADIUS，并尝试以alice身份进行身份验证，同时在radtest命令的末尾添加1。这将包括请求中的Framed-Protocol AVP :

`radtest alice passme 127.0.0.1 100 testing123 1`

你将会得一个Acces-Reject packet

3.尝试使用users文件中不存在的用户名和密码进行身份验证。您应该得到一个Access-Reject数据包。

4.最后，尝试使用users文件中不存在的用户名和密码进行身份验证，并将1添加到radtest命令的末尾。您应该得到一个Access-Reject数据包。

FreeRADIUS服务器的调试反馈将指示在请求期间如何评估if条件。

## 属性和变量（Attributes and variables）

> RADIUS中的授权在很大程度上取决于属性。我们可以在Access-Request中使用AVPs来验证它在授权期间是否满足我们的要求。我们还可以在Access-Reply中返回AVPs，以指示NAS用户仅被授权执行某些操作。

> 由于RADIUS协议都是关于属性的，因此unlang大多使用属性作为变量。还有一些变量不是属性的例外情况。这也将在本节中介绍。

### 属性列表

FreeRADIUS通过将属性存储在列表中来管理属性。列表类似于命名空间，它允许具有相同名称的属性独立存在于不同位置。Unlang可用于在以下不同列表中操作或添加属性:

* 有一个request列表，其中包含请求中的所有AVPs，例如User-Name。
* 有一个reply列表，其中包含最终将在回复中的所有AVPs，例如Reply-Message。
* 我们还可以工作在前面章节中的control列表，其中我们将此列表中的属性称为内部属性，例如Auth-Type。
* 要引用特定列表中的属性，我们使用列表名称和冒号，后跟属性名称，例如request:Framed-Protocol。
* 如果列表的名称被省略，则它引用request列表。这就是为什么我们可以在前面的练习中不指定列表名称。
* 以下属性列表可供使用:`request, reply, control, proxy-request, proxy-reply, outer.request, outer.reply, outer.control, outer.proxy-request,` and `outer.proxy-reply`。
* 属性是通过使用update关键字添加或修改的。update关键字与必须修改的列表名称一起使用将创建一个更新部分，可以在其中修改或添加属性。

在介绍了属性和属性列表之后，现在是在实际练习中使用它们的时候了。

## 引用属性

> 在本节中，我们将使用属性。

### `if`语句中的属性

Unlang可以在虚拟服务器定义的各个部分中使用。以前我们在authorize（授权）部分使用过它。您不应按照FreeRADIUS作者的说明在authenticate（身份验证）部分中使用unlang。我们将在post-auth使用unlang来确定是否使用了Auth-Type = PAP，并在确实用于验证用户时给出反馈。

1.编辑FreeRADIUS配置目录下的sites-available/default虚拟服务器，并在post-auth部分的顶部添加以下内容:


```
if(control:Auth-Type == 'PAP'){    update reply {        Reply-Message := "We are using %{control:Auth-Type} authentication"    }}
```

2.在调试模式下重新启动FreeRADIUS，并尝试验证为Alice。指定的回复消息应包含在回复中。

### 刚刚发生了什么

我们已使用unlang测试control属性列表中的Auth-Type AVP的值。如果它等于PAP，我们修改了回复属性列表中的Reply-Message AVP。虽然if语句仅包含五行，但有重要的事情要讨论。我们将讨论以下问题:

* 引用条件中属性的方法
* 比较运算符
* 在属性列表中更改和添加属性

### 引用条件中的属性

引用属性时，我们在条件中使用了另一种语法。其内容如下:

`if("%{control:Auth-Type}" == 'PAP'){`

虽然两者都可以使用，但为了便于阅读，第一种是优选的。替代语法通常用在字符串中。然后将插入属性值以成为字符串的一部分。这称为字符串扩展。我们使用字符串扩展来创建Reply-Message的值。

`Reply-Message := "We are using %{control:Auth-Type} authentication"`

如果省略对属性列表(control:)的引用，则unlang将使用request:Auth-Type。如果此属性不在属性列表中，则unlang返回false。

### 比较运算符

有相当多的比较运算符可用于条件测试。AVP的数据类型将确定哪些操作员可用。

Operator | Data type | Sample
---|---|---== | Strings and numbers |(control:Auth-Type == 'PAP')!= | |(reply:Idle-Timeout != 60)<  |Numbers |(reply:Idle-Timeout <= 60)
<= | |\> | |\>= | |=~| 字符串转换为正则|`(request:User-Name =~ /^.*\.co\.za/i)`!~ |公式 |同上

### 属性操作

Unlang可用于修改AVPs。要修改AVP的值，需要在Unlang中使用update关键字。update语句的概要如下:


```
update <list> {attribute <op> value...}
```

update语句只能包含属性。运算符的值非常重要，因为它将确定如何处理列表中具有该名称的现有属性。我们将在此讨论三个常用运算符。请注意，其他运算符确实存在。有关详细信息，请参阅unlang手册页。

Operator | Description
---|---=  | 将属性添加到列表中，前提是该列表中尚未存在同名的属性:= |将属性添加到列表中。如果该列表中已存在同名的任何属性，则其值将替换为当前属性的值.+= | 将属性添加到列表尾部，即使列表中已经存在同名的属性.

请注意，指定给属性的值必须是正确的类型。当你把字符串值分配给应该采用整数值的属性时，将导致错误。

## 变量

变量不能像在其他语言中那样在unlang中声明。unlang所有属性都是变量，但并非所有变量都是属性。属性在属性列表中作为变量引用之前，必须先将其添加到列表中。对变量的所有引用必须包含在双引号或反引号字符串中。引用此引用字符串中的变量的格式为%{\<variable>}。在上一节中，我们参考了Auth-Type变量，它是一个属性:

`Reply-Message := "We are using %{control:Auth-Type} authentication"`

在本节中，我们将参考不是属性的变量。

### 作为变量的SQL语句

unlang的一个非常强大的功能是它允许您通过SQL模块执行SQL查询。查询实际上是一个变量，此查询的返回值是变量的值。我们现在将修改上一个练习，从数据库中获取时间，并将其添加到Reply-Message 值中。

> 要执行SQL查询，您需要包括并配置FreeRADIUS以使用SQL模块。SQL模块还需要在至少一个部分中使用，例如，authorize(授权)部分或accounting（核算）部分。

1.编辑FreeRADIUS配置目录下的sites-available/default虚拟服务器，并在post-auth部分的顶部添加以下内容:


```
if(control:Auth-Type == 'PAP'){    update reply {        Reply-Message := "We are using %{control:Auth-Type}authentication and the time in the database is now %{sql:SELECT curtime();}"    }}
```

2.在调试模式下重新启动FreeRADIUS，并尝试验证为alice.。指定的Reply-Message应包含在回复中，并应返回数据库中的时间。

### 刚刚发生了什么

我们已使用SQL语句作为变量，并将此语句的结果作为此变量的值返回。

以下是FreeRADIUS服务器的调试输出，指示如何执行SQL状态:


```
++? if (control:Auth-Type == 'PAP')? Evaluating (control:Auth-Type == 'PAP') -> TRUE++? if (control:Auth-Type == 'PAP') -> TRUE++- entering if (control:Auth-Type == 'PAP') {...}sql_xlat    expand: %{User-Name} -> alice
sql_set_user escaped user --> 'alice'    expand: SELECT curtime(); -> SELECT curtime();rlm_sql (sql): Reserving sql socket id: 3sql_xlat finishedrlm_sql (sql): Released sql socket id: 3    expand: We are using %{control:Auth-Type} authentication and thetime in the database is now %{sql:SELECT curtime();} -> We are using PAPauthentication and the time in the database is now 17:48:54+++[reply] returns noop++- if (control:Auth-Type == 'PAP') returns noop
```

如您所见，SQL查询的处理方式与属性的处理方式大致相同。以下关于SQL语句作为变量的几点便于记忆:

* SQL查询应返回单个值。此值是分配给SQL语句变量的值。
* SQL查询可以引用查询本身内部的属性。如果要获取当前用户的总使用量，可以使用以下行:

```
"The total octets is: %{sql: SELECT IFNULL(SUM(AcctInputOctets + AcctOutputOctets),0) FROM radacct WHERE UserName='%{User-Name}';}"
```

* 带引号或带反引号的字符串可以包含SQL查询和属性的组合。这些将被扩展以返回结果。
* 如果SQL模块未在FreeRADIUS中使用，则SQL查询扩展的结果将为空字符串，调试消息将显示错误:

`WARNING: Unknown module "sql" in string expansion "%{sql:SELECTcurdate();}"`
> 扩展字符串的长度最多可达8000个字符。这为相当复杂的SQL语句留下了足够的空间。### 设置变量的默认值
-------


我们并不总是确定变量是否存在。Unlang提供语法功能，以便我们在变量不存在的情况下指定默认值。我们将再次通过修改上一练习来演示这一点。我们将使用radtest首先在请求中包括Framed-Protocol = PPP，并将其排除在外。如果Framed-Protocol = PPP不存在，我们将返回一个默认字符串。

1.编辑FreeRADIUS配置目录下的sites-available/default虚拟服务器，并在post-auth部分的顶部添加以下代码:

```
if(control:Auth-Type == 'PAP'){    update reply {        Reply-Message := "Framed protocol is: %{ %{request:Framed-Protocol}:-Not in request}"    }}
```

2.在调试模式下重新启动FreeRADIUS，并尝试验证作为alice.。首先在radtest命令的末尾添加1，然后省略1。

3.radtest中添加了1的Reply-Message的值应为:

`Reply-Message = "Framed protocol is: PPP"`

4.radtest中省略1的Reply-Message的值应为:

`Reply-Message = "Framed protocol is: Not in request"`

### 刚刚发生了什么

我们已经使用unlang的能力为变量赋值。

变量引用中的`:-`字符序列是一种指示，用于在第一个变量不存在时取消标记，它应尝试使用下面的`:-`字符序列。注意以下要点:

* 如果`:-`后跟未加引号的字符串，则返回此字符串。
* 如果`:-`后面是对另一个变量的引用，则可以创建一个链来最终测试多个变量的存在，例如:`%{ %{request:Framed-Protocol}:-%{request: NAS-Name}:- Default value}`
* 此语法称为条件语法和FreeRADIUS版本之间的更改。某些模块仍然使用旧语法，这会导致调试消息中出现警告。LDAP模块就是一个例子。您可以从更改LDAP模块配置文件中的以下行:


```
filter = "(uid=%{Stripped-User-Name:-%{User-Name}})"to:filter = "(uid=%{ %{Stripped-User-Name}:-%{User-Name}})"
```

### 使用命令替换
-------


到目前为止，我们已经使用双引号来包含将被引用的变量。unlang还具有后引号字符串，这允许命令替换。这些双引号中的字符串的计算方式与可以进行字符串扩展的双引号字符串相似。

让我们修改上一个练习，以显示正在执行的命令替换:

1.编辑FreeRADIUS配置目录下的sites-available/default虚拟服务器，并在post-auth部分的顶部添加以下内容:

```
if(control:Auth-Type == 'PAP'){    update reply {        Reply-Message := `/bin/echo We are using %{control:Auth-Type}`    }}
```

2.在调试模式下重新启动FreeRADIUS，并尝试验证作为alice。echo命令的反馈现在将分配给Reply-Message属性。

### 刚才发生了什么

我们使用了回引号来执行命令替换。此命令的输出已分配给属性。

请注意unlang中有关命令替换的以下几点:

* 命令在FreeRADIUS服务器的sub-shell中执行。在此sub-shell中传递并执行字符串的内容。
* 字符串被拆分为命令和参数。在我们的示例中，命令将是/bin/echo以及参数We are using PAP。
* 指定可执行文件时使用完整路径。
* 可执行文件以运行FreeRADIUS服务器的用户身份运行。
* 您可以在命令替换字符串中包含其他变量。
* 如果该命令无法执行，则会带来问题。这取决于调用的可执行文件的退出代码。卸载测试可执行文件的退出代码，如果不为零，则返回拒绝。拒绝(如本章开头所列)会立即拒绝请求。
* 退出代码和命令输出是两个不同的东西。如果退出代码为零，则将命令输出分配给变量。
* 仅返回并分配多行输出中的第一行。
* 小心使用命令替换，因为它会影响性能。

### 使用正则表达式
-------

unlang允许在条件检查中进行正则表达式计算。这些通常是Posix正则表达式。运算符`=~`和`!~`与正则表达式关联。为了简单地证明概念，我们将修改前面的练习:

1.编辑FreeRADIUS配置目录下的sites-available/default虚拟服务器，并在post-auth部分的顶部添加以下内容：

```
if(request:Framed-Protocol =~ /.*PP$/i){update reply {Reply-Message := "Regexp match for %{0}"}}
```

2.在调试模式下重新启动FreeRADIUS，并尝试验证为Alice。首先在radtest命令的末尾添加1，然后省略1。请注意，当您将1添加到radtest命令时，正则表达式匹配如何更改回复消息的值。

### 刚刚发生了什么

我们已经展示了unlang的正则表达式功能。请注意以下有关unlang中正则表达式的要点:

* 运算符`=~`和`!~`与正则表达式一起使用。
* 正则表达式在2个`/`字符之间指定。
* 正则表达式允许您引用其中的变量，例如`/^%{Framed-Protocol}$/i`。
* 您可以在正则表达式的末尾添加可选的`i`字符，以使搜索不区分大小写。
* 如果匹配，则特殊变量`%{0}`将保存正则表达式中测试的变量的值。

这就结束了unlang导言。现在您应该知道unlang中使用的大多数构造块。下一节将充分利用这些构造块创建unlang的实际应用程序。

## 实用unlang

在前一章第6章“会计”中，我们介绍了sql_counter模块。此模块用于限制用户每天、每周或每月在网络上花费的时间，但是sql_counter在限制用户的数据使用方面存在问题。

### 限制数据使用
-------


要限制用户的每日、每周或每月数据使用，我们必须采取不同的方法。unlang将SQL语句用作变量的能力打开了许多可能性。我们将使用前面提到的运行WISP的Isaac的场景。Isaac现在想限制一个人在一段时间内可以使用的数据量。他利用Mikrotik和Coova Chilli captive portals来控制网络访问，并拥有一个FreeRADIUSRADIU服务器。

### 使用unlang来创建数据计数器

我们首先必须确保某些事情到位，以便这项工作取得成功。下列项目应首先完成作为准备:

* 在字典中定义自定义属性。
* 创建将由FreeRADIUS Perl模块使用的Perl脚本。
* 更新Mikrotik和Chillispot词典。
* 准备users文件。
* 准备SQL数据库。
* 向虚拟服务器添加unlang代码以用作数据计数器。
* 识别LD_PRELOAD错误(如果存在)。

这个练习涉及很多工作。采取分而治之的方法将防止我们被压倒。让我们解决它！

### 定义自定义属性

我们从对unlang的介绍中已经看到，变量主要是通过属性来使用的。我们需要定义一些要在数据计数器中使用的属性。

编辑FreeRADIUS配置目录下的dictionary文件，并添加以下属性定义:


```
ATTRIBUTE FRBG-Reset-Type  3050 stringATTRIBUTE FRBG-Total-Bytes 3051 stringATTRIBUTE FRBG-Start-Time  3052 integerATTRIBUTE FRBG-Used-Bytes  3053 stringATTRIBUTE FRBG-Avail-Bytes 3054 string
```

每个属性以“FRBG”开头。这只是为了将其与其他属性区分开来，并代表FreeRADIUS Beginner's Guide（本书）。

> RADIUS协议只能通过导线传输数值最多255的属性。值大于255的属性在内部使用。

下表列出了每个属性及其含义:

Attribute | Meaning
---|---FRBG-Reset-Type | 指定计数器应重置的时间。值可以是每日、每周、每月或从不。FRBG-Total-Bytes | 用户可以使用的数据量(以字节为单位)，也称为数据上限。FRBG-Start-Time |  指定计数器开始时间的Unix时间戳。FRBG-Used-Bytes | 从开始到现在使用的数据量。FRBG-Avail-Bytes |  可用数据量:`FRBG-Avail-Bytes = FRBG-Total-Bytes – FRBG-Used-Bytes`

如果仔细观察，您会注意到涉及字节的属性都定义为string而不是integer.。我们使用它作为integer类型的32位限制的解决方法。以下小节将更详细地解释32位整数限制。

### 32位限制

半径中的整数值限制为32位。这意味着整型属性的值不能超过`4,294,967,295(2^31-1)`。为了克服这一限制，RADIUS使用千兆字属性，它充当32位属性的 进位 位。例如，Accounting-Request数据包将包含Acct-Input-Octets和Acct-Input-Gigawords以表示大于4,294,967,295的值。

假设我们有一个8.5 GB的值。这可以通过以下方式使用千兆字(Gigaword)进位来定义:

* 8.5 GB的整数值为9,126,805,504字节(8.5 x 1024 x 1024 x 1024)。byte(字节)在网络术语中等于octet(八位字节)。
* 为了计算千兆位（Gigaword）进位的值，我们将9,126,805,504除以4,294,967,296。结果是2.125。`(9126805504 / 4294967296 = 2.125)`。千兆字（Gigaword）进位的值是2。
* 为了计算余数，我们将千兆字进位值乘以4,294,967,296，并从原始进位值中减去该值`(9126805504 - (2 x 4294967296) = 536870912)`。
* 这意味着8.5GB可以按如下方式呈现在计费请求分组中:`Acct-Input-Gigawords = 2, Acct-Input-Octets = 536870912.`。

此32位整数限制仅适用于RADIUS协议。数据库模式已经满足大整数bigint(20)的要求。

> 2.x发行版之前的FreeRADIUS部署将需要进行修改，以纳入对千兆字的支持。

对于32位整数限制，当值大于4,294,967,295时，使用未标记来执行比较变得困难。因此，我们将属性设置为string，并使用perl模块为我们执行比较。perl模块没有32位限制。将属性类型定义为字符串允许以下操作:

* 指定大于4,294,967,295的属性值。
* 允许Perl将字符串值转换为用于计算的数值。这克服了32位限制。

### 使用 perl module
-------


FreeRADIUS允许对节（section.）进行可选命名。这使我们能够为perl模块定义各种命名部分。每个命名部分随后可以用作模块。定义如下:


```
perl <name> {    ...}
```

我们将在modules目录下创建两个命名的perl部分:

1.在FreeRADIUS配置目录下的modules目录中，创建一个名为reset_time的文件，其中包含以下内容:

```
perl reset_time {    module = ${confdir}/reset_time.pl}
```

2.在FreeRADIUS配置目录下的modules目录中，创建一个名为check_usage的文件，其中包含以下内容:

```
perl check_usage {    module = ${confdir}/check_usage.pl}
```

3.这两个Perl部分各自引用一个Perl脚本，该脚本将由perlmodule调用。确保这些Perl脚本位于FreeRADIUS配置目录中。

脚本及其内容在以下各节中列出。

### reset_time.pl

reset_time.pl脚本用于执行以下操作:

* 如果FRBG-Reset-Type的值为daily, weekly, or monthly，则将添加FRBG-Start-Time AVP。
* FRBG启动时间的值是计数器启动时的Unix时间。
* 如果添加了FRBG-Start-Time AVP，则脚本的返回代码将具有更新的值。
* 如果FRBG-Start-Time AVP没有添加(FRBG-Reset-Type = never)，则脚本的返回代码将具有noop值。

以下是reset_time.pl的内容:


```
#! /usr/bin/perl -wuse strict;use POSIX;
# use ...# This is very important !use vars qw(%RAD_CHECK);use constant RLM_MODULE_OK=>        2;# /* the module is OK,continue */use constant RLM_MODULE_NOOP=>      7;use constant RLM_MODULE_UPDATED=>   8;# /* OK (pairs modified) */
sub authorize {    #Find out when the reset time should be    if($RAD_CHECK{'FRBG-Reset-Type'} =~ /monthly/i){        $RAD_CHECK{'FRBG-Start-Time'} = start_of_month()    }    if($RAD_CHECK{'FRBG-Reset-Type'} =~ /weekly/i){            $RAD_CHECK{'FRBG-Start-Time'} = start_of_week()    }    if($RAD_CHECK{'FRBG-Reset-Type'} =~ /daily/i){        $RAD_CHECK{'FRBG-Start-Time'} = start_of_day()    }    if(exists($RAD_CHECK{'FRBG-Start-Time'})){        return RLM_MODULE_UPDATED;    }else{        return RLM_MODULE_NOOP;    }}
sub start_of_month {    #Get the current timestamp;    my $reset_on = 1; #you decide when the monthly CAP will reset    my $unixtime;    my ($sec,$min,$hour,$mday,$mon,$year,$wday,$yday,$isdst)=localtime(time);    if($mday < $reset_on ){        $unixtime = mktime (0, 0, 0, $reset_on, $mon-1, $year, 0, 0);#We use the previous month    }else{        $unixtime = mktime (0, 0, 0, $reset_on, $mon, $year, 0, 0);#We use this month    }    return $unixtime;}sub start_of_week {    #Get the current timestamp;    my ($sec,$min,$hour,$mday,$mon,$year,$wday,$yday,$isdst)=localtime(time);    #create a new timestamp:    my $unixtime = mktime (0, 0, 0, $mday-$wday, $mon, $year, 0, 0);    return $unixtime;}
sub start_of_day {    #Get the current timestamp;
    my ($sec,$min,$hour,$mday,$mon,$year,$wday,$yday,$isdst)=localtime(time);    #create a new timestamp:    my $unixtime = mktime (0, 0, 0, $mday, $mon, $year, 0, 0);    return $unixtime;}
```

### check_usage.pl

check_usage.pl脚本用于执行以下操作:

* 添加reply属性以指定用户的可用字节。这包括千兆字（Gigaword）值的计算。
* 如果添加了reply属性，请将返回代码指定为updated。
* 如果数据使用率超过分配的部分，则通过将返回代码指定为reject并添加Reply-Message来reject请求。

check_usage.pl脚本用于执行以下操作:


```
#! usr/bin/perl -wuse strict;# use ...# This is very important!use vars qw(%RAD_CHECK %RAD_REPLY);use constant RLM_MODULE_OK=> 2;# /* the module is OK,continue */use constant RLM_MODULE_UPDATED=> 8;# /* OK (pairs modified) */use constant RLM_MODULE_REJECT=> 0;# /* immediately reject the request */use constant RLM_MODULE_NOOP=> 7;my $int_max = 4294967296;
sub authorize {    #We will reply, depending on the usage    #If FRBG-Total-Bytes is larger than the 32-bit limit we have to set a Gigaword attribute    if(exists($RAD_CHECK{'FRBG-Total-Bytes'}) && exists($RAD_CHECK{'FRBG-Used-Bytes'})){        $RAD_CHECK{'FRBG-Avail-Bytes'} = $RAD_CHECK{'FRBGTotal-Bytes'} - $RAD_CHECK{'FRBG-Used-Bytes'};    }else{        return RLM_MODULE_NOOP;    }
    if($RAD_CHECK{'FRBG-Avail-Bytes'} <= $RAD_CHECK{'FRBG-Used-Bytes'}){        if($RAD_CHECK{'FRBG-Reset-Type'} ne 'never'){            $RAD_REPLY{'Reply-Message'} = "Maximum $RAD_CHECK{'FRBG-Reset-Type'} usage exceeded";        }else{            $RAD_REPLY{'Reply-Message'} = "Maximum usage exceeded";        }        return RLM_MODULE_REJECT;    }
    if($RAD_CHECK{'FRBG-Avail-Bytes'} >= $int_max){        #Mikrotik's reply attributes        $RAD_REPLY{'Mikrotik-Total-Limit'} = $RAD_CHECK{'FRBGAvail-Bytes'} % $int_max;        $RAD_REPLY{'Mikrotik-Total-Limit-Gigawords'} = int($RAD_CHECK{'FRBG-Avail-Bytes'} / $int_max );        #Coova Chilli's reply attributes        $RAD_REPLY{'ChilliSpot-Max-Total-Octets'} = $RAD_CHECK{'FRBG-Avail-Bytes'} % $int_max;        $RAD_REPLY{'ChilliSpot-Max-Total-Gigawords'} = int($RAD_CHECK{'FRBG-Avail-Bytes'} / $int_max );    }else{        $RAD_REPLY{'Mikrotik-Total-Limit'} = $RAD_CHECK{'FRBGAvail-Bytes'};        $RAD_REPLY{'ChilliSpot-Max-Total-Octets'} = $RAD_CHECK{'FRBG-Avail-Bytes'};        }    return RLM_MODULE_UPDATED;
}
```

由于Isaac使用了Mikrotik和Coova Chilli，我们将它们的AVPs用于限制总数据。请参阅NAS的文档，以确定它是否支持数据限制，以及应使用哪些AVPs进行限制。

### 在CentOS上安装perl模块
-------

FreeRADIUS perl模块是被单独打包的。请确保已安装此软件包。SUSE和Ubuntu已经在FreeRADIUS的标准安装中包含了perl模块，不过还有一个额外的错误，我们稍后将删除。

### 更新dictionary文件
-------

FreeRADIUS包括各种供应商的字典文件。我们希望返回的属性是这些供应商以后开发的一部分，不包括在FreeRADIUS标准字典文件中。我们必须通过将它们添加到现有字典文件中来包括它们。供应商的dictionary文件通常位于/usr/share/freeradius目录下。如果使用“配置、制作、安装（configure,make, make install）”模式安装FreeRADIUS，则安装模式将位于/usr/local/share/freeradius下。

查找供应商词典的位置，并将以下属性添加到相应的词典中:

dictionary.chillispot

```
ATTRIBUTE ChilliSpot-Max-Input-Gigawords 21 integerATTRIBUTE ChilliSpot-Max-Output-Gigawords 22 integerATTRIBUTE ChilliSpot-Max-Total-Gigawords 23 integer
```
dictionary.mikrotik

```
ATTRIBUTE Mikrotik-Total-Limit 17 integerATTRIBUTE Mikrotik-Total-Limit-Gigawords 18 integer
```

我们刚才所做的更改是对供应商(VSAs)属性的更改，并且可以在RADIUS数据包中通过导线传输。VSAs不同于我们之前添加到主dictionary文件中的属性，这些属性由FreeRADIUS服务器内部使用。

请记住，使用最新版本的Mikrotik和Coova Chilli将确保支持这些属性。还要查看是否需要包括最新版本中引入的其他属性。

### 推荐的词典更新方式

本练习不遵循更新字典文件的建议方法。本书后面有一章专门介绍词典，其中介绍了使词典保持最新的推荐方法。

启动时词典的收录方式也将在词典一章中详细介绍。简而言之，FreeRADIUS配置目录下的主dictionary文件包括名为`/usr/share/freeradius/dictionary`的文件。此文件又包括位于`/usr/share/freeradius/`下的各种`.dictionary`文件。

### 准备users文件

我们将使用users文件作为用户存储。在现实世界中，您可以使用其他来源，例如SQL数据库或LDAP目录。

确保用户文件包含以下Alice条目:

```
"alice" Cleartext-Password := "passme",FRBG-Total-Bytes :='9126805504',FRBG-Reset-Type := 'monthly'Reply-Message = "Hello, %{User-Name}"
```

这将限制每月使用8.5 GB数据。

### 准备数据库

虽然我们在用户文件中定义了Alice，但我们将使用SQL进行核算。按照以下步骤准备数据库:

1.确保您具有第5章用户名和密码源中指定的工作SQL配置。

2.我们将不使用SQL作为用户存储。确认在FreeRADIUS配置目录下sites-enabled/default文件的authorize部分中禁用(注释掉)了SQL。

3.要使我们的数据计数器成功运行，请确保在FreeRADIUS配置目录下的sites-enabled/default文件的accounting部分中启用(取消注释) SQL。

4.使用MySQL客户端程序并清理数据库中可能仍然存在的任何以前的会计详细信息:

```
$>mysql -u root -p radiusdelete from radacct;
```

5.在本练习的后面部分，我们将使用radclient程序模拟会计。确保您创建了第6章“会计”中指定的

`4088_06_acct_start.txt` 和`4088_06_acct_stop.txt` 文件。

您可能会担心SQL模块也有32位整数限制。幸运的是，在FreeRADIUS的较新版本中，这一切都得到了解决。FreeRADIUS负责在更新SQL数据库之前将千兆位进位AVP与八位AVP相结合。MySQL的SQL模式还使用bigint(20)存储八位字节值，这足以存储大量数据。

### 向虚拟服务器添加unlang代码

到目前为止，所有准备工作都已完成，我们向虚拟服务器定义添加一段unlang代码。在FreeRADIUS配置目录下sites-enabled/default文件的authorize（授权）部分的daily条目下方添加以下代码。

```
if((control:FRBG-Total-Bytes)&&(control:FRBG-Reset-Type)){    reset_time    if(updated){    # Reset Time was updated,                    # we can now use it in a query        update control {        #Get the total usage up to now:        FRBG-Used-Bytes := "%{sql:SELECT IFNULL(SUM(acctinputoctets - GREATEST((%{control:FRBG-Start-Time} - UNIX_TIMESTAMP(acctstarttime)), 0))+SUM(acctoutputoctets -GREATEST((%{control:FRBG-Start-Time} - UNIX_TIMESTAMP(acctstarttime)), 0)),0) FROM radacct WHERE username='%{request:User-Name}' AND UNIX_TIMESTAMP(acctstarttime) + acctsessiontime > '%{control:FRBG-Start-Time}'}"            }        }        else{            #Asumes reset type = never            #Get the total usage of the user            update control {                FRBG-Used-Bytes := "%{sql:SELECT IFNULL(SUM(acctinputoctets)+SUM(acctoutputoctets),0) FROM radacct WHERE username='%{request:User-Name}'}"            }        }        #Now we know how much they are allowed to use and the usage.        check_usage}
```

数据计数器现在完成。

### 测试数据计数器

检验真理的时刻到了。现在是测试数据计数器的时候了。如果您使用SUSE或Ubuntu，请记LD_PRELOAD。CentOS没有这个问题。

1.在调试模式下重新启动FreeRADIUS；如果需要，设置LD_PRELOAD环境变量。我们假设Ubuntu在这里。请更改以适合您的分发:

`#>LD_PRELOAD=/usr/lib/libperl.so /usr/sbin/freeradius -X`

2.尝试使用radtest命令验证为alice:

`$>radtest alice passme 127.0.0.1 100 testing123`

3.您应该在FreeRADIUS的答复中获得以下AVPs:

```
ChilliSpot-Max-Total-Gigawords = 2ChilliSpot-Max-Total-Octets = 536870912Mikrotik-Total-Limit-Gigawords = 2Mikrotik-Total-Limit = 536870912Reply-Message = "Hello, alice"
```

4.将radtest与4088_06_acct_start.txt和4088_06_acct_stop.txt文件结合使用，模拟一些会计核算:

```
$>radclient 127.0.0.1 auto testing123 -f 4088_06_acct_start.txt$>radclient 127.0.0.1 auto testing123 -f 4088_06_acct_stop.txt
```

5.尝试使用radtest命令再次验证为Alice。您现在应该在回复中获取不同的值，以显示剩余数据是如何耗尽的:

```
ChilliSpot-Max-Total-Gigawords = 2ChilliSpot-Max-Total-Octets = 536866638Mikrotik-Total-Limit-Gigawords = 2Mikrotik-Total-Limit = 536866638
```

重复几次记帐模拟和身份验证测试周期。注意可用字节如何随着每个周期而减少。

### 清理

要以正确的方式完成本练习，您可以执行以下挑战:

* 在policy.conf文件中定义策略。内容将是我们添加到default文件中的unlang代码。用新创建策略替换代码。
* 将LD_PRELOAD环境变量添加到SUSE和Ubuntu上的FreeRADIUS启动脚本中。

## 摘要

授权可以成为FreeRADIUS中最复杂的部分。通过充分利用unlang提供的服务，我们可以克服几乎所有可以想象的问题。

在本章中，我们介绍了:

* 限制的应用:可以在RADIUS服务器或NAS设备上应用限制。
* Unlang : Unlang是一种强大的处理语言，它允许我们操纵FreeRADIUS处理传入请求的方式。它具有可以控制请求流的条件检查。它还允许与某些模块(如SQL模块)交互，以从SQL数据库中获取结果。unlang使我们能够操作和添加将随访问接受数据包返回的AVPs。任何想要在FreeRADIUS中创建灵活多样的配置的人都应该掌握Unlang的使用。

随着本章关于授权的结束，我们现在已经完成了AAA框架的覆盖。本书的其余部分将集中讨论更高级的RADIUS主题，以及特定于自由RADIUS的主题。

> 翻译于2018/2/19日完成


