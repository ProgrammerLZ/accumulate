> 原文地址：https://segmentfault.com/a/1190000020339810

MySQL在创建数据库是，需要设置数据库的字符集和排序规则，如图所示：

![图1](MySQL%E5%AD%97%E7%AC%A6%E5%92%8C%E5%AD%97%E7%AC%A6%E9%9B%86/bVbxvtx.png)

我觉得这里有必要解释下字符集和排序规则这两个概念。

## 字符集

说到字符集，需要先提下**字符**、**字符集**和**字符编码**这几个词的含义。

- **字符(Character)**是各种文字和符号的总称，包括各国家文字、标点符号、图形符号、数字等。
- **字符集(Character set)**是多个字符的集合，字符集种类较多，每个字符集包含的字符个数不同，常见字符集名称：ASCII字符集、GB2312字符集、BIG5字符集、 GB18030字符集、Unicode字符集等。
- **字符编码**是把字符集中的字符编码为特定的二进制数，以便在计算机中存储。编码方式一般就是对二维表的横纵坐标进行变换的算法。一般都比较简单，直接把横纵坐标拼一起就完事了。后来随着字符集的不断扩大，为了节省存储空间，才出现了各种各样的算法。

字符集和字符编码一般都是成对出现的，如ASCII、IOS-8859-1、GB2312、GBK，都是即表示了字符集又表示了对应的字符编码，以后统称为编码。Unicode比较特殊，后面细说。

在MySQL中需要注意的utf8和utf8mb4这两种字符集的区别，utf-8编码格式我们经常会碰到，但是这里的utf8却不是指utf-8这种编码格式，那么又为啥会出现utf8mb4这种字符集呢？

据说MySQL一开始没有utf8mb4这个字符集，因为utf8只支持每个字符最多三个字节，而真正的UTF-8是每个字符最多四个字节，这就造成UTF-8编码下的一些字符无法保存到数据库中，为了修复这个bug而出现了utf8mb4这种字符集。

> 三个字节的UTF-8最大能编码的Unicode字符是0xFFFF，也就是Unicode中的基本多文平面（BMP）。也就是说，任何不在基本多文平面的Unicode字符，都无法使用MySQL原有的utf8字符集存储。这些不在BMP中的字符包括哪些呢？最常见的就是Emoji表情（Emoji是一种特殊的Unicode编码，常见于ios和android手机上），和一些不常用的汉字，以及任何新增的Unicode字符等等。
>
> 转载处:[https://www.jianshu.com/p/f90...](https://www.jianshu.com/p/f9073c8c85b9)

如果要在MySQL中保存4字节长度的UTF-8字符，就需要使用utf8mb4编码，但是要注意只有5.5.3版本以后的MySQL才支持（查看版本命令： select version()）。为了获取更好的兼容性，建议使用utf8mb4而非utf8. 对于CHAR类型数据，utf8mb4会多消耗一些空间，但根据 MySQL官方建议，可以使用VARCHAR替代CHAR。

扩展：char是一种固定长度的类型，varchar则是一种可变长度的类型（因为char长度固定，方便程序的存储与查找，所以char类型存取速度优于varchar，即以空间换效率）

## 排序规则

MySQL中常用的排序规则(这里以utf8字符集为例)主要有：utf8_general_ci、utf8_general_cs、utf8_unicode_ci等。

这里需要注意下*ci*和*cs*的区别:

- ci的完整英文是'Case Insensitive', 即“大小写不敏感”，a和A会在字符判断中会被当做一样的;
- cs的完整英文是‘Case Sensitive’，即“大小写敏感”，a 和 A 会有区分；

比如下面这个查询：

```
# 假设数据库中SC_Teacher表存在一条数据，其中TeacherName字段的值为 "A"

select * from SC_Teacher where TeacherName = 'a' 
-- 如果数据库使用的是utf8_general_ci排序规则, 下面的查询是可以查询到这条数据
-- 如果数据库使用的是utf8_general_cs排序规则, 下面的查询是查询不到这条数据
```

正因为这个性质，导致utf8_general_ci的查询速度比utf8_general_cs快，(纯属个人推测，没有实际依据)

- **utf8_general_ci**： 查询时不区分大小写匹配
- **utf8_general_cs**： 查询时区分大小写匹配
- **utf8_bin**： 字符串每个字符串用二进制数据编译存储。 区分大小写，而且可以存二进制的内容，与utf8_general_cs一样，区分大小写
- **utf8_unicode_ci** ： 和utf8_general_ci一样，不区分大小写

> 当前utf8_general_ci校对规则仅部分支持Unicode校对规则算法。一些字符还是不能支持。并且，不能完全支持组合的记号。这主要影响越南和俄罗斯的一些少数民族语言，如：Udmurt、Tatar、Bashkir和Mari。 
>
> utf8_general_ci的最主要的特色是支持扩展，即当把一个字母看作与其它字母组合相等时。例如，在德语和一些其它语言中‘ß’等于‘ss’。
>
> utf8_general_ci是一个遗留的校对规则，不支持扩展。它仅能够在字符之间进行逐个比较。这意味着utf8_general_ci校对规则进行的比较速度很快，但是与使用utf8_unicode_ci的校对规则相比，比较正确性较差）。 
>
> 例如，使用utf8_general_ci和utf8_unicode_ci两种 校对规则下面的比较相等：
>
> ```
> Ä = A
> Ö = O
> Ü = U
> ```
>
> 两种校对规则之间的区别是，对于utf8_general_ci下面的等式成立：
>
> ```
> ß = s
> ```
>
> 但是，对于utf8_unicode_ci下面等式成立：
>
> ```
> ß = ss
> ```
>
> 对于一种语言仅当使用utf8_unicode_ci排序做的不好时，才执行与具体语言相关的utf8字符集 校对规则。例如，对于德语和法语，utf8_unicode_ci工作的很好，因此不再需要为这两种语言创建特殊的utf8校对规则。
>
> utf8_general_ci也适用与德语和法语，除了‘ß’等于‘s’，而不是‘ss’之外。如果你的应用能够接受这些，那么应该使用utf8_general_ci，因为它速度快。否则，使用utf8_unicode_ci，因为它比较准确。 
>
> 转载处:[http://www.chinaz.com/program...](http://www.chinaz.com/program/2010/0225/107151.shtml)

### 简短总结

utf8_unicode_ci和utf8_general_ci对中、英文来说没有实质的差别。
utf8_general_ci校对速度快，但准确度稍差。
utf8_unicode_ci准确度高，但校对速度稍慢。

如果你的应用有德语、法语或者俄语，请一定使用utf8_unicode_ci。一般用utf8_general_ci就够了，到现在也没发现问题。