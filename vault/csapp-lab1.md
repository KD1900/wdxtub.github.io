title: 深入理解计算机系统 习题课 1 Datalab
categories:
- Technique
date: 2016-01-18 08:01:12
toc: true
tags:
- CMU
- 计算机
- 习题课
- Datalab
---

这一讲主要是介绍第一次作业 Datalab 的相关内容以及解法，还包括如何在远程机器上编写代码和测试。

<!-- more -->

---

我们先来看看 Datalab 需要我们做什么。主要是通过这次的作业来熟悉整型及浮点数的位表达形式，简单来说，就是解开一些人工谜题。列表如下：

![csapp28](/images/csapp28.jpg)

![csapp29](/images/csapp29.jpg)

![csapp30](/images/csapp30.jpg)

一共 13 个需要补充的函数（会在后面分别详细说明），要开始做 lab，有以下四个步骤（针对 CMU 学生）

1. 从 Autolab 下载 lab 文件和文档
2. lab 的代码需要在远程机器上执行，需要解压文件（可能还需要修改权限 `chmod xxx filename`）
3. 修改 `bits.c` 文件，测试的话有三种方式：`btest`, `dlc`, 和 `BDD checker`，另外 `driver.pl` 是 autolab 将要进行的测试
4. 有无限次提交，但是不要用 autolab 来当做测试的方式，用 `driver.pl` 测试，Autolab 需要在网页端提交（使用 `tar -cvzf out.tar.gz file1 file2 ...` 这样来压缩，可以 tcp 赋复制到自己的电脑中）

一些小技巧：

+ 在函数开始时声明所有变量
+ `}` 应该在第一列
+ 注意运算符号的优先级，使用括号确保顺序的正确
+ 关注 !, 0, TMin 等

## 准备工作

我们先登录到远程机器，命令为 `ssh -X dawang@shark.ics.cs.cmu.edu`，输入密码后可以看到如下界面

![csapp31](/images/csapp31.jpg)

然后另开一个终端，把之前下载好的 `datalab-handout.tar` 传到服务器上，命令是 `scp datalab-handout.tar dawang@shark.ics.cs.cmu.edu:`

![csapp32](/images/csapp32.jpg)

可以看到上面有一些我之前课程的内容，我们创建一个新的文件夹 `513` 并把这个压缩包移动进去

```bash
mkdir 513
mv datalab-handout.tar ./513
clear # 让屏幕向后翻一页，看得更清楚一些
```

这里稍微啰嗦一点，如果想要一边改动文件，然后一边编译测试的话，可以开启两个终端连接到远程机器上，这样就不用切换来切换去了。

解压这次试验所需的文件： `tar xvf datalab-handout.tar`

![csapp33](/images/csapp33.jpg)

这次的作业主要是在 `bits.c` 文件里完成，我们来看看里面的内容 `vim bits.c`

![csapp34](/images/csapp34.jpg)

任务指引还是比较清晰的，主要有以下一些说明：

1. 整型的范围是 0 到 255(0xFF)，不允许用更大
2. 只能包含参数和局部变量
3. 一元操作符 `!` `~`
4. 二元操作符 `&` `|` `+` `<<` `>>`

不允许的操作有：

1. 使用任何条件控制语句
2. 定义和使用宏
3. 定义其他的函数
4. 调用函数
5. 使用其他的操作符
6. 使用类型转换
7. 使用除 int 之外的类型（针对整型）
8. 使用除 int, unsigned 之外的类型（针对浮点数）

可以认为机器：

+ 使用 2's complent，32位
+ 执行算术右移
+ 移动超过字长的位数会出问题

其他需要注意的事情有：

1. 使用 dlc(data lab checker) 来检测代码的合法性（有没有使用不给使用的符号语法等等）
2. 每个函数都有操作数的上限值，注意 `=` 不算
3. 使用 btest 来测试结果的正确与否
4. 使用 BDD checker 来正规测试你的函数

## 题目及解法

