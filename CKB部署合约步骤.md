# CKB部署合约步骤

## 基础准备
ckb节点  运行在测试网 版本 0.28.0 (728eff2 2020-02-04).   
ckb-ruby-sdk    版本 0.28.0   
一些测试币.  



##  主要参考
- [编译合约、部署合约](https://xuejie.space/2019_07_13_introduction_to_ckb_script_programming_script_basics/)
- [开启debug](https://docs.nervos.org/dev-guide/debugging-ckb-script.html#debug-syscall)
- [获取测试币](https://docs.nervos.org/dev-guide/faucet.html)


## sdk命令概览

***与参考网址不同的话以此处为主***

```ruby
wallet = CKB::Wallet.from_hex("0x92...31")  # 从私钥生成钱包
api = CKB::API.new(host:"http://127.0.0.1:8114")  # 连接CKB节点 换成自己节点端口 默认8114
############################################
# 程序上链

data = File.read("main")  # 读入编译好的程序
data.bytesize
carrot_tx_hash = wallet.send_capacity(wallet.address, CKB::Utils.byte_to_shannon(容量), CKB::Utils.bin_to_hex(data), fee:50000)   # 分配容量 确保 容量 > data.bytesize


############################################
# 生成自定义的交易
tx = wallet.generate_tx(wallet2.address, CKB::Utils.byte_to_shannon(100), fee: 50000) # 生成交易

carrot_data_hash = CKB::Blake2b.hexdigest(data)
carrot_type_script = CKB::Types::Script.new(code_hash: carrot_data_hash, args: "0x")

tx.outputs[0].type = carrot_type_script.dup
carrot_cell_dep = CKB::Types::CellDep.new(out_point: CKB::Types::OutPoint.new(tx_hash: carrot_tx_hash, index: 0))
tx.cell_deps << carrot_cell_dep.dup

tx.witnesses[0] = CKB::Types::Witness.new

tx = tx.sign(wallet.key)

############################################
# 发送交易  如果内有debug函数 ckb节点能看到
api.send_transaction(tx)


```
## 