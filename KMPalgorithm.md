# 1.字符串的前缀、后缀和部分匹配值

以'ababa'为例

·'a'的前缀和后缀都为空集，最长相等前后缀为0。

·'ab'的前缀为{a},后缀为{b},{a}∩{b}=NULL，最长相等后缀为0。

·'aba'的前缀为{a,ab},后缀为{a,ba},所以前缀∩后缀={a},最长相等前后缀为1。

主串为ababcabcacbaab,字串为abcac的匹配问题：

字串'abcac'的部分匹配值***(Partial Match,PM)***为00010,下面用PM表来进行字符串匹配：

主串 ababcabcacbab

字串 abc

第一趟匹配过程中，发现c与a不匹配，前面两个字符’ab'是匹配的，字符b对应的的PM值是0，因此按照以下公式算出字串需前后移动的位数：

```python
#移动位数=已匹配的字符数-对应的部分匹配值
Move=(j-1)-PM[j-1]
```

所以将字串向后移动2位，进行第二趟匹配

主串 ababcabcacbab

字串      abcac

整个匹配过程中，主串始终没有回退，故KMP算法可以在O(n+m)时间上完成串的模式匹配操作。若某趟发生失配时，对应的PM值为0，此时移动的位数最大，直接将字串移动到主串i位置；如果已匹配的字符存在最大相等前后缀，那么将字串向右滑动到前后缀对齐，显然这部分字符已不需要比较，再从主串i位置进行下一趟比较。

# 2.KMP算法原理

使用部分匹配值时，每当匹配失败，就去找它前一个元素的自动匹配值，这样有一些不方便，所以将PM表右移一位，得到next数组，这样那样那个元素匹配失败，直接看自己的next值即可。这样上式就改写为：

```python
Move=(j-1)-next[j]
```

有时候为了使公式更加简洁、计算假单，将next数组整体加一。

求next值的程序如下：

```c
void get_next(string T,int next[]){
    int i=1,j=0;
    next[1]=0;
    while(i<T.length){
        if(j==0||T.ch[i]==T.ch[j]){
            ++i;++j;
            next[i]=j;
        }
        else
            j=next[j];
    }
}
```

KMP算法与简单模式匹配算法的不同之处仅在于当匹配过程中产生失配时，指针i不变，指针j退回到next[j]的位置并重新比较，代码如下：

```c
int Index_KMP(string S,string T,int next[]){
    int i=1,j=1;
    while(i<=S.length&&j<=T.length){
        if(j==0||S.ch[i]==T.ch[j]){
            ++i;++j;
        }
        else
            j=next[j];
    }
    if(j>T.length)
        return i-T.length;
    else
        return 0;
}
```

# 3.KMP算法的优化

前面定义的next数组在某些情况下尚有缺陷，模式'aaaab'在和主串'aaabaaaaab'进行匹配时：当i=4、j=4时，s4与p4失配，如果用之前的next数组还需进行s4与p3进行比较，这样必然失配，**问题在于不应该出现pj=pnext[j]**。理由是：当pj!=sj，下次匹配必然是pnext[j]与sj比较，如果pj=pnext[j]，那么必然继续失配。

如果出现了，则需要再次递归，将next[j]修正为next[next[j]],直至两者不相等为止：

```c
void get_nextval(string T,int nextval[]){
    int i=1,j=0;
    nextval[1]=0;
    while(i<T.length){
        if(j==0||T.ch[i]==T.ch[j]){
            ++i;++j;
            if(T.ch[i]!=T.ch[j]) nextval[i]=j;
            else nextval[i]=nextval[j];
        }
        else
            j=nextval[j];
    }
}
```