### thirdBits

+ 题目要求：return word with every third bit (starting from the LSB) set to 1
+ 允许操作：`! ~ & ^ | + << >>`
+ 操作数限制：8
+ 分值：1

我们返回的结果是：`0100 1001 0010 0100 1001 0010 0100 1001`，因为题目要求每个变量不可以超过 255，也就是最多 `1111 1111`，所以只能分步骤来进行组合，如下面代码所示

```
Desired output: 0100 1001 0010 0100 1001 0010 0100 1001 
Step 1:         0000 0000 0000 0000 0000 0000 0100 1001  0x49
Step 2:         0000 0000 0000 0000 1001 0010 0000 0000  Shift << 9
Step 3:         0000 0000 0000 0000 1001 0010 0100 1001  Add 0x49
Step 4:         0100 1001 0010 0100 0000 0000 0000 0000  Shift << 18
Step 5:         0100 1001 0010 0100 1001 0010 0100 1001  Add result from step 3

int thirdBits(void) {
  int a = 0x49;
  int b = (a << 9);
  int c = b + a;
  return (c << 18) + c; // Steps 4 and 5
}
```

然后我们来测试一下正确性，先保存代码，然后：`make; ./btest`，结果如下：

![csapp35](/images/csapp35.jpg)

可以看到第一个函数已经写对的得到了一分，然后我们再来检测一下有没有用非法的操作符：`./dlc -e bits.c`

![csapp36](/images/csapp36.jpg)

可以看到没有显示错误信息，`-e` 会输出操作符的数量，这里也都没有问题。接下来的题目都会用这种方式测试，但是就不会再贴图了。

### isTmin

+ 题目要求：returns 1 if x is the minimum, two's complement number, and 0 otherwise
+ 允许操作：`! ~ & ^ | +`
+ 操作数限制：10
+ 分值：1

根据 2's complement 的定义，Tmin 的值是 `10000000 00000000 00000000 00000000`，我们要怎么判断一个数是不是 Tmin 呢，原则上来说，只需要把 x 和 Tmin 做 `&` 操作，判断即可，但是题目要求不能左移，于是就要想其他的办法了，观察 Tmin 的值，发现如果左移一次，就变成全部为 0，但是全部为零的情况还有另外一种就是本身全部就是 0，所以只要排除第二种情况，就可以判断是否是 Tmin 了，代码如下：

```c
// 前面一部分用于判断左移一位后是否全部为0，后面一部分用来判断 x 值是否为 0
int isTmin(int x) {
   return !(x+x)&!!(x);
}
```

### isNotEqual

+ 题目要求：return 0 if x == y, and 1 otherwise 
	+	例如: isNotEqual(5,5) = 0, isNotEqual(4,5) = 1
+ 允许操作：`! ~ & ^ | + << >>`
+ 操作数限制：6
+ 分值：2

这题比较简单，发现可以使用异或操作，那么只需要判断两个数异或后结果是否为 0 即可，这里同样使用了 !! 来把 bit 转换成 boolean 值

```c
int isNotEqual(int x, int y)
{
	return(!!(x ^ y));
}
```

### anyOddBit

+ 题目要求：return 1 if any odd-numbered bit in word set to 1
	+ 例如： anyOddBit(0x5) = 0, anyOddBit(0x7) = 1
+ 允许操作：`! ~ & ^ | + << >>`
+ 操作数限制：12
+ 分值：2

我们依旧不能超过 0xFF 的限制，所以需要把前面的 24 位都用 `|` 和 `>>` 运算符移动到最后八位中，再和 `10101010` 来做 `&` 操作，只要不为零，就说明在偶数位上有不为 0 位

```c
int anyOddBit(int x) {
    return !!((x | (x >> 8) | (x >> 16) | (x >> 24)) & 0xaa);
}
```

### negate

+ 题目要求：return -x
	+ 例如：negate(1) = -1. 
+ 允许操作：`! ~ & ^ | + << >>`
+ 操作数限制：5
+ 分值：2

