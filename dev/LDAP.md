##LDAP目录服务
在本章中，我们将会了解轻量级目录访问协议（Lightweight Directory Access Protocol，LDAP）以及它怎样集成到使用Spring Security的应用中以提供认证、授权和用户信息服务。
在本章的内容中，我们将会：

	l  学习一些LDAP协议相关的基本概念以及服务器实现；
	l  在Spring Security中配置一个嵌入式的LDAP服务器；
	l  使用LDAP认证和授权；
	l  理解LDAP查找和用户匹配背后的模型；
	l  从标准的LDAP结构中查询额外的用户信息；
	l  不同的LDAP授权方法，并比较它们的优劣；
	l  使用Spring Bean声明明确配置Spring Security LDAP；
	l  连接LDAP目录，包括Microsoft Active Directory进行认证。

理解LDAP

 LDAP在逻辑目录模型方面能够追溯到超过三十年前——在概念上类似于组织机构图和地址薄。今天，LDAP越来越多的用来作为集中管理组织用户信息的方式，可以将成千的用户分成逻辑上的组并允许在不同的分布式系统间共享统一的用户信息。
 为了安全目的，LDAP经常被用来帮助集中的用户名和密码认证——用户的凭证存储在LDAP目录中，而对于用户来说，认证请求基于目录进行。这对于管理员来说可以便于管理，因为用户的凭证——登录、密码和其它信息——都存储在LDAP中的一个地址。另外，组织机构信息，如组和团队的分配、地理位置以及组织的等级结构，也定义在用户在目录中的位置上。

LDAP

 如果你以前从来没有使用过LDAP，你可能想知道它到底是什么。我们通过一个Apache Directory Server 1.5的截图作为LDAP模式的例子进行介绍。   
