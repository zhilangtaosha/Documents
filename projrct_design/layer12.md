# 大整数群(BigInt)
BigInt大整数是密码学运算中的计算单位基础，同样也是我们构建的密码库的第一层，基于C++的函数重载等特性，可以为单个运算符定义多种传参方式。这里我们将整个大整数群逻辑上分成两部分，BigInt(大整数)和NumThrory(数论)。
## BigInt
我们将与大整数构建、简单运算和其数据结构的方法封装到同一个类(class)中。
### class BigInt
#### BigInt()
构造函数，创建一个BigInt类型的大整数，其数值为零。
#### BigInt(int n)
构造函数，创建一个BigInt类型的大整数，其数值为n.
#### BigInt(string &str)
构造函数，创建一个BigInt类型的大整数，将其赋值为传入的字符串相对应的整型变量，默认情况为十进制，当传入的字符串以'0x'开头则解析为16进制。
#### BigInt(RNG &rng, size_t bits)
构造函数，创建一个BigInt类型的大整数，在给定范围的条件下。

#### 运算操作符(operator)
| +, -, *, / |    << , >>     |  ++, --  | +=, -=, %=, >>= |
| :--------: | :------------: | :------: | :-------------: |
|  加减乘除  | 大整数移位运算 | 原子加减 |   复合操作符    |
每一个运算符都会封装一个对应的运算函数，在函数体内完成大整数的对应运算，函数的返回类型都为BigInt。

#### 判断类型的函数(bool)
##### is_even(): Rerurn true if *this is even.  
##### is_odd(): Return true if *this is odd.  
##### is_nonzero(): Return true if *this is not zero.  
##### is_zero(): Return true if *this is zero.  
##### is_negative(): Return true if *this is is negative.  
##### is_positive(): Return true if *this is positive.  
##### abs(): Return absolute value of *this.  
上述函数都封装在BigInt类中，通过实例化的对象调用，返回的结果对应该对象的某一特征。

#### bit操作类型的函数
同样对于bit运算来说，对应的函数也封装在BigInt类中，通过调用函数，对实例化的对象进行相应的bit操作。
##### set_bit(): Set bit n of *this.
##### clear_bit(): Clear bit n of *this.
##### get_bit(): Get bit n of *this.

#### 编码解码，进制转换
##### binary_encode(uint8_t buf[]): Encode this BigInt as a big-endian integer.
##### binary_decode(uint8_t buf[]): Decode this BigInt as a big-endian integer. 
##### to_dec_string(uint8_t buf[]): Encode the integer as a decimal string.
##### to_hex_string(uint8_t buf[]): Encode the integet as a hexadecimal string.

## 数论(Number Theory)
在大整数群层面，通过封装好大整数的基本类BigInt，我们还需对其进行数论领域的基本运算，数论相关的函数和class BigInt在整个系统架构层面可以看作同一级，且都在同一个命名域(namespace)空间下。
### BigInt gcd(x, y): 计算大整数x和y的最大公约数
### BigInt lcm(x, y): 计算大整数x和y的最小公倍数(z%x==0||z%y==0)
### BigInt jacobi(a, n): 计算Jacobi符号(a|n)
### BigInt inverse_mod(x, m): 计算乘法逆元，返回y(x*y)%m==1
### BigInt power_mod(b, x, m): 指数模运算，x为底，b为指数，m是模
### BigInt ressol(x, p): 计算平方根，返回y(y*y)%p == x，当结果不存在时返回-1

## Header(.h) files & Requires [to do list]
### public
bigint.h
divide.h
curve_nistp.h
numthry.h
pow_mod.h
reducer.h
monty.h
### internal
primality.h
monty_exp.h
### Requires
mp
hex
rng

# 有限群层(密码学算法的数学基础--Group)
密码学与有限循环群。
现代密码学算法和协议中，消息是作为有限空间中的数字或元素来处理的。加密和解密的各种操作必须在消息之间进行变换，以使变换服从有限消息空间内部的封闭性。然而，数的一般运算诸如加减乘除并不满足有限空间内部的封闭性。所以密码算法通常运行于具有某些保持封闭性的代数结构的空间中，这种代数结构就是有限循环群。在数学中，群是一种代数结构，由一个集合以及一个二元运算组成。群必须满足以下四个条件：封闭性，结合律，存在单位元和存在逆元。
这里我们选择两个密码学领域中典型和最常用的有限群进行代码封装。
## DL group
离散对数群，将该群封装为一个class，其底层需要我们在Layer1封装好的BigInt和数论相关的代码。整个类的伪代码如下所示:  
#include "bigint.h"  
#include "numthry.h"  
/**  
This class represents discrete logarithm groups. It holds a prime modulus p, a generator g, and (optionally) a prime q which is a factor of (p-1). In most cases g generates the order-q subgroup.  
*/  
class DL_Group  
{  
&emsp;&emsp;// 构造函数，Create a new group with kinds of paramenters.  
&emsp;&emsp;DL_Group(some paras[])  
&emsp;&emsp;// 通过layer1的BigInt类中的方法进行群的相应运算.  
&emsp;&emsp;BigInt mod_p(const BigInt& x);  
&emsp;&emsp;BigInt multiply_mod_p(BigInt& x, BigInt& y);  
&emsp;&emsp;BigInt power_g_p(BigInt& x);  
&emsp;&emsp;BigInt ... ;  

}  

## EC group
椭圆曲线群，同样将其封装为一个class，底层需要BigInt和数论相应的代码。
#include "point_gfp.h"  
#include "curve_gfp.h"  
#include "bigint.h"  
class EC_Group  
{  
&emsp;&emsp;// 不同参数的构造函数  
&emsp;&emsp;EC_Group(some paras[])  
&emsp;&emsp;// 在类的内部定义一些点坐标的运算  
&emsp;&emsp;PointGFp point(BigInt& x,BigInt& y);  
&emsp;&emsp;PointGFp point_multiply(BigInt& x, PointGFp& pt, BigInt& y);  
&emsp;&emsp;// 数论相关的运算  
&emsp;&emsp;...;  
}  
与EC_Group同级的还有class EC_Group_Data和class EC_Group_Data_Map用来描述椭圆曲线群的属性信息。v