第一感觉就是用到取反 `~` 符号，但需要考虑三种情况：正，零，负

+ 假设是 `0010`(2)，取反之后是 `1101`(-3)
+ 假设是 `1110`(-2)，取反之后是 `0001`(1)
+ 假设是 `0000`(0)，取反之后是 `1111`(-1) 

可以发现一个规律，就是都差 1，为什么呢，就是因为 2's complement 的定义中是加上了 1 的，所以只要再加一就好。

```c
int negate(int x) {  
  return ~x + 1;  
}
```

### conditional

+ 题目要求：same as x ? y : z
	+ 例如：conditional(2,4,5) = 4 
+ 允许操作：`! ~ & ^ | + << >>`
+ 操作数限制：16
+ 分值：3

这一题稍微有一些复杂，我们来看看怎么去想。因为不能用 if 来做流程判断，所以我们返回的表达式例一定要包含 y 和 z，但是可以根据 x 的值来进行变换，所以大概的式子是 `(y op expr) | (z op expr)`(op 表示操作符， expr 是某个表达式)。

然后就简单很多了，我们只要想办法做一个 expr，要么为 `0x00000000`，要么为 `0xffffffff` 即可，于是表达式 `~!x + 1` 就可以满足我们的需求，x 为 0 时，表达式为 `0xffffffff`，不等于 0 时也满足条件，就等于有了答案

```c
int conditional(int x, int y, int z) {
    /*
     *if x!=0,mask=0x00000000,so y&~mask==y and z&mask==0
     *if x==0,mask=0xffffffff,so y&~mask = y&0 =0; z&mask=z
     */
    int mask= ~!x+1; 
    return (y & ~mask)|(z & mask);
}
```

### subOK

+ 题目要求：Determine if can compute x-y without overflow
	+ 例如：
	+ subOK(0x80000000,0x80000000) = 1
	+ subOK(0x80000000,0x70000000) = 0
+ 允许操作：`! ~ & ^ | + << >>`
+ 操作数限制：20
+ 分值：3

这题也不算轻松，但是我们可以一步一步来搞定，首先，既然是计算 x-y，我们要能够知道结果，由于不给使用减号，那就用倒数（之前的方法），所以 x-y 的结果为 `~y+1+x`。然后需要怎么判断呢，观察发现，只有在以下这两种情况同时发生的时候，才是 overflow

1. x 和 y 符号不同
2. x-y 的符号和 x 不同

可能有点难以理解，overflow 指的是除符号位的最高位进位，也就是说符号会变化，所以需要 x 和 y 的符号不同（这样 x-y 就等同于两个同符号的加法），也就是第一个条件；符号到底有没有变化呢，就要看 x-y 与 x 的符号是否相同，也就是第二个条件。所以代码如下：

```c
int subOK(int x, int y) {
  /*
   * overflow of sub happens iff 
   * 1) x and y have different signs
   * 2) res = x - y has different sign with x
   */
  int res = x + (~y + 1);
  int sameSign = x ^ y;
  int resSign = res ^ x;

  return !((sameSign & resSign) >> 31);
}
```

### isGreater

+ 题目要求：if x > y  then return 1, else return 0
	+ 例如：isGreater(4,5) = 0, isGreater(5,4) = 1
+ 允许操作：`! ~ & ^ | + << >>`
+ 操作数限制：24
+ 分值：3

因为要考虑正负号，所以这个问题变成：

1. 两个数符号相同的情况
2. 两个数符号不同的情况

具体可以参见代码

```c
int isGreater(int x, int y)
{
        // Boolean value indicating sign of x
        // 1 = Negative
        // 0 = Non-Negative
        int sign_x = x >> 31;

        // Boolean value indicating sign of y
        // 1 = Negative
        // 0 = Non-Negative
        int sign_y = y >> 31;

	// if the signs are equal, then
	// if x is larger, sign bit of (~y + x) is 0
	// if y is larger, sign bit of (~y + x) is 1
	int equal = !(sign_x ^ sign_y) & ((~y + x) >> 31);

	// if signs are not equal, these principles are reversed.
	int notEqual = sign_x & !sign_y;

	// this | returns 0 when it is x is greater, so you have to negate it.
	return !( equal | notEqual);
}
```

