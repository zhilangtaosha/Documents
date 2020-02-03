# Pedersen commitment

Pedersen commitment 承诺协议基于离散对数群构建的密码元语，属于构建密码库的第三层。我们参照libsecp256k1上Pedersen Commitment的视线，列出项目实现时候的依赖关系、实现方法。

##  Pedersen commitment依赖
我们选择基于secp256k1椭圆曲线群，描述Pedersen commitment 在代码实现上的依赖关系
基于  [该库](https://github.com/ElementsProject/secp256k1-zkp)

### 理论工具依赖关系
第一层：

大整数结构和常见运算，随机数产生

第二层：

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

##  Pedersen commitment 实现方法
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


# 基于BLS的聚合签名

# 基于BLS的门限签名

# 分布式密钥生成 GJKR07方案