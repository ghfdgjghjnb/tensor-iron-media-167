# ESP32：WiFi 连接与扫描



## 说明

ESP32 最大亮点是内置 WiFi。这个项目教你如何扫描周围 WiFi 热点、连接到指定网络。理解 WiFi 库的基本用法。



## 硬件

- ESP32 开发板 ×1



## 代码

```cpp

#include <WiFi.h>



void setup() {

  Serial.begin(115200);

  

  // 1. 扫描周围 WiFi

  Serial.println("正在扫描 WiFi...");

  int n = WiFi.scanNetworks();

  Serial.print("发现 "); Serial.print(n); Serial.println(" 个网络:");

  

  for (int i = 0; i < n; i++) {

    Serial.print("  ");

    Serial.print(i + 1);

    Serial.print(": ");

    Serial.print(WiFi.SSID(i));

    Serial.print(" (");

    Serial.print(WiFi.RSSI(i));  // 信号强度 dBm

    Serial.print(" dBm) ");

    // 加密类型

    switch (WiFi.encryptionType(i)) {

      case WIFI_AUTH_OPEN: Serial.println("开放"); break;

      case WIFI_AUTH_WEP:  Serial.println("WEP");  break;

      case WIFI_AUTH_WPA_PSK: Serial.println("WPA"); break;

      case WIFI_AUTH_WPA2_PSK: Serial.println("WPA2"); break;

      default: Serial.println("其他"); break;

    }

    delay(10);

  }



  // 2. 连接到指定 WiFi

  Serial.println("\n正在连接到 WiFi...");

  WiFi.begin("你的WiFi名", "你的WiFi密码");



  int attempts = 0;

  while (WiFi.status() != WL_CONNECTED && attempts < 30) {

    delay(500);

    Serial.print(".");

    attempts++;

  }



  if (WiFi.status() == WL_CONNECTED) {

    Serial.println("\n连接成功！");

    Serial.print("IP 地址: ");   Serial.println(WiFi.localIP());

    Serial.print("子网掩码: ");  Serial.println(WiFi.subnetMask());

    Serial.print("网关: ");      Serial.println(WiFi.gatewayIP());

    Serial.print("MAC 地址: ");  Serial.println(WiFi.macAddress());

  } else {

    Serial.println("\n连接失败！");

  }

}



void loop() { }

```



## 教学重点

- ESP32 的 `WiFi` 库是 Arduino 风格的封装

- `WiFi.scanNetworks()` 扫描可用网络，`RSSI()` 返回信号强度

- `WiFi.begin(ssid, password)` 连接 WiFi

- 连接是异步的，通过 `WiFi.status()` 轮询连接状态

- MAC 地址和 IP 地址是网络基础知识



## 常见错误

- 串口波特率不匹配（ESP32 常用 115200）

- WiFi 密码大小写错误

- 2.4GHz 和 5GHz：ESP32 只支持 2.4GHz

- 连接超时没设置重试次数导致无限等待

