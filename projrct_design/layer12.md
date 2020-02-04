# 大整数群(BigInt)
BigInt大整数是密码学运算中的计算单位基础，同样也是我们构建的密码库的第一层，基于C++的函数重载等特性，可以为单个运算符定义多种传参方式。这里我们将整个大整数群逻辑上分成两部分，BigInt(大整数)和NumThrory(数论)。
## BigInt 依赖关系
依赖关系主要描述的是cpp文件包含.h文件的详细信息，以及对应.h头文件的具体内容。   
**.h头文件内容**  
按照层级类划分，即上一层调用下一层，cpp文件直接调用最上层封装好的头文件即可。   
前面已经出现过的.h文件就不再重复说明。
* bigint.h; loadstor.h; mp_core.h; rounding.h; rng.h;
* types.h, secmem.h, exceptn.h; mem_ops.h, bswap.h; mp_asmi.h, ct_utils.h; mutex.h;
* build.h, assert.h, mem_ops.h; ; mp_madd.h, bit_ops.h;  
* compile.h; ; mul128.h; 

1. **bigint.h**
```c++

#include "bigint.h"
/**
1. 定义了大整数的基本特性，如符号(positive, negative)和进制除零处理等。
2. 定义创建大整数的方法，创建一个随机大整数、创建一个空(empty)大整数、
   通过字符串创建一个大整数(通过输入的参数的前缀进行解析，如0x解析为16进制)、
   通过数组arrary创建大整数(利用C++函数重载特性，设计不同函数对应不同的传参)。
3. 定义大整数的运算符(+，-，*，/，+=，-=，++，--等)对应的操作。
4. 定义大整数bit移位操作(<<=，>>=)。
5. 定义大整数布尔运算。
6. 定义大整数模运算(模加，模减，模除，乘法逆元，开平方等)。
7. 定义大整数的进制转换，符号变换，比特指定位置操作等。
8. 定义大整数的长度等操作。
*/
    -> #include "types.h"
    -> #include "secmem.h"
    -> #include "exceptn.h"
/// end of bigint.h
```
**一级调用**
```c++
#include "types.h"
    | bigint.h -> types.h
    --> #include "build.h" //二级调用
    --> #include "assert.h" //二级调用
//define types

//////////////////////////////////////////////
    using byte   = std::uint8_t;
    using u16bit = std::uint16_t;
    using u32bit = std::uint32_t;
    using u64bit = std::uint64_t;
    using s32bit = std::int32_t;
//////////////////////////////////////////////

/*
* These typedefs are no longer used within the library headers
* or code. They are kept only for compatability with software
* written against older versions.
*/

/// end of types.h
```
```c++
#include "secmem.h"
    from | bigint.h -> secmem.h
    -> #include "types.h" //一级调用
    --> #include "mem_ops.h" //二级调用
// code
class secure_allocator{
      /*
      * Assert exists to prevent someone from doing something that will
      * probably crash anyway (like secure_vector<non_POD_t> where ~non_POD_t
      * deletes a member pointer which was zeroed before it ran).
      * MSVC in debug mode uses non-integral proxy types in container types
      * like std::vector, thus we disable the check there.
      */
}

/// end of secmem.h
```
```c++
#include "exceptn.h"
    from | bigint.h -> exceptn.h
    -> #include "types.h" //一级调用
主要是异常处理的相对应的动作，通过枚举的方式进行映射。
/**
* Different types of errors that might occur
*/
enum class ErrorType {......}

class BOTAN_PUBLIC_API(2,0) ...... : ......

/// end of exceptn.h
```
**二级调用**
```c++
#include "build.h"
    from | bigint.h -> types.h -> build.h
// 通过 #define 进行宏定义
// 主要是configure配置作用
```
```c++
#include "assert.h"
    from | bigint.h -> types.h -> assert.h
    --> #include "build.h" //二级调用
    ---> #inlcude "compile.h" //三级调用
// make assertion and will be called at some special conditions
/// end of assert.h
```
```c++
#include "mem_ops.h"
    from | bigint.h -> secmem.h -> memops.h

// 主要为大整数的运算设计相应的虚拟内存空间，便于相应的数学计算等操作。
// 这个头文件中定义为大整数所设计的数据结构的相应运算操作。
template<typename T> inline void copy_mem(T* out, const T* in, size_t n) {}
template<typename T> inline void clear_mem(T* ptr, size_t n) {}
......
/// end of mem_ops.h
```
**三级调用**
```c++
#include "compile.h"
    from | bigint.h -> types.h -> assert.h -> compile.h
// 主要是编译相关的配置定义等。
// 考虑到ckb的特性，并且我们采用RISC-V的gcc工具链编译C和C++文件，这里我们对其不做考虑。
//// end of compile.h
```

