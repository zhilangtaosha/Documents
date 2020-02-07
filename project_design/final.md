# 整体方案
## 第一层 大整数群(BigInt)
BigInt大整数是密码学运算中的计算单位基础，同样也是我们构建的密码库的第一层，基于C++的函数重载等特性，可以为单个运算符定义多种传参方式。这里我们将整个大整数群逻辑上分成两部分，BigInt(大整数)和NumThrory(数论)。
## BigInt 依赖关系
依赖关系主要描述的是cpp文件包含.h文件的详细信息，以及对应.h头文件的具体内容。   
**.h头文件内容**  
按照层级类划分，即上一层调用下一层，cpp文件直接调用最上层封装好的头文件即可。   
前面已经出现过的.h文件就不再重复说明。
* bigint.h, loadstor.h, mp_core.h, rounding.h, rng.h;
* types.h, secmem.h, exceptn.h, mem_ops.h, bswap.h, mp_asmi.h, ct_utils.h, mutex.h;
* build.h, assert.h, mem_ops.h, mp_madd.h, bit_ops.h;  
* compile.h, mul128.h; 

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


## 第二层 有限群层(密码学算法的数学基础--Group)
密码学与有限循环群。
现代密码学算法和协议中，消息是作为有限空间中的数字或元素来处理的。加密和解密的各种操作必须在消息之间进行变换，以使变换服从有限消息空间内部的封闭性。然而，数的一般运算诸如加减乘除并不满足有限空间内部的封闭性。所以密码算法通常运行于具有某些保持封闭性的代数结构的空间中，这种代数结构就是有限循环群。在数学中，群是一种代数结构，由一个集合以及一个二元运算组成。群必须满足以下四个条件：封闭性，结合律，存在单位元和存在逆元。
这里我们选择两个密码学领域中典型和最常用的有限群进行代码封装。
### 离散对数群依赖关系
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


### 离散对数群 DL group
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

### 椭圆曲线群依赖关系
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


### 椭圆曲线群 EC group
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



## 第三层 密码学原语层
主要包括一些具有原子性质的密码学算法，可以独立完成某个功能，
### 哈希函数
这里选取区块链中最常见的哈希函数，如SHA-256, RIPEMD-160, BLAKE, Keccak, SM3等，这些哈希算法相对独立，可单独实现，关键是为后面的密码学方案层的复合哈希提供底层支持。

### 基本数字签名

#### ECDSA
**理论依赖关系**  
第一层：

大整数结构和常见运算，随机数产生

第二层：

 secp256k1 椭圆曲线，GF(p)中曲线上的点结构及运算：点的乘法、盲化乘法、指数乘法等

第三层：

杂凑函数，ECDSA的公私钥结构、构造方法

其他：

内存工具，编码算法
**代码依赖关系**  
```c++
#include <botan/ecdsa.h>
// 定义了ECDSA的公私钥结构、构造方法
->	#include <botan/ecc_key.h>
	// 定义了抽象ECC公共密钥结构、构造方法
    -> 	 #include <botan/ec_group.h>
    	// 定义了椭圆曲线类，定义了椭圆曲线域参数数据结构，枚举了椭圆曲线编码方式,提供基本运算（曲线上的乘法、计算曲线零点等等）
    	 ->	   #include <botan/point_gfp.h>
    			// 定义GF(p)中曲线上的点结构及基本运算
    		     ->		#include <botan/curve_gfp.h>// 定义GF(p)中的曲线结构，声明了NIST P-192、NIST P-384等曲线
    					->	 #include <botan/bigint.h>// BigInt大整数类声明和构造方法，声明了随机数发生器类
 							#include <memory>
                         #include <botan/exceptn.h>
                         #include <vector>
                #include <botan/asn1_oid.h>
                #include <memory>
                #include <set>
		#include <botan/pk_keys.h>// 定义了公钥密码学中公钥、私钥、秘密导出密钥的基本结构、构造方法，枚举了签名类型，提供创建十六进制指纹方法
#include <botan/internal/pk_ops_impl.h>
// 定义了使用EME编码的加解密、使用密钥导出函数KDF的加解密及密钥协商、使用EMSA编码的签名及验证
#include <botan/internal/point_mul.h>
// 提供GF(p)中椭圆曲线上点的乘法、盲化乘法、指数乘法
->	#include <botan/point_gfp.h>
    // 定义GF(p)中曲线上的点结构及基本运算
#include <botan/keypair.h>// 定义了密钥对结构，提供了一对密钥的一致性的检测方法
#include <botan/reducer.h>// 定义了Barrett除法
->   #include <botan/numthry.h> // 数论，提供融合乘加等运算方法
    ->	#include <botan/bigint.h>// BigInt大整数类声明和构造方法，声明了随机数发生器类
#include <botan/emsa.h>// 定义了EMSA编码方法
```

