# Bulletproofs实现的依赖关系

## Bulletproofs最常用的开源库

- 地址：https://github.com/dalek-cryptography/bulletproofs/
- 简述：使用 Rust 语言编写；采用 Curve25519 曲线；实现了 Range Proof；arbitrary statement 处在开发中；该库实现了对单个或多个范围的单方证明；实现了在线多方计算，用于在多方之间进行范围证明的聚合；实现了用于R1CS约束系统以及证明和验证任意语句的证明（不稳定，正在开发中，具有该`yoloproofs`功能）；未来将实现用于聚集约束系统证明的在线多方计算。

## Bulletproofs中密码学相关的依赖关系

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