2. **loadstor.h**
```c++
#include "loadstor.h"
    -> #include "types.h"
    -> #include "bswap.h"
    -> #include "mem_ops.h"
// 主要是对字节或字等基础数据结构的处理操作。(bits, bytes, words......)
/// end of loadstor.h
```
**一级调用**
```c++
#include "types.h"
#inlcude "bswap.h"
    loadstor.h -> bswap.h
// Swap 16, 32, 64 bit integer
/// end of bswap.h
```

3. **mp_core.h**
```c++
#include "mp_core.h"
    -> #include "types.h"
    -> #include "exceptn.h"
    -> #include "mem_ops.h"
    -> #include "mp_asmi.h"
    -> #include "ct_utils.h"
// 定义大整数的高级运算，下面举一个函数的栗子
/*
* If cond == 0, does nothing.
* If cond > 0, swaps x[0:size] with y[0:size]
* Runs in constant time
*/
inline void bigint_cnd_swap(word cnd, word x[], word y[], size_t size) {......}
// 类似的还有许多这样的运算函数
/// end of mp_core.h
```
**一级调用**
```c++
#include "mp_asmi.h"
    mp_core.h -> mp_asmi.h
// 通过#define设计一些计算的汇编操作。
/*
    #define ADDSUB2_OP(OPERATION, INDEX)                     \
            ASM("movl 4*" #INDEX "(%[y]), %[carry]")         \
            ASM(OPERATION " %[carry], 4*" #INDEX "(%[x])")   \
/
/// end of mp_asmi.h
```
```c++
#inlcude "ct_utils.h"
    mp_core.h -> ct_utils.h
// ct is constant time
class Mask
{
    /**
    * A Mask type used for constant-time operations. A Mask<T> always has value
    * either 0 (all bits cleared) or ~0 (all bits set). All operations in a Mask<T>
    * are intended to compile to code which does not contain conditional jumps.
    * This must be verified with tooling (eg binary disassembly or using valgrind)
    * since you never know what a compiler might do.
    */
}
/// end of ct_utils.h
```
**二级调用**
```c++
#inlcude "mp_madd.h"
    from | mp_core.h -> mp_asmi.h -> mp_madd.h
    #include "mul128.h"
// Word Multiply/Add
/// end of mp_madd.h
```
```c++
#include "bit_ops.h"
    from | mp_core.h -> ct_utils.h -> bit_ops.h
// bit操作定义
/// end of bit_ops.h
```
**三级调用**
```c++
#include "mul128.h"
    from | mp_core.h -> mp_asmi.h -> mp_madd.h -> mul128.h
/**
* Perform a 64x64->128 bit multiplication
*/
/// end of mul128.h
```

4. **rounding.h**
```c++
#include "rounding.h"
// Round up & Round down & Clamp
inline size_t round_up(size_t n, size_t align_to) {...}
template<typename T> inline constexpr T round_down(T n, T align_to) {...}
inline size_t clamp(size_t n, size_t lower_bound, size_t upper_bound) {...}
/// end of rounding.h
```
5. **rng.h**
```c++
#include "rng.h"
    -> #include "secmem.h"
    -> #include "exceptn.h"
    -> #include "mutex.h" // no threads class lock_guard final {...}
/**
* An interface to a cryptographic random number generator
*/
class BOTAN_PUBLIC_API(2,0) RandomNumberGenerator {...}

/**
* Hardware_RNG exists to tag hardware RNG types (PKCS11_RNG, TPM_RNG, RDRAND_RNG)
*/
class BOTAN_PUBLIC_API(2,0) Hardware_RNG : public RandomNumberGenerator {...}

/**
* Null/stub RNG - fails if you try to use it for anything
* This is not generally useful except for in certain tests
*/
class BOTAN_PUBLIC_API(2,0) Null_RNG final : public RandomNumberGenerator {...}

/**
* Wraps access to a RNG in a mutex
* Note that most of the time it's much better to use a RNG per thread
* otherwise the RNG will act as an unnecessary contention point
*/
class BOTAN_PUBLIC_API(2,0) Serialized_RNG final : public RandomNumberGenerator {...}

```