#### RSA签名
**理论依赖关系**  
第一层：

大整数结构和常见运算，随机数产生

第二层：

整数分解

第三层：

RSA的公私钥结构、构造方法，公钥密码学中的常见操作

其他：

内存工具，编码算法

**代码依赖关系**  

```c++
#include <botan/rsa.h>
// 定义了RSA签名算法的公私钥结构
        ->	#include <botan/pk_keys.h>// 定义了公钥密码学中公钥、私钥、秘密导出密钥的基本结构、构造方法，枚举了签名类型，提供创建十六进制指纹方法
            #include <botan/bigint.h>	// 定义了大整数结构和常见运算
            #include <string>
            #include <memory>
            #include <vector>
#include <botan/internal/pk_ops_impl.h>
// 定义了使用EME编码的加解密、使用密钥导出函数KDF的加解密及密钥协商、使用EMSA编码的签名及验证
        ->	#include <botan/pk_ops.h>
            // 定义公钥密码学中的常见操作为基本类，如加密、解密、密钥封装加解密、密钥协商、签名、验证
            #include <botan/eme.h> // 针对加密算法的编码方法
            #include <botan/kdf.h> // 定义了密钥导出函数KDF
            #include <botan/emsa.h> // 定义了EMSA编码方法
#include <botan/keypair.h>
// 定义了密钥对结构，提供了一对密钥的一致性的检测方法
        ->	#include <botan/pk_keys.h>
             // 定义了公钥密码学中公钥、私钥、秘密导出密钥的基本结构、构造方法，枚举了签名类型，提供创建十六进制指纹方法
#include <botan/blinding.h>
// 提供盲化、去盲化方法
        ->	#include <botan/bigint.h> // 定义了大整数结构和常见运算
            #include <botan/reducer.h> // 定义了Barrett除法
            #include <functional>
#include <botan/reducer.h>// 定义了Barrett除法
#include <botan/workfactor.h>// 定义了针对某算法的功系数
#include <botan/der_enc.h>// 定义了DER可辨别编码规则
#include <botan/ber_dec.h>// 定义了BER基本编码规则
#include <botan/monty.h>// 定义了Montgomery算法需要的数据结构及基本运算
#include <botan/divide.h>// 定义大整数除法及模运算
#include <botan/internal/monty_exp.h>// 定义了Montgomery指数运算
```



#### ED25519 
**理论依赖关系**  
第一层：

大整数结构和常见运算，随机数产生

第二层：

Curve25519 椭圆曲线，GF(p)中曲线上的点结构及运算：点的乘法、盲化乘法、指数乘法等

第三层：

杂凑函数，ed25519的公私钥结构、构造方法

其他：

内存工具，编码算法

**代码依赖关系**  

```c++
#include <botan/ed25519.h>	
// 定义了此算法用到的公私钥的结构、构造方法，签名算法、验证算法的参数
        -> #include <botan/pk_keys.h>
        // 定义了公钥密码学中公钥、私钥、秘密导出密钥的基本结构、构造方法，枚举了签名类型，提供创建十六进制指纹方法
                ->	#include <botan/secmem.h>
                    // 定义了安全内存缓冲区
                    ->	#include <botan/types.h> 
                        // 包含word等类型定义，不再使用，出于兼容性考虑而保留
                        #include <botan/mem_ops.h> 
                        // 定义了内存操作
                        ->	  #include <botan/types.h>// 包含word等类型定义，不再使用，出于兼容性考虑而保留
                              #include <cstring> // 字符串
                              #include <type_traits> // 类型处理
                              #include <vector> // 向量
                         #include <vector> // 向量
                         #include <algorithm> // 常用算法
                         #include <deque>  // 双端队列
                         #include <type_traits> // 类型处理
                    #include <botan/asn1_oid.h>
                    // 定义了 ASN.1 对象标识符 
                    ->   #include <botan/asn1_obj.h>
                         // 基本的ASN.1对象接口
                         #include <string> // 字符串
                         #include <vector> // 向量
                    #include <botan/alg_id.h>
                    // 定义了算法标识符
                    ->  #include <botan/asn1_obj.h> // 基本的ASN.1对象接口
                        #include <botan/asn1_oid.h>// 定义了 ASN.1 对象标识符 
                        #include <string>// 字符串
                        #include <vector>// 向量
                    #include <botan/pk_ops_fwd.h>
                    // 常见操作声明
                    #include <string>
#include <botan/internal/ed25519_internal.h>
// 定义了扩展的群元素结构 ge_p3。定义了32位/64位多位、进位加法器，定义了群元素与字符串的比较函数，定义了群元素与标量乘法
    ->	#include <botan/internal/ed25519_fe.h>
        // 定义了Curve25519曲线上元素 FE_25519 fe的基本结构，以及基本操作：加、复制、转换、判断非零等等
        #include <botan/loadstor.h>
        // 定义了字节提取、字节与大整数的转换、载入大小写字母、存储数据等函数
        #include <botan/sha2_64.h> 
        // 定义了SHA_384，SHA_512，SHA_512_256
        ->	#include <botan/mdx_hash.h> // mdx杂凑函数
#include <botan/rng.h>
// 随机数发生器类的实现
        ->	#include <botan/secmem.h>// 定义了安全内存缓冲区
             #include <botan/exceptn.h>// 错误类型
             #include <botan/mutex.h>
             #include <chrono>
             #include <string>
```

