
# DictionaryMini

## 为什么写这么一篇博客

### 从一道亲身经历的面试题说起

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

## DictionaryMini

上次在苏州参加苏州微软技术俱乐部成立大会时,有幸参加了`蒋金楠` 老师讲的Asp .net core框架解密,蒋老师有句话让我印象很深刻,"学好一门技术的最好的方法,就是模仿它的样子,自己造一个出来"于是他弄了个Asp .net core mini,所以我效仿蒋老师,弄了个DictionaryMini  

其源代码我放在了Github仓库,有兴趣的可以看看:https://github.com/liuzhenyulive/DictionaryMini

我觉得字典有意思的主要在这几个方面:

1. 数据存储的最小单元
2. 字典的初始化
3. 添加新元素
4. 字典的扩容
5. 移除元素

字典中还有其他功能,但我相信,只要弄明白的这几个方面的工作原理,我们也就恰中肯綮,他么问题也就迎刃而解了.  

### 数据存储的最小单元

``` c#
   private struct Entry
        {
            public int HashCode;
            public int Next;
            public TKey Key;
            public TValue Value;
        }
```

一个存储单元包括该key的HashCode,以及下个Entry的索引Next,该键值对的Key以及数据Vaule.

### 字典初始化

``` c#
        private void Initialize(int capacity)
        {
            int size = HashHelpersMini.GetPrime(capacity);
            _buckets = new int[size];
            for (int i = 0; i < _buckets.Length; i++)
            {
                _buckets[i] = -1;
            }

            _entries = new Entry[size];

            _freeList = -1;
        }
```
字典初始化时,首先要创建两个指定长度的int数组,分别作为buckets和entries,其中buckets的index是key的`哈希值%size`,它的value是数据在entries中的index,我们要取得数据就存在entries中.当某一个bucket没有指向任何entry时,它的value为-1.  
另外,很有意思得一点,的这个指定长度是多少呢?这个我研究了挺久,发现取得是大于capacity的最小质数.

### 添加新元素

``` c#
  private void Insert(TKey key, TValue value, bool add)
        {
            if (key == null)
            {
                throw new ArgumentNullException();
            }
            //如果buckets为空,则重新初始化字典.
            if (_buckets == null) Initialize(0);
            //获取传入key的 哈希值
            var hashCode = _comparer.GetHashCode(key);
            //把hashCode%size的值作为目标Bucket的Index.
            var targetBucket = hashCode % _buckets.Length;
            //遍历判断传入的key对应的值是否已经添加字典中
            for (int i = _buckets[targetBucket]; i > 0; i = _entries[i].Next)
            {
                if (_entries[i].HashCode == hashCode && _comparer.Equals(_entries[i].Key, key))
                {
                    //当add为true时,直接抛出异常,告诉给定的值已存在在字典中.
                    if (add)
                    {
                        throw new Exception("给定的关键字已存在!");
                    }
                    //当add为false时,重新赋值并退出.
                    _entries[i].Value = value;
                    return;
                }
            }
            //表示本次存储数据的数据在Entries中的索引
            int index;
            //当有数据被Remove时,freeCount会加1
            if (_freeCount > 0)
            {
                //freeList为上一个移除数据的Entries的索引,这样能尽量地让连续的Entries都利用起来.
                index = _freeList;
                _freeList = _entries[index].Next;
                _freeCount--;
            }
            else
            {
                //当已使用的Entry的数据等于Entries的长度时,说明字典里的数据已经存满了,需要对字典进行扩容,Resize.
                if (_count == _entries.Length)
                {
                    Resize();
                    targetBucket = hashCode % _buckets.Length;
                }
                //默认取未使用的第一个
                index = _count;
                _count++;
            }
            //对Entries进行赋值
            _entries[index].HashCode = hashCode;
            _entries[index].Next = _buckets[targetBucket];
            _entries[index].Key = key;
            _entries[index].Value = value;
            //用buckets来登记数据在Entries中的索引.
            _buckets[targetBucket] = index;
        }
```

### 字典的扩容

``` c#
private void Resize()
        {
            //获取大于当前size的最小质数
            Resize(HashHelpersMini.GetPrime(_count), false);
        }
 private void Resize(int newSize, bool foreNewHashCodes)
        {
            var newBuckets = new int[newSize];
            //把所有buckets设置-1
            for (int i = 0; i < newBuckets.Length; i++) newBuckets[i] = -1;
            var newEntries = new Entry[newSize];
            //把旧的的Enties中的数据拷贝到新的Entires数组中.
            Array.Copy(_entries, 0, newEntries, 0, _count);
            if (foreNewHashCodes)
            {
                for (int i = 0; i < _count; i++)
                {
                    if (newEntries[i].HashCode != -1)
                    {
                        newEntries[i].HashCode = _comparer.GetHashCode(newEntries[i].Key);
                    }
                }
            }
            //重新对新的bucket赋值.
            for (int i = 0; i < _count; i++)
            {
                if (newEntries[i].HashCode > 0)
                {
                    int bucket = newEntries[i].HashCode % newSize;
                    newEntries[i].Next = newBuckets[bucket];
                    newBuckets[bucket] = i;
                }
            }

            _buckets = newBuckets;
            _entries = newEntries;
        }
```