### bitParity

+ 题目要求：returns 1 if x contains an odd number of 0's
	+ 例如：bitParity(5) = 0, bitParity(7) = 1
+ 允许操作：`! ~ & ^ | + << >>`
+ 操作数限制：20
+ 分值：4

这道题要我们统计有有多少个零，这里我们需要利用一个特点，就是堆两个数进行异或操作，不改变奇偶性，所以只需要一步一步来异或就可以了

```c
int bitParity(int x) {
  /* XORing two numbers returns a number with same bit parity.
     Keep shifting half of our number until reduced to 1 bit simple case.*/

  x = ( x >> 16 ) ^ x;
  x = ( x >> 8 ) ^ x;
  x = ( x >> 4 ) ^ x;
  x = ( x >> 2 ) ^ x;
  x = ( x >> 1) ^ x;
  return (x & 1);
}
```


### howManyBits

+ 题目要求：return the minimum number of bits required to represent x in two's complement
	+ 例如：
	+ howManyBits(12) = 5
	+ howManyBits(298) = 10
	+ howManyBits(-5) = 4
	+ howManyBits(0)  = 1
	+ howManyBits(-1) = 1
	+ howManyBits(0x80000000) = 32
+ 允许操作：`! ~ & ^ | + << >>`
+ 操作数限制：90
+ 分值：4

这题从操作数限制的数目来看就知道比较复杂，但是代码还是比较清晰的，可以直接查看代码中的注释（特别鸣谢：@guojiex）

```c
int howManyBits(int x) {
    int temp=x^(x>>31);//get positive of x;
    int isZero=!temp;
    //notZeroMask is 0xffffffff
    int notZeroMask=(!(!temp)<<31)>>31;
    int bit_16,bit_8,bit_4,bit_2,bit_1;
    bit_16=!(!(temp>>16))<<4;
    //see if the high 16bits have value,if have,then we need at least 16 bits
    //if the highest 16 bits have value,then rightshift 16 to see the exact place of  
    //if not means they are all zero,right shift nothing and we should only consider the low 16 bits
    temp=temp>>bit_16;
    bit_8=!(!(temp>>8))<<3;
    temp=temp>>bit_8;
    bit_4=!(!(temp>>4))<<2;
    temp=temp>>bit_4;
    bit_2=!(!(temp>>2))<<1;
    temp=temp>>bit_2;
    bit_1=!(!(temp>>1));
    temp=bit_16+bit_8+bit_4+bit_2+bit_1+2;//at least we need one bit for 1 to tmax,
    //and we need another bit for sign
    return isZero|(temp&notZeroMask);
}
```

### float_half

+ 题目要求：Return bit-level equivalent of expression (float) x. Result is returned as unsigned int, but it is to be interpreted as the bit-level representation of a single-precision floating point values.
+ 允许操作：Any integer/unsigned operations incl. ||, &&. also if, while
+ 操作数限制：30
+ 分值：4

这个就是考察基本的对于 IEEE 浮点数格式的转换了，思路也比较清晰，就是根据不同的部分来求出对应的值

```c
unsigned float_half(unsigned uf) {
  int round, S, E, maskE, maskM, maskS,maskEM, maskSM, tmp;
  round = !((uf&3)^3);
  maskS = 0x80000000;
  maskE = 0x7F800000;
  maskM = 0x007FFFFF;
  maskEM= 0x7FFFFFFF;
  maskSM= 0x807FFFFF;
  E = uf&maskE;
  S = uf&maskS;
  //Nan or Infinity
  if (E==0x7F800000) return uf;
  //E=1 - specialcase
  if (E==0x00800000){
    return S | (round + ((uf & maskEM)>>1)) ;
  }
  //E=0 - denormalized
  if (E==0x00000000) {
    tmp = (uf&maskM)>>1;
    return S | (tmp + round);
  }
  //normalized case
  return (((E>>23)-1)<<23) | (uf & maskSM);
}
```