#### EC-Schnorr
**理论依赖关系**  
第一层：

大整数结构和常见运算，随机数产生

第二层：

椭圆曲线类，GF(p)中曲线上的点结构及运算：点的乘法、盲化乘法、指数乘法等

第三层：

杂凑函数，EC-Schnorr 的公私钥结构、构造方法

其他：

内存工具，编码算法  
**代码依赖关系**
> Botan 未包括，参考了 [Zilliqa 的实现](https://github.com/Zilliqa/schnorr)，其中用到了 [OpenSSL 的大整数](https://github.com/openssl/openssl/tree/master/crypto/bn)与[椭圆曲线实现](https://github.com/openssl/openssl/blob/master/include/openssl/ec.h)。该库包括基于 Schnorr 实现的多重签名。

```c++
#include <openssl/err.h>
#include <openssl/obj_mac.h>
#include <openssl/opensslv.h>
#include <array>
#include "Schnorr.h"
// 指定了可字节序列化的类所需的接口，定义公私钥、签名的数据结构，定义了签名算法类，其中规定了曲线数据结构、签名及验证方法的参数
        ->	#include <openssl/bn.h>
            // OpenSSL的大整数实现，提供基本的大整数运算
            #include <openssl/ec.h>
            // OpenSSL的椭圆曲线实现，给出Fp上椭圆曲线及点的数据结构定义，以及点运算
            #include <array>
            #include <boost/functional/hash.hpp>
            #include <memory>
            #include <string>
            #include <vector>
#include "SchnorrInternal.h"
// 规定了常用的数据大小，提供了序列化大整数和椭圆曲线点的方法
        ->	#include <openssl/bn.h>
            #include <openssl/ec.h>
            #include <array>
            #include <boost/algorithm/hex.hpp>
            #include <boost/functional/hash.hpp>
            #include <memory>
            #include <mutex>
            #include <string>
            #include <vector>
            #include "generate_dsa_nonce.h"// 产生一个[0 , range)内的随机数
            #include "Sha2.h"// 实现SHA2哈希算法
```

#### BLS签名
**理论依赖关系**  
第一层：

大整数结构和常见运算，随机数产生，pairing 运算

第二层：

椭圆曲线类，GF(p)中曲线上的点结构及运算：点的乘法、盲化乘法、指数乘法等

第三层：

杂凑函数，BLS 的公私钥结构、构造方法

其他：

内存工具，编码算法  
**代码依赖关系**  
> Botan 未包括，参考了[libBLS](https://github.com/skalenetwork/libBLS)。 libBLS中也包含门限签名的实现

bls.cpp

实现了bls类中的哈希计算、密钥生成、签名、验证、密钥恢复、签名恢复函数

```c++
#include <bls/bls.h>
// 包含bls类声明，bls.cpp中函数声明
        ->  #include <third_party/cryptlite/sha256.h>// 调用第三方sha256函数的实现
            #include <string>
            #include <vector>
            #include <utility>
            #include <memory>
            #include <iostream>
            #include <libff/algebra/curves/alt_bn128/alt_bn128_pp.hpp>//libff密码库对alt_bn128（Barreto-Naehrig椭圆曲线）上公共参数的实现
#include <chrono>
#include <ctime>
#include <stdexcept>
#include <thread>
#include <bitset>
#include <boost/multiprecision/cpp_int.hpp> // boost大整数库
#include <libff/algebra/curves/alt_bn128/alt_bn128_pairing.hpp>// libff密码库对alt_bn128（Barreto-Naehrig椭圆曲线）上配对运算的实现
#include <libff/algebra/exponentiation/exponentiation.hpp>// libff密码库对指数运算的实现
#include <libff/common/profiling.hpp>// libff密码库中声明的用于性能分析的函数
#include <bls/BLSutils.h>
// 提供初始化BLS类、有限域元素转换为字符串方法、字符串分割等方法
        ->  #include <array>
            #include <memory>
            #include <libff/algebra/curves/alt_bn128/alt_bn128_pp.hpp>//libff密码库对alt_bn128（Barreto-Naehrig椭圆曲线）上公共参数的实现
```

### 承诺协议 Pedersen commitment
Pedersen commitment 承诺协议基于离散对数群构建的密码元语，属于构建密码库的第三层。我们参照libsecp256k1上Pedersen Commitment的实现，列出项目实现时候的依赖关系、实现方法。
**理论依赖关系**  
我们选择基于secp256k1椭圆曲线群的构造，描述Pedersen commitment 在代码实现上的依赖关系
基于  [该库](https://github.com/ElementsProject/secp256k1-zkp)  
第一层  
大整数结构和常见运算，随机数产生  
第二层  
secp256k1 椭圆曲线，GF(p)中曲线上的点结构及运算：点的乘法、盲化乘法、指数乘法等  
**代码依赖关系**  
```c++
#include "eckey.h" // 椭圆曲线公私钥的操作
#include "ecmult_const.h" // secp256k1_ecmult_const
#include "ecmult_gen.h" // SECP256K1椭圆曲线的初始化、构建
#include "group.h" // SECP256K1群的各类运算、初始化
#include "field.h" // SECP256K1有限域上的各类运算、初始化
#include "scalar.h" //标量库 用于序列化盲化因子
# include "secp256k1.h" //定义调用的API
# include "secp256k1_generator.h"
``` 
**具体实现方法**  
我们选择基于secp256k1椭圆曲线群，描述Pedersen commitment 在代码实现所需要实现的方法

```
//群相关
secp256k1_pedersen_scalar_set_u64()
secp256k1_pedersen_ecmult_small()
secp256k1_pedersen_ecmult()
--------------------------------------------
//准备操作
secp256k1_generator
secp256k1_pedersen_commitment_parse()
secp256k1_pedersen_commitment_serialize()
secp256k1_pedersen_context_initialize()
-------------------------------------------
//生成、盲加、验证
secp256k1_pedersen_commit()
secp256k1_pedersen_blind_sum()
secp256k1_pedersen_verify_tally()
-------------------------------------------
secp256k1_pedersen_blind_generator_blind_sum()

```

## 第四层 密码学方案层
由很多原子算法和协议组合在一起实现一个更复杂功能的密码学方案。  

### Boneh 多重签名  

> [Compact Multi-Signatures for Smaller Blockchains](https://eprint.iacr.org/2018/483.pdf) 
>
> 此方案缺少可信的 c/c++实现，可暂时参考 rust library https://github.com/lovesh/signature-schemes
>
> 此实现使用了Apache Milagro库中的BLS12-381曲线以及签名和验证API

### CryptoNote 环签名
具体参考https://github.com/cryptonotefoundation/cryptonote/blob/8edd998304431c219b432194b7a3847b44b576c3/src/crypto/crypto.cpp

```c++
#include <alloca.h>
#include <cassert>
#include <cstddef>
#include <cstdint>
#include <cstdlib>
#include <cstring>
#include <memory>
#include <mutex>// 互斥锁类
#include "Common/Varint.h"// varint编码方法实现
#include "crypto.h"
        ->	#include <cstddef>
            #include <limits>
            #include <mutex>
            #include <type_traits>
            #include <vector>
            #include <CryptoTypes.h>
            // 定义了Hash、公私钥、签名、导出密钥、密钥镜像的数据结构
                    ->	#include <cstdint>// 提供定宽整数类型
            #include "generic-ops.h"
            // 定义了通用操作，实现了==、!=、()等操作符的重载
                    ->	#include <cstddef>
                        #include <cstring>
                        #include <functional>
            #include "hash.h"// Cryptonight哈希函数实现
#include "hash.h"// Cryptonight哈希函数实现
        ->	#include <stddef.h>
            #include <CryptoTypes.h>// 定义了Hash、公私钥、签名、导出密钥、密钥镜像的数据结构
            #include "generic-ops.h"// 定义了通用操作，实现了==、!=、()等操作符的重载
            #include "hash-ops.h"// 哈希相关的方法声明
#include "crypto-ops.h"
// 有限域上点结构以及常见操作的预声明及实现
#include "random.h"
// 提供随机数生成方法
```



### BBS04 群签名方案
> [Short Group Signatures](https://crypto.stanford.edu/~dabo/pubs/papers/groupsigs.pdf)
>
> 参考 https://github.com/FISCO-BCOS/sig-service/blob/master/algorithm/bbs04/GroupSig_BBS_Impl.cpp
>
> 代码使用了[PBC-C语言库](http://crypto.stanford.edu/pbc/download.html)

GroupSig_BBS_Impl.cpp	// 此文件为BBS04群签名算法实现  

```c++
#include "devcore/SHA3.h"
// 基于 libkeccak-tiny 实现的sha3算法
#include "devcore/CommonFunc.h"
// 提供常用方法，如签名算法枚举、将参数序列化成字符串、将element_t转化成字符串等，此处用到了字符串转换
#include "algorithm/KeyLoaderDumper.h"// 声明密钥管理类，提供密钥加载、加密存储管理、恢复系统参数等方法
        ->	#include <string>
            #include "pbc/pbc.h"
            #include "devcore/ConfigParser.h"// 解析配置文件
            #include "devcore/CommonFunc.h"// 提供常用方法，如签名算法枚举、将参数序列化成字符串、将element_t转化成字符串等
            #include "devcore/easylog.h"// 日志记录
            #include "devcore/StatusCode.h"// 定义状态码
            #include "algorithm/bbs04/GroupSig_BBS.h"// 声明 BBSGroupSig 类，定义BBS群签名算法接口
            #include "algorithm/bbs04/GroupSig_BBS_Impl.h"//bbs04算法的签名结构，生成具有指定线性对的随机数，实现BBS04算法的签名、打开签名和群验签
            #include "algorithm/GroupSigInterface.h"// 群签名接口，存储线性对配置文件路径等
#include "algorithm/bbs04/GroupSig_BBS.h"
// 声明 BBSGroupSig 类，定义BBS群签名算法接口
#include "algorithm/bbs04/GroupSig_BBS_Impl.h"
//bbs04算法的签名结构，生成具有指定线性对的随机数，实现BBS04算法的签名、打开签名和群验签
        ->	#include "pbc/pbc.h"// 使用了PBC库中的element_t、pairing_ptr等数据类型
            #include "pbc/pbc_sig.h"//PBC中基于配对的签名库，使用了bbs_group_public_key_t和bbs_manager_private_key_t密钥结构
            #include "devcore/easylog.h"// 日志记录
            #include "algorithm/LinearPair.h"// 定义了PBC支持的几种线性对
            #include "algorithm/GroupSigInterface.h"// 群签名接口，存储线性对配置文件路径等
                ->	#include "devcore/CommonStruct.h"
                    #include "devcore/ConfigParser.h"
                    #include "devcore/CommonFunc.h"// 提供常用方法，如签名算法枚举、将参数序列化成字符串、将element_t转化成字符串等
                    #include "devcore/easylog.h"// 日志记录
                    #include "algorithm/LinearPair.h"// 定义了PBC支持的几种线性对
```


### Monero RingCT 环签名
具体参考https://github.com/monero-project/monero/blob/master/src/ringct/rctSigs.cpp   
**代码依赖关系**  
```
src/ringct
├── bulletproofs.cc
├── bulletproofs.h
├── CMakeLists.txt
├── multiexp.cc
├── multiexp.h
├── rctCryptoOps.c
├── rctCryptoOps.h
├── rctOps.cpp
├── rctOps.h
├── rctSigs.cpp
├── rctSigs.h
├── rctTypes.cpp
└── rctTypes.h
环签名主要实现在rctSigs.cpp中
```
`src/ringct`目录下的源码主要依赖为：

- `src/crypto/`目录下定义的密码学函数；
- `src/common/`, `src/serialization/`和`contrib/epee/include/`目录下的工具类函数；
- `src/cryptonote_config.h`和`cryptonote_basic/cryptonote_format_utils.h`文件中定义的通用格式和函数；
- `boost`和`openssl/ssl.h`等外部依赖库。

rctSigs.cpp // 实现多层自发匿名组签名（MLSAG签名）

```c++
#include "misc_log_ex.h"// 日志
#include "common/perf_timer.h"// 性能计时器
#include "common/threadpool.h"// 线程池
#include "common/util.h"// 提供系统版本信息读取、IP解析、正则表达式转换等常用工具
#include "rctSigs.h"
// 环签名生成方法genRct、验证方法verRct、打开方法decodeRct的声明，从区块链获取密钥方法的声明等
	 ->	   #include <cstddef>
            #include <vector>
            #include <tuple>
            #include "crypto/generic-ops.h"// 定义了通用操作，实现了==、!=、()等操作符的重载
            #include "crypto/random.h"// 声明方法：产生随机字节、增加额外的熵
         			->	  #include <stddef.h>
            #include "crypto/keccak.h"// SHA3算法实现
         			->	  #include <stdint.h>
						#include <string.h>
            #include "crypto/crypto.h"
         	//公钥public_keyM、公钥public_keyV、私钥secret_keyV、曲线点ec_point、签名signature等类的声明，密钥生成、验证、签名验证、交易证明的生成与验证等方法的声明
         			->	  #include <cstddef>
                            #include <iostream>
                            #include <boost/optional.hpp>
                            #include <type_traits>
                            #include <vector>
                            #include <random>
                            #include "common/pod-class.h"// POD (Plain Old Data) 数据类型
                            #include "memwipe.h"// 内存清理
                            #include "mlocker.h"// 内存锁定
                            #include "generic-ops.h"// 定义了通用操作，实现了==、!=、()等操作符的重载
                            #include "hex.h"// 十六进制转换方法
                            #include "span.h"// 此类可用作采用可写或只读数据序列的函数的参数类型
                            #include "hash.h"// Cryptonight哈希函数实现
                                    ->	#include <stddef.h>
                                        #include <CryptoTypes.h>// 定义了Hash、公私钥、签名、导出密钥、密钥镜像的数据结构
                                        #include "generic-ops.h"// 定义了通用操作，实现了==、!=、()等操作符的重载
                                        #include "hash-ops.h"// 哈希相关的方法声明
                            #include "random.h"// 提供随机数生成方法
            #include "rctTypes.h"
         	// 定义了rct名称空间中所有的对象，包括密钥key、环密钥ctkey、签名rctSigBase、证明bulletproof等结构容器以及对应的转换函数
                      ->    #include <cstddef>
                            #include <vector>
                            #include <iostream>
                            #include <cinttypes>
                            #include <sodium/crypto_verify_32.h>// sodium库中定义的验证函数
         					->	#include <string>
					     #include "crypto/crypto-ops.h"// 有限域上点结构以及常见操作的预声明及实现
                            #include "crypto/random.h"// 提供随机数生成方法
                            #include "crypto/keccak.h"// SHA3算法实现
                            #include "crypto/generic-ops.h"// 定义了通用操作，实现了==、!=、()等操作符的重载
                            #include "crypto/crypto.h" 同上
                            #include "hex.h"
                            #include "span.h"
                            #include "serialization/vector.h"// 以下均为序列化工具
                            #include "serialization/debug_archive.h"
                            #include "serialization/binary_archive.h"
                            #include "serialization/json_archive.h"
            #include "rctOps.h"
         // 声明与向量或点的操作有关的常量和函数（初始化，随机生成，加法，乘法，Pedersen承诺，哈希到点等）
         			->    #include <cstddef>
                            #include <tuple>
                            #include "crypto/generic-ops.h"
                            #include "crypto/random.h"
                            #include "crypto/keccak.h"
                            #include "rctCryptoOps.h"
         				// 仅包含sc_reduce32copy函数的声明，计算一个32位输入的模除法，模数为曲线Ed25519的主子群的阶（2252 + 27742317777372353535851937790883648493）
                            #include "crypto/crypto.h"
                            #include "rctTypes.h"
#include "bulletproofs.h"//	声明两个主要功能bulletproof_PROVE和bulletproof_VERIFY
#include "cryptonote_basic/cryptonote_format_utils.h"// 定义通用格式和函数
```





### 基于BLS的聚合签名
基于BLS签名的聚合签名方案属于第四层密码方案层，其密码原语基于双线性对、椭圆曲线。我们参照 [libBLS](https://github.com/skalenetwork/libBLS)，列出项目实现时候的依赖关系、实现方法。  
**理论依赖关系**  
**第一层**：

大整数结构和常见运算，随机数产生

**第二层**：

alt_bn128椭圆曲线群
双线性对

**第三层**：

BLS签名  
**代码实现依赖关系**  
alt_bn128椭圆曲线群：[libff](https://github.com/scipr-lab/libff)椭圆曲线库   
双线性对：[pbc by Ben Lynn](https://github.com/skalenetwork/pbc)
**实现方法、类**  

```
BLSSigShareSet - class for a set of pieces of signature. Has methods Add (to add a piece of signature) and merge ( to get common signature, if enough pieces of signature added)

BLSSignature - class for common signature.

```

### 基于BLS的门限签名
基于BLS签名的门限签名方案属于第四层密码方案层，其密码原语基于双线性对、椭圆曲线。我们参照 [libBLS](https://github.com/skalenetwork/libBLS)，列出项目实现时候的依赖关系、实现方法。
**理论依赖关系**  
我们选择基于alt_bn128椭圆曲线群的方案  
**第一层**  
- 大整数结构和常见运算，随机数产生  
**第二层**  
- alt_bn128椭圆曲线群  
- 双线性对  
**第三层**  
- BLS签名  
**代码依赖关系**  
alt_bn128椭圆曲线群：[libff](https://github.com/scipr-lab/libff)椭圆曲线库   
双线性对：[pbc by Ben Lynn](https://github.com/skalenetwork/pbc)  
**实现方法和类**  
```
BLSPrivateKeyShare - class for private key for each participant. Has methods sign and signWithHelper to sign hashed message.

BLSPrivateKey - class for common private key

BLSPublicKeyShare - class for public key for each participant. Has methods VerifySig and VerifySigWithHelper to Verify a piece of signature.

BLSPublicKey - class for common public key. Has method VerifySig and VerifySigWithHelper for verifying common signature.

BLSSigShare - class for a piece of common signature.

BLSSigShareSet - class for a set of pieces of signature. Has methods Add (to add a piece of signature) and merge ( to get common signature, if enough pieces of signature added)

BLSSignature - class for common signature.

All these classes (except BLSSigShareSet) can be created from shared_ptr to string(or to vector of strings) and converted to shared_ptr to string(or to vector of strings) with the method toString().
```


### 分布式密钥生成 GJKR07方案
分布式密钥生成方案属于第四层密码方案层，其密码原语基于双线性对、椭圆曲线。我们参照 [libBLS](https://github.com/skalenetwork/libBLS)，列出项目实现时候的依赖关系、实现方法。该方案用于BLS门限签名时的密钥生成。

**理论依赖关系**  
我们选择基于alt_bn128椭圆曲线群的方案。
- 第一层  
大整数结构和常见运算，随机数产生
- 第二层  
alt_bn128椭圆曲线群
双线性对  
- 第三层  
BLS签名
**代码具体实现**  
alt_bn128椭圆曲线群：[libff](https://github.com/scipr-lab/libff)椭圆曲线库   
双线性对：[pbc by Ben Lynn](https://github.com/skalenetwork/pbc)
```
实现方法和类
```
```
实现 dkg_obj类

方法有：
    DKGBLSWrapper(size_t _requiredSigners, size_t _totalSigners);

    bool VerifyDKGShare( size_t signerIndex, const libff::alt_bn128_Fr& share,
                         const std::shared_ptr<std::vector<libff::alt_bn128_G2>>& _verification_vector);

    void setDKGSecret(std::shared_ptr<std::vector< libff::alt_bn128_Fr>> _poly_ptr);

    std::shared_ptr < std::vector < libff::alt_bn128_Fr>> createDKGSecretShares();

    std::shared_ptr < std::vector <libff::alt_bn128_G2>> createDKGPublicShares();

    BLSPrivateKeyShare CreateBLSPrivateKeyShare(std::shared_ptr<std::vector<libff::alt_bn128_Fr>> secret_shares_ptr);

```
**方法交互过程**  

1.  选择参与者数n，门限值t ( t<= n)

2.  每一个参与者使用参数t，n创建一个dkg实例  
```
 DKGBLSWrapper dkg_obj(t, n);
```
3.  每一个参与者生成一个公开分享系数的向量，并广播
```
std::shared_ptr < std::vector <libff::alt_bn128_G2>>  public_shares = dkg_obj.createDKGPublicShares();
```

4.   每个参与者生成秘密分享系数的向量，并将矢量的第 j 个参数发送给 第 j 个参与者
```
 std::shared_ptr <std::vector <libff::alt_bn128_Fr>>  private_shares   
 = dkg_obj.createDKGSecretShares();
```

5.  每个参与者验证从其他参与者收到的秘密分享参数是否匹配公开参数
```
  assert(dkg_obj. VerifyDKGShare( signerIndex, secret_share, public_shares_vector));
```


6.  如果验证通过每个参与方可以通过收到的秘密分享参数生成密钥
```
   BLSPrivateKeyShare privateKeyShare = dkg_obj.CreateBLSPrivateKeyShare(secret_shares_vector);
```

### 复合哈希函数
#### X11/X13/X15/X16r/X17
**算法简介**  
X11 包括以下 11 个函数，全部皆为 NIST 哈希函数竞赛之参赛者：

BLAKE、Blue Midnight Wish、Grøstl、JH、Keccak、Skein、Luffa、CubeHash、SHAvite-3、SIMD、ECHO

这些函数都可以在第三层封装好，并在这一层调用

X11 https://github.com/dashdot/python-x11_hash

X13 https://github.com/OPEC-Coin/x13_hash

X15 https://github.com/minings/x15_hash

X16r https://github.com/brian112358/x16r_hash

X17 https://github.com/SKlayer/x17_hash  

#### Quark
**算法简介**  
Quark 哈希算法基于一级哈希函数，用六种不同的加密算法来进行九级加密，分别是 Grøstl、Blue Midnight Wish、Keccak、JH、Skein、Blake。

https://github.com/Neisklar/quarkcoin-hash-python

#### Lyra2rev2
**算法简介**  
Lyra2rev2 算法由一系列不同的哈希算法组成，其中包括 Blake、Keccak、Cubehash、LYRA2、Skein、Cubehash、Blue Midnight Wish

https://github.com/zsulocal/lyra2rev2_hash

#### NIST5
**算法简介**  
NIST5 算法由五个 SHA-3 候选算法组成，分别是 Blake、Grøstl、JH、Kecaak、Skein。

https://github.com/talkcoin/nist5_hash

#### Lbry
**算法简介**  

LBRY 由 SHA-256、SHA-512 和 RIPEMD 组成。

https://github.com/splxsg/hashing_lbry

https://github.com/lbryio/lbrycrd/blob/master/src/hash.cpp



### Cuckoo cycle
项目文档https://eprint.iacr.org/2014/059.pdf

项目代码https://github.com/tromp/cuckoo  
**算法简介**  

1. 对新块头进行哈希处理，nonce 将用作 siphash 函数的输入，siphash 的输出作为 cuckoo 图的节点生成函数的输入，得到 cuckoo 图的所有节点；
2. nonce 将用作 SIPHASH 函数的输入，该函数将为图中的每个元素生成位置对； 
3. 通过剪边，执行 Cuckoo 循环检测算法试图在生成的图中找到解（即长度为 42 的循环）； 
4. 对找到的环进行 Blake2b 哈希并将其与当前目标难度进行比较； 
5. 如果哈希难度大于或等于目标难度，则将块广播到网络，并在下一个块开始工作； 
6. 如果没有找到解决方案，则将区块头中的 Nonce 增加 1，并更新时间戳，以便下一次哈希值迭代。

随着图的规模增大，L 的增大，问题的难度也随之增大。图的稀疏度为 $M/N = 1/2$，平均的度为 1，找到一个长度为 L 的难度为 1/L。要保证 POW 的工作量证明的安全性和公平性，意味着需要所有参与方无法通过某种方法来提高解决问题的概率。Cuckoo Cycle 存在的概率，和图的节点多少，边的多少有关，随着 M、N 的增加，图中寻找到 L 大小的环路概率会趋于稳定。


## 第五层 密码学协议层

### Bulletproofs最常用的开源库

- 地址：https://github.com/dalek-cryptography/bulletproofs/
- 简述：使用 Rust 语言编写；采用 Curve25519 曲线；实现了 Range Proof；arbitrary statement 处在开发中；该库实现了对单个或多个范围的单方证明；实现了在线多方计算，用于在多方之间进行范围证明的聚合；实现了用于R1CS约束系统以及证明和验证任意语句的证明（不稳定，正在开发中，具有该`yoloproofs`功能）；未来将实现用于聚集约束系统证明的在线多方计算。

### Bulletproofs中密码学相关的依赖关系

- **Pedersen承诺**

  Bulletproofs对需要证明的证明值v生成Pedersen承诺V，从而确保承诺V和金额v的绑定关系。


- **椭圆曲线**
  该库中的Bulletproofs采用Curve25519椭圆曲线

  介绍：https://tools.ietf.org/html/rfc7748

  实现：https://doc.dalek.rs/curve25519_dalek/inde x.html（并行计算时，是libsecp256k1速度的两倍）

- **内积证明**
  Bulletproofs中提出的Range Proof,使用了内积证明，并且改进了内积证明，将范围证明的大小降低至对数级别。

- **非交互转化**
  该库中的Bulletproofs使用Merlin库（基于Fiat-Shamir heuristic），用于将交互式证明转化为非交互式证明
  简介及实现：https://doc.dalek.rs/merlin/index.html

- **MiMC哈希算法**
  简介：MiMC哈希算法一种适用于ZK、MPC、FHE的低乘法复杂度的高效哈希算法，其对电路更加友好，有利于实现Bulletproofs中算术电路的零知识证明。
  文档：https://www.researchgate.net/publication/309772946_MiMC_Efficient_Encryption_and_Cryptographic_Hashing_with_Minimal_Multiplicative_Complexity