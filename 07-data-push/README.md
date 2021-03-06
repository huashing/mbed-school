# 第 7 章：IoT 物件與 Websocket 技術

WebSocket 是 HTML5 裡的一個標準，它是一種 TCP 的連線技術。在協定部份，則是基於 HTTP（Over HTTP）協定。因此，WebSocket 標準定義了一些 HTTP Headers 來進行 Client/Server 的通訊。

Websocket 能讓 client 與 server 能建立永續性的 TCP 連線。簡單來說，有了 Websocket，就能實作出 real-time data streaming 機制。

以下將說明 IoT 第 4 階段，也就是 WoT 最重要的一個觀念：使用 Websocket channel server 來封裝 IoT objects，讓 IoT devices 成為抽象化的 Websocket server [1]。

[1]: http://www.jollen.org/blog/2015/01/arm-mbed-iot-objects-websocket.html

## 關於 IoT 與 Websocket

一般來說，Websocket 的使用案例（use case）是 server push（data push）機制，也就是說，ARM mbed 物件本身，應該是扮演 Websocket server 的角色。但現實層面，讓 IoT 扮演 Websocket server 的話，會有幾個技術問題：

* ARM mbed 要管理 client 端的 connections
* 需要更多的內存來維護 client connections
* 需要實作 Data push 的演算法，例如：round-roubin
* 要考量 error handling 與 exception handling

因此，最簡單的 scenario 設計如下：

1. 佈署專用的 Websocket channel server
2. ARM mbed 將 data 即時推送（push）到 Websocket channel server
3. 用戶（user）與 Websocket channel server 建立 Websocket connection
4. 用戶接收 Websocket channel server 的即時資料（經由 server push）

抽像上來看，ARM mbed 仍然是 server 端，而不是 client 端；真正的 client 端是用戶。

## Websocket Channel Server

Websocket channel server 扮演封裝 IoT 物件的角色，對 Websocket server 來說，只要能定義好「channel」的結構，就能封裝數以萬計、千萬計的 IoT 物件。「抽像上來看，ARM mbed 仍然是 server 端」，就是這樣的觀念。

ARM mbed 官方就提供了 Websocket channel server 的服務：

```
http://sockets.mbed.org
```

## ARM mbed 與 Websocket Client

了解上述觀念後，技術的實作就能區隔為二個部份：

* Websocket channel server 服務，本書將使用 *sockets.mbed.org*
* ARM mbed 的 Websocket client 實作

## ARM mbed 的 Websocket client 實作

使用 Websocket Library ，引入 *Websocket.h* 標頭檔後，建立 Websocket 連線後傳送 "WebSocket Hello World over Ethernet" 至 Websocket channel server 。

```
#include "mbed.h"
#include "EthernetInterface.h"
#include "Websocket.h"

int main() {
    char recv[1024];
 
    EthernetInterface eth;
    //eth.init(); // Use DHCP
    eth.init("192.168.21.33", "255.255.255.0", "192.168.21.2"); // Use static ip
    eth.connect();
    printf("IP Address is %s\n\r", eth.getIPAddress());
 
    Websocket ws("ws://sockets.mbed.org/ws/mbedschool/viewer");

    ws.connect();
 
    while (1) {
        wait(1.0);
        ws.send("WebSocket Hello World over Ethernet");
        wait(1.0);
    }
}
```

![圖 7.1：Websocket channel server](http://i.imgur.com/RGTk67m.png)

圖 7.1：Websocket channel server

