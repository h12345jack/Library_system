﻿# 数据库小组作业
使用了[ThinkPHP](http://www.thinkphp.cn)，虽然本人觉得该框架实在有点反人类，但是也算是第一次使用PHP的框架。
可以对于MVC有更深的理解。网址在[这里](http://h12345jack.tk/homework/db/Library_system)
前端这样：

![](http://ww1.sinaimg.cn/mw690/8785ef6ejw1f37po7yjs9j21570ju40s.jpg)

后台这样：

![](http://ww3.sinaimg.cn/mw690/8785ef6ejw1f37ppiky34j21h00gan1p.jpg)

好像是有角色管理，记不得了。年久失修。
后面大概是报告的内容，当时用word写的。手转了一份MD
登录后台的默认管理员是2333333333，密码admin

##  功能设计
### 数据库设计
####   需求分析
#####  系统任务
我们所要做的是一个高校图书馆藏书借阅系统，用于读者对图书的检索、浏览、借阅。因此，需要的功能包括对图书的检索查询、借阅、预约、续借、安全性保障。
而在拓展功能板块，我们的考虑为以图书为中心建立起图书与图书之间的关系，图书与人之间的关系，以图书为中心的推荐系统强调的是如何建立起有效的链接关系。
#####  有需求的用户组成
对此系统有需求的用户是有浏览、借阅图书需求的读者——由教师、学生、普通游客组成。
其中，教师、学生是有借阅/预约图书资格的用户，而且借阅权限有所不同；
普通游客是那些非本校的、可以浏览、检索此系统所藏图书而没有借阅和预约资格的用户。
#####  用户的信息要求
教师与学生使用此系统主要用于检索、浏览、借阅图书，包括查询书目信息，借书、还书，预约与续借。教师与学生为获得借阅资格，需要在此系统上进行用户注册并登陆，然后通过检索，并根据每本书的书目信息——其ISBN、标题、摘要、封面图、出版者、出版年份、中图法的图书分类号、价格来确定所借阅的图书。
教师与学生还有对自身用户状态的信息需求，包括借阅记录、预约记录、信用值、是否被罚款等。
同时，还有对所需图书的借阅状态的信息需求，比如只有知道此书是否已被他人预约而确定能否续借等。额外的需求还有图书推荐等。
普通游客不需要进行用户注册和登陆，只是浏览，其需求包括了解此系统内所含图书的大体信息，比如标题、封面图、摘要等从而了解馆藏的大致情况。

#####  系统边界
对此系统的人工操作包括供图书馆管理员操作的图书的录入，新用户的等级，图书的修改，用户的修改等，而数据库管理员拥有上述的所有权限，并且能够定义用户的权限等。
总体而言，DBA拥有对于系统的全部掌握。

####   概念结构设计
首先分析数据库描述的主要对象中有哪些实体，最主要的便是书和用户，此外还有拥有最高操作权限的数据库管理员。
书和用户存在着：借阅、（反复）预约关系。
其中借阅会有如下情形：按时归还与逾期还书、丢失、续借；

预约则有如下要求：
1.  仅能在到期时间前5天进行续借；
2.  如果此书已被人预约，则不可续借；
3.  多人续借同一本书则按照预约时间进行排队

特别注意的地方：
1.  作者和译者本是书的两个属性，但是现实中存在一本书存在多个作者和译者的情形，所以需要单独为作者和译者建立一个表（由于作者和译者没有属性上的差别，且同一个人既可能是作者有可能是另外书籍的译者），另外建立书-作者表和书-译者表
2.  图书有ISBN号作为同一版本同一书籍的唯一标识码，但是图书管中存在同一本书的多个复本，由此建立书目表和复本表。书目表记录同一书籍的书目信息，复本表记录各个复本的流通情况。
3.  用户分为三类：学生读者、教师读者、图书馆管理员。其中学生读者和教师读者的借阅权限有所不同；图书馆管理员可以修改学生和教师的权限，修改书目信息，但不能修改自己的权限；数据库管理员可以修改三类用户的权限和数据库的其它信息。


E-R图如下：
![](http://ww2.sinaimg.cn/large/8785ef6ejw1f32g1jhg1fj20j90eijt8.jpg)

 
####  逻辑结构设计
共有10张表：
（1）图书书目表

（2）图书复本表

（3）著者（即作者与译者）表

（4）书-作者表

（5）书-译者表

（6）用户表

（7）借阅记录表

（8）预约记录表

（9）数据库管理员表

（10）评论表

各表的具体属性如下：

（1） 图书书目表:yzz_book

 ![](http://ww1.sinaimg.cn/large/8785ef6ejw1f32g2orj85j20ta0543zs.jpg)
 
（2） 图书复本表:yzz_copy

 ![](http://ww1.sinaimg.cn/large/8785ef6ejw1f32g2zkr1vj20er04ewf4.jpg)
 
（3）著者（即作者与译者）表: yzz_creator

| id | Name | country | birthday |
|:------:|:------:|:-----:|:-------:|
|著者的流水编号| 姓名  | 国家 | 出生年份 |

（4）书-作者表：yzz_author

| author_id | book_id |
|:---------:|:-------:|
| 作者的流水编号 | 书目的流水编号 |

（5）书-译者表：yzz_translator

| translatorr_id  | book_id |
|:---------------|:---------|
| 译者的流水编号 | 书目的流水编号 |

（6）用户表

 ![](http://ww2.sinaimg.cn/large/8785ef6ejw1f32g6m7nwkj20vl054wfv.jpg)

（7）借阅记录表：yzz_book_ borrow 

| id |  userid |  bookid |  borrowtime |  returntime |  status |  xjcs| 
|:-----------|:---------:|:---------:|:---------:|:---------:|:---------:|:---------:|
| 借阅记录流水编号 |    用户的账号 |   复本的编号  |  借出时间 |    归还时间（未还则为NULL） |  还书：0未归还，1已归还  |   续借次数 |

（8）预约记录表:yzz_book_ booked

| id |  userid |  bookid |  booktime  |   status| 
|:---------:|:---------:|:---------:|:---------:|:---------:|
| 预约记录流水编号  |   用户的号 |    复本的编号  |  预约时间 |    预约是否有效：仍在预约1，借书成功0| 

（9）数据库管理员表：DBA

| id |  password | 
|:-----:|:-----:|
| DBA账号  |  密码 | 

（10）评论表:remark

| id |  userid |  content|  bookid  | replyid| 
|:------:|:------:|:------:|:------:|:------:|
| 评论的流水编号 | 用户账号|    评论内容  |   评论的书目的编号|     被复的评论的编号，如果为原创话题则为NULL| 

### RBAC角色结构

RBAC：基于角色的访问控制（Role-Based Access Control）

1. 共有4个角色：普通游客，教师/学生，图书馆管理员，DBA
（1）普通游客：可以进行检索、浏览，不能借阅或者预约
（2）教师/学生：除了检索或者浏览之外，登陆之后可以借阅/预约图书。
（3）图书馆管理员：除了拥有“教师/学生”的角色的权限外，可以对“教师/学生”的操作记录、借阅和预约、信用值、借书上限进行修改。
（4）DBA：数据库管理员对数据库拥有最高权限，可以修改教师/学生和图书馆管理员这两类角色的相关记录。

2. 页面模块
（1）Admin：供图书馆管理员操作
（2）DBA：供数据库管理员操作
（3）User：供登录的在校用户操作
（4）Home：检索，在前端使用    
（5）PublicUse：函数、功能部分，在登录时使用
       
### MVC

我们采用了MVC软件构建模式， MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写。
其中：
Model（模型）是应用程序中用于处理应用程序数据逻辑的部分。通常模型对象负责在数据库中存取数据。
View（视图）是应用程序中处理数据显示的部分。通常视图是依据模型数据创建的。
Controller（控制器）是应用程序中处理用户交互的部分。可以接受访问并对访问做出应答。通常控制器负责从视图读取数据，控制用户输入，并向模型发送数据。

MVC的主要优点有
：
1.  MVC 分层有助于管理复杂的应用程序。这是因为可以在一个时间内专门关注一个方面。例如，可以在不依赖业务逻辑的情况下专注于视图设计。同时也让应用程序的测试更加容易。
2.  MVC 分层同时也简化了分组开发。不同的开发人员可同时开发视图、控制器逻辑和业务逻辑。

### 系统安全

（1）密码安全：MD5
使用了MD5（Message Digest Algorithm 5），是目前应用最广泛的Hash算法，具有单向性、抗冲突性、映射分布均匀性和差分分布均匀性等关键特性。
它的算法的特征是不可逆性，并且才计算的时候所有的数据都参与了运算，其中任何一个数据变化了都会导致计算出来的Hash值完全不同，所以通常用来校验数据是否正确或用作身份验证。

我们数据库中的密码是经过MD5等Hash算法算出来的Hash值进行保存的。用户登录时将根据实时输入得到一个Hash值，与数据库中密文储存的密码相匹配。

（2）防止重复登录：Session记录

Session相当于服务器端的缓存，通过Session的记录判断用户是否已经登录，如果不存在相应的记录则跳转到登陆界面。

### 查找功能
（1）系统支持多字段进行检索，检索字段包括标题、文摘、作者等等。
（2）图书检索的显示结果会分页显示，便于用户浏览。
###  图书的相关操作

不同用户可以进行图书相关的操作如下

|普通访客  |   教师/学生  |  图书馆管理员 | DBA |
|:------:|:------:|:------:|:------:|
|查询书目信息|  √ |  √ |  √ |  √|
|借书、还书  |    | √  | √  | | |
|预约与续借  |    | √  | √  | |
|书目信息修改 |   |    |  √ |  |

注：DBA负责整个数据库的宏观维护，不参与具体的书籍流通过程。

### 用户相关操作
对用户的相关操作，仅对图书馆管理员和DBA两类角色开放：
（1）两类角色都可以进行新用户的注册与用户的删除；
（2）修改数据库记录：图书馆管理员可以修改“教师/学生”的相关记录，但不能修改自己或者其他管理员的记录；DBA可以对用户授权，并修改其它所有用户的记录。
（3）用户的借书上限和信用值和角色类型有关。比如教师、学生、图书馆管理员的借书上限分别为40、30、50。之后可由图书馆管理员和DBA进行修改。信用值默认为均为100。

###  记录的保存
小组的数据库会存储借书、预约和续借的记录，这些日志记录可以用于之后的数据分析，进一步应用于文献的推荐，分析书籍与读者的内在联系，使得图书的流通过程成为可以强化图书馆服务职能的工具。

###  还书提醒、罚款与缴费
如果读者距离应还书时间仅有5天时，系统会提醒读者应该及时还书，并提供网上续借的跳转网页链接。当此复本没有被预约时，读者可以进行续借。
如果读者逾期还书，那么相应地会产生罚款与催款两种体系。我们使用的是信用额，在其归还书的时候根据其超时进行超时的罚款。

（1）罚款
如果逾期还书，那么超时1天信用值下降1。信用值默认为100，当信用值下降到10以下时读者不可再借阅其他书籍。
（2）缴费
当信用值下降到10以下时，读者需要向图书馆管理员缴纳欠款，由图书馆管理员手动修改回信用值。
#
#  数控设计
###  参照完整性
为了保证数据库系统正常运行，我们在系统内设置了外键，提供参照完整性方案。
如在book_borrow、book_booked和book三个表中，book_borrow表中的bookid不是该表的主键，但和book表中的id对应，是外键。这种设定能保证在book_borrow中出现的书籍条目在数据库中都能找到，即被借出的书都是图书馆中存在的书。
同样的，book_booked表中的bookid也是外键，这保证了在图书馆系统中被预定的书都是图书馆中存在的书。
另外，在translator表和author表中的bookid也都是外键，这两个外键确保了在译者、著者表中的作者都能和系统中的书籍相对应。
其中，translator是允许为空的，因为中文的书籍不需要翻译，而author是不允许为空的，因为各个图书都有对应的作者（编纂集体）。

### Ajax保证完整性
在设计注册、借还等页面时，我们使用了异步javascript和XML技术，保证局部刷新时不影响整体页面，也不需要用户来回登录注册页面。

Ajax是用于创建快速动态网页的技术，通过在后台和服务器进行少量数据交换，Ajax可以使网页实现异步更新。这意味着不需要重新载入整个网页来显示新内容，而只需要进行局部的刷新来获得变更内容。
在我们的系统中，用户注册时会被要求输入用户名，因为实际情况中可能使用的用户是非常多的，难免造成用户名一致的问题。当用户名与之前注册的用户一致时，会直接显示错误提示，要求用户更换用户名，而不需要在注册提交时再告知重复，让用户重新填写所有内容。
另外，在借书时系统会要求用户登录自己的帐号，此时如果用户未注册，却输入了已经存在的用户名，也会提示用户名被注册。
这种设计主要目的在于为用户节省时间，一方面保证了系统中的注册用户是唯一的，实现了系统完整性的要求；另一方面方便用户注册，不需要一次次输入全部注册信息。
### 时间戳
时间戳是一个字符序列，唯一地标识某一刻的时间。数字时间戳技术是数字签名技术的一种变种。在电子商务交易中，时间戳是重要信息，它和签名一样能防止文件或合同被篡改。
在数据库中，在表上加时间戳来制作索引，能方便我们在后台查找某一时间内被借走的图书，方便管理员的工作。由于时间戳是按格林威治时间至今换算为秒来计算，基本上能够保证唯一性，在检查图书流通记录时是很好的依据。

##   关键功能的算法设计

### ThinkPHP框架
ThinkPHP是快速兼容简单的php开发框架，使用面向对象的开发结构和MVC模式，融合了Struts的思想和标签库、ROR的ORM映射和ActiveRecord模式。ThinkPHP能解决应用开发中的大多数需要，包含了底层架构、兼容处理、基类库、数据库访问层、模版引擎、缓存机制、插件机制、角色认证、表单处理等常用组件。
在使用ThinkPHP框架时，我们能将更多的精力放在图书馆系统的逻辑设计、功能实现上，因为繁琐的基本功能或需要多次使用的程序段都能直接从组件中调用。在图书管理系统中，图书管理员的身份相对于用户和图书记录来说是长期身份，在保存到数据库后一段时间内不会变动。
ThinkPHP中的静态模型功能能在访问数据库后生成缓存，之后再次查看管理员记录表时就不需要再次链接数据库，而直接从缓存文件中查看，能节省时间提高效率。
另外，它还提供了灵活和方便的数据操作方法，实现了对数据库的创建、读取、更新和删除，方便在数据库构建过程中使用。

### ORM
对象关系映射（ORM）是用于实现面向对象编程语言里不同类型系统的数据转换时采用的技术。面向对象是从计算机编程技术的角度衍生的，而关系数据库是从严格的数学模型推导衍生的，ORM为这两个不同领域工具的衔接提供了帮助。
这使得我们在开发图书馆管理系统时，减少数据访问层的代码编写数量，不用频繁地涉及数据库的保存、删除、更新和插入操作。使用ORM保存、删除和更新对象，我们能更集中于功能的实现和解决逻辑问题，而SQL语句的输入和调用都能交给ORM来实现。

### CURD
CURD表示创建、更新、读取和删除四个基本操作。它们在数据库中处于基础位置，主要通过关系型数据库系统中的结构化查询语言（SQL）实现的。
我们的图书馆管理系统能为读者提供基本的读取查询操作，方便其浏览图书。图书管理员和系统管理员则可以参与到图书表内容的创建、更新和删除中。

### RBAC
RBAC是基于角色的访问控制。在RBAC中，权限与角色相关联，用户通过成为适当角色成员来获得这些角色的权限，能极大地简化权限的管理。在一个组织中，角色为了完成某种工作而创造，用户则依据他的责任和资格被委派相应的角色，并可以很容易的在角色之间转换。同时，角色可以因实际需要被赋予新的权限，也能随时收回被赋予的权限。

在我们的图书馆系统中，使用者被分为了四个主要权限：系统管理员、图书管理员、学生和老师。他们分别使用不同的权限。
如教师和学生都是直接用户，他们的权限包括浏览、借阅、预定和续借等；
图书管理员还需要对图书进行登记，在归还时改变图书的状态，因而有操作图书表的权限；系统管理员对整个数据库进行维护，其权限范围更广。
通过对不同的角色赋权，再将角色赋予用户，能极大地减少管理员工作，提高效率。

### 文件上传
在图书管理员后台，可以进行增加图书的操作。我们的系统不仅支持文字的录入，即书目相关信息、作者相关信息等内容的记录；还支持图片上传功能，当管理员登记一本图书时，他还可以选择上传图书封面图片，这样在读者借阅时就能直接浏览图书封面，来确定是否是自己需要的图书。
文件上传功能还为图书馆功能拓展提供了帮助。如可以增加儿童图书馆，儿童在选择书籍时可能更关注书的颜色，对他们而言，鲜亮卡通的颜色搭配更能提高他们的注意力。这时因为我们在图书登记时就加入了封面，能很方便的进行图书颜色分类，实现功能拓展，而不需要重新对每本书的外观进行描述。

### 其他
####   分页
在本次课程作业中，我们的初步设想是向图书馆系统中增加100本书，跟实际情况相比，100本书是很少的量，完全可以通过平铺浏览的方式直接显示出来。
但考虑到实际工作中，图书馆系统面对的常常是十万册、百万册的图书，即便是精确检索的记录也会提供诸多近似结果。这时分页就显得尤为重要了。

分页可以选择相关性最高的图书优先显示在第一页，如果这一页中有读者需要的书本，就能大量减少读者的使用时间，提高借阅效率。同时，如果读者采用的查询词不够精确，返回了大量结果，则可以分页显示来减少视觉压力，也能提高系统的反应速度，每一次响应时间变短，但切换页面时的响应次数变多，变相地减少读者的等待时间，提高用户体验。
####  验证码
验证码可以自动区分使用方是人还是计算机，可以防止恶意破解密码、刷票等恶意行为。对图书馆管理系统来说，在注册、借阅环节采用验证码，可以防止某些用户使用机器高频率、重复注册来破坏系统，或者对一个用户的密码进行多次尝试暴力破解，也能在借书和预定环节的逻辑判断上更加公平。

在我们的系统中，注册时，用户需要参照给出的图片输入其中的数字和字母，如验证正确则判断是人工注册，可以进行下一步环节；如因机器注册而不能识别，则不允许进入注册环节。另外，用户还可以手动刷新验证码来选择有较高清晰度的图片进行验证。
####   日志记录
我们的图书馆系统在图书的每次变动后都会增加一条新记录。如读者A借阅了图书1，日志文件中就会生成一条新纪录。当图书1的借阅时间到期，而读者A还没有看完时，他可以选择续借，这时会生成新记录。而读者B需要图书1，则可以在我们的前端预定这本书，此时新增booked记录。当一本图书被预定又被借阅时，系统会判断先后，被预定的图书不允许再次续借，而预定会排在之前的续借完成后才会提醒。通过日志文件，我们就能清楚的分析图书流通过程。
另一方面，当图书馆系统出现问题，如数据丢失、数据库损坏、病毒或人为破坏时，我们可以读取日志记录来恢复之前的借阅，并在新的记录中延续旧日志，实现图书记录工作长期化和有效化。



##  进一步扩展完善的设想
### 图书的近似匹配
####   摘要的切词处理
由于时间有限，无法收集图书全文信息，而只是录入了相应的摘要信息。因此本图书馆系统的全文检索实质上是根据用户输入的检索词与摘要的匹配来实现的。单纯的全文匹配显然无法满足用户需求，很有可能会遗漏许多和用户需要有高度相关性却和检索词稍有区别的图书。于是我们便不可避免的遇到了摘要的切词处理问题，只有通过合适的中文切分词算法，将摘要自动切分，便可以进行精准的词匹配。
而进行切词之前，还需要使用停用词表，去掉在每篇文摘中都会出现的高频词，如“的”，“本书”之类的词语。当去掉这些词语之后，倒排档的规模会得到极大的缩减，从而使得整个系统的运算速度大幅度提高。
具体的操作步骤便为对照《哈工大停用词表》，去掉在每篇文献中出现频道都很高的词，如“一些”，“啊”，“的”等词语。之后再使用《NLPIR/ICTCLAS汉语分词系统2015版》，对我们的所有文摘进行切分词，进而得到所有出现过的词语的倒排档统计。并进一步利用倒排档统计进行接下来的工作，如tf*idf的计算。
####   tf*idf

通过前文阐述的中文切分词技术，我们已经实现了较为精准的匹配，保证用户输入检索词之后所发起的搜索得到的结果都是和用户实际需求有较高相关性的，接下来需要处理的便是排序的问题。
比如用户输入“信息管理系统”，如何将相关度最高的图书排在最前列的位置，如何使得《管理信息系统》这本书的排序高于《信息管理系历史回顾》。这便是需要通过tf*idf来解决的问题。此外，比较两本书的相关程度，也是通过向量空间模型来计算tf*idf，从而发现不同图书之间的相关性，进而为读者做图书推荐。

通过自动切分词结合通用词表，便可以把所有图书的文摘中出现的关键词提取，进而建立倒排档。而tf(I,j)是指关键词i在文献j中出现的次数，意即其词频。Idf则是分配给关键词的相对权重，其中常用词的idf值较低，而使用较少的词idf值较高。常用的idf公式为idf=log(N/n)，其中N为总的文献篇数，n指某一词语在全部N篇文献中出现了n次。如果某个词语在全部N篇文章中都有出现，则其idf值等于log(1)，等于0。而如果某个关键词则全部N篇文献中总共只出现在某一篇文献中，则其idf值等于log(N)。

此外，我们还需要考虑摘要长度不同带来的问题，200字的摘要中和20字的摘要中，“信息”一词都出现了3次，显然“信息”在20字文摘中的重要程度更高。因此，我们需要将文摘长度归一化。在此处理的办法即为通过文摘中每个词语的绝对词频除以其文摘向量的长度。

当以上三项准备就绪之后，我们便可以通过计算检索词与文摘之间的tf*idf值来进行检索结果的排序。若关键词与文摘的tf*idf结果为0，则统一不显示在检索结果中；其他情况下，则通过比较，将tf*idf值较高的摘要代表的图书排在靠前的位置，将tf*idf值较低的排在靠后的位置。

并且，我们可以用向量空间模型来计算两篇文摘之间的相似度，若两篇文摘完全一样，则其相关度为1，若两篇文摘之间没有任何共同的关键词，则其相关性为0，而在为读者进行相关图书推荐时，我们便只需要将与用户借阅或收藏的图书的相关性较高的图书展现出来即可。

### 通过图书找人的模式进行可视化
通过图书找人的模式是我们图书馆系统的创新所在。传统的思维方式为通过读者的借阅历史记录分析读者感兴趣的话题内容，并根据向量空间模型为读者推荐与该书相似度较高的其他图书。

我们的想法为：
首先，通过数据可视化的方式来展现图书和读者之间的联系，运用到传统的推荐模式上，便是用线条粗细等方式反应读者和图书之间联系的强弱，图书与图书之间联系的强弱。这样读者便可以清楚明了的看到与自己联系较为密切的图书，以及与自己借阅的图书关系较为密切的其他图书。

其次，我们希望突破传统的方式，不仅是为读者推荐图书，而是与此同时做到为书推荐读者。文献领域的二八率表示80%的读者经常借阅的为整个文献资源20%的图书，而剩下的80%的图书则并不常用，很有可能只是被剩下的20%的读者借阅，因而面临着利用率低，甚至是资源浪费的问题。而我们通过帮图书推荐读者这一新颖的思维方式，便可以有效的辅助这一问题的解决。

而且，为书找合适的读者也是出版社和书商特别关注的话题。而如果能准备的为图书推荐合适的读者，对书商和出版社来说也很有可能增加其利润。比如，若借阅《白夜行》的读者有A，B，C，D等四名读者，而出版社即将出版与《白夜行》为同一作者，相似题材的《解忧杂货铺》。为了减小宣传成本，使得广告投放起到最大的效果，出版社通过借阅记录，发现A，B，C，D四名读者与和《解忧杂货铺》相似度很高的《白夜行》的联系非常密切，便将广告投放的重点放到这些读者上。

以这样的方式来有效的节约成本，增加利润。提高文献资源的利用率采取的相似的方法：图书馆经常面临着图书利用率不足的问题，而且经常会面临图书利用率的审核。因此，以这样的方式来获取对新书可能感兴趣的潜在读者，便可以有效的提高图书馆文献资源的利用率。与此同时，还可以使读者阅读到自己感兴趣的图书，相当于一举两得。

综上，我们将数据可视化与为图书推荐读者联结起来，用简单明了直接的方式显示图书与读者的联系，从而为书商、出版社和文化服务机构提供帮助，提高他们的利润或是绩效考核业绩。

##  参考文献：
[1] 王珊，萨师煊，数据库系统概论第四版[M]. 北京，高等教育出版社，2006年.

[2]孟楠. 分布式内存数据库系统设计实现与应用[D].南京理工大学,2005.

[3]郭丽英. 高校图书馆数据库系统中SQL语句的查询重写研究[J]. 科技信息(学术研究),2008,04:220+223.

[4]胡鹰. 中小型图书馆数据库管理系统的设计[J]. 图书馆,1989,02:33-36+17. 

[5]胡根桥. 图书馆流通数据仓库的设计与实现[J]. 现代情报,2007,07:68-70.