### float_i2f

+ 题目要求：Return bit-level equivalent of expression (float) x. Result is returned as unsigned int, but it is to be interpreted as the bit-level representation of a single-precision floating point values.
+ 允许操作：Any integer/unsigned operations incl. ||, &&. also if, while
+ 操作数限制：30
+ 分值：4

和上题一样，这个就是考察基本的对于 IEEE 浮点数格式的转换了，思路也比较清晰，就是根据不同的部分来求出对应的值（特别鸣谢@guojiex）

```c
unsigned float_i2f(int x) {
    /*int exponent=0;
    return ((sign<<31)|(exponent<<23)|fraction)+flag;*/
    int sign=x>>31&1;
    int i;
    int exponent; 
    int fraction; 
    int delta;
    int fraction_mask;
    if(x==0)//||x==0x8000000)
        return x;
    else if(x==0x80000000)
        exponent=158;
    else{
        if (sign)//turn negtive to positive
            x = -x;
        i = 30;
        while ( !(x >> i) )//see how many bits do x have(rightshift until 0) 
            i--;
        //printf("%x %d\n",x,i);
        exponent = i + 127;
        x = x << (31 - i);//clean all those zeroes of high bits
        fraction_mask = 0x7fffff;//(1 << 23) - 1;
        fraction = fraction_mask & (x >> 8);//right shift 8 bits to become the fraction,sign and exp have 8 bits total
        x = x & 0xff;
        delta = x > 128 || ((x == 128) && (fraction & 1));//if lowest 8 bits of x is larger than a half,or is 1.5,round up 1
        fraction += delta;
        if(fraction >> 23) {//if after rounding fraction is larger than 23bits
            fraction &= fraction_mask;
            exponent += 1;
        }
    }
    return (sign<<31)|(exponent<<23)|fraction;
}
```

### float_f2i

+ 题目要求：Return bit-level equivalent of expression (int) f for floating point argument f. Argument is passed as unsigned int, but it is to be interpreted as the bit-level representation of a single-precision floating point value. Anything out of range (including NaN and infinity) should return 0x80000000u.
+ 允许操作：Any integer/unsigned operations incl. ||, &&. also if, while
+ 操作数限制：30
+ 分值：4

和上题一样，这个就是考察基本的对于 IEEE 浮点数格式的转换了，思路也比较清晰，就是根据不同的部分来求出对应的值

```c
int float_f2i(unsigned uf) {
  int exp = (uf >> 23) & 0xFF;
  int frac = uf & 0x007FFFFF;
  int sign = uf & 0x80000000;
  int bias = exp - 127;

  if (exp == 255 || bias > 30) {
    // exponent is 255 (NaN), or number is too large for an int
    return 0x80000000u;
  } else if (!exp || bias < 0) {
    // number is very small, round down to 0
    return 0;
  }

  // append a 1 to the front to normalize
  frac = frac | 1 << 23;

  // float based on the bias
  if (bias > 23) {
    frac = frac << (bias - 23);
  } else {
    frac = frac >> (23 - bias);
  }

  if (sign) {
    // original number was negative, make the new number negative
    frac = ~(frac) + 1;
  }

  return frac;
}
```

## 检查与提交

全部题目做完之后，就可以检查一下了，这里我们可以用另外一个命令 `./driver.pl`，结果如下：

![csapp37](/images/csapp37.jpg)

这样就算完成啦，提交的时候只需要提交 `bits.c` 所以我们先把它从远程主机复制回来 `scp dawang@shark.ics.cs.cmu.edu:513/datalab-handout/bits.c ./`

不用压缩，直接提交即可，检查无误之后，至此本次实验完成。