![](http://localhost/pic/1.png)

让我们从一个特定用户Albert Einstein（截图中高亮显示）的条目开始，我们可以看到Mr. Einstein的组织成员信息可以从他的树节点往上移动看到。我们可以看到einstein是组织单元（organizational unit，ou）users的成员，而组织单元本身是域example.com的一部分（截屏中显示的dc代表的域组件“domain component”）。在此之前的是LDAP树本身的组织机构元素（DIT和Root DSE），这些现在与我们无关。用户aeinstein在LDAP结构中有其语义且意义明确——你可以想象一个巨大组织更复杂的等级结构也能够很容易的说明其组织机构和部门的边界。
一个叶子节点在树上从上到下的路径形成了包含所有参与节点的字符串，如Mr. Einstein的节点路径为：
	
	uid=aeinstein,ou=users,dc=example,dc=com

 这个节点路径是唯一的，并被称为节点的标识名（distinguished name，DN）。标识名类似于数据库里的主键，允许节点在复杂的树结构中唯一标识和定位。我们将会看到的节点的DN广泛应用于Spring Security LDAP集成的认证和查找过程。

 我们可以看到还有几个其他的用户和Mr. Einstein在同一个等级的组织结构上。我们假设所有的这些用户都与Mr. Einstein在相同的组织下。尽管这个组织机构的例子相对简单，但是LDAP的结构是极其灵活的，使得很多层级的逻辑组织机构嵌套成为可能。
 Spring Security LDAP需要Spring LDAP模块的支持（http://www.springsource.org/ldap），它是单独于Spring框架核心和Spring Security的工程。它被认为很稳定并对标准的Java LDAP功能进行了有用的封装。

###通用的LDAP属性名

树上的每个实际节点都是通过一个或多个的对象类（object class）来定义的。一个对象类是组织机构的一个逻辑单元，分为一系列具有语义的相关属性。通过将一个条目声明为特定对象类的实例，如person，LDAP目录的管理人员就能够为目录的用户提供每个条目的确切含义。

LDAP具有很丰富的标准模式（schema），涵盖了可用的LDAP对象类和它们的可用属性（以及一些其它信息）。如果你计划广泛的使用LDAP，强烈建议你参考一个较好的用户手册，如Zytrax OpenLDAP的附录（http://www.zytrax.com/books/ldap/ape/ ），或者Internet2组织提供的人员相关模式（http://middleware.internet2.edu/eduperson/）。

在上一节中，我们了解到LDAP中的每个条目都会有一个标识名，它在树上唯一标识节点。DN有一系列的属性组成，其中一个（或更多）用来标识树上的用DN代表的路径。因为DN中路径的每一部分都代表一个LDAP属性，所以你能够通过定义良好的LDAP模式和类对象来确定DN中每个属性的含义。
我们在以下的表格中，列出了一些常见的属性和它们的含义。这些属性是用来作为组织相关的属性——一意味着它们一般用来定义LDAP树的组织机构——并按照结构上从上到下的顺序，正如你通常在LDAP中会见到的那样。

![图表](http://localhost/pic/2.png)

  要记住的是有上百个标准的LDAP属性——上面只是其中的一小部分，当你与一个完整LDAP集成的话会看到它们。但是表中的这些属性是目录树中组织相关的属性，当你配置Spring Security与LDAP交互的时候可能会用来形成各种查询表达式或匹配符。

###运行一个嵌入式的LDAP服务

  作为测试，Spring Security允许使用嵌入式的LDAP服务器。就像我们使用嵌入式数据库那样，这使得应用可以启动一个基于内存的LDAP服务器并插入初始化数据。当然，这样的一个配置只能用于测试的目的，但是这能够节省我们很多配置单独LDAP服务器的时间。

 嵌入式的LDAP服务器功能是通过使用Apache Directory Server (DS) 1.5来支持的，它是一个基于Java、开源且完全符合规范的LDAP服务器。实际上，你也可以使用Apache DS作为独立的服务器，它很相对很容易配置并易于获取和安装。本章实例代码中的Dependencies目录下包含了嵌入式LDAP服务器所需要的JAR包——如果你要自己使用它的话，你要么使用Maven要么自己到以下地址http://directory.apache.org/ 下载Apache DS。

 如同嵌入式的HSQL数据库允许在启动时加载SQL脚本，嵌入式的LDAP服务器提供了启动时从LDAP数据交换格式（LDAP Data Interchange Format ，LDIF）文件中插入目录的方法。LDIF是一种简洁的且对人和机器都很易读的数据定义格式，它提供了LDAP对象和支持数据的灵活定义。在本章的源码中提供了几个实例性的LDIF文件。

###配置基本的LDAP集成

 现在让我们让JBCP Pets支持基于LDAP的认证。幸运的是，通过使用嵌入式的LDAP服务器和实例LDIF文件，这是一个相对容易的练习。在这个练习中，我们使用为本书创建的LDIF文件，这个文件用来进行表述LDAP和Spring Security的常用配置场景。我们提供了几个其它的LDIF文件，其中一些来自Apache DS 1.5，还有一个来自SpringSecurity的单元测试，你可能会愿意选择它们进行体验

###配置LDAP服务器引用
第一步是在dogstore-security.xml中声明嵌入式LDAP服务器的引用。LDAP服务器的声明在<http>元素之外，与<authentication-manager>相同的等级：
 
	<ldap-server ldif="classpath:JBCPPets.ldif" id="ldapLocal" 
	root="dc=jbcppets,dc=com"/>

我们从classpath中加载JBCPPets.ldif，并用其为LDAP服务器插入数据。这意味着（如同嵌入式HSQL数据库启动那样）我们应该在WEB-INF/classes放置JBCPPets.ldif文件。root属性用特定的DN声明了LDAP目录的根。这应该与我们使用的LDIF文件逻辑根DN相对应。
【注意，对于嵌入式的LDAP服务器，root是必须的，尽管XML模式并没有这样声明。如果它没有指明或指明错误，你会在Apache DS server启动的时候看待几个奇怪的错误。】
当我们在Spring Security配置文件中声明LDAP用户服务和其它配置元素时，会重用这里定义的bean ID。对于嵌入式的LDAP模式来说，<ldap-server>声明的其它属性都是可选的。

###启用LDAP AuthenticationProvider

接下来，我们要配置另一个AuthenticationProvider，它用来用LDAP来检查用户凭证。简单得添加另一个AuthenticationProvider即可，如下：

	<authentication-manager alias="authenticationManager">
	<!-- Other authentication providers are here -->
	  <ldap-authentication-provider server-ref="ldapLocal"
	    user-search-filter="(uid={0})" 
	    group-search-base="ou=Groups"/>
	</authentication-manager>

我们稍后将会介绍这些属性——现在，回到应用并运行，使用用户名ldapguest和密码password进行登录。你应该能够登录进去了

###解决嵌入式LDAP的问题

很可能你在使用嵌入式LDAP时，调试问题很困难。Apache DS的出错信息并不友好，这在SpringSecurity嵌入模式下更严重。如果你不能让这个简单的例子正常运行，请仔细检查以下的地方：
l  确保Apache DS依赖的所有JAR都在web应用的classpath下。这会有很多——最好的方式就是包含所有的（实例代码就是这样做的）；
l  确保在你的配置文件中<ldap-server>设置了root属性，且它与启动时加载的LDIF文件中root的定义相匹配。如果你遇到了找不到引用的错误，很可能要么缺少root元素，要么与LDIF文件不匹配；
l  注意的是启动嵌入式LDAP的错误并不会是一个致命错误。为了分析加载LDIF文件的错误，你需要确保适当设置了日志，包括Apache DS的日志启用，至少要在ERROR级别。LDIF的加载器在包下，它应该被用来启用LDIF加载错误的日志；
l  如果应用没有被正常关闭，为了重新启动服务，你可能会需要删除临时目录下的一些文件（Windows系统下为%TEMP%）。这个的出错信息（幸运的是）很清楚。

遗憾的是，嵌入式LDAP并不像嵌入式HSQL数据库那样简单，但是相对于需要下载和配置的很多外部LDAP服务器来说，已经比较简单了。
一个用于排除问题和访问LDAP的好工具是Apache Directory Studio，它提供了独立的和Eclipse插件的版本。免费下载地址：http://directory.apache.org/studio/ 。

###理解Spring LDAP认证如何工作

我们看到可以使用LDIF文件定义的用户（也就会在LDAP目录中出现）进行登录了。一个LDAP用户进行登录时到底发生了什么？在LDAP认证过程中有三个基本的步骤：
l  将用户提供的凭证与LDAP目录进行认证；
l  基于LDAP上的信息，确定用户拥有的GrantedAuthority；
l  为了应用以后用到，从LDAP条目中预先加载用户信息到自定义的UserDetails对象中。

####认证用户凭证

第一步，通过织入AuthenticationManager的自定义认证提供者与LDAP目录进行认证。o.s.s.ldap.authentication.LdapAuthenticationProvider将用户提供的凭证与LDAP目录进行校验，如下图所示：
![](http://localhost/pic/3.png)

我们可以看到o.s.s.ldap.authentication.LdapAuthenticator接口定义了一个代理从而允许提供者以自定义的方式认证请求。在这里我们明确配置的是o.s.s.ldap.authentication.BindAuthenticator，它会尝试使用用户的凭证绑定（登录）LDAP服务器，就像用户本身尝试建立连接。对嵌入式的服务器来说，这对于我们的认证要求是足够的，但是，外部的LDAP服务器在用户绑定LDAP目录上可能要求更严格。幸运的是，还有一种替代的认证方式，我们将会在本章稍后介绍。
正如图中所标注的那样，记住查找是在<ldap-server>引用的manager-dn属性所创建的LDAP上下文中进行的。对于嵌入式的服务器，我们没有使用这个信息，但是对于外部的服务器引用，除非提供manager-dn，否则的话将会进行匿名绑定。为了保持目录中公开访问信息的限制，通常需要合法的凭证来进行LDAP目录的搜索，这样的话，manager-dn在现实世界场景中基本上就是必需的了。manager-dn代表了用户的全DN，基于合法的访问绑定目录并进行查找
####确定用户的角色

在用户基于LDAP服务器成功认证之后，接下来必须要进行权限信息的确定。授权是通过安全实体的一系列角色定义的，LDAP认证过的用户角色确定如下图所示：
![](http://localhost/pic/4.png)
 我们可以看到，用户在使用LDAP认证之后，LdapAuthenticationProvider委托给了一个LdapAuthoritiesPopulator。DefaultLdapAuthoritiesPopulator将会尝试在LDAP等级中另一个条目的同级或下级属性中查找认证用户的DN。（译者注：即在LDAP目录角色相关的条目中寻找当前用户，以确定用户的角色）
         查找用户角色分配的DN是通过group-search-base属性定义的——在我们的例子中，我们这样设置group-search-base="ou=Groups"。当一个用户的DN在group-search-base DN下面的条目中时，包含用户DN的条目中的一个属性将会作为这些用户的角色。
【你可能注意到我们混合使用了属性的写法——在类流程图中使用了groupSearchBase，在文本中使用的是 group-search-base。这是有意的——文本中对应的是XML配置属性而图中指的是相关类的成员（属性）。他们的命名相似，但是在不同的上下文中（XML和Java）要适当调整。】
         Spring Security中的角色和LDAP中的用户如何关联还是有点令人迷惑，所以让我们看一下JBCP Pets库以及用户与角色关联是如何进行的。DefaultLdapAuthoritiesPopulator使用了几个<ldap-authentication-provider>声明的属性来管理为用户查找角色。这些属性大致按以下的顺序使用：
l  group-search-base：它定义了基础的DN，LDAP集成应该基于此往下为用户查找一个或多个的匹配项。默认值会在LDAP根中进行查找，这可能会代价较高；
l  group-search-filter：它定义了LDAP查找的过滤器，用来匹配用户的DN与group-search-base之下的条目属性。这个过滤器通过两个参数进行参数化设置——第一个（{0}）作为用户的DN，第二个作为（{1}）作为用户的名字。默认值为（uniqueMember={0}）。
l  group-role-attribute：它定义了匹配条目中用来组装用户GrantedAuthority的属性，默认值为cn；
l  role-prefix：要拼到在group-role-attribute中发现值的前缀以产生Spring Security的GrantedAuthority。默认值为ROLE_。
这对于新的开发人员可能会比较抽象和困难，因为这与我们基于JDBC的UserDetailsService实现有很大的区别。让我们以JBCP Pets LDAP目录中的ldapguest用户登录以了解其过程。
用户的DN是uid=ldapguest,ou=Users,dc=jbcppets,dc=com而group-search-base被配置成了ou=Groups。对于这个ou的LDAP树展现如下：

![](http://localhost/pic/5.png)

我们可以看到在ou=Groups之下，有两个条目（cn=Admin和cn=User）。每个条目都具有objectClass: groupOfUniqueNames（你可能会记起我们在本章前面讨论过的对象类）。这种类型的LDAP对象允许多个DN值存储在这个条目下并进行逻辑分组。条目cn=User的属性列在下图中：

![1111](http://localhost/pic/6.png)

我们可以看到cn=User的uniqueMember属性用来标识这个组里面的LDAP用户。你也可能会发现uniqueMember的属性值就是对应用户的DN。
现在再看角色搜索的逻辑的就很容易了。从ou=Groups (group-search-base)开始，Spring Security将会查找任何uniqueMember属性值与用户DN（group-search-filter）匹配的条目。当它找到匹配的条目，条目的cn值（group-role-attribute）——在本例中即为User，将会加上ROLE_ (role-prefix)前缀然后转换成大写字母组成用户的GrantedAuthority。一旦我们使用过它，再理解起来就容易一些了，不是吗？
【Spring LDAP很灵活。要记住的是尽管这是一个组织LDAP兼容Spring Security的方式，但是通常的使用场景恰恰相反——LDAP目录已经存在，Spring Security需要织入。在很多场景下，你可以重新配置Spring Security来处理LDAP的等级结构。但是，很关键的一点是你要有效规划并理解Spring在查询时如何与LDAP一起工作。开动你的大脑，勾画出用户查找和组查找以及你能想到的最优方案——让查询范围尽可能小和精确。】
如果你此时还是感到困惑，我们建议你休息一下然后尝试使用Apache Directory Studio来看一下运行系统配置的嵌入式LDAP服务器。如果你按照前面描述的算法，尝试自己查找一下目录将会有助于你了解Spring Security LDAP配置的流程。

####匹配UserDetails的其它属性

 最后，在通过LDAP查找分配给用户GrantedAuthority后，

o.s.s.ldap.userdetails.LdapUserDetailsMapper将会使用

o.s.s.ldap.userdetails.UserDetailsContextMapper来获取另外的细节信息来填充UserDetails。

使用我们到现在为止配置的<ldap-authentication-provider>，LdapUserDetailsMapper将会使用用户LDAP条目中的信息填充UserDetails对象。


![1111](http://localhost/pic/7.png)

我们稍后将会看到UserDetailsContextMapper怎样配置才能从标准的LDAP person和inetOrgPerson中获取丰富的信息。使用基本的LdapUserDetailsMapper，仅仅能够存储用户名、密码以及GrantedAuthority。

尽管在LDAP用户认证里面还有很多的结构，但是你会发现整体的流程与我们前面学习的JDBC认证很类似（认证用户、填充GrantedAuthoritys）。如同JDBC认证中那样，在LDAP集成中也有进行高级配置的能力——让我们了解的更深入一些并看看还能做什么。

##LDAP的高级配置 

一旦我们要了解LDAP基础集成之外的知识，就会发现security XML命名空间方式的配置中，Spring Security LDAP模块还有许多的可用配置。它包括查询用户的个人信息、用户认证的其它方式以及使用LDAP作为UserDetailsService且与DaoAuthenticationProvider结合。
####实例JBCP LDAP用户

在JBCP Pets LDIF文件中，我们提供了许多的用户。在高级配置练习和自学中，以下的快速查询表可能会对你有所帮助。要注意的是除了userwithphone以外，所有用户的密码均为password。

![1111](http://localhost/pic/8.png)