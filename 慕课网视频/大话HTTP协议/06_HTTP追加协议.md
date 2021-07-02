# 基于HTTP追加协议

## HTTP协议的瓶颈

- 一条连接上只可发送一个请求
- 请求只能从客户端开始。客户端不可以接受除响应以外的指令。
- 请求/响应头部不经压缩就发送
- 每次互相发送相同的头部造成的浪费较多
- 非强制压缩发送

## websocket与HTTP

WebSocket的“握手”

upgrade:websocket
connection:upgrade

sec-websocket-key: xxx
sec-websocket-protocol: chat, superchat
sec-websocket-version: 13

ajax轮询
long poll 长轮询
非常消耗资源

真正的全双工方式
减少通信量

