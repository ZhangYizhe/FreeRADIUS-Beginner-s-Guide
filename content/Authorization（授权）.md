# Authorization（授权）

> 授权是评估请求中信息的过程。此信息可用于对照从文件、数据库或LDAP目录获得的关于用户的信息进行验证。授权发生在身份验证之前，不涉及密码检查。我们可以使用各种逻辑和比较来确定用户是否被授权连接到网络。我们还可以确定他们使用网络的时间或服务质量。这些都是授权的组成部分，本章将对此进行讨论。

在本章中，我们将:

* 查看如何对用户应用限制
* 查看FreeRADIUS如何执行授权
* FreeRADIUS下的unlang处理语言探讨
* 使用unlang创建数据重置计数器

那么，让我们继续吧...

### 实施限制

授权本质上是关于限制的。基于某些检查，可以限制用户。限制可应用于以下两个位置之一:

* 在RADIUS服务器上
* 在NAS上

当Access-Request数据包发送到RADIUS服务器时，在身份验证过程中确定限制。Accounting-Request数据包不会也无法确定限制。

在RADIUS服务器上应用限制时，服务器返回访问拒绝数据包，其中应包括指定拒绝原因的Reply-Message AVP。

在NAS应用限制时，RADIUS服务器返回一个访问接受数据包，其中包括NAS应应用的AVPs。这意味着您必须确保NAS收到正确的AVPs以实施限制，并且它首先还支持这些AVPs。

### 在FreeRADIUS授权

这一部分可以看作是我们到目前为止所涉及的主题的概述，也可以看作是在获得授权进行更多实践练习之前的复习。

* 请求从NAS (客户端)发送到FreeRADIUS (服务器)。
* 这些请求由在FreeRADIUS配置中定义的虚拟服务器处理。默认虚拟服务器称为默认服务器。
* 处理传入请求的方式取决于虚拟服务器文件内各个部分的配置。
* 请求本身按逻辑顺序由各个部分处理。授权部分始终在验证部分之前处理访问请求数据包。preacct部分同样总是在accounting部分之前处理Accounting-Request包。
* 虽然不能更改顺序，但我们在其中有很大的灵活性来处理请求。

下一部分将介绍处理请求的方法。通过使用比较和逻辑，我们可以控制请求的流程和操作属性。

### unlang介绍

FreeRADIUS中提供的unlang语言在授权到新高度时具有灵活性。unlang不是一种成熟的编程语言，而是一种处理语言。unlang的目的是实现策略，而不是替换使用Perl或Python创建的复杂脚本。unlang坚持一个基本语法，包括条件语句和变量操作。unlang代码不会编译，而是由FreeRADIUS服务器解释。解释发生在服务器读取配置文件时，这通常发生在启动期间。unlang标记的使用仅限于配置文件中的指定部分，不能在模块中使用。

unlang的一个关键特性是能够使用条件语句来控制处理请求的进程。

> FreeRADIUS为unlang安装了一个手册页，您可以参考:
> `$>man unlang`

我们将演示如何使用各种条件语句来显示如何处理请求。

### 使用条件语句

条件语句很简单，但功能强大，以至于它们仍然是任何软件的构建块。unlang功能有两种实现条件检查的方法:

1. `if`语句。这包括作为语句一部分的`else`和`elsif`选项。
2. `switch`语句

本节将介绍`if`语句的各种用途。

#### 使用unlang中的`if`语句

if语句本身不是非常复杂。它具有以下格式:

```
if(condition){...}
```

条件部分由于其多种可能性而变得复杂。

#### 使用if语句获取返回代码

现在，我们将查看模块的返回代码，并使用此代码与指定的条件进行比较。FreeRADIUS中的每个模块都需要在调用后返回代码。

此代码的值随后可以用作if语句中的条件检查。

#### 使用if语句授权用户

如果用户不在users文件中，本练习将使用if条件拒绝访问请求。

编辑FreeRADIUS配置目录下的sites-available/default虚拟服务器，并在authorize部分的files条目下添加以下行:

```
if(noop){reject}
```

在调试模式下重新启动FreeRADIUS，并尝试使用users文件中不存在的用户名和密码进行身份验证。我们假设ali在任何地方都没有定义:

`radtest ali passme 127.0.0.1 100 testing123`

您应该得到一个拒绝访问数据包。

#### 刚才发生了什么？

我们在authorize部分的files模块下添加了条件检查。要激活此更改，必须重新启动FreeRADIUS。这将导致radius服务器读取和解释unlang代码。然后通过向服务器发送访问请求分组来测试unlang代码。

我们将首先查看if条件，然后查看满足if条件时采取的操作。

#### 模块返回码

如果查看FreeRADIUS的调试输出，您将看到每个模块如何返回代码。下面列出了可用的返回代码及其含义:

Module return code | Description
---|---notfound |Information was not found（未找到信息）noop |The module did nothing（模块什么也没做）ok |The module succeeded（模块成功）updated |The module updated the request （For example, it set the Auth-Type internal AVP）（模块更新了请求。“例如，它设置授权类型内部AVP”）fail |The module failed（模块失败）reject |The module rejected the request（模块拒绝了请求）userlock |The user was locked out（用户被锁定）
invalid |The configuration was invalid（配置无效）handled |The module handled the request itself（模块本身处理请求）

我们可以使用unlang的if语句来测试模块中指定的返回代码。为此，我们必须向unlang提示它必须测试模块返回代码。因此，我们在if语句的条件下将要测试的返回代码指定为非引号字符串。

> 如果条件是未加引号的字符串，并且是上表中列出的模块返回代码之一，则unlang会将此字符串与最新模块的返回代码进行比较。

你可能认为我们应该测试notfound而不是noop。files模块在用户不在users文件中时返回noop而不是notfound。在创建条件测试之前，请记住在某些情况下测试或找出模块返回的值。如果你做不到这一点，结果可能与你预期的不同。

#### unlang关键词

当满足if条件时，我们可以采取各种措施。unlang使用关键字处理请求。if语句就是这样一个关键字。还有update关键字，在处理属性时使用，本章稍后将讨论。然后还有一个关键字列表，可以在if语句中使用。如果用户不在users文件中，我们使用reject关键字立即拒绝请求。下表列出了可在if语句中使用的关键字及其对请求的影响。

Keywords | Description
---|---noop | Do nothing.ok |Instructs the server that the request was processed properly.This keyword can be used to over-ride earlier failures, if the local administrator determines that the failures are not catastrophic.（指示服务器请求已正确处理。如果本地管理员确定故障不是灾难性的，则此关键字可用于忽略早期故障。）fail |  Causes the request to be treated as if a failure had occurred.（使请求被视为已发生故障。）reject | Causes the request to be rejected immediately.（导致请求立即被拒绝。）

请注意，虽然名称与模块的返回代码相同，但模块返回代码与这些关键字之间存在差异。这些关键字是unlang的一部分，并在if语句中使用。如果if语句中没有定义关键字，则默认情况下它将返回noop。

除了这些关键字，我们还可以指定任何FreeRADIUS模块的名称。模块名称被视为关键字。`if`语句也可以嵌套。

### 使用条件语句的其他测试

条件语句为我们提供了多种测试功能。例如，在授权过程中可以使用这些属性来检查NAS是否为Access-Request提供了必需的属性。我们甚至可以使用逻辑运算符组合测试来创建在授权用户之前必须满足的复杂条件。本节将介绍另外两个条件测试，它们可以用作构建块来创建灵活的授权策略。

这些练习假定为没有动过sites-available/default文件，并且可以独立完成。我们还假设users文件包含一个名为Alice的用户，密码为passme (这与前面所有章节中定义和使用的用户相同)。

#### 检查属性是否存在

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

#### 使用逻辑表达式对用户进行身份验证

unlang还支持条件语句中的逻辑AND (`&&`)和逻辑OR (`||`)。在本练习中，我们将拒绝不在users文件中或请求中存在Framed-Protocol AVP的用户。

