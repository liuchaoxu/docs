---
layout: doc
lang: zh-CN
title: Java开发规范
titleTemplate: 规范
description: java
#导航栏
#navbar: true
#侧边栏
sidebar: true
#侧边大纲默认在右侧 ，通过 aside 设置左侧或关闭，默认 true
aside: true
#右侧的大纲，默认显示是二级标题，通过设置 outline 实现多级标题
#注意:设置到六级标题可以用 'deep' ，关闭 false,此设置与 页面中的大纲 设置相同，会覆盖！
#outline: [2,3]
outline: [1,2,3,4,5,6]
#上次更新时间，默认开启，不想显示可以关闭
lastUpdated: false

#仅更改上/下页显示的文字，跳转还是按照侧边栏配置的读取的
#prev: '页面 | 更详细的页面配置'
#next: 'Markdown | 更详细的markdown'

#更改跳转链接
#  可更改成任意自己想跳转的文章
prev:
  text: '知识地图'
  link: '/docs/知识地图'
#next:
#  text: 'Markdown'
#  link: '/markdown'

#  不想显示可以选择关闭
#prev: false
next: false
#页脚
footer: false

---

# （一）编码规约
## 一，命名
### 1.类名使用UpperCamelCase风格，必须遵从驼峰形式
但以下情形例外：DO / BO / DTO / VO / AO 

<font style="color:#5C8D07;">正例：</font>MarcoPolo / UserDO / XmlService / TcpUdpDeal / TaPromotion 

<font style="color:#E4495B;">反例：</font>macroPolo / UserDo / XMLService / TCPUDPDeal / TAPromotion  

###  2.方法名、参数名、成员变量、局部变量都统一使用lowerCamelCase风格，必须遵从 驼峰形式。
 <font style="color:#5C8D07;">正例：</font> localValue / getHttpMessage() / inputUserId  

###  3.常量命名全部大写，单词间用下划线隔开，力求语义表达完整清楚，不要嫌名字长。
<font style="color:#5C8D07;">正例：</font>MAX_STOCK_COUNT 

<font style="color:#E4495B;">反例：</font>MAX_COUNT  

### 4. 抽象类命名使用Abstract或Base开头；异常类命名使用Exception结尾；测试类 命名以它要测试的类的名称开始，以Test结尾。  
### 5. 中括号是数组类型的一部分，数组定义如下：String[] args; 
<font style="color:#E4495B;">反例：</font>使用String args[]的方式来定义。  

### 6. POJO类中布尔类型的变量，都不要加is，否则部分框架解析会引起序列化错误。
<font style="color:#E4495B;"> 反例：</font>定义为基本数据类型Boolean isDeleted；的属性，它的方法也是isDeleted() 框架在反向解析的时候，“以为”对应的属性名称是deleted，导致属性获取不到，进而抛出异常。  

### 7. 包名统一使用小写，点分隔符之间有且仅有一个自然语义的英语单词。包名统一使用 单数形式，但是类名如果有复数含义，类名可以使用复数形式。 
<font style="color:#5C8D07;">正例： </font>应用工具类包名为com.sanyi.util、类名为MessageUtils（此规则参考 spring 的框架结构）  

### 8. 杜绝完全不规范的缩写，避免望文不知义。
 <font style="color:#E4495B;">反例：</font>AbstractClass“缩写”命名成AbsClass；condition“缩写”命名成 condi，此类随 意缩写严重降低了代码的可阅读性。  



### 9. 为了达到代码自解释的目标，任何自定义编程元素在命名时，使用尽量完整的单词 组合来表达其意。
 <font style="color:#5C8D07;">正例：</font>从远程仓库拉取代码的类命名为PullCodeFromRemoteRepository。

 <font style="color:#E4495B;">反例：</font>变量int a; 的随意命名方式。  

### 10. 如果模块、接口、类、方法使用了设计模式，在命名时体现出具体模式。 说明：将设计模式体现在名字中，有利于阅读者快速理解架构设计理念。
 <font style="color:#5C8D07;">正例：</font>public class OrderFactory; public class LoginProxy; public class ResourceObserver;  

### 11. 接口类中的方法和属性不要加任何修饰符号（public 也不要加），保持代码的简洁 性，并加上有效的Javadoc注释。尽量不要在接口里定义变量，如果一定要定义变量，肯定是 与接口方法相关，并且是整个应用的基础常量。
 <font style="color:#5C8D07;">正例：</font>接口方法签名：void f(); 接口基础常量表示：String COMPANY = "sanyi";

 <font style="color:#E4495B;">反例：</font>接口方法定义：public abstract void f(); 说明：JDK8中接口允许有默认实现，那么这个default方法，是对所有实现类都有价值的默 认实现。  

### 12. 接口和实现类的命名有两套规则： 
1）【强制】对于Service和DAO类，基于SOA的理念，暴露出来的服务一定是接口，内部 的实现类用Impl的后缀与接口区别。

 <font style="color:#5C8D07;">正例：</font>CacheServiceImpl实现CacheService接口。

 2）【推荐】 如果是形容能力的接口名称，取对应的形容词做接口名（通常是–able的形式）。

 <font style="color:#5C8D07;">正例：</font>AbstractTranslator实现 Translatable。  

