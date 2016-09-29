---
title: php路由机制的实现
date: 2016-09-19 09:36:33
tags:
---

## 是什么

**维基百科**路由（routing）就是通过互联的网络把信息从源地址传输到目的地址的活动。路由发生在OSI网络参考模型中的第三层即网路层。路由引导分组转送，经过一些中间的节点后，到它们最后的目的地。作成硬件的话，则称为路由器。
在php中的路由同OSI中路由的作用相似, 用来处理接收到的请求，将请求指向相应的控制器或者处理程序.

<!-- more -->

## 为什么

在我们见到的大多数网站的url里, 都会有对路径的指示, 比如这种

    https://www.google.com/search?q=html&oq=html&aqs=chrome.0.69i59j69i60l3j69i57j69i60.596j0j4&sourceid=chrome&ie=UTF-8
或者

    https://developer.mozilla.org/en-US/docs/Web/HTML/Element/address
第一种url内容比较长而且不易读, 第二种url用户或者搜索引擎可以直接读取到有用的内容, 通过这种url, 爬虫可以方便的通过处理url(通过'/'分割)来对内容进行分类.要让服务器理解第二种url并返回正确的数据的话, 需要对服务器进行相应的配置. 

## 怎样做

* 工具 **.htacess**

> **百度百科**.htaccess文件(或者"分布式配置文件"）提供了针对目录改变配置的方法， 即，在一个特定的文档目录中放置一个包含一个或多个指令的文件， 以作用于此目录及其所有子目录。作为用户，所能使用的命令受到限制。管理员可以通过Apache的AllowOverride指令来设置。

> **维基百科**.htaccess 是Apache HTTP Server的文件目录系统级别的配置文件的默认的名字。它提供了在主配置文件中定义用户自定义指令的支持。 这些配置指令需要在 .htaccess 上下文 和用户需要的适当许可。

通过.htaccess可以实现url的重定向, 伪静态, 设置错误页面, 限制ip访问等等功能.
在服务器根目录(或者单独需要路由规则的目录下)创建文件, 文件名.htaccess, 在这个文件中用#注释.

1. 设置目录的默认页面
```
    #设置默认页面
    DirectoryIndex index.html index.PHP index.htm
```
打开目录时默认打开index.html文件

2. 设置错误页面
```
    ErrorDocument 404 404.html
```
自定义错误页面, 规则是ErrorDocument 错误码 错误页

3. 重定向
```
    #开启Rewrite
    RewriteEngine on
    #默认为/, 即根目录, 在根目录下不需要设置
    RewriteBase /MVC/
```

Rewirte主要的功能就是实现URL的跳转和隐藏真实地址, 通过正则表达式将匹配到的URL重定向到对应的文件中
 
比如要将url
    
    http://www.xx.com/api.php?type=detail&id=123
跳转至

    http://www.xx.com/detail/123
Rewrite代码为

    RewriteRule ^([a-zA-Z0-9]+)/([0-9]+)$ api.php?type=$1&id=$2
    #$1和$2为正则表达式中通过小括号取到的子表达式
Rewirte模块内部处理

Rewirte模块的内部处理极为复杂，但是为了使一般用户避免犯低级错误，也让管理员能充分利用其功能，在此仍然做一下说明。

Rewirte模块API阶段

首先，你必须了解Apache是分若干阶段来处理HTTP请求的。Apache API对每个阶段都提供了一个hook程序。mod_rewrite使用两个hook程序：其一，从URL到文件名的转换hook(用在读取HTTP请求之后、授权开始之前)； 其二，修正hook(用在授权阶段和读取目录级配置(.htaccess)之后、内容处理器激活之前)。

所以，Apache收到一个请求并且确定了响应主机(或虚拟主机)之后，重写引擎即开始处理服务器级配置中的所有mod_rewrite指令(此时处于从URL到文件名转换的阶段)，此阶段完成后，最终的数据目录便确定了。接下来进入修正程序段并触发目录级配置中的mod_rewrite指令。这两个阶段并不是泾渭分明的，但都实施了把URL重写成新的URL或者文件名。虽然API最初不是为此目的而设计的，但是现在它已经成为了API的一种用途。记住以下两点，会有助于更好地理解：

1、虽然mod_rewrite可以将URL重写为新的URL或文件名，甚至将文件名重写为新的文件名，但是之前的API只提供从URL到文件名的hook。在Apache 2.0中，增加了两个丢失的hook以使得处理过程更加清晰。不过这样做并没有给用户带来麻烦，用户只需记住这样一个事实：借助从URL到文件名的hook比最初API设计的目标功能更强大。

2、令人难以置信的是，mod_rewrite还提供了目录级的URL操作(.htaccess文件)，而这些文件必须在将URL转换成文件名之后才会被处理(这是必须的，因为.htaccess存在于文件系统中)。换句话说，根据API阶段，这时再处理任何URL操作已经太晚了。为了解决这个”鸡和蛋”的问题，mod_rewrite使用了一个小技巧：在进行一个目录级的URL/文件名操作时，先把文件名重写回相应的URL(通常这个操作是不可行的，但是参考下面的RewriteBase指令就能明白它是怎么实现的了)，然后，对这个新的URL建立一个新的内部的子请求，再重新开始API阶段的执行。

另外，mod_rewrite尽力使这些复杂的操作对用户透明。但仍须记住：服务器级的URL操作速度快而且效率高，而目录级的操作由于这个”鸡和蛋”的问题速度较慢而且效率也低。但从另一个侧面看，这却是mod_rewrite得以为一般用户提供(局部限制的)URL操作的唯一方法。

Rewirte模块规则集的处理

当mod_rewrite在这两个API阶段中开始执行时，它会读取配置结构中配置好的 (或者是在服务启动时建立的服务器级的，或者是在遍历目录采集到的目录级的)规则集，然后，启动URL重写引擎来处理(带有一个或多个条件的)规则集。无论是服务器级的还是目录级的规则集，都是由同一个URL重写引擎处理，只是最终结果处理不同而已。

规则集中规则的顺序是很重要的，因为重写引擎是按一种特殊的顺序处理的：逐个遍历每个规则(RewriteRule指令)，如果出现一个匹配条件的规则，则可能回头遍历已有的规则条件(RewriteCond指令)。由于历史的原因，条件规则是前置的，所以控制流程略显冗长，细节见图-1。


重写规则集中的控制流

![image](http://www.yezhongqi.com/wp-content/plugins/wp-ueditor/ueditor/php/upload/93371392111902.gif)
图-1：重写规则集中的控制流


可见，URL首先与每个规则的Pattern匹配，如果匹配失败，mod_rewrite将立即终止此规则的处理，继而处理下一个规则。如果匹配成功，mod_rewrite将寻找相应的规则条件，如果一个条件都没有，则简单地用Substitution构造的新值来替换URL，然后继续处理其他规则；但是如果条件存在，则开始一个内部循环按其列出的顺序逐个处理。对规则条件的处理有所不同：URL并不与模式进行匹配，而是首先通过扩展变量、反向引用、查找映射表等步骤建立一个TestString字符串，然后用它来与CondPattern匹配。如果匹配失败，则整个条件集和对应的规则失败；如果匹配成功，则执行下一个规则直到所有条件执行完毕。如果所有条件得以匹配，则以Substitution替换URL，并且继续处理。(本部分引用译者：金步国)

RewriteCond指令格式

语法: RewriteCond TestString CondPattern [flags]

RewriteCond指令定义一条规则条件。在一条RewriteRule指令前面可能会有一条或多条RewriteCond指令，只有当自身的模板(pattern)匹配成功且这些条件也满足时规则才被应用于当前URL处理。

1、 TestString是一个纯文本的字符串，除了包含普通的字符外，还可以包括下列的可扩展结构：

1）$N：RewriteRule后向引用，其中(0 <= N <= 9) 。$N引用紧跟在RewriteCond后面的RewriteRule中模板中的括号中的模板在当前URL中匹配的数据。

2）%N：RewriteCond后向引用，其中(0 <= N <= 9) 。%N引用最后一个RewriteCond的模板中的括号中的模板在当前URL中匹配的数据。

3）${mapname:key|default}：RewriteMap扩展。

2、CondPattern是条件pattern, 即一个应用于当前实例TestString的正则表达式, 即TestString将会被计算然后与CondPattern匹配。作为一个标准的扩展正则式，CondPattern有以下补充：

1）可以在模板串前增加一个!前缀，以用表示不匹配模板。但并不是所有的test都可以加！前缀。

2)CondPattern中可以使用以下特殊变量：

‘>CondPattern’ (大于) 将condPattern当作一个普通字符串，将它和TestString进行比较，当TestString 的字符大于CondPattern为真。

‘=CondPattern’ (等于) 将condPattern当作一个普通字符串，将它和TestString进行比较，当TestString 与CondPattern完全相同时为真.如果CondPattern只是 “” (两个引号紧挨在一起) 此时需TestString 为空字符串方为真。

‘-d’ (是否为目录) 将testString当作一个目录名，检查它是否存在以及是否是一个目录。

‘-f’ (是否是regular file) 将testString当作一个文件名，检查它是否存在以及是否是一个regular文件。

‘-s’ (是否为长度不为0的regular文件) 将testString当作一个文件名，检查它是否存在以及是否是一个长度大于0的regular文件。

‘-l’ (是否为symbolic link) 将testString当作一个文件名，检查它是否存在以及是否是一个 symbolic link。

‘-F’ (通过subrequest来检查某文件是否可访问) 检查TestString是否是一个合法的文件，而且通过服务器范围内的当前设置的访问控制进行访问。这个检查是通过一个内部subrequest完成的, 因此需要小心使用这个功能以降低服务器的性能。

‘-U’ (通过subrequest来检查某个URL是否存在) 检查TestString是否是一个合法的URL，而且通过服务器范围内的当前设置的访问控制进行访问。这个检查是通过一个内部subrequest完成的, 因此需要小心使用这个功能以降低服务器的性能。

3、[flags]是第三个参数，多个标志之间用逗号分隔。

1）’nocase|NC’ (不区分大小写) 　　在扩展后的TestString和CondPattern中，比较时不区分文本的大小写。注意，这个标志对文件系统和subrequest检查没有影响.

2）’ornext|OR’ (建立与下一个条件的或的关系) 　　默认的情况下，二个条件之间是AND的关系，用这个标志将关系改为OR。例如： RewriteCond %{REMOTE_HOST} ^host1.* [OR] RewriteCond %{REMOTE_HOST} ^host2.* [OR] RewriteCond %{REMOTE_HOST} ^host3.* RewriteRule … 如果没有[OR]标志，需要写三个条件/规则.

RewriteRule 指令

语法: RewriteRule Pattern Substitution [flags]

1) Pattern是一个作用于当前URL的兼容perl的正则表达式. 这里的“当前”是指该规则生效时的URL的值。

