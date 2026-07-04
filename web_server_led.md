# ESP32：Web 服务器（控制 LED）



## 说明

用 ESP32 搭建一个微型 Web 服务器，通过手机浏览器远程控制 LED 开关。这是 IoT（物联网）最经典的入门项目。



## 硬件

- ESP32 开发板 ×1

- LED ×1，220Ω 电阻 ×1



## 代码

```cpp

#include <WiFi.h>

#include <WebServer.h>



const char* ssid = "你的WiFi名";

const char* password = "你的WiFi密码";



WebServer server(80);  // HTTP 端口 80

const int LED_PIN = 2; // ESP32 板载 LED



bool ledState = false;



// 首页 HTML

String getHTML() {

  String html = "<!DOCTYPE html><html><head>";

  html += "<meta charset='utf-8'><meta name='viewport' content='width=device-width'>";

  html += "<title>ESP32 灯光控制</title>";

  html += "<style>body{text-align:center;font-family:Arial;margin-top:50px;}";

  html += "button{font-size:30px;padding:20px 40px;margin:20px;border:none;border-radius:10px;color:white;}";

  html += ".on{background:#4CAF50;}.off{background:#f44336;}</style></head><body>";

  html += "<h1>ESP32 灯光控制</h1>";

  html += "<h2>状态: " + String(ledState ? "🟢 开" : "🔴 关") + "</h2>";

  html += "<a href='/on'><button class='on'>开灯</button></a>";

  html += "<a href='/off'><button class='off'>关灯</button></a>";

  html += "</body></html>";

  return html;

}



void handleRoot() {

  server.send(200, "text/html", getHTML());

}



void handleOn() {

  ledState = true;

  digitalWrite(LED_PIN, HIGH);

  server.send(200, "text/html", getHTML());

}



void handleOff() {

  ledState = false;

  digitalWrite(LED_PIN, LOW);

  server.send(200, "text/html", getHTML());

}



void setup() {

  pinMode(LED_PIN, OUTPUT);

  digitalWrite(LED_PIN, LOW);

  Serial.begin(115200);



  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {

    delay(500); Serial.print(".");

  }

  Serial.print("\nIP: "); Serial.println(WiFi.localIP());



  server.on("/", handleRoot);

  server.on("/on", handleOn);

  server.on("/off", handleOff);

  server.begin();

  Serial.println("服务器已启动！");

}



void loop() {

  server.handleClient();  // 处理客户端请求

  delay(10);

}

```



## 教学重点

- ESP32 作为 Web Server 的核心是 `WebServer` 库

- `server.on(path, handler)` 注册路由处理函数

- `server.send(code, contentType, body)` 发送 HTTP 响应

- 内置 HTML/CSS 实现手机友好的控制页面

- `server.handleClient()` 必须在 `loop()` 中持续调用



## 常见错误

- 路由冲突：`/on` 和 `off` 可能被解析为同一路由

- 忘记 `handleClient()` → 浏览器永远打不开

- HTML 字符串太长 → 考虑用 PROGMEM 存储

- 局域网才能访问，外网需要端口映射