### 13. 枚举类名建议带上Enum后缀，枚举成员名称需要全大写，单词间用下划线隔开。
 说明：枚举其实就是特殊的常量类，且构造方法被默认强制是私有。

 <font style="color:#5C8D07;">正例：</font>枚举名字为ProcessStatusEnum的成员名称：SUCCESS / UNKOWN_REASON。  



### 14. 各层命名规约：
####  A) Service层方法命名规约
1）新增方法 必须使用采用“insert*()”前缀格式

2）更新方法 必须使用采用“update*()”前缀格式

3）删除方法 必须使用采用“delete*()”前缀格式

4）修改方法 必须使用采用“modify*()”前缀格式

例如：审核状态、出库状态、入库状态、等等修改

5）导入方法 必须使用采用“import*()”前缀格式

特例：当语义严重不符时，涉及多表或多表关联数据修改必须显示的开启事务

####  B) DAO层方法命名规约
 1） 获取单个对象的方法用get做前缀。 

 2） 获取多个对象的方法用list做前缀。

 3） 获取统计值的方法用count做前缀。

 4） 插入的方法用save/insert做前缀。

 5） 删除的方法用remove/delete做前缀。

 6） 修改的方法用update做前缀。

####  C) 领域模型命名规约 
1） 数据对象：xxxDO，xxx即为数据表名。 

2） 数据传输对象：xxxDTO，xxx为业务领域相关的名称。

3） 展示对象：xxxVO，xxx一般为网页名称。 

4） POJO是DO/DTO/BO/VO的统称，禁止命名成xxxPOJO。  





## 二，常量
### 1，不允许任何魔法值（即未经定义的常量）直接出现在代码中。
 <font style="color:#E4495B;">反例：</font>String key = "Id#taobao_" + tradeId; cache.put(key, value); 



### 2. long或者Long初始赋值时，使用大写的L，不能是小写的l，小写容易跟数字1混 淆，造成误解。
 说明：Long a = 2l; 写的是数字的21，还是Long型的2? 



### 3. 不要使用一个常量类维护所有常量，按常量功能进行归类，分开维护。
 说明：大而全的常量类，非得使用查找功能才能定位到修改的常量，不利于理解和维护。

 <font style="color:#5C8D07;">正例：</font>缓存相关常量放在类CacheConsts下；系统配置相关常量放在类ConfigConsts下。



###  4. 常量的复用：应用内共享常量、包 内共享常量、类内共享常量。
 1） 应用内共享常量：放置在一方库中，通常是modules中的constant目录下。

 <font style="color:#E4495B;">反例：</font>易懂变量也要统一定义成应用内共享常量，两位攻城师在两个类中分别定义了表示 “是”的变量： 类A中：public static final String YES = "yes"; 类B中：public static final String YES = "y"; A.YES.equals(B.YES)，预期是true，但实际返回为false，导致线上问题。 

在我们项目中：com.sanyi.tools.constant

2） 包内共享常量：即在当前包下单独的constant目录下。 

3） 类内共享常量：直接在类内部private static final定义。

### 5. 如果变量值仅在一个范围内变化，且带有名称之外的延伸属性，定义为枚举类。
下面 <font style="color:#5C8D07;">正例</font>中的数字就是延伸信息，表示星期几。 

<font style="color:#5C8D07;">正例：</font>

public Enum { MONDAY(1), TUESDAY(2), WEDNESDAY(3), THURSDAY(4), FRIDAY(5), SATURDAY(6), SUNDAY(7);}  

## 三，代码格式
### 1，IDE的text file encoding设置为UTF-8
### 2，IDE中文件的换行符使用Unix格式：\n，不要使用 Windows格式


## 四，OOP
### 1, 所有的覆写方法，必须加 @Override注解。
 说明：getObject() 与get0bject() 的问题。一个是字母的O，一个是数字的0，加 @Override可以准确判断是否覆盖 成功。另外，如果在抽象类中对方法签名进行修改，其实现类会马上编译报错。  



### 2, 相同参数类型，相同业务含义，才可以使用的可变参数，参数类型避免定义为Object。
 说明：可变参数必须放置在参数列表的最后。（建议开发者尽量不用可变参数编程） <font style="color:#5C8D07;">正例：</font>public List listUsers(String type, Long... ids) {...} 



###  3,外部正在调用的接口或者二方库依赖的接口，不允许修改方法签名，避免对接口调用方产生影响。
接口过时必须加 @Deprecated注解，并清晰地说明采用的新接口或者新服务是什么。  



### 4, 不能使用过时的类或方法。
 说明：java.net.URLDecoder 中的方法decode(String encodeStr) 这个方法已经过时，应该使用双参数 decode(String source, String encode)。接口提供方既然明确是过时接口，那么有义务同时提供新的接口；作为调用 方来说，有义务去考证过时方法的新实现是什么。  

### 5,Object的equals方法容易抛空指针异常，应使用常量或确定有值的对象来调用equals。
<font style="color:#5C8D07;"> 正例：</font>"test".equals(param);