2) Substitution是，当原始URL与Pattern相匹配时，用以替代(或替换)的字符串。

3) 此外，Substitution还可以追加特殊标记[flags] 作为RewriteRule指令的第三个参数。 Flags是一个包含以逗号分隔的下列标记的列表:

redirect|R [=code] (强制重定向 redirect)

以 http://thishost[:thisport]/(使新的URL成为一个URI) 为前缀的Substitution可以强制性执行一个外部重定向。 如果code没有指定，则产生一个HTTP响应代码302(临时性移动)。如果需要使用在300-400范围内的其他响应代码，只需在此指定这个数值即可， 另外，还可以使用下列符号名称之一: temp (默认的), permanent, seeother. 用它可以把规范化的URL反馈给客户端，如, 重写“/~”为 “/u/”，或对/u/user加上斜杠，等等。

注意: 在使用这个标记时，必须确保该替换字段是一个有效的URL! 否则，它会指向一个无效的位置! 并且要记住，此标记本身只是对URL加上 http://thishost[:thisport]/的前缀，重写操作仍然会继续。通常，你会希望停止重写操作而立即重定向，则还需要使用’L’标记.

forbidden|F (强制URL为被禁止的 forbidden)

强制当前URL为被禁止的，即，立即反馈一个HTTP响应代码403(被禁止的)。使用这个标记，可以链接若干RewriteConds以有条件地阻塞某些URL。

