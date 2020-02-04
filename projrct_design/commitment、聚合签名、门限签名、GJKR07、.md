# Pedersen commitment

Pedersen commitment 承诺协议基于离散对数群构建的密码元语，属于构建密码库的第三层。我们参照libsecp256k1上Pedersen Commitment的实现，列出项目实现时候的依赖关系、实现方法。

##  依赖
我们选择基于secp256k1椭圆曲线群的构造，描述Pedersen commitment 在代码实现上的依赖关系
基于  [该库](https://github.com/ElementsProject/secp256k1-zkp)

### 理论工具依赖关系
**第一层**：

大整数结构和常见运算，随机数产生

**第二层**：

 secp256k1 椭圆曲线，GF(p)中曲线上的点结构及运算：点的乘法、盲化乘法、指数乘法等

### 代码上的依赖

**一级调用**

```
#include "eckey.h" // 椭圆曲线公私钥的操作
#include "ecmult_const.h" // secp256k1_ecmult_const
#include "ecmult_gen.h" // SECP256K1椭圆曲线的初始化、构建
#include "group.h" // SECP256K1群的各类运算、初始化
#include "field.h" // SECP256K1有限域上的各类运算、初始化
#include "scalar.h" //标量库 用于序列化盲化因子
```

```
# include "secp256k1.h" //定义调用的API
# include "secp256k1_generator.h"
``` 

## 实现方法
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


# 基于BLS的门限签名
基于BLS签名的门限签名方案属于第四层密码方案层，其密码原语基于双线性对、椭圆曲线。我们参照 [libBLS](https://github.com/skalenetwork/libBLS)，列出项目实现时候的依赖关系、实现方法。

## 依赖
我们选择基于alt_bn128椭圆曲线群的方案，
### 理论工具依赖关系
**第一层**：

大整数结构和常见运算，随机数产生

**第二层**：

alt_bn128椭圆曲线群
双线性对

**第三层**：

BLS签名

### 代码实现依赖关系
alt_bn128椭圆曲线群：[libff](https://github.com/scipr-lab/libff)椭圆曲线库   
双线性对：[pbc by Ben Lynn](https://github.com/skalenetwork/pbc)

## 实现方法、类

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


# 基于BLS的聚合签名
基于BLS签名的聚合签名方案属于第四层密码方案层，其密码原语基于双线性对、椭圆曲线。我们参照 [libBLS](https://github.com/skalenetwork/libBLS)，列出项目实现时候的依赖关系、实现方法。

## 依赖
我们选择基于alt_bn128椭圆曲线群的方案，
### 理论工具依赖关系
**第一层**：

大整数结构和常见运算，随机数产生

**第二层**：

alt_bn128椭圆曲线群
双线性对

**第三层**：

BLS签名

### 代码实现依赖关系
alt_bn128椭圆曲线群：[libff](https://github.com/scipr-lab/libff)椭圆曲线库   
双线性对：[pbc by Ben Lynn](https://github.com/skalenetwork/pbc)

## 实现方法、类

```

BLSSigShareSet - class for a set of pieces of signature. Has methods Add (to add a piece of signature) and merge ( to get common signature, if enough pieces of signature added)

BLSSignature - class for common signature.

```


# 分布式密钥生成 GJKR07方案

分布式密钥生成方案属于第四层密码方案层，其密码原语基于双线性对、椭圆曲线。我们参照 [libBLS](https://github.com/skalenetwork/libBLS)，列出项目实现时候的依赖关系、实现方法。该方案用于BLS门限签名时的密钥生成。

## 依赖
我们选择基于alt_bn128椭圆曲线群的方案，
### 理论工具依赖关系
**第一层**：

大整数结构和常见运算，随机数产生

**第二层**：

alt_bn128椭圆曲线群
双线性对

**第三层**：

BLS签名

### 代码实现依赖关系
alt_bn128椭圆曲线群：[libff](https://github.com/scipr-lab/libff)椭圆曲线库   
双线性对：[pbc by Ben Lynn](https://github.com/skalenetwork/pbc)

## 实现方法、类

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
**方法交互过程**：

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