<font style="color:#E4495B;"> 反例：</font>param.equals("test"); 

说明：推荐使用JDK7引入的工具类java.util.Objects#equals(Object a, Object b)  

### 6,BigDecimal的等值比较应使用compareTo() 方法，而不是equals() 方法。
 说明：equals() 方法会比较值和精度（1.0 与 1.00 返回结果为 false），而compareTo() 则会忽略精度。  

### 7, 禁止使用构造方法BigDecimal(double) 的方式把double值转化为BigDecimal对象。
说明：BigDecimal(double) 存在精度损失风险，在精确计算或值比较的场景中可能会导致业务逻辑异常。

 如： BigDecimal g = new BigDecimal(0.1F)；

实际的存储值为：0.100000001490116119384765625 

<font style="color:#5C8D07;">正例：</font>

优先推荐入参为String的构造方法，或使用BigDecimal的valueOf方法，

此方法内部其实执行了Double的 toString，而 Double 的toString 按 double的实际能表达的精度对尾数进行了截断。

BigDecimal recommend1 = new BigDecimal("0.1"); 

BigDecimal recommend2 = BigDecimal.valueOf(0.1);  

###  8,关于基本数据类型与包装数据类型的使用标准如下：
 1）【强制】所有的POJO类属性必须使用包装数据类型。

 2）【强制】RPC方法的返回值和参数必须使用包装数据类型。

 3）【推荐】所有的局部变量使用基本数据类型。 

 说明：POJO类属性没有初值是提醒使用者在需要使用时，必须自己显式地进行赋值，任何NPE问题，或者入库检查， 都由使用者来保证。 

<font style="color:#5C8D07;">正例：</font>数据库的查询结果可能是null，因为自动拆箱，用基本数据类型接收有NPE风险。

 <font style="color:#E4495B;">反例：</font>某业务的报表上显示成交涨跌情况，即正负x%，x为基本数据类型，调用的RPC服务，调用不成功时， 返回的是默认值，页面显示为0%，这是不合理的，应该显示成中划线-。所以包装数据类型的null值，能够表示额外的 信息，如：远程调用失败，异常退出。  

### 9,定义DO / PO / DTO / VO等POJO类时，不要设定任何属性默认值。
<font style="color:#E4495B;">反例：</font>某业务的DO的createTime默认值为new Date()；但是这个属性在数据提取时并没有置入具体值，在更新其 它字段时又附带更新了此字段，导致创建时间被修改成当前时间。  

### 10.构造方法里面禁止加入任何业务逻辑，如果有初始化逻辑，请放在init方法中。  
###  11.POJO类必须写toString方法。
使用IDE中的工具source > generate toString时，如果继 承了另一个POJO类，注意在前面加一下super.toString()。

 说明：在方法执行抛出异常时，可以直接调用POJO的toString() 方法打印其属性值，便于排查问题。 

###  12.禁止在POJO类中，同时存在对应属性xxx的isXxx() 和getXxx() 方法。
 说明：框架在调用属性xxx的提取方法时，并不能确定哪个方法一定是被优先调用到，神坑之一。   



## 五，日期时间
###  1.日期格式化时，传入pattern中表示年份统一使用小写的y。 
说明：日期格式化时，yyyy表示当天所在的年，而大写的YYYY代表是week in which year（JDK7之后引入的概念）， 意思是当天所在的周属于的年份，一周从周日开始，周六结束，只要本周跨年，返回的YYYY就是下一年。 

<font style="color:#5C8D07;">正例：</font>表示日期和时间的格式如下所示： new SimpleDateFormat("yyyy-MM-dd HH:mm:ss") 

<font style="color:#E4495B;">反例：</font>某程序员因使用YYYY/MM/dd进行日期格式化，2017/12/31执行结果为2018/12/31，造成线上故障。 

### 2.在日期格式中分清楚大写的M和小写的m，大写的H和小写的h分别指代的意义。
 说明：日期格式中的这两对字母表意如下： 

1）表示月份是大写的M 

2）表示分钟则是小写的m 

3）24小时制的是大写的H 

4）12小时制的则是小写的h 

### 3.获取当前毫秒数：System.currentTimeMillis()；而不是new Date().getTime()。
 说明：获取纳秒级时间，则使用System.nanoTime的方式。在JDK8中，针对统计时间等场景，推荐使用Instant类。 

### 4.不允许在程序任何地方中使用：
1）java.sql.Date 

2）java.sql.Time 

3）java.sql.Timestamp。 

说明：

第1个不记录时间，getHours() 抛出异常；

第2个不记录日期，getYear() 抛出异常；

第3个在构造方法 super((time / 1000) * 1000)，在 Timestamp 属性 fastTime和 nanos 分别存储秒和纳秒信息。

<font style="color:#E4495B;"> 反例：</font>java.util.Date.after(Date) 进行时间比较时，当入参是java.sql.Timestamp 时，会触发JDK BUG（JDK9已修 复），可能导致比较时的意外结果。 

