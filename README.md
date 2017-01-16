# 大智慧云交易所

大智慧云交易所提供了开放API使得第三方交易所可以将行情报价发送到大智慧金融信息云服务（简称大智慧云），在收到了行情数据后，
大智慧云会对行情进行额外的增值处理，包括

1. 提供键盘宝API，通过输入拼音简称、全称、代码、汉字等各种方式，获取到相应的商品。
2. 对行情数据进行归档，产生分时、分钟线、K线等历史数据
3. 提供将行情分发至客户端的API。
4. 提供行情展示的开源客户端，包括移动APP、WEB以及PC

# 性能指标

大智慧金融信息云本身处理了包括沪深Level2，上海、郑州、大连期货，港美股、外汇、股指等各大交易所的行情数据。

| 指标       | 能力      | 备注   |
| -------    | ------    | ------ |
| 报价更新   | 每秒10w笔 |        |
| 客户端延迟 | <100ms    |        |
| 商品数目 | 5000    |        |
| 商品数目 | 5000    |        |
| 历史日线 | 10000    |        |
| 历史分钟线 | 10000    |        |
| 历史分时 | 20天    |        |

# 安全性

交易所通过双向认证的HTTPS向大智慧云交易所发送报价信息，保证数据在传送途中不会被篡改，同时对于一些更敏感的操作，提供基于密码对的二次认证。

# 可扩展性

传输协议由protobuf来实现，保证前后兼容，新增字段不影响之前以及之后的实现。

# 技术实现

API的实现基于 [gRPC](http://www.grpc.io/)，gRPC是一个高性能、通用的开源RPC框架，其由Google主要面向移动应用开发并基于HTTP/2协议标准而设计，
基于[ProtoBuf](https://github.com/google/protobuf)(Protocol Buffers)序列化协议开发，且支持众多开发语言。


```python
# 云交易所公钥
root_cert = open('server.pem').read()

# 交易所公钥和私钥
client_key = open('client_key.pem', 'rb').read()
client_cert = open('client_cert.pem', 'rb').read()

# 创建双向认证的HTTPS通道
creds = grpc.ssl_channel_credentials(root_cert, client_key, client_cert)
channel = grpc.secure_channel('localhost:6666', creds)

# 创建API stub
stub = exchange_pb2_grpc.ExchangeAPIStub(channel)

# 创建一个交易所
resp = stub.New(pb.NewRequest(appid='123456', secret='123456', name='测试交易所', prefix='BY'))
uuid = resp.uuid
print(resp)

# 为该交易所增加交易品种
resp = stub.NewSymbols(pb.SymbolsRequest(uuid=uuid, symboles=list(symbol_feed(symbols_count))))
print(resp)

# 发送报价
for i in range(1, 10):
    stub.Report(report_feed(uuid, randint(1, 10)))
```

exchangeAPI 参看 `exchange.proto` 的定义。 