## 数论依赖关系
依赖于大整数Bigint，一些典型的数论运算的代码实现。
* numthry.h, monty.h, curve_nistp.h, pow_mod.h, hash.h, reducer.h
* primality.h, monty_exp.h, buf_comp.h
1. **numthry.h**
```c++
#include "numthry.h"
    -> #include "bigint.h" 
// 定义数论相关计算的函数，规定参数传入，主要用到了我们已经定义好的大整数。
/**
* Compute the greatest common divisor
* @param x a positive integer
* @param y a positive integer
* @return gcd(x,y)
*/
BigInt BOTAN_PUBLIC_API(2,0) gcd(const BigInt& x, const BigInt& y);

/**
* Modular inversion
* @param x a positive integer
* @param modulus a positive integer
* @return y st (x*y) % modulus == 1 or 0 if no such value
* Not const time
*/
BigInt BOTAN_PUBLIC_API(2,0) inverse_mod(const BigInt& x,
                                         const BigInt& modulus);
// ......
/// end of numthry.h
```

2. **monty.h**
```c++
#include "monty.h"
    -> #include "bigint.h"
/**
* The Montgomery representation of an integer
*/
class BOTAN_UNSTABLE_API Montgomery_Int final {...}

/**
* Parameters for Montgomery Reduction
*/
class BOTAN_UNSTABLE_API Montgomery_Params final {...}
/// end of monty.h
```

3. **curve_nistp.h**
```c++
#include "curve_nistp.h"
    -> #include "bigint.h"
```

4. **pow_mod.h**
```c++
#include "pow_mod.h"
    -> #include "bigint.h"
/**
* Modular Exponentiator Proxy
*/
class BOTAN_PUBLIC_API(2,0) Power_Mod {...}

/**
* Fixed Exponent Modular Exponentiator Proxy
*/
class BOTAN_PUBLIC_API(2,0) Fixed_Exponent_Power_Mod final : public Power_Mod {...}

/**
* Fixed Base Modular Exponentiator Proxy
*/
class BOTAN_PUBLIC_API(2,0) Fixed_Base_Power_Mod final : public Power_Mod {...}
```

5. **reducer.h**
```c++
#include "reducer.h"
    -> #include "numthry.h"
/**
* Modular Reducer (using Barrett's technique)
*/
class BOTAN_PUBLIC_API(2,0) Modular_Reducer
{
    BigInt reduce(const BigInt& x) const;
    BigInt multiply(const BigInt& x, const BigInt& y) const
         { return reduce(x * y); }
    BigInt square(const BigInt& x) const
         { return reduce(Botan::square(x)); }
    // ...
}
/// end of reducer.h
```

6. **primality.h**
```c++
#include "primality.h"
    -> #include "types.h"
// do some primality test
// Perform Lucas primality test
bool BOTAN_TEST_API is_lucas_probable_prime(const BigInt& n, const Modular_Reducer& mod_n);
// ...
/// end of primality.h
```

7. **monty_exp.h**
```c++
#include "monty_exp.h"
// g^x mod p
/// end of monty_exp.h
```

8. **hash.h**
```c++
#include "hash.h"
    -> #include "buf_comp.h"
/**
* This class represents hash function (message digest) objects
*/
class BOTAN_PUBLIC_API(2,0) HashFunction : public Buffered_Computation {...}
/// end of hash.h
```

9. **buf_comp.h**
```c++
#include "buf_comp.h"
    -> #include "secmem.h"
    from | hash.h -> buf_comp.h
/**
* This class represents any kind of computation which uses an internal
* state, such as hash functions or MACs
*/
class BOTAN_PUBLIC_API(2,0) Buffered_Computation {...}
```

## BigInt 具体实现
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