### 5.禁止在程序中写死一年为365天，避免在公历闰年时出现日期转换错误或程序逻辑错误。
<font style="color:#5C8D07;"> 正例：</font>

 // 获取今年的天数 

int daysOfThisYear = LocalDate.now().lengthOfYear();

 // 获取指定某年的天数

 LocalDate.of(2011, 1, 1).lengthOfYear(); 

<font style="color:#E4495B;">反例：</font>

 // 第一种情况：在闰年366天时，出现数组越界异常

 int[] dayArray = new int[365];

 // 第二种情况：一年有效期的会员制，2020年1月26日注册，硬编码365返回的却是2021年1月25日

 Calendar calendar = Calendar.getInstance(); calendar.set(2020, 1, 26);

 calendar.add(Calendar.DATE, 365); 

【注意】避免公历闰年2月问题。闰年的2月份有29天，一年后的那一天不可能是2月29日。 

### 6.使用枚举值来指代月份。
如果使用数字，注意Date，Calendar等日期相关类的月份month取 值范围从0到11之间。

 说明：参考JDK原生注释，Month value is 0-based. e.g., 0 for January. 

<font style="color:#5C8D07;">正例：</font>Calendar.JANUARY，Calendar.FEBRUARY，Calendar.MARCH 等来指代相应月份来进行传参或比较。  







##  六，集合处理
###  1.【强制】关于hashCode和equals的处理，遵循如下规则： 
1）只要覆写equals，就必须覆写hashCode。 

2）因为Set存储的是不重复的对象，依据hashCode和equals进行判断，所以Set存储的对象必须覆写这两种方法。 

3）如果自定义对象作为Map的键，那么必须覆写hashCode和equals。 说明：String 因为覆写了hashCode和equals方法，所以可以愉快地将String对象作为key来使用。



###  2.【强制】判断所有集合内部的元素是否为空，使用isEmpty() 方法，而不是size() == 0的方式。
 说明：在某些集合中，前者的时间复杂度为O(1)，而且可读性更好。

 <font style="color:#5C8D07;">正例：</font> Map map = new HashMap<>(16); if (map.isEmpty()) { System.out.println("no element in this map."); } 

### 3.【强制】在使用java.util.stream.Collectors 类的 toMap() 方法转为 Map 集合时，一定要使用参数类型 为BinaryOperator，参数名为mergeFunction 的方法，否则当出现相同key时会抛出 IllegalStateException 异常。
 说明：参数mergeFunction的作用是当出现key重复时，自定义对value的处理策略。 

<font style="color:#5C8D07;">正例：</font>

 List> pairArrayList = new ArrayList<>(3);

 pairArrayList.add(new Pair<>("version", 12.10));

 pairArrayList.add(new Pair<>("version", 12.19)); 

pairArrayList.add(new Pair<>("version", 6.28));

 // 生成的map集合中只有一个键值对：

{version=6.28} Map map = pairArrayList.stream() .collect(Collectors.toMap(Pair::getKey, Pair::getValue, (v1, v2) -> v2)); 

<font style="color:#E4495B;">反例： </font>

String[] departments = new String[]{"RDC", "RDC", "KKB"}; 

// 抛出IllegalStateException 异常

 Map map = Arrays.stream(departments) .collect(Collectors.toMap(String::hashCode, str -> str)); 



### 4.【强制】在使用java.util.stream.Collectors 类的 toMap() 方法转为 Map 集合时，一定要注意当value 为null 时会抛NPE异常。 
说明：

在java.util.HashMap 的merge方法里会进行如下的判断：

 if (value == null || remappingFunction == null) throw new NullPointerException(); 

<font style="color:#E4495B;">反例：</font>

 List> pairArrayList = new ArrayList<>(2); 

pairArrayList.add(new Pair<>("version1", 8.3));

 pairArrayList.add(new Pair<>("version2", null)); 

// 抛出 NullPointerException 异常

 Map map = pairArrayList.stream()

 .collect(Collectors.toMap(Pair::getKey, Pair::getValue, (v1, v2) -> v2)); 

### 5.【强制】ArrayList 的subList 结果不可强转成ArrayList，否则会抛出ClassCastException异常： java.util.RandomAccessSubList cannot be cast to java.util.ArrayList。 
说明：

subList() 返回的是ArrayList的内部类SubList，并不是ArrayList本身，而是ArrayList的一个视图，对于 SubList 的所有操作最终会反映到原列表上。 

### 6.【强制】使用Map的方法keySet() / values() / entrySet() 返回集合对象时，不可以对其进行添加元素 操作，否则会抛出UnsupportedOperationException 异常。
###  7.【强制】Collections 类返回的对象，如：emptyList() / singletonList() 等都是 immutable list，不可 对其进行添加或者删除元素的操作。
 <font style="color:#E4495B;">反例：</font>如果查询无结果，返回Collections.emptyList() 空集合对象，调用方一旦在返回的集合中进行了添加元素的操 作，就会触发UnsupportedOperationException 异常。 

### 8.【强制】在subList 场景中，高度注意对父集合元素的增加或删除，均会导致子列表的遍历、增加、删 除产生ConcurrentModificationException 异常。
 说明：抽查表明，90% 的程序员对此知识点都有错误的认知。

###  9.【强制】使用集合转数组的方法，必须使用集合的toArray(T[] array)，传入的是类型完全一致、长度为 0 的空数组。 
<font style="color:#E4495B;">反例：</font>直接使用toArray无参方法存在问题，此方法返回值只能是Object[]类，若强转其它类型数组将出现 ClassCastException 错误。

 <font style="color:#5C8D07;">正例：</font>

 List list = new ArrayList<>(2);

 list.add("guan"); 

list.add("bao");

 String[] array = list.toArray(new String[0]);

 说明：使用toArray带参方法，数组空间大小的length： 

1）等于0，动态创建与size相同的数组，性能最好。 

2）大于0但小于size，重新创建大小等于size的数组，增加GC负担。 

3）等于size，在高并发情况下，数组创建完成之后，size正在变大的情况下，负面影响与2相同。 

4）大于size，空间浪费，且在size处插入null值，存在NPE隐患。

###  10.【强制】使用Collection接口任何实现类的addAll() 方法时，要对输入的集合参数进行NPE判断。
 说明：在ArrayList#addAll 方法的第一行代码即Object[] a = c.toArray()；其中c为输入集合参数，如果为null， 则直接抛出异常。 

### 11.【强制】使用工具类Arrays.asList() 把数组转换成集合时，不能使用其修改集合相关的方法，它的add / remove / clear 方法会抛出UnsupportedOperationException 异常。
 说明：asList的返回对象是一个Arrays内部类，并没有实现集合的修改方法。Arrays.asList体现的是适配器模式，只 是转换接口，后台的数据仍是数组。

 String[] str = new String[]{ "yang", "guan", "bao" };

 List list = Arrays.asList(str); 

第一种情况：list.add("yangguanbao"); 运行时异常。

 第二种情况：str[0] = "change"; list 中的元素也会随之修改，反之亦然。

###  12.【强制】泛型通配符来接收返回的数据，此写法的泛型集合不能使用add方法， 而不能使用get方法，两者在接口调用赋值的场景中容易出错。
 说明：扩展说一下PECS(Producer Extends Consumer Super) 原则，即频繁往外读取内容的，适合用 ，经常往里插入的，适合用 

### 13.【强制】在无泛型限制定义的集合赋值给泛型限制的集合时，在使用集合元素时，需要进行 instanceof 判断，避免抛出ClassCastException 异常。
 说明：毕竟泛型是在JDK5后才出现，考虑到向前兼容，编译器是允许非泛型集合与泛型集合互相赋值。

<font style="color:#E4495B;"> 反例：</font>

 List generics = null;

 List notGenerics = new ArrayList(10);

 notGenerics.add(new Object());

 notGenerics.add(new Integer(1)); 

generics = notGenerics; // 此处抛出ClassCastException 异常 String string = generics.get(0); 

### 14.【强制】不要在foreach循环里进行元素的remove / add操作。remove元素请使用iterator方式， 如果并发操作，需要对iterator对象加锁。
<font style="color:#5C8D07;"> 正例：</font>

 List list = new ArrayList<>();

 list.add("1");

 list.add("2");

 Iterator iterator = list.iterator();

 while (iterator.hasNext()) {

 String item = iterator.next();

 if (删除元素的条件) { 

iterator.remove(); 

} 

 } 

<font style="color:#E4495B;">反例：</font>

 for (String item : list) {

 if ("1".equals(item)) { 

list.remove(item); 

} 

}

 说明：<font style="color:#E4495B;">反例</font>中的执行结果肯定会出乎大家的意料，那么试一下把“1”换成“2”会是同样的结果吗？ 

### 15.【强制】在JDK7版本及以上，Comparator实现类要满足如下三个条件，不然Arrays.sort， Collections.sort 会抛 IllegalArgumentException 异常。
 说明：

三个条件如下 

1）x，y的比较结果和y，x的比较结果相反。 

2）x > y，y > z，则x > z。 

3）x = y，则x，z比较结果和y，z比较结果相同。 

<font style="color:#E4495B;">反例：</font>

下例中没有处理相等的情况，交换两个对象判断结果并不互反，不符合第一个条件，在实际使用中可能会出现异 常。

 new Comparator() {

 	@Override public int compare(Student o1, Student o2) { 

return o1.getId() > o2.getId() ? 1 : -1; 

}

 }; 



##  七，并发处理
###  1.【强制】获取单例对象需要保证线程安全，其中的方法也要保证线程安全。
 说明：资源驱动类、工具类、单例工厂类都需要注意。

