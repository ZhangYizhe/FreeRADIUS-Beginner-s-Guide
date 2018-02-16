# 用户名和密码的来源（Sources of Usernames and Passwords）

在已有的章节中，用户的详细信息被保存在用户文件中。然后，FreeRADIUS使用该文件的内容来验证身份验证过程中的凭证。FreeRADIUS很可能是企业设置的一部分，现有用户已经在其他地方创建了。本章将讨论如何利用现有的用户存储。

在本章中，我们将：

1. 查看用户存储选项
2. 使用Linux系统用户作为FreeRADIUS的用户存储
3. 使用MySQL作为FreeRADIUS的用户存储
4. 使用LDAP作为FreeRADIUS的用户存储
5. 使用Microsoft Active Directory作为FreeRADIUS的用户存储

让我们开始吧。

### 用户存储

用户存储是保存用户详细信息的地方。最好有一个单独的用户存储，使用不同的系统来使用这个单一的源。企业对此类存储的需求导致了目录的出现。Novell的eDirectory、Microsoft的Active Directory和OpenLDAP都是目录的示例。

在WWW空间中，像Google和Facebook这样的流行的web环境允许第三方通过web服务使用他们的用户存储。这使得外部web应用程序可以使用它们进行身份验证。

FreeRADIUS允许我们合并外部用户存储。这减少了管理用户和密码所涉及的管理开销。下面的示意图显示了配置FreeRADIUS时的不同可能性:

![配置FreeRADIUS](https://github.com/ZhangYizhe/FreeRADIUS-Beginner-s-Guide/blob/master/images/Sources_of_Usernames_and_Passwords.png)

FreeRADIUS有两种方法可以利用用户存储进行身份验证:

1. 在FreeRADIUS模块的帮助下阅读存储的内容。然后，其他FreeRADIUS模块可以使用这些内容。例如，pap模块使用sql模块提供的信息进行密码验证。
2. 通过将用户的凭证发送到FreeRADIUS模块或程序来对用户进行身份验证，从而与用户存储进行交互。ldap模块在Novell eDirectory中执行此工作。mschap模块使用ntlm_auth程序与Microsoft Active Directory进行交互。

本章的其余部分将会亲自动手，看看不同的用户商店如何被整合到FreeRADIUS中。

### 系统用户

在FreeRADIUS正在运行的服务器上的系统用户可以用作用户存储。

系统用户通常与/etc/password、/etc/shadow和/etc/group文件相关联。

Linux机器还可以使用诸如NIS和LDAP之类的其他方法，从而允许系统用户的中心位置更加集中。不过，本节将重点讨论在服务器上本地定义的系统用户。

### 采取行动的时间——在FreeRADIUS中加入Linux系统用户

FreeRADIUS文档推荐它作为一个非特权用户运行。当我们将系统用户作为用户存储时，这个非特权用户将需要访问/etc/shadow、file。这三个发行版中的每个都有不同的默认配置，关于/etc/影子文件的权限和所有权。

### 权限准备

默认情况下，Ubuntu有正确的/etc/阴影文件的权限。在Ubuntu中，/etc/影子文件是由名为影子的组所拥有的，它已经读取了该文件的权限。当FreeRADIUS安装时，它添加了一个名为freerad的用户和组。用户freerad被添加到影子组，允许freerad读取/etc/影子。

您可以使用下面的命令在Ubuntu上确认这一点。检查/etc/阴影文件的所有权:

**「……」文件方式存储暂不翻译。**


-------
### MySQL作为用户存储

FreeRADIUS可以连接到一个SQL数据库来检索用户的详细信息。FreeRADIUS SQL模块成对工作。一个通用的SQL模块使用特定的数据库模块与数据库进行交互。这允许对不同的数据库进行简单的支持。

正如文件模块使用用户文件来检索授权和身份验证的信息一样，通用SQL模块也使用特定的数据库模块从数据库中检索相同类型的信息。

MySQL是一个非常流行的开放源码数据库。尽管人们对Oracle的未来有了猜测，但它仍然是一个值得信赖的数据库，数以百万计的人依赖它。

MySQL很容易配置，大多数人都很熟悉它。使用MySQL的FreeRADIUS部署的数量超过了FreeRADIUS部署的任何其他数据库。我们将遵循这一趋势，并将向您展示如何将MySQL数据库作为用户存储。

#### 在FreeRADIUS中合并MySQL数据库

我们假设MySQL还没有安装在您使用FreeRADIUS的系统上。我们将首先安装然后配置MySQL，以便它可以用于FreeRADIUS。

Distribution | Command to install MySQL server
---|---
CentOS | yum install mysql-server
SUSE | zypper install mysql
Ubuntu | sudo apt-get install mysql-server

MySQL服务器有一个名为root的用户，默认情况下，本地机器上没有任何密码。强烈建议您为该用户提供密码。

##### 请注意以下几点:

CentOS:

* 可能已经安装了mysql-client包。
* 在MySQL服务器安装之后，需要第一次启动它。
* 使用命令/etc/init.d / mysqld开始。反馈消息有关于如何为根用户添加密码的说明。
* 通过使用/sbin/chkconfig mysqld，确保MySQL服务器在重新启动后启动。

现在，我们可以继续为FreeRADIUS安装MySQL模块(如果需要的话)，并为FreeRADIUS准备一个数据库。

#### FreeRADIUS的MySQL安装包

CentOS和Ubuntu有独立的FreeRADIUS包，其中包含针对MySQL的特定sql模块(rlm_sql_mysql)。使用下面的表格来安装它们:

Distribution | Command to install FreeRADIUS's MySQL package
---|---
CentOS| **yum install freeradius2-mysql** Or **yum --nogpgcheck install freeradiusmysql-2.1.10-1.i386.rpm** (if built from source)
Ubuntu | **sudo apt-get install freeradius-mysql** Or **sudo dpkg -i freeradius-mysql_2.x.y+git_i386.deb** (if built from source)

请记住，您必须安装FreeRADIUS的MySQL包，这是已经安装的FreeRADIUS构建的一部分。您不能从源代码构建和安装最新的FreeRADIUS，然后期望由包管理器引用的旧包将安装。由于这个原因，我们区分了前一个表中的两个。

#### 准备数据库

FreeRADIUS提供所有需要的文件来准备一个数据库供其使用。

FreeRADIUS配置目录包含一个名为sql的子目录。在sql子目录下是FreeRADIUS支持的各种数据库的子目录。如果只有一个MySQL目录，那是因为支持其他数据库的FreeRADIUS包没有安装。


1. 要创建名为radius的数据库，需要发出以下命令: `mysqladmin -u root -p create radius`
2. 要创建具有正确权限的管理员用户，请使用admin.sql件作为一个模板，并在radius数据库中运行它。您被鼓励更改默认值。使用下面的命令: `mysql - u root - p < /etc/raddb/sql/mysql/admin.sql`
3. 使用模式为数据库创建schema.sql文件，使用以下命令: `mysql -u root -p radius < /etc/raddb/sql/mysql/schema.sql`
4. 将Bob作为测试用户添加到数据库中。

```
mysql -u root -p radius
INSERT INTO radcheck (username, attribute, op, value) VALUES('bob', 'Cleartext-Password', ':=', 'passbob');
INSERT INTO radreply (username, attribute, op, value) VALUES('bob', 'Reply-Message', '=', 'Hello Bob!');
```

如果您是MySQL的新手，有一个叫做`mysqlshow`的方便命令，您可以使用它来获得快速的信息。要获得可以使用的数据库列表：`mysqlshow –u root`

然后，您可以通过向mysqlshow命令添加参数(例如，在表中添加参数，从而进一步深入数据库、表和列中。`mysqlshow –u root radius radcheck`)。使用该命令来确认radius数据库的创建及其表。

#### 配置FreeRADIUS

SQL模块打破了FreeRADIUS的传统，其中一个modules的配置位于FreeRADIUS配置目录下的模块子目录下。

#### 连接信息

`sql.conf`位于FreeRADIUS配置目录中。conf文件包含连接到数据库的所有配置选项。如果您使用了默认值，那么您就不需要在这个文件中更改任何内容。但是，鼓励您浏览这个文件的内容，以便更好地理解可以指定的各种指令。这还有助于对前面步骤中使用的值进行双重检查和确认。

#### 包含SQL配置

为了让FreeRADIUS在启动时包含SQL模块，请在radiusd.conf中对以下行进行取消注释:

`#$INCLUDE sql.conf`

#### 虚拟服务器

如前所述，每个虚拟服务器都包含主要部分。要使用SQL模块作为用户存储，请在启用了`sites-enabled/default`的`authorize`部分中对SQL行进行取消注释。

如果您仍然有unix部分未从上一个练习中得到注释，请再次禁用它。如果不这样做，将导致FreeRADIUS使用系统用户的详细信息对bob进行身份验证。

#### 测试MySQL用户存储

所有的东西都已经配置好并准备好进行测试。按照以下步骤测试用户存储：


1. 在调试模式下重新启动FreeRADIUS。扫描调试输出并检查rlm_sql反馈。这表明包含了SQL模块。
2. 使用radtest程序进行身份验证
`radtest bob passbob 127.0.0.1 100 testing123`
3. 观察FreeRADIUS服务器的输出，以查看rlmsql如何处理请求。


#### 刚才发生了什么?

我们已经配置并添加了一个MySQL数据库来作为FreeRADIUS的用户存储。让我们来看看一些有趣的观点。

正如本章开头所述，FreeRADIUS有两种使用数据存储的方式。使用MySQL数据库，它从数据库中读取用户的信息。然后，这些数据可以被PAP模块用于密码验证模块。
    
FreeRADIUS不针对数据库进行身份验证，而是使用数据库作为存储用户数据的存储。数据库可以作为 users 文件的替换或替代。

如果用户的详细信息不是来自MySQL数据库，那么请确认您是否已经禁用了authorize部分中的unix模块，并将其用于调试输出以获得更多信息以定位问题。

#### 与平面文件相比，SQL的优势

将用户的详细信息存储在一个SQL数据库中比将它们存储在一个平面文件中有很多优势。以下是一些优势:

* **可伸缩**:
数据库可以位于另一个服务器上，不需要在FreeRADIUS服务器上。
* **用户友好**:
有很多基于web的软件可以管理数据库中的数据。
* **灵活**:
可以在不需要重新启动FreeRADIUS的情况下添加或删除用户和属性。
* **可管理**:
可以将用户分配给一个或多个组，以管理公共属性。使用概要文件也是可能的。
* **安全**:
敏感信息可以使用内置函数进行散列和加密，这些功能通常是SQL数据库引擎的一部分。

#### SQL数据库的其他用途

SQL数据库不仅用于在FreeRADIUS中存储用户详细信息。额外的功能包括:

* **会计**——我们可以将用户的会计信息写入数据库，而不是平面文件。
* **使用控制**——rlm_sqlcounter模块允许定义各种计数器(基于时间或数据)，以跟踪用户的使用情况。
* **NAS设备**——默认情况下，clients.conf文件中定义的NAS设备也可以存储在数据库表中。
* **IP池管理**——向数据库中添加额外的表允许我们在数据库的帮助下管理IP leases。
    
核算和使用控制在本书后面的一章中被介绍。

### 重复用户

FreeRADIUS允许不同的用户存储共存，但是当在不同的用户存储中定义相同的用户时会发生什么情况？

这取决于模块在`authorize`部分中列出的顺序。最后一个模块的用户详细信息将由身份验证部分使用。

不幸的是，此规则并不适用于所有模块。unix模块始终将其重复用户的详细信息发送到`authenticate`（身份验证）部分，而不管`authorize`（授权）部分中的顺序如何。

使用SQL数据库中定义的默认顺序，用户将“赢得”`files`文件中定义的顺序。

如果您希望users文件中定义的用户“赢得”sql数据库中的副本，则files模块应列在sql模块之后。

### 数据库模式

SQL数据库包含与files模块使用的users文件相同类型的详细信息。就像用户文件一样，我们有check和reply项目。这些项目分别存储在radcheck和radreply表中。

### 组

SQL数据库还允许我们定义组的检查和答复属性。它们分别存储在radgroupcheck和radgroupresply表中。

现在可以将用户分配给零个或多个定义的组。组通过radusergroup表分配。此表中的条目指定某个组对用户的优先级。这允许具有较高优先级(较小值)的组中的某些值覆盖具有较低优先级(较大值)的组中的值。

考虑到这一点，让我们看一些实际例子。

#### 探索群组使用

本节介绍SQL数据库的更高级方面。我们将通过实际练习介绍以下内容:

* 1. 组分配
* 2. 使用Fall-Through内部AVP
* 3. 使用User-Profile内部AVP进行配置文件分配

#### 使用SQL组

在本练习中，我们将把bob添加到students组中。students组具有一个check属性，用于测试Access-Request是否包含具有PPP值的Framed-Protocol AVP。如果AVP存在且正确，我们将返回答复AVP :

1.登录radius MySQL数据库并发出以下SQL命令以创建所需条目:


```delete from radcheck;delete from radreply;delete from radgroupreply;delete from radgroupcheck;delete from radusergroup;INSERT INTO radcheck (username, attribute, value,op)VALUES('bob', 'Cleartext-Password', 'passbob',':=');INSERT INTO radreply (username, attribute,value,op)VALUES('bob', 'Reply-Message','Hello Bob!','=');INSERT INTO radgroupreply (groupname, attribute,value,op) VALUES('students', 'Reply-Message', 'Hello PPP protocol!',':=');INSERT INTO radgroupreply (groupname, attribute,value,op) VALUES('students', 'Session-Timeout', '900',':=');
INSERT INTO radgroupcheck (groupname, attribute,value,op) VALUES('students', 'Framed-Protocol', 'PPP','==');INSERT INTO radusergroup (username, groupname,priority) VALUES('bob', 'students', 10);
```
2.使用radtest程序验证为bob，但最后添加1。这将导致Access-Request分组包括Framed-Protocol = PPP的AVP :

`radtest bob passbob 127.0.0.1 100 testing123 1`

3.您应该获得一个Access-Accept包，其中包含Reply-Message的消息：Reply-Message = "Hello PPP protocol!”。

4.再次使用radtest程序对bob进行身份验证，但这次取消1，你将得到带有一条信息为Reply-Message = "Hello Bob!”的Reply-Message的Access-Accept 包。

您可能已经预料到，当Framed-Protocol AVP缺少或不同的值时，请求将被拒绝。Radgroupcheck的工作方式与radcheck不同，如下方式:

* 当在radcheck中定义了检查属性且该属性不匹配时，请求失败，并返回一个Access-Reject。
* 当在radgroupcheck中定义了检查属性且该属性不匹配或
丢失时，请求通过，但是在radgroupreply中返回的reply属性没有返回。

这种行为使一个用户可以属于多个组，并且根据radgroupcheck属性传递的不同，该组的reply属性将返回给应答。

> 作为对op字段值的提示，您可以坚持以下规则，直到我们进入授权的章节。
> 应答项包含=和检查条目，需要匹配传入的AVPs使用==，而其他则使用:=。如果您想要一个应答项覆盖现有的一个，则使用:=。

radgroupcheck项按逻辑进行排序。第一个失败将导致组不返回任何reply属性。

#### 控制组的使用

默认情况下，如果用户在radcheck表中，SQL模块将检查是否有分配给该用户的组。可以通过两种方式控制此行为:

1. 若要全局关闭此功能，请在sql.conf文件中将read_groups指令的值设置为“no”。
2. 要为单个用户再次激活已分配组的检查，请在radreply表中指定“Fall-Through = Yes”。

首先将read_groups设置为“no”，然后重新启动FreeRADIUS并以Bob身份进行身份验证。即使在将1添加到radtest命令之后，您仍然应该得到"Hello Bob!"。

添加以下SQL查询:

```
INSERT INTO radreply (username, attribute, value,op)VALUES ('bob', 'Fall-Through', 'Yes','=');
```
这将使SQL模块检查分配给Bob的组。当您在radtest命令末尾添加1时，您现在应该得到"Hello PPP protocol!"。

如果在radcheck表中使用密码定义用户，并且所有检查都通过，则上述情况将发生在正常身份验证中。

当发生错误时，例如密码不正确，或者radcheck中定义的检查未通过，或者用户甚至未列在radcheck表中，SQL模块将检查分配给用户的组。

当用户不在radcheck表中或radcheck中指定的必要AVPs不匹配时，请记住以下两点。这些点不受sql.conf文件中read_groups值的影响。这意味着不管你喜不喜欢它都会发生。

* 当SQL模块在radcheck表中找不到用户时，它将检查是否有分配给该用户的组。
* 当为用户定义的radcheck项不匹配时，SQL模块将检查是否有分配给用户的组。

> **Reply-Message AVP**
> 当您在radgroupresply中指定Reply-Message AVP时，即使用户提供了错误的密码，但当radgroupcheck'sub test'通过时，也会返回该消息。Reply-Message AVP是特殊的，因为它是RADIUS中唯一可以伴随接入拒绝分组的AVP。即使radgroupcheck'sub test'通过，groupreply中指定的其他AVPs也不会与访问拒绝数据包一起返回。

#### Profiles

Profiles可以用SQL创建，然后通过两种方式分配给用户:

* 通过default_user_profile指令为sql/mysql/dialup.conf文件中的所有用户指定默认配置文件
* 通过以User-Profile检查属性的形式为用户显式指定配置文件

配置文件是属于至少一个组的用户。此用户不需要radcheck和radreply表中的任何条目。让我们修改数据库，以便bob使用配置文件:

1.登录RADIUS MySQL数据库并发出以下SQL命令以创建所需条目:


```
delete from radcheck;delete from radreply;delete from radgroupreply;delete from radgroupcheck;delete from radusergroup;INSERT INTO radcheck (username, attribute, value,op) VALUES('bob', 'Cleartext-Password', 'passbob',':=');INSERT INTO radcheck (username, attribute, value,op)VALUES('bob', 'User-Profile','student_profile',':=');INSERT INTO radgroupreply (groupname, attribute,value,op)VALUES ('students', 'Reply-Message', 'Hello Student!','=');INSERT INTO radusergroup (username, groupname,priority) VALUES('student_profile', 'students', 10);
```

2.在sql.conf中将read_groups更改回“yes“，重新激活组的读取。

3.重启FreeRADIUS在Debug模式。

4.使用radtest程序验证bob:

`radtest bob passbob 127.0.0.1 100 testing123`

应返回针对用户student_profile的答复消息。

User-Profile AVP的值是一个用户，该用户至少是一个组的成员。在我们的示例中，配置文件用户称为student_profile，它是students组的成员。

分配给学生的radgroupcheck和radgroupresply属性随后将应用于具有check属性`User-Profile := student_profile`的任何用户。用户student_profile在此处用作配置文件，而不是普通用户。

