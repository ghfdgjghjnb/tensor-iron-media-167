# ESP32：HTTP 客户端获取天气



## 说明

ESP32 作为一个 HTTP 客户端，通过 WiFi 请求免费的天气 API，获取某城市的实时天气数据并解析 JSON。学习 HTTP 请求、JSON 解析。



## 硬件

- ESP32 ×1



## 代码

```cpp

#include <WiFi.h>

#include <HTTPClient.h>

#include <ArduinoJson.h>



const char* ssid = "你的WiFi名";

const char* password = "你的WiFi密码";



// 使用免费的天气 API（以 wttr.in 为例）

String getWeatherURL(String city) {

  return "http://wttr.in/" + city + "?format=j1";

}



void setup() {

  Serial.begin(115200);



  WiFi.begin(ssid, password);

  while (WiFi.status() != WL_CONNECTED) {

    delay(500); Serial.print(".");

  }

  Serial.println("\nWiFi 已连接");

}



void loop() {

  HTTPClient http;

  String url = getWeatherURL("Beijing");



  Serial.print("请求: "); Serial.println(url);

  http.begin(url);



  int httpCode = http.GET();



  if (httpCode == 200) {

    String payload = http.getString();



    // JSON 解析

    StaticJsonDocument<2048> doc;

    DeserializationError error = deserializeJson(doc, payload);



    if (error) {

      Serial.print("JSON 解析失败: ");

      Serial.println(error.c_str());

    } else {

      const char* city   = doc["nearest_area"][0]["areaName"][0]["value"];

      const char* country = doc["nearest_area"][0]["country"][0]["value"];

      const char* temp    = doc["current_condition"][0]["temp_C"];

      const char* desc    = doc["current_condition"][0]["weatherDesc"][0]["value"];

      const char* humidity = doc["current_condition"][0]["humidity"];



      Serial.println("========== 天气报告 ==========");

      Serial.print("城市: "); Serial.print(city);

      Serial.print(", "); Serial.println(country);

      Serial.print("温度: "); Serial.print(temp); Serial.println(" °C");

      Serial.print("天气: "); Serial.println(desc);

      Serial.print("湿度: "); Serial.print(humidity); Serial.println(" %");

      Serial.println("================================");

    }

  } else {

    Serial.print("HTTP 请求失败，错误码: ");

    Serial.println(httpCode);

  }



  http.end();



  // 每 60 秒更新一次

  delay(60000);

}

```



## 教学重点

- `HTTPClient` 库用于发起 HTTP 请求

- `http.begin(url) → http.GET() → getString() → http.end()` 标准流程

- ArduinoJson 解析 JSON：`deserializeJson()` + 路径访问

- 免费 API（wttr.in）友好且无需注册

- `StaticJsonDocument<N>` 静态分配内存，N 需根据数据量设定



## 常见错误

- URL 硬编码 HTTP 而非 HTTPS → API 拒绝服务

- JSON 结构路径写错 → 访问到空指针

- `StaticJsonDocument` 大小不足 → 解析失败（增大 N）

- 频繁请求被 API 限制 → 加延时