# 有限群层(密码学算法的数学基础--Group)
密码学与有限循环群。
现代密码学算法和协议中，消息是作为有限空间中的数字或元素来处理的。加密和解密的各种操作必须在消息之间进行变换，以使变换服从有限消息空间内部的封闭性。然而，数的一般运算诸如加减乘除并不满足有限空间内部的封闭性。所以密码算法通常运行于具有某些保持封闭性的代数结构的空间中，这种代数结构就是有限循环群。在数学中，群是一种代数结构，由一个集合以及一个二元运算组成。群必须满足以下四个条件：封闭性，结合律，存在单位元和存在逆元。
这里我们选择两个密码学领域中典型和最常用的有限群进行代码封装。
## 离散对数群依赖关系
* dl_group.h, numthry.h, reducer.h, monty.h, der_enc.h, ber_dec.h, pem.h, workfactor.h, monty_exp.h
* bigint.h, asn1_obj.h, secmem.h, exceptn.h, data_src.h, types.h, 

1. **dl_group.h**
```c++
#include "dl_group.h"
    -> #include "bigint.h"
/**
* This class represents discrete logarithm groups. It holds a prime
* modulus p, a generator g, and (optionally) a prime q which is a
* factor of (p-1). In most cases g generates the order-q subgroup.
*/
class BOTAN_PUBLIC_API(2,0)  final
{
    // 离散对数群属性变量
    // 定义计算类型的函数
    // ...
}
/// end of dl_group.h
```

2. **der_enc.h**
```c++
#include "der_enc.h"
    -> #include "asn1_obj.h"
/**
* General DER Encoding Object
*/
class BOTAN_PUBLIC_API(2,0) DER_Encoder final
{
    // 构造函数、特性函数、计算函数等   
}
// end of der_enc.h
```
**一级调用**
```c++
#include "asn1_obj.h"
    from | der_enc.h -> asn1_obj.h
    #include "secmem.h"
    #include "exceptn.h"
/**
* Basic ASN.1 Object Interface
*/
class BOTAN_PUBLIC_API(2,0) ASN1_Object {...}
/**
* BER Encoded Object
*/
class BOTAN_PUBLIC_API(2,0) BER_Object final {...}
// and some Error Exception
/// end of asn1_obj.h
```

3. **ber_dec.h**
```c++
#include "ber_dec.h"
    #include "asn1_obj.h"
    #include "data_src.h"
/**
* BER Decoding Object
*/
class BOTAN_PUBLIC_API(2,0) BER_Decoder final {...}
/*
* Decode an OPTIONAL or DEFAULT element
*/
template<typename T>
BER_Decoder& BER_Decoder::decode_optional ();
/*
* Decode an OPTIONAL or DEFAULT element
*/
template<typename T>
BER_Decoder& BER_Decoder::decode_optional ();
/*
* Decode a list of homogenously typed values
*/
template<typename T>
BER_Decoder& BER_Decoder::decode_list ();
/// end of ber_dec.h
```
**一级调用**
**data_src.h**
```c++
#include "data_src.h"
    from | ber_dec.h
    #include "secmem.h"
/**
* This class represents an abstract data source object.
*/
class BOTAN_PUBLIC_API(2,0) DataSource {...}
/**
* This class represents a Memory-Based DataSource
*/
class BOTAN_PUBLIC_API(2,0) DataSource_Memory final : public DataSource {...}
/**
* This class represents a Stream-Based DataSource.
*/
class BOTAN_PUBLIC_API(2,0) DataSource_Stream final : public DataSource {...}
/// end of data_src.h
```

4. **pem.h**
```c++
#include "pem.h"
    #include "secmem.h"
/**
* Encode some binary data in PEM format
*/
BOTAN_PUBLIC_API(2,0) std::string encode();
/**
* Decode PEM data
*/
BOTAN_PUBLIC_API(2,0) secure_vector<uint8_t> decode();
/// end of pem.h
```

5. **workfactor.h**
```c++
#include "workfactor.h"
    #include "types.h"
/**
* Estimate work factor for discrete logarithm
*/
BOTAN_PUBLIC_API(2,0) size_t dl_work_factor();
/**
* Return the appropriate exponent size to use for a particular prime group. 
*/
BOTAN_PUBLIC_API(2,0) size_t dl_exponent_size();
/**
* Estimate work factor for integer factorization
*/
BOTAN_PUBLIC_API(2,0) size_t if_work_factor();
/**
* Estimate work factor for EC discrete logarithm
*/
BOTAN_PUBLIC_API(2,0) size_t ecp_work_factor();
```