### 移除元素

``` c#
        //通过key移除指定的item
        public bool Remove(TKey key)
        {
            if (key == null)
                throw new Exception();

            if (_buckets != null)
            {
                //获取该key的HashCode
                int hashCode = _comparer.GetHashCode(key);
                //获取bucket的索引
                int bucket = hashCode % _buckets.Length;
                int last = -1;
                for (int i = _buckets[bucket]; i >= 0; last = i, i = _entries[i].Next)
                {
                    if (_entries[i].HashCode == hashCode && _comparer.Equals(_entries[i].Key, key))
                    {
                        if (last < 0)
                        {
                            _buckets[bucket] = _entries[i].Next;
                        }
                        else
                        {
                            _entries[last].Next = _entries[i].Next;
                        }
                        //把要移除的元素置空.
                        _entries[i].HashCode = -1;
                        _entries[i].Next = _freeList;
                        _entries[i].Key = default(TKey);
                        _entries[i].Value = default(TValue);
                        //把该释放的索引记录在freeList中
                        _freeList = i;
                        //把空Entry的数量加1
                        _freeCount++;
                        return true;
                    }
                }
            }

            return false;
        }
```

我.Net中的Dictionary的源码进行了精简,做了一个DictionaryMini,有兴趣的可以到我的github查看相关代码.
https://github.com/liuzhenyulive/DictionaryMini

## 答疑时间

#### 字典为什么能无限地Add呢?

向Dictionary中添加元素时,会有一步进行判断字典是否满了,没有满的,会用Resize对字典进行扩容,所以字典不会向数组那样有固定的容量.

#### 从字典中取Item速度非常快,为什么呢?

Key-->HashCode-->HashCode%Size-->Bucket Index-->Bucket-->Entry Index-->Value  
整个过程都没有通过遍历来查找数据,一步到下一步的目的性时非常明确的,所以取数据的过程非常快.

#### 初始化字典可以指定字典容量,这是否多余呢?

前面说过,当向字典中插入数据时,如果字典已满,会自动地给字典Resize扩容.
扩容的标准时大于当前前容量的最小质数作为当前字典的容量,比如,当我们的字典最终存储的元素为15个时,会有这样的一个过程.
new Dictionary()------------>size:3
字典中有3个元素---->Resize--->size:7
字典中有7个元素---->Resize--->size:11
字典中有11个元素--->Resize--->size:23

可以看到一共进行了次Resize,如果我们预先知道最终字典要存储15个元素,那么我们可以用new Dictionary(15)来创建一个字典.  
new Dictionary(15)---------->size:23

就不需要进行Resize了,可以想象,每次Resize都是消耗一定的时间资源的,所以我们在创建字典时,如果知道字典的中要存储的字典的元素个数,在创建字典时,就传入capacity.

Tips:  
即使指定字典容量capacity,后期如果添加的元素超过这个数量,字典也是会自动扩容的.

#### 字典的桶buckets 长度为素数,为什么呢?

https://cs.stackexchange.com/questions/11029/why-is-it-best-to-use-a-prime-number-as-a-mod-in-a-hashing-function/64191#64191

我们假设有这样的一系列keys K={ 0, 1,..., 100 },假设某一个buckets的长度m=12,因为3是12的一个因子,当key时3的倍数时,那么targetBucket将会是3.

``` 
        Keys {0,12,24,36,...}
        TargetBucket将会是0.
        Keys {3,15,27,39,...}
        TargetBucket将会是3.
        Keys {6,18,30,42,...} 
        TargetBucket将会是6.
        Keys {9,21,33,45,...} 
        TargetBucket将会是9.
```
        //If K is uniformly distributed(i.e., every key in K is equally likely to occur), then the choice of m is not so critical.But, what happens if K is not uniformly distributed? Imagine that the keys that are most likely to occur are the multiples of 3. In this case, all of the buckets that are not multiples of 3 will be empty with high probability (which is really bad in terms of hash table performance).

如果Key的值是均匀分布的(K中的每一个Key中出现的可能性相同),那么Buckets的Length就没有那么重要了,但是如果Key不是均匀分布呢?  
想象一下,如果Key在3的倍数时出现的可能性特别大,其他的基本不出现,TargetBucket那些不是3的倍数的索引就基本不会存储什么数据了,这样就可能有2/3的Bucket空着了.  

这种情况其实很常见。 例如，有一种场景，您根据对象存储在内存中的位置来跟踪对象。如果你的计算机的字节大小是4，如果你的Buckets的长度时4,那么所有的内存地址都会时4的倍数,也就是说key都是4的倍数,那么所有的数据都会存储在TargetBucket=0(Key%4=0)的bucket中,而剩下的3/4的Buckets都是空的.  

        //Every key in K that shares a common factor with the number of buckets m will be hashed to a bucket that is a multiple of this factor.

        //Therefore, to minimize collisions, it is important to reduce the number of common factors between m and the elements of K. How can this be achieved? By choosing m to be a number that has very few factors: a prime number.