gone|G’(强制URL为已废弃的 gone)

强制当前URL为已废弃的，即，立即反馈一个HTTP响应代码410(已废弃的)。使用这个标记，可以标明页面已经被废弃而不存在了.

proxy|P (强制为代理 proxy)

此标记使替换成分被内部地强制为代理请求，并立即(即， 重写规则处理立即中断)把处理移交给代理模块。你必须确保此替换串是一个有效的(比如常见的以 http://hostname开头的)能够为Apache代理模块所处理的URI。使用这个标记，可以把某些远程成分映射到本地服务器名称空间， 从而增强了ProxyPass指令的功能。

注意: 要使用这个功能，代理模块必须编译在Apache服务器中。 如果你不能确定，可以检查“httpd -l”的输出中是否有mod_proxy.c。 如果有，则mod_rewrite可以使用这个功能；如果没有，则必须启用mod_proxy并重新编译“httpd”程序。

last|L (最后一个规则 last)

立即停止重写操作，并不再应用其他重写规则。 它对应于Perl中的last命令或C语言中的break命令。这个标记可以阻止当前已被重写的URL为其后继的规则所重写。 举例，使用它可以重写根路径的URL(’/’)为实际存在的URL, 比如, ‘/e/www/’.

next|N (重新执行 next round)

重新执行重写操作(从第一个规则重新开始). 这时再次进行处理的URL已经不是原始的URL了，而是经最后一个重写规则处理的URL。它对应于Perl中的next命令或C语言中的continue命令。 此标记可以重新开始重写操作，即, 立即回到循环的头部。
但是要小心，不要制造死循环!

chain|C (与下一个规则相链接 chained)

此标记使当前规则与下一个(其本身又可以与其后继规则相链接的， 并可以如此反复的)规则相链接。 它产生这样一个效果: 如果一个规则被匹配，通常会继续处理其后继规则， 即，这个标记不起作用；如果规则不能被匹配，则其后继的链接的规则会被忽略。比如，在执行一个外部重定向时， 对一个目录级规则集，你可能需要删除“.www” (此处不应该出现“.www”的)。

type|T=MIME-type(强制MIME类型 type)

强制目标文件的MIME类型为MIME-type。 比如，它可以用于模拟mod_alias中的ScriptAlias指令，以内部地强制被映射目录中的所有文件的MIME类型为“application/x-httpd-cgi”。

nosubreq|NS (仅用于不对内部子请求进行处理 no internal sub-request)

在当前请求是一个内部子请求时，此标记强制重写引擎跳过该重写规则。比如，在mod_include试图搜索可能的目录默认文件(index.xxx)时， Apache会内部地产生子请求。对子请求，它不一定有用的，而且如果整个规则集都起作用，它甚至可能会引发错误。所以，可以用这个标记来排除某些规则。

根据你的需要遵循以下原则: 如果你使用了有CGI脚本的URL前缀，以强制它们由CGI脚本处理，而对子请求处理的出错率(或者开销)很高，在这种情况下，可以使用这个标记。

nocase|NC (忽略大小写 no case)

它使Pattern忽略大小写，即, 在Pattern与当前URL匹配时，’A-Z’ 和’a-z’没有区别。

qsappend|QSA (追加请求串 query string append)

此标记强制重写引擎在已有的替换串中追加一个请求串，而不是简单的替换。如果需要通过重写规则在请求串中增加信息，就可以使用这个标记。

noescape|NE (在输出中不对URI作转义 no URI escaping)

此标记阻止mod_rewrite对重写结果应用常规的URI转义规则。 一般情况下，特殊字符(如’%’, ‘$’, ‘;’等)会被转义为等值的十六进制编码。 此标记可以阻止这样的转义，以允许百分号等符号出现在输出中，如：

RewriteRule /foo/(.*) /bar?arg=P1\=$1 [R,NE] 可以使’/foo/zed’转向到一个安全的请求’/bar?arg=P1=zed’.

passthrough|PT (移交给下一个处理器 pass through)

此标记强制重写引擎将内部结构request_rec中的uri字段设置为 filename字段的值，它只是一个小修改，使之能对来自其他URI到文件名翻译器的 Alias，ScriptAlias, Redirect 等指令的输出进行后续处理。举一个能说明其含义的例子：如果要通过mod_rewrite的重写引擎重写/abc为/def，然后通过mod_alias使/def转变为/ghi，可以这样:

RewriteRule ^/abc(.*) /def$1 [PT]
Alias /def /ghi
如果省略了PT标记，虽然mod_rewrite运作正常， 即, 作为一个使用API的URI到文件名翻译器，它可以重写uri=/abc/…为filename=/def/…，但是，后续的mod_alias在试图作URI到文件名的翻译时，则会失效。

注意: 如果需要混合使用不同的包含URI到文件名翻译器的模块时， 就必须使用这个标记。。混合使用mod_alias和mod_rewrite就是个典型的例子。

For Apache hackers

如果当前Apache API除了URI到文件名hook之外，还有一个文件名到文件名的hook， 就不需要这个标记了! 但是，如果没有这样一个hook，则此标记是唯一的解决方案。 Apache Group讨论过这个问题，并在Apache 2.0 版本中会增加这样一个hook。

skip|S=num (跳过后继的规则 skip)

此标记强制重写引擎跳过当前匹配规则后继的num个规则。 它可以实现一个伪if-then-else的构造: 最后一个规则是then从句，而被跳过的skip=N个规则是else从句. (它和’chain|C’标记是不同的!)

env|E=VAR:VAL (设置环境变量 environment variable)

此标记使环境变量VAR的值为VAL, VAL可以包含可扩展的反向引用的正则表达式$N和%N。 此标记可以多次使用以设置多个变量。这些变量可以在其后许多情况下被间接引用，但通常是在XSSI (via ) or CGI (如 $ENV{’VAR’})中， 也可以在后继的RewriteCond指令的pattern中通过%{ENV:VAR}作引用。使用它可以从URL中剥离并记住一些信息。

cookie|CO=NAME:VAL:domain[:lifetime[:path]] (设置cookie)

它在客户端浏览器上设置一个cookie。 cookie的名称是NAME，其值是VAL。 domain字段是该cookie的域，比如’.apache.org’, 可选的lifetime是cookie生命期的分钟数，可选的path是cookie的路径。

案例：

city_map.txt的内容：

hangzhou 12

beijing 13

1、hangzhou.google.com/tianqi/20090401 跳转到 www.google.com/service/detail.html?id=tianqi&date=20090401

RewriteMap city–map txt:/etc/httpd/conf.d/map/city_map.txt

RewriteCond %{HTTP_HOST}    ^(.+)\.google\.com$

RewriteRule ^/([\w]+)/([\d]+)$ /service/detail\.html\?id=$1&date=$2&c=${city–map:%1|%1} [PT,L]

解释：

%{HTTP_HOST}：取请求的域名

^(.+)\.google\.com$：^,开头；$结尾。.（逗号），除终止符外的任意字符。+，重复一个或一个以上的字符。\，转义字符。

^/([\w]+)/([\d]+)$：[]，集合字符。\w，数字或字母。\d，数字。

$1：表示的是符合RewriteRule 中[\w]+正则式的字符串，也就是tianqi。

$2：表示的是符合RewriteRule 中[\d]+ 正则式的字符串，也就是20090401。

%1：表示的是符合RewriteCond 中.+正则式的字符串，也就是hangzhou。

${city-map:%1|%1}：表示取city-map中%1也就是hangzhou对应的值，如果没有则为%1也就是hangzhou。

2、能看出下面的规则是做了什么吗？

RewriteCond     %{HTTP_HOST}    ^(.+)\.google\.com$

RewriteRule ^/([\w]+)/([^-]+)–([^-]+)—([^-]+)–([^-]+)—([^-]+)–([^-]+)—([^-]+-[^-]+–[^-]+-[^-]+–[^-]+-[^-]+)$  /$1/$2=$3&$4=$5&$6=$7&$8   [C]

RewriteCond     %{HTTP_HOST}    ^(.+)\.google\.com$

RewriteRule ^/([\w]+)/([^-]+)–([^-]+)—([^-]+)–([^-]+)—([^-]+)–([^-]+)$     /service/list\.html\?frontCategoryId=${category–map:$1|0}&$2=$3&$4=$5&$6=$7&city=${city–map:%1|%1}   [PT,L]

解释：

这个规则是想把-（中划线）转为=，把- -（两条中划线）转为&。

[^-]：^在字符集合符号（[]）之内表示反向选择，之外表示行首，所以表示不以-开头。

因为$N，N最大为9，所以使用了C，用第二条RewriteRule把第一条RewriteRule中的最后一个节点，即$8，进行继续转换。

此外，rewrite规则中如果遇到中文，相当有可能会出现乱码问题，因为apache在rewrite时会做一次url解码，这时jk进行请求转发时，就不会再是编码后的字符串了。此种情况，可以在一开始就进行两次编码(encode)，或者在接收请求时先用ISO-8859-1取字节流，再使用UFT-8来new String。(new String(str.getBytes(”ISO-8859-1″),”UFT-8″))