## 离散对数群 DL group
离散对数群，将该群封装为一个class，其底层需要我们在Layer1封装好的BigInt和数论相关的代码。整个类的伪代码如下所示:  
```c++
#include "bigint.h"  
#include "numthry.h"  
/**  
This class represents discrete logarithm groups. It holds a prime modulus p, a generator g, and (optionally) a prime q which is a factor of (p-1). In most cases g generates the order-q subgroup.  
*/  
class DL_Group  
{  
    // 构造函数，Create a new group with kinds of paramenters.  
    DL_Group(some paras[])  
    // 通过layer1的BigInt类中的方法进行群的相应运算.  
    BigInt mod_p(const BigInt& x);  
    BigInt multiply_mod_p(BigInt& x, BigInt& y);  
    BigInt power_g_p(BigInt& x);  
    BigInt ... ;  
}  
```

## 椭圆曲线群依赖关系
* ec_group.h, point_mul.h, primality.h, ber_dec.h, der_enc.h, pem.h, reducer.h, mutex.h, rng.h
* point_gfp.h, asn1_oid.h
* curve_gfp.h, asn1_obj.h 
* bigint.h
1. **ec_group.h**
```c++
#include "ec_group.h"
    #include "point_gfp.h"
    #include "asn1_old.h"
class CurveGFp;

class EC_Group_Data;
class EC_Group_Data_Map;

/**
* Class representing an elliptic curve
*
* The internal representation is stored in a shared_ptr, so copying an
* EC_Group is inexpensive.
*/
class BOTAN_PUBLIC_API(2,0) EC_Group final {...}
/// end of ec_group.h
```
**一级调用**
**point_gfp.h**
```c++
#include "point_gfp.h"
    from | ec_group.h -> point_gfp.h
    #include "curve_gfp.h"
    #include "exceptn.h"

/**
* This class represents one point on a curve of GF(p)
*/
class BOTAN_PUBLIC_API(2,0) PointGFp final {...}
// relational operators
inline bool operator!=(const PointGFp& lhs, const PointGFp& rhs) {...}
// arithmetic operators
inline PointGFp operator+(const PointGFp& lhs, const PointGFp& rhs) {...} // + - *
/// end of point_gfp.h
```
**asn1_oid.h**
```c++
#include "asn1_oid.h"
    from | ec_group.h -> asn1_oid.h
    #include "asn1_obj.h"
/**
* This class represents ASN.1 object identifiers.
*/
class BOTAN_PUBLIC_API(2,0) OID final : public ASN1_Object {...}
// some oid oprations
/// end of san1_oid.h

```

**二级调用**
**curve_gfp.h**
```c++
#include "curve_gfp.h"
    from | ec_group.h -> point_gfp.h -> curve_gfp.h
    #include "bigint.h"
/**
* This class represents an elliptic curve over GF(p)
* There should not be any reason for applications to use this type.
* If you need EC primitives use the interfaces EC_Group and PointGFp
*/
class BOTAN_UNSTABLE_API CurveGFp final {...}
/// end of curve_gfp.h
```

2. **point_mul.h**
```c++
#include "point_mul.h"
    #include "point_gfp.h"
// 定义在类中
PointGFp mul() const;
/*
* Return (g1*k1 + g2*k2)
* Not constant time, intended to use with public inputs
*/
PointGFp multi_exp(const BigInt& k1, const BigInt& k2) const;
/// end of point_mul.h
```


## 椭圆曲线群 EC group
椭圆曲线群，同样将其封装为一个class，底层需要BigInt和数论相应的代码。
```c++
#include "point_gfp.h"  
#include "curve_gfp.h"  
#include "bigint.h"  
class EC_Group  
{  
    // 不同参数的构造函数  
    EC_Group(some paras[])  
    // 在类的内部定义一些点坐标的运算  
    PointGFp point(BigInt& x,BigInt& y);  
    PointGFp point_multiply(BigInt& x, PointGFp& pt, BigInt& y);  
    // 数论相关的运算  
    ...;  
}
``` 
与EC_Group同级的还有class EC_Group_Data和class EC_Group_Data_Map用来描述椭圆曲线群的属性信息。

