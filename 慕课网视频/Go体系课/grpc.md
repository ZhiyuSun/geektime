协议：HTTP2

二进制，高效的传输
流式传输（streaming）
多路复用（fully multiplexed）
安全性提升

方法：POST

数据：二进制数据流

数据结构：protobuf

protobuf

protocolbuffers下载

mac要设置.zshrc或.bashprofile ?

protobuf 下载工具

protobuf-gateway

trip.proto 定义模型

protoc -I =. --go_out=paths=source_relative

grpc gateway  restapi转发

trip.yml 定义http的格式

protoc -I = .--grpc-gateway_out=source_relative,grpc_api_configuration=trip.yml

http可以直接在浏览器上测试

grpc跑在8081，ggateway跑在8080

开发步骤：
- 第一步：写proto
    - 定义message, service
- 第二步，genProto.sh
- 