###  2.【强制】创建线程或线程池时请指定有意义的线程名称，方便出错时回溯。
 <font style="color:#5C8D07;">正例：</font>自定义线程工厂，并且根据外部特征进行分组，比如，来自同一机房的调用，把机房编号赋值给 whatFeatureOfGroup： public class UserThreadFactory implements ThreadFactory { private final String namePrefix; private final AtomicInteger nextId = new AtomicInteger(1); // 定义线程组名称，在利用jstack来排查问题时，非常有帮助 UserThreadFactory(String whatFeatureOfGroup) { namePrefix = "FromUserThreadFactory's" + whatFeatureOfGroup + "-Worker-"; } @Override public Thread newThread(Runnable task) { String name = namePrefix + nextId.getAndIncrement(); Thread thread = new Thread(null, task, name, 0, false); System.out.println(thread.getName()); return thread; } } 

### 3.【强制】线程资源必须通过线程池提供，不允许在应用中自行显式创建线程。
 说明：线程池的好处是减少在创建和销毁线程上所消耗的时间以及系统资源的开销，解决资源不足的问题。如果不使用 线程池，有可能造成系统创建大量同类线程而导致消耗完内存或者“过度切换”的问题。 

### 4.【强制】线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方 式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。
 说明：Executors 返回的线程池对象的弊端如下： 1）FixedThreadPool 和 SingleThreadPool： 允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。 2）CachedThreadPool： 允许的创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。 3）ScheduledThreadPool：  允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。 

### 5.【强制】SimpleDateFormat 是线程不安全的类，一般不要定义为static变量，如果定义为static，必须 加锁，或者使用DateUtils工具类。
 <font style="color:#5C8D07;">正例：</font>注意线程安全，使用DateUtils。亦推荐如下处理： private static final ThreadLocal dateStyle = new ThreadLocal() { @Override protected DateFormat initialValue() { return new SimpleDateFormat("yyyy-MM-dd"); } }; 说明：如果是JDK8的应用，可以使用Instant代替Date，LocalDateTime代替Calendar，DateTimeFormatter代替 SimpleDateFormat，官方给出的解释：simple beautiful strong immutable thread-safe。 

### 6.【强制】必须回收自定义的ThreadLocal变量，尤其在线程池场景下，线程经常会被复用，如果不清理 自定义的ThreadLocal变量，可能会影响后续业务逻辑和造成内存泄露等问题。尽量在代理中使用 try-finally 块进行回收。
 <font style="color:#5C8D07;">正例：</font> objectThreadLocal.set(userInfo); try { // ... } finally { objectThreadLocal.remove(); } 

7.【强制】高并发时，同步调用应该去考量锁的性能损耗。能用无锁数据结构，就不要用锁；能锁区块，就 不要锁整个方法体；能用对象锁，就不要用类锁。 说明：尽可能使加锁的代码块工作量尽可能的小，避免在锁代码块中调用RPC方法。 

### 8.【强制】对多个资源、数据库表、对象同时加锁时，需要保持一致的加锁顺序，否则可能会造成死锁。
 说明：线程一需要对表A、B、C依次全部加锁后才可以进行更新操作，那么线程二的加锁顺序也必须是A、B、C，否则可 能出现死锁。 

### 9.【强制】在使用阻塞等待获取锁的方式中，必须在try代码块之外，并且在加锁方法与try代码块之间没 有任何可能抛出异常的方法调用，避免加锁成功后，在finally中无法解锁。
 说明一：在lock方法与try代码块之间的方法调用抛出异常，无法解锁，造成其它线程无法成功获取锁。 说明二：如果lock方法在try代码块之内，可能由于其它方法抛出异常，导致在finally代码块中，unlock对未加锁的对 象解锁，它会调用AQS的tryRelease方法（取决于具体实现类），抛出IllegalMonitorStateException异常。 说明三：在Lock对象的lock方法实现中可能抛出unchecked异常，产生的后果与说明二相同。<font style="color:#5C8D07;"> </font>

<font style="color:#5C8D07;">正例：</font> Lock lock = new XxxLock(); // ... lock.lock(); try { doSomething(); doOthers(); } finally { lock.unlock(); } 

<font style="color:#E4495B;">反例：</font> Lock lock = new XxxLock();  // ... try { // 如果此处抛出异常，则直接执行finally代码块 doSomething(); // 无论加锁是否成功，finally代码块都会执行 lock.lock(); doOthers(); } finally { lock.unlock(); } 

### 10.【强制】在使用尝试机制来获取锁的方式中，进入业务代码块之前，必须先判断当前线程是否持有锁。
 锁的释放规则与锁的阻塞等待方式相同。 说明：Lock对象的unlock方法在执行时，它会调用AQS的tryRelease方法（取决于具体实现类），如果当前线程不 持有锁，则抛出IllegalMonitorStateException 异常。<font style="color:#5C8D07;"> 正例：</font> Lock lock = new XxxLock(); // ... boolean isLocked = lock.tryLock(); if (isLocked) { try { doSomething(); doOthers(); } finally { lock.unlock(); } } 

### 11.【强制】并发修改同一记录时，避免更新丢失，需要加锁。
要么在应用层加锁，要么在缓存加锁，要么 在数据库层使用乐观锁，使用version作为更新依据。 说明：如果每次访问冲突概率小于20%，推荐使用乐观锁，否则使用悲观锁。乐观锁的重试次数不得小于3次。 

### 12.【强制】多线程并行处理定时任务时，Timer运行多个TimeTask时，只要其中之一没有捕获抛出的异 常，其它任务便会自动终止运行，使用ScheduledExecutorService则没有这个问题。 
## 八，控制语句
###  1.【强制】在一个switch块内，每个case要么通过continue / break / return等来终止，要么注释说明 程序将继续执行到哪一个case为止；在一个switch块内，都必须包含一个default语句并且放在最 后，即使它什么代码也没有。 
说明：注意break是退出switch语句块，而return是退出方法体。 

### 2.【强制】当switch括号内的变量类型为String并且此变量为外部参数时，必须先进行null判断。
<font style="color:#E4495B;"> 反例：</font>

如下的代码输出是什么？ public class SwitchString { public static void main(String[] args) { method(null); } public static void method(String param) { switch (param) { // 肯定不是进入这里 case "sth":  System.out.println("it's sth"); break; // 也不是进入这里 case "null": System.out.println("it's null"); break; // 也不是进入这里 default: System.out.println("default"); } } } 

### 3.【强制】在if / else / for / while / do语句中必须使用大括号。
 <font style="color:#E4495B;">反例：</font> if (condition) statements; 

说明：即使只有一行代码，也要采用大括号的编码方式。 

### 4.【强制】三目运算符condition ? 表达式1：表达式2中，高度注意表达式1和2在类型对齐时，可能 抛出因自动拆箱导致的NPE异常。
 说明：以下两种场景会触发类型对齐的拆箱操作：

 1）表达式1或 表达式2的值只要有一个是原始类型。

 2）表达式1或 表达式2的值的类型不一致，会强制拆箱升级成表示范围更大的那个类型。

<font style="color:#E4495B;"> 反例：</font>

Integer a = 1; 

Integer b = 2;

 Integer c = null; 

Boolean flag = false;

 // a*b 的结果是int类型，那么c会强制拆箱成int类型，抛出NPE异常 Integer result = (flag ? a * b : c); 

### 5.【强制】在并发场景中，避免使用“等于”判断作为中断或退出的条件。
 说明：

如果并发控制没有处理好，容易产生等值判断被“击穿”的情况，使用大于或小于的区间判断条件来代替。

 <font style="color:#E4495B;">反例：</font>

判断剩余奖品数量等于0时，终止发放奖品，但因为并发处理错误导致奖品数量瞬间变成了负数，这样的话， 活动无法终止。  

##  九，注释规约
###  1.【强制】类、类属性、类方法的注释必须使用Javadoc规范，使用 /** 内容 */ 格式，不得使用 // xxx 方式。
 说明：在IDE编辑窗口中，Javadoc方式会提示相关注释，生成Javadoc可以正确输出相应注释；在IDE中，工程调用 方法时，不进入方法即可悬浮提示方法、参数、返回值的意义，提高阅读效率。

###  2.【强制】所有的抽象方法（包括接口中的方法）必须要用Javadoc注释、除了返回值、参数异常说明 外，还必须指出该方法做什么事情，实现什么功能。 说明：对子类的实现要求，或者调用注意事项，请一并说明。 
### 3.所有的类都添加创建者和创建日期。
 说明：在设置模板时，日期 的设置统一为yyyy/MM/dd的格式。 

###  4.【强制】方法内部单行注释，在被注释语句上方另起一行，使用 // 注释。方法内部多行注释使用 /* */ 注释，注意与代码对齐。
### 5.【强制】所有的枚举类型字段必须要有注释，说明每个数据项的用途。  


##  十，其他
###  1.在使用正则表达式时，利用好其预编译功能，可以有效加快正则匹配速度。
 说明：

不要在方法体内定义：Pattern pattern = Pattern.compile("规则"); 

### 2.避免用ApacheBeanutils进行属性的copy。 
说明：

ApacheBeanUtils 性能较差，可以使用其他方案比如SpringBeanUtils，CglibBeanCopier，注意均是浅拷贝。

### 3.注意Math.random() 这个方法返回是double类型，注意取值的范围0 ≤ x < 1（能够 取到零值，注意除零异常），如果想获取整数类型的随机数，不要将x放大10的若干倍然后取 整，直接使用Random对象的nextInt或者nextLong方法。 
### 4.枚举enum（括号内）的属性字段必须是私有且不可变。 
### 5.任何数据结构的构造或初始化，都应指定大小，避免数据结构无限增长吃光内存。 
### 6.及时清理不再使用的代码段或配置信息。
 说明：对于垃圾代码或过时配置，坚决清理干净，避免程序过度臃肿，代码冗余。

<font style="color:#5C8D07;"> 正例：</font>

对于暂时被注释掉，后续可能恢复使用的代码片断，在注释代码上方，统一规定使用三个斜杠(///) 来说明注释掉代码的理由：

 public static void hello() {

 /// 业务方通知活动暂停

 // Business business = new Business(); 

// business.active(); 

System.out.println("it's finished"); 

}  

# （二）MySQL数据库
##   一，ORM映射
###  1.【强制】在表查询中，一律不要使用 * 作为查询的字段列表，需要哪些字段必须明确写明。
 说明： 

1）增加查询分析器解析成本。 

2）增减字段容易与resultMap配置不一致。

3）无用字段增加网络消耗，尤其是text类型的字段。 

### 2.【强制】POJO类的布尔属性不能加is，要求在resultMap中进行字段与属 性之间的映射。 
说明：参见定义POJO类以及数据库字段定义规定，在sql.xml增加映射，是必须的。

###  3.【强制】不要用resultClass当返回参数，即使所有类属性名与数据库字段一一对应，也需要定义 ；反过来，每一个表也必然有一个与之对应。 
说明：配置映射关系，使字段与DO类解耦，方便维护。 

### 4.【强制】sql.xml 配置参数使用：#{}，#param# 不要使用 ${} 此种方式容易出现SQL注入。
###  5.【强制】iBATIS自带的queryForList(String statementName，int start，int size) 不推荐使用。
 说明：其实现方式是在数据库取到statementName对应的SQL语句的所有记录，再通过subList取start，size 的子集合。

<font style="color:#5C8D07;">正例：</font>

 Map map = new HashMap<>(16);

 map.put("start", start);

 map.put("size", size); 

### 6.【强制】不允许直接拿HashMap与Hashtable作为查询结果集的输出。
 <font style="color:#E4495B;">反例：</font>为避免写一个xxx，直接使用Hashtable来接收数据库返回结果，结果出现 日常是把bigint转成Long值，而线上由于数据库版本不一样，解析成BigInteger，导致线上问题。

###  7.【强制】更新数据表记录时，必须同时更新记录对应的update_time字段值为当前时间。 
### 8.【强制】不要写一个大而全的数据更新接口。
传入为POJO类，不管是不是自己的目标更新字段，都进行

 update table set c1 = value1 , c2 = value2 , c3 ==value3；

这是不对的。

执行SQL 时，不要更新无改动的字段，一是易出错；二是效率低；三是增加binlog存储。 





# （三）事务
## 一，防事务失效策略


代理模式检查：确保Spring代理生效



异常处理规范：避免catch块吞噬异常



访问控制：事务方法必须为public



## 二，事务声明准则


强制指定隔离级别和传播行为

```java
@Transactional(
    isolation = 一般情况下都使用默认,
    propagation = Propagation.REQUIRED,
    rollbackFor = spring默认回滚的是运行时异常如果想要更加严谨，建议指定类型为Throwable.class
)
```



手动实现

```java
TransactionTemplate template = new TransactionTemplate(transactionManager);
template.execute(status -> {
    try {
        // 业务逻辑
        return result;
    } catch (Exception e) {
        status.setRollbackOnly();  // 强制回滚
        throw e;
    } 
});
```



#  附1：专有名词解释
  
**POJO**（Plain Ordinary Java Object）：在本规约中，POJO专指只有setter / getter / toString 的简单类，包括  
DO / DTO / BO / VO 等。 

  
**DO**（Data Object）：专指数据库表一 一对应的POJO类。  此对象与数据库表结构一 一对应，通过**DAO**层向上传输数据源对象。 

  
**PO**（Persistent Object）：也指数据库表一 一对应的POJO类。  此对象与数据库表结构一 一对应，通过DAO层向上传输数据源对象。 

  
**DTO**（Data Transfer Object ）：数据传输对象，Service或Manager向外传输的对象。 

  
**BO**（Business Object）：业务对象，可以由Service层输出的封装业务逻辑的对象。 

  
**Query**：数据查询对象，各层接收上层的查询请求。注意超过2个参数的查询封装，禁止使用Map类来传输。 

  
**VO**（View Object）：显示层对象，通常是Web向模板渲染引擎层传输的对象。 

  
**CAS**（Compare And Swap）  ：解决多线程并行情况下使用锁造成性能损耗的一种机制，这是硬件实现的原子操作。CAS 操作包含三个操作数：内存位置、预期原值和新值。如果内存位置的值与预期原值相匹配，那么处理器会自动将该位置值更新为新值。否则，处理器不做任何操作。 

  
**GAV**（GroupId、ArtifactId、Version）：Maven 坐标，是用来唯一标识jar包。 

  
**OOP**（Object Oriented Programming）：本文泛指类、对象的编程处理方式。 

  
**AQS**（AbstractQueuedSynchronizer）：利用先进先出队列实现的底层同步工具类，它是很多上层同步实现类的基础，比如： ReentrantLock、CountDownLatch、  Semaphore 等，它们通过继承AQS实现其模版方法，然后将AQS子类作为同步组件的内部类，通常命名为Sync。 

  
**ORM**（Object Relation Mapping）：对象关系映射，对象领域模型与底层数据之间的转换，本文泛指iBATIS，  
mybatis 等框架。 

  
**NPE**（java.lang.NullPointerException）：空指针异常。 

  
**OOM**（Out Of Memory）：源于java.lang.OutOfMemoryError，当JVM没有足够的内存来为对象分配空间并且垃圾回收器也无法回收空间时，系统出现的严重状况。 

  
**GMT**（Greenwich Mean Time）：指位于英国伦敦郊区的皇家格林尼治天文台的标准时间，因为本初子午线被定义在通过那里的经线。地球每天的自转是有些不规则的，而且正在缓慢减速，现在的标准时间是协调世（UTC），它由原子钟提供。





