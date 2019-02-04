
# DictionaryMini
## 为什么写这么一篇博客?   
### 从一道亲身经历的面试题说起. 
半年前,我参加我现在所在公司的面试,面试官给了一道题,说有一个Y形的链表,知道起始节点,找出交叉节点.   
![Y形链表](https://raw.githubusercontent.com/liuzhenyulive/DictionaryMini/master/Pic/chain.gif)  
为了便于描述,我把上面的那条线路称为线路1,下面的称为线路2.    
####  思路1:  
先判断线路1的第一个节点的下级节点是否是线路2的第一个节点,如果不是,再判断是不是线路2的第二个,如果也不是,判断是不是第三个节点,一直到最后一个.  
如果第一轮没找到,再按以上思路处理线路一的第二个节点,第三个,第四个... 找到为止.  
时间复杂度n<sup>2</sup>,相信如果我用的是这种方法,可肯定被Pass了.  
####  思路2:  
首先,我遍历线路2的所有节点,把节点的索引作为key,下级节点索引作为value存入字典中.
然后,遍历线路1中节点,判断字典中是否包含该节点的下级节点索引的key,即`dic.ContainsKey((node.next)`  ,如果包含,那么该下级节点就是交叉节点了.
时间复杂度是n.  
那么问题来了,面试官问我了,为什么时间复杂度n呢?你有没有研究过字典的`ContainsKey`这个方法呢?难道它不是通过遍历内部元素来判断Key是否存在的呢?如果是的话,那时间复杂度还是n<sup>2</sup>才是呀?  
我当时支支吾吾,确实不明白字典的工作原理,厚着面皮说 "不是的,它是通过哈希表直接拿出来的,不用遍历",面试官这边是敷衍过去了,但在我心里却留下了一个谜,已经入职半年多了,马上要过年,今年的疑惑不能留到明年,那么,好好来研究一下.  
## 带着问题来阅读  
在看这篇文章前,不知道您使用字典的时候是否有过这样的疑问.  
1. 字典为什么能无限地Add呢?
2. 从字典中取Item速度非常快,为什么呢?
3. 初始化字典可以指定字典容量,这是否多余呢?
4. 字典的桶buckets 长度为素数,为什么呢?

除了问题4,可能大家都有过,那么我们带着这些疑问,走进这篇文章.

## 从哈希函数说起
什么是哈希函数?  
哈希函数又称散列函数,是一种从任何一种数据中创建小的数字“指纹”的方法。  
下面,我们看看JDK中Sting.GetHashCode()方法.
```java
public int hashCode() {
        int h = hash;
 //hash default value : 0 
        if (h == 0 && value.length > 0) {
 //value : char storage
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }

```
可以看到,无论多长的字符串,最终都会返回一个int值,当哈希函数确定的情况下,任何一个字符串的哈希值都是唯一且确定的.  
当然,这里只是找了一种最简单的字符数哈希值求法,理论上只要能把一个对象转换成唯一且确定值的函数,我们都可以把它称之为哈希函数.  
这是哈希函数的工作原理图.  
![哈希函数工作原理图](https://raw.githubusercontent.com/liuzhenyulive/DictionaryMini/master/Pic/HashFunction.svg?sanitize=true)  

## 字典
通过上面解释的什么是哈希函数,我们应该可以得出一个这样的结论:  
`一个对象的哈希值是确定且唯一的,并求出一个对象的哈希值远比在大量数据中遍历寻找我们要的数据来得简单得多.`  
那么,如何把哈希值和在集合中我们要的数据的地址关联起来呢?
解开这个疑惑前我来看看一个这样不怎么恰当的例子:  
有一天,我不小心干了什么坏事,警察叔叔没有逮到我本人,但是他知道是一个叫`阿宇`的干的,他要找我肯定先去我家,他怎么知道我家的地址呢?他不可能在全中国的家庭一个个去遍历,敲门,问`阿宇`是你们家的熊孩子吗?  
正常应该是通过我的名字,找到我的身份证号码,然后我的身份证上登记着我的家庭地址,这样来找我.  
  
`阿宇`-----> 身份证(身份证号码,家庭住址)------>我家  
  
我们就可以把由阿宇找到身份证号码的过程,理解为`哈希函数`,身份证存储着我的号码的同时,也存储着我家的地址,身份证这个角色在字典中就是 `bucket`,bucket存储着数据的内存地址(索引).  
key--->bucket的过程     ~=     `阿宇`----->身份证  的过程.  
![Alt text](https://raw.githubusercontent.com/liuzhenyulive/DictionaryMini/master/Pic/hashtable0.svg?sanitize=true)  

警察叔叔通过家庭住址找到了我家之后,我家除了住我,还住着我爸,我妈,警察叔叔敲门的时候,是我爸开门,于是问我爸爸,`阿宇`在哪,我爸不知道,于是我爸问我妈,`阿宇`在哪?我妈告诉警察叔叔,在书房呢.很好,警察叔叔就这样把我给逮住了.  
字典也是这样,同一个bucket可能有多个key对应,即下图中的Johon Smith和Sandra Dee,但是bucket只能记录一个内存地址(索引),也就是警察叔叔通过家庭地址找到我家时,正常来说,只有一个人过来开门,那么,如何找到也在这个家里的我的呢?我爸记录这我妈在厨房,我妈记录着我在书房,就这样,我就被揪出来了,我爸,我妈,我 就是字典中的一个entry.  

![Alt text](https://raw.githubusercontent.com/liuzhenyulive/DictionaryMini/master/Pic/hashtable1.svg?sanitize=true)  
如果有一天,我妈妈老来得子又生了一个小宝宝,怎么办呢?很简单,我妈记录小宝宝的位置,那么我的只能巴结小宝宝,让小宝宝来记录我的位置了.  
![Alt text](https://raw.githubusercontent.com/liuzhenyulive/DictionaryMini/master/Pic/hashtable2.svg?sanitize=true)  
![Alt text](https://raw.githubusercontent.com/liuzhenyulive/DictionaryMini/master/Pic/hashtable3.svg?sanitize=true)  

既然大的原理明白了,是不是要看看源码,来研究研究代码中字典怎么实现的呢?

# DictionaryMini

上次在苏州参加苏州微软技术俱乐部成立大会时,有幸参加了`蒋金楠` 老师讲的Asp .net core框架解密,蒋老师有句话让我印象很深刻,"学好一门技术的最好的方法,就是模仿它的样子,自己造一个出来"于是他弄了个Asp .net core mini,所以我效仿蒋老师,弄了个DictionaryMini  

其源代码我放在了Github仓库,有兴趣的可以看看:https://github.com/liuzhenyulive/DictionaryMini

我觉得字典有意思的主要在这几个方面:
1. 字典的初始化.
2. 添加新元素.
3. 通过Key取值.
4. 移除指定元素

字典中还有其他功能,但我相信,只要弄明白的这几个方面的工作原理,我们也就恰中肯綮,他么问题也就迎刃而解了.

### 字典初始化

