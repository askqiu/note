# whatis
WebSocket是一种网络通信协议，允许在客户端和服务器之间建立持久的双向连接。与传统的HTTP请求相比，WebSocket能实时传输数据，适用于需要频繁交换信息的应用，如在线游戏、聊天应用和实时通知系统。

# 建立websocket连接
通过JavaScript建立、
var ws=new WebSocket("wss://website.com/chat");
通过http执行握手
```
GET /chat HTTP/1.1
Host: normal-website.com
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: wDqumtseNBJdhkihL6PW7w==
Connection: keep-alive, Upgrade
Cookie: session=KOsEJNuflw4Rd9BDNrVmvwBF9rEijeE2
Upgrade: websocket
```
如果服务器接受连接，它将返回 WebSocket 握手响应，如下所示：

```
HTTP/1.1 101 Switching Protocols
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Accept: 0FFP+2nmNIf/h+4BP36k9uzrYGk=
```
此时，网络连接保持打开状态，可用于向任一方向发送 WebSocket 消息。
建立 WebSocket 连接后，客户端或服务器可以向任一方向异步发送消息。
可以使用客户端 JavaScript 从浏览器发送一条简单的消息，如下所示：
ws.send("Peter Wiener");
原则上，WebSocket 消息可以包含任何内容或数据格式。在现代应用程序中，通常使用 JSON 在 WebSocket 消息中发送结构化数据。
例如，使用 WebSockets 的聊天机器人应用程序可能会发送如下消息：
**{"user":"Hal Pline","content":"I wanted to be a Playstation growing up, not a device to answer your inane questions"}**

虽然视频是持续的数据流，但它通常使用专门的流媒体协议（如HLS或DASH）来优化传输和缓冲。WebSocket适用于需要低延迟和实时交互的场景，而视频流的要求不同，更关注带宽和缓冲效